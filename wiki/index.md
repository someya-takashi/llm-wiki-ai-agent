---
type: index
updated: 2026-07-31
---

# Index — AI Agent LLM Wiki

この wiki の全ページカタログ。1 行 = 1 ページ（`- [[<slug>]] — <一行の説明>`）。
ingest / query で新規ページを作るたびに必ずここへ追記する（CLAUDE.md §5）。

## Overview

- [[overview]] — AI Agent 領域全体の総括。原典が増えるたびに更新する。

## Summaries

原典 1 件につき 1 ページ。`source_kind` ごとに小見出しを分ける。

### Papers

- [[summaries/2021-switch-transformers]] — Switch Transformers（JMLR 2022）。MoE 実用化の転換点: top-1 ルーティング・selective precision・負荷分散損失・蒸留 30%・初の 1.6T モデル。並列化体系（data/model/expert）の原典。
- [[summaries/2020-rag]] — RAG（NeurIPS 2020）。パラメトリック/非パラメトリック記憶の end-to-end 結合。幻覚減・索引差し替えによる知識更新・retrieval collapse の初記録。
- [[summaries/2022-chain-of-thought]] — CoT（NeurIPS 2022）。例示に思考連鎖を入れるだけで推論が創発。「考えてから答える」設計すべての祖形。
- [[summaries/2022-react]] — ReAct（ICLR 2023）。思考と行動を交互に生成させ、外部接地で幻覚を抑えつつ行動を推論で導くパラダイム。agent loop の原型。
- [[summaries/2023-reflexion]] — Reflexion（NeurIPS 2023）。報酬を反省文に増幅して記憶し、重み更新なしで試行間学習。HumanEval 91%。
- [[summaries/2024-deepseekmath]] — DeepSeekMath（2024）。GRPO の発明論文。critic をグループ相対 advantage で置換・統一パラダイム・「RL は Maj@K を上げ Pass@K を上げない」。120B 数学コーパスの採掘パイプラインとコード訓練→推論の転移実証。
- [[summaries/2023-memgpt]] — MemGPT（ICML 2024）。OS の仮想メモリに倣い、LLM 自身が function call で記憶を階層管理。working context・archival memory の語彙の出発点。
- [[summaries/2026-sakana-fugu]] — Sakana Fugu（2026, テクニカルレポート）。フロンティア LLM 群を束ねる学習されたオーケストレータ。オーケストレーション＝新スケーリング軸の実証。
- [[summaries/2025-masft]] — MASFT（2025, UC Berkeley）。150+ トレース分析による MAS 失敗の初の分類法（14 モード×3 カテゴリ）。「失敗は組織設計の欠陥」。
- [[summaries/2025-deepseek-r1]] — DeepSeek-R1（2025）。検証可能報酬だけの大規模 RL で推論・reflection が創発（RLVR）。o1 級推論の初のオープン実証。
- [[summaries/2025-cot-faithfulness]] — CoT 忠実性（Anthropic, 2025）。効いたヒントを CoT が明かす率 25〜39%。reward hack は >99% 悪用・言語化 <2%。CoT モニタリングの限界の実測。
- [[summaries/2025-kimi-k2]] — Kimi K2（Moonshot, 2025）。1T MoE のエージェント特化オープンモデル。MuonClip・ツール利用データ合成・自己批評ルーブリック RL。非思考で SWE-bench 65.8。
- [[summaries/2025-a-mem]] — A-Mem（2025, Rutgers）。Zettelkasten 型の動的記憶: 保存時の意味づけ・LLM によるリンク判断・記憶進化。multi-hop 2 倍超を 1/10 のトークンで。
- [[summaries/2025-long-cot-survey]] — Long CoT サーベイ（2025, 813 文献）。推論モデルを 3 特性（深い推論・探索・反省）で定義し、6 現象（推論境界・overthinking・test-time scaling 等）を体系化。
- [[summaries/2026-kimi-k2.5]] — Kimi K2.5（Moonshot, 2026）。K2 後継のマルチモーダル agentic モデル。テキスト×視覚の同時最適化（early fusion・zero-vision SFT・双方向転移）と Agent Swarm（PARL で並列化を学習。BrowseComp 78.4%・レイテンシ 1/4.5）。
- [[summaries/2026-gemma-4]] — Gemma 4（Google DeepMind, 2026）。エッジ〜31B のオープンマルチモーダル系列。encoder-free 12B・KV cache −37.5%・MTP drafter・QAT・thinking トグル。Arena で dense オープン首位。
- [[summaries/2026-deepseek-v4]] — DeepSeek-V4（2026, プレビュー）。1M コンテキストを日常運用可能に: CSA/HCA（圧縮＋スパース）で KV 10%/FLOPs 27%（対 V3.2）、mHC・Muon・OPD（mixed RL 置換）・DSML・agentic search 実測。Pro-Max はオープン SOTA 再定義。
- [[summaries/2026-meta-harness]] — Meta-Harness（Stanford/MIT/KRAFTON, 2026）。ハーネス設計を外側ループ探索で自動化。コーディングエージェント proposer＋ファイルシステム全履歴。ACE +7.7pt（トークン 1/4）・未見 5 モデルへ転移 +4.7pt・TerminalBench-2 で人手超え。

### Articles / Blogs

- [[summaries/2023-moe-explained]] — Hugging Face（2023）。MoE の定番入門。疎な MoE 層＋ルータ・負荷分散 3 点セット・「メモリ 47B/FLOPs 12B」の分離・FT の落とし穴。
- [[summaries/2024-building-effective-agents]] — Anthropic（2024, 改訂版）。workflow/agent の区別・5 パターン・3 原則（simplicity/transparency/ACI）。実務指針の事実上の標準。
- [[summaries/2025-multi-agent-research-system]] — Anthropic（2025）。Research 機能の本番 orchestrator-worker。+90.2%・トークン 15 倍の経済性・プロンプト 8 原則・20 クエリ評価。
- [[summaries/2026-gpt2-to-kimi3]] — X 記事（@waterloo_intern, 2026）。GPT-2→Kimi K3 の 22,580 倍を「固定容量メモリ＋追い出しポリシー」の系譜として実装コード付きで解説。
- [[summaries/2026-llm-optimization-guide]] — Mirantis（2026, ベンダーブログ）。本番推論最適化の実務ガイド。量子化 −75%・batching で稼働率 40→90%・PagedAttention −55% の定量カタログと運用の型。
- [[summaries/2025-effective-harnesses]] — Anthropic（2025）。長時間エージェントのハーネス設計。initializer/coding の二部構成・feature list JSON・bearings 手順・E2E 検証。「要約でなく構造化 artifact で継ぐ」。
- [[summaries/2026-harness-design]] — Anthropic Labs（2026, 前作の続編）。GAN 着想の planner/generator/evaluator・sprint contract・context anxiety と reset・「部品＝モデル能力への仮定」と 1 部品ずつ剥がす縮小の方法論。solo $9 vs ハーネス $200 の実測。

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
- [[translations/2023-moe-explained]] — Hugging Face「Mixture of Experts Explained」の全文翻訳（欠落していた GShard 図を原ページから復元。数式 5 本を正規化。図 12 枚収録）。
- [[translations/2026-sakana-fugu]] — Sakana Fugu テクニカルレポートの全文翻訳（付録・棋譜含む。プロンプトと棋譜は原文のまま収録）。
- [[translations/2025-masft]] — MASFT 論文の全文翻訳（付録の失敗事例トレース・介入プロンプト含む。トレースとプロンプトは原文のまま収録）。
- [[translations/2025-deepseek-r1]] — DeepSeek-R1 論文の全文翻訳（GRPO の式・aha moment の記録含む。テンプレートと応答例は原文のまま収録）。
- [[translations/2024-building-effective-agents]] — Anthropic「Building Effective Agents」の全文翻訳（付録のツール設計論含む）。
- [[translations/2025-multi-agent-research-system]] — Anthropic「How we built our multi-agent research system」の全文翻訳（付録の運用 Tips 含む。図 3 枚収録）。
- [[translations/2026-kimi-k2.5]] — Kimi K2.5 テクニカルレポートの全文翻訳（欠落していた図7 と脚注 2 件を ar5iv から復元。Table 4 は太字含め正規化。システムプロンプト・ツールスキーマは原文のまま収録）。
- [[translations/2026-gemma-4]] — Gemma 4 テクニカルレポートの全文翻訳（PDF 原典。表 12 点＋Algorithm 1 を転記、図 2 枚はユーザー提供。制御トークン書式・会話例は原文のまま収録）。
- [[translations/2026-deepseek-v4]] — DeepSeek-V4 テクニカルレポートの全文翻訳（欠落していた SVG 図 11 枚を ar5iv から復元。プロンプトボックス 2 点は SVG からテキスト化。脚注 4 件復元・HTML 表 8 個を正規化）。
- [[translations/2021-switch-transformers]] — Switch Transformers 論文の全文翻訳（多パネル図の欠落 4 枚と Mesh TF 擬似コード 3 本・脚注 11 件を ar5iv から復元。付録 A〜F 含む）。
- [[translations/2024-deepseekmath]] — DeepSeekMath 論文の全文翻訳（欠落していた貢献箇条書き・脚注 7 件を ar5iv から復元。ar5iv 自体で平文化していた表 6 個をセル順序から再構成。付録 A.1 の全導出含む）。
- [[translations/2025-effective-harnesses]] — Anthropic「Effective harnesses for long-running agents」の全文翻訳（脚注 1 件を原ページから復元。feature JSON・セッション例は原文のまま収録。GIF 1 枚）。
- [[translations/2026-harness-design]] — Anthropic「Harness design for long-running application development」の全文翻訳（欠落していた画像 4 枚を原ページの Sanity データから位置特定して復元。QA フィードバック・プラン例は原文のまま収録）。
- [[translations/2026-meta-harness]] — Meta-Harness 論文の全文翻訳（欠落していた Figure 1 右パネルと Table 2 を ar5iv から復元＝クリップは図に Table 2 のキャプションが誤結合。脚注 2 件復元・SVG フロー図 4 点と要点／ログボックスをテキスト再構成。proposer の推論ログは原文のまま収録）。

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
- [[computer-use-agents]] — スクリーンショットを観測し GUI を直接操作するエージェント（CUA）。行動空間・OSWorld/WebArena・グラウンディング律速・安全性。
- [[coding-agents]] — コードを書き・実行し・検証するエージェント。ACI・最小ツール構成・長時間ハーネス（initializer/coding）・検証の設計・SWE-bench 系。

未作成の想定スラグ（CLAUDE.md §1 の命名規約より。作成したら上のリストへ移す）：
`llm-agents` / `model-context-protocol` / `web-agents` / `agent-observability` / `parameter-efficient-fine-tuning`

### 略称リダイレクト

略称に専用ページは作らない。対応する正式名称の概念ページを参照する（CLAUDE.md §1）。

- RAG / DPR / dense retrieval / BM25 → [[retrieval-augmented-generation]]
- MCP → [[model-context-protocol]]
- CoT / CoT-SC / ToT / ReAct → [[reasoning-and-planning]]
- function calling / tool call → [[tool-use-and-function-calling]]
- MoA / Mixture-of-Agents / orchestrator-worker / MASFT / Agent Swarm / PARL → [[multi-agent-systems]]
- CUA / computer use / GUI エージェント / OSWorld / WebArena / Operator → [[computer-use-agents]]
- LLM-as-a-judge / pass@k / Cohen's κ → [[agent-evaluation]]
- ACI / workflow パターン / prompt chaining / evaluator-optimizer / harness / ハーネス / Meta-Harness / ハーネスエンジニアリング / GEPA / AlphaEvolve / OpenEvolve / プロンプト自動最適化 → [[agent-frameworks]]
- SWE-agent / Claude Code / Devin / Cursor / initializer agent / feature list / generator-evaluator / sprint contract / TerminalBench / Terminus / 環境ブートストラップ → [[coding-agents]]
- compaction / claude-progress / context anxiety / context reset → [[context-engineering]]
- Reflexion / Self-Refine / verbal reinforcement → [[self-reflection]]
- RLHF / RLVR / GRPO / PPO / DPO / RFT / PRM / OPD / On-Policy Distillation / GRM → [[reinforcement-learning-from-human-feedback]]
- PoT / Program-of-Thought / ツール統合推論 → [[reasoning-and-planning]]
- Maj@K → [[test-time-compute]]
- MemGPT / A-Mem / agentic memory / Zettelkasten / memory evolution / working context / archival memory / recursive summary → [[agent-memory]]
- lost in the middle / scratchpad → [[context-engineering]]
- PEFT / LoRA / SFT → [[parameter-efficient-fine-tuning]]
- HITL / CoT モニタリング / CoT faithfulness / prompt injection / sandboxing / safety case → [[agent-safety-and-guardrails]]
- Long CoT / test-time scaling / overthinking / reasoning boundary / Best-of-N / budget forcing → [[test-time-compute]]
- KV cache / TTFT / TPS / prefill / decode / FlashAttention / PagedAttention / quantization / continuous batching / speculative decoding / MTP / QAT → [[llm-inference-optimization]]
- MoE / MLA / linear attention / DeltaNet / Mamba / KDA / AttnRes / decoder-only / Mixtral / Switch Transformers / expert capacity / router / MoonViT / NaViT / early fusion / DEP / encoder-free / p-RoPE / per-layer embeddings / CSA / HCA / mHC / Hyper-Connections / Muon / attention sink → [[transformer-architecture]]
- Arena / Chatbot Arena / Elo → [[agent-evaluation]]
- DSML → [[tool-use-and-function-calling]]
- agentic search → [[retrieval-augmented-generation]]
- Interleaved Thinking / Quick Instruction → [[context-engineering]]
- context sharding / Discard-all / Hide-Tool-Result → [[context-engineering]]
- Toggle / length-overfitting → [[test-time-compute]]
- expert parallelism / capacity factor / Megablocks / QMoE → [[llm-inference-optimization]]

## Questions

- [[questions/gemma-4-effective-parameters]] — Gemma 4 E2B/E4B の総 5B/8B と実効 2.3B/4.5B の差はなぜ生まれるか。per-layer embeddings の仕組みと、MoE と対比した「容量と実効コストの分離」の整理。
