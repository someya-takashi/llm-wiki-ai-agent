---
type: index
updated: 2026-08-02
---

# Index — AI Agent LLM Wiki

この wiki の全ページカタログ。1 行 = 1 ページ（`- [[<slug>]] — <一行の説明>`）。
ingest / query で新規ページを作るたびに必ずここへ追記する（CLAUDE.md §5）。

## Overview

- [[overview]] — AI Agent 領域全体の総括。原典が増えるたびに更新する。

## Summaries

原典 1 件につき 1 ページ。`source_kind` ごとに小見出しを分ける。

### Papers

- [[summaries/2023-llm-agents-survey]] — Xi et al.（Fudan NLP, 2023, 686 文献）。分野の語彙を定めた古典サーベイ: brain/perception/action の 3 モジュール・応用 3 分類（単一/マルチ/人間協調）・エージェント社会・評価 4 観点・安全 3 論点。
- [[summaries/2024-llm-security-privacy-survey]] — Das et al.（FIU, 2024）。LLM のセキュリティ攻撃（prompt injection・jailbreak・backdoor・poisoning）とプライバシー攻撃（勾配漏洩・MIA・PII 漏洩）＋防御機構＋応用リスクの脅威分類サーベイ。ジェイルブレイクの 2 失敗モード。
- [[summaries/2022-instructgpt]] — InstructGPT（Ouyang et al., OpenAI, NeurIPS 2022）。RLHF を汎用の指示追従に初適用した一次資料で ChatGPT の前身。SFT→報酬モデル→PPO の 3 段・1.3B が 175B に勝つ・**アラインメント税**と PPO-ptx・整合コストは事前学習の 1.6%・「誰に整合させているのか」の 4 層分解。バイアスは改善せず、指示されれば GPT-3 より有害になる。
- [[summaries/2023-dpo]] — DPO（Rafailov et al., Stanford, NeurIPS 2023）。KL 制約付き最適化の解を逆に解いて報酬を方策で表し、Bradley-Terry が差にしか依存しないので分配関数が消える——**報酬モデルの訓練と RL ループを両方排除**して二値交差エントロピー 1 本にする。報酬-KL フロンティアが PPO を（真の報酬を持つ PPO-GT ですら）支配。GPT-4 ジャッジ検証の参照実装も。
- [[summaries/2024-deepseek-v3]] — DeepSeek-V3（DeepSeek-AI, arXiv:2412.19437, 2024-12）。671B 総/37B 活性の MoE を **278.8 万 H800 GPU 時間（$5.576M）・loss spike ゼロ**で訓練。**補助損失を使わない負荷分散**（バイアス項をルーティングにのみ使う。ただし本質は「均衡の粒度を系列→バッチへ緩めること」で、バッチ単位補助損失も同等）・MLA・MTP・**極大規模での FP8 訓練の初の検証**・DualPipe・冗長エキスパート。R1 の Base であり、同時に R1 から推論を蒸留している。
- [[summaries/2025-moe-survey]] — MoE 包括サーベイ（Mu & Lin, arXiv:2503.07137, 2025-03）。基本設計・学習パラダイム別アルゴリズム・理論・応用の 4 部。**wiki に無かった 2 領域**を持ち込む: ゲーティングの設計空間（コサイン・Soft MoE・指数型分布族）とルーティング水準、継続学習/メタ学習/マルチタスク/RL での MoE、そして理論（普遍近似・MLE 収束と Voronoi 損失・2 層 CNN 単一エキスパートの 87.5% 上限）。独自実験はなく、システム記述は DeepSeek-V3 で止まる。
- [[summaries/2021-switch-transformers]] — Switch Transformers（JMLR 2022）。MoE 実用化の転換点: top-1 ルーティング・selective precision・負荷分散損失・蒸留 30%・初の 1.6T モデル。並列化体系（data/model/expert）の原典。
- [[summaries/2020-rag]] — RAG（NeurIPS 2020）。パラメトリック/非パラメトリック記憶の end-to-end 結合。幻覚減・索引差し替えによる知識更新・retrieval collapse の初記録。
- [[summaries/2022-chain-of-thought]] — CoT（NeurIPS 2022）。例示に思考連鎖を入れるだけで推論が創発。「考えてから答える」設計すべての祖形。
- [[summaries/2022-react]] — ReAct（ICLR 2023）。思考と行動を交互に生成させ、外部接地で幻覚を抑えつつ行動を推論で導くパラダイム。agent loop の原型。
- [[summaries/2023-toolformer]] — Toolformer（Meta AI, NeurIPS 2023）。LM が自己教師ありでツール使用を学習。パープレキシティ改善で API 呼び出しを自己採点・選別。6.7B が GPT-3(175B) を上回る。ツール利用を重みに埋め込む系譜の起点。
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
- [[summaries/2025-llm-reasoning-to-agents]] — LLM 推論から自律 AI エージェントへの横断レビュー（2025, arXiv 2504.19678）。推論・評価・フレームワーク・応用・プロトコルの 5 軸で地図化。約 60 ベンチマークを 8 分類・応用 11 領域・プロトコル MCP/A2A/ACP・Agentic RAG・Agent-as-a-Judge・6 つの未解決課題。

### Articles / Blogs

- [[summaries/2022-rlhf-illustrated]] — Hugging Face（2022-12, Lambert et al.）。RLHF の定番解説。事前学習→報酬モデル→PPO の 3 段パイプライン・なぜ点数でなく順位か（Elo）・KL ペナルティと reward hacking の原初的記述。**3 年半前の記事なので「賞味期限」節で陳腐化を対応づけ済み**。
- [[summaries/2023-moe-explained]] — Hugging Face（2023）。MoE の定番入門。疎な MoE 層＋ルータ・負荷分散 3 点セット・「メモリ 47B/FLOPs 12B」の分離・FT の落とし穴。
- [[summaries/2024-building-effective-agents]] — Anthropic（2024, 改訂版）。workflow/agent の区別・5 パターン・3 原則（simplicity/transparency/ACI）。実務指針の事実上の標準。
- [[summaries/2025-multi-agent-research-system]] — Anthropic（2025）。Research 機能の本番 orchestrator-worker。+90.2%・トークン 15 倍の経済性・プロンプト 8 原則・20 クエリ評価。
- [[summaries/2026-gpt2-to-kimi3]] — X 記事（@waterloo_intern, 2026）。GPT-2→Kimi K3 の 22,580 倍を「固定容量メモリ＋追い出しポリシー」の系譜として実装コード付きで解説。
- [[summaries/2026-llm-optimization-guide]] — Mirantis（2026, ベンダーブログ）。本番推論最適化の実務ガイド。量子化 −75%・batching で稼働率 40→90%・PagedAttention −55% の定量カタログと運用の型。
- [[summaries/2025-effective-harnesses]] — Anthropic（2025）。長時間エージェントのハーネス設計。initializer/coding の二部構成・feature list JSON・bearings 手順・E2E 検証。「要約でなく構造化 artifact で継ぐ」。
- [[summaries/2026-harness-design]] — Anthropic Labs（2026, 前作の続編）。GAN 着想の planner/generator/evaluator・sprint contract・context anxiety と reset・「部品＝モデル能力への仮定」と 1 部品ずつ剥がす縮小の方法論。solo $9 vs ハーネス $200 の実測。
- [[summaries/2025-effective-context-engineering]] — Anthropic（2025-09）。**コンテキストエンジニアリングを命名・定式化した出典**。注意予算という制約と context rot の機構（n² の対関係・訓練分布の偏り・位置内挿の劣化 →「崖でなく性能勾配」）、指導原理「最小の高信号なトークン集合」、system prompt の適切な高度・ツールの最小集合、just-in-time 取得と段階的開示、長時間タスクの 3 手法（compaction / ノート取り / サブエージェント）と使い分け。**定量的な結果は無く、指針として読む記事**。
- [[summaries/2025-manus-context-engineering]] — Manus（Yichao 'Peak' Ji, 2025）。本番エージェントのコンテキスト設計 6 原則: KV cache 中心設計（入出力比 100:1・単価 10 倍差）・マスクせよ削除するな・ファイルシステム＝究極のコンテキスト・復唱・誤りを残す・few-shot の轍。
- [[summaries/2026-managed-agents]] — Anthropic（2026-04, ハーネス連作 5 本目）。エージェントを session（イベントログ）/ harness＝brain / sandbox＝hands の 3 インターフェースに仮想化。pets vs cattle・認証情報の到達不能化（Git 配線・vault＋プロキシ）・session ≠ context window・TTFT p50 −60%/p95 −90%。
- [[summaries/2026-agent-orchestration-guide]] — Databricks（2026, ベンダーブログ）。エンタープライズ運用の実務チェックリスト。デプロイ形態 6 パターン（中央集権/分散/階層/ハイブリッド/フェデレーション/創発）・単一責務と入出力契約・CI/CD と policy-as-code・暴走コスト上限・承認ゲートと監査証跡。**掲載される 3 つの統計値（35%/30%/75%）はいずれも出典なし**。

### Docs

（公式ドキュメント・プロトコル仕様・system card の要約をここに置く）

## Translations

- [[translations/2020-rag]] — RAG 論文の全文翻訳（付録 A〜I 含む。周辺化・DPR の式は LaTeX 維持、生成例は原文のまま収録）。
- [[translations/2023-dpo]] — DPO 論文の全文翻訳（本文 §1〜7 ＋ **付録 A〜D 全訳**。ar5iv クリップ底本で、多パネル図の右側 3 枚 x2/x4/x6 と脚注 6 件を原ページから復元。付録 A の全導出・全証明、付録 B の PyTorch 実装、付録 C の GPT-4 ジャッジのプロンプト全文（S 版・C 版）を収録。プロンプトとモデル出力は原文のまま。References・謝辞・Author Contributions は除外）。
- [[translations/2022-instructgpt]] — InstructGPT 論文の翻訳（**本文 §1〜5 のみ**＝ユーザー指示により付録 A〜F は対象外。ar5iv クリップ底本。途切れていた著者リストと欠落していた脚注 8 件を原ページから復元。図 7 枚収録、Figure 8/9 の HTML 対比表を markdown 化。プロンプトとモデル出力は原文のまま）。
- [[translations/2022-chain-of-thought]] — CoT 論文の全文翻訳（付録の全プロンプト・結果表含む。プロンプトと例は原文のまま収録）。
- [[translations/2022-react]] — ReAct 論文の全文翻訳（付録・プロンプト含む。プロンプトと軌跡は原文のまま収録）。
- [[translations/2023-toolformer]] — Toolformer 論文の全文翻訳（ar5iv クリップ。欠落していた脚注 8 件を復元。付録 A〜D のプロンプト・実装詳細を原文のまま収録。図 4 枚・Table 2/9 を markdown 化）。
- [[translations/2023-reflexion]] — Reflexion 論文の全文翻訳（Algorithm 1・欠落パネルを ar5iv から復元。軌跡と反省文は原文のまま収録）。
- [[translations/2023-memgpt]] — MemGPT 論文の全文翻訳（付録の全プロンプト含む。Figure 8 キャプションを ar5iv から復元。プロンプトは原文のまま収録）。
- [[translations/2025-cot-faithfulness]] — CoT 忠実性論文の全文翻訳（脚注 2 件を ar5iv から復元。忠実性スコアの定義式は LaTeX 維持、ヒント例は原文のまま収録）。
- [[translations/2025-kimi-k2]] — Kimi K2 テクニカルレポートの全文翻訳（欠落図 6 枚・キャプション 4 件・脚注 6 件を ar5iv から復元。ツール呼び出しテンプレートは原文のまま収録）。
- [[translations/2025-a-mem]] — A-Mem 論文の全文翻訳（多パネル図 13 枚と主キャプション 4 件を ar5iv から復元。付録のプロンプト 3 種は SVG から原文のまま起こして収録）。
- [[translations/2025-long-cot-survey]] — Long CoT サーベイの全文翻訳（図 11 枚・表 7 点収録。分類法ツリーと囲みボックス 10 個をテキスト復元。定義式は LaTeX 維持）。
- [[translations/2026-gpt2-to-kimi3]] — 「From GPT2 to Kimi3, Explained」の全文翻訳（図 22 枚収録。X の数式連結を復元。コード 12 個は原文のまま収録）。
- [[translations/2026-llm-optimization-guide]] — Mirantis「LLM Optimization: Techniques and Guide」の全文翻訳（本文に図なし。カバーバナーは chrome として除外）。
- [[translations/2022-rlhf-illustrated]] — Hugging Face「Illustrating Reinforcement Learning from Human Feedback (RLHF)」の全文翻訳（原ページ照合済み・クリップに欠落なし。図 4 枚収録。ChatGPT 対話のスクリーンショットは原文のままテキストにも起こした。注釈つき文献ガイド「Further reading」は本文の一部として訳出し URL は arXiv 識別子のみ保持。Citation/BibTeX・謝辞・読者コメント欄は除外）。
- [[translations/2023-moe-explained]] — Hugging Face「Mixture of Experts Explained」の全文翻訳（欠落していた GShard 図を原ページから復元。数式 5 本を正規化。図 12 枚収録）。
- [[translations/2026-sakana-fugu]] — Sakana Fugu テクニカルレポートの全文翻訳（付録・棋譜含む。プロンプトと棋譜は原文のまま収録）。
- [[translations/2025-masft]] — MASFT 論文の全文翻訳（付録の失敗事例トレース・介入プロンプト含む。トレースとプロンプトは原文のまま収録）。
- [[translations/2025-deepseek-r1]] — DeepSeek-R1 論文の全文翻訳（GRPO の式・aha moment の記録含む。テンプレートと応答例は原文のまま収録）。
- [[translations/2024-building-effective-agents]] — Anthropic「Building Effective Agents」の全文翻訳（付録のツール設計論含む）。
- [[translations/2025-multi-agent-research-system]] — Anthropic「How we built our multi-agent research system」の全文翻訳（付録の運用 Tips 含む。図 3 枚収録）。
- [[translations/2026-kimi-k2.5]] — Kimi K2.5 テクニカルレポートの全文翻訳（欠落していた図7 と脚注 2 件を ar5iv から復元。Table 4 は太字含め正規化。システムプロンプト・ツールスキーマは原文のまま収録）。
- [[translations/2026-gemma-4]] — Gemma 4 テクニカルレポートの全文翻訳（PDF 原典。表 12 点＋Algorithm 1 を転記、図 2 枚はユーザー提供。制御トークン書式・会話例は原文のまま収録）。
- [[translations/2026-deepseek-v4]] — DeepSeek-V4 テクニカルレポートの全文翻訳（欠落していた SVG 図 11 枚を ar5iv から復元。プロンプトボックス 2 点は SVG からテキスト化。脚注 4 件復元・HTML 表 8 個を正規化）。
- [[translations/2024-deepseek-v3]] — DeepSeek-V3 テクニカルレポートの翻訳（本文 §1〜6 ＋ 付録 B/C。ar5iv クリップ底本で、欠落していた脚注 4 件と Figure 11 の総キャプション、§3.2.2 で途中で切れていた一文を原ページから復元。図 15 枚収録、HTML 表 5 個を markdown 化。Table 6 のみ分量の都合で要点訳。Appendix A の貢献者一覧は謝辞相当として除外）。
- [[translations/2025-moe-survey]] — MoE 包括サーベイの全文翻訳（本文 §I〜VII 全訳）。**ar5iv でも画像化されていない Figure 1（TikZ forest の分類ツリー）を LaTeX マークアップからネスト箇条書きに再構成**。本文図 2 枚を収録。原典に表・脚注は 1 つも存在しないことを ar5iv 照合で確認。
- [[translations/2021-switch-transformers]] — Switch Transformers 論文の全文翻訳（多パネル図の欠落 4 枚と Mesh TF 擬似コード 3 本・脚注 11 件を ar5iv から復元。付録 A〜F 含む）。
- [[translations/2024-deepseekmath]] — DeepSeekMath 論文の全文翻訳（欠落していた貢献箇条書き・脚注 7 件を ar5iv から復元。ar5iv 自体で平文化していた表 6 個をセル順序から再構成。付録 A.1 の全導出含む）。
- [[translations/2025-effective-harnesses]] — Anthropic「Effective harnesses for long-running agents」の全文翻訳（脚注 1 件を原ページから復元。feature JSON・セッション例は原文のまま収録。GIF 1 枚）。
- [[translations/2026-harness-design]] — Anthropic「Harness design for long-running application development」の全文翻訳（欠落していた画像 4 枚を原ページの Sanity データから位置特定して復元。QA フィードバック・プラン例は原文のまま収録）。
- [[translations/2026-meta-harness]] — Meta-Harness 論文の全文翻訳（欠落していた Figure 1 右パネルと Table 2 を ar5iv から復元＝クリップは図に Table 2 のキャプションが誤結合。脚注 2 件復元・SVG フロー図 4 点と要点／ログボックスをテキスト再構成。proposer の推論ログは原文のまま収録）。
- [[translations/2023-llm-agents-survey]] — LLM エージェント・サーベイ（Xi et al.）の全文翻訳（PDF 原典・86 ページ中本文 45 ページ。画像なし＝ar5iv 変換失敗。分類ツリー図 5 点はネスト箇条書きとしてテキスト転写、脚注 6 件収録）。
- [[translations/2025-effective-context-engineering]] — Anthropic「Effective context engineering for AI agents」の全文翻訳（原ページと文単位で照合しクリップに欠落なしを確認。空だった著者・公開日を復元、語連結 `needle-in-a-haystackstyle` を補正。図 2 枚収録し、システムプロンプトの 3 段階比較図はプロンプト全文を原文のままテキストにも転記。謝辞と末尾の製品導線 1 行は除外。原典側のリンク切れ＝「Sonnet 4.5 launch」が記事自身を指す点も注記）。
- [[translations/2025-manus-context-engineering]] — Manus「Context Engineering for AI Agents」の全文翻訳（クリップで脱落した一文の冒頭を原ページから復元＝リンク句が frontmatter author 欄に化けていた。図 6 枚収録・prefill 文字列は原文のまま）。
- [[translations/2024-llm-security-privacy-survey]] — LLM セキュリティ・プライバシー・サーベイ（Das et al.）の全文翻訳（ar5iv クリップ。図 7 枚・Table 1/2・脚注 1 件・数式を収録。分類図と魚骨図は訳注でテキスト補足）。
- [[translations/2026-managed-agents]] — Anthropic「Scaling Managed Agents: Decoupling the brain from the hands」の全文翻訳（クリップは良好で図 4 枚とも欠落なし。原典に図キャプションが無いため figcaption は訳注として付した。インターフェース仕様の図は表としても転記。冒頭の製品導線 1 行と Acknowledgements は除外）。
- [[translations/2026-agent-orchestration-guide]] — Databricks「AI Agent Orchestration: A Guide for Enterprise Systems」の全文翻訳（本文中に図版なし＝原ページ照合済み、画像はすべてサイト UI）。本文に混入していた資料 DL のプロモ枠を除外し、自社製品への誘導リンクはテキストのみ訳出して URL は不収録。
- [[translations/2025-llm-reasoning-to-agents]] — 「From LLM Reasoning to Autonomous AI Agents」（Ferrag et al.）の全文翻訳（Abstract〜VI 全章、929 行）。ar5iv クリップ底本。図 6 枚（x1/x3/x4/x5・agentic/RAG_agentic drawio）を `<figure>` 収録、SVG のみの図（2・3・8〜12 等）はテキスト補足。平坦化していた Table V〜XII を markdown 表に再構成。IV-B 応用（11 領域）は各研究を訳出しドメイン別網羅表を併載。

## Concepts

- [[llm-agents]] — 総論ハブ。エージェントの定義と系譜・brain/perception/action・応用 3 形態・エージェント社会。各論ページへの入口。
- [[reasoning-and-planning]] — LLM に思考過程・計画を明示的に生成させる手法群。CoT・CoT-SC・ReAct・ToT を扱う。
- [[agent-loop]] — 観測→思考→行動の実行ループ。定式化、thought の密度、停止条件、典型的失敗モード。
- [[tool-use-and-function-calling]] — モデルが外部ツールを呼ぶ仕組み。ReAct の Wikipedia API から function calling までの系譜。
- [[multi-agent-systems]] — 複数 LLM エージェントの協調。debate / MoA / ルーティング / 学習されたオーケストレータ（Fugu）の類型と、MASFT による失敗分類。
- [[agent-evaluation]] — エージェント評価の方法論。ベンチマーク型／トレース分析型／LLM-as-a-judge の 3 類型と指標の整理。
- [[agent-frameworks]] — 設計パターン（workflow 5 種＋agent）とフレームワーク観。「まず単純に、複雑さは実証されたときだけ」。
- [[self-reflection]] — 失敗を言語で振り返り試行間で学ぶ仕組み。Reflexion / Self-Refine と、盲目的リトライ無効・FP 即死などの設計論点。
- [[reinforcement-learning-from-human-feedback]] — 事後訓練の RL。**時系列構成**（2022 古典的 RLHF → 2023 DPO → 2024 GRPO → 2025 RLVR/蒸留 → 2025 K2 joint RL → 2026 K2.5 → 2026 OPD）で「何が何を置き換えたか」を追う。
- [[retrieval-augmented-generation]] — 検索で外部知識を注入して生成。訓練時組み込み型と推論時注入型の 2 層、hot-swap、collapse。
- [[agent-memory]] — コンテキストを超えて保持・想起する記憶の設計。MemGPT の階層記憶・Reflexion のエピソード記憶・共有境界。
- [[context-engineering]] — 限られたウィンドウに何をどう積むか。注意予算と context rot・各構成要素の絞り方・just-in-time 取得・区画化・圧縮と引き継ぎ・参照渡し・長時間タスクの 3 手法。
- [[agent-safety-and-guardrails]] — 安全対策の 4 層（行動空間・ガードレール・監視・HITL）。CoT モニタリングの可能性と限界。モデル内在の安全性がどこから来るか（整合の宛先の 4 層分解）。
- [[test-time-compute]] — 推論時に計算を積んで精度を買う第二のスケーリング軸。垂直/並列の 2 型・推論境界・overthinking。
- [[transformer-architecture]] — decoder-only の基本構造と attention の系譜（softmax→linear→delta→gated→ハイブリッド）・AttnRes。MoE は概要のみで詳細は下記へ。
- [[mixture-of-experts]] — 条件付き計算としての MoE。ゲーティング関数の設計空間・ルーティング水準・負荷分散とシステム設計・スパース性の経済・アーキテクチャ外での利用（継続学習/メタ学習/マルチタスク/RL）・理論の現在地。
- [[llm-inference-optimization]] — 推論を速く安く捌く側。prefill/decode・TTFT/TPS・KV cache の帯域律速・カーネル融合。
- [[computer-use-agents]] — スクリーンショットを観測し GUI を直接操作するエージェント（CUA）。行動空間・OSWorld/WebArena・グラウンディング律速・安全性。
- [[coding-agents]] — コードを書き・実行し・検証するエージェント。ACI・最小ツール構成・長時間ハーネス（initializer/coding）・検証の設計・SWE-bench 系。
- [[model-context-protocol]] — エージェントを外部ツール・データ・他エージェントに繋ぐ標準プロトコル群。MCP（縦＝ツール接続）／A2A（横＝エージェント間相互運用）／ACP（ローカル協調）の三つ巴とプロトコル層のセキュリティ。
- [[agent-observability]] — 本番で動くエージェントを外から見えるようにする層。評価との境界・行動ログ/指標/監査証跡の 3 層・非決定性とトレースの限界（CoT の不忠実性）。

未作成の想定スラグ（CLAUDE.md §1 の命名規約より。作成したら上のリストへ移す）：
`web-agents` / `parameter-efficient-fine-tuning`

### 略称リダイレクト

略称に専用ページは作らない。対応する正式名称の概念ページを参照する（CLAUDE.md §1）。

- RAG / DPR / dense retrieval / BM25 → [[retrieval-augmented-generation]]
- MCP / A2A / ACP / Agent-to-Agent / Agent Communication Protocol / エージェント間プロトコル → [[model-context-protocol]]
- CoT / CoT-SC / ToT / ReAct → [[reasoning-and-planning]]
- function calling / tool call / logit masking / response prefill / Toolformer / Gorilla / TALM / 自己教師ありツール学習 → [[tool-use-and-function-calling]]
- MoA / Mixture-of-Agents / orchestrator-worker / MASFT / Agent Swarm / PARL / エージェント社会 / agent society / Generative Agents / CAMEL / MetaGPT → [[multi-agent-systems]]
- brain-perception-action / AI エージェント / autonomous agents / AutoGPT / Voyager / instructor-executor / World Scope / AGI / embodied action / AaaS → [[llm-agents]]
- CUA / computer use / GUI エージェント / OSWorld / WebArena / Operator → [[computer-use-agents]]
- LLM-as-a-judge / pass@k / Cohen's κ → [[agent-evaluation]]
- observability / 可観測性 / トレーシング / tracing / 監査証跡 / audit trail / immutable trace / rainbow deployment / ウォッチドッグ / watchdog / スコープ漂流 / context drift → [[agent-observability]]
- オーケストレーション / orchestration / 中央集権型 / 非中央集権型 / 階層型 / ハイブリッド / フェデレーション型 / 創発型 / policy-as-code / 入出力契約 / group chat orchestration → [[multi-agent-systems]]
- ACI / workflow パターン / prompt chaining / evaluator-optimizer / harness / ハーネス / ハーネスエンジニアリング / GEPA / AlphaEvolve / OpenEvolve / プロンプト自動最適化 → [[agent-frameworks]]
- **meta-harness（語義が 2 つあるので注意 → 両義の対置は [[agent-frameworks]] の「ランタイムの分離」節）**: ①ハーネスを探索で最適化する外側ループ＝Meta-Harness 論文 → [[summaries/2026-meta-harness]] ／ ②多様なハーネスを載せる汎用インターフェース層＝Managed Agents → [[summaries/2026-managed-agents]]
- session / イベントログ / append-only ログ / sandbox / サンドボックス / brain と hands / pets vs cattle / ランタイム分離 / Managed Agents → [[agent-frameworks]]
- SWE-agent / Claude Code / Devin / Cursor / initializer agent / feature list / generator-evaluator / sprint contract / TerminalBench / Terminus / 環境ブートストラップ → [[coding-agents]]
- compaction / claude-progress / context anxiety / context reset / recitation / 復唱 / KV cache ヒット率 / Manus / restorable compression / attention budget / 注意予算 / context rot / context pollution / just-in-time / JIT 取得 / progressive disclosure / 段階的開示 / right altitude / 適切な高度 / structured note-taking / 構造化されたノート取り → [[context-engineering]]
- Reflexion / Self-Refine / verbal reinforcement → [[self-reflection]]
- RLHF / RLVR / GRPO / PPO / DPO / RFT / PRM / OPD / On-Policy Distillation / GRM / 報酬モデル / reward model / 選好モデル / preference model / KL ペナルティ / reward hacking / InstructGPT / PPO-ptx / pretraining mix / alignment tax / アラインメント税 / Bradley-Terry / Plackett-Luce / 暗黙の報酬 / implicit reward / 分配関数 / partition function / Best of N / unlikelihood / HHH / hh-rlhf / Iterated Online RLHF / アラインメント / alignment / 事後訓練 / post-training / TRL / TRLX / RL4LMs / ILQL / NLPO / A2C → [[reinforcement-learning-from-human-feedback]]
- 誰に整合させるのか / who are we aligning to / ラベラー / labeler / 選好データの偏り / epistemic humility / 認識的謙虚さ → [[agent-safety-and-guardrails]]（整合の宛先の 4 層分解）と [[reinforcement-learning-from-human-feedback]]（報酬設計側）
- BLEU / ROUGE / 自動評価指標 → [[agent-evaluation]]（「参照文との照合では品質を測れない」という評価の原点として扱う）
- PoT / Program-of-Thought / ツール統合推論 → [[reasoning-and-planning]]
- Maj@K → [[test-time-compute]]
- MemGPT / A-Mem / agentic memory / Zettelkasten / memory evolution / working context / archival memory / recursive summary → [[agent-memory]]
- lost in the middle / scratchpad → [[context-engineering]]
- PEFT / LoRA / SFT → [[parameter-efficient-fine-tuning]]
- HITL / CoT モニタリング / CoT faithfulness / prompt injection / sandboxing / safety case / jailbreak / DAN / backdoor attack / data poisoning / membership inference / MIA / PII leakage / gradient leakage / SmoothLLM / goal hijacking / competing objectives → [[agent-safety-and-guardrails]]
- Long CoT / test-time scaling / overthinking / reasoning boundary / Best-of-N / budget forcing → [[test-time-compute]]
- MLA / Multi-head Latent Attention → [[transformer-architecture]]
- FP8 / 低精度訓練 / 細粒度量子化 / DualPipe / all-to-all / expert parallelism / EP / IB / NVLink → [[llm-inference-optimization]]
- KV cache / TTFT / TPS / prefill / decode / FlashAttention / PagedAttention / quantization / continuous batching / speculative decoding / MTP / QAT → [[llm-inference-optimization]]
- MLA / linear attention / DeltaNet / Mamba / KDA / AttnRes / decoder-only / MoonViT / NaViT / early fusion / DEP / encoder-free / p-RoPE / per-layer embeddings / CSA / HCA / mHC / Hyper-Connections / Muon / attention sink → [[transformer-architecture]]
- MoE / Mixture of Experts / 専門家混合 / Mixtral / Switch Transformers / GShard / GLaM / ゲーティング関数 / gating function / router / ルータ / expert capacity / 負荷分散 / load balancing / token dropping / Soft MoE / コサインルータ / Top-P ルーティング / 共有エキスパート / shared expert / スパース性スケーリング則 / MMoE / MoA / 条件付き計算 / conditional computation / auxiliary-loss-free / 補助損失なし負荷分散 / バイアス項 / node-limited routing / ノード制限ルーティング / redundant experts / 冗長エキスパート / DeepSeekMoE → [[mixture-of-experts]]
- Arena / Chatbot Arena / Elo → [[agent-evaluation]]
- DSML → [[tool-use-and-function-calling]]
- agentic search → [[retrieval-augmented-generation]]
- Interleaved Thinking / Quick Instruction → [[context-engineering]]
- context sharding / Discard-all / Hide-Tool-Result → [[context-engineering]]
- Toggle / length-overfitting → [[test-time-compute]]
- expert parallelism / capacity factor / Megablocks / QMoE → [[llm-inference-optimization]]

## Questions

- [[questions/gemma-4-effective-parameters]] — Gemma 4 E2B/E4B の総 5B/8B と実効 2.3B/4.5B の差はなぜ生まれるか。per-layer embeddings の仕組みと、MoE と対比した「容量と実効コストの分離」の整理。
