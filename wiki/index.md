---
type: index
updated: 2026-07-24
---

# Index — AI Agent LLM Wiki

この wiki の全ページカタログ。1 行 = 1 ページ（`- [[<slug>]] — <一行の説明>`）。
ingest / query で新規ページを作るたびに必ずここへ追記する（CLAUDE.md §5）。

## Overview

- [[overview]] — AI Agent 領域全体の総括。原典が増えるたびに更新する。

## Summaries

原典 1 件につき 1 ページ。`source_kind` ごとに小見出しを分ける。

### Papers

- [[summaries/2022-chain-of-thought]] — CoT（NeurIPS 2022）。例示に思考連鎖を入れるだけで推論が創発。「考えてから答える」設計すべての祖形。
- [[summaries/2022-react]] — ReAct（ICLR 2023）。思考と行動を交互に生成させ、外部接地で幻覚を抑えつつ行動を推論で導くパラダイム。agent loop の原型。
- [[summaries/2023-reflexion]] — Reflexion（NeurIPS 2023）。報酬を反省文に増幅して記憶し、重み更新なしで試行間学習。HumanEval 91%。
- [[summaries/2026-sakana-fugu]] — Sakana Fugu（2026, テクニカルレポート）。フロンティア LLM 群を束ねる学習されたオーケストレータ。オーケストレーション＝新スケーリング軸の実証。
- [[summaries/2025-masft]] — MASFT（2025, UC Berkeley）。150+ トレース分析による MAS 失敗の初の分類法（14 モード×3 カテゴリ）。「失敗は組織設計の欠陥」。

### Articles / Blogs

- [[summaries/2024-building-effective-agents]] — Anthropic（2024, 改訂版）。workflow/agent の区別・5 パターン・3 原則（simplicity/transparency/ACI）。実務指針の事実上の標準。

### Docs

（公式ドキュメント・プロトコル仕様・system card の要約をここに置く）

## Translations

- [[translations/2022-chain-of-thought]] — CoT 論文の全文翻訳（付録の全プロンプト・結果表含む。プロンプトと例は原文のまま収録）。
- [[translations/2022-react]] — ReAct 論文の全文翻訳（付録・プロンプト含む。プロンプトと軌跡は原文のまま収録）。
- [[translations/2023-reflexion]] — Reflexion 論文の全文翻訳（Algorithm 1・欠落パネルを ar5iv から復元。軌跡と反省文は原文のまま収録）。
- [[translations/2026-sakana-fugu]] — Sakana Fugu テクニカルレポートの全文翻訳（付録・棋譜含む。プロンプトと棋譜は原文のまま収録）。
- [[translations/2025-masft]] — MASFT 論文の全文翻訳（付録の失敗事例トレース・介入プロンプト含む。トレースとプロンプトは原文のまま収録）。
- [[translations/2024-building-effective-agents]] — Anthropic「Building Effective Agents」の全文翻訳（付録のツール設計論含む）。

## Concepts

- [[reasoning-and-planning]] — LLM に思考過程・計画を明示的に生成させる手法群。CoT・CoT-SC・ReAct・ToT を扱う。
- [[agent-loop]] — 観測→思考→行動の実行ループ。定式化、thought の密度、停止条件、典型的失敗モード。
- [[tool-use-and-function-calling]] — モデルが外部ツールを呼ぶ仕組み。ReAct の Wikipedia API から function calling までの系譜。
- [[multi-agent-systems]] — 複数 LLM エージェントの協調。debate / MoA / ルーティング / 学習されたオーケストレータ（Fugu）の類型と、MASFT による失敗分類。
- [[agent-evaluation]] — エージェント評価の方法論。ベンチマーク型／トレース分析型／LLM-as-a-judge の 3 類型と指標の整理。
- [[agent-frameworks]] — 設計パターン（workflow 5 種＋agent）とフレームワーク観。「まず単純に、複雑さは実証されたときだけ」。
- [[self-reflection]] — 失敗を言語で振り返り試行間で学ぶ仕組み。Reflexion / Self-Refine と、盲目的リトライ無効・FP 即死などの設計論点。

未作成の想定スラグ（CLAUDE.md §1 の命名規約より。作成したら上のリストへ移す）：
`llm-agents` / `agent-memory` / `retrieval-augmented-generation` / `context-engineering` / `model-context-protocol` / `coding-agents` / `computer-use-agents` / `web-agents` / `agent-safety-and-guardrails` / `agent-observability` / `reinforcement-learning-from-human-feedback` / `test-time-compute` / `llm-inference-optimization` / `parameter-efficient-fine-tuning` / `transformer-architecture`

### 略称リダイレクト

略称に専用ページは作らない。対応する正式名称の概念ページを参照する（CLAUDE.md §1）。

- RAG → [[retrieval-augmented-generation]]
- MCP → [[model-context-protocol]]
- CoT / CoT-SC / ToT / ReAct → [[reasoning-and-planning]]
- function calling / tool call → [[tool-use-and-function-calling]]
- MoA / Mixture-of-Agents / orchestrator-worker / MASFT → [[multi-agent-systems]]
- LLM-as-a-judge / pass@k / Cohen's κ → [[agent-evaluation]]
- ACI / workflow パターン / prompt chaining / evaluator-optimizer → [[agent-frameworks]]
- Reflexion / Self-Refine / verbal reinforcement → [[self-reflection]]
- RLHF / RLVR → [[reinforcement-learning-from-human-feedback]]
- PEFT / LoRA / SFT → [[parameter-efficient-fine-tuning]]
- HITL → [[agent-safety-and-guardrails]]
- KV cache → [[llm-inference-optimization]]

## Questions

（まだありません。query の成果物をここに追記します）
