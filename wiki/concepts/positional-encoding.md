---
type: concept
aliases: [位置符号化, 位置埋め込み, position encoding, position embedding, RoPE, Rotary Position Embedding, 回転位置埋め込み, 相対位置埋め込み, 絶対位置埋め込み, YaRN, ALiBi, NoPE]
tags: [positional-encoding, transformer-architecture, llm-foundations, context-engineering]
related:
  - "[[transformer-architecture]]"
  - "[[llm-inference-optimization]]"
  - "[[context-engineering]]"
  - "[[llm-serving-systems]]"
  - "[[model-quantization]]"
summaries:
  - "[[summaries/2021-roformer]]"
  - "[[summaries/2023-lost-in-the-middle]]"
  - "[[summaries/2025-deepseek-series]]"
  - "[[summaries/2024-deepseek-v3]]"
  - "[[summaries/2026-deepseek-v4]]"
  - "[[summaries/2026-gemma-4]]"
  - "[[summaries/2024-flashattention-3]]"
  - "[[summaries/2025-kimi-linear]]"
updated: 2026-08-11
---

# Positional Encoding（位置符号化）

**トランスフォーマに「トークンの順序」を教える仕組み**を扱うページ。attention（注意機構）は本質的に集合演算であり、そのままでは語順を見ていない。だから位置情報は外から注入するしかない——その注入の仕方を巡る 8 年分の設計史がここにある。

現在の事実上の標準は **RoPE**（Rotary Position Embedding, 回転位置埋め込み。→ [[summaries/2021-roformer]]）である。LLaMA・Qwen・DeepSeek・Kimi・Gemma を含むほぼすべての現代モデルが、この上に載っている。そしてコンテキスト長を 4K から 128K・1M へ伸ばす作業の実体も、その多くは**位置符号化のパラメータをどういじるか**の話である（→ [[context-engineering]]）。

## なぜ必要なのか — attention は順序を見ていない

self-attention は、各トークンについて他のすべてのトークンとの内積を取って重み付き和を作る。この計算は**入力の並べ替えに対して同変（permutation-equivariant）** である——入力を並べ替えれば出力も同じように並べ替わるだけで、中身は変わらない。「猫が犬を追う」と「犬が猫を追う」の区別がつかない。

RNN は時間方向の再帰で、CNN は受容野で順序を持つ（ただし CNN も、パディング操作を通じて暗黙的に位置を学習しているという指摘がある）。attention だけがこれを持たない。したがって設計問題は「**位置情報をどの形で、計算のどこに注入するか**」に集約される。注入先の候補は 3 つある。

1. **入力ベクトルに足す**（絶対位置埋め込み）
2. **attention スコアに足す**（相対バイアス）
3. **query / key を変換する**（RoPE）

この 3 つの違いが、そのまま以下の系譜になる。

## 系譜

### (a) 絶対位置埋め込み — 足す

位置 $i$ ごとに $d$ 次元ベクトル $\boldsymbol{p}_i$ を用意し、単語埋め込みに加算する: $f(\boldsymbol{x}_i, i) = \boldsymbol{W}(\boldsymbol{x}_i + \boldsymbol{p}_i)$。

- **正弦位置符号化**（sinusoidal, Transformer 原論文 2017）: $\boldsymbol{p}_i$ を $\sin(i/10000^{2t/d})$ と $\cos(i/10000^{2t/d})$ で決定的に生成する。次元 $t$ ごとに周波数が異なり、低次元は速く・高次元はゆっくり変化する。学習不要で、任意の $i$ を計算できる。
- **学習可能な絶対位置埋め込み**（BERT・GPT-2 など）: $\{\boldsymbol{p}_t\}_{t=1}^{L}$ を単に学習する。単純だが、**最大長 $L$ の表を持つので $L$ を超える位置には値が存在しない**。

decoder-only の標準構成では、この位置埋め込みがトークン埋め込みと足されてブロック列に入る（→ [[transformer-architecture]]）。

**共通の弱点**は、絶対位置しか符号化していないのに、言語で意味を持つのは多くの場合**相対的な距離**だという点である。「2 語前の単語」という関係は、文全体が 3 語ずれても変わらないはずである。

### (b) 相対位置埋め込み — 展開項をいじる

そこで、$\boldsymbol{q}_m^\top \boldsymbol{k}_n$ を $\boldsymbol{x}+\boldsymbol{p}$ で展開した 4 項——

$$\boldsymbol{q}_m^\top \boldsymbol{k}_n = \underbrace{\boldsymbol{x}_m^\top \boldsymbol{W}_q^\top \boldsymbol{W}_k \boldsymbol{x}_n}_{\text{内容 × 内容}} + \underbrace{\boldsymbol{x}_m^\top \boldsymbol{W}_q^\top \boldsymbol{W}_k \boldsymbol{p}_n}_{\text{内容 × 位置}} + \underbrace{\boldsymbol{p}_m^\top \boldsymbol{W}_q^\top \boldsymbol{W}_k \boldsymbol{x}_n}_{\text{位置 × 内容}} + \underbrace{\boldsymbol{p}_m^\top \boldsymbol{W}_q^\top \boldsymbol{W}_k \boldsymbol{p}_n}_{\text{位置 × 位置}}$$

——の各項を、絶対位置 $\boldsymbol{p}$ でなく相対位置 $\tilde{\boldsymbol{p}}_{m-n}$ に差し替える方向の研究群が出る。[[summaries/2021-roformer]] の §2.3 がこの系譜を整理しており、本 wiki のこの節はそれを底本にしている。

- **Shaw ら（2018）**: key と value に相対位置埋め込みを足す。距離を $r=\operatorname{clip}(m-n, r_{\min}, r_{\max})$ で**打ち切る**——「ある距離を超えると正確な相対位置は役に立たない」という仮説にもとづく。
- **Transformer-XL（2019）**: 上の 4 項分解を明示的に行い、第 2 項の $\boldsymbol{p}_n$ を正弦符号化された $\tilde{\boldsymbol{p}}_{m-n}$ に、第 3・4 項の $\boldsymbol{p}_m$ を query 位置に依存しない学習ベクトル $\mathbf{u}, \mathbf{v}$ に置き換える。value 側の位置情報は落とす。以降の研究は「**相対位置は attention の重みにだけ入れればよい**」という結論を共有する。
- **T5 系（2020）**: 思い切って $\boldsymbol{q}_m^\top\boldsymbol{k}_n = \boldsymbol{x}_m^\top\boldsymbol{W}_q^\top\boldsymbol{W}_k\boldsymbol{x}_n + b_{i,j}$ とし、**学習可能なスカラーのバイアス $b_{i,j}$ を attention スコアに直接足す**だけにする。
- **DeBERTa 系（2020）**: 中央 2 項だけで相対位置は完全にモデル化できると論じ、$\boldsymbol{p}_m, \boldsymbol{p}_n$ を $\tilde{\boldsymbol{p}}_{m-n}$ に置換する。
- **ALiBi（2021）**: 位置情報を「距離に比例した線形のペナルティ」として attention スコアに足すだけの方式（MPT が採用。→ [[summaries/2023-lost-in-the-middle]]）。埋め込みを一切持たない極端に単純な相対バイアスである。

**この系統に共通する構造的な代償**が 2 つある。

- **$N \times N$ のバイアス行列が要る**（またはそれに相当する計算）。長系列でメモリと帯域を食う。
- **$q$ 側と $k$ 側に分離できない**。$\boldsymbol{x}_m^\top \boldsymbol{W}_q^\top \boldsymbol{W}_k \tilde{\boldsymbol{p}}_{m-n}$ のような項は $m$ と $n$ を同時に含むので、$\phi(\boldsymbol{q}_m)^\top \varphi(\boldsymbol{k}_n)$ の形に書けない。したがって **linear attention（→ [[transformer-architecture]] の attention 系譜）と両立しない**。2021 年当時、これは深刻な制約だった。

### (c) RoPE — 回す

[[summaries/2021-roformer]]（Su ら 2021）は、この行き詰まりを**要求から解く**ことで抜けた。「内積が相対位置 $m-n$ にしか依存しない」を関数方程式として立て、2 次元で複素数を使って解くと、答えは一意に「**位置に比例した角度だけ回転させる**」になる。

$$f_{\{q,k\}}(\boldsymbol{x}_m, m) = \boldsymbol{R}^d_{\Theta,m}\boldsymbol{W}_{\{q,k\}}\boldsymbol{x}_m,\qquad \boldsymbol{q}_m^\top\boldsymbol{k}_n = \boldsymbol{x}_m^\top\boldsymbol{W}_q^\top\,\boldsymbol{R}^d_{\Theta,\,n-m}\,\boldsymbol{W}_k\boldsymbol{x}_n$$

$\boldsymbol{R}^d_{\Theta,m}$ は $2\times2$ の回転ブロックを対角に並べたブロック対角行列で、$d$ 次元を $d/2$ 個の平面に分け、平面 $i$ を角速度 $\theta_i = 10000^{-2(i-1)/d}$ で回す。角速度の列は正弦位置符号化と**同じもの**を流用しており、その意味で RoPE は「正弦位置符号化を足す代わりに掛ける」変形と読める。

<figure>

![](../../raw/assets/2021-roformer/x1.png)

<figcaption>図: RoPE の実装（[[summaries/2021-roformer]] 図1 より引用）。上段は d=2 の場合で、ベクトルを位置 m と角速度 θ₁ の積の角度だけ回転させる。下段は d 次元への一般化——query / key を 2 次元ずつのペアに区切り、ペアごとに異なる角速度、トークンごとに異なる位置を掛けた角度で回す。</figcaption>
</figure>

**加法から乗法へ**の転換が、上の 2 つの代償を同時に消す。

- 回転行列は**直交行列**なのでノルムを保つ。したがって $\phi(\boldsymbol{q}_m)$ と $\varphi(\boldsymbol{k}_n)$ にそれぞれ独立に掛けられ、**linear attention と組める**。
- $N \times N$ のバイアスが要らない。**追加パラメータもゼロ**（$\boldsymbol{R}$ は位置から決定的に計算される）。
- **トークンごとに閉じている**。$\boldsymbol{q}_m$ の回転は $m$ だけで決まり $n$ を知らなくてよい。だから **KV cache と素直に両立する**（key を回転済みでキャッシュすれば、後から来るどのクエリにもそのまま使える）。
- $\boldsymbol{R}$ は疎なので実体化せず、**要素ごとの積 2 回と加算 1 回**で計算する。実質メモリ帯域律速であり、attention カーネルへ融合できる。

**長距離減衰**も持つ。相対距離が伸びるほど内積の**上界**が小さくなる（ただし内積そのものが減衰する保証ではない。→ [[summaries/2021-roformer]] の限界節）。

### (d) 拡張 — base 周波数というつまみ

RoPE の重要な副産物は、**調整できるハイパーパラメータが実質 1 個（角速度の底 $10000$）しかない**ことである。ここを触るだけで有効な位置解像度が変わるので、**コンテキスト長の拡張が「再訓練」ではなく「短いファインチューニング」の問題に降格した**。学習済みの位置埋め込み表ではこれができない（表に無い位置は補間するしかなく、その補間に意味がある保証がない）。

- **Position Interpolation（位置内挿）**: 位置インデックスそのものを縮めて、訓練済みの範囲に押し込む。
- **NTK-aware scaling / YaRN**（Yet another RoPE extensioN）: 周波数帯ごとに扱いを変えて内挿する。高周波（近距離を担う）は残し、低周波（遠距離を担う）を伸ばす。**実務での標準**であり、本 wiki では [[summaries/2024-deepseek-v3]] が **4K → 32K → 128K の 2 段階拡張（各 1000 ステップ）** に使った記録があり、Kimi 系（[[summaries/2026-kimi-k2.5]]）でも採用されている。DeepSeek-V3 の 128K 対応は MLA の設計変更ではなく **YaRN から来ている**（→ [[summaries/2025-deepseek-series]] が V2/V3 の主張と実際を突き合わせて確認している）。
- **部分適用（partial / p-RoPE）**: 全次元でなく一部の次元だけを回す。Gemma 4（[[summaries/2026-gemma-4]]）は **$p=0.25$**、DeepSeek-V4（[[summaries/2026-deepseek-v4]]）も**部分 RoPE** を採る。回さない次元は位置に依存しない「内容だけ」のチャネルになり、極端な長距離での位置情報の干渉を減らす狙いがある。
- **NoPE（No Positional Encoding）**: decoder-only の因果マスク自体が位置情報を含む（各トークンが見られる先行トークンの個数が位置を表す）ため、位置符号化を一切入れなくても学習できるという主張がある。ただし純 NoPE を大規模で使う実例は少なく、実際には**別の演算子に位置の責任を移したうえでフル層だけ NoPE にする**形が現れている。その一次資料が Kimi Linear（[[summaries/2025-kimi-linear]]）で、次の (e) に独立させた。

### (e) KDA — 位置符号化を「学習可能な遷移行列」に畳む

RoPE の系譜 (c) を「$q$/$k$ を固定回転で回す乗法的位置符号化」と読むと、**線形アテンションのゲート付きデルタ則は同じ骨格の一般化**として読める、というのが Kimi Linear（[[summaries/2025-kimi-linear]]）の理論的貢献である。RoPE は $s_{t,i}=q_t^\top(\prod_{j=i+1}^t \boldsymbol{R}_j)k_i$ と**回転行列の累積積**で相対位置を入れる。KDA（Kimi Delta Attention, → [[transformer-architecture]]）の出力も $o_t=\sum_i q_t^\top\big(\prod_{j=i+1}^t \mathrm{Diag}(\alpha_j)(I-\beta_j k_jk_j^\top)\big)k_i\,v_i$ と書け、**RoPE の固定回転行列 $\boldsymbol{R}_j$ を「データ依存で学習可能な遷移行列」に置き換えたもの**とみなせる。含意は 3 つ:

- **直交性の縛りを外す**。RoPE の $\boldsymbol{R}_j$ は直交行列に限られるが、KDA の遷移行列にその制約はない。ぶん表現力が高く、RoPE の固定周波数ゆえの外挿問題（学習時の長さへの過適合、YaRN 等での再調整が要る原因）を原理的に回避しうる。
- **チャネルごとの位置解像度**。RoPE が次元対ごとに異なる回転周波数を割り当てて非一様フーリエ変換のように働くのと同様、KDA の**チャネルごと減衰 $\mathrm{Diag}(\alpha_t)$** が次元ごとの位置解像度の多様性を担う。ヘッド単位スカラー減衰の GDN にはこの多様性がなく、これが KDA を細粒度ゲートにした動機。
- **だからフル層は NoPE にできる**。Kimi Linear は位置と直近性バイアスの責任を全部 KDA に委ね、MLA（フル）層には位置符号化を一切入れない。副産物として (1) NoPE の MLA は推論時に純 MQA へ変換できて速い、(2) YaRN 等の RoPE 周波数再調整なしに長コンテキスト化できる。表5 では NoPE 版が長コンテキストで RoPE 版を上回り、RoPE 版は短コンテキストのみ同等だった。後継の K3（[[summaries/2026-kimi-k3]]）もこの NoPE 設計を継いでいる。

この「位置を専用の位置認識演算子に移し、フル層は NoPE」という分業は、上の (d) の部分 RoPE（一部次元だけ回す）と同じ問題意識——**位置情報を全次元に強制しない**——の、より踏み込んだ形と読める。

## システムとの相互作用

位置符号化は「モデルの表現力」の話に見えて、実際には**推論システムの他の層と最も強く絡む部品**である。RoPE が普及した理由の大半はここにある。

### KV cache — 回転済みで持てる

RoPE の回転はトークンごとに閉じているので、key を**回転してからキャッシュ**できる。attention スコアに $(m,n)$ 依存の項を足す方式（T5 の相対バイアス等）はこの単純さを持たず、クエリが来るたびに位置対に応じた項を作る必要がある。→ [[llm-serving-systems]]

### カーネル融合 — 演算がタダに近い

RoPE の計算はブロック対角回転だけで、実質メモリ帯域律速である。だから attention カーネルの前段に融合すると追加コストがほぼ消える。**逆に、そこが「ついでに何かを混ぜ込める場所」にもなる**——[[summaries/2024-flashattention-3]] は FP8 量子化の前処理（ブロック量子化・非干渉化処理）を「直前の rotary embedding（メモリ帯域律速）へ融合できる」として、追加コストなしで実現している（→ [[llm-inference-optimization]]）。

### 量子化 — RoPE 次元は精度を落とせない

KV cache を低精度化するとき、**RoPE を担う次元は BF16 のまま残す**設計が現れている。DeepSeek-V4 は「RoPE 次元は BF16・残りは FP8」で KV を混合精度格納する（→ [[summaries/2026-deepseek-v4]]、[[model-quantization]]）。位置情報は角度として符号化されているので、量子化誤差が角度の誤差になり、相対位置がずれる。**「何を低精度にしてはいけないか」の一覧に位置符号化が入る**という点は、[[low-precision-training]] が集めた「6 年間変わらないリスト」（埋め込み・出力ヘッド・正規化・attention 内部・オプティマイザ状態）と同じ性格の知見である。

### MLA — 乗法であることの代償

RoPE の乗法性には裏面がある。**attention の中に位置依存の行列が挟まるので、$q$ 側と $k$ 側の行列を事前に畳んでおく最適化が壊れる**。

DeepSeek の **MLA**（Multi-head Latent Attention）がその実例である（→ [[summaries/2025-deepseek-series]]、[[transformer-architecture]]）。MLA は key/value を低ランク潜在ベクトル $\boldsymbol{c}^{KV}$ に圧縮して、生成時はそれだけをキャッシュする。RoPE が無ければ key の復元行列 $\boldsymbol{W}^{UK}$ は query 側に吸収でき、key を実体化する必要すらない。ところが RoPE を挟むと間に $\boldsymbol{R}^d_{\Theta,n-m}$ が入り、**位置依存なので固定行列に畳めない**。DeepSeek の解は、**RoPE を担う専用の小さな key $\boldsymbol{k}^R_t$ を分離してキャッシュする**（decoupled RoPE key）ことだった。

「乗法だから相対位置がタダで手に入る」ことと「乗法だから行列を畳めない」ことは同じ性質の表裏であり、後者は 5 年後の KV cache 圧縮設計に具体的な形で現れている。

## 長コンテキストの実態 — 位置符号化は「読める」を保証しない

コンテキスト長を伸ばすことと、その長さを実際に使えることは別である。[[summaries/2023-lost-in-the-middle]] は、**入力の中盤に置かれた情報の利用率が両端より顕著に落ちる**（系列位置効果）ことを示した。位置符号化が任意の長さを表現できても、モデルがその位置を等しく参照できるとは限らない。

さらに [[summaries/2025-effective-context-engineering]] は **context rot**（コンテキストの劣化）の機構の 1 つとして**位置内挿の劣化**を挙げている——訓練時より長い範囲へ内挿で押し広げた位置は、訓練分布から外れる。「128K 対応」は「128K を等しく使える」ではない。

したがって運用側の規律は [[context-engineering]] に属する: 重要な情報は端に置く、無闇に詰めない、必要になった時点で取りに行く。**位置符号化はウィンドウの上限を決める部品であって、その中身の使われ方までは決めない**。

## 設計論点

- **どこに注入するか**が最初の分岐である。入力に足す（絶対）／attention スコアに足す（相対バイアス）／query・key を変換する（RoPE）。3 番目だけが $q$/$k$ に分離可能で、そのことが linear attention・KV cache・カーネル融合のすべてに効いた。
- **「勝った理由」は精度ではなかった**。[[summaries/2021-roformer]] の実験は弱い（GLUE 3 勝 3 敗、BLEU +0.2、外挿は未検証）。RoPE が標準になったのは、**追加パラメータ・追加メモリを要求せず、システムの他の層と喧嘩しない**という運用上の性質による。これは [[summaries/2022-flashattention]] の「FLOPs でなく IO」、[[summaries/2021-switch-transformers]] の「top-1 で足りる」と同型の教訓である。
- **調整つまみを 1 個残しておくことの価値**。base 周波数という単一のスカラーが、後年の長コンテキスト拡張エコシステム全体の土台になった。設計時に意図されたものではない。
- **直交変換という共通語彙**。RoPE は $q$ と $k$ に**異なる**直交行列を掛けて内積を $n-m$ にだけ依存させる。[[summaries/2024-flashattention-3]] の非干渉化処理は $Q$ と $K$ に**同じ**直交行列を掛けて内積を**不変**に保つ。[[model-quantization]] の外れ値対策と NVFP4 の Random Hadamard 変換（→ [[low-precision-training]]）も同じ道具である。「不変量を保ちながら座標系だけ変える」操作が、LLM 実装技術の共通語彙になっている。
- **エージェントへの含意**。長い trajectory（エージェントが辿った行動列）を抱えるエージェントにとって、コンテキスト拡張の実体は位置符号化の内挿である。「128K のモデルだから 128K のログを保持できる」という前提は、lost-in-the-middle と context rot の両方によって崩れる。→ [[context-engineering]] の compaction・ノート取り・サブエージェント。

## 本 wiki における根拠の所在

このページの記述の重みは一様ではない。読むときの注意として明示しておく。

- **一次資料がある**: RoPE 本体（[[summaries/2021-roformer]]）、系列位置効果（[[summaries/2023-lost-in-the-middle]]）、MLA と decoupled RoPE key（[[summaries/2025-deepseek-series]]）、YaRN の実務適用（[[summaries/2024-deepseek-v3]]、[[summaries/2026-kimi-k2.5]]）、p-RoPE と部分 RoPE（[[summaries/2026-gemma-4]]、[[summaries/2026-deepseek-v4]]）、rotary へのカーネル融合（[[summaries/2024-flashattention-3]]）、**NoPE ＋ 位置認識演算子の設計と「KDA＝学習可能な乗法的位置符号化」の読み**（[[summaries/2025-kimi-linear]]、K3 が継承）。
- **二次資料しかない**: 系譜 (b) の Shaw ら・Transformer-XL・T5・DeBERTa は、いずれも [[summaries/2021-roformer]] の関連研究節を通した記述である。**RoPE を提案する側による整理**なので、加法的手法の弱点の描き方には当然バイアスがありうる。
- **原典が無い**: **Position Interpolation・NTK-aware scaling・YaRN の各原論文**、**ALiBi の原論文**。これらは本 wiki では「他の原典が使っている手法」としてしか登場していない。長コンテキスト化は現在の主戦場なので、**YaRN 原典（Peng ら 2023）と ALiBi 原典（Press ら 2021）は次に取り込む候補**として記録しておく（NoPE は Kimi Linear で一次資料を得た）。

## 関連ページ

- [[transformer-architecture]] — 位置符号化が組み込まれる器。attention の系譜と MLA
- [[llm-inference-optimization]] — カーネル融合・KV cache の帯域律速
- [[llm-serving-systems]] — KV cache の配り方と prefix キャッシュ
- [[context-engineering]] — 伸ばしたウィンドウをどう使うか
- [[model-quantization]] ・ [[low-precision-training]] — 「何を低精度にしてはいけないか」
- [[summaries/2021-roformer]] — 本ページの主要な根拠原典
- [[summaries/2025-kimi-linear]] — NoPE ＋ KDA を「学習可能な位置符号化」とする節 (e) の一次資料
