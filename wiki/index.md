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

- [[summaries/2022-react]] — ReAct（ICLR 2023）。思考と行動を交互に生成させ、外部接地で幻覚を抑えつつ行動を推論で導くパラダイム。agent loop の原型。

### Articles / Blogs

（まだありません。`raw/articles/` に記事 markdown を置いて ingest してください）

### Docs

（公式ドキュメント・プロトコル仕様・system card の要約をここに置く）

## Translations

- [[translations/2022-react]] — ReAct 論文の全文翻訳（付録・プロンプト含む。プロンプトと軌跡は原文のまま収録）。

## Concepts

- [[reasoning-and-planning]] — LLM に思考過程・計画を明示的に生成させる手法群。CoT・CoT-SC・ReAct・ToT を扱う。
- [[agent-loop]] — 観測→思考→行動の実行ループ。定式化、thought の密度、停止条件、典型的失敗モード。
- [[tool-use-and-function-calling]] — モデルが外部ツールを呼ぶ仕組み。ReAct の Wikipedia API から function calling までの系譜。

未作成の想定スラグ（CLAUDE.md §1 の命名規約より。作成したら上のリストへ移す）：
`llm-agents` / `self-reflection` / `agent-memory` / `retrieval-augmented-generation` / `multi-agent-systems` / `context-engineering` / `model-context-protocol` / `agent-frameworks` / `coding-agents` / `computer-use-agents` / `web-agents` / `agent-evaluation` / `agent-safety-and-guardrails` / `agent-observability` / `reinforcement-learning-from-human-feedback` / `test-time-compute` / `llm-inference-optimization` / `parameter-efficient-fine-tuning` / `transformer-architecture`

### 略称リダイレクト

略称に専用ページは作らない。対応する正式名称の概念ページを参照する（CLAUDE.md §1）。

- RAG → [[retrieval-augmented-generation]]
- MCP → [[model-context-protocol]]
- CoT / CoT-SC / ToT / ReAct → [[reasoning-and-planning]]
- function calling / tool call → [[tool-use-and-function-calling]]
- RLHF / RLVR → [[reinforcement-learning-from-human-feedback]]
- PEFT / LoRA / SFT → [[parameter-efficient-fine-tuning]]
- HITL → [[agent-safety-and-guardrails]]
- KV cache → [[llm-inference-optimization]]

## Questions

（まだありません。query の成果物をここに追記します）
