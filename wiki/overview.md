---
type: overview
tags: [llm-agents, ai-agent, overview]
updated: 2026-07-24
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
- planning（計画立案）と self-reflection（自己反省による軌道修正）→ [[reasoning-and-planning]], `[[self-reflection]]`
- memory（短期＝コンテキスト内、長期＝外部ストア）→ `[[agent-memory]]`
- context engineering（限られたコンテキストウィンドウに何をどう積むかの設計）→ `[[context-engineering]]`

### 2. 知識の接続

- RAG（Retrieval-Augmented Generation, 外部知識を検索してプロンプトに与え、それを根拠に生成させる手法）→ `[[retrieval-augmented-generation]]`
- MCP（Model Context Protocol, ツールやデータソースをモデルに接続する標準プロトコル）→ `[[model-context-protocol]]`

### 3. 構成とスケール

- multi-agent systems（複数エージェントの分業・協調、orchestrator-worker 構成）→ [[multi-agent-systems]]
  - Sakana Fugu（[[summaries/2026-sakana-fugu]], 2026）は「どのモデルにどう働かせるか」を学習したオーケストレータで個々のフロンティアモデル単体を超え、**オーケストレーションをモデルスケーリングと直交する新しいスケーリング軸**として実証した。固定集約役の debate/MoA からクエリ適応的なワークフロー生成への世代交代を示す原典。
- agent frameworks（LangGraph, AutoGen, CrewAI, Claude Agent SDK 等）→ `[[agent-frameworks]]`

### 4. 応用

- coding agents（SWE-agent, Devin, Claude Code, Cursor 等）→ `[[coding-agents]]`
- computer use / GUI 操作エージェント → `[[computer-use-agents]]`
- web agents（ブラウザ操作・情報収集）→ `[[web-agents]]`

### 5. 評価・運用・安全性

- agent evaluation（SWE-bench, GAIA, WebArena, τ-bench 等のベンチマークと、解決率・コスト・ステップ数といった指標）→ [[agent-evaluation]]
  - MASFT（[[summaries/2025-masft]], 2025）はスコアでなく**トレースを一次データとする失敗分析**の方法論（Grounded Theory・Cohen's κ・LLM-as-a-judge）を確立し、「MAS の失敗は個々の LLM でなく組織設計の欠陥」という診断を与えた。
- agent safety and guardrails（prompt injection（外部入力に埋め込まれた指示でエージェントを乗っ取る攻撃）、権限設計、sandboxing、HITL）→ `[[agent-safety-and-guardrails]]`
- agent observability（trajectory のトレーシングとデバッグ）→ `[[agent-observability]]`

### 6. 土台となる LLM 側

- transformer architecture と個別モデルの世代 → `[[transformer-architecture]]`
- post-training（RLHF, RLVR 等）→ `[[reinforcement-learning-from-human-feedback]]`
- test-time compute（推論時に計算量を増やして精度を上げる考え方）→ `[[test-time-compute]]`
- 推論の高速化・サービング（KV cache, バッチング, コストとレイテンシ）→ `[[llm-inference-optimization]]`
- ファインチューニング（LoRA 等の PEFT）→ `[[parameter-efficient-fine-tuning]]`

## 現状のカバレッジ

| 軸 | 取り込み済みの原典 |
| --- | --- |
| 基本構造 | [[summaries/2022-react]]（agent loop・推論と行動の統合・初期のツール利用） |
| 知識の接続 | （なし） |
| 構成とスケール | [[summaries/2026-sakana-fugu]]（学習されたオーケストレータ）、[[summaries/2025-masft]]（MAS の失敗分類） |
| 応用 | （なし。ただし [[summaries/2026-sakana-fugu]] がコーディング・自律研究・CAD 等の応用例に言及) |
| 評価・運用・安全性 | [[summaries/2025-masft]]（トレース分析・LLM-as-a-judge・失敗分類）。ベンチマークは [[summaries/2026-sakana-fugu]]（SWE-Bench Pro / Terminal Bench / GPQA / HLE / τ³ 等）、[[summaries/2022-react]]（HotpotQA / FEVER / ALFWorld / WebShop・HITL 介入）も言及 |
| LLM 基盤 | （なし。ただし [[summaries/2026-sakana-fugu]] が SFT・進化戦略・GRPO の訓練レシピに言及） |

原典が増えたらこの表を更新し、空白の軸は `lint` の「データギャップ」として次に読むべき原典の候補を挙げる。

## 関連ページ

- [[index]] — 全ページのカタログ
- [[log]] — 取り込み・更新の時系列ログ
