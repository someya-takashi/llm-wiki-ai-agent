---
type: summary
source_path: "raw/papers/DeepSeek-V4_ Towards Highly Efficient Million-Token Context Intelligence.md"
source_kind: paper
title: "DeepSeek-V4: Towards Highly Efficient Million-Token Context Intelligence"
authors: [DeepSeek-AI]
year: 2026
venue: "arXiv 2606.19348"
ingested: 2026-07-29
tags: [transformer-architecture, llm-inference-optimization, reinforcement-learning-from-human-feedback, tool-use-and-function-calling, retrieval-augmented-generation, test-time-compute, context-engineering, agent-evaluation, million-token-context, csa-hca, opd, deepseek]
translation: "[[translations/2026-deepseek-v4]]"
---

# DeepSeek-V4: Towards Highly Efficient Million-Token Context Intelligence（DeepSeek-AI, 2026）

> 原典: [[translations/2026-deepseek-v4]] ・ `raw/papers/DeepSeek-V4_ Towards Highly Efficient Million-Token Context Intelligence.md`
> 著者・年・出典: DeepSeek-AI・2026・arXiv:2606.19348（プレビュー版）

## 一言まとめ

**100 万トークンコンテキストを「特別な機能」でなく「日常運用できるデフォルト」にする**ことを狙ったオープン MoE 系列（Pro 1.6T/活性 49B・Flash 284B/活性 13B）。KV cache の圧縮（CSA/HCA）とスパース選択を重ねて 1M 時の推論 FLOPs を V3.2 比 **27%**・KV cache を **10%**（一般的な BF16 GQA8 比では約 **2%**）に削り、事後学習では mixed RL を **On-Policy Distillation（OPD）** に全面置換した。Pro-Max はオープンモデルの SOTA を再定義し（SimpleQA-Verified で従来オープン +20pt、Codeforces で人間 23 位相当）、[[summaries/2025-deepseek-r1]]（推論の創発）→ V3.2（スパース attention と Discard-all）に続く DeepSeek 系譜の現在地を示す原典である。

<figure>

![](../../raw/assets/2026-deepseek-v4/dsv4_performance.png)

<figcaption>図1（再掲）: 左は Pro-Max と対抗モデルのベンチマーク性能、右は V4 系列と V3.2 の推論 FLOPs・KV cache サイズ（コンテキスト長別）。</figcaption>
</figure>

## 背景と問題意識

test-time scaling（推論時に計算を積んで精度を買う → [[test-time-compute]]）と長ホライズンのエージェントタスクは、どちらも**長いコンテキストを安く回せること**を前提にする。しかし素の attention は系列長の二乗の計算と O(N) の KV cache（過去トークンの key/value 保存領域）を要求し、100 万トークン級ではこの物理コストが「できるが高すぎる」水準に達する。[[summaries/2026-gpt2-to-kimi3]] が整理した linear attention 化（固定状態）、[[summaries/2026-gemma-4]] の間引き・共有はこの問題への既存の 2 路線だが、V4 は**第三の路線——「圧縮してから、選ぶ」**——を 1.6T 級で実証した。同時に、コンテキストが 1M に伸びると事後学習側にも新しい問題（100 万トークン rollout の RL、複数専門家の統合）が生じ、本レポートはそのインフラまで含めて開示している。

## 提案手法 / 主張

### 柱 1: ハイブリッド attention CSA＋HCA——圧縮とスパース化の二段構え

<figure>

![](../../raw/assets/2026-deepseek-v4/basic_arch.svg)

<figcaption>図2（再掲）: V4 の全体アーキテクチャ。attention はハイブリッド CSA/HCA、FFN は DeepSeekMoE、残差は mHC で強化。</figcaption>
</figure>

- **CSA（Compressed Sparse Attention）**: KV を **m=4 トークンごとに 1 エントリへ学習圧縮**（重み付き和。隣接ブロックとオーバーラップ）した上で、**Lightning Indexer** が各クエリに top-k（Flash 512 / Pro 1024）個の圧縮エントリだけを選ぶ DeepSeek Sparse Attention（DSA, V3.2 由来）を適用する。圧縮エントリは key と value を兼ね（shared-KV MQA）、クエリは低ランク射影で生成される。
- **HCA（Heavily Compressed Attention）**: **m′=128 の激圧縮**をかけ、スパース選択なしの密 attention。CSA と層で交互配置する。
- **補助装置**: 直近 128 トークンの sliding window ブランチ（圧縮では拾えない局所依存を補完）、attention sink（合計 attention を 1 未満に調整できる学習ロジット）、部分 RoPE（末尾 64 次元。KV が value を兼ねるため**出力側にも位置 −i の RoPE を当てて相対位置化**する工夫）、QK と KV エントリへの RMSNorm（ロジット爆発防止——これにより K2 系の QK-Clip が不要になる）。
- **効率の内訳**: 圧縮・スパースに加え、KV を RoPE 次元 BF16＋残り FP8 の混合精度で格納（ほぼ半減）、indexer の attention は FP4。合計で冒頭の 27%/10%/2% に至る。

<figure>

![](../../raw/assets/2026-deepseek-v4/CSA.svg)

<figcaption>図3（再掲）: CSA の中核構造。KV エントリを 1/m に圧縮し、DeepSeek Sparse Attention で top-k 選択。sliding window の KV が局所依存を補う。</figcaption>
</figure>

### 柱 2: mHC——残差接続を「安定に」太くする

Hyper-Connections（HC）は残差ストリームを n_hc=4 倍幅に拡張して表現力を上げるが、深く積むと数値不安定になる。mHC は残差写像行列 B を**二重確率行列の多様体（Birkhoff 多面体）へ Sinkhorn-Knopp 反復（20 回）で射影**し、スペクトルノルム ≤1 の非拡大写像に制約する——信号が層を経ても増幅されない保証を持たせたまま、残差の混合を学習可能にする。[[transformer-architecture]] の AttnRes（深さ方向の選択的読み出し）と並ぶ「残差ストリーム自体を設計する」系譜。

### 柱 3: Muon と訓練安定化

オプティマイザは Muon（勾配行列を Newton-Schulz 反復で直交化——8 回の高速係数＋2 回の安定係数のハイブリッド）。K2 の MuonClip と違い **QK-Clip を使わない**（attention 側の RMSNorm で爆発を防げるため）。1.6T 級で遭遇した loss spike には 2 つの実用技で対処: **Anticipatory Routing**（MoE のルーティングだけ過去のパラメータ θ_{t−Δt} で先読み計算し、ルーティングと本体の同期更新が作る悪循環を断つ。spike 検出時のみ動的に起動）と **SwiGLU Clamping**（線形成分を ±10 に制限）。「**原理は未解明だが有効なので公開する**」と明記する開示姿勢も特筆に値する。

### 柱 4: 事後学習——mixed RL を OPD に置換

事後学習の最大の転換は、V3.2 の mixed RL ステージを **On-Policy Distillation（OPD）** で置き換えたこと。手順は (1) ドメイン別スペシャリスト（10 体超）を SFT＋GRPO の RL で個別に鍛える → (2) **学生モデル自身が生成した trajectory の上で、教師たちの出力分布への逆 KL を最小化**して単一モデルへ統合する。重みマージや mixed RL が起こす性能劣化を、ロジットレベルの整合で回避する——[[summaries/2025-deepseek-r1]] の「発見は RL、転写は蒸留が安くて強い」という知見を、単一系列の訓練パイプライン内の分業（**発見はスペシャリスト RL、統合は OPD**）へ昇華した形である。トークンレベル近似ではなく**全語彙ロジット蒸留**を採り（勾配分散を抑える）、それを可能にするインフラ（教師の隠れ状態のみキャッシュしロジットはオンザフライ再構築、教師ヘッドはミニバッチあたり 1 個だけ常駐）まで書かれている。

GRM（Generative Reward Model, 生成型報酬モデル）は **actor 自身が兼ねる**——判定能力と生成能力を同一パラメータで同時最適化し、少量の人手アノテーションで頑健なルーブリック評価を得る（K2 の自己批評ルーブリック → K2.5 の複数ルーブリック GRM と収斂する潮流）。

### エージェント関連の設計

- **reasoning effort 3 段**: Non-think（8K）/ Think High（128K）/ Think Max（384K＋専用システムプロンプト注入）。モードごとに長さペナルティとコンテキスト窓を変えた RL で別々に鍛え、`<think>` タグ書式で統合。[[summaries/2026-gemma-4]] の 2 値トグルの多段版にあたる。
- **DSML ツールコールスキーマ**: `<|DSML|invoke name="...">` / `<|DSML|parameter name="..." string="true|false">` という **XML 型書式**。「XML はエスケープ失敗を効果的に緩和しツール呼び出しエラーを減らす」と JSON 対比の実験的根拠を明言（→ [[tool-use-and-function-calling]]。Anthropic の関数呼び出し書式と同型なのも興味深い）。
- **Interleaved Thinking**: ツール呼び出しシナリオでは**推論トレースをユーザーターン境界を越えて全ラウンド保持**へ転換（V3.2 は新ユーザーメッセージで全破棄していた）。1M 窓が「捨てるコンテキスト管理」を「持ち続ける管理」に変えた実例 → [[context-engineering]]。
- **Quick Instruction**: 検索要否判定・クエリ生成・タイトル生成などの補助タスクを専用特殊トークン（`<|action|>` `<|query|>` 等）で**本体の KV cache を再利用して**実行。別立て小型モデルと冗長 prefill を廃し TTFT を削減。
- **DSec サンドボックス**: Function Call／Container／microVM／fullVM の 4 実行基盤を単一 SDK で抽象化し、**数十万並行**のサンドボックスを管理。trajectory ログによる決定論的リプレイとプリエンプション安全な再開。
- **WAL rollout**: トークン粒度の Write-Ahead Log で生成を永続化。**「中断されたリクエストのゼロからの再生成は、短い応答ほど中断を生き延びやすいため長さバイアスを生み、数学的に不正」**という指摘は、RL インフラの正しさが統計的性質にまで効くことを示す教材。
- **batch-invariant・決定論的カーネル**: トークンの出力がバッチ内位置に依らずビット同一（デュアルカーネル attention・DeepGEMM・決定論的縮約）。訓練・推論の完全一致を「補正」（K2.5 の token-level clipping）でなく**根絶**で達成する路線。

## 実験結果と知見

- **ベース比較**（表1）: Flash-Base は**より少ない総・活性パラメータで V3.2-Base を大半のベンチで超える**。Pro-Base はほぼ全カテゴリで両者に対し優位。
- **Pro-Max vs フロンティア**（表6）: LiveCodeBench 93.5（**全モデル中最高**）・Codeforces 内製 Elo 3206（GPT-5.4 の 3168 超え、**人間 23 位相当・オープン初のクローズド級**）・IMOAnswerBench 89.8・Apex Shortlist 90.2。知識は SimpleQA-Verified 57.9 でオープン最高（従来 +20pt）だが Gemini-3.1-Pro（75.6）には届かない。agentic は SWE Verified 80.6・BrowseComp 83.4 で K2.6/GLM-5.1 と同水準、クローズド最高値には僅差で劣後。
- **形式数学**: Lean エージェント設定（最大 500 ツール呼び出し）で Seed-Prover 超えの SOTA、計算集約パイプラインでは **Putnam-2025 を証明完全 120/120**。
- **1M コンテキスト**: MRCR 1M で 83.5——Gemini-3.1-Pro（76.3）超え、Claude Opus 4.6（92.9）には劣後。128K までは劣化がほぼなく、1M でも強い残存性能（図9）。
- **reasoning effort の効き**（表7）: Non-think→High→Max で単調改善（HLE 7.7→34.5→37.7、BrowseComp −→80.4→83.4）。HLE では V3.2 比で**トークン効率も改善**（図10）。

<figure>

![](../../raw/assets/2026-deepseek-v4/dsv4_effort.svg)

<figcaption>図10（再掲）: 推論努力別の HLE / Terminal Bench 2.0 の性能とコスト。V4 系列は V3.2（Speciale 含む）より少ないトークンで高い精度に到達する。</figcaption>
</figure>
- **agentic search vs RAG**（表9/10）: 同一モデルの実測比較で agentic search が全難度帯で優位。コストは**ツール呼び出し 16.2 回（大半並列）・prefill 13.6k vs 10.5k トークン**と僅増に留まる——「反復検索は高い」という直感への反証データ。
- **実世界評価**: 中国語ライティングで Gemini-3.1-Pro に勝率 62.7%、ただし高難度・多ターンでは Opus 4.5 が優位（52.0% vs 45.9%）。内製 R&D コーディングベンチ（実リポジトリ 30 タスク）で 67%——Sonnet 4.5（47%）超え・Opus 4.5（70%）に肉薄・Opus 4.6 Thinking（80%）には差。社内開発者サーベイ（N=85）では 52% が「主力コーディングモデルにできる」と回答。

## 限界・批判的視点

- **プレビュー版＋自認する複雑さ**: 「リスク最小化のため検証済みコンポーネントを多く残した結果、アーキテクチャは比較的複雑」と自認し、将来の蒸留・簡素化を予告している。CSA/HCA/SWA/sink/部分 RoPE/mHC の同時採用は、どの部品がどれだけ効いたかのアブレーションを欠く。
- **安定化技術の原理未解明**: Anticipatory Routing と SwiGLU Clamping は「効くが理由は分からない」と明記——正直だが、再現側は条件依存性を見積もれない。
- **内製評価の再現不可**: Codeforces Elo（手続きは開示）・R&D コーディング・White-Collar・Search Q&A は内製で外部検証できず、比較対象の一部（K2.6/GLM-5.1/GPT-5.4）は API 不応答で欠測——ベースライン非対称は自己申告されている。実世界評価は中国語タスク中心で一般化に注意。
- **agentic はなおクローズドに劣後**: SWE 系・Terminal Bench では Opus 4.6/GPT-5.4 に届かず、著者も「オープンモデルはいずれもクローズドに後れを取る」と認める。
- **開発者サーベイの解釈**: N=85 は全員 DeepSeek 社内の利用者であり、選好バイアスを含む参考値。

## 実装・運用上の示唆

- **長コンテキストの費用対効果を再計算する**: 「1M は高い」という前提はモデル依存になった。KV 10%・FLOPs 27% 級の系列が普及すれば、[[context-engineering]] の切り詰め戦略の損益分岐（捨てて再構築 vs 持ち続ける）は変わる——Interleaved Thinking はその最初の運用例。
- **shared-prefix の on-disk KV cache**: 圧縮 KV はディスク格納・再利用に向く（同一プレフィックスの再 prefill 排除）。SWA 分の 3 戦略（全保存／周期チェックポイント／ゼロ保存＋再計算）はストレージと計算のトレードオフ設計の教科書例。
- **ツール書式は XML を検討**: エスケープ失敗の削減という実測に基づく主張。JSON スキーマで tool-call エラーに悩む場合の代替案として、DSML の設計（string 属性で型を明示）はそのまま参考になる。
- **RL/rollout インフラの統計的正しさ**: 中断リクエストの扱い（WAL 継続 vs 再生成）のような一見エンジニアリング上の選択が、学習される方策の長さ分布を歪める。分散 RL を組む際のチェックリスト項目。
- **補助タスクの Quick Instruction 化**: 「ルータ・分類器は別の小型モデル」という定石に対し、本体の KV cache 再利用＋専用トークンで置き換える設計は、チャット製品のレイテンシ・運用コスト削減に直結する。

## 用語と略称

- **LLM** = Large Language Model ／ **MoE** = Mixture-of-Experts（→ [[transformer-architecture]]）／ **DeepSeekMoE**（細粒度＋共有エキスパートの MoE 設計。Pro は共有 1＋384 中 6 活性）
- **KV cache**（過去トークンの key/value 保存領域 → [[llm-inference-optimization]]）／ **GQA / MQA** = Grouped-/Multi-Query Attention（KV ヘッドを共有して cache を削る方式）
- **CSA / HCA** = Compressed Sparse Attention / Heavily Compressed Attention（m=4 圧縮＋top-k スパース選択／m′=128 の激圧縮＋密 attention）
- **DSA** = DeepSeek Sparse Attention（V3.2 由来の top-k スパース attention）／ **Lightning Indexer**（スパース選択のスコアを計算する軽量モジュール）
- **SWA** = Sliding Window Attention（近傍窓のみの attention。V4 では局所依存の補助ブランチ）／ **attention sink**（合計 attention を 1 未満に調整可能にする学習ロジット）
- **RoPE**（回転位置埋め込み）／ **MTP** = Multi-Token Prediction（→ [[summaries/2026-gemma-4]] の drafter と同系の多トークン予測）
- **HC / mHC** = (Manifold-Constrained) Hyper-Connections（残差ストリームの多重化とその多様体制約版）／ **Birkhoff 多面体**（二重確率行列全体の集合）／ **Sinkhorn-Knopp**（行・列正規化の反復で二重確率行列へ射影するアルゴリズム）
- **Muon / Newton-Schulz**（勾配行列を直交化するオプティマイザとその反復法）／ **QK-Clip**（K2 系の attention ロジット爆発対策。V4 は不採用）
- **Anticipatory Routing / SwiGLU Clamping**（loss spike 対策の 2 技法）
- **EP / CP / ZeRO** = Expert / Context Parallelism, Zero Redundancy Optimizer（分散訓練の並列化・シャーディング）／ **MegaMoE**（EP 通信・計算を融合したオープンソースのメガカーネル）
- **TileLang / Z3**（カーネル開発 DSL とその整数解析に使う SMT ソルバ）／ **batch invariance**（出力がバッチ内位置に依存しないこと）
- **OPD** = On-Policy Distillation（学生の自己生成軌跡上で教師分布への逆 KL を最小化する蒸留 → [[reinforcement-learning-from-human-feedback]]）／ **GRPO** = Group Relative Policy Optimization ／ **GRM** = Generative Reward Model
- **DSML**（V4 のツールコール用特殊トークン・XML 型書式）／ **Quick Instruction**（KV cache 再利用の補助タスク用特殊トークン群）
- **QAT / MXFP4 / STE** = Quantization-Aware Training / FP4 のブロック量子化形式 / Straight-Through Estimator（量子化を通す勾配近似）
- **WAL** = Write-Ahead Log（先行書き込みログ）／ **DSec** = DeepSeek Elastic Compute（サンドボックス基盤）／ **3FS**（分散ファイルシステム）
- **MRCR / CorpusQA**（1M 級の長コンテキスト検索・QA ベンチマーク）／ **Apex / IMOAnswerBench / PutnamBench**（高難度数学ベンチ）／ **MCPAtlas / Toolathlon**（多様なツール・MCP サービスの agentic ベンチ）／ **GDPval-AA**（実務タスクの Elo 評価）
- **TTFT** = Time To First Token ／ **FIM** = Fill-in-Middle ／ **RAG** = Retrieval-Augmented Generation（→ [[retrieval-augmented-generation]]）

## 関連ページ

- [[transformer-architecture]] — CSA/HCA（第三の attention 路線）・mHC・Muon・訓練安定化
- [[llm-inference-optimization]] — 1M コンテキストの KV 経済・FP4 QAT・on-disk KV cache・batch invariance・MegaMoE
- [[reinforcement-learning-from-human-feedback]] — OPD への転換・actor 兼 GRM・WAL の長さバイアス
- [[tool-use-and-function-calling]] — DSML（XML 型ツールコール書式）
- [[retrieval-augmented-generation]] — agentic search vs RAG の同一モデル実測
- [[test-time-compute]] — reasoning effort 3 段・1M による scaling の天井引き上げ
- [[context-engineering]] — Interleaved Thinking（思考トレースの全保持への転換）
- [[agent-evaluation]] — 内製 Codeforces Elo の手続き・開発者サーベイという評価形式
- [[agent-safety-and-guardrails]] — DSec（4 実行基盤の隔離＋trajectory ログ）は本番規模 sandboxing の実例
- [[summaries/2025-deepseek-r1]] — 系譜の前身（RLVR・GRPO・蒸留の知見）
- [[summaries/2025-kimi-k2]] / [[summaries/2026-kimi-k2.5]] — 同時代のオープン競合（MuonClip との対比・GRM の収斂）
- [[summaries/2026-gemma-4]] — KV 削減の別解（間引き・共有）と thinking トグル
- [[summaries/2026-gpt2-to-kimi3]] — attention 系譜の中での位置（linear 化との対比）
