---
type: summary
source_path: raw/papers/FlashAttention_ Fast and Memory-Efficient Exact Attention with IO-Awareness.md
source_kind: paper
title: "FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness"
authors: [Tri Dao, Daniel Y. Fu, Stefano Ermon, Atri Rudra, Christopher Ré]
year: 2022
venue: NeurIPS 2022 (arXiv:2205.14135)
ingested: 2026-08-02
tags: [llm-inference-optimization, transformer-architecture, context-engineering, flash-attention, io-awareness]
translation: "[[translations/2022-flashattention]]"
---

# FlashAttention: IO を意識した高速かつメモリ効率のよい厳密 attention

> 原典: [[translations/2022-flashattention]] ・ `raw/papers/FlashAttention_ Fast and Memory-Efficient Exact Attention with IO-Awareness.md`
> 著者: Tri Dao, Daniel Y. Fu, Stefano Ermon, Atri Rudra, Christopher Ré（Stanford / SUNY Buffalo）
> 年・会議: 2022・NeurIPS 2022（arXiv:2205.14135）

## 一言まとめ

attention が遅いのは**計算量が多いからではなく、GPU のメモリ階層の間でデータを往復させすぎているから**だと指摘し、attention 行列を一度も大容量メモリに書き出さずに計算する CUDA カーネルを作った論文。数学は 1 ミリも変えていない（出力は厳密に同一）のに GPT-2 の訓練が 3 倍速くなり、しかも**演算回数はむしろ増えている**。

## 背景と問題意識

トランスフォーマーの self-attention（自己注意, 系列内の全トークン対の関連度を計算する機構）は、系列長 $N$ に対して時間もメモリも $N^2$ で増える。2020〜2022 年、この 2 乗の壁を破るために大量の**近似 attention**——Reformer（ハッシュで疎化）、Linformer（低ランク射影）、Performer（カーネル近似）、Longformer / BigBird（疎パターン）——が提案された。これらは FLOPs（floating point operations, 浮動小数点演算数）を線形〜準線形まで落とすことに成功していた。

**それなのに、どれも普及しなかった。** 本論文はその理由を一言で切っている——「FLOP の削減に注目し、メモリアクセス（IO）のオーバーヘッドを無視しがちだから」。FLOPs を減らしても wall-clock（実時間）が縮まないなら、実務上は何も改善していない。

なぜそうなるのか。GPU のメモリは階層構造をしている。A100 では:

| 階層 | 容量 | 帯域幅 |
|---|---|---|
| オンチップ **SRAM**（Static RAM, 演算器のすぐ隣にある小容量の超高速メモリ） | 192 KB × 108 SM | 約 19 TB/s |
| **HBM**（High Bandwidth Memory, GPU 基板上の大容量メモリ。いわゆる「VRAM」） | 40〜80 GB | 1.5〜2.0 TB/s |
| CPU の DRAM | 1 TB 超 | 12.8 GB/s |

SRAM は HBM より 1 桁速いが、容量は 5 桁小さい。そして**計算速度の伸びがメモリ速度の伸びを上回り続けてきた**結果、いまや多くの演算は「計算が終わるのを待っている」のではなく「データが届くのを待っている」。この状態を**メモリ律速（memory-bound）**と呼び、対義語が**計算律速（compute-bound）**である。両者を分ける指標が **arithmetic intensity（演算強度, メモリアクセス 1 バイトあたりの演算数）**。

ここで attention の標準実装を見ると、$\mathbf{S}=\mathbf{Q}\mathbf{K}^{\top}$ を計算して $N\times N$ 行列を HBM に書き、それを読み直して softmax をかけ、また書き、また読んで $\mathbf{V}$ を掛ける——**$N\times N$ の巨大な中間行列を HBM に 3 往復させている**。しかも softmax は典型的なメモリ律速演算である。近似 attention はこの往復を減らさないまま行列の中身だけ疎にするので、理論上の FLOPs は減っても実時間は縮まない。

## 提案手法

FlashAttention の目標はひとつ、**attention 行列を HBM に書かないこと**。そのためには 2 つの技術的障害がある。

**(1) softmax は全体を見ないと計算できない。** softmax は行全体の和で正規化するので、素朴には行を全部揃えないと確定しない。本論文は**タイリング（tiling, 入力をブロックに切って SRAM に載る単位で処理すること）**と、softmax を統計量つきで分解する式を使ってこれを回避する。各ブロックについて行最大値 $m$ と指数和 $\ell$ の 2 つの統計量だけを持ち歩き、新しいブロックが来るたびに

$$m^{new}=\max(m,\tilde m),\qquad \ell^{new}=e^{m-m^{new}}\ell+e^{\tilde m-m^{new}}\tilde\ell$$

と再スケールして出力を漸進的に更新する。最後まで回すと厳密に正しい softmax になる（付録 C で帰納法により証明）。**これが「厳密（exact）」の意味である——近似ではなく、順序を変えただけ**。

**(2) 逆伝播が attention 行列を必要とする。** 通常、backward pass（逆伝播, 勾配を計算する後ろ向きの計算）は forward で作った $\mathbf{S},\mathbf{P}$ を必要とするので保存しておく。FlashAttention は代わりに、**出力 $\mathbf{O}$ と統計量 $(m,\ell)$ だけを保存し、backward では SRAM 上で attention 行列を作り直す（recomputation, 再計算）**。これは勾配チェックポインティング（gradient checkpointing, 中間値を捨てて必要時に再計算しメモリを節約する技法）の選択的な一形態だが、決定的に違う点がある。**従来の勾配チェックポインティングは「メモリと引き換えに速度を落とす」ものだったのに対し、ここでは HBM アクセスが減るぶん再計算しても速くなる**。

この 2 つが揃うと、行列積 → マスク → softmax → dropout → 行列積のすべてを**単一の CUDA カーネルに融合**でき、HBM に触れるのは入力を読むときと出力を書くときだけになる。

### 中心的な証拠——FLOPs が増えて、速くなる

GPT-2 medium（系列長 1024、ヘッド次元 64、16 ヘッド、バッチ 64、A100）の実測値:

| | 標準 attention | FlashAttention |
|---|---|---|
| GFLOPs | 66.6 | **75.2**（+13%） |
| HBM 読み書き (GB) | 40.3 | **4.4**（9.2 分の 1） |
| 実行時間 (ms) | 41.7 | **7.3**（5.7 倍速） |

この 3 行が本論文の主張のすべてである。**演算は増えたのに 5.7 倍速い。したがって律速していたのは演算ではなかった。**

### IO 計算量と下界

本論文は主張を漸近解析でも裏づける。$M$ を SRAM のサイズとして、標準 attention が $\Theta(Nd+N^2)$ 回の HBM アクセスを要するのに対し、FlashAttention は $\Theta(N^2d^2M^{-1})$ で済む（定理 2）。典型的な $d=64\sim128$、$M\approx100$ KB では $d^2\ll M$ なので何倍も小さい。

さらに**命題 3 が下界を与える**——$[d,Nd]$ の全範囲の $M$ に対して $o(N^2d^2M^{-1})$ 回の HBM アクセスで厳密な attention を計算するアルゴリズムは存在しない。つまり FlashAttention は定数倍を除いて**最適**である。理論的な高速化のアルゴリズム論文が、同時に「これ以上は原理的に無理」と自分で天井を示しているのは珍しい。

### ブロックスパース FlashAttention

副産物として、著者らは近似 attention との組み合わせも示す。ブロック単位のスパース性マスクを与え、ゼロブロックをスキップするだけで IO 計算量は $\Theta(Nd+N^2d^2M^{-1}s)$（$s$ は非ゼロブロック率）になる。**近似 attention がうまくいかなかった理由が IO であるなら、IO を直してから近似すればよい**——という筋の通った位置づけである。

## 実験結果と知見

**訓練速度。**

| 対象 | ベースライン | FlashAttention | 差 |
|---|---|---|---|
| BERT-large（512） | Nvidia MLPerf 1.1 記録 20.0 ± 1.5 分 | 17.4 ± 1.4 分 | **15% 速い**（記録更新） |
| GPT-2 small（1K） | HuggingFace 9.5 日 / Megatron-LM 4.7 日 | 2.7 日 | **3.5× / 1.7×** |
| GPT-2 medium（1K） | HuggingFace 21.0 日 / Megatron-LM 11.5 日 | 6.9 日 | **3.0× / 1.7×** |
| LRA（1K〜4K） | 標準 attention | — | **2.4×**（ブロックスパースで 2.8×） |

パープレキシティ（perplexity, 言語モデルの予測の当てにくさ。低いほどよい）は 3 実装とも一致する。当然で、**モデルの定義を一切変えていない**からである。

**長いコンテキストが「安く」なると何が起きるか。** ここが agent 実務にとって本質的な結果である。

- GPT-2 small をコンテキスト長 4K で訓練しても 3.6 日で終わり、**コンテキスト長 1K の Megatron（4.7 日）より 30% 速く、かつパープレキシティが 0.7 良い**。長い窓は「高価な贅沢」から「安い性能向上」に変わった。
- 長文書分類: MIMIC-III（集中治療室の退院サマリ）で 512 → 16K にすると micro F₁ が 52.8 → 57.1（+4.3）、ECtHR（欧州人権裁判所の判例）で 512 → 8K にすると 72.2 → 80.7（+8.5）。
- **Path-X（系列長 16K）でランダム超えを達成した初のトランスフォーマー**（61.4%）。ブロックスパース版は **Path-256（系列長 64K）で 63.1%**。それまで全トランスフォーマーはメモリ不足になるかランダム性能しか出せなかった。

**attention 単体のベンチマーク。** 系列長 128〜2K で標準実装より最大 3 倍速く、メモリは厳密 attention 比 20 分の 1（系列長に対して線形）。ただし**交差点がある**——系列長 512〜1024 を超えると Linformer など一部の近似手法のほうが速くなる。著者らはこれを隠していない。ブロックスパース版は全系列長で既知のすべての近似手法より速い。

## 限界・批判的視点

**著者ら自身が挙げる限界。**

1. **新しい attention 変種ごとに CUDA を手書きする必要がある。** PyTorch よりずっと低レベルな言語で、GPU アーキテクチャ間の移植性も保証されない。著者らは画像処理の Halide のような「高レベル言語で書いて IO 意識実装へコンパイルする」仕組みを求めている。これは 2022 年時点の要請だが、後の Triton や `torch.compile` の普及はまさにこの穴を埋める動きだった。
2. **単一 GPU に閉じている。** 最適性の主張は 1 台の GPU 上での話であり、GPU 間のデータ転送は IO 解析に入っていない。実際の大規模訓練では attention を 4〜8 GPU に分割するのが普通なので、そこは未解決として残されている。
3. **attention 以外の層は手つかず。** 深層ネットワークの全層が HBM に触れるので、IO を意識した実装は他モジュールにも要る。

**読むときに補うべき点。**

4. **数値は 2022 年の A100 / RTX 3090 / T4 のもの**である。SRAM 容量と HBM 帯域幅の比が世代で変わるので高速化率も変わる（本論文自身、T4 では SRAM が小さいためブロックを縮めざるを得ず高速化が小さいと報告している）。H100 以降・FlashAttention-2/3 では数字は当然異なる。**この論文から持ち帰るべきは倍率ではなく原理のほうである。**
5. **「最適」の範囲は狭く取るべき**。定数倍を除いた漸近的最適であって、実装レベルではまだ改善余地があった。実際 FlashAttention-2（[[summaries/2023-flashattention-2]], 2023）は同じ IO 計算量のまま work partitioning を直して約 2 倍速くなっている——本論文の時点で attention は理論ピークの 25〜40% しか使えていなかった。**同じ漸近計算量の中に 2 倍が残っていた**という事実自体、「FLOPs もオーダーも wall-clock の代理指標としては粗い」という本論文の教訓の再演になっている。
6. **Apex FMHA との比較は正直だが控えめ**（表 7）。短い系列では FMHA に対し系列長 128 で 4% 遅い。FlashAttention の利得は本質的に**長い系列で効く**ものであり、短系列の万能薬ではない。

## 実装・運用上の示唆

- **ボトルネックを測る場所を間違えない。** 「FLOPs を数える」は最適化の出発点として自然に見えるが、メモリ律速の演算では無意味どころか誤導になる。本論文の 66.6 → 75.2 GFLOPs で 5.7 倍速という数字は、その反例として最も引用しやすい形をしている。この教訓は wiki 内で繰り返し現れる——[[llm-inference-optimization]] の「チャンク化とハードウェア整合」（DeltaNet のチャンクサイズは C=1 が FLOPs 最小だが C=64/128 が実効最速）はまったく同じ構図である。
- **再計算はメモリ節約の手段とは限らない。** 「メモリを節約したいときに速度を犠牲にして使うもの」という勾配チェックポインティングの常識が、IO を勘定に入れると反転する。**キャッシュか再計算かの判断は、保存コストではなく往復コストで決めるべき**である。これはエージェントの文脈でも同型で、[[context-engineering]] の「参照渡し（コンテキストに実体を入れず、必要時に取りに行く）」も同じ判断をしている。
- **新しいアーキテクチャ要素を評価するとき、実装の成熟度と数学的性質を分けて見る。** ある手法が遅いのは、その数学が悪いからか、まだ誰も良いカーネルを書いていないからか。FlashAttention の登場は、softmax attention に「まだ誰も良いカーネルを書いていなかった」ぶんの余地が大量に残っていたことを示した。
- **エージェント実務への含意。** FlashAttention 自体はエージェント技術ではない。しかし**長いコンテキストを経済的に成立させた土台**であり、今日のエージェントが数万〜数十万トークンの trajectory（エージェントが辿った行動列）や大量のツール定義をコンテキストに載せられるのは、この系列の仕事の上に立っている。同時に注意すべきは、**窓が安くなったことと、モデルが窓を使いこなせることは別問題**だという点である。ここは [[summaries/2023-lost-in-the-middle]] が示した通りで、FlashAttention は「入れられる量」を増やしたが「入れた情報が使われるか」は解決していない。

## 用語と略称

- **HBM** = High Bandwidth Memory（GPU 基板上の大容量メモリ。A100 で 40〜80 GB、1.5〜2.0 TB/s）
- **SRAM** = Static Random-Access Memory（演算器の隣にあるオンチップの小容量・超高速メモリ。A100 で SM あたり 192 KB、約 19 TB/s）
- **SM** = Streaming Multiprocessor（GPU の演算ユニットの単位。A100 に 108 個）
- **IO** = Input/Output（ここではメモリ階層間の読み書き）
- **FLOPs** = FLoating point OPerations（浮動小数点演算数）
- **wall-clock** = 実時間（理論演算量ではなく実際に経過する時間）
- **arithmetic intensity** = 演算強度（メモリアクセス 1 バイトあたりの算術演算数。compute-bound / memory-bound の判定に使う）
- **tiling** = タイリング（入力を SRAM に載るブロックへ切って処理すること）
- **recomputation** = 再計算（中間値を保存せず必要時に作り直すこと）
- **kernel fusion** = カーネル融合（複数の演算を 1 つの GPU カーネルにまとめ、メモリ往復を省く実装）
- **gradient checkpointing** = 勾配チェックポインティング（中間値を捨てて逆伝播時に再計算しメモリを節約する技法）
- **perplexity** = パープレキシティ（言語モデルの予測の当てにくさ。低いほどよい）
- **LRA** = Long-Range Arena（長系列モデリングを測るベンチマーク集。ListOps / Text / Retrieval / Image / Pathfinder / Path-X）
- **MLPerf** = 機械学習の標準ベンチマークスイート（訓練速度記録の公式な比較基盤）
- **FMHA** = Fused Multi-Head Attention（Nvidia Apex に含まれる、BERT 向けの融合 attention 実装。本論文の出発点）

## 関連ページ

- [[summaries/2023-flashattention-2]] — 直接の続編。IO を直した後に残る壁（占有率と warp 間の分業）を扱い、さらに 2 倍を得た
- [[summaries/2024-flashattention-3]] — 三部作の完結編。Hopper の非同期実行と FP8 を前提に設計し直した
- [[llm-inference-optimization]] — IO-awareness を最適化の第一原理として扱う。本論文が主要な根拠
- [[transformer-architecture]] — attention の系譜の中で、本論文は「近似も圧縮もせず、メモリアクセスだけを変える」直交した軸にあたる
- [[context-engineering]] — 長いコンテキストが経済的に成立する前提を与えた
- [[summaries/2023-lost-in-the-middle]] — 窓を長くできることと、長い窓を使いこなせることは別だと示した対の論文
- [[summaries/2026-gpt2-to-kimi3]] — 「O(N²) の枠づけには惑わされた……それは Flash Attention が解決したこと」と著者が書いた箇所の一次資料
- [[summaries/2026-llm-optimization-guide]] — 本論文を「効率的 attention: 長系列で 2〜4 倍（出力は不変）」として二次引用している
- [[summaries/2024-deepseek-v3]] — FP8 訓練・DualPipe と同じ「アルゴリズムとハードウェアを一緒に設計する」路線の後継
- [[llm-serving-systems]] — 本論文を「トランスフォーマにおける最も重要なカーネル融合」として位置づけ、FlashInfer 等のサービング側の実装へつなぐページ
