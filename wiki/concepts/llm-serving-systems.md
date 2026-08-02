---
type: concept
aliases: [LLM serving, サービング, サービングシステム, serving system, vLLM, SGLang, TensorRT-LLM, Triton, continuous batching, in-flight batching, 継続的バッチング, PagedAttention, RadixAttention, prefix cache, prefix キャッシュ, プレフィックスキャッシュ, chunked prefill, prefill decode disaggregation, PD 分離, TTFT, TPOT, SLO, Roofline, ルーフラインモデル, 演算密度, arithmetic intensity, memory-bound, compute-bound, CUDA Graph, FlashInfer, NanoFlow, Multi-LoRA, Punica, S-LoRA, Jenga, StreamingLLM, 構造的デコーディング, structured decoding, model cascade, モデルカスケード, ルータ, router, Ray Serve, BentoML, OpenRouter, shadow deployment, シャドー展開, expert parallelism, エキスパート並列, EP]
tags: [llm-serving-systems, llm-inference-optimization, model-quantization, transformer-architecture]
related:
  - "[[llm-inference-optimization]]"
  - "[[llm-programming-systems]]"
  - "[[model-quantization]]"
  - "[[low-precision-training]]"
  - "[[transformer-architecture]]"
  - "[[mixture-of-experts]]"
  - "[[context-engineering]]"
  - "[[agent-observability]]"
summaries:
  - "[[summaries/2023-pagedattention]]"
  - "[[summaries/2023-sglang]]"
  - "[[summaries/2025-llm-serving-techniques]]"
  - "[[summaries/2025-understanding-llm-serving]]"
  - "[[summaries/2026-llm-optimization-guide]]"
  - "[[summaries/2024-deepseek-v3]]"
  - "[[summaries/2022-flashattention]]"
updated: 2026-08-03
---

# LLM Serving Systems（LLM サービングシステム）

**サービングシステム**とは、訓練済みの LLM（Large Language Model, 大規模言語モデル）を使って、**入出力長がばらばらな多数のクライアントを、限られた GPU で同時に捌く**ソフトウェアである。vLLM・SGLang・TensorRT-LLM がその代表にあたる。

このページを [[llm-inference-optimization]] から切り出したのは、**解いている問題が違う**からである。

| | [[llm-inference-optimization]] | **本ページ** |
|---|---|---|
| 問い | **1 回の前向き計算をどう速くするか** | **1 台の GPU で多数のクライアントをどう多重化するか** |
| 主題 | カーネル・アーキテクチャ・精度 | 資源の多重化・スケジューリング・断片化・SLO |
| 近い分野 | HPC・コンパイラ | **OS・分散システム** |
| 代表的な問い | attention の HBM アクセスをどう減らすか | 入出力長が読めないリクエストにメモリをどう配るか |

FlashAttention を 3 倍速くしても、**リクエストが 1 件しか来ていなければ GPU は 99% 遊んでいる**。逆に、カーネルが凡庸でもバッチサイズを 100 倍にできれば実効スループットは桁で変わる。この層が独立した主題である理由がそこにある。

**エージェントにとっては、この層の性質が経済性を直接決める。** 多ターンの trajectory（エージェントが辿った行動列）・ツール呼び出しの往復・並列サブエージェントは、すべて「同じ prefix を何度も投げ直す」形をしているので、**prefix キャッシュが効くかどうかがコストを左右する**（後述）。

## 出発点 — なぜバッチングが第一戦略なのか

サービングのほぼすべての設計判断は、**「バッチサイズを上げたい、だが KV cache のメモリが足りない」**という 1 本の緊張から出てくる。まずその前半を押さえる。

<figure>

![](../../raw/assets/2025-llm-serving-techniques/roofline.png)

<figcaption>図1: Roofline モデル。横軸が演算密度（1 回のデータアクセスあたり何回計算するか）、縦軸が達成可能な計算効率（FLOPS）。左の斜面が memory-bound、右の平坦部が compute-bound。出典: [[summaries/2025-llm-serving-techniques]] が引用する NERSC のドキュメント。</figcaption>
</figure>

**decode フェーズの線形層は、極端に memory-bound である。** hidden size を $d$、重みを $d\times n$ とすると、リクエスト 1 件の decode は $1\times d$ と $d\times n$ の行列積になる。メモリアクセス量が $d+dn$、計算量が $dn$ なので、**演算密度は $dn/(d+dn)=n/(1+n)\approx 1$**（$n$ は通常数千以上）。

**1 個データを読むごとに 1 回しか計算していない。** 現代の GPU は 1 回のメモリアクセスあたり数百回の演算ができる設計なので、これは帯域の圧倒的な無駄遣いである。

ここが決定的である。**リクエストを 2 件にすると演算密度は約 2 になり、FLOPS も約 2 倍になる。** 計算量が 2 倍になったのに計算効率も 2 倍になるので、**所要時間はほぼ変わらない**——**リクエストを 1 件から 2 件に増やすのは時間的にタダ**である。

memory-bound の領域にいる限りこれが続く。**現代の GPU と LLM の組み合わせでは、バッチサイズ数百でようやく compute-bound になる。** そこまで上げることがサービングの目標になる。

> **prefill は例外である。** 1 リクエストでも複数トークンを同時処理するので compute-bound になりうる。[[summaries/2025-llm-serving-techniques]] は「LLM の文脈ではバッチサイズ＝合計トークン数と考えるとシンプル」と整理している。

この 1 枚の図が、以降のほぼすべてを説明する。**量子化がどこで効くか**（後述）、**GQA を入れると動作領域が変わるか**、**投機的デコーディングがスループットを上げない理由**——すべて「演算密度をどちらへ動かすか」で読める。

## バッチをどう組むか

### Continuous Batching

<figure>

![](../../raw/assets/2025-llm-serving-techniques/static-batching.png)

<figcaption>図2: 素朴なバッチ推論。黄がプロンプト、青が生成、赤が終了トークン。S3 は 2 サイクル目で終わっているのに、最長の S2 が終わる 6 サイクル目までスロットが空のまま無駄になる。出典: [[summaries/2025-llm-serving-techniques]] が引用する Anyscale のブログ。</figcaption>
</figure>

<figure>

![](../../raw/assets/2025-llm-serving-techniques/continuous-batching.png)

<figcaption>図3: Continuous batching。バッチ内の全リクエストの完了を待たず、空きが出た時点で新しいリクエスト（S5）を割り当てる。出典: 同上。</figcaption>
</figure>

**LLM 推論の固有の難しさは、リクエストごとに何トークン出力するかが事前に分からないこと**にある。素朴なバッチングでは、バッチ内で最も長いリクエストが終わるまで他のスロットが遊ぶ。

**continuous batching**（in-flight batching とも）は、終わったスロットに即座に次を入れる。入出力長がばらついても GPU 利用率を高く保てる。

定量的には、**単一リクエスト処理は GPU 容量の約 90% を浪費**しており、バッチ 32 で**トークン単価 −85%**（レイテンシ +20%）、稼働率は **40% → 90%+** に上がる（[[summaries/2026-llm-optimization-guide]]）。

## KV cache をどう配るか

バッチサイズを上げたいが、**KV cache のメモリ**が壁になる。しかも入出力長が読めないので、**リクエストごとにどれだけ確保すべきかが事前に決まらない**。

見積もりが具体的である（[[summaries/2025-llm-serving-techniques]]）。**Llama 2 7B** の 1 トークンあたりの KV cache は

```
32 (layer) × 4096 (hidden size) × 2 (K/V) × 2 (byte/param) = 512 KB
```

最大コンテキスト長 4096 なので、**素朴に最大長ぶん確保すると 1 リクエストあたり 2GB**。80GB の H100 でも、モデル重み 14GB を引いて **同時に 32 リクエスト程度**しか載らない。最近のモデルはコンテキスト長が数十万なので、この実装は成立しない。

### PagedAttention — OS のページングを持ち込む

<figure>

![](../../raw/assets/2023-pagedattention/x6.png)

<figcaption>図4: PagedAttention。attention のキーとバリューのベクトルが非連続なブロックとして格納されている。クエリ "forth" に対し、3 つのブロックが物理メモリ上でばらばらの位置にある。出典: [[summaries/2023-pagedattention]] の Figure 5。</figcaption>
</figure>

固定サイズの物理ブロックを事前に用意し、論理ブロックから物理ブロックへ **block table**（OS のページテーブルにあたる）で変換する。リクエスト終了時にブロックが解放されるので、**断片化が起きず、バッチサイズを限界まで上げられる**。vLLM はリクエストごとの無駄を **1 ブロック以内**に抑える。

**無駄の内訳が 3 つに分解されている**のが原典の見どころである（[[summaries/2023-pagedattention]]）。

| 種類 | 何が起きているか | いつ分かるか |
|---|---|---|
| **予約（reserved）** | 将来のトークンのために確保した分。最終的には使うが、その間ほかのリクエストが使えない | サービング前から |
| **内部断片化** | 最大長に合わせた過剰供給。実際の出力が短ければ丸ごと無駄 | **リクエストが終わって初めて**分かる |
| **外部断片化** | アロケータ（buddy allocation）に由来。**生成トークンに決して使われない** | サービング前から |

**実測が痛烈である——既存システムが KV cache のメモリのうち実際のトークンの状態に使えているのは 20.4〜38.2% にすぎない。** 結果としてスループットは **2〜4 倍**（対 Orca・FasterTransformer）、同時処理数は Orca (Oracle) の 2.2 倍・Orca (Max) の 4.3 倍になる。

**OS の比喩が最後まで使い切られている**点が、この層の性格をよく表している。

| OS | vLLM |
|---|---|
| ページ | KV ブロック |
| バイト | トークン |
| プロセス | リクエスト |
| ページテーブル | block table |
| スワップ空間 | CPU メモリ |
| fork の copy-on-write | **参照カウント ＋ ブロック粒度の copy-on-write** |
| 共有ライブラリ | 共有 prefix |

そして**どこで比喩を捨てるか**も明示されている——**all-or-nothing の退避**（系列のブロックはまとめてアクセスされるので、全部退避するか全くしないか）と、**再計算による復旧**（生成済みトークンを元のプロンプトへ連結すれば 1 回の prefill で作り直せる。**OS では不可能**）である。

> **カーネル単体は遅い、という正直なアブレーションが重要である。** PagedAttention の attention カーネルは block table へのアクセス・追加の分岐・可変系列長の処理という間接参照の税を払うので、**FasterTransformer より 20〜26% 遅い**。**それでも端から端までは 2〜4 倍速い**——より多くのリクエストが載るからである。**カーネルを速くするのでなく memory-bound の領域でバッチサイズを上げることで勝つ**、という前節の Roofline の帰結そのものである。

**実務的な既定値も原典にある。** ブロックサイズは **16**（ShareGPT では 16〜128 が最良だが、Alpaca では系列がブロックより短くなるので大きいブロックが大きく劣化する）。復旧は、**小ブロックなら再計算・大ブロックなら退避**が有利で、**再計算のオーバーヘッドが退避のレイテンシの 20% を超えることは決してない**。

### Prefix キャッシュ — エージェントで最も効く

もう 1 つの性質——**KV cache の prefix は再利用できる**。LLM は causal attention を使うので依存関係が過去→未来にしかなく、**プロンプトの prefix が共通なら対応する KV cache も同じ**である。

<figure>

![](../../raw/assets/2023-sglang/x6.png)

<figcaption>図5: LRU 退避を伴う RadixAttention の 9 時点にわたる動作。2 つのチャットセッション・few-shot 学習のバッチ・自己一貫性サンプリングに応じて、radix tree が分割・共有・退避されていく。緑が新規、青がアクセスされたキャッシュ、赤が退避されたノード。出典: [[summaries/2023-sglang]] の Figure 6。</figcaption>
</figure>

効くのは**システムプロンプトを使う場合・長い文章に複数の質問をする場合・多ターンの会話**である。**prefill の計算量と KV cache のメモリの両方**が減る。

> **系譜を正確に押さえる（本 wiki の以前の記述の訂正）。** 「PagedAttention → prefix キャッシュ」と滑らかに繋ぐのは誤りである。**vLLM 論文の共有 prefix は「提供者が事前定義した prefix のために物理ブロックを予約しておく」手動の仕組み**であり、しかも [[summaries/2023-sglang]] は「**vLLM の論文はこの機能を論じているが、信頼できる高性能なカーネルがないため公開されたコードは対応していない**」と明言している。**自動の prefix 再利用は SGLang の貢献である。**

**RadixAttention** の設計が要点である（[[summaries/2023-sglang]]）。

- **radix tree（基数木）** は trie と違って**辺に可変長のトークン列をラベル付けできる**ので空間効率が良い。キーがトークン列、値が KV cache のテンソルになる
- **LRU で葉ノードを再帰的に退避**する
- **実行中のバッチが使っているノードは退避できない**ので、各ノードが**参照カウンタ**を持つ
- 木の構造は **CPU 上**に置く。**オーバーヘッドは順伝播 17.6 秒に対し木の演算 0.07 秒、キャッシュヒットゼロのトレースでもわずか 0.5%**
- **continuous batching や paged attention と互換**である

**キャッシュだけでは足りず、スケジューリングが組で要る。** 先着順のままだと、大量のリクエストが来たときにスケジューラが切り替えを起こして**キャッシュのスラッシング**が発生する。SGLang は**一致した prefix の長さでリクエストを並べ替える**（cache-aware scheduling）。

> **エージェント設計への直接の含意がここにある。** 多ターンの trajectory は定義上「共通 prefix ＋ 少しずつ伸びる末尾」であり、prefix キャッシュが最もよく効く形をしている。逆に言えば、**プロンプトの可変部分（タイムスタンプ・ランダムな ID・シャッフルしたツール一覧）を先頭に置くと、キャッシュが毎回無効になる**。「安定した接頭辞、変動する末尾」はエージェントのプロンプト設計の実務規律である（→ [[context-engineering]]）。

**この含意には一次の数字がある。** [[summaries/2023-sglang]] は **ReAct エージェント（HotpotQA）で vLLM 比スループット 5.6 倍・レイテンシ 13%** を報告し、理由をこう述べている——**「この改善は主に、エージェントが現在の状態（思考・行動・観測）を続く LLM 呼び出しのためのプロンプトへ追記していく過程に由来する」**。13B・33B でも同様に約 5 倍である。

対照的に、**1 シミュレーションあたり 1 回しか LLM を呼ばない generative agents では 30% の改善に留まる**——連鎖がなければ再利用する prefix も伸びないからである。**「エージェントだから速くなる」のではなく、「状態を追記する形をしているから速くなる」。** ただしそこでも、**タイムスタンプに基づいて異なる速度で変化する複数の引数を含む複雑な再利用パターンを、ランタイムが自動的に検出できる**と報告されている。

## スケジューリング — SLO を 2 つ持つ

サービングの SLO（Service Level Objective, サービス水準目標）は 2 つで持つ。

- **TTFT（Time To First Token）**: リクエストから最初のトークンまで。**prefill が決める**。
- **TPOT（Time Per Output Token）**: 2 トークン目以降 1 個あたりの平均時間。**decode が決める**。

**両立が難しいのが要点である。** 入力の非常に長いリクエストが来たとき、素朴に到着順で処理すると、**そのユーザーの prefill が他のユーザーの TPOT を壊す**。対処は 2 系統ある。

<figure>

![](../../raw/assets/2025-llm-serving-techniques/chunked-prefill.png)

<figcaption>図6: Chunked Prefill。長い入力の prefill を分割し、1 サイクルで処理するトークン数に上限を設ける。他リクエストの decode を chunk された prefill に相乗り（ピギーバック）させることで TPOT を抑える。出典: [[summaries/2025-llm-serving-techniques]] が引用する NVIDIA のブログ。</figcaption>
</figure>

<figure>

![](../../raw/assets/2025-llm-serving-techniques/pd-disaggregation.png)

<figcaption>図7: Prefill と Decode の分離（disaggregation）。prefill 用の GPU でプロンプトを処理し、KV cache を decode 用の GPU へ送る。TTFT と TPOT を独立に最適化でき、両者を独立にスケールできる。出典: [[summaries/2025-llm-serving-techniques]] が引用する arXiv:2401.09670。</figcaption>
</figure>

- **chunked prefill**: 1 サイクルの処理トークン数に上限を設け、他リクエストの decode を相乗りさせる。**1 台の中で**両立させる。
- **P/D disaggregation**: prefill と decode を**別々の GPU**で走らせ、KV cache を転送する。それぞれに適したバッチ戦略を使え、**独立にスケールできる**。

## 実装の層 — GPU の外側も速くする

### カーネル

**カーネル融合**は複数の GPU カーネルを 1 つにまとめ、起動オーバーヘッドと中間結果のメモリ往復を消す。**FlashAttention はトランスフォーマにおける最も重要なカーネル融合**である（→ [[summaries/2022-flashattention]]）。

**FlashInfer** は FlashAttention と PagedAttention を実装した高性能カーネルライブラリで、多くのサービングシステムが attention のバックエンドに使う。KV cache を **CSR 形式の疎行列**として表現し、`plan`（コア間のロードバランシングを考慮した実行プランを作る）→ `run` の 2 段構えを取る。

<figure>

![](../../raw/assets/2025-llm-serving-techniques/nanoflow.png)

<figcaption>図8: NanoFlow。1 つのバッチをさらに小さく分割し、memory-bound な計算（decode の attention）・compute-bound な計算（prefill の attention、線形層）・GPU 間通信を重ね合わせる。出典: [[summaries/2025-llm-serving-techniques]] が引用する arXiv:2408.12757。</figcaption>
</figure>

**NanoFlow** の着想は、**同じバッチの中に memory-bound な仕事と compute-bound な仕事が混在している**ことを利用して、それらを重ね合わせる点にある。

**Multi-LoRA**（Punica・S-LoRA）は、LoRA の追加パラメータが元モデルの数 % しかないことを利用して、**1 つのベースモデルに複数の LoRA を載せ、リクエストごとに切り替える**。ユーザーごとにチューニングされたモデルを 1 台のサーバから提供できる（→ [[parameter-efficient-fine-tuning]]）。運用側の言葉では「**モジュラー適応**」にあたる（→ [[summaries/2025-understanding-llm-serving]]）。

### CPU オーバーヘッド

**GPU をいくら速くしても、CPU 側が遅ければ全体は速くならない。** 特にモデルが小さいとき、Python のスケジューリングがボトルネックになる。

<figure>

![](../../raw/assets/2025-llm-serving-techniques/cuda-graph.png)

<figcaption>図9: CUDA Graph。多数のカーネル起動やメモリ転送を「グラフ」として事前記述し、以降は 1 回の操作でまとめて実行する。出典: [[summaries/2025-llm-serving-techniques]] が引用する PyTorch のブログ。</figcaption>
</figure>

**CUDA Graph には実務的な制約がある——入出力のバッファサイズが一定でなければならない。** LLM サービングでは attention 層の入出力トークン数がサイクルごとに変わるので、そのままでは使えない。vLLM の V1 は **attention と attention の間だけをグラフ化**し、起動時に `[1, 2, 4, 8, ...]` のバッチサイズのグラフを用意して実行時にパディングする、という回避を採る。

<figure>

![](../../raw/assets/2025-llm-serving-techniques/async-scheduling.png)

<figcaption>図10: 非同期スケジューリング。CPU のスケジューラを GPU と非同期に動かし、GPU が i サイクル目を計算している間に CPU が i+1 サイクル目を準備する。出典: [[summaries/2025-llm-serving-techniques]] が引用する SGLang のブログ。</figcaption>
</figure>

**トレードオフの立て方が示唆的である。** リクエストが i サイクル目で終了トークンを出しても、CPU がそれを知るのは i+1 サイクル目の途中なので、**1 トークン余分に生成してしまう**。その代わりスケジューリングがクリティカルパスから外れ、**全体としては速くなる**。**厳密さを少し捨てて構造的なボトルネックを外す**という一般的なパターンの好例である。

実運用ではさらに、**トークナイザを走らせる部分とモデル推論の部分を別プロセスに分け、ZeroMQ のような IPC でつなぐ**。

## アルゴリズムの層 — 計算を「サボる」

ここまでは**モデル本来の計算を保ったまま**速くする話だが、計算を簡略化する路線もある。

### 量子化はどこで効くのか

<figure>

![](../../raw/assets/2025-llm-serving-techniques/quantization-roofline.png)

<figcaption>図11: 量子化の 2 系統を Roofline モデルで説明したもの。(a) weight-activation quantization は低ビットで計算するので達成可能な FLOPS の上限そのものが上がり（INT4 Peak Ops）、compute-bound の領域でも効く。(b) weight-only quantization は演算密度が上がるだけなので memory-bound の領域でしか効かない。出典: [[summaries/2025-llm-serving-techniques]] が引用する Atom 論文（arXiv:2310.19102）。</figcaption>
</figure>

**これは [[model-quantization]] の「重みのみか、重みと活性か」の軸を、Roofline 図で直接示したものである。**

- **weight-only（GPTQ・AWQ・GGUF）**: 重みが小さくなって演算密度が上がるので memory-bound では効くが、**サービングシステムが大きなバッチで動かすときは効果が限られる**（重みのメモリが減る分バッチサイズを増やせる、という別の効果はある）。
- **weight-activation（LLM.int8()・SmoothQuant・Atom）**: 低ビットで行列積するので **Tensor Core が使え、達成可能な FLOPS の上限そのものが上がる**。compute-bound な大バッチでもスループットが上がる。

**「ローカル LLM なら weight-only、サービングなら weight-activation」**という実務の線引きが、この 1 枚から読める。

### KV cache のスパース化

**StreamingLLM** が代表例である。全部のキャッシュを持つとメモリが増え、直近 N トークンだけ持つ（Window Attention）と精度が大きく落ちる。**系列の最初の数トークンの KV cache だけは常に保持する**ことで、メモリを一定に保ったまま半永久的に生成を続けられる。

根拠は**「文頭のトークンは人間的には意味がなくても、LLM 的には attention スコアが高くなる」**というヒューリスティクスである（理論ではない）。ほかに H2O・Quest・SeerAttention があり、**DeepSeek の NSA** はこれらの知見をまとめてモデルごと訓練している。

### 投機的デコーディング — 個人の体感とシステム効率は一致しない

<figure>

![](../../raw/assets/2025-llm-serving-techniques/speculative-decoding.png)

<figcaption>図12: 投機的デコーディング。緑がドラフトモデルの提案でターゲットに採用されたトークン、赤が却下、青がターゲットが代わりに生成したトークン。1 行がターゲットモデルの 1 サイクル。約 40 トークンの生成にターゲットは 9 回しか回っていない。出典: [[summaries/2025-llm-serving-techniques]] が引用する arXiv:2211.17192。</figcaption>
</figure>

**サービングの視点からの但し書きが重要である。** 投機的デコーディングは**ユーザーが体験する TPOT は下げるが、却下されたドラフトの計算は無駄になるので、全体のスループットは必ずしも上がらない**。

**「誰にとって速いのか」を明示せずに高速化を語ってはいけない**、という一般則の実例である。

### 構造的デコーディング — 正しさの機能が高速化でもある

<figure>

![](../../raw/assets/2025-llm-serving-techniques/structured-decoding.png)

<figcaption>図13: 構造的デコーディングによる高速化。オレンジのトークンは制約によって一意に定まるため LLM に予測させる必要がない。JSON のキー（"name" など）は順序が決まっていれば一意、寮の名前も valid なリストを指定しておけば「G」が出た時点で Gryffindor に確定する。出典: [[summaries/2025-llm-serving-techniques]] が引用する LMSYS のブログ。</figcaption>
</figure>

出力形式を正規表現や文脈自由文法で制約すると（`xgrammar` など）、**次のトークンが一意に確定する箇所では LLM を呼ぶ必要がない**。

**ツール呼び出しの JSON は、まさにこの形をしている。** スキーマでキーの順序と列挙値が決まっていれば、実際に生成が要るのは値の自由記述部分だけになる。**構造化出力を「モデルに正しい形式を守らせる仕組み」としてだけ見ると、この高速化を取り逃す**（→ [[tool-use-and-function-calling]]）。

## アーキテクチャがサービングを決める

モデル側の設計判断が、サービングシステムの動作領域を決めてしまう。

- **MQA / GQA / MLA** は 1 トークンあたりの KV cache を減らす。クエリヘッド 32・KV ヘッド 8 の GQA なら **MHA の 4 分の 1**、すなわち**バッチサイズを 4 倍にできる**。
- **NanoFlow 論文の指摘が鋭い**——一般的な入出力長のもとでは、**GQA の有無によって memory-bound か compute-bound かが切り替わる**。
- **SWA（Sliding Window Attention）** は過去 N トークンだけを見る構成（Mistral 7B）。最近は**レイヤごとにグローバル attention と SWA を混ぜる**構成が増えている（Gemma 3・Apple Intelligence・Command A）。**Mamba などの SSM も、キャッシュサイズが固定という点では SWA の類似**と読める。
- こうした多様なキャッシュ形式に対して、**Jenga** は PagedAttention を一般化し、SWA や SSM のキャッシュも包括的に扱う。

系譜の詳細は [[transformer-architecture]] にある。ここで押さえるべきは、**モデル選定がサービング設計の前提条件になる**ということである。

## 分散サービング

推論でも 3D 並列（DP/PP/TP）が使われるが、**訓練とは効き方が違う**。

| 並列化 | メモリ | スループット | レイテンシ | 通信 |
|---|---|---|---|---|
| **DP**（データ並列） | 減らない | 台数に比例 | **下がらない** | 不要 |
| **PP**（パイプライン並列） | 減る | 上がる | **下がらない** | レイヤ間の活性のみ |
| **TP**（テンソル並列） | 減る | 上がる | **下がる** | All-Reduce が必要で多い |

**訓練との違い**として、推論の PP では backward がないので**パイプラインのバブルが基本発生しない**。MoE では **EP（エキスパート並列）** も必須になるが、**All-to-All** の通信コストとデバイス間のロードバランシングが課題になる（→ [[mixture-of-experts]]）。

実務では組み合わせる——**ノード間は PP でノード内は TP**、**MoE なら attention 層は DP で MoE 層は EP**。

**オフローディング**も選択肢で、KV cache の一部を CPU メモリやストレージへ、あるいは特定のレイヤ・エキスパートを CPU へ逃がす。

### 実例 — DeepSeek の推論インフラ

<figure>

![](../../raw/assets/2025-llm-serving-techniques/deepseek-system-overview.png)

<figcaption>図14: DeepSeek-V3/R1 の推論システム構成。prefill と decode が分離され、それぞれ異なる並列度（prefill は DP32 と EP32、decode は DP144 と EP144）とロードバランサを持つ。出典: [[summaries/2025-llm-serving-techniques]] が引用する DeepSeek の open-infra-index。</figcaption>
</figure>

**なぜこの構成になるのかの説明が、本ページの論点をすべて使っている。** 総パラメータ 685B、エキスパート 256 個のうちトークンあたり 8 個しか使わない極端にスパースなモデルなので、効率を上げるには**バッチサイズを非常に大きくする必要があり、メモリの観点から大きな EP が要る**。エキスパート層は EP、それ以外（共有エキスパートを含む）は DP。

**prefill と decode で並列度が違う理由**——prefill はスループット最大化が目的だが、decode はレイテンシ制約でバッチサイズを上げきれない。**decode 側を大きな並列度にすることで GPU あたりのエキスパート数を減らしてレイテンシを下げ、reasoning モデルのように decode トークン数が多いケースに必要なメモリも確保する。**

**ロードバランシングの目標も prefill と decode で違う。**

- **prefill**: 「合計トークン数」と「attention の計算時間」を一定に
- **decode**: 「KV cache の使用量」と「リクエスト数（バッチサイズ）」を一定に

エキスパート側では、使用頻度の統計をもとに**よく使われるエキスパートを複数 GPU に複製し、階層的にロードバランシングする**（EPLB）。

これは [[llm-inference-optimization]] の **HPC co-design**——アーキテクチャとインフラを切り離さずに設計する——の、サービング側の現れである。

## モデル間のルーティング

ここまでは「1 つのモデルをどう効率よく提供するか」だったが、**そもそもどのモデルで応えるか**という層がある（→ [[summaries/2025-understanding-llm-serving]]）。

- **モデルカスケード**: 能力とコストの順にモデルを並べ、**小さいモデルで受け、信頼度の閾値を満たせば返し、満たさなければ大きいモデルへ引き上げる**。単純なクエリと複雑なクエリが混在するとき、平均コストとレイテンシを下げる。
- **ルータ**: 軽量な分類器やルールで、クエリの特徴（トピック・長さ・複雑さ）から**最初に**適切なモデルへ振り分ける。**冗長な計算を避けたいとき**に向く。

**実務上の難所は閾値の設定である。** カスケードは「信頼度が足りなければ引き上げる」という判定に依存するが、**LLM の自己申告の信頼度は当てにならない**ことが知られている（→ [[self-reflection]]）。元記事はこの点に触れていないので、実装するなら別途検証が要る。

## スタックと運用

[[summaries/2025-understanding-llm-serving]] が層ごとに具体名を挙げている（2025 年 4 月時点。**この層は最も速く陳腐化するので、名前より層の分け方のほうが持ちが良い**）。

| 層 | 例 |
|---|---|
| **サービングエンジン** | vLLM、SGLang、TensorRT-LLM、Triton |
| **ルーティング** | Ray Serve、BentoML |
| **展開プラットフォーム** | Hugging Face Inference Endpoints、OpenRouter、Replicate、オンプレミス |
| **監視・実験管理** | MLflow、LangSmith、Weights & Biases |
| **クラウド** | AWS（SageMaker / Bedrock）、Azure（Azure ML / OpenAI Service）、GCP（Vertex AI） |

運用側で押さえる規律は 3 つ。

1. **追跡する指標**——レイテンシ（TTFT と全体）・コスト（トークンあたり／リクエストあたり）・正しさ（正解率・ハルシネーション率・指示追従）。
2. **shadow deployment**——候補モデルを本番と並行して走らせ、ユーザーに影響を与えずに出力を比較する。オフライン評価では捕まらない退行を捕まえる（→ [[agent-observability]]）。
3. **ワークロードのクラス分けと縮退設計**——長コンテキスト／レイテンシ敏感／バッチ志向でクラスを分け、クラスごとに p95/p99 の予算とバッチ方針を定義し、過負荷時の縮退（コンテキスト短縮・小モデル切り替え・低価値トラフィックの拒否）をランブック化する（→ [[summaries/2026-llm-optimization-guide]]）。

## サービングシステムは推論だけのものではない

**強化学習の訓練システムの内部でもサービングシステムが使われている。** RL では LLM に複数の文章を生成させて報酬を与えるので、**訓練システムであっても推論性能が要る**。OpenRLHF・verl・SkyRL はいずれも内部で vLLM を使っている（→ [[reinforcement-learning-from-human-feedback]]）。

「推論を提供するインフラ」と「訓練のインフラ」が分離できなくなっている、という点で、この層の重要性は推論サービスの外へ広がっている。

## 設計論点

- **Roofline に位置づけて考える。** バッチング・量子化・GQA のいずれも「演算密度をどちらへ動かすか」で説明できる。個別の技法を暗記するより応用が利く。
- **スループットとレイテンシは別の目的関数である。** 投機的デコーディングは TPOT を下げるがスループットは上げない。TP はレイテンシを下げるが DP と PP は下げない。**どちらを最適化しているのか、誰にとって速いのかを明示する。**
- **SLO は TTFT と TPOT の 2 つで持ち、両立できない状況を先に想定する。** 長い入力が他のユーザーの TPOT を壊すケースに対して、chunked prefill と P/D 分離のどちらを採るか。
- **prefix の安定性はエージェントのコスト設計そのものである。** プロンプトの可変部分を先頭に置かない。ツール一覧をシャッフルしない。タイムスタンプを冒頭に入れない（→ [[context-engineering]]）。
- **構造化出力は高速化でもある。** ツール呼び出しのスキーマを厳密に定義することは、正しさだけでなく推論サイクル数にも効く。
- **アーキテクチャ選定はサービング設計の前提条件である。** GQA の有無で動作領域が切り替わる。
- **精度への影響は別途検証する。** 本ページの技法群のうち、KV cache のスパース化・量子化・投機的デコーディングは出力を変えうる。とくに**エージェントの多段ツール呼び出しや長い trajectory での累積影響は、本 wiki の原典群では誰も測っていない**（→ [[model-quantization]] に置いた同じ但し書き）。

## 関連ページ

- [[llm-inference-optimization]] — 「1 回の前向き計算を速くする」側。本ページはそこから切り出した
- [[llm-programming-systems]] — フロントエンド側。SGLang はこの 2 つの層の**協調設計**として設計されている
- [[summaries/2023-pagedattention]] — PagedAttention と vLLM の原典
- [[summaries/2023-sglang]] — RadixAttention とキャッシュを意識したスケジューリングの原典。エージェントの 5.6 倍もここ
- [[model-quantization]] — 量子化の詳細。本ページの Roofline 図が「重みのみか、重みと活性か」の根拠を視覚化している
- [[low-precision-training]] — 訓練側の低精度。サービングとは目的が違う
- [[transformer-architecture]] — MQA/GQA/MLA・SWA・SSM の系譜。アーキテクチャがサービングの動作領域を決める
- [[mixture-of-experts]] — EP とエキスパートのロードバランシング
- [[context-engineering]] — prefix キャッシュがエージェントの多ターン trajectory で効く接点
- [[tool-use-and-function-calling]] — 構造的デコーディングが高速化でもあるという接点
- [[agent-observability]] — shadow deployment・トレーシング・ダッシュボード
- [[reinforcement-learning-from-human-feedback]] — RL 訓練の内部でサービングシステムが使われる接点
- [[summaries/2025-llm-serving-techniques]] — 本ページの主要な根拠。技術の体系的な地図
- [[summaries/2025-understanding-llm-serving]] — ルーティング／カスケードとツールスタックの根拠
- [[summaries/2026-llm-optimization-guide]] — 定量値（continuous batching −85%、PagedAttention −55%）と運用の型
- [[summaries/2024-deepseek-v3]] — 本ページが実例として引く推論インフラのモデル側
