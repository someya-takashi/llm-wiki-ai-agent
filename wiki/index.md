---
type: index
updated: 2026-07-26
---

# Index — AI Agent LLM Wiki

この wiki の全ページカタログ。1 行 = 1 ページ（`- [[<slug>]] — <一行の説明>`）。
ingest / query で新規ページを作るたびに必ずここへ追記する（CLAUDE.md §5）。

## Overview

- [[overview]] — AI Agent 領域全体の総括。原典が増えるたびに更新する。

## Summaries

原典 1 件につき 1 ページ。`source_kind` ごとに小見出しを分ける。

### Papers

- [[summaries/2020-rag]] — RAG（NeurIPS 2020）。パラメトリック/非パラメトリック記憶の end-to-end 結合。幻覚減・索引差し替えによる知識更新・retrieval collapse の初記録。
- [[summaries/2022-chain-of-thought]] — CoT（NeurIPS 2022）。例示に思考連鎖を入れるだけで推論が創発。「考えてから答える」設計すべての祖形。
- [[summaries/2022-react]] — ReAct（ICLR 2023）。思考と行動を交互に生成させ、外部接地で幻覚を抑えつつ行動を推論で導くパラダイム。agent loop の原型。
- [[summaries/2023-reflexion]] — Reflexion（NeurIPS 2023）。報酬を反省文に増幅して記憶し、重み更新なしで試行間学習。HumanEval 91%。
- [[summaries/2023-memgpt]] — MemGPT（ICML 2024）。OS の仮想メモリに倣い、LLM 自身が function call で記憶を階層管理。working context・archival memory の語彙の出発点。
- [[summaries/2026-sakana-fugu]] — Sakana Fugu（2026, テクニカルレポート）。フロンティア LLM 群を束ねる学習されたオーケストレータ。オーケストレーション＝新スケーリング軸の実証。
- [[summaries/2025-masft]] — MASFT（2025, UC Berkeley）。150+ トレース分析による MAS 失敗の初の分類法（14 モード×3 カテゴリ）。「失敗は組織設計の欠陥」。
- [[summaries/2025-deepseek-r1]] — DeepSeek-R1（2025）。検証可能報酬だけの大規模 RL で推論・reflection が創発（RLVR）。o1 級推論の初のオープン実証。
- [[summaries/2025-cot-faithfulness]] — CoT 忠実性（Anthropic, 2025）。効いたヒントを CoT が明かす率 25〜39%。reward hack は >99% 悪用・言語化 <2%。CoT モニタリングの限界の実測。
- [[summaries/2025-kimi-k2]] — Kimi K2（Moonshot, 2025）。1T MoE のエージェント特化オープンモデル。MuonClip・ツール利用データ合成・自己批評ルーブリック RL。非思考で SWE-bench 65.8。
- [[summaries/2025-a-mem]] — A-Mem（2025, Rutgers）。Zettelkasten 型の動的記憶: 保存時の意味づけ・LLM によるリンク判断・記憶進化。multi-hop 2 倍超を 1/10 のトークンで。
- [[summaries/2025-long-cot-survey]] — Long CoT サーベイ（2025, 813 文献）。推論モデルを 3 特性（深い推論・探索・反省）で定義し、6 現象（推論境界・overthinking・test-time scaling 等）を体系化。

### Articles / Blogs

- [[summaries/2024-building-effective-agents]] — Anthropic（2024, 改訂版）。workflow/agent の区別・5 パターン・3 原則（simplicity/transparency/ACI）。実務指針の事実上の標準。
- [[summaries/2025-multi-agent-research-system]] — Anthropic（2025）。Research 機能の本番 orchestrator-worker。+90.2%・トークン 15 倍の経済性・プロンプト 8 原則・20 クエリ評価。
- [[summaries/2026-gpt2-to-kimi3]] — X 記事（@waterloo_intern, 2026）。GPT-2→Kimi K3 の 22,580 倍を「固定容量メモリ＋追い出しポリシー」の系譜として実装コード付きで解説。
- [[summaries/2026-llm-optimization-guide]] — Mirantis（2026, ベンダーブログ）。本番推論最適化の実務ガイド。量子化 −75%・batching で稼働率 40→90%・PagedAttention −55% の定量カタログと運用の型。

### Docs

（公式ドキュメント・プロトコル仕様・system card の要約をここに置く）

## Translations

- [[translations/2020-rag]] — RAG 論文の全文翻訳（付録 A〜I 含む。周辺化・DPR の式は LaTeX 維持、生成例は原文のまま収録）。
- [[translations/2022-chain-of-thought]] — CoT 論文の全文翻訳（付録の全プロンプト・結果表含む。プロンプトと例は原文のまま収録）。
- [[translations/2022-react]] — ReAct 論文の全文翻訳（付録・プロンプト含む。プロンプトと軌跡は原文のまま収録）。
- [[translations/2023-reflexion]] — Reflexion 論文の全文翻訳（Algorithm 1・欠落パネルを ar5iv から復元。軌跡と反省文は原文のまま収録）。
- [[translations/2023-memgpt]] — MemGPT 論文の全文翻訳（付録の全プロンプト含む。Figure 8 キャプションを ar5iv から復元。プロンプトは原文のまま収録）。
- [[translations/2025-cot-faithfulness]] — CoT 忠実性論文の全文翻訳（脚注 2 件を ar5iv から復元。忠実性スコアの定義式は LaTeX 維持、ヒント例は原文のまま収録）。
- [[translations/2025-kimi-k2]] — Kimi K2 テクニカルレポートの全文翻訳（欠落図 6 枚・キャプション 4 件・脚注 6 件を ar5iv から復元。ツール呼び出しテンプレートは原文のまま収録）。
- [[translations/2025-a-mem]] — A-Mem 論文の全文翻訳（多パネル図 13 枚と主キャプション 4 件を ar5iv から復元。付録のプロンプト 3 種は SVG から原文のまま起こして収録）。
- [[translations/2025-long-cot-survey]] — Long CoT サーベイの全文翻訳（図 11 枚・表 7 点収録。分類法ツリーと囲みボックス 10 個をテキスト復元。定義式は LaTeX 維持）。
- [[translations/2026-gpt2-to-kimi3]] — 「From GPT2 to Kimi3, Explained」の全文翻訳（図 22 枚収録。X の数式連結を復元。コード 12 個は原文のまま収録）。
- [[translations/2026-llm-optimization-guide]] — Mirantis「LLM Optimization: Techniques and Guide」の全文翻訳（本文に図なし。カバーバナーは chrome として除外）。
- [[translations/2026-sakana-fugu]] — Sakana Fugu テクニカルレポートの全文翻訳（付録・棋譜含む。プロンプトと棋譜は原文のまま収録）。
- [[translations/2025-masft]] — MASFT 論文の全文翻訳（付録の失敗事例トレース・介入プロンプト含む。トレースとプロンプトは原文のまま収録）。
- [[translations/2025-deepseek-r1]] — DeepSeek-R1 論文の全文翻訳（GRPO の式・aha moment の記録含む。テンプレートと応答例は原文のまま収録）。
- [[translations/2024-building-effective-agents]] — Anthropic「Building Effective Agents」の全文翻訳（付録のツール設計論含む）。
- [[translations/2025-multi-agent-research-system]] — Anthropic「How we built our multi-agent research system」の全文翻訳（付録の運用 Tips 含む。図 3 枚収録）。

## Concepts

- [[reasoning-and-planning]] — LLM に思考過程・計画を明示的に生成させる手法群。CoT・CoT-SC・ReAct・ToT を扱う。
- [[agent-loop]] — 観測→思考→行動の実行ループ。定式化、thought の密度、停止条件、典型的失敗モード。
- [[tool-use-and-function-calling]] — モデルが外部ツールを呼ぶ仕組み。ReAct の Wikipedia API から function calling までの系譜。
- [[multi-agent-systems]] — 複数 LLM エージェントの協調。debate / MoA / ルーティング / 学習されたオーケストレータ（Fugu）の類型と、MASFT による失敗分類。
- [[agent-evaluation]] — エージェント評価の方法論。ベンチマーク型／トレース分析型／LLM-as-a-judge の 3 類型と指標の整理。
- [[agent-frameworks]] — 設計パターン（workflow 5 種＋agent）とフレームワーク観。「まず単純に、複雑さは実証されたときだけ」。
- [[self-reflection]] — 失敗を言語で振り返り試行間で学ぶ仕組み。Reflexion / Self-Refine と、盲目的リトライ無効・FP 即死などの設計論点。
- [[reinforcement-learning-from-human-feedback]] — 事後訓練の RL。RLHF（選好報酬）と RLVR（検証可能報酬）の 2 系統、GRPO、蒸留 vs 直接 RL。
- [[retrieval-augmented-generation]] — 検索で外部知識を注入して生成。訓練時組み込み型と推論時注入型の 2 層、hot-swap、collapse。
- [[agent-memory]] — コンテキストを超えて保持・想起する記憶の設計。MemGPT の階層記憶・Reflexion のエピソード記憶・共有境界。
- [[context-engineering]] — 限られたウィンドウに何をどう積むか。区画化・圧縮と引き継ぎ・参照渡し・lost in the middle。
- [[agent-safety-and-guardrails]] — 安全対策の 4 層（行動空間・ガードレール・監視・HITL）。CoT モニタリングの可能性と限界。
- [[test-time-compute]] — 推論時に計算を積んで精度を買う第二のスケーリング軸。垂直/並列の 2 型・推論境界・overthinking。
- [[transformer-architecture]] — decoder-only の基本構造と attention の系譜（softmax→linear→delta→gated→ハイブリッド）・MoE・AttnRes。
- [[llm-inference-optimization]] — 推論を速く安く捌く側。prefill/decode・TTFT/TPS・KV cache の帯域律速・カーネル融合。

未作成の想定スラグ（CLAUDE.md §1 の命名規約より。作成したら上のリストへ移す）：
`llm-agents` / `model-context-protocol` / `coding-agents` / `computer-use-agents` / `web-agents` / `agent-observability` / `parameter-efficient-fine-tuning`

### 略称リダイレクト

略称に専用ページは作らない。対応する正式名称の概念ページを参照する（CLAUDE.md §1）。

- RAG / DPR / dense retrieval / BM25 → [[retrieval-augmented-generation]]
- MCP → [[model-context-protocol]]
- CoT / CoT-SC / ToT / ReAct → [[reasoning-and-planning]]
- function calling / tool call → [[tool-use-and-function-calling]]
- MoA / Mixture-of-Agents / orchestrator-worker / MASFT → [[multi-agent-systems]]
- LLM-as-a-judge / pass@k / Cohen's κ → [[agent-evaluation]]
- ACI / workflow パターン / prompt chaining / evaluator-optimizer → [[agent-frameworks]]
- Reflexion / Self-Refine / verbal reinforcement → [[self-reflection]]
- RLHF / RLVR / GRPO / PRM → [[reinforcement-learning-from-human-feedback]]
- MemGPT / A-Mem / agentic memory / Zettelkasten / memory evolution / working context / archival memory / recursive summary → [[agent-memory]]
- lost in the middle / scratchpad → [[context-engineering]]
- PEFT / LoRA / SFT → [[parameter-efficient-fine-tuning]]
- HITL / CoT モニタリング / CoT faithfulness / prompt injection / sandboxing / safety case → [[agent-safety-and-guardrails]]
- Long CoT / test-time scaling / overthinking / reasoning boundary / Best-of-N / budget forcing → [[test-time-compute]]
- KV cache / TTFT / TPS / prefill / decode / FlashAttention / PagedAttention / quantization / continuous batching → [[llm-inference-optimization]]
- MoE / MLA / linear attention / DeltaNet / Mamba / KDA / AttnRes / decoder-only → [[transformer-architecture]]

## Questions

（まだありません。query の成果物をここに追記します）
