---
type: summary
source_path: raw/papers/KIMI LINEAR- AN EXPRESSIVE, EFFICIENT ATTENTION ARCHITECTURE.pdf
source_kind: paper
title: "Kimi Linear: An Expressive, Efficient Attention Architecture"
authors: [Kimi Team]
year: 2025
venue: arXiv:2510.26692
ingested: 2026-08-11
tags: [transformer-architecture, positional-encoding, llm-inference-optimization, linear-attention, kimi-delta-attention, kimi-linear]
translation: "[[translations/2025-kimi-linear]]"
---

# Kimi Linear: 表現力が高く効率的なアテンション・アーキテクチャ

> 原典: [[translations/2025-kimi-linear]] ・ `raw/papers/KIMI LINEAR- AN EXPRESSIVE, EFFICIENT ATTENTION ARCHITECTURE.pdf`
> 著者・年・会議: Kimi Team（Moonshot AI ほか）・2025・arXiv:2510.26692v2 [cs.CL]（2025-11-01）

## 一言まとめ

**線形アテンション（linear attention, softmax を外して系列長に対し線形時間・固定サイズ状態で計算するアテンション）とフルアテンションを層単位で混ぜたハイブリッド構成**で、**公正な比較のもとフルアテンションを一貫して上回った初めての実証**を示したアーキテクチャ論文。中核は **KDA（Kimi Delta Attention）**——Gated DeltaNet の忘却ゲートを「チャネルごと（channel-wise）」に細粒度化した線形アテンション——で、これを MLA（Multi-Head Latent Attention）と **3:1** で交互配置する。48B 総パラメータ／3B アクティブの MoE で 1.4T トークン学習し、**KV キャッシュを最大 75% 削減、1M コンテキストでデコード・スループット最大 6 倍**を達成した。本 wiki にとってこれは、K3（[[summaries/2026-kimi-k3]]）と gpt2-to-kimi3（[[summaries/2026-gpt2-to-kimi3]]）が**又聞きで書いていた KDA の一次資料**であり、同時に [[positional-encoding]] が「原典が無い」と記録していた **NoPE の一次資料**でもある。

## 背景と問題意識

エージェントとテストタイム・スケーリング（推論時に長い軌跡・ツール利用・思考を積むほど精度が上がる考え方、→ [[test-time-compute]]）が主戦場になると、**推論のボトルネックはモデルの賢さより「長い系列を安く回せるか」に移る**。softmax アテンションの弱点はここで二重に効く: 計算量が系列長の 2 乗、かつ KV キャッシュ（生成済みトークンの key/value を貯める領域）が系列長に**線形に増え続ける**。1M トークンのデコードでは、この線形増加がメモリ帯域を食い潰してスループットを殺す。

線形アテンションはこの両方を消す——状態を**固定サイズの行列 1 枚**に畳むので、KV キャッシュが要らず計算も線形になる。だが歴史的に品質で softmax に負けてきた。近年その差を詰めたのが 2 つの道具で、本論文はこの系譜の最新点にある（詳細な系譜は [[transformer-architecture]] の「attention の系譜」節）:

- **ゲーティング／減衰（decay）**: 状態を毎ステップ縮めて古い連想を忘れる。RetNet（データ非依存スカラー）→ Mamba2（データ依存スカラー）→ GLA（チャネルごと）と細かくなってきた。
- **デルタ則（delta rule）**: 状態を「$k\mapsto v$ の連想記憶」と見て、書き込みのたびに**再構成誤差 $\tfrac12\lVert S^\top k-v\rVert^2$ を勾配降下で減らす**ランク 1 更新。単なる追記でなく**選択的に上書き**する。DeltaNet → Gated DeltaNet（GDN, 忘却ゲート付き）。

それでも純粋な線形アテンションは**正確なコピーと長距離の厳密検索に弱い**（固定容量なので必然的に情報を捨てる）。実務的な妥協が**ハイブリッド**——大半を高速な線形層にし、少数のフル層で取りこぼしを回収する。ただし従来のハイブリッドは規模が小さいか評価が薄く、「フルアテンションに本当に匹敵/凌駕するか」は未決着だった。Kimi Linear はここへ、1.4T トークンの管理下比較で答えを出しにいく。

## 提案手法

### KDA — ゲートを「チャネルごと」に細粒度化する

GDN の状態更新は $S_t=\alpha_t(I-\beta_t k_t k_t^\top)S_{t-1}+\beta_t k_t v_t^\top$ で、忘却ゲート $\alpha_t$ が**ヘッド単位のスカラー 1 個**（状態全体を一様に減衰）。KDA はこれを**対角行列 $\mathrm{Diag}(\alpha_t)$** に替える——各特徴次元が独立の忘却率を持つ:

$$S_t=(I-\beta_t k_t k_t^\top)\,\mathrm{Diag}(\alpha_t)\,S_{t-1}+\beta_t k_t v_t^\top$$

「どの記憶を、どのチャネルで、どれだけ忘れるか」を細かく制御できるので、有限状態 RNN メモリ（RNN 型の固定状態）を精密に使える。合成タスク（回文・MQAR・状態追跡、図4）で KDA は GDN より速く収束し、GLA 系のチャネル減衰とデルタ則の**両取り**であることが効いている。減衰のない Mamba2 は同設定で全タスク失敗した。

### DPLR の制約付き変種 — 「a=b=k に縛る」ことが効率の肝

KDA の遷移行列は一般化すると **DPLR（Diagonal-Plus-Low-Rank, 対角＋低ランク）** $D-a_t b_t^\top$ の形。一般 DPLR は表現力が高い代わりに並列化しにくく重い。KDA は $D=\mathrm{Diag}(\alpha_t),\ a_t=\beta_t k_t,\ b_t=k_t\odot\alpha_t$ と**低ランク項を $k$ に束縛（$a=b=k$）**することで、専用のチャンクワイズ（chunk-wise, 系列をチャンクに切ってチャンク内は並列・チャンク間は再帰する）アルゴリズムを可能にする。この束縛で、数値不安定を避けるための「二次チャンキング」を 4 回→2 回に減らし、行列積を 3 つ削れる。結果として**カーネル速度は DPLR のほぼ 2 倍**（図2、最大 64k）。表現力は一般 DPLR とほぼ同等のまま、コストだけ落とす設計である。

### Kimi Linear アーキテクチャ — KDA:MLA = 3:1 の層単位ハイブリッド

<figure>

![](../../raw/assets/2025-kimi-linear/fig3.png)

<figcaption>図3（再掲）: Kimi Linear のブロック構成。トークン混合層（KDA を N 回、MLA を 1 回）＋ MoE チャネル混合層を積む。実装では N=3、すなわち KDA 3 層ごとに MLA 1 層。KDA ブロック内は Linear→ShortConv→（q,k は L2Norm）と σ ゲート群で構成される。</figcaption>
</figure>

- **層単位（layerwise）で交互配置**。ヘッド単位で混ぜるのでなく層ごと丸ごと切り替える——インフラが単純で学習も安定する。アブレーション（表1）で **3:1 が最良**: 7:1 は学習損失は同等だが検証（分布シフト下）が悪化、1:1 は品質同等だが推論が重い、0:1（純フル）は性能が低い。3:1 のとき**長系列で KV キャッシュを 75% 削れる**（4 層に 1 層しかフル層が KV を貯めない）。
- **フル層は MLA**（DeepSeek 由来の潜在圧縮つきフルアテンション、→ [[transformer-architecture]] の MLA 節）。固定状態の KDA が取りこぼす厳密検索を、周期的な完全 attention で回収する分業。
- **土台は Moonlight [62]**（DeepSeek-V3 系の MoE）。256 エキスパート中 8＋共有 1 を起動、48B 総／3B アクティブ。第 1 層だけ dense。

### NoPE — 位置符号化を KDA に丸投げする

これが [[positional-encoding]] にとっての核心的な新規性。**KDA を「学習可能な乗法的位置符号化」と読み替える**のが本論文の理論的貢献である。RoPE（Rotary Position Embedding）は $q_t^\top(\prod_{j=i+1}^t R_j)k_i$ のように**回転行列の累積積**で相対位置を入れる。ゲート付きデルタ則も同じ骨格 $o_t=\sum_i q_t^\top(\prod_{j=i+1}^t \mathrm{Diag}(\alpha_j)(I-\beta_j k_j k_j^\top))k_i\,v_i$ で書けて、**RoPE の固定回転行列を「データ依存で学習可能な遷移行列」に置き換えた一般化**とみなせる。RoPE が課す直交性の縛りを外すぶん、原理的により強く、RoPE の固定周波数ゆえの外挿問題（学習時の長さへの過適合）も回避しうる。しかも RoPE がチャネルごとに違う回転周波数を割り当てて非一様フーリエ変換のように働くのと同様、KDA の**チャネルごとゲート**が次元ごとの位置解像度の多様性を担う。

だから Kimi Linear は **MLA（フル）層に位置符号化を一切入れない（NoPE）**——位置と直近性バイアスの責任を全部 KDA に委ねる。実務的な副産物として、(1) NoPE の MLA は推論時に純粋な MQA（Multi-Query Attention）へ変換できて速い、(2) YaRN のような RoPE 周波数の再調整なしに長コンテキスト学習できる。表5 では NoPE 版が長コンテキストで RoPE 版を明確に上回る（RoPE 版は短コンテキストのみ同等）。

## 実験結果と知見

- **短コンテキスト（表3 事前学習・表4 SFT）**: Kimi Linear が MLA・GDN-H をほぼ全項目で上回る。MMLU-Pro 51.0（MLA 47.2）、GPQA-Diamond 62.1（MLA 57.1）など。序列は概ね **Kimi Linear > GDN-H > MLA**。
- **長コンテキスト（表5, 128k）**: 平均 54.5 で首位（MLA 52.2, GDN-H 51.2）。RULER 84.3・RepoQA 68.5 は大差。ここでは**GDN-H が MLA を下回る**——単純なスカラー減衰のハイブリッドでは長距離が持たず、KDA の細粒度減衰が効いていることの傍証。
- **RL スケーリング（図6）**: 数学 RLVR（検証可能報酬つき RL）で、Kimi Linear の学習・テスト精度が MLA より**速く・高く**伸び、差が広がる。エージェント／推論で重要な「長い生成の RL」で線形ハイブリッドが不利にならないどころか有利、という主張。
- **スケーリング則（図5）**: 計算最適点で **MLA 比 1.16× の計算効率**（同じ損失に 16% 少ない計算で到達）。
- **効率（図1・図7）**: プレフィルは 512k で 2.3×・1M で 2.9× 高速、デコード TPOT（Time Per Output Token）は 1M で **6.3×**（1.84ms vs 11.48ms、大バッチ効果込み）。バッチ1でも 1M デコード 2.2×。固定サイズ状態（ヘッドあたり 128×128）で系列長に依らず、KV キャッシュ削減ぶんを大きいバッチに回せるのが 6× の源。
- **付録 D（5.7T 版）**: 公開チェックポイントは 5.7T トークン・1M 対応。同じ 3B アクティブの Moonlight（16B 総）をほぼ全項目で上回り、RULER@1M で 94.8。

## 限界・批判的視点

- **「フルアテンション超え」は自己申告の管理下比較**である。同一レシピ・同一パラメータでの内製比較としては誠実だが、外部の強いフルアテンション実装との比較ではない。[[transformer-architecture]] と [[agent-evaluation]] が記録する **provider-reported ベンチマークの読み方**の注意がそのまま当てはまる。
- **1.4T の主結果はベース／SFT どまり**で、フロンティア級の事後学習（大規模 RL・エージェント能力）まで通した比較ではない。5.7T 版も対 Moonlight であって対フロンティアではない。**「アーキ単体の器の良さ」は示せても「最終的なモデルの強さ」は別レイヤ**（→ [[transformer-architecture]] の「アーキと能力は別」）。
- **長コンテキストの「速い」は「読める」を保証しない**。RULER 等は改善したが、[[positional-encoding]] が整理する「長コンテキストの実態」（位置符号化を伸ばしても中盤が読めるとは限らない）という論点は残る。固定状態が「どの連想を減衰するか」は学習任せなので、**長期タスクでの記憶の一貫性**は新しい未解決評価軸になる。
- **再現性はカーネルに律速される**。線形アテンションの速度優位は最適化されたカーネル（KDA kernel＋vLLM 統合を公開）に強く依存し、「数学が良いから速い」と「良いカーネルがあるから速い」は分けて読む必要がある（→ [[transformer-architecture]] の FlashAttention の教訓）。
- **チャネルごと減衰の数値精度**は本質的な弱点で、$1/\Gamma$（累積減衰の逆数）がオーバーフローしうる。Kimi Linear は $a=b=k$ 束縛と二次チャンキングで対処するが、K3（[[summaries/2026-kimi-k3]]）はさらに**下限つき減衰**でこれを BF16 の密行列積に載せ替えている——**Kimi Linear の設計の弱点を K3 が数値面で継いで直した**関係にある。

## 実装・運用上の示唆

- **ハイブリッド比率は「品質×推論コスト」の設計変数**。3:1 は KV を 75% 削りつつ品質を保つ点。用途が長コンテキスト・デコード重視なら線形層の比率を上げ、厳密検索が要るなら下げる、という連続体の一点として読む。
- **NoPE ＋ 位置認識演算子**の組み合わせは、長コンテキスト化を「RoPE 周波数いじり（YaRN 等）」から解放する実務的な逃げ道。位置を担う専用機構（KDA・短畳み込み・SWA 等）があれば、フル層は NoPE にできる。
- **drop-in 互換を最初から狙う**——キャッシュ／スケジューリングの IF を変えず既存のフルアテンション・パイプラインに差し込める形で公開したのは、ハイブリッド普及の現実解。
- **「圧縮こそ知性」の設計思想**: スパースアテンション（KV 全体を持ってトークンを選ぶ）と違い、線形は固定状態に畳んで捨てる。両者は排他でなく、将来は線形の圧縮＋スパースの細粒度検索の合流もありうる、と著者は展望する。

## 用語と略称

- **KDA** = Kimi Delta Attention（Gated DeltaNet をチャネルごと減衰に拡張した線形アテンション）
- **GDN** = Gated DeltaNet / **GDN-H** = そのハイブリッド版（本論文のベースライン）
- **DPLR** = Diagonal-Plus-Low-Rank（対角＋低ランクの遷移行列）
- **MLA** = Multi-Head Latent Attention（潜在圧縮つきフルアテンション、→ [[transformer-architecture]]）
- **MQA** = Multi-Query Attention（key/value を全ヘッドで共有する軽量アテンション）
- **NoPE** = No Position Encoding（明示的位置符号化を入れない、→ [[positional-encoding]]）
- **RoPE** = Rotary Position Embedding（回転位置埋め込み、→ [[positional-encoding]]）
- **KV キャッシュ** = Key-Value cache（生成済みトークンの中間状態、→ [[llm-inference-optimization]]）
- **TPOT** = Time Per Output Token（出力トークンあたり時間、デコード速度の指標）
- **RLVR** = Reinforcement Learning with Verifiable Rewards（検証可能報酬による RL、→ [[reinforcement-learning-from-human-feedback]]）
- **MoE** = Mixture of Experts（→ [[mixture-of-experts]]）
- **TTT** = Test-Time Training（表7 の状態更新則を統一的に見る枠組み）

## 関連ページ

- [[transformer-architecture]] — attention の系譜（固定容量メモリと追い出しポリシー）。本要約は KDA/Kimi Linear の一次資料。
- [[positional-encoding]] — KDA を「学習可能な乗法的位置符号化」と読む節・NoPE の一次資料。
- [[llm-inference-optimization]] — 固定状態による KV 削減・6× デコード・チャンクカーネルの一次資料。
- [[test-time-compute]] — RL テストタイム・スケーリングがこのアーキの動機。
- [[mixture-of-experts]] — 土台の Moonlight/DeepSeek 系 MoE。
- [[summaries/2026-kimi-k3]] — KDA を継いで下限つき減衰・全ランクゲート・NoPE で精緻化した後継。
- [[summaries/2026-gpt2-to-kimi3]] — KDA/MLA/ハイブリッドを系譜として概説した二次資料。
- [[summaries/2025-deepseek-series]] — MLA と decoupled RoPE key の一次資料。
