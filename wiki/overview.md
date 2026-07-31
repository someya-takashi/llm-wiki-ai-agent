---
type: overview
tags: [llm-agents, ai-agent, overview]
updated: 2026-07-31
---

# Overview — AI Agent

この wiki が扱う領域全体の総括ページ。**原典を取り込むたびに更新する**（ingest skill の手順 7）。
原典はまだ少なく、以下の大部分は「これから埋めていく骨組み」である。原典に裏付けられた箇所には `[[summaries/...]]` への参照を付してある。

## この領域の対象

**AI Agent（AI エージェント）** とは、LLM（Large Language Model, 大規模言語モデル）を推論エンジンとして、目標を与えられると自ら計画を立て、ツールを呼び出し、環境から返ってきた結果を観測しながら多段階のタスクを遂行するシステムを指す。1 回のプロンプトで答えを返す従来の LLM 利用と違い、**観測→思考→行動を繰り返すループ**を回す点、そして**外部世界に副作用を及ぼせる**点が本質的な差になる。

この wiki は、エージェント本体だけでなく、その土台となる LLM と開発基盤までを射程に入れる。

## 骨組み（今後埋めていく軸）

### 1. エージェントの基本構造

- agent loop（エージェントループ, 観測→思考→行動の反復）、ReAct 型の推論と行動の交互実行 → [[agent-loop]], [[reasoning-and-planning]]
  - この骨格の原型は ReAct（[[summaries/2022-react]], 2022）が確立した。思考を「環境に作用しない行動」として行動空間に加えるという定式化により、外部接地で幻覚を抑えつつ（CoT の失敗要因 56% → 0%）、few-shot プロンプトのみで模倣・強化学習を上回れることを示した。
- tool use / function calling（モデルが JSON スキーマに沿った引数で外部ツールを呼ぶ仕組み）→ [[tool-use-and-function-calling]]
  - 初期形は ReAct の 3 アクション Wikipedia API のようなプロンプト規約によるツール定義で、その後 API レベルの構造化された function calling へ発展した。
- planning（計画立案）と self-reflection（自己反省による軌道修正）→ [[reasoning-and-planning]], [[self-reflection]]
  - 推論の系譜の起点は CoT（[[summaries/2022-chain-of-thought]], 2022）。few-shot 例示に思考連鎖を入れるだけで推論が創発する（約 100B 規模で急伸）ことを示し、「答える前に考えさせる」設計と test-time compute の発想の源流となった。
  - Reflexion（[[summaries/2023-reflexion]], 2023）は、失敗の反省文をエピソード記憶に蓄えて次試行に注入する「言語的強化学習」で、重み更新なしの試行間学習を実現した——CoT（考えてから答える）→ ReAct（考えて動く）→ Reflexion（失敗から学ぶ）で単一エージェントの基本系譜が完結する。
- memory（短期＝コンテキスト内、長期＝外部ストア）→ [[agent-memory]]
  - 原型は MemGPT（[[summaries/2023-memgpt]], 2023）: コンテキストを OS の物理メモリに見立て、**LLM 自身が function call で記憶をページング**する virtual context management を定式化した（working context・archival memory・recursive summary の語彙の出発点）。Deep Memory Retrieval で GPT-4 単体 32.1% → 92.5%。
  - A-Mem（[[summaries/2025-a-mem]], 2025）は自己管理の対象を配置から**組織化・進化**へ拡張: Zettelkasten 型のノート・リンク・記憶進化（新しい記憶が既存記憶を書き換える）で、長期会話の multi-hop QA を約 1/10 のトークンで 2 倍超の性能。「検索の agency（agentic RAG）」と「索引の agency（agentic memory）」を分ける境界も整理した。
- context engineering（限られたコンテキストウィンドウに何をどう積むかの設計）→ [[context-engineering]]
  - MemGPT の main context 3 分割（不変の規則／更新される要点／流れる履歴）と閾値駆動の退避が区画化の原型。本番運用のパターン（フェーズ要約・handoff・参照渡し）は [[summaries/2025-multi-agent-research-system]] が記録。
  - Kimi K2.5（[[summaries/2026-kimi-k2.5]], 2026）は「溢れてから削る」reactive な切り詰め（Summary / Discard-all 等）に対し、分解時点でコンテキストを分割する **context sharding**（Agent Swarm）を対置し、同一モデル比較で精度・効率の優位を示した。

### 2. 知識の接続

- RAG（Retrieval-Augmented Generation, 外部知識を検索してプロンプトに与え、それを根拠に生成させる手法）→ [[retrieval-augmented-generation]]
  - 原典（[[summaries/2020-rag]], 2020）はパラメトリック記憶（重み）と非パラメトリック記憶（文書索引）の end-to-end 結合として RAG を定式化し、幻覚の減少・索引差し替えによる知識更新・retrieval collapse を実証した。「知識はパラメータでなく索引に置く」という設計原則の出発点。
- MCP（Model Context Protocol, ツールやデータソースをモデルに接続する標準プロトコル）→ `[[model-context-protocol]]`

### 3. 構成とスケール

- multi-agent systems（複数エージェントの分業・協調、orchestrator-worker 構成）→ [[multi-agent-systems]]
  - Sakana Fugu（[[summaries/2026-sakana-fugu]], 2026）は「どのモデルにどう働かせるか」を学習したオーケストレータで個々のフロンティアモデル単体を超え、**オーケストレーションをモデルスケーリングと直交する新しいスケーリング軸**として実証した。固定集約役の debate/MoA からクエリ適応的なワークフロー生成への世代交代を示す原典。
  - 本番運用の実例は Anthropic Research（[[summaries/2025-multi-agent-research-system]], 2025）: リード＋並列サブエージェント＋CitationAgent の orchestrator-worker で単一エージェント比 +90.2%。効果の正体は「別コンテキストで並列にトークンを費やす容量」（BrowseComp 分散の 80% をトークン量が説明）であり、代償はチャット比 **15 倍**のトークン——適用条件（幅優先・高価値・並列化可能）まで含めた経済学を開示した。
  - Kimi K2.5 の Agent Swarm（[[summaries/2026-kimi-k2.5]], 2026）は、その「いつ・いくつ・どう分けるか」を**プロンプト設計から RL（PARL）へ**移した: 凍結サブエージェント＋補助報酬のアニーリング＋critical steps（最長ブランチ）制約で並列化の意思決定自体を学習し、BrowseComp 78.4%（同一モデル比 +17.8pt。絶対値は発表時点）・実行時間 3〜4.5 倍短縮。フロンティアモデル自体が並列オーケストレーションを内蔵する段階に入った。
- agent frameworks（LangGraph, AutoGen, CrewAI, Claude Agent SDK 等）→ [[agent-frameworks]]
  - 実務の標準語彙は Anthropic「Building Effective Agents」（[[summaries/2024-building-effective-agents]], 2024）が確立: workflow（事前定義コードパス）と agent（動的制御）の区別、5 つの設計パターン、「まず単純に、複雑さは実証されたときだけ」の原則。

### 4. 応用

- coding agents（SWE-agent, Devin, Claude Code, Cursor 等）→ [[coding-agents]]
  - 初出典は Anthropic の長時間ハーネス記事（[[summaries/2025-effective-harnesses]], 2025）: 数日級の自律コーディングを「initializer が環境の足場（feature list JSON・進捗ログ・init.sh・git）を作り、coding agent が毎セッション 1 機能ずつクリーンに進める」二部構成で解く。引き継ぎは要約でなく検査可能な構造化 artifact——時間方向の分業の実務解。
  - 続編（[[summaries/2026-harness-design]], 2026）は planner/generator/evaluator の 3 エージェント（GAN 着想の生成・採点分離、sprint contract）へ発展させ、同時に**縮小の方法論**——「ハーネスの部品はモデル能力への仮定であり、新モデルごとに 1 部品ずつ外して検証する」——を実録（Opus 4.5→4.6 で context reset・スプリント分解が不要化）。ハーネス設計を「作る技術」から「保守する技術」へ更新した。
  - Meta-Harness（[[summaries/2026-meta-harness]], 2026）はその先——**ハーネス設計の自動化**——を実証: コーディングエージェントを proposer に、過去の全候補のコード・スコア・実行トレースをファイルシステム越しに検分させてハーネスコードを探索させると、テキスト分類（ACE +7.7pt・トークン 1/4）・数学検索（未見 5 モデルへ転移 +4.7pt）・TerminalBench-2（人手ベースライン超え）の 3 領域で人手設計を上回った。鍵は「フィードバックを要約せず生のまま選択的に読ませる」こと。人手の連作（作る→剥がす）に対する「探索で発見する」第三の道であり、コーディングエージェントの成熟が自分自身の実行基盤の改善という再帰を可能にした記録 → [[agent-frameworks]]。
- computer use / GUI 操作エージェント → [[computer-use-agents]]
  - 初出典は Kimi K2.5（[[summaries/2026-kimi-k2.5]], 2026）: スクリーンショット観測 → pyautogui 操作のループで、汎用マルチモーダルモデルのまま OSWorld-Verified 63.3%（Operator 42.9% 超え・Opus 4.5 の 66.3% に肉薄）・WebArena 58.9%。GUI trajectory を事前学習データに混ぜ、視覚 RL でグラウンディングを鍛える製法まで開示。
- web agents（ブラウザ操作・情報収集）→ `[[web-agents]]`

### 5. 評価・運用・安全性

- agent evaluation（SWE-bench, GAIA, WebArena, τ-bench 等のベンチマークと、解決率・コスト・ステップ数といった指標）→ [[agent-evaluation]]
  - MASFT（[[summaries/2025-masft]], 2025）はスコアでなく**トレースを一次データとする失敗分析**の方法論（Grounded Theory・Cohen's κ・LLM-as-a-judge）を確立し、「MAS の失敗は個々の LLM でなく組織設計の欠陥」という診断を与えた。
  - 実務側の評価・運用の教訓は [[summaries/2025-multi-agent-research-system]]（2025）: 約 20 クエリの小規模評価から直ちに始める、LLM-as-a-judge は単一プロンプト・単一呼び出しが最も人間と整合、人間のテスターだけが情報源選択バイアス（SEO ファーム優先）を発見、状態変更エージェントは終了状態評価。運用面ではエラー地点からの再開・rainbow deployment・会話内容を見ないトレーシングを記録。
- agent safety and guardrails（prompt injection（外部入力に埋め込まれた指示でエージェントを乗っ取る攻撃）、権限設計、sandboxing、HITL）→ [[agent-safety-and-guardrails]]
  - 監視面の実測が [[summaries/2025-cot-faithfulness]]（Anthropic, 2025）: 推論モデルの CoT が実際の判断理由（ヒント）を明かす率は平均 25〜39%、RL で仕込んだ reward hack は >99% 悪用されながら言語化 <2%。**CoT モニタリングは「気づく層」としては有望だが「排除の保証」には使えない**という運用原則を確立した。
- agent observability（trajectory のトレーシングとデバッグ）→ `[[agent-observability]]`

### 6. 土台となる LLM 側

- transformer architecture と個別モデルの世代 → [[transformer-architecture]]
  - 系譜の俯瞰は [[summaries/2026-gpt2-to-kimi3]]（2026）: GPT-2 → Kimi K3 の **22,580 倍**を「固定容量の連想メモリには追い出しポリシーが要る」という一本の線で読む——KV cache の O(N) 成長 → linear attention の固定状態（干渉）→ delta rule の選択的上書き → ゲート減衰 → KDA/MLA ハイブリッド＋MoE。スケーリングは「容量をどこに足すか」の設計である。
  - MoE の基礎は [[summaries/2023-moe-explained]]（Hugging Face, 2023）: 疎な MoE 層＋ルータの仕組み、1991 年からの系譜（Shazeer → GShard → Switch → Mixtral）、負荷分散の 3 点セット（aux loss・z-loss・expert capacity）、「メモリは総パラメータ・計算は活性化分」という MoE 経済の本質。K2/K3・DeepSeek 世代の MoE 採用を理解する土台。その中心にある一次資料が [[summaries/2021-switch-transformers]]（2021）——top-1 ルーティング・selective precision・蒸留 30%・初の 1.6T モデルと、data/model/expert 並列の体系化の原典。
- post-training（RLHF, RLVR 等）→ [[reinforcement-learning-from-human-feedback]]
  - GRPO の起点は DeepSeekMath（[[summaries/2024-deepseekmath]], 2024）: PPO の critic をグループ相対 advantage で置き換えて発明し、「全事後学習手法＝データソース×報酬×勾配係数」の統一パラダイムと「RL は Maj@K を上げるが Pass@K を上げない（分布の尖鋭化）」という基礎観察を残した。
  - DeepSeek-R1（[[summaries/2025-deepseek-r1]], 2025）は、その GRPO を検証可能な報酬だけの大規模 RL（RLVR）で回し、長い思考・自己検証・reflection が**創発**することを公開実証して o1 型推論モデルの製法を開いた。GRPO は [[summaries/2026-sakana-fugu]] のオーケストレータ訓練にも使われ、モデルとエージェントの両方を貫く訓練レシピになっている。
  - 後継の DeepSeek-V4（[[summaries/2026-deepseek-v4]], 2026）は事後学習の主役を **On-Policy Distillation（OPD）** に置換: ドメイン専門家を RL で個別に鍛え、学生の自己生成軌跡上の全語彙逆 KL 蒸留で単一モデルへ統合する——「発見は RL・統合は蒸留」の分業がパイプライン設計の原理に昇格した。
- test-time compute（推論時に計算量を増やして精度を上げる考え方）→ [[test-time-compute]]
  - 体系化は Long CoT サーベイ（[[summaries/2025-long-cot-survey]], 2025, 813 文献）: 推論モデルの思考を deep reasoning・extensive exploration・feasible reflection の**3 特性の統合（Long CoT）**として定義し、垂直/並列スケーリングの 2 型、**推論境界**と **overthinking**（長考は閾値を超えると性能が落ちる）、PRM vs ORM、aha moment への反証までを整理した推論モデル時代の地図。
- 推論の高速化・サービング（KV cache, バッチング, コストとレイテンシ）→ [[llm-inference-optimization]]
  - prefill/decode の 2 相・メモリ帯域律速・カーネル融合の基礎は [[summaries/2026-gpt2-to-kimi3]] が実装レベルで解説（固定状態化でデコード 6 倍、FLOPs 最小 ≠ wall-clock 最小、融合カーネルなしの新活性化は 3 倍遅）。エージェントのトークン経済（[[summaries/2025-multi-agent-research-system]] の 15 倍）の物理的な下部構造。
  - 実務側のレバーと運用の型は [[summaries/2026-llm-optimization-guide]]（Mirantis, 2026）: 量子化 −75%・continuous batching で稼働率 40→90%・PagedAttention −55% の定量カタログと、ワークロードのクラス分け・縮退設計・構成のコード化という運用規律。
  - **エッジ／オンデバイス側の設計**は Gemma 4（[[summaries/2026-gemma-4]], 2026）が参照実装: KV cache −37.5%（local:global 5:1・values=keys・p-RoPE）、QAT（int2/4 で E2B 4.6→0.8GB）、投機的デコード用 MTP drafter の同梱、encoder-free 化（12B は視覚・音声エンコーダを射影に置換）。フロンティア・スケール（K2.5 の 1T MoE）と対をなす「小さく速く配る」路線で、Arena 人間評価では 31B dense が 744B〜1.6T MoE 群に伍した。
  - **1M コンテキストの効率化**は DeepSeek-V4（[[summaries/2026-deepseek-v4]], 2026）が最初の大規模実証: KV の学習圧縮＋スパース選択（CSA/HCA）で 1M 時の KV cache を V3.2 比 10%・一般的な BF16 GQA8 比約 2% に削り、100 万トークンを「日常運用」の水準にした。on-disk KV cache（shared-prefix 再利用）・batch-invariant 決定論カーネル・agentic search の同一モデル実測（精度優位・コスト僅増）まで含め、長ホライズンのエージェントタスクの物理的基盤を更新している。
- エージェント特化の基盤モデル訓練（データ合成＋RL でエージェント能力を作る）
  - [[summaries/2025-kimi-k2]]（Moonshot, 2025）: 1.04T MoE を MuonClip で loss spike ゼロ事前学習し、**実 MCP ツール 3000+＋合成ツール 20,000+ による tool-use trajectory 合成**と、検証可能報酬＋自己批評ルーブリックの joint RL でエージェント能力を仕込む。**非思考のまま** SWE-bench Verified 65.8・τ²-Bench 66.1——「長考の創発」（R1）と対をなす「非思考のエージェント化」の代表原典。
  - 後継の Kimi K2.5（[[summaries/2026-kimi-k2.5]], 2026）は**マルチモーダル化**の製法を追加: 15T トークンの joint 事前学習は early fusion・低 vision 比率が最良、視覚エージェント能力の発火はテキストのみの SFT で足りる（zero-vision SFT）、視覚 RL はテキスト性能まで上げる（双方向転移）。MoonViT-3D（画像・動画共有エンコーダ）・DEP（視覚エンコーダの並列化分離）・Toggle（トークン効率 RL, −25〜30%）まで含め、「エージェント基盤モデルは単一モダリティでは作らない」方向を示した。
- ファインチューニング（LoRA 等の PEFT）→ `[[parameter-efficient-fine-tuning]]`

## 現状のカバレッジ

| 軸 | 取り込み済みの原典 |
| --- | --- |
| 基本構造 | [[summaries/2022-chain-of-thought]]（推論の創発・CoT）、[[summaries/2022-react]]（agent loop・推論と行動の統合・初期のツール利用）、[[summaries/2023-reflexion]]（自己反省・試行間学習）、[[summaries/2023-memgpt]]（階層記憶・仮想コンテキスト管理・イベント駆動制御）、[[summaries/2025-a-mem]]（動的記憶組織化・記憶進化） |
| 知識の接続 | [[summaries/2020-rag]]（検索拡張生成・非パラメトリック記憶・hot-swap） |
| 構成とスケール | [[summaries/2026-sakana-fugu]]（学習されたオーケストレータ）、[[summaries/2025-masft]]（MAS の失敗分類）、[[summaries/2024-building-effective-agents]]（設計パターンとフレームワーク観）、[[summaries/2025-multi-agent-research-system]]（本番 orchestrator-worker・トークン経済学）、[[summaries/2026-kimi-k2.5]]（RL で学習された並列オーケストレーション・context sharding） |
| 応用 | [[summaries/2026-kimi-k2.5]]（computer use: OSWorld / WebArena・GUI エージェント構成）、[[summaries/2025-effective-harnesses]]（coding agents: 長時間自律コーディングのハーネス）、[[summaries/2026-harness-design]]（coding agents: 3 エージェント構成とハーネス縮小）、[[summaries/2026-meta-harness]]（coding agents / frameworks: ハーネスの自動探索・TerminalBench-2）。[[summaries/2026-sakana-fugu]] もコーディング・自律研究・CAD 等の応用例に言及 |
| 評価・運用・安全性 | [[summaries/2025-masft]]（トレース分析・LLM-as-a-judge・失敗分類）、[[summaries/2025-multi-agent-research-system]]（小規模評価・単一ジャッジ・終了状態評価・本番運用の信頼性）、[[summaries/2025-cot-faithfulness]]（CoT 忠実性・CoT モニタリングの限界・安全性）。ベンチマークは [[summaries/2026-sakana-fugu]]（SWE-Bench Pro / Terminal Bench / GPQA / HLE / τ³ 等）、[[summaries/2022-react]]（HotpotQA / FEVER / ALFWorld / WebShop・HITL 介入）も言及 |
| LLM 基盤 | [[summaries/2024-deepseekmath]]（GRPO の発明・統一パラダイム・数学コーパス採掘）、[[summaries/2025-deepseek-r1]]（RLVR・GRPO・蒸留・推論の創発）、[[summaries/2025-long-cot-survey]]（Long CoT の体系化・test-time scaling・推論境界）、[[summaries/2026-gpt2-to-kimi3]]（アーキテクチャの系譜・KV cache・推論効率）、[[summaries/2026-llm-optimization-guide]]（本番推論最適化の実務・サービング運用）、[[summaries/2025-kimi-k2]]（エージェント特化の事前学習＋データ合成＋joint RL）、[[summaries/2023-moe-explained]]（MoE の仕組み・歴史・負荷分散）、[[summaries/2026-kimi-k2.5]]（マルチモーダル joint 訓練・zero-vision SFT・トークン効率 RL）、[[summaries/2026-gemma-4]]（エッジ〜31B の効率化設計・encoder-free・QAT・投機的デコード）、[[summaries/2026-deepseek-v4]]（1M コンテキスト効率・CSA/HCA・mHC・OPD・RL/rollout インフラ）。[[summaries/2026-sakana-fugu]] も SFT・進化戦略・GRPO の訓練レシピに言及 |

6 軸すべてに少なくとも 1 件の原典が入った（2026-07-26 時点）。「応用」軸には 2026-07-28 の Kimi K2.5 で初の専用記述（computer use）が入った。以後は各軸の深化（例: 知識の接続における MCP、応用における coding agents の専用原典）を `lint` のデータギャップとして追う。

## 関連ページ

- [[index]] — 全ページのカタログ
- [[log]] — 取り込み・更新の時系列ログ
