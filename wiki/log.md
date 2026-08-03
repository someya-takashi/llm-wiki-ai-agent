---
type: log
---

# Log — AI Agent LLM Wiki

時系列の append-only ログ。新しいエントリは**このファイルの末尾に追記**する。
見出しは必ず `## [YYYY-MM-DD] <ingest|query|lint|schema-update> | <タイトル>` の形式にする（CLAUDE.md §5）。

`grep "^## \[" log.md | tail -10` で直近の動きを追える。

---

## [2026-07-23] schema-update | Diffusion Model スキーマを AI Agent 向けに書き換え、wiki 雛形を作成

- 更新: `CLAUDE.md`, `.claude/skills/ingest/SKILL.md`, `.claude/skills/query/SKILL.md`, `.claude/skills/lint/SKILL.md`
- 作成: `raw/{papers,articles,images,assets}/`, `wiki/{summaries,translations,concepts,questions}/`, [[index]], [[log]], [[overview]]
- メモ:
  - 対象領域を拡散モデルから AI Agent（＋周辺の LLM・開発基盤）に差し替え。概念スラグ例、landmark 手法の扱い、ベンチマーク例、略称リストを総入れ替えした。
  - `source_kind` に `docs`（公式ドキュメント・プロトコル仕様・system card）を追加。
  - ingest: 付録のシステムプロンプト／ツール定義は再現に不可欠なので翻訳対象に含める、コード・プロンプト例・ツール定義 JSON・対話ログは原文のまま残す、というルールを追加。要約テンプレートに「実装・運用上の示唆」節を追加。
  - query: 回答時に wiki 記述の時点（`ingested` / `updated`）を明示する「鮮度確認」ステップを追加。
  - lint: 陳腐化しやすい記述（ベンチマークスコア、モデル世代、SDK の非推奨 API、価格・コンテキスト長、プロトコル版）を重点点検対象に追加。

## [2026-07-24] ingest | ReAct: Synergizing Reasoning and Acting in Language Models

- 取り込み: `raw/papers/ReAct_Synergizing Reasoning and Acting in Language Models.md`（ar5iv → Obsidian Web Clipper、arXiv:2210.03629, ICLR 2023）
- 作成: [[summaries/2022-react]], [[translations/2022-react]], [[concepts/reasoning-and-planning]], [[concepts/agent-loop]], [[concepts/tool-use-and-function-calling]], `raw/assets/2022-react/`（x1〜x6.png の 6 枚）
- 更新: [[overview]], [[index]]
- メモ:
  - クリップ不良 (1): LaTeX マクロ `\model` が脱落し本文から「ReAct」の語がほぼすべて欠落していたため、翻訳・要約では文脈から復元した（翻訳冒頭に訳注あり）。正式タイトルはユーザー確認済み。
  - クリップ不良 (2): Figure 2（CoT-SC サンプル数比較, x2/x3.png の 2 パネル）が「Table 1」のキャプションと取り違えられ、本物の Table 1（数値表）と脚注 1〜8・Figure 2 キャプションが丸ごと欠落していた。すべて ar5iv 原ページ（HTML）から復元し、翻訳に反映した。
  - 画像は ar5iv から 6 枚全部をローカル保存（ケース A）。取得失敗なし。`file` で PNG であることを検証済み。
  - 付録 A〜E も翻訳対象に含めた（デフォルト規定）。付録 C のプロンプトと付録 D の trajectory は skill 規定により英語原文のまま収録。references / acknowledgments は除外。
  - 概念ページは 3 件構成（ユーザー選択）。reasoning-and-planning を ReAct/CoT の本拠地とし、agent-loop・tool-use-and-function-calling に横断的知見を配置した。

## [2026-07-24] ingest | Sakana Fugu Technical Report

- 取り込み: `raw/papers/Sakana Fugu Technical Report.md`（ar5iv → Obsidian Web Clipper、arXiv:2606.21228, Sakana AI, 2026）
- 作成: [[summaries/2026-sakana-fugu]], [[translations/2026-sakana-fugu]], [[concepts/multi-agent-systems]], `raw/assets/2026-sakana-fugu/`（6 枚）
- 更新: [[concepts/agent-loop]], [[concepts/tool-use-and-function-calling]], [[concepts/reasoning-and-planning]], [[overview]], [[index]]
- メモ:
  - クリップ不良 (1): マクロ脱落により Figure 3/7/8/9 キャプション等で「Fugu」「Fugu-Ultra」の語が欠落 → 文脈から復元（翻訳冒頭に訳注）。
  - クリップ不良 (2): 脚注 1（チェスの Stockfish 解析設定）・脚注 2（株取引の免責）の本文が欠落 → ar5iv 原ページから復元し `[^fn1]` `[^fn2]` として訳出。
  - クリップ不良 (3): Table 1 に空のスペーサ列が混入（ar5iv 由来と確認）→ 正規化した表にした。
  - Figure 1・2・4・6 は ar5iv 自体に画像が存在しない（TikZ/pgfplots 未レンダリングと推定。`<img>` 不在を HTML で確認）→ キャプション訳＋訳注のみ。取得可能な 6 枚（Fig3/5/7/8/9/10）はローカル保存し `file` で PNG 検証済み。画像 URL は `assets/figures/` と `assets/` 直下の 2 系統だった。
  - 付録 A〜C 全訳。blindfold chess のプロンプト（Listing 1）・棋譜（SAN）・FEN は skill 規定により原文のまま。Authors List も全訳。references / acknowledgments は除外。
  - 概念ページは multi-agent-systems のみ新規（ユーザー選択。agent-evaluation は見送り——本原典はベンチマーク情報が豊富なので、将来 SWE-bench 等を ingest する際の lint データギャップ候補として残す）。

## [2026-07-24] schema-update | ingest skill に「クリップ不良への対処」節を追加

- 更新: `.claude/skills/ingest/SKILL.md`
- メモ: ReAct・Sakana Fugu の 2 件の ingest で Obsidian Web Clipper のクリップ不良（マクロ脱落・脚注本文欠落・図表の欠落／取り違え・表の空列・語の連結等）が続けて発生したため、典型パターンと対処原則（読解時の兆候点検、原ページ照合による復元、創作の禁止、訳注と log への記録、raw/ 不変、迷ったらユーザー確認）を skill に規則化した（ユーザー指示）。標準フロー手順 1（読解）にも点検の参照を追記。

## [2026-07-24] ingest | Why Do Multi-Agent LLM Systems Fail?（MASFT）

- 取り込み: `raw/papers/Why Do Multi-Agent LLM Systems Fail_.md`（ar5iv → Obsidian Web Clipper、arXiv:2503.13657, UC Berkeley, 2025）
- 作成: [[summaries/2025-masft]], [[translations/2025-masft]], [[concepts/agent-evaluation]], `raw/assets/2025-masft/`（x1〜x7.png の 7 枚）
- 更新: [[concepts/multi-agent-systems]]（「なぜ失敗するか — MASFT」節を新設）, [[concepts/agent-loop]], [[overview]], [[index]]
- メモ:
  - クリップ不良 (1): FC1〜FC3 の定義ボックスがインライン SVG として混入 → foreignObject 内テキストを抽出し引用ブロックとして復元（訳注あり）。
  - クリップ不良 (2): 脚注 1（MASFT の GitHub リポジトリ URL）が欠落 → ar5iv から復元。
  - 原典由来の誤植: Appendix A で FM-3.2（No or incomplete verification）が「FM-3.3」と重複採番されている（ar5iv 照合で原典由来と確認。Figure 2・付録 D.5 は FM-3.2 表記）→ 忠実に残し訳注を付した。
  - 画像 7 枚（x1〜x7）はクリップ・ar5iv 双方に存在し全取得・`file` 検証済み。取得失敗なし。
  - 付録 A〜F 全訳。Appendix D のトレース抜粋・E/F のプロンプトは skill 規定により原文のまま収録。references は除外（acknowledgments は原典になし）。
  - 概念ページは agent-evaluation を新規作成（ユーザー選択。前回 Fugu ingest で見送った分の回収。根拠原典に 2025-masft と 2026-sakana-fugu の両方を接続）。

## [2026-07-24] ingest | Chain-of-Thought Prompting Elicits Reasoning in Large Language Models

- 取り込み: `raw/papers/Chain-of-Thought Prompting Elicits Reasoning in Large Language Models.md`（ar5iv → Obsidian Web Clipper、arXiv:2201.11903, NeurIPS 2022）
- 作成: [[summaries/2022-chain-of-thought]], [[translations/2022-chain-of-thought]], `raw/assets/2022-chain-of-thought/`（x1〜x4.png の 4 枚）
- 更新: [[concepts/reasoning-and-planning]]（CoT 節を根拠付きに全面改稿。「原典未 ingest」注記を解消）, [[overview]], [[index]]
- メモ:
  - 図 2・4・5・6・7・8・11 は **ar5iv 自体がインライン SVG（pgfplots）で PNG が存在しない**ことを HTML 照合で確認 → キャプション訳＋「数値は付録 B の表に完備」の訳注で対応（図 2 のみ SVG から読み取った数値を訳注に記載）。クリップにも同じ SVG が混入しており、原典 markdown が 678 行で 14 万トークンに膨張していた主因。
  - 脚注 1〜5 の本文欠落 → ar5iv から復元し `[^fnN]` 形式で訳出。
  - Checklist 節はクリップに見出しのみ（NeurIPS 事務書式のため references 同様に訳出対象外とし、訳注で明記）。
  - 付録 A〜H 全訳。付録 F/G/H の入出力例・全プロンプト表（Table 8〜30 の 23 表）は skill 規定により**原文のままスクリプトで転記**（手作業による転記ミスを排除）。付録 B の巨大な結果表は主要規模の抜粋とし、省略行は訳注で raw 参照を明示。
  - 概念ページの新規作成はなし（CoT は landmark 手法のため reasoning-and-planning 内の「代表手法」として拡充。スキーマ §1 の規約どおり）。

## [2026-07-24] ingest | Building Effective AI Agents（Anthropic）

- 取り込み: `raw/articles/Building Effective AI Agents.md`（Anthropic Engineering Blog → Obsidian Web Clipper。初出 2024-12、取り込んだのは Claude Agent SDK 言及を含む改訂版）
- 作成: [[summaries/2024-building-effective-agents]], [[translations/2024-building-effective-agents]], [[concepts/agent-frameworks]], `raw/assets/2024-building-effective-agents/`（fig1〜fig8.png）
- 更新: [[concepts/tool-use-and-function-calling]]（ACI の節を追加）, [[concepts/agent-loop]]（停止条件・ground truth の実務指針を追記）, [[concepts/multi-agent-systems]]（orchestrator-worker の用語の出所を明記）, [[overview]], [[index]]
- メモ:
  - ケース C の初適用。クリップ自体は良好（欠落・マクロ脱落なし）。画像 8 枚はすべて Next.js の CDN 変換 URL（`/_next/image?url=...`）だったため、内側の `www-cdn.anthropic.com` URL にデコードして取得（fig1〜8 連番、全 PNG 検証済み、chrome 混入なし）。
  - Web 記事のため付録相当（Appendix 1・2）も全訳。プロンプト実物はなし。
  - 取り込んだ版は初出（2024-12）からの改訂版で、フレームワーク一覧が Claude Agent SDK / Strands 等に更新されている点を frontmatter と訳注に明記（鮮度管理のため）。
  - 概念ページは agent-frameworks を新規作成（ユーザー選択）。パターン（長持ち）とフレームワーク観（陳腐化しやすい）を分けて記述し、lint の鮮度点検対象であることを意識した構成にした。

## [2026-07-26] ingest | Reflexion: Language Agents with Verbal Reinforcement Learning

- 取り込み: `raw/papers/Reflexion_ Language Agents with Verbal Reinforcement Learning.md`（ar5iv → Obsidian Web Clipper、arXiv:2303.11366, NeurIPS 2023）
- 作成: [[summaries/2023-reflexion]], [[translations/2023-reflexion]], [[concepts/self-reflection]], `raw/assets/2023-reflexion/`（x1〜x8.png の 8 枚）
- 更新: [[concepts/reasoning-and-planning]]（系譜の第 3 幕として Reflexion を根拠付きに）, [[concepts/agent-loop]]（試行間の改善ループ＝ループの入れ子）, [[concepts/agent-evaluation]]（自己テストの FP/FN 非対称・汚染対策ベンチ設計）, [[overview]], [[index]]
- メモ:
  - クリップ不良 (1): Figure 2 のキャプション（"(a) Diagram of Reflexion. (b) Reflexion reinforcement algorithm"）と **Algorithm 1 の擬似コード本体が欠落** → ar5iv から復元し、翻訳にコードブロックとして収録。
  - クリップ不良 (2): 多パネル図の後続パネル欠落（Figure 3 の x4、Figure 4 の x6・x7）→ ar5iv の HTML 構造（S4.F3.1/F3.2、S4.F4.1〜F4.3）でパネル帰属を確認のうえ全 8 枚を取得・配置。
  - クリップ不良 (3): §2 の関連研究比較表 2 つの multicolumn ヘッダ崩れ → 正規化。
  - Figure 5・7 は ar5iv でも画像なしのテキスト枠図（trajectory 例）で、クリップに本文として残存（正常）。
  - 付録 A〜D 全訳。trajectory・自己反省文・Actor/Self-Reflection の指示文は skill 規定により原文のまま収録。references 除外（謝辞なし）。
  - 概念ページは self-reflection を新規作成（スキーマ §1 で Reflexion の帰属先として規定済みのため確認なしで実施）。これで CoT → ReAct → Reflexion の単一エージェント基本系譜が根拠付きで完結。

## [2026-07-26] ingest | DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning

- 取り込み: `raw/papers/DeepSeek-R1_ Incentivizing Reasoning Capability in LLMs via Reinforcement Learning.md`（ar5iv → Obsidian Web Clipper、arXiv:2501.12948, DeepSeek-AI, 2025）
- 作成: [[summaries/2025-deepseek-r1]], [[translations/2025-deepseek-r1]], [[concepts/reinforcement-learning-from-human-feedback]], `raw/assets/2025-deepseek-r1/`（x1, plot_aime_with_maj, plot_length の 3 枚）
- 更新: [[concepts/reasoning-and-planning]]（「推論モデル — プロンプトから報酬へ」節を追加）, [[concepts/self-reflection]]（「内生する反省」節を追加）, [[concepts/agent-evaluation]]（推論モデルの評価プロトコル・reward hacking）, [[overview]], [[index]]
- メモ:
  - クリップは良好。欠落は脚注 1〜3（Aider / Codeforces / CNMO の URL）のみで ar5iv から復元。HTML テーブル（Table 2/4/5/6）の multicolumn ヘッダと空列を正規化。
  - Appendix A（Contributions and Acknowledgments）は著者名一覧と謝辞のため、スキーマの除外規定（acknowledgments）に準じて翻訳対象外とした（訳注に明記）。
  - R1-Zero テンプレート・aha moment の応答例は skill 規定により原文のまま収録。GRPO の目的関数・advantage・pass@1 の式は LaTeX 維持。
  - 概念ページは reinforcement-learning-from-human-feedback のみ新規（ユーザー選択。test-time-compute は s1 等の専用原典を待って dangling のまま。思考長の創発は RLHF ページ内に記述）。カバレッジ表の「LLM 基盤」軸が初めて埋まった。

## [2026-07-26] ingest | Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks

- 取り込み: `raw/papers/Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks.md`（ar5iv → Obsidian Web Clipper、arXiv:2005.11401, NeurIPS 2020, Lewis et al.）
- 作成: [[summaries/2020-rag]], [[translations/2020-rag]], [[concepts/retrieval-augmented-generation]], `raw/assets/2020-rag/`（x1〜x3, annotation_interface の 4 枚）
- 更新: [[concepts/tool-use-and-function-calling]]（RAG と ReAct の「同じ目的、逆の層」対比）, [[concepts/reasoning-and-planning]]（内部/外部知識の分業の先行定式化）, [[overview]], [[index]]
- メモ:
  - クリップは良好。欠落は脚注 1〜3（コード公開・Fairseq・HF Transformers の URL）のみで ar5iv から復元。HTML テーブル（Table 1/2/6）の rowspan/colspan 崩れと Table 1 の TriviaQA「- /」2 列表記を正規化（標準 test / Wiki test の 2 列に展開）。
  - Table 7 の転記を raw と照合し、WebQuestions dev の誤記（361→362）を修正のうえ確定。
  - 付録 A〜I 全訳。生成例（Table 3）は skill 規定により原文のまま。周辺化・DPR の数式は LaTeX 維持。references・acknowledgments 除外。
  - 概念ページは retrieval-augmented-generation を新規作成（スキーマ §1 規定のスラグのため確認なしで実施）。訓練時組み込み型（原典）と推論時注入型（現代の主流）の 2 層を分けて記述した。
  - **これでカバレッジ表の 6 軸すべてに原典が入った**（知識の接続が最後の空白だった）。overview の表下の注記を「全軸充足」に更新。

## [2026-07-26] ingest | How we built our multi-agent research system（Anthropic）

- 取り込み: `raw/articles/How we built our multi-agent research system.md`（Anthropic Engineering Blog, 2025 年 6 月 → Obsidian Web Clipper）
- 作成: [[summaries/2025-multi-agent-research-system]], [[translations/2025-multi-agent-research-system]], `raw/assets/2025-multi-agent-research-system/`（fig1 アーキテクチャ図・fig2 プロセス図・fig3 Clio プロットの 3 枚）
- 更新: [[concepts/multi-agent-systems]]（「(d) 本番の orchestrator-worker」節を新設。トークン経済学・同期実行・成果物のファイル永続化）, [[concepts/agent-evaluation]]（20 クエリからの小規模評価・終了状態評価・単一ジャッジ・人間テスターの役割）, [[concepts/tool-use-and-function-calling]]（MCP ツール説明のばらつき・ツールテスト用エージェント −40%・並列ツール呼び出し −90%）, [[concepts/retrieval-augmented-generation]]（静的検索 vs 動的多段検索の対比）, [[overview]], [[index]]
- メモ:
  - クリップは良好（ケース C だが本文・画像 3 枚とも欠落なし）。CDN 変換 URL（`/_next/image?url=...`）を素の www-cdn PNG に展開して取得。除外した chrome は記事末尾の Anthropic Academy 誘導バナー（パズルピース SVG リンク）のみ。
  - 謝辞（Acknowledgements）はスキーマの除外規定に準じて翻訳対象外。references 相当なし。
  - 概念ページの新規作成はなし（既存 4 概念の更新で吸収。orchestrator-worker は multi-agent-systems の規定スラグ内）。
  - Building Effective Agents（2024, 一般論）→ 本記事（2025, 本番実践）という同社内の系譜、および MASFT の失敗分類（研究側）との対応を summaries に明記した。

## [2026-07-26] ingest | MemGPT: Towards LLMs as Operating Systems

- 取り込み: `raw/papers/MemGPT_ Towards LLMs as Operating Systems.md`（ar5iv → Obsidian Web Clipper、arXiv:2310.08560, ICML 2024, Packer et al., UC Berkeley）
- 作成: [[summaries/2023-memgpt]], [[translations/2023-memgpt]], [[concepts/agent-memory]], [[concepts/context-engineering]], `raw/assets/2023-memgpt/`（x1〜x8 の 8 枚）
- 更新: [[concepts/tool-use-and-function-calling]]（自分のコンテキストに作用する「内向きのツール」・function chaining）, [[concepts/retrieval-augmented-generation]]（ページング反復による top-K 上限の突破）, [[concepts/agent-loop]]（イベント駆動トリガと heartbeat/yield の節を新設）, [[overview]], [[index]]
- メモ:
  - クリップは概ね良好（図 8 枚・表 3 点・脚注なし）。不良は Figure 8 キャプションの矢印連鎖の欠落のみで、ar5iv から復元（`831..ea5 → 5b8..4c3 → f37...617` と括弧内の `f37...617`）。数式まわりの `→ \rightarrow` 併記・`∼` 化けを正規化、Table 1 の HTML テーブルを Markdown 化。
  - 「provides function calls that the LLM processor to manage」「as show in Figure 1」等は ar5iv も同一＝原典由来の誤植と確認し、文意を汲んで訳出（訳注なし・機械修正なし）。
  - 付録 §6.1 の全プロンプト（DMR ペルソナ・LLM judge ×2・self-instruct 生成・文書分析・KV）は skill 規定により原文のままコードブロック収録。References 除外（謝辞セクションなし）。
  - 概念ページは agent-memory（スキーマ §1 規定の MemGPT 帰属先）に加え、ユーザー承認のうえ context-engineering も新規作成（MemGPT＋Anthropic Research の 2 原典を主根拠に、memory=永続化／context engineering=積載の棲み分けを明示）。これで未作成スラグは 11 個に減少。
  - agent-memory / context-engineering の作成に伴い、multi-agent-systems・self-reflection・retrieval-augmented-generation・agent-loop の「（未作成）」注記を除去。

## [2026-07-26] ingest | Reasoning Models Don’t Always Say What They Think

- 取り込み: `raw/papers/Reasoning Models Don’t Always Say What They Think.md`（ar5iv → Obsidian Web Clipper、arXiv:2505.05410, Anthropic Alignment Science, Chen et al., 2025）
- 作成: [[summaries/2025-cot-faithfulness]], [[translations/2025-cot-faithfulness]], [[concepts/agent-safety-and-guardrails]], `raw/assets/2025-cot-faithfulness/`（図 7 枚、元アセット名保持）
- 更新: [[concepts/reasoning-and-planning]]（「書かれた思考は信じられるか」節を新設）, [[concepts/reinforcement-learning-from-human-feedback]]（reward hacking は CoT に現れない）, [[concepts/agent-evaluation]]（トレース中の thought の信頼性・行動ベースの推論推定）, [[overview]], [[index]]
- メモ:
  - クリップは概ね良好（図 7 枚とも取り込みあり）。欠落は脚注 2 件の本文のみ——著者行の `*`（連絡先）・`+`（Anthropic 在籍時の仕事）と本文脚注 2（o1/o3 は CoT 非公開のため対象外）——で ar5iv から復元。Table 1 の HTML（rowspan）を Markdown に正規化。
  - References・Author Contributions・Acknowledgements は除外規定に準じて翻訳対象外（Author Contributions は謝辞相当と判断、訳注に明記）。忠実性スコアの定義式は LaTeX 維持、ヒント例文は原文のまま収録。
  - 概念ページは agent-safety-and-guardrails を新規作成（ユーザー承認。スキーマ §1 規定スラグ）。脅威モデルと対策 4 層（行動空間・ガードレール・監視・HITL）の骨組みを置き、CoT モニタリングを本論文根拠で詳述。prompt injection 等は概説（専用原典待ち）。
  - overview 第 5 軸「評価・運用・安全性」に安全性の原典が初めて入った。未作成スラグは 10 個に減少。

## [2026-07-26] ingest | A-Mem: Agentic Memory for LLM Agents

- 取り込み: `raw/papers/A-Mem_ Agentic Memory for LLM Agents.md`（ar5iv → Obsidian Web Clipper、arXiv:2502.12110, Rutgers / Ant Group, Xu et al., 2025）
- 作成: [[summaries/2025-a-mem]], [[translations/2025-a-mem]], `raw/assets/2025-a-mem/`（x1〜x18 の 18 枚）
- 更新: [[concepts/agent-memory]]（「A-Mem — 記憶の組織化と進化の自己管理」節を追加。実装層 3 の記述と忘却・監査／読み出しの論点も更新）, [[concepts/retrieval-augmented-generation]]（「agentic RAG と agentic memory の境界」を追加）, [[overview]], [[index]]
- メモ:
  - クリップ不良: **多パネル図の後続パネルと主キャプションの欠落**（Figure 1 は (a) のみ・(b) と主キャプション欠落、Figure 3 は (a) のみで (b)〜(e) 欠落、Figure 4 は (a) のみ、Figure 5 は (a) のみで (b)〜(h) 欠落）。ar5iv と照合して x1〜x18 の全 18 枚と主キャプション 4 件を復元。§4.2 の脚注 1・2（Ollama / LiteLLM の GitHub URL）も復元。
  - 付録 C のプロンプトテンプレート 3 種と Q/A 例は原典では SVG ボックス描画のため、SVG 内テキストをコードブロックとして原文のまま起こした（訳注に明記）。
  - "Methodolodgy" / "Empricial" / "constrctd" は ar5iv も同一＝原典由来の誤植と確認し、文意を汲んで訳出。
  - 表は 5 点（Table 1〜5）とも全数値を Markdown 化して収録（Table 1/3/4 の rowspan/multicolumn 構造を平坦化。数値は raw の HTML テーブルからスクリプトで転記し取り違えを防止）。
  - 概念ページの新規作成なし（A-Mem はスキーマ §1 の landmark 規約により [[agent-memory]] 内の代表手法として記述）。

## [2026-07-26] ingest | Towards Reasoning Era: A Survey of Long Chain-of-Thought

- 取り込み: `raw/papers/Towards Reasoning Era_ A Survey of Long Chain-of-Thought for Reasoning Large Language Models.md`（ar5iv → Obsidian Web Clipper、arXiv:2503.09567, HIT/中南大学ほか, Chen et al., 2025。813 文献・本文約 19 万字の大型サーベイ＝過去最大の ingest）
- 作成: [[summaries/2025-long-cot-survey]], [[translations/2025-long-cot-survey]], [[concepts/test-time-compute]], `raw/assets/2025-long-cot-survey/`（x2〜x12 の 11 枚）
- 更新: [[concepts/reasoning-and-planning]]（「Long CoT — 3 能力の統合としての推論モデル」節を新設）, [[concepts/self-reflection]]（「Long CoT への統合 — feasible reflection」節と aha moment への反証を追加）, [[concepts/reinforcement-learning-from-human-feedback]]（結果監督の理論的補強・SFT=記憶/RL=汎化）, [[overview]], [[index]]
- メモ:
  - クリップは良好: コンテンツ図 11 枚（Figure 1, 2, 4〜12）すべて取り込みあり。ar5iv の x1 はプロジェクトロゴのため保存対象外（chrome 扱い）。脚注 1（ロゴ＝Snake Puppy の説明）を ar5iv から復元。
  - Figure 3（分類法）は原典が LaTeX forest の文字ツリーで、クリップにソースコードが流出 → 構造を保った入れ子リストとして訳出（手法名は原文のまま）。「Key Difference」×4・「Takeaways」×6 の SVG ボックスはテキストを引用ブロックとして復元（Takeaways の一部は原典 SVG 内で文が切れており、その旨を訳注に明記）。
  - Table 1〜7 は rowspan/colspan を展開する自作パーサで全数値を Markdown 化（セクション区切り行は太字行として保持）。本文中の [^N] 引用番号は原文のまま保持（参考文献一覧 813 件は除外規定に準拠）。
  - 概念ページは test-time-compute を新規作成（ユーザー承認。スキーマ §1 規定スラグ）。垂直/並列スケーリング・推論境界・overthinking・計算配分の内在化・マルチエージェント＝並列スケーリングの見方を整理。未作成スラグは 9 個に減少。
  - reasoning-and-planning の「self-reflection（未作成）」という古い注記を発見し修正（Reflexion ingest 時の消し忘れ）。

## [2026-07-28] ingest | From GPT2 to Kimi3, Explained

- 取り込み: `raw/articles/22580_ From GPT2 to Kimi3, Explained.md`（X 記事 → Obsidian Web Clipper、@waterloo_intern（ali）, 2026-07-28。タイトル先頭の「22580:」はユーザー指示により表題から除外）
- 作成: [[summaries/2026-gpt2-to-kimi3]], [[translations/2026-gpt2-to-kimi3]], [[concepts/transformer-architecture]], [[concepts/llm-inference-optimization]], `raw/assets/2026-gpt2-to-kimi3/`（fig1〜fig22 の 22 枚、pbs.twimg.com から document 順で取得）
- 更新: [[concepts/agent-memory]]（「アーキテクチャ内部にも同型の問題がある」を設計論点に追加）, [[concepts/context-engineering]]（有限性の物理的実体＝KV cache を追記）, [[overview]], [[index]]
- メモ:
  - **X 記事はログイン壁のため原ページの curl 照合が不可**。本文は冒頭〜結論まで一貫しており、内部整合性チェックで欠落なしと判断（新条件として記録）。
  - 復元 2 件: §AttnRes の数式 2 箇所（X の数式レンダリングテキストと LaTeX ソースが連結 → LaTeX 抽出で正規化）、「Gated DeltaNet withM Mamba」の語割れ（文脈判断で "with Mamba"）。いずれも訳注に明記。
  - 画像は 22 枚（プラン時見積 20 → 実際 22）。冒頭 2 枚（アーキテクチャ総覧・系譜タイムライン）は内容確認のうえコンテンツ図として採用（chrome ではない）。原典の図にキャプションがないため、図番号と説明は直前本文に基づく訳注として付した。コードブロック 12 個は原文のまま（不規則インデントも原文由来）。
  - 概念ページは transformer-architecture と llm-inference-optimization の両方を新規作成（ユーザー承認）。overview 軸 6 のアーキテクチャ・推論効率の行が初めて埋まり、未作成スラグは 7 個に減少。

## [2026-07-28] ingest | LLM Optimization: Techniques and Guide（Mirantis）

- 取り込み: `raw/articles/LLM Optimization_ Techniques and Guide.md`（Mirantis Blog, 2026-07-14 → Obsidian Web Clipper。ベンダーブログ）
- 作成: [[summaries/2026-llm-optimization-guide]], [[translations/2026-llm-optimization-guide]]（本文に図なし・画像取得なし）
- 更新: [[concepts/llm-inference-optimization]]（モデル圧縮・バッチング/KV cache 管理・運用の規律の 3 節を追加/拡充）, [[overview]], [[index]]
- メモ:
  - クリップは良好。原ページ照合の結果、本文に欠落なし。原ページの画像はすべてサイト UI（ロゴ・アイコン）と装飾カバーバナーで、コンテンツ図はゼロ——クリップに含まれていたカバーバナー 1 枚は chrome として除外。
  - ベンダーブログのため、末尾の製品紹介（k0rdent AI）も本文として訳出しつつ、要約では宣伝的性格と引用値（Introl 等の転記）の未検証性を批判的視点として明記。
  - 概念ページの新規作成なし（llm-inference-optimization の 2 本目の原典として吸収）。

## [2026-07-28] schema-update | ingest 完了後のコミットを標準フロー化

- ユーザー指示により、ingest skill（`.claude/skills/ingest/SKILL.md`）の標準フローに手順 10「コミット」を追加。今後は検証完了後、ユーザーに確認せず `git add -A` → 既存フォーマットでのコミットまでを ingest の一部として実施する。

## [2026-07-28] ingest | Kimi K2: Open Agentic Intelligence

- 取り込み: `raw/papers/Kimi K2_ Open Agentic Intelligence.md`（ar5iv → Obsidian Web Clipper、arXiv:2507.20534, Kimi Team / Moonshot AI, 2025-07）
- 作成: [[summaries/2025-kimi-k2]], [[translations/2025-kimi-k2]], `raw/assets/2025-kimi-k2/`（x2〜x19 の 18 枚。x1 はタイトルロゴのため対象外）
- 更新: [[concepts/tool-use-and-function-calling]]（「ツール利用能力の製造 — 大規模データ合成」節を新設、function calling 実装の内側＝TypeScript 宣言・enforcer を追記）, [[concepts/reinforcement-learning-from-human-feedback]]（「Kimi K2 — 検証可能報酬と自己批評の統合、エージェント RL」節を新設）, [[concepts/transformer-architecture]]（スパース性スケーリング則・エージェント推論効率優先のヘッド数決定）, [[concepts/test-time-compute]]（予算制御・「非思考のエージェント SOTA」という対照点）, [[overview]]（軸 6 に「エージェント特化の基盤モデル訓練」の弾を追加）, [[index]]
- メモ:
  - クリップ不良: 多パネル図の欠落パターン——Figure 2 右（x4）・Figure 6 全体（x8, 本文参照のみで画像とキャプションが両方欠落）・Figure 8(b)（x11）・Figure 9(b)（x13）・Figure 13(b)(c)（x18/x19）の計 6 枚と主キャプション 4 件を ar5iv から復元。
  - 脚注 6 件（チェックポイント URL・LMSYS・MRCR・Promptfoo・lm-format-enforcer 等）の本文が欠落し、[^N] マーカーが末尾の参考文献と番号衝突していた → ar5iv から復元し [^fnN] として区別。Figure 1 キャプションの「3」は脚注マーカーの残骸と判断し統合。
  - Table 1・2 はクリップに Markdown で残存、Table 3〜6（ベンチマーク比較・安全性評価）は HTML から rowspan/colspan 展開パーサで全数値を転記。Algorithm 1（MuonClip）は構造を保ってコードブロック化。
  - Appendix A（Contributions＝著者一覧）は謝辞相当として訳出対象外（DeepSeek-R1 の前例に準拠）。Appendix B のツール呼び出しトークンテンプレートと TypeScript/JSON ツール定義は原文のまま収録。
  - 概念ページの新規作成なし（Kimi K2 はスキーマ §1 の landmark 規約により既存 4 概念内で記述）。

## [2026-07-28] ingest | Mixture of Experts Explained（Hugging Face）

- 取り込み: `raw/articles/Mixture of Experts Explained.md`（Hugging Face Blog, 2023-12-11 → Obsidian Web Clipper。Sanseviero, Tunstall, Schmid, Mangrulkar, Belkada, Cuenca）
- 作成: [[summaries/2023-moe-explained]], [[translations/2023-moe-explained]], `raw/assets/2023-moe-explained/`（00〜11 の 12 枚、元名保持）
- 更新: [[concepts/transformer-architecture]]（MoE 節に系譜・負荷分散 3 点セット・selective precision・専門化と FT の知見を追加）, [[concepts/llm-inference-optimization]]（「MoE のサービング — パラメータと FLOPs の分離」節を新設）, [[overview]], [[index]]
- メモ:
  - クリップ不良: **GShard の MoE Transformer Encoder 図（02_moe_block.png）が画像欠落**（キャプションのみ残存。HF の連番 00,01,03… から欠番を特定）→ 原ページから復元。数式 5 本の LaTeX が崩れていた（`\left(\right.` 混入）→ クリーンな LaTeX へ正規化（訳注明記）。05・11 の画像は初回 DL が失敗しリトライで取得。
  - 除外 chrome: 末尾の Citation（BibTeX）・関連記事誘導 2 件・Community。冒頭の第 2 版（2026-02）案内は原ページの編集注記として訳出。
  - 概念ページの新規作成なし（MoE はスキーマ規定どおり transformer-architecture 内で扱う）。2023 年時点の記事のため、shared experts・細粒度分割・スパース性スケーリング則（K2/K3 世代）とのギャップを要約の限界節に明記。

## [2026-07-28] ingest | Kimi K2.5: Visual Agentic Intelligence

- 取り込み: `raw/papers/Kimi K2.5_ Visual Agentic Intelligence.md`（ar5iv → Obsidian Web Clipper、arXiv:2602.02276, Kimi Team / Moonshot AI, 2026）
- 作成: [[summaries/2026-kimi-k2.5]], [[translations/2026-kimi-k2.5]], **[[concepts/computer-use-agents]]（新設・ユーザー確認済み）**, `raw/assets/2026-kimi-k2.5/`（figures/ 3 枚＋x2〜x10 の 9 枚 = 12 枚、元名保持）
- 更新: [[concepts/multi-agent-systems]]（類型 (e)「Agent Swarm — 並列化の意思決定を RL で学習する（PARL）」を新設）, [[concepts/context-engineering]]（「分割（sharding）— 溢れる前に分ける」節を新設）, [[concepts/reinforcement-learning-from-human-feedback]]（「Kimi K2.5 — 同時マルチモーダル RL・PARL・トークン効率」節を新設）, [[concepts/transformer-architecture]]（「マルチモーダル拡張 — 視覚をいつ・どう繋ぐか」節を新設: MoonViT-3D・early fusion・DEP）, [[concepts/agent-evaluation]]（agentic search / computer use ベンチ・スコアとトークン併記の報告様式）, [[concepts/test-time-compute]]（Toggle と length-overfitting）, [[overview]]（応用軸に初の専用記述＝computer use、構成とスケール軸に PARL）, [[index]]
- メモ:
  - クリップ不良: **Figure 7（Agent Swarm vs Discard-all の BrowseComp 比較, x5.png）が画像・キャプションごと欠落**（x4→x6 の欠番と Fig 6→Fig 8 の飛びで特定）→ ar5iv から復元し Figure 6 直後に配置。
  - 脚注 2 件（HF チェックポイント URL・WorldVQA GitHub URL）の本文欠落 → ar5iv から復元し [^fn1] [^fn2] として収録。付録 A の著者リスト脚注（アルファベット順・Tao Yu† = 香港大学）も訳注で反映。
  - Table 4（59 行の HTML 表）はパーサで markdown 化。クリップで失われていた**太字（グローバル SOTA 表示）も ar5iv から復元**。x1.png はタイトル横 21×21 ロゴのため対象外（K2 と同パターン）。
  - 付録 A の著者一覧は K2 では訳出対象外としたが、今回は「Kimi K2 / Kimi K2.5 自身が貢献者として記載」という特記事項があるためカンマ区切りに整形して収録。付録 E のシステムプロンプト（agentic search・computer use・Agent Swarm オーケストレータ）とツールスキーマ（create_subagent / assign_task）は原文のまま収録。
  - 概念ページ新設 1 件: computer-use-agents（CLAUDE.md §1 の想定スラグ。K2.5 の computer use 節＋E.7 を初出典に、Operator・Claude computer use・OSWorld/WebArena を整理）。

## [2026-07-28] ingest | Gemma 4 Technical Report

- 取り込み: `raw/papers/Gemma 4 Technical Report.pdf`（arXiv:2607.02770v2, Gemma Team / Google DeepMind, 2026-06-19。17 ページ・ケース B＝PDF）
- 作成: [[summaries/2026-gemma-4]], [[translations/2026-gemma-4]], `raw/assets/2026-gemma-4/`（fig1.png＝Figure 1 MTP drafter・fig2.png＝Figure 2 画像リサイズ。**ユーザー提供画像を raw/images/ から移動**）
- 更新: [[concepts/transformer-architecture]]（attention 系譜に「間引きと共有」＝5:1・values=keys・p-RoPE・per-layer embeddings、マルチモーダル拡張節に encoder-free の対置）, [[concepts/llm-inference-optimization]]（「投機的デコードとドラフタ」「オンデバイス推論 — QAT とエンコーダの軽量化」の 2 節を新設）, [[concepts/tool-use-and-function-calling]]（Gemma 4 の制御トークン書式を K2 enforcer の対例として追記）, [[concepts/test-time-compute]]（thinking のトグル化＝第三の形）, [[concepts/agent-evaluation]]（Arena / Elo 人間評価の項）, [[overview]]（軸 6 にエッジ／オンデバイス設計の弾）, [[index]]
- メモ:
  - PDF 原典のためクリップ不良なし。表 12 点（Table 1〜12）と Algorithm 1 を PDF から転記（Table 7 の CoVoST/FLEURS 二段表は 2 表に分割）。Table 11 の制御トークン・会話例・関数呼び出し例はコードブロック原文のまま収録。
  - References と Core contributors / Contributors（p.13-15 の著者一覧＝謝辞相当）は訳出対象外（K2・DeepSeek-R1 の前例に準拠）。
  - 画像はケース B の例外規定（ユーザー指示画像）を適用: raw/images/fig1.png・fig2.png → raw/assets/2026-gemma-4/ へ git mv で集約。
  - 概念ページの新規作成なし（Gemma 4 は landmark 規約により既存 5 概念内で記述）。K2.5（共有エンコーダ路線・フロンティア MoE）との対比を transformer-architecture と要約の両方に明記。

## [2026-07-29] ingest | DeepSeek-V4: Towards Highly Efficient Million-Token Context Intelligence

- 取り込み: `raw/papers/DeepSeek-V4_ Towards Highly Efficient Million-Token Context Intelligence.md`（ar5iv → Obsidian Web Clipper、arXiv:2606.19348, DeepSeek-AI, 2026。プレビュー版）
- 作成: [[summaries/2026-deepseek-v4]], [[translations/2026-deepseek-v4]], `raw/assets/2026-deepseek-v4/`（PNG 5 枚＋SVG 11 枚 = 16 枚、元名保持）
- 更新: [[concepts/transformer-architecture]]（attention 系譜に「圧縮してから選ぶ」＝CSA/HCA、残差ストリーム節に HC/mHC と Muon・安定化 2 技）, [[concepts/llm-inference-optimization]]（「100 万トークンの経済」節を新設: 混合精度 KV・on-disk KV cache 3 戦略・batch invariance/決定論・Quick Instruction・MegaMoE と C/B 提言）, [[concepts/reinforcement-learning-from-human-feedback]]（「DeepSeek-V4 — mixed RL を OPD に置換する」節を新設: スペシャリスト RL→全語彙逆 KL 蒸留・actor 兼 GRM・WAL 長さバイアス・DSec）, [[concepts/tool-use-and-function-calling]]（DSML＝XML 型書式。K2 enforcer・Gemma 4 制御トークンとの 3 方比較）, [[concepts/retrieval-augmented-generation]]（agentic search vs RAG の同一モデル実測——精度優位・コスト僅増）, [[concepts/test-time-compute]]（reasoning effort 3 段化）, [[concepts/context-engineering]]（「保持 — 窓が伸びたら捨てない選択肢」節: Interleaved Thinking）, [[concepts/agent-evaluation]]（内製 Codeforces Elo の手続き開示・開発者サーベイ・MCPAtlas/Toolathlon）, [[overview]]（LLM 基盤軸に 1M 効率と OPD の弾）, [[index]]
- メモ:
  - クリップ不良: **SVG 形式の図 11 枚が全欠落**（Figure 2〜6, 8〜12: basic_arch, CSA, HCA, mega_moe_pipeline, kv_cache, putnam×2, mrcr, dsv4_effort, winrate, scores。クリップに残存したのは PNG 5 枚のみ）→ ar5iv から SVG のまま取得して復元。
  - Table 3（Think Max 注入プロンプト）・Table 4（DSML ツールコールスキーマ）はクリップ内にインライン SVG のプロンプトボックスとして残存 → SVG 内の span テキストを抽出してコードブロック化（トークン区切り由来の空白を正規化。訳注明記）。
  - 脚注 4 件（HF inference dir・DeepGEMM PR・NVIDIA docs×2）の本文欠落 → [^fn1]〜[^fn4] で復元。
  - HTML 表 8 個（Table 1, 6, 7, 9, 11, 12, 13, 14）を rowspan 展開パーサで markdown 化（太字＝最良値を ar5iv から復元。次点の下線は割愛と訳注明記）。分断された表示数式・\penalty マクロをクリーン LaTeX に正規化。
  - Appendix A.1 の著者一覧（約 300 名）は謝辞相当として割愛（K2・R1 前例）。「* は離職者」の注記と A.2 謝辞は訳出。
  - 概念ページの新規作成なし（8 既存概念で吸収）。DSML が Anthropic 式ツールコール書式と同型である点・「原理未解明だが有効」と明記して安定化技術を公開する姿勢を要約で特記。

## [2026-07-29] schema-update | transformer-architecture の分割基準を設定

- lint での粒度点検を受けた判断: [[concepts/transformer-architecture]]（18KB, 6 テーマ）は現時点では**分割しない**（multi-agent-systems と同水準・attention/MoE/残差/オプティマイザは相互依存が強い）。
- 分割トリガーを設定: **「マルチモーダル拡張」節がマルチモーダル専業の原典 2 件以上を根拠に持った時点**で `multimodal-language-models`（仮）を新設して同節を移す。次点候補は「訓練の安定化・オプティマイザ」（Muon 系原典が増えた場合）。
- CLAUDE.md 本体の変更なし（スラグ新設はトリガー成立時に §7 の手順で実施）。

## [2026-07-29] lint | 陳腐化 3 件の修正・双方向リンク補修

- lint（20 原典・17 概念）の検出に基づく修正。孤立ページ 0・dangling は未実現スラグ 6 種のみ（被リンク数: agent-observability 6 / model-context-protocol 5 / coding-agents 3）。
- 陳腐化修正:
  - [[concepts/multi-agent-systems]]・[[overview]]: K2.5 Agent Swarm の BrowseComp 78.4「GPT-5.2 Pro 超え」に発表時点の限定を付し、V4 世代（単一エージェント 83.4〜85.9）との突き合わせ注記を追加——「並列化の同一モデル内利得」と「絶対値 SOTA」の区別を明文化。
  - [[concepts/test-time-compute]]・[[concepts/tool-use-and-function-calling]]: K2 の「非思考エージェント/ツール利用 SOTA」に「当時（2025）」の限定を追加。
  - [[concepts/computer-use-agents]]: 「主導的 CUA = Opus 4.5」に K2.5 評価時点の限定を追加。
- 双方向リンク補修: 古い要約 9 ページの「関連ページ」節に概念ページへの逆リンク計 18 件を追加（reflexion→agent-memory/reasoning-and-planning/test-time-compute、multi-agent-research-system→agent-memory/safety/llm-inference-optimization/test-time-compute 等）。
- [[concepts/agent-safety-and-guardrails]]: sandboxing 節に DSec（[[summaries/2026-deepseek-v4]]）を本番規模の実例として追記、CUA の攻撃面（[[computer-use-agents]]・K2.5 のパスワード平文プロンプト）の段落を新設。frontmatter に summaries/related を追加。
- 見送り: summary→concept の frontmatter 追加 25 件は傍系言及と判定し現状維持（`summaries` は根拠原典に限る規約）。context-engineering への 2026-llm-optimization-guide 追加も本文未参照のためスキップ。

## [2026-07-29] ingest | Switch Transformers

- 取り込み: `raw/papers/Switch Transformers_ Scaling to Trillion Parameter Models with Simple and Efficient Sparsity.md`（ar5iv → Obsidian Web Clipper、arXiv:2101.03961 / JMLR 2022, Fedus, Zoph, Shazeer / Google）
- 作成: [[summaries/2021-switch-transformers]], [[translations/2021-switch-transformers]], `raw/assets/2021-switch-transformers/`（x1〜x17 の 17 枚、元名保持）
- 更新: [[concepts/transformer-architecture]]（MoE 節に Switch の一次資料段落——top-1 の反通念・安定化レシピ 3 点・「不安定性は FLOPs/トークンに相関」・否定的結果 2 件）, [[concepts/llm-inference-optimization]]（expert parallelism の体系化（§5/図9）と蒸留の一次数値を出典化）, [[summaries/2023-moe-explained]]（関連ページに一次資料リンク）, [[overview]], [[index]]
- メモ:
  - クリップ不良: **多パネル図の 2 枚目が 4 枚欠落**（x2=Figure 1 右・x6=Figure 4 右・x9=Figure 6 右・x17=Figure 13 右。K2 と同じ脱落パターン）→ 17 枚全取得で復元。
  - **付録 F の Mesh TensorFlow 擬似コード 3 本（Figure 14/15/16）がコードブロックとして欠落**（行がバラけ ␣ 混入）→ ar5iv 埋め込みの base64 プレーンテキストからデコードして復元し、翻訳内の 3 ブロックと diff 照合で完全一致を確認（照合で発見した転記ミス 2 箇所を修正）。
  - 脚注 11 件の本文欠落（クリップは sup マーカーのみ・番号が途中で重複）→ ar5iv から復元し出現順に [^fn1]〜[^fn11] へ振り直し（訳注明記）。
  - Table 7（HTML 表）を markdown 化。Table 5 の 3 段組は 3 表に分割。
  - 概念ページの新規作成なし（Switch Transformers は landmark 規約により transformer-architecture 内で記述。既存の入門解説 2023-moe-explained の一次資料という位置づけ）。

## [2026-07-30] query | Gemma 4 の実効パラメータと per-layer embeddings

- 質問: E2B/E4B の総パラメータ 5B/8B に対して実効 2.3B/4.5B となる理由。
- 参照: [[summaries/2026-gemma-4]], [[translations/2026-gemma-4]]（表1 の内訳）, [[concepts/transformer-architecture]], [[summaries/2023-moe-explained]], [[summaries/2021-switch-transformers]]
- 作成: [[questions/gemma-4-effective-parameters]]（表1 の実効/非実効の分解表＋MoE との「分離」比較表）。index を更新。
- メモ: per-layer embeddings の機構詳細は Gemma 3n 由来だが同原典は未 ingest のため概説と明記（今後の ingest 候補）。

## [2026-07-31] ingest | DeepSeekMath

- 取り込み: `raw/papers/DeepSeekMath_ Pushing the Limits of Mathematical Reasoning in Open Language Models.md`（ar5iv → Obsidian Web Clipper、arXiv:2402.03300, Shao et al. / DeepSeek-AI, 2024-02）
- 作成: [[summaries/2024-deepseekmath]], [[translations/2024-deepseekmath]], `raw/assets/2024-deepseekmath/`（Math.png, corpus_comparisons.png, x1〜x5 の 7 枚、元名保持）
- 更新: [[concepts/reinforcement-learning-from-human-feedback]]（GRPO 節を一次資料化——PPO からの導出・統一パラダイム・「RL は Maj@K↑/Pass@K→」。R1 は「大規模実証」に位置づけ直し）, [[concepts/test-time-compute]]（検証器ボトルネック論に Pass@K 不変の実測を追加——RL と並列サンプリング＋検証は同じ利得を別の場所で買う）, [[concepts/reasoning-and-planning]]（「推論はどこから来るか」節: コード訓練の転移・PoT/ツール統合推論）, [[overview]]（post-training 行に GRPO の起点を追加）, [[index]]
- メモ:
  - クリップ不良: §1.1 の「Math Pre-Training at Scale」の貢献箇条書き 4 点が丸ごと欠落 → ar5iv から復元。脚注 7 件の本文欠落 → [^fn1]〜[^fn7] で復元。
  - **表 6 個（Table 1, 2, 5, 6, 7, 8）が ar5iv 自体のレンダリング段階で表構造を失い平文化**していた（クリップは忠実）——原ページに構造が存在しないため、平文のセル順序を保って markdown 表に再構成（数値は原文のまま・訳注明記）。Table 5 の「グレー＝32 候補多数決」の書式区別は平文化で失われており再現不可（訳注明記）。Table 3（HTML 表）は通常どおり正規化。
  - 画像 7 枚はすべてクリップに残存（多パネル欠落なし——今回は良好なケース）。
  - 概念ページの新規作成なし（GRPO はスキーマ規定どおり reinforcement-learning-from-human-feedback 内で扱う）。DeepSeek 推論系譜（Math 2024 → R1 2025 → V4 2026 OPD）が一次資料から現在まで貫通。

## [2026-07-31] ingest | Effective harnesses for long-running agents（Anthropic）

- 取り込み: `raw/articles/Effective harnesses for long-running agents.md`（Anthropic Engineering Blog, Justin Young, 2025 → Obsidian Web Clipper。ケース C）
- 作成: [[summaries/2025-effective-harnesses]], [[translations/2025-effective-harnesses]], **[[concepts/coding-agents]]（新設・ユーザー確認済み。lint ギャップ指摘（被リンク 3）の解消）**, `raw/assets/2025-effective-harnesses/puppeteer-testing.gif`
- 更新: [[concepts/context-engineering]]（「切り詰めに備える」を拡張——compaction の限界・構造化 artifact での引き継ぎ・JSON の改変耐性）, [[concepts/agent-frameworks]]（harness の語彙・Anthropic 実務 3 部作・「同一ハーネスでプロンプトだけ違う」= simplicity の実践）, [[overview]]（応用軸の coding-agents 行を実体化）, [[index]]（coding-agents を未作成スラグから昇格・リダイレクト追加）
- メモ:
  - クリップ復元: 脚注 1 件（「initializer/coding は初期プロンプトが違うだけで、システムプロンプト・ツール・ハーネスは同一」——重要な設計情報）を原ページから復元。GIF 1 枚を Next.js 変換 URL から素の CDN URL で取得。表タイトルの平文化をキャプションとして整形。
  - 除外 chrome: サイトロゴ SVG。Acknowledgements は謝辞として除外。
  - feature JSON・bearings 指示文・典型セッション例はコードブロック原文のまま収録。
  - coding-agents 新設により、agent-evaluation・tool-use-and-function-calling・overview からの旧 dangling link 3 箇所が実体ページへ解決。未作成スラグ残: llm-agents / model-context-protocol / web-agents / agent-observability / parameter-efficient-fine-tuning。

## [2026-07-31] ingest | Harness design for long-running application development（Anthropic Labs）

- 取り込み: `raw/articles/Harness design for long-running application development.md`（Anthropic Engineering Blog, Prithvi Rajasekaran / Labs, 2026 → Obsidian Web Clipper。ケース C。2025-effective-harnesses の直接の続編）
- 作成: [[summaries/2026-harness-design]], [[translations/2026-harness-design]], `raw/assets/2026-harness-design/`（PNG 6 枚、内容ベースの安定名）
- 更新: [[concepts/coding-agents]]（「generator/evaluator/planner — 分業とハーネスの縮小」節を新設）, [[concepts/agent-evaluation]]（judge の懐疑チューニング 4 手順・「基準の文言は出力を方向づける」）, [[concepts/context-engineering]]（context anxiety・reset vs compaction を基本制約に追記）, [[concepts/self-reflection]]（自己評価の甘さと「分離した懐疑」という実務解）, [[concepts/agent-frameworks]]（実務連作 4 本目・「部品＝モデル能力への仮定」の運用形）, [[overview]], [[index]]
- メモ:
  - クリップ不良: **画像 4 枚が欠落**（原ページ 6 枚中クリップ 2 枚のみ）——solo のスプライトエディタ／プレイ失敗・ハーネスのスプライトエディタ／AI レベル生成。原ページの Sanity データからキャプションと文書内位置を確定して復元。
  - 動画 2 本（美術館サイトデモ・DAW デモ）は規定どおり DL せず参照リンクとして訳注に記載。付録プラン例末尾の「...」は原文自体の省略と原ページで確認（クリップ切断ではない）。
  - QA フィードバック引用・evaluator の発見表・プラン例はコード/引用ブロック原文のまま収録。Acknowledgements は除外。
  - 概念ページの新規作成なし（前作で新設した coding-agents に主として吸収）。

## [2026-07-31] ingest | Meta-Harness: End-to-End Optimization of Model Harnesses

- 取り込み: `raw/papers/Meta-Harness_ End-to-End Optimization of Model Harnesses.md`（ar5iv クリップ, arXiv:2603.28052, Stanford/MIT/KRAFTON）
- 作成: [[summaries/2026-meta-harness]], [[translations/2026-meta-harness]]
- 更新: [[concepts/agent-frameworks]]（「ハーネスの自動探索 — 第三の道」節を新設）, [[concepts/coding-agents]]（「コーディングエージェントがハーネスを書く」節を新設）, [[concepts/context-engineering]]（ハーネス定義の明文化・自動探索と転移の設計論点）, [[concepts/agent-evaluation]]（探索セット過適合の監視・ハーネスのモデル間汎化）, [[overview]], [[index]]
- 画像: `raw/assets/2026-meta-harness/` に 6 枚（x1〜x5・val_vs_test_by_dataset.png, 元名保持）
- メモ: クリップ不良を ar5iv 照合で復元——(1) Figure 1 右パネル（x2.png）の欠落、(2) **Figure 3 の画像に Table 2 のキャプションが誤結合し、本物の Table 2（数値表）が丸ごと欠落**（図表取り違えの既知パターン）→ 表と Figure 3 キャプションを復元、(3) 脚注 2 件の本文欠落、(4) HTML 表 3 個（Table 7/8/9）の markdown 化＋太字復元、(5) SVG の要点ボックス 3 個とプロポーザー推論ログ引用 8 個をテキスト復元（英文ログは原文のまま＋訳を併記）、(6) Figure 5/6/8/9 は原ページでもインライン SVG フロー図（画像アセットなし）のためテキストのフロー図として再構成、(7) Algorithm 1 を擬似コードブロックに整形。Anthropic ハーネス連作（作る→剥がす）への「探索で発見する」という応答として位置づけ、agent-frameworks を接続の主軸にした。

## [2026-08-01] ingest | The Rise and Potential of Large Language Model Based Agents: A Survey

- 取り込み: `raw/papers/The Rise and Potential of Large Language Model Based Agents- A Survey.pdf`（Xi et al., Fudan NLP, arXiv:2309.07864v3, 86 ページ・686 文献）
- 作成: [[summaries/2023-llm-agents-survey]], [[translations/2023-llm-agents-survey]], [[concepts/llm-agents]]（**新設**・ユーザー承認済み。総論ハブ）
- 更新: [[concepts/multi-agent-systems]]（2023 年サーベイの協調/敵対分類・エージェント社会・幻覚増幅と誤合意の警告を起源として追記）, [[concepts/agent-safety-and-guardrails]]（§6.3 の脅威整理＝「行動空間を持つエージェントでは敵対的攻撃が破壊的行動になる」を起点として追記）, [[concepts/agent-evaluation]]（評価 4 観点 utility/sociability/values/継続進化を評価史の起点として追記）, [[overview]]（総論原典の位置づけ）, [[index]]（llm-agents を未作成リストから昇格）
- メモ: ケース B（PDF）。ar5iv は本論文の変換に失敗（プレースホルダのみ・図アセットなし）を確認したため**画像なし**で作成。図 12 点はキャプション全訳＋「訳注: PDF 原典のため画像省略」、分類ツリー図 5 点（Figure 3/4/5/6/11）は図中テキストをネスト箇条書きに転写（リーフの引用番号は省略と訳注明記）。脚注 6 件復元収録。翻訳は 7 チャンク分割で作成し切り詰めなし。要約では 2023 年時点の地図としての鮮度限界（プロンプト外装時代の前提・ハーネス論/MCP/computer use/RLVR 以前）を明示し、後続原典との対応を記載。

## [2026-08-01] ingest | Context Engineering for AI Agents: Lessons from Building Manus

- 取り込み: `raw/articles/Context Engineering for AI Agents_ Lessons from Building Manus.md`（Manus 公式ブログ, Yichao 'Peak' Ji, 2025-07）
- 作成: [[summaries/2025-manus-context-engineering]], [[translations/2025-manus-context-engineering]]
- 更新: [[concepts/context-engineering]]（KV cache 経済＝「積んだものを動かさない」規律・復元可能圧縮とファイルシステム外部化・復唱（recitation）節を新設・「履歴が意図せぬ few-shot になる」・「失敗の痕跡は消さない」節を新設）, [[concepts/agent-memory]]（「ファイルシステム＝究極のコンテキスト — Manus」節）, [[concepts/tool-use-and-function-calling]]（「削除するな、マスクせよ」＝ロジットマスク・prefill 3 モード・プレフィックス命名）, [[concepts/llm-inference-optimization]]（KV cache ヒット率をアプリ層の設計変数として追記）, [[overview]], [[index]]
- 画像: `raw/assets/2025-manus-context-engineering/` に 6 枚（原ページから取得, 4K PNG, 元 CDN 名保持）。アプリ DL バナーと「part of Meta」バナーは chrome として除外
- メモ: クリップ不良 1 件を復元——「Manipulate Attention Through Recitation」節の一文の冒頭「By constantly rewriting the todo list, Manus is」が本文から脱落し、リンク付きの句が frontmatter の author 欄に混入していた（新パターン: リンク句の author 欄化け）。原ページと照合して復元。著者名（Yichao 'Peak' Ji）も原ページから確定。

## [2026-08-01] ingest | Security and Privacy Challenges of Large Language Models: A Survey

- 取り込み: `raw/papers/Security and Privacy Challenges of Large Language Models_ A Survey.md`（ar5iv クリップ, Das/Amini/Wu, Florida International University, arXiv:2402.00888, 2024）
- 作成: [[summaries/2024-llm-security-privacy-survey]], [[translations/2024-llm-security-privacy-survey]]
- 更新: [[concepts/agent-safety-and-guardrails]]（脅威モデルの俯瞰を攻撃分類＝prompt injection/jailbreak/backdoor/poisoning/privacy へ大幅拡充・入出力ガードレール節に §6 の防御手法カタログを追加・攻撃側カタログの根拠原典として登録）, [[concepts/reinforcement-learning-from-human-feedback]]（ジェイルブレイクの 2 失敗モード＝competing objectives/mismatched generalization を安全訓練の構造的穴として追記）, [[overview]], [[index]]
- 画像: `raw/assets/2024-llm-security-privacy-survey/` に 7 枚（x1〜x7, ar5iv から取得, 元名保持）
- メモ: ケース A（ar5iv クリップ）。クリップは比較的健全。脚注 1 件（Figure 1 の Hugging Face 出典注記）を [^fn1] で復元。Table 1（略語 34 語）・Table 2（既存サーベイ 17 件との比較・✓/× マトリクス 12 列）を markdown 表として収録。図 2（脅威の集合ベン図）・図 7（魚骨図）は画像を保存しつつ図中テキストを訳注で補足。数式（敵対的サンプルの定式化）は LaTeX 維持。この wiki の agent-safety-and-guardrails に「攻撃側カタログ」を供給し、lint で挙がっていた prompt injection 一次原典の不足を埋めた。新概念ページは作らず既存 agent-safety-and-guardrails を受け皿とした。

## [2026-08-01] ingest | Toolformer: Language Models Can Teach Themselves to Use Tools

- 取り込み: `raw/papers/Toolformer_ Language Models Can Teach Themselves to Use Tools.md`（ar5iv クリップ, Schick et al., Meta AI Research, arXiv:2302.04761, NeurIPS 2023）
- 作成: [[summaries/2023-toolformer]], [[translations/2023-toolformer]]
- 更新: [[concepts/tool-use-and-function-calling]]（「Toolformer — ツール利用を重みに埋め込む（自己教師あり学習）」節を本格化＝従来は「原典未 ingest のため概説」だった。自己採点フィルタリング・GPT-3 超え・775M での創発・連鎖/対話の限界と agent 化の動機・RLVR/データ合成との類縁）, [[overview]]（ツール利用の系譜に Toolformer 追加・カバレッジ表）, [[index]]
- 画像: `raw/assets/2023-toolformer/` に 4 枚（x1〜x4, ar5iv から取得, 元名保持）
- メモ: ケース A（ar5iv クリップ）。本文中に `<sup>N</sup>` マーカーだけが残り本文が欠落していた**脚注 8 件を ar5iv から復元**して [^fnN] で収録（特殊トークンの実装注記・フィルタリングの設計理由・perplexity 評価が扱いにくい理由など、再現に効く情報）。Table 2/9 の HTML テーブルを markdown 化。付録 A.2・C のプロンプト（QA/計算機/WikiSearch/MT/カレンダー）は一字一句が挙動に効くため原文のまま収録し、計算機プロンプト 2 例目のクリップ崩れ（"This is 11.4Output:"）を原文に補正。Table 10（10 例の API 呼び出し一覧）は長いため代表 3 例のみ訳注つきで収録。tool-use-and-function-calling の Toolformer 記述が「概説のみ」だったのを本格記述に格上げし、ReAct（プロンプト時代）との「同じ目的・逆の層」の対比を明示。

## [2026-08-01] ingest | From LLM Reasoning to Autonomous AI Agents: A Comprehensive Review

- 取り込み: `raw/papers/From LLM Reasoning to Autonomous AI Agents_ A Comprehensive Review.md`（ar5iv クリップ, Ferrag et al., arXiv:2504.19678, 2025）
- 作成: [[summaries/2025-llm-reasoning-to-agents]], [[translations/2025-llm-reasoning-to-agents]], [[concepts/model-context-protocol]]（新設・ユーザー承認済み）
- 更新: [[concepts/agent-evaluation]]（約 60 ベンチマークの 8 分類・Agent-as-a-Judge を追記）, [[concepts/multi-agent-systems]]（MAST 失敗分類の再確認 = 本サーベイは Pan et al. として引用、実体は [[summaries/2025-masft]] Cemri et al. と同一）, [[concepts/retrieval-augmented-generation]]（Agentic RAG・Table VI の 5 段階対比・§V-E ReSearch を追記）, [[concepts/agent-frameworks]]（フレームワーク 4 類型 = 汎用オーケストレーション／データ・RAG 指向／マルチエージェント協調／軽量・低抽象、とプロトコル層の分離を追記）, [[concepts/tool-use-and-function-calling]]（MCP リンクの「未作成」注記を解消）, [[overview]]（知識の接続軸にプロトコル層 MCP/A2A/ACP を追記・カバレッジ表）, [[index]]
- 画像: `raw/assets/2025-llm-reasoning-to-agents/` に 6 枚（x1=Fig1, x3=Fig6, x4=Fig7, x5=Fig13, agentic.drawio.jpg=Fig4, RAG_agentic.drawio.jpg=Fig5, ar5iv から取得・元名保持）
- メモ: ケース A（ar5iv クリップ）。翻訳は Abstract〜VI の全章（929 行）。画像アセットのある図 6 枚（x1=Fig1・agentic.drawio=Fig4・RAG_agentic.drawio=Fig5・x3=Fig6・x4=Fig7・x5=Fig13）を `<figure>` 収録、SVG のみで画像の無い図（Fig 2,3,8〜12）はテキスト転写＋キャプション訳。平坦化していた Table V/VI/VII/VIII/IX/X/XI/XII を markdown 表に再構成。IV-C 本文の `<sup>1〜3</sup>`（BeeAI・MCP サーバカタログ・A2A の URL 脚注）はクリップに本文が残らずマーカーのみ保持。**IV-B 応用（11 領域）は当初のサブエージェント翻訳が各領域先頭のみ訳し多数を「以下略」で欠落させたが、最終的に全 11 領域を各研究レベルで訳出しドメイン別網羅表（表VIII〜XI 等）を併載する形で完成させた。** プロトコル層は MCP（Anthropic, 縦）／A2A（Google, 横）／ACP（IBM, ローカル）の三つ巴として concepts/model-context-protocol に新設し、§V-F のプロトコルセキュリティも収録。**注記: 本サーベイのサイバーセキュリティ節（OCCULT/AgentHarm/CyberMetric 等の攻撃系ベンチマーク言及）が内容スキャナに誤検知されたが、学術論文の正当な内容としてユーザー承認のうえ収録・コミット。** また、環境不安定により一部の Write/Edit が無反映になった事故があり、翻訳の IV 章以降・要約・concepts/model-context-protocol・index/overview/log 等を復旧して整合を取り直した。

## [2026-08-01] ingest | AI Agent Orchestration: A Guide for Enterprise Systems

- 取り込み: `raw/articles/AI Agent Orchestration_ A Guide for Enterprise Systems.md`（Databricks Blog, 2026-07-20, ベンダーブログ）
- 作成: [[summaries/2026-agent-orchestration-guide]], [[translations/2026-agent-orchestration-guide]], [[concepts/agent-observability]]（新設・ユーザー承認済み）
- 更新: [[concepts/multi-agent-systems]]（「デプロイ形態の類型 — 実務側の分類」節を新設＝6 パターン比較表と研究側類型 (a)〜(e) との対応づけ、および運用側で決まること＝暴走コスト上限・ウォッチドッグ・モデル多様化・入出力契約・rainbow deployment）, [[concepts/agent-safety-and-guardrails]]（HITL 節にエンタープライズ運用の定式化＝承認ゲートの発火条件・承認者側のロールベースアクセス・エスカレーション経路・監査証跡・policy-as-code を追記。原典が prompt injection に触れていない点も明記）, [[concepts/agent-frameworks]]（オーケストレーション・プラットフォームの評価軸 4 点とベンダーロックインを追記）, [[concepts/agent-evaluation]]・[[concepts/llm-agents]]（agent-observability の「未作成」注記を解消）, [[overview]]（評価・運用・安全性軸に可観測性を追記・カバレッジ表）, [[index]]
- 画像: なし。原ページを `curl` で取得して確認したところ、本文中に図版は 1 枚も存在せず（`<img>` はロゴ・資料サムネイル・製品アイコン等のサイト UI のみ）、Web Clipper の取りこぼしではないことを確認した。
- メモ: ケース C（非 arXiv の Web 記事）。**ベンダーの SEO 記事**であり、"ai agent orchestration platform" 等のキーワードの不自然な反復、同じ主張を言い換えただけの重複段落、全見出しが動名詞で統一されたテンプレート構造が見られる。翻訳では圧縮せず原文構成のまま訳出した。本文に混入していた資料 DL のプロモ枠（"REPORT / The agentic AI playbook for the enterprise"）は本文でないため除外。自社製品（Agent Bricks・Unity Catalog・AI Gateway・MLflow）への誘導リンクはリンクテキストのみ訳出し URL は不収録。**記事が掲げる 3 つの統計値（マルチエージェントは 35% 速い／専門エージェントで 30% 効率向上／75% の企業がハイブリッド採用）はいずれも出典が示されておらず**、要約に専用の注意節を設けて「ベンダー公称値」と明示し、概念ページには一切持ち込まなかった。研究側原典との緊張（[[summaries/2025-masft]] の 14 失敗モード、[[summaries/2025-multi-agent-research-system]] のトークン 15 倍、Fugu で単発ルーティングがワークフロー型を上回った事例）も要約の「限界・批判的視点」に記載。concepts/agent-observability の新設により、既存 8 箇所の dangling link（overview・llm-agents・agent-frameworks・agent-evaluation・agent-safety-and-guardrails・2025-cot-faithfulness・2022-chain-of-thought）が解決した。可観測性を主題とする一次資料は未取り込みで、次に読むべき候補（OpenTelemetry の生成 AI 向けセマンティック規約、トレーシング基盤の設計文書、本番インシデントの事後報告）をデータギャップとして概念ページと overview に明記した。

## [2026-08-01] ingest | Scaling Managed Agents: Decoupling the brain from the hands

- 取り込み: `raw/articles/Scaling Managed Agents_ Decoupling the brain from the hands.md`（Anthropic Engineering Blog, 2026-04-08, Lance Martin / Gabe Cemaj / Michael Cohen）
- 作成: [[summaries/2026-managed-agents]], [[translations/2026-managed-agents]]
- 更新: [[concepts/agent-frameworks]]（**主な受け皿**。`### ランタイムの分離 — brain / session / hands` 節を新設＝3 インターフェース・pets vs cattle・OS 仮想化の類比・TTFT・**meta-harness の語義対置表**。連作を 4 本立て→5 本立てに更新。関連ページの model-context-protocol「未作成」注記も解消）, [[concepts/context-engineering]]（`### session はコンテキストウィンドウではない — 保存と積載を分ける` 節を新設＝不可逆な取捨選択の問題・`getEvents()` の位置指定スライス・保存と積載の関心分離・Recursive Language Models への言及）, [[concepts/agent-safety-and-guardrails]]（層(1) に「認証情報を『狭める』のではなく『届かなくする』」を追記＝prompt injection→トークン窃取→無制限セッション生成の連鎖、narrow scoping が仮定を符号化する問題、Git 配線と vault＋プロキシの 2 パターン）, [[concepts/agent-observability]]（「アーキテクチャが可観測性を規定する」＝障害を局在化できなかった実例を第 5 の困難として追記。監査証跡層に「永続ログが実行の一次記録を兼ねる」を追記）, [[concepts/model-context-protocol]]（設計論点にプロキシ＋vault パターンを追加＝§V-F「標準化された認証機構の欠如」への具体的な実装解として接続）, [[concepts/multi-agent-systems]]（`### many brains, many hands` 節を新設＝hands が brain に結合しないので受け渡せること、モデル能力の向上がアーキテクチャ制約を反転させた経緯）, [[concepts/agent-loop]]（ループを session / harness / sandbox に分解する見方＝状態を外に置けばループ自体は落ちてよい）, [[summaries/2025-effective-harnesses]]（陳腐化していた「Anthropic 実務 3 部作」表記を全 5 本の連作に修正）, [[overview]], [[index]]
- 画像: `raw/assets/2026-managed-agents/` に 4 枚（`architecture-overview.png` / `interface-table.png` / `session-events.png` / `many-brains-many-hands.png`）。クリップには Next.js の `_next/image` プロキシ URL で埋まっていたため、内側の `www-cdn.anthropic.com` URL に展開して取得（全 PNG 検証済み）。原ページ照合の結果、**クリップの図の欠落はなし**（サイト UI の SVG ロゴのみ chrome として除外）。
- メモ: ケース C だが**クリップは良好**（本文・図とも欠落なし）。原典に図キャプションが無いため `<figcaption>` はすべて訳注として明示。**2 枚目の図は各コンポーネントのインターフェース仕様表**（Session / Orchestration / Harness / Sandbox / Resources / Tools の擬似コードと「何がそれを満たすか」）だったため、skill のケース C-6 に従い画像に加えて markdown 表としても転記した——散文に出てこない Orchestration・Resources・Tools の契約や `yield Effect<T> -> EffectResult<T>` 等のシグネチャはこの表にしかない。除外したのは冒頭の製品導線 1 行（"Get started with Claude Managed Agents..."）と Acknowledgements。自社記事へのリンクはリンクテキストのみ訳出し URL 不収録、学術引用（Recursive Language Models = arXiv:2512.24601、Sutton "The Bitter Lesson"）は識別子・書名を保持。
- 注意点の処理: (1) **「meta-harness」の語義衝突**——既存の [[summaries/2026-meta-harness]] は「ハーネスを探索で最適化する外側ループ」、本記事は「多様なハーネスを載せる汎用インターフェース層」で meta の向きが逆。agent-frameworks に比較表を置いて対置し、index の略称リダイレクトから `Meta-Harness` の一括ルーティングを外して両義に分割した。(2) **連作の番号**を 5 本立てに統一（agent-frameworks・overview・2025-effective-harnesses）。(3) **context anxiety 撤去時期**は本記事の記述（Opus 4.5 で消えていた）を原典どおり訳し、既存記述（4.5 で大幅減・4.6 で撤去）と矛盾しないことを要約に 1 文で注記するに留め、第 3 の記述を持ち込まなかった。
- データギャップ: Anthropic の **「Effective context engineering for AI agents」** は本記事と [[summaries/2026-harness-design]] の双方から参照されているが未 ingest で、[[concepts/context-engineering]] の根拠原典にも入っていない。次に読むべき候補。

## [2026-08-01] ingest | Illustrating Reinforcement Learning from Human Feedback (RLHF)

- 取り込み: `raw/articles/Illustrating Reinforcement Learning from Human Feedback (RLHF).md`（Hugging Face Blog, 2022-12-09, Nathan Lambert / Louis Castricato / Leandro von Werra / Alex Havrilla）
- 作成: [[summaries/2022-rlhf-illustrated]], [[translations/2022-rlhf-illustrated]]
- 更新: [[concepts/reinforcement-learning-from-human-feedback]]（**ページ全体を時系列に再構成**＝ユーザー選択。`## 代表手法` を廃し、`## 系譜 — 何が何を置き換えたか` の対応表＋年代別 6 節（2022 古典的 RLHF〔新設〕→ 2024 GRPO → 2025 RLVR/蒸留 → 2025 K2 joint RL → 2026 K2.5 → 2026 OPD）に組み替え。既存記述は保持したまま、各節に「前の世代の何を置き換えたか」の接続文を追加。設計論点に「絶対スコアより相対比較」「データ収集ループを持てるかが手法を決める」の 2 項を追加）, [[concepts/agent-evaluation]]（冒頭に「なぜ人間の判断を測るのか」の起点を追加＝BLEU/ROUGE の限界→人間の選好→LLM-as-a-judge の系譜。Arena の Elo が報酬モデル訓練の道具の転用であること、LLM-as-a-judge が RLHF のコスト問題と同型の解であることを明記）, [[concepts/agent-safety-and-guardrails]]（jailbreak の 2 失敗モードに出所を接続＝HHH 基準・hh-rlhf・レッドチーミングの系譜。competing objectives は helpful と harmless を単一スカラーに畳んだ帰結、mismatched generalization は選好データのカバレッジの帰結、と報酬設計まで降りて説明）, [[overview]]（post-training 軸の先頭に古典的 RLHF・評価軸に起点・カバレッジ表 2 行）, [[index]]
- 画像: `raw/assets/2022-rlhf-illustrated/` に 4 枚（`chatgpt-explains.png` / `pretraining.png` / `reward-model.png` / `rlhf.png`。`huggingface.co/datasets/huggingface/documentation-images` の直リンクから取得、元名保持、全 PNG 検証済み）
- メモ: ケース C（非 arXiv の Web 記事）だが**クリップは良好**。原ページを `curl` で再取得して照合したところ、本文・図とも欠落なし（`<img>` はサイトロゴを除き本文 4 枚のみで、クリップの 4 枚と一致）。除外したのは冒頭の他言語訳への案内行、末尾の Citation・BibTeX・謝辞、およびページ下部の読者コメント欄。**「Further reading」は各文献に一行の貢献説明が付いた注釈つき文献ガイド**（TAMER 2008 → Christiano 2017 → InstructGPT 2022 → Llama 2 2023 の系譜）であり、単なる参考文献一覧ではなく本文の一部と判断して訳出した（URL は不収録・arXiv 識別子のみ保持）。**1 枚目の図は ChatGPT 対話のスクリーンショット**なので skill のケース C-6 に従い、画像に加えて本文にもテキストとして起こした（対話ログの自然言語は原文のまま）。本文中の他の外部リンクは指す先がすべて技術資料で宣伝リンクを含まないため URL を残し、講義録画（YouTube）のみ要約側に参照として記載した。
- 陳腐化の扱い: **公開が 2022-12 で 3 年半前**のため、要約に専用節「この記事の賞味期限 — 2022 年 12 月から何が変わったか」を設け、記述ごとに現在との対応を表で整理した（PPO 一択→GRPO が critic を排除・DPO／人間の選好→RLVR／「大規模データセットは hh-rlhf のみ」→陳腐化／TRL・TRLX・RL4LMs の 3 本柱→TRL へ一本化／パラメータ凍結は PEFT 側の事情／Iterated Online RLHF→閉ループ critic 洗練／報酬モデルは別モデル→actor が兼ねる）。**OSS の現況は GitHub API で実測**: TRLX は 2024-01-08、RL4LMs は 2024-03-01 で最終コミットが止まり（アーカイブ宣言はなし）、TRL は 2026-08-01 当日にコミットあり・約 19,000 stars。逆に生き残っている骨格（比較から報酬を作る／スカラー化／KL 型の乖離ペナルティ／reward hacking の構造／絶対評価より相対比較）も明示した。概念ページには陳腐化した記述を持ち込まず、2022 年節の末尾に注記ブロックで要約側の表へ誘導している。
- 補足として収録: 原典のコメント欄で複数の読者が疑問を呈している「図4 直後の技術注記は正しいか（プロンプトを両モデルに与えて分布を比べるほうが自然では）」について、要約に短い解説を置いた——KL ペナルティは方策が**実際にサンプルしたトークン列の上で**推定されるので参照モデルにも同じ列の対数確率を計算させる必要があり、プロンプトだけを比べると最初の 1 トークン位置の差しか得られない。初学者が最も引っかかる箇所のため。
- データギャップ: **InstructGPT 論文（Ouyang et al. 2022, arXiv:2203.02155）そのものは依然として未 ingest**。本記事で古典的 RLHF の一次資料は入ったが、RLHF を確立した論文本体・**DPO**（Rafailov et al. 2023, arXiv:2305.18290）・Anthropic HH 論文（arXiv:2204.05862）はいずれも未取り込みで、概念ページでも「原典未 ingest」として扱っている。前回記録した Anthropic「Effective context engineering for AI agents」と並ぶ次点候補。

## [2026-08-02] ingest | Training language models to follow instructions with human feedback (InstructGPT)

- 取り込み: `raw/papers/Training language models to follow instructions with human feedback.md`（ar5iv クリップ, Ouyang et al., OpenAI, arXiv:2203.02155, NeurIPS 2022）
- 作成: [[summaries/2022-instructgpt]], [[translations/2022-instructgpt]]
- 更新: [[concepts/reinforcement-learning-from-human-feedback]]（**主な受け皿**。2022 節を一般向け解説ベースから**一次資料ベースに格上げ**＝SFT が必須の第 1 段であること・実データ量（SFT 12.7k / RM 33.2k / PPO 31k）・K=4〜9 のまとめ順位づけと 1 バッチ化（シャッフルすると 1 エポックで過学習）・6B RM を選んだ理由は安さでなく 175B RM の訓練不安定性・KL の参照は事前学習モデルでなく **SFT モデル**・環境はバンディットでエージェント能力は対象外。`### アラインメント税 — 整合のコストをどう測るか` を新設。限界に「改善は一様でない（バイアスは改善せず、敬意を指示するとむしろ偏りが上がる）」「指示されれば有害になる」を追加。設計論点に「ルーブリックの一文が出力の性格になる」「アラインメント税は消えたのか」「指示追従の能力はそのまま悪用の能力である」の 3 項を追加。冒頭の「InstructGPT 論文そのものは未 ingest」注記を解消）, [[concepts/agent-safety-and-guardrails]]（`## 整合はどこから来るのか — 誰の選好が埋め込まれているか` を新設＝安全性は事前学習の目的関数になく報酬モデル 1 個を通じて事後注入されること、competing objectives / mismatched generalization の由来、`### 誰の選好か — 整合の宛先を分解する` の 4 層。あわせて脅威モデル節の jailbreak 項に前日入れた説明が重複したため、新節への誘導に圧縮）, [[overview]]（post-training 軸の先頭を一次資料に差し替え・安全性軸に「整合の宛先」を追加・カバレッジ表 2 行）, [[index]]
- 画像: `raw/assets/2022-instructgpt/` に 7 枚（x1〜x7, ar5iv から取得・元名保持）。本文 §1〜5 の図はこれで全数。付録の図（`labelserver_likert.png` / `labelserver_ranking.png` / x8〜x22）は翻訳対象外のため取得していない。
- メモ: ケース A（ar5iv クリップ）。**翻訳範囲はユーザー指示により本文 §1〜5 のみ**とし、付録 A〜F・References・Acknowledgements は訳出していない（skill が認める例外ルート）。ただし付録にしかない事実——ラベラー選抜の 4 基準とソフト足切り（一致率 75%・デモンストレーション 6/7）、属性調査 19 名の内訳（東南アジア系 52.6%・18〜34 歳 73.7%・学士以上 89.4%）、Table 6 のデータセット内訳——は**要約と概念ページ側で「付録より」と明示したうえで引用**した。復元したクリップ不良: (1) 著者リストが "Amanda Askell &Peter" で途切れ **Peter Welinder・Paul Christiano・Jan Leike・Ryan Lowe が脱落**していたので原ページから復元（主著者マーク \* と所属注記 † も）。(2) `<sup>1</sup>`〜`<sup>8</sup>` のマーカーだけ残って本文が消えていた**脚注 8 件を復元**（[^fnN] で収録）。(3) Figure 8/9 の HTML 対比表を markdown 化（プロンプトとモデル出力は原文のまま）。**Table 4/5 は ar5iv にも存在せず**、原典の番号飛びでクリップ不良ではないことを確認。付録側では Figure 12 のキャプションとパネル (b)（`labelserver_ranking.png`）が欠落していたが、付録が訳出対象外のため復元は行わなかった。`[^N]`（数字のみ）は参考文献への引用マーカーであり脚注ではない旨を訳注に明記した（参考文献一覧は訳出対象外なのでリンクは解決しない）。
- 原典側の記述の揺れ: §3.2 と §4.1 の「Table 2」は用途カテゴリ分布の **Table 1** を指すとみられ、Figure 4 キャプションの「Appendix E.2」は **E.3** とみられる。いずれも ar5iv 原文でも同じであることを確認したうえで、訳文は原文どおりにして訳注を添えた。軽微な誤記（"evaluatoins"・"Satisifies"・"Most of the use-cases have are generative"・閉じ括弧の欠落）は訳文で意味の通る形にした。
- 前日のデータギャップの解消と残り: 2026-08-01 の log に記録した「InstructGPT 論文そのものが未 ingest」を**解消**。残る近接ギャップは **DPO**（Rafailov et al. 2023, arXiv:2305.18290）と **Anthropic HH 論文**（arXiv:2204.05862）で、いずれも概念ページ内では「原典未 ingest」として扱っている。Anthropic「Effective context engineering for AI agents」も未取り込みのまま。

## [2026-08-02] ingest | Direct Preference Optimization: Your Language Model is Secretly a Reward Model (DPO)

- 取り込み: `raw/papers/Direct Preference Optimization_ Your Language Model is Secretly a Reward Model.md`（ar5iv クリップ, Rafailov et al., Stanford, arXiv:2305.18290, NeurIPS 2023）
- 作成: [[summaries/2023-dpo]], [[translations/2023-dpo]]（704 行）
- 更新: [[concepts/reinforcement-learning-from-human-feedback]]（**主な受け皿**。系譜表に 2023 年の行を追加し、`## 2023 — 報酬モデルも RL ループも捨てる: DPO` 節を 2022 節と 2024 GRPO 節の間に新設＝3 手の導出・暗黙の報酬・同値クラスと補題 1/2 と定理 1・重要度重みが無いと退化すること・実測・§5.2 の PPO 分散の診断・「消えたのは報酬モデルの訓練とサンプリングループの 2 つで、policy と reference は残る」。GRPO 節の統一パラダイムから DPO へリンク。設計論点に「オフラインで足りるか、オンラインの探索が要るか」を追加）, [[concepts/agent-evaluation]]（LLM-as-a-judge 節に **DPO §6.4 をジャッジ検証の参照実装として追記**＝人間同士の一致率を基準線にする型、プロンプトの文言で勝率が 47%→54% 動くこと、「選好の代理 ≠ 事実性の判定者」を Table 10 の GPT-4 誤判定例で補強）, [[overview]]（post-training 軸に DPO・評価軸にジャッジ検証手続き・カバレッジ表）, [[index]]
- 画像: `raw/assets/2023-dpo/` に 8 枚（`teaser.png` / `x1`〜`x6` / `survey.png`、ar5iv から取得・元名保持）
- メモ: ケース A（ar5iv クリップ）。**翻訳は本文 §1〜7 に加えて付録 A〜D を全訳**（ユーザー選択）——付録 A は DPO の導出と証明そのもの（本文はその結果を引用しているだけ）、付録 C には GPT-4 ジャッジのプロンプト全文（S 版・C 版）が入るため。除外は References・Acknowledgements・Author Contributions（著者ごとの貢献記載＝クレジット）・付録 D.3 末尾のボランティア氏名一覧（謝辞相当）。復元したクリップ不良: (1) **本文の Figure 2・3・4 はいずれも左右 2 パネル構成だが、クリップには左（x1/x3/x5）しか残っておらず右パネル x2/x4/x6 が丸ごと欠落**していたため原ページから取得。(2) `<sup>1</sup>`〜`<sup>6</sup>` のマーカーだけ残って本文が消えていた**脚注 6 件を復元**（[^fnN] で収録）。(3) 著者の等貢献マーク欠けを復元。(4) Table 1 の HTML を markdown 化。(5) ar5iv が align 環境を 1 行ずつ独立した `$$\displaystyle ...$$` に分解していたため、ひとつながりの導出を `aligned` にまとめ直した。プロンプト・PyTorch コード・モデル出力サンプル（Table 3〜10）は原文のまま収録し、長い反復部分のみ […] で省略。
- **原典側の不整合を検出**: 付録 A.4 の式(21) は本文の式(7) と $y_w$／$y_l$ が入れ替わっており、そこから導かれる勾配の重み係数も本文の式(8) と符号が逆になる（本文 $\sigma(\hat r_l-\hat r_w)$ vs 付録 $\sigma(\hat r_w-\hat r_l)$）。ar5iv の LaTeX ソースでも同じであることを確認したうえで原文どおりに訳出し、該当箇所と冒頭の訳注に「実装上正しいのは本文側」と明記した。
- 前日までのデータギャップの解消: 2026-08-01 と 08-02 の log に記録した **DPO（arXiv:2305.18290）を解消**。これで RLHF 概念ページの年表は 2022 → 2023 → 2024 → 2025 → 2026 が一次資料で埋まった。残るのは **Anthropic HH 論文**（arXiv:2204.05862。本論文の Anthropic-HH データセットの出所であり、概念ページでも hh-rlhf として言及のみ）と、Anthropic「Effective context engineering for AI agents」。

## [2026-08-02] ingest | Effective context engineering for AI agents

- 取り込み: `raw/articles/Effective context engineering for AI agents.md`（Anthropic Engineering Blog, 2025-09-29, Applied AI チーム: Prithvi Rajasekaran / Ethan Dixon / Carly Ryan / Jeremy Hadfield）
- 作成: [[summaries/2025-effective-context-engineering]], [[translations/2025-effective-context-engineering]]
- 更新: [[concepts/context-engineering]]（**全体に統合**＝ユーザー選択。冒頭に命名・定式化の出典と指導原理を追加。基本制約の枠組みを **注意予算**で置き直し、項目 2 を lost in the middle から **context rot** へ拡張して機構（n² の対関係・訓練分布の偏り・位置内挿の劣化・「崖でなく性能勾配」）を収録＝lost in the middle の「原典未 ingest」注記は残しつつ一般形に一次資料が付いた。`### 各構成要素の絞り方 — システムプロンプトの「高度」とツールの数` と `### just-in-time 取得と段階的開示` の 2 節を新設、`### 長時間タスク — 3 手法の使い分け` を追加。圧縮節に compaction のチューニング手順（recall → precision）とツール結果消去を追記。設計論点に「事前取得か実行時取得か」「この記事の各論は剥がされる候補でもある」の 2 項）, [[concepts/retrieval-augmented-generation]]（設計論点に「事前に埋め込むか、実行時に引くか」＝ embedding-based pre-inference retrieval と JIT の対比、agentic search との力点の違い＝検索の agency vs 格納側の設計、ハイブリッドの境界は内容の動的さ）, [[concepts/agent-memory]]（`### 構造化されたノート取り — 最小のエージェント的記憶` 節を新設＝NOTES.md 程度の実装で足りること、Claude のポケモンプレイでの自発的な記憶構造の獲得、A-Mem との対比＝作り込む vs 場所だけ与える）, [[concepts/tool-use-and-function-calling]]（設計論点に「絞り込みの判定基準——人間で試す」＝人間のエンジニアが断言できないならエージェントにもできない、ツール＝情報／行動空間との契約）, [[concepts/multi-agent-systems]]（設計論点に「分業の目的はコンテキストの隔離である」＝数万トークン探索して **1,000〜2,000 トークンに蒸留して返す**比率、返却が長いなら隔離の利点が消えているという点検基準、「トークン 15 倍」への理由側の数字）, [[overview]], [[index]]
- 画像: `raw/assets/2025-effective-context-engineering/` に 2 枚（`prompt-vs-context-engineering.png` / `system-prompt-altitude.png`）。`_next/image` プロキシ URL を展開して `www-cdn.anthropic.com` から取得。
- メモ: ケース C（非 arXiv の Web 記事）だが**クリップは良好**。原ページを `curl` で取得し、**本文を文単位で突き合わせて欠落なしを確認**した。本文図も 2 枚とも残っており（原ページの 3 枚目は記事見出し部の装飾 SVG なので chrome として除外）、ケース C で通例の図の取りこぼしは今回発生していない。復元したのは 3 点: (1) frontmatter の `author:` と `published:` が空だったため**著者 4 名と公開日 2025-09-29 を原ページから補充**、(2) 語連結 `needle-in-a-haystackstyle` を原文どおり `needle-in-a-haystack style` に、(3) 著者情報に伴う所属・チーム名。**2 枚目の図はプロンプト例のスクリーンショット**なので skill のケース C-6 に従い、画像に加えて 3 段階（Too specific / Just right / Too vague）の**プロンプト全文を原文のままテキストにも転記**した——「適切な高度」は文章で説明されるより実例で示されるほうが伝わるため。除外は末尾の Acknowledgements と結論末の製品導線 1 行。**原典側のリンク切れも検出**: 「Sonnet 4.5 launch」のリンク先がこの記事自身の URL になっている（原ページで確認済み・クリップ不良ではない）ため、訳文ではリンクテキストのみ残して訳注に明記した。
- wiki 上の位置づけ: この記事は **[[concepts/context-engineering]] の主題そのものを命名した出典**でありながら、同ページの根拠原典 12 件に入っていなかった（[[summaries/2026-harness-design]] と [[summaries/2026-managed-agents]] の双方が参照している）。取り込み前の検索では `context rot` / `attention budget` / `just-in-time` / `progressive disclosure` / `structured note-taking` / `right altitude` がいずれも wiki 内ヒット 0 件で、`compaction` のみハーネス連作経由で実践として存在していた（原理からの説明はなし）。
- 限界の扱い: **定量的な結果が一切ない記事**（ベンチマークもアブレーションも A/B もなし。具体的な数字はサブエージェントの 1,000〜2,000 トークン・直近 5 ファイル・ポケモンの 1,234 ステップのみで、いずれも設計値か逸話）である点を要約の冒頭近くと index に明記し、概念ページ側でも「Anthropic の実務経験」を超えて扱わないようにした。また記事自身が「より賢いモデルほど規定的なエンジニアリングを必要としなくなる」と書いており、各論が [[summaries/2026-harness-design]] の「部品＝モデル能力への仮定」の枠組みで剥がされる候補である点を設計論点に入れた。compaction の危険性の扱いが [[summaries/2025-manus-context-engineering]] の「復元可能な圧縮」より甘い点も要約に記載。
- データギャップ: 3 セッション追跡していた本記事の未 ingest を**解消**。残るのは **Anthropic HH 論文**（arXiv:2204.05862）と、本記事が引く **context rot の研究（Chroma）**・**lost in the middle（Liu et al. 2023）**の 2 件の一次資料。後者 2 件は [[concepts/context-engineering]] の基本制約の根拠として概説のみで置いている。

## [2026-08-02] ingest | A Comprehensive Survey of Mixture-of-Experts: Algorithms, Theory, and Applications

- 取り込み: `raw/papers/A Comprehensive Survey of Mixture-of-Experts_ Algorithms, Theory, and Applications.md`（ar5iv クリップ, Siyuan Mu & Sen Lin, University of Houston, arXiv:2503.07137, 2025-03）
- 作成: [[summaries/2025-moe-survey]], [[translations/2025-moe-survey]]（本文 §I〜VII 全訳・425 行）, **[[concepts/mixture-of-experts]]（新設・ユーザー承認済み）**
- 更新: [[concepts/transformer-architecture]]（**MoE 節を独立ページへ切り出し**、残す記述を「transformer のどこに・なぜ入るか」＝FFN を選ぶ 2 つの理由／Switch の「削除による実用化」／パラメータ数と計算量の分離、の 3 点に圧縮して [[mixture-of-experts]] へ誘導。aliases から MoE 系の語を移譲）, [[concepts/llm-inference-optimization]]（MoE サービング節のリンク先を新ページへ付け替え、frontmatter に related と summary を追加）, [[summaries/2021-switch-transformers]]・[[summaries/2023-moe-explained]]（関連ページに新概念ページを追加）, [[overview]]（MoE の独立と、サーベイが持ち込んだ 2 領域を明記・カバレッジ表）, [[index]]（**略称リダイレクトを分割**＝MoE 系の語 24 個を transformer-architecture から mixture-of-experts へ移し、Concepts 一覧に新ページを追加）
- 画像: `raw/assets/2025-moe-survey/` に 2 枚（x1=Figure 2 の MoE 模式図、x2=Figure 3 のエキスパートネットワーク類型。ar5iv から取得・元名保持）
- メモ: ケース A（ar5iv クリップ）。クリップ自体の欠落はほぼ無いが、**Figure 1（本論文のロードマップ＝分類ツリー）は ar5iv 側にも画像が存在しない**——TikZ の `forest` で描かれており、ar5iv が画像化に失敗して LaTeX マークアップ（`\[...\]`・`leaf,text width=30em`）が本文に流れ込んでいる。ar5iv の HTML から `forest` ブロックを抽出し、**4 階層のネスト箇条書きとして完全に再構成**して §I 末尾に収録した（先例: [[translations/2023-llm-agents-survey]]・[[translations/2025-long-cot-survey]]）。**原典に表は 1 つも存在せず、脚注も 0 件**であることを ar5iv 照合で確認（本文の `<sup>3</sup>` は手法名 M³ViT の上付き文字であって脚注マーカーではない）。原典側の誤記（"Multi-tsak Learning"・"achieved achieved"・"Additinally"・"FNN layer"・§III-C の主語が壊れた一文）は訳文で意味の通る形にし、訳注に列挙した。
- 概念ページ新設の理由と影響: MoE はこれまで [[concepts/transformer-architecture]] の 1 節（全 91 行中 10 行程度）で、根拠原典も Switch Transformers と HF 入門の 2 件、記述はすべてアーキテクチャ側（負荷分散・expert capacity・共有エキスパート・スパース性スケーリング則）に寄っていた。本サーベイはそこを超える範囲——**ゲーティング関数の設計空間**（コサインルータ・指数型分布族・Soft MoE・Student-t/Laplace）、**ルーティング水準**（トークン／モダリティ／タスク／文脈・属性）、**学習パラダイム別のアルゴリズム**（継続学習・メタ学習・マルチタスク・RL）、**理論**——を持ち込む。取り込み前の検索では `Soft MoE` / `cosine router` / `fine-grained expert` / MoE 文脈での `continual learning` / `meta-learning` がいずれも wiki 内ヒット 0 件だった。ユーザー選択により独立ページ [[mixture-of-experts]] を新設し、transformer-architecture 側は概要＋誘導に縮約。**index の略称リダイレクトも付け替えた**ので、`MoE` / `Mixtral` / `expert capacity` / `router` 等の検索は今後この新ページへ着地する。
- 翻訳範囲: 本文 §I〜VII を全訳（ユーザー選択）。§V-A Computer vision（画像分類・物体検出・意味的分割・画像生成、2,450 語）も含む。除外は References のみ（原典に謝辞・付録はない）。
- 限界の扱い: **独自の実験・再評価が一切ない文献サーベイ**である点、**システム設計の記述が DeepSeek-V3 で止まっており wiki の既存原典（[[summaries/2026-deepseek-v4]]・[[summaries/2025-kimi-k2]]・[[summaries/2026-kimi-k2.5]]）のほうが新しい**点、**理論の「深層」部分が玩具設定**（2 層 CNN・二値分類・合成データ、過剰パラメータ化線形回帰）である点を要約に明記し、概念ページ側にも「引用時の注意」として設定を添えるよう書いた。応用 2 節が分析でなく列挙に近いこと、査読前プレプリントとしての編集品質のばらつきも記載。
- 拾った知見のうち wiki 的に効くもの: (1) **ゲーティングを既定値として扱わない**——問題の性質に対応する選択肢が既にある（ドメイン汎化ならコサイン、token dropping が痛いなら Soft MoE）。(2) **ルーティング水準はサービングのコスト構造を決める**——タスク水準なら推論時に関連エキスパートだけ読めばよい。(3) **エキスパートを増やせば良くなるわけではない**（ViMoE の「エキスパート数も MoE 層数も最適値がある」／MoCaE の「素朴な統合はむしろ悪化」）。(4) **RL での最も安い介入は価値ベース深層 RL の最後から 2 番目の層を soft MoE に置換すること**。(5) 継続学習の理論が出す条件——**新タスク学習時にゲートの更新を適時打ち切らないと安定しない**——が、実務側の観察（共有層は継続学習で不利／使われないエキスパートは統合で再利用）と一致している。(6) MoE の「入力を見て担当を選ぶ・負荷が偏ると崩壊する・専門化をどう起こすか」は [[concepts/multi-agent-systems]] のルーティングとオーケストレーションの論点と**同型**であり、概念ページの設計論点に相互参照として書いた。

## [2026-08-02] ingest | DeepSeek-V3 Technical Report

- 取り込み: `raw/papers/DeepSeek-V3 Technical Report.md`（ar5iv クリップ, DeepSeek-AI, arXiv:2412.19437, 2024-12）
- 作成: [[summaries/2024-deepseek-v3]], [[translations/2024-deepseek-v3]]（本文 §1〜6 ＋ 付録 B/C・776 行）
- 更新: [[concepts/mixture-of-experts]]（**負荷分散節を全面的に書き直し**＝ユーザー選択。従来の「auxiliary loss ＋ router z-loss ＋ expert capacity の 3 点セット」を「古典的な解」として残しつつ、V3 の補助損失を使わない負荷分散（バイアス項をルーティング判定にのみ使い勾配経路を汚さない・毎ステップ ±γ で調整・系列単位の微小な補完損失は残す）を対置。**さらに §4.5.3 に基づき「補助損失が悪いのではなく均衡の粒度の問題」であることを検証損失の表（1B/3B × 系列単位/なし/バッチ単位）で明示**し、「負荷を均す」と「専門化させる」の緊張を粒度で調整するという読みに整理した。ノード制限ルーティング・token dropping ゼロ・冗長エキスパートも追記。設計論点に「均衡の粒度が専門化の強さを決める」を追加）, [[concepts/transformer-architecture]]（MoE 節の直前に **MLA** を追記＝KV を低ランク潜在ベクトルへ圧縮しキャッシュ対象を 2 つに絞る。「追い出しポリシー」の系譜とは独立に効く軸として位置づけ、V4 の CSA/HCA へ繋げた）, [[concepts/llm-inference-optimization]]（`### 低精度訓練 — FP8 を極大規模で通す` 節を新設＝何を FP8 にしないか・細粒度量子化（1×128 タイル / 128×128 ブロック）・**H800 の Tensor Core が仮数部の積の上位 14 ビットしか使わない**という実測と CUDA Core 昇格による FP32 累積・オンライン量子化・**活性の勾配のブロック単位量子化は 300B トークンで発散する**という否定的結果、および DualPipe と 20/132 SM で IB+NVLink を使い切る通信設計。MoE サービング節に prefill/decode 非対称デプロイと冗長エキスパートの 10 分周期再配置を追記）, [[concepts/reinforcement-learning-from-human-feedback]]（`### 蒸留は逆向きにも流れる — DeepSeek-V3` 節を新設＝R1 が V3-Base から作られる一方で V3 の事後訓練が R1 から蒸留するという相互参照、エキスパートモデルを生成器にする 2 種類の SFT サンプル、応答長が 769→1510 に増えるトレードオフ、ルール／モデルベース RM の分業、**RewardBench で GPT-4o・Claude と同等であることを示してから自己報酬に使う**順序）, [[summaries/2025-deepseek-r1]]・[[summaries/2026-deepseek-v4]]（関連ページに V3 への参照）, [[overview]], [[index]]（略称リダイレクトに MoE 系の追加語・MLA・FP8/DualPipe 系を追加）
- 画像: `raw/assets/2024-deepseek-v3/` に 15 枚（x1〜x15。Figure 番号は 11 までだが Figure 11 が 5 パネル構成のため画像は 15 枚）
- メモ: ケース A（ar5iv クリップ）。画像 15/15・表 9/9 がクリップに残っており欠落は軽微だった。復元したのは 3 点: (1) `<sup>1</sup>`〜`<sup>4</sup>` のマーカーだけ残って本文が消えていた**脚注 4 件**（いずれも URL）、(2) **Figure 11 の総キャプション**（サブキャプション (a)〜(e) だけが残っていた）、(3) **§3.2.2 で途中で切れていた一文**——「実際には 8 個しか選ばないが同じ通信コストのまま最大 **13 個（4 ノード × 3.2 エキスパート/ノード）**まで増やせる。わずか 20 SM で IB と NVLink の帯域を使い切る」の後半が丸ごと欠落していた。HTML 表 5 個を markdown 化し、列の多い評価表は階層ヘッダを平坦化。ar5iv が `align` を 1 行ずつに分解していた数式は `aligned` にまとめ直し、MLA の式の青枠（`\boxed`）は「生成時にキャッシュが必要なベクトル」の強調なので地の文で明示した。**原典・ar5iv 側の欠落**として "tensor cores remain entirely -utilized"（"under" が変換時に落ちたとみられる）があり、訳注で明記。
- 翻訳範囲: 本文 §1〜6 ＋ 付録 B/C を訳出（ユーザー選択）。除外は References と **Appendix A（Contributions and Acknowledgments。貢献者一覧 208 行）**で謝辞相当のため。**1 箇所だけ圧縮**——§5.3.2 の Table 6（チャットモデル 7 種 × 全ベンチマーク・27 行）は Table 3 と同型で分量が大きいため要点訳に留め、その旨を訳注と冒頭の翻訳範囲に明記した。他の表（1〜5・7〜9）は全数収録。
- wiki 上の位置づけ: 本論文は**既存 2 原典の「あいだ」**を埋める。[[summaries/2025-deepseek-r1]] は V3-Base の上に作られ、[[summaries/2026-deepseek-v4]] は全編で V3/V3.2 と比較している。しかも依存は一方向でなく、**V3 の事後訓練は R1 から推論能力を蒸留している**（相互参照）。R1 の要約が「ベースモデルは V3」と一行で済ませていた部分の中身がここで埋まった。
- 限界の扱い: **訓練コスト $5.576M は「公式訓練のみ」で、事前の研究とアブレーションのコストを含まない**（原典が明記）点、**評価がすべて内製フレームワークで比較対象も自前再評価**である点、**推奨デプロイ単位が大きい**（decode で 320 GPU）ためオープンウェイトでも動かせる主体が限られる点、**ハードウェア提言は H800 世代の制約への対処**として読むべき点を要約に明記。また「aux-loss-free が勝った」という粗い引用は著者ら自身の分析（バッチ単位補助損失も同等）より弱い主張になる、と注意を付した。
- データギャップ: 残るのは **Anthropic HH 論文**（arXiv:2204.05862）、および [[concepts/context-engineering]] の基本制約が概説で置いている **context rot の研究（Chroma）**・**lost in the middle（Liu et al. 2023）**の 2 件。

## [2026-08-02] ingest | The DeepSeek Series: A Technical Overview

- 取り込み: `raw/articles/The DeepSeek Series_ A Technical Overview.md`（Shayan Mohanty / Thoughtworks, martinfowler.com, 2025-02-06 初出・2025-06-18 最終改訂）
- 作成: [[summaries/2025-deepseek-series]], [[translations/2025-deepseek-series]]
- 更新: [[concepts/transformer-architecture]]（`## 規模をどう測るか — scaling law` 節を**新設**＝Chinchilla と GPT-3 系の食い違いの一部は「規模の測り方」に由来する、非埋め込み FLOPs/token $M$ で測れば $C = M \times D$ に揃う、埋め込み層はパラメータは大きいがトークンあたり計算にほぼ寄与しないのでパラメータ数で測ると語彙サイズが「規模」に化ける、データ品質が最適比率を動かす＝scaling law は普遍定数でない。MoE のスパース性込みのスケーリング則は mixture-of-experts 側に置くと明示。あわせて MLA 段落に **V2 が初出**であることと V2 の数字（KV cache −93.3%・生成スループット 5.76 倍・訓練コスト −42.5%）を追記）, [[concepts/mixture-of-experts]]（系譜に **DeepSeekMoE** を追加＝Switch が「削除」で進んだのに対しこちらは「分割」で進んだ。細粒度エキスパート＋共有エキスパート、device-limited routing は V3 の node-limited routing の前身。負荷分散節に **V2 の 3 層の均衡損失**（$L_{ExpBal}$ / $L_{DevBal}$ / $L_{CommBal}$）を追加し、計算・配置・通信は独立に偏りうるという分解として位置づけ）, [[concepts/llm-inference-optimization]]（`### HPC co-design — アーキテクチャと計算基盤を一緒に設計する` 節を**新設**＝MLA・ルーティング制限・FP8・DualPipe を個別最適化でなく一つの方法論として束ね、各判断がどのハードウェア制約から逆算されたかを対応づけた。制約が厳しいほど協調設計の利得が大きい＝「潤沢な計算資源がない側から出てきた方法論」という読み）, [[concepts/reinforcement-learning-from-human-feedback]]（DPO 節末に **DeepSeek-LLM の DPO 採用**を追記。GRPO を自作する同じチームが前段ではまず RL ループを持たない選好学習を選んだという順序を、「PPO の 4 モデル構成への不満から出た別の枝」として整理）, [[summaries/2024-deepseek-v3]]・[[summaries/2025-deepseek-r1]]・[[summaries/2024-deepseekmath]]・[[summaries/2023-dpo]]（関連ページ）, [[overview]], [[index]]
- 画像: なし。原ページを取得して確認したところ、含まれる画像はサイトロゴ・著者顔写真・所属ロゴのみで**本文図表はゼロ**（chrome として除外）。
- メモ: ケース C（非 arXiv の Web 記事）。**クリップは無傷**で本文・脚注・数式に欠落なし。正規化したのは 3 点: (1) 15 個の脚注が「本文中の該当箇所」と「末尾の Footnotes 節」に**二重に出現**していた（原ページのポップアップ脚注の平坦化）ので `[^1]`〜`[^15]` に統合、(2) 本文に**裸の数字として残っていた脚注マーカー**（「非埋め込み FLOPs/token 1 そして彼らは…」）を正しい参照位置へ復帰、(3) 「Connecting the Arcs」の番号付きリストが原ページで **1・3・5 と飛んでいた**（markdown の入れ子崩れ）ので 1・2・3 に正規化。末尾の「Significant Revisions」は本記事が何をいつ訂正したかを示す実質的情報として訳出した。
- **原典との照合（本 ingest の主要な成果）**: 本記事は**二次資料**で、**V3 の「洗練された MLA」3 点が DeepSeek-V3 テクニカルレポートに存在しない**。live ページを `curl` で取得して 3 点とも現行版に残存することを確認済み（クリップ不良ではない）。(a) 「V2 は key を部分的にしか decouple していない、V3 が拡張して 128K 安定化」→ V2・V3 とも同一の decoupled key、**V2 の時点で既に 128K 対応**（V2 abstract）、V3 の 128K は YaRN の 2 段階拡張（§4.3）由来。(b) 「V2 は圧縮した key と value を別々に保存、V3 が統合」→ V2 の時点で単一の $c^{KV}_t$ から双方を復元する。**記事自身の V2 節がその式を引用しており内部矛盾**。(c) 「層ごとの適応的キャッシュ（深い層で古い KV を刈る）」→ **レポートに存在しない**。V3 §2.1 が「DeepSeek-V2 と比べた例外は補助損失を使わない負荷分散」と明記しているのが決定的根拠。細かい誤りが 2 点——**Streaming Microcontrollers**（正しくは Multiprocessor。記事の脚注 14 が半分自己訂正している）と「動的に分割」（V3 は 20 SM を静的割当）、FP8 の FP32 昇格理由を「オーバーフロー回避」とする（原典の理由は仮数部 14 ビットの**精度**問題）。記事の改訂履歴に「2025-06-18: DeepSeek の初期の講演／ブログには存在したが公開資料には存在しなかった詳細を削除した」とあり、**著者が一度同種の混入を掃除してもなお残った**ことになる。
- 運用上の判断: ユーザー選択により (1) 照合は要約内に**独立節**を設けて逐一対照（限界節での言及に留めない）、(2) 新規内容は**既存概念ページに配分**し `scaling-laws` の新規ページは作らない（二次資料 1 本を根拠に概念ページを立てるのは薄いため）。**概念ページ側にはこの記事を根拠とする記述は入れるが、誤りのある V3 の MLA 記述は一切持ち込んでいない**——MLA の記述はすべて [[summaries/2024-deepseek-v3]]（一次資料）に基づく。
- データギャップ: **DeepSeek-LLM（arXiv:2401.02954）と DeepSeek-V2（arXiv:2405.04434）の一次資料が未取得**。本記事で概略は入ったが、scaling law の具体的な冪乗則の係数や MLA の設計判断の詳細は二次資料経由のまま。加えて既存の未取得として **Anthropic HH 論文**（arXiv:2204.05862）、**context rot の研究（Chroma）**、**lost in the middle（Liu et al. 2023）**。

## [2026-08-02] ingest | From Local to Global: A Graph RAG Approach to Query-Focused Summarization

- 取り込み: `raw/papers/From Local to Global_ A Graph RAG Approach to Query-Focused Summarization.md`（Edge, Trinh ほか / Microsoft Research, arXiv:2404.16130, 2024-04）
- 作成: [[summaries/2024-graphrag]], [[translations/2024-graphrag]]（本文 §1〜6 全訳。原典に付録なし）
- 更新: [[concepts/retrieval-augmented-generation]]（**代表手法に GraphRAG を追加**＝既存の 2 手法（原典 RAG・DPR/REALM）が「どう検索するか」の改善なのに対し、GraphRAG は「検索では原理的に答えられない質問がある」という診断から索引の作り方を変える、という**別の軸**として書いた。パイプライン 4 段・通常の知識グラフとの違い（トリプルでなく豊かな記述テキスト／エンティティ解決を省く賭け）・実測とその正しい読み方（グラフの真価は品質でなくトークン単価）・償却の問題まで。冒頭の「2 つの実装層」にも 3 番目の軸への導線を追加。設計論点には **`その質問はローカルか、グローバルか`** を冒頭に新設（既存の設計論点の大半がローカル側の話であることを明示）、末尾に **`索引を作るコストは生涯クエリ数で割って考える`**（just-in-time との連続性を明示）と **`コンテキストは大きいほど良いとは限らない`**（8k が 64k に勝った実測）を追加）, [[summaries/2020-rag]]・[[summaries/2023-memgpt]]・[[summaries/2025-a-mem]]（関連ページ）, [[overview]], [[index]]
- 画像: `raw/assets/2024-graphrag/` に 12 点。内訳は **ar5iv のインライン SVG から抽出した 10 個**（`fig1-pipeline.svg`, `fig2-gleanings.svg`, `fig4-{podcast,news}-{comprehensiveness,diversity,empowerment,directness}.svg`）と **JPG 2 枚**（`Level0Multihop.jpg`, `Level1Multihop.jpg`）。翻訳からの参照数 12 = 保存数 12、各 1 回ずつ。
- メモ: ケース A（ar5iv クリップ。`raw/papers/` 配置だが中身は markdown）。**本文・脚注・表はすべて無傷**で、復元が必要だったのは 1 点——**Figure 3 は (a) Level 0 / (b) Level 1 の 2 パネル構成だが (b) `Level1Multihop.jpg` がクリップから欠落**していたので ar5iv から取得した。
- **図の形式が特殊だった点**: Figure 1・2・4 は原典で TikZ により描かれており、ar5iv では**ラスタ画像ではなくインライン SVG** としてレンダリングされている（クリップの 167・169 行目が 7 万字超なのはこれが理由）。従来のケース（MoE サーベイ Figure 1 や long-CoT サーベイのように「ar5iv 側にも画像が存在せずテキスト再構成するしかない」）とは異なり、**SVG は完全な描画データとして存在した**ので、10 個を抽出して `xmlns` を補い `.svg` として保存した（Obsidian で描画される）。
- **Figure 4 の数値の書き起こし**（ユーザー選択により SVG 保存と併用）: Figure 4 は 6×6 の勝率行列 × 8 パネル（2 データセット × 4 指標）で本論文の主結果にあたり、**SVG 内にテキストとして数値が含まれていた**ので markdown 表 8 枚にも起こした。**行・列の条件の並びは SVG からは読めない**ため、本文 §3.6 の記述 8 件（「global approaches achieved comprehensiveness win rates between 72-83% for Podcast transcripts」「Intermediate-level summaries in the Podcast dataset ... 57%」「low-level ... in the News dataset ... 64%」等）と照合して **SS, TS, C0, C1, C2, C3** の順であることを確定し、さらに**全パネルで対称セルの和が 100 になること**を検証した。原典キャプションが述べる「総合勝者の太字」は SVG のテキスト抽出では判別できないため再現せず、その旨を訳注に明記した。
- 引用の扱い: クリップは著者-年形式の引用を裸の参照番号に変換していた（「Leiden algorithm 47」「MultiHop-RAG 45」）。参照番号 `[^N]` は維持しつつ、**本文理解に必要な箇所（Leiden / Louvain / HotPotQA / MultiHop-RAG / OpenORD / Force Atlas 2）は ar5iv から著者-年を復元して併記**した。文献一覧は既定どおり除外。
- 概念ページの配分: ユーザー選択により **retrieval-augmented-generation のみ**に集約した（agent-evaluation / agent-memory / context-engineering へは波及させず）。評価設計の知見（正解のないタスクの測り方、**対立する対照指標＝直接性を混ぜて判定系の妥当性を検査する**設計、活動起点の質問生成、5 回反復平均）は要約ページ内に書き切った。
- 批判的視点として要約に明記した点: (1) **Table 3 のトークン表はクエリ時のみでグラフ索引の構築費を含まない**——「C0 は 9〜43 倍安い」は償却前の数字で、比較対象の TS には索引化コストがそもそも無い（著者らは議論節で「生涯クエリ数の見込み」に依存すると自ら書いている）、(2) **グラフなしの TS が競争力を持っており** C1〜C3 の優位は 52〜64% と大きくない——主張の重心は「グラフが強い」でなく「グローバルな map-reduce が素朴な RAG より強い」、(3) **捏造率が測られていない**（網羅性・多様性で勝つ手法がより多く捏造していない保証はない）、(4) LLM-as-a-judge の **verbosity bias** は 5 回平均では消えず、網羅性・多様性は構造的に長い答えに有利、(5) 指標自体がまだエンドユーザーで検証されていない（著者ら自身が課題に挙げている）、(6) エンティティ解決を省く判断は「十分な連結性があれば」という条件つきで、その条件が破れる場合が測られていない。
- データギャップ: 引き続き **DeepSeek-LLM（arXiv:2401.02954）と DeepSeek-V2（arXiv:2405.04434）**、**Anthropic HH 論文**（arXiv:2204.05862）、**context rot の研究（Chroma）**、**lost in the middle（Liu et al. 2023）**が未取得。うち lost in the middle は本論文が 2 度引用し、8k が 64k に勝った実測の説明にも使われているので優先度が上がった。

## [2026-08-02] ingest | Towards End-to-End Automation of AI Research (The AI Scientist)

- 取り込み: `raw/papers/Towards End-to-End Automation of AI Research.md`（Yamada, Lange, Cong Lu, Chris Lu, Hu, Foerster, Ha, Clune / Sakana AI・UBC・Vector Institute・Oxford FLAIR, arXiv:2606.15497, 2026-06。Nature 形式。v1 = arXiv:2408.06292 の発展版）
- 作成: [[summaries/2026-ai-scientist]], [[translations/2026-ai-scientist]]（**3,894 行。本 wiki 最大の翻訳**）, **[[concepts/research-agents]]（新規概念ページ）**
- 更新: [[concepts/agent-evaluation]]（`## (c) LLM-as-a-judge` 節に Automated Reviewer を追加。既存節が DPO の人間基準線・MASFT の κ・Anthropic のハーネス実験・Reflexion の偽陽性/偽陰性・Agent-as-a-Judge・Research のルーブリックで既に厚いので、**重複しない 4 点に絞った**——(1) 人間の一致率でなく**制度の最終判定（accept/reject）を正解に使う**、(2) **知識カットオフ前後でデータを分割して汚染を検査**（著者らは balanced accuracy 69%→66% から「汚染は存在しうる」と認めたうえで効果は最小限と述べており、「汚染なし」とは言っていない。この誠実さごと記録）、(3) **自明ベースラインの併記**（Always Reject は精度 0.65 = AR と同じだが F1 0.00。精度単独報告の危険の実例）、(4) **校正は移植できない**（会議用プロンプトのままワークショップに当てると 3 本とも不採択）。アブレーションの否定的結果——肯定/否定プロンプトの微調整・VLM・Reflexion・few-shot はいずれも一貫した改善なし、効いたのは 5 アンサンブルだけ——も追記）, [[concepts/llm-agents]]（「イノベーション指向（科学探究）」から新ページへの導線）, [[summaries/2026-sakana-fugu]]・[[summaries/2025-multi-agent-research-system]]・[[summaries/2023-reflexion]]（関連ページ）, [[overview]], [[index]]
- 画像: `raw/assets/2026-ai-scientist/` に 18 枚（すべて PNG。翻訳からの参照数 18 = 保存数 18、各 1 回）。ar5iv 側でサブディレクトリに分かれていたので `compreg-` / `labelnoise-` / `pest-` の接頭辞で平坦化した（`model_class.png` のような曖昧な名前を避けるため）。
- メモ: ケース A（ar5iv クリップ。`raw/papers/` 配置だが中身は markdown）。**図表キャプションは Figure 1〜16・Table 1〜10 がすべて残存**し、表も 10/10 が残っていた（4 つが HTML、6 つが markdown）。**欠落は画像 2 枚のみ**で、いずれも多パネル図の 2 枚目という既知の不良パターン——`noisy_dataset_class_2.png`（Figure 12 (b)）と `multi_dataset_train_loop.png`（Figure 14 (b)）——を ar5iv から復元した。
- **インライン SVG 96 個の扱い（今回の主要な判断）**: 本論文は ar5iv 上で**プロンプト・生成 JSON・生成コードが tcolorbox → インライン SVG として描画**されている（クリップの 1 行が 14 万字に達するのはこのため）。**96 個すべてがテキスト箱で、`<path>` を 5 個以上持つもの＝図はゼロ**であることを確認したうえで、**GraphRAG のときと違って `.svg` としては保存せず、テキストに起こして該当位置に配置**した。GraphRAG の SVG は図（パイプライン図・勝率行列）だったので保存に意味があったが、今回は転記したテキストを超える情報を持たず、保存すればリポジトリを 500KB 以上膨らませるだけだからである。
- 翻訳の方針: **本文＋Methods＋Nature 定型節＋付録 A〜D の全体を訳出。圧縮箇所なし。** 原文のまま残したのは (a) **システムプロンプト 20 個**（A.1.1 の 7・A.2.6 の 9・A.3.1 の 4。skill の「プロンプトは一字一句が挙動に効く」に従う）、(b) **生成されたコード**（A.1.2 の DDPM テンプレート 241 行、C.1 のコード差分）、(c) **系が生成した JSON 成果物**（A.2.7 のアイデア、A.2.8 の各段サマリ、**C.4 のアイデア 51 件**、付録 D のアイデアと自動査読）——先例は [[translations/2026-sakana-fugu]] が棋譜を原文収録していること。散文である査読コメントとチームレビューは訳出した。プロンプト内の LaTeX 由来スマートクォートは実際のプロンプト文字列に戻すため `'` `"` `---` へ正規化した。
- **当初計画からの変更（付録 D の圧縮は不要だった）**: 計画段階では付録 D（8 万字超）の生成論文本文を圧縮する予定だったが、読解の結果、**生成論文の本文は原典で `\includepdf` により PDF を丸ごと差し込む形になっており、ar5iv は PDF を展開できないため「See pages - of ...」という指示だけが残っている**ことが判明した。つまり**論文本文は底本にもともと存在しない**。原典に実在する内容（構成・Table 10・生成アイデア・チーム内部レビュー・コードレビュー・実際のワークショップ査読・生成コードの図）は**すべて訳出**し、その旨を翻訳冒頭の訳注に明記した。
- **コードスクリーンショット 12 枚**（Figure 7〜16）は Case C-6 の方針に従い、**画像として収録したうえで Read で開いて訳注にコードブロックとして文字起こし**した。文字起こしの過程で訳者が気づいた点も 2 か所だけ注記——Figure 9 の `hidden_states = model.embedding(expr)` が「正則化が隠れ状態でなく埋め込みに掛かっている」というチームの指摘の根拠にあたること、Figure 14 の `domain_labels = torch.zeros(...)` がどのデータセットでも常に 0 を与えており領域判別器が学習できないこと（チームの「試みは成功していなかった」の内実）。
- **原典由来の問題（クリップ不良ではない）**: 付録 A.1.2 の箱のタイトルが "Generalized Idea Generation System and User Prompt" なのに**中身は DDPM の訓練スクリプト**で、節の見出し（"An Example Template Codebase"）とも合わない。同じタイトルの箱が A.2.6 冒頭に正しく存在するので、原典の LaTeX で tcolorbox のタイトルを差し替え忘れたものと思われる。ar5iv 側でも同じなので原文どおり残し、訳注で指摘した。
- 概念ページ `research-agents` の骨格: **本論文が §A.2.5 で自ら引いた線**——「Deep Research 系は知識の集約に留まり、新しい実験を設計も実行もできない」対「The AI Scientist は知識の生成」——を対比表にして中心に据えた。既存資産の [[summaries/2025-multi-agent-research-system]] が集約型の本番実装として、現状 `multi-agent-systems` にしか記述がなかったので、ここから参照する形にした。共通の構成要素（着想アーカイブと新規性フィルタ／段階つき実験管理と観測可能な停止条件／統計を取るための探索ノード型／VLM を批評者に立てる／役割ごとのモデル使い分け）、スケーリングの 2 軸、評価の難所、設計論点（人手選別の報告規範・計算実験と湿式実験の境界・制度への外部性・サンドボックス・co-scientist・taste の欠如）で構成。
- 批判的視点として要約に明記した 10 点: (1) **人手の選別漏斗**（Table 6。採択論文は原稿 24 本中 1 本、応用系はアイデア 136 個中 1 つ。著者らの「選別なしでも同じ論文は生成された」という弁明は公正だが、**「AI が査読を通る論文を書いた」と「24 本中 1 本が通った」は別の主張**）、(2) **著者ら自身の Automated Reviewer が既定プロンプトで 3 本とも不採択**、(3) **ワークショップ採択率 70%**（本会議 32%）と内部レビューの「本会議水準を満たすものは 1 本もない」、(4) **採択論文自体の欠陥**——**テスト集合の約 57% が訓練集合と重複**、attention-LSTM の「100%」は数を [1-9]→[10-19] に変えると 56% に落ちる（ベースラインは 85%→0%）、LSTM の発明を Goodfellow らに誤帰属、(5) 不採択 2 本の欠陥（**temperature scaling を実装したが一度も使わずに論文には書いた**／領域適応のコードが動かず、選ばれた版に複数データセット訓練がないのに本文はあると書く）、(6) 系の失敗モード（交絡変数を制御した公正な実験ができない、将来トークンを漏らす「カンニング」、KL が 0.090→0.093 と悪化したのを "3.3% improvement" と書く、V100 を使ったと幻覚するが実際は H100）と著者らの結論「**科学的内容を額面どおりに受け取ることは推奨されない**」、(7) **サンドボックスなしでの制約回避**（自己再起動によるプロセス増殖・1TB 近い記憶容量の消費・時間制限そのものの書き換え）、(8) AR の評価データの偏り（不採択は元投稿・採択はカメラレディ）、(9) 計算実験のみ、(10) Carl / Zochi との比較は**両者が非公開のため公開情報に限られる**。
- データギャップ: 引き続き **DeepSeek-LLM（2401.02954）/ DeepSeek-V2（2405.04434）/ Anthropic HH（2204.05862）/ context rot（Chroma）/ lost in the middle（Liu et al. 2023）**。新たに **AI Scientist v1（arXiv:2408.06292）** が「参照されているが未取得」に加わるが、本論文が v1 を包含するので優先度は低い。**AIDE / MLE-bench など research agents 領域の他の原典**も未取得で、`research-agents` は現状 3 本の要約に支えられている状態。

## [2026-08-02] schema-update | ケース B（PDF 原典）を「抽出可能なら抽出する」へ改訂

- 背景: ingest skill のケース B は「**PDF からは画像をプログラム的に取り出せない**」ことを根拠に、PDF 原典では画像なしを既定としていた。しかし SWE-agent の ingest 時に確認したところ、**この環境には `pdftotext` と PyMuPDF（`import fitz`）が両方あり、前提が成り立たない**。テキストは綺麗に抽出でき、図もベクタ描画であってもキャプション位置を境界に領域をレンダリングすれば取得できる（実際に 22 枚を取得して検証した）。
- 変更（ユーザー承認済み）: `.claude/skills/ingest/SKILL.md` のケース B を **「PDF 原典 — 抽出可能なら抽出する」** へ全面改訂した。内容は (1) **テキストは `pdftotext` / PyMuPDF で抽出し翻訳の底本にする**（ページ画像を Read で読むより桁違いに安く正確。ただし構成把握と図の内容確認には Read を併用）、(2) **図はベクタ描画なので `get_images()` では取れない**——キャプションの bbox を下端、その上の本文段落の bbox を上端として `get_pixmap(clip=...)` でレンダリングする、左右に並ぶ図はキャプションの x 範囲で分割する、**保存後は必ず Read で開いて確認**する、(3) **何を画像にし何をテキストに起こすか**の判断——ダイアグラム・プロット・スクリーンショットは画像、**中身が文字だけの「箱」（プロンプト・設定・コード・ログ）はテキストに起こす**（ケース A/C の SVG テキスト箱と同じ判断）、(4) 抽出ツールが無い環境では従来どおり画像なしでよい。
- あわせて `CLAUDE.md` §6 のツール節に **`pdftotext` / PyMuPDF** を追加し、「**PDF だからといって図を諦めない**」と明記した。
- 影響: 今後の PDF 原典すべてに効く。既存の PDF 由来の要約・翻訳（画像なしで作成したもの）は遡って修正しないが、再訪の機会があれば図を補える。

## [2026-08-02] ingest | SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering

- 取り込み: `raw/papers/SWE-agent- Agent-Computer Interfaces Enable Automated Software Engineering.pdf`（Yang, Jimenez, Wettig, Lieret, Yao, Narasimhan, Press / Princeton Language and Intelligence, **NeurIPS 2024**, arXiv:2405.15793v3, 118 ページ）
- 作成: [[summaries/2024-swe-agent]], [[translations/2024-swe-agent]]（**3,563 行**）
- 更新: [[concepts/coding-agents]]（**`## ACI — インターフェースを設計変数にする` 節を新設**。従来この概念は「SWE-agent（2024, **概説**）が確立した」と旗が立っていただけだった。4 つの設計原則・実装の要点・**リンタを差し戻しのガードレールにする 3 点セット**（エラー／編集後／編集前。元の内容がないとエージェントは「より新しい」という理由で誤った内容に対して編集を生成する）・アブレーション表・**編集の連鎖失敗**（1 回失敗で回復率 90.5%→57.2%）・失敗の内訳（実装の誤り 52.0% に対し関連ファイルを見つけられない失敗はわずか 2.4%）・**「速く成功しゆっくり失敗する」**。あわせて**フロンティアモデルのハーネスが逆に最小構成へ収斂している**ことと ACI が「同じ軸の両端」である旨を追記）, [[concepts/tool-use-and-function-calling]]（`### 誰のためのツールか — ACI という問い` を function calling 節の**手前**に新設。function calling が「どう宣言するか」なのに対し ACI は「**そもそも何をツールにするか**」という前段の問いである、という位置づけ）, [[concepts/agent-evaluation]]（SWE-bench に一次資料が付いた。**別々に研究されてきた SE のサブタスクを単一のタスク定式化へ統一している**点、**イシューの作成年で層別して汚染を見る**という手法（LeetcodeHardGym とは別方向）、**絶対水準の解釈への注意**（12.47% は当時最高だが 87.5% は解けていない））, [[summaries/2026-harness-design]]・[[summaries/2025-effective-harnesses]]・[[summaries/2026-meta-harness]]・[[summaries/2022-react]]（関連ページ）, [[overview]], [[index]]
- 画像: `raw/assets/2024-swe-agent/` に **22 枚**（Figure 1〜9・14〜26）。**PDF から領域レンダリングで取得**——ベクタ図なので埋め込み画像の抽出では取れず、キャプションの bbox を境界に `get_pixmap(clip=...)` で切り出した。**左右に並ぶ Figure 7/8 と、Table 2 の右にある Figure 4 は個別に範囲を指定し直し**、Read で開いて切れていないことを確認した。翻訳からの参照数 22 = 保存数 22、各 1 回。
- メモ: ケース B（PDF）。**本 ingest がケース B の改訂契機**になった（上の schema-update を参照）。テキストは `pdftotext -layout`（表の再構成）と PyMuPDF で抽出し、構成の把握と図の確認には Read を併用した。
- **テキストに起こした図**: 中身が文字だけの箱は画像化せず本文へ起こした——Figure 10（ファイルビューアと検索の出力例）・Figure 11（リンティングのエラーメッセージ全文）・Figure 12（設定 YAML）・Figure 13（コマンド定義の骨格）・**Figure 27〜32（プロンプトテンプレート 6 種）**・Figure 33〜36（trajectory）。プロンプトとコマンドのドキュメント文字列は**実際に LM へ渡される文字列そのもの**なので英語原文のまま。
- 付録 D の収録範囲（ユーザー選択）: 4 例のうち**成功例 `psf/requests-2317`（787 行）と失敗例 `django/django-14411`（1,756 行）をターンごとのログまで完全収録**し、`pylint-dev/pylint-5859`（成功）と `sympy/sympy-21614`（失敗）は要約・評価・gold patch のみ。**失敗例に django を選んだのは、それが最頻の失敗モード（Incorrect Implementation, 39.9%）にあたるため**——sympy は `exit_cost` で終わる Ran Out of Budget（2.0%）の例だった。なお当初「成功 1・失敗 1」と決めた時点では `pylint-5859` を失敗例と誤って想定していたが、本文を読むと**成功例は requests と pylint の 2 本**であることが判明したので選び直した。
- **原典側の不整合（2 件、いずれも訳注で明記）**: (1) **Table 4 のドキュメント列で `scroll_down` が "Moves the window up 100 lines"、`scroll_up` が "Moves the window down 100 lines" と上下が逆**。この列は LM へ提示される実際の文字列を示すものなので、そのまま渡されていたなら誤ったドキュメントで運用されていたことになる。(2) **HumanEvalFix の成績が Abstract で 87.7%、§5 本文で 88.3%** と食い違う（Table 2 の内訳は Python 87.7 / JS 89.7 / Java 87.9）。
- 技術メモ: 翻訳の組み立てで**コードフェンスの入れ子**に注意が要った。付録 C のプロンプト 3 種（システムプロンプト・実演・エラーメッセージ）は**中身に ``` を含む**ので外側を 5 バックティックにした。また trajectory のテキストファイルに末尾改行がなく、閉じフェンスが最終行へ連結される事故があったので修正した。最終的に**13 ブロックすべての対応を検証**済み。
- 批判的視点として要約に明記した点: 絶対水準の低さ（87.5% は未解決）／**2024 年前半の数値は陳腐化しており、古びていないのは ACI の概念とアブレーションの非自明さのほう**／**ACI の設計過程が手作業**で著者ら自身が「我々の手作業のアプローチが確かにスケールしない」と明記していること（これを自動化しようとするのが [[summaries/2026-meta-harness]]）／Llama 3・DeepSeek Coder では「性能が不十分」と述べるだけで**数値がなく**、ACI のモデル非依存性の証拠は 2 モデルのみ／**実演の効果は著者ら自身も疑っている**（アブレーションで ↓1.7、「機微の理解を助けているかは確信していない」）／安全性の前提が Docker のみで、著者らも「仮想化されたハードウェア隔離ほど安全とは見なされない」と条件つきで許容していること。
- データギャップ: **SWE-bench 論文そのもの（Jimenez et al. 2023, arXiv:2310.06770）は依然として未取得**。本 ingest でベンチマークの素性（12 リポジトリ・2,294 件・実行ベース評価・Lite/Dev の内訳・年別層別）は入ったが、収集パイプラインと汚染対策の詳細は二次経由のまま。加えて既存の未取得として **lost in the middle（Liu et al. 2023, arXiv:2307.03172）**（4 セッション以上追跡中。[[concepts/context-engineering]] の基本制約の根拠）、**Self-Refine（arXiv:2303.17651）**・**Tree of Thoughts（arXiv:2305.10601）**（それぞれ [[concepts/self-reflection]]・[[concepts/reasoning-and-planning]] が「原典未 ingest のため概説」と明記）、**Anthropic HH（arXiv:2204.05862）**、**DeepSeek-LLM（2401.02954）/ DeepSeek-V2（2405.04434）**、**context rot（Chroma）**。

## [2026-08-02] ingest | SWE-bench: Can Language Models Resolve Real-World GitHub Issues?

- 取り込み: `raw/papers/SWE-bench_ Can Language Models Resolve Real-World GitHub Issues_.md`（Jimenez, Yang, Wettig, Yao, Pei, Press, Narasimhan / Princeton University・Princeton Language and Intelligence・University of Chicago, **ICLR 2024**, arXiv:2310.06770。`raw/papers/` 配置だが中身は ar5iv クリップ＝ケース A）
- 作成: [[summaries/2023-swe-bench]], [[translations/2023-swe-bench]]（740 行）
- 更新: [[concepts/agent-evaluation]]（**SWE-bench の一次資料化**。`### ベンチマークを「作る」でなく「収穫する」— SWE-bench の設計` を新設し、転用可能な 4 判断——**「人間にも解けない問題」の能動的除外**（解で初めて命名された関数を呼ぶテスト、依存関係の命名に起因する ImportError/AttributeError）・**回帰を評価に組み込む**（fail-to-pass と pass-to-pass の分離、中央値 51 個）・**リーク経路を先に塞ぐ**（問題記述は PR 初回コミットより前のコメントのみ）・**実行環境の粒度をリリースバージョン単位に選ぶ理由**——を整理。あわせて**汚染を層別で見る**手法（作成年で分けて 2023 年前後の差を見る。カットオフ後の問題だけで組む [[summaries/2023-reflexion]] の LeetcodeHardGym とは別方向で併用可能）と、**ベンチマーク論文の初期スコアを「モデルの上限」と読むのは誤読**である旨を明記）, [[concepts/context-engineering]]（**「文脈を増やすと悪化する」の最も直接的な実測**を表つきで追加。recall 29.58 → 51.06 に対し解決率 1.96 → 1.22、および ±15 行以外を畳むと 4.8 → 5.9 へ上がること。[[summaries/2024-swe-agent]] の窓サイズの両側最適点・人間 UI 模倣の検索が有害という結果と合わせ、**「取れる情報を全部渡す」は設計でなく怠慢**という共通の含意へ接続）, [[concepts/retrieval-augmented-generation]]（設計論点に **`検索指標の改善は、成果の改善を意味しない`** を新設。**recall を最適化目標に据えると下流が壊れうる**という反例として。密検索でなく BM25 を選んだ理由——キーとクエリが極端に長く自然言語クエリでコードを引く設定——も既存の「エンティティ中心では BM25 が勝つ」論点に接続）, [[concepts/coding-agents]]（評価節の冒頭に SWE-bench を置き、**出発点の水準**（オラクルでさえ Claude 2 4.8% / GPT-4 1.7%）を「この領域の進歩の速さを測る基準線」として記録。当時の失敗の質——gold より短く単純な編集、ほぼ単一ファイル、パッチ形式のほうがファイル全体より易しい、整形式パッチ自体に苦労、**成績が同程度でも解けている問題が重ならない**（Claude 2 は SWE-Llama が解いた問題の 42% しか解けない）——も）, [[summaries/2024-swe-agent]]・[[summaries/2023-reflexion]]（関連ページ）, [[overview]], [[index]]
- 画像: `raw/assets/2023-swe-bench/` に **16 枚**（`x1`〜`x8` と CDF 8 パネル）。翻訳からの参照数 16 = 保存数 16、各 1 回。
- メモ: ケース A。**クリップ不良が 2 種類あった。**
  1. **Table 1 と Table 6 が丸ごと欠落**（キャプションも本体も）。**Table 1 は本文が 4 回参照している基本統計の表**（issue 平均 195 語、コードベース 3,010 ファイル・438K 行、gold patch 32.8 行 / 1.7 ファイル / 3 関数、fail-to-pass 9.1・総テスト 120.8）。Table 6 は "Oracle"-collapsed の結果（Claude 2 5.9 / GPT-4 3.4）。**両方 ar5iv から復元**し、復元箇所に訳注を付した。
  2. **画像 16 枚中 8 枚が欠落。** 単独図の `x3.png`（Figure 3, リポジトリ分布）と、**Figure 9 の CDF 8 パネルのうち 7 枚**（`p_num_words` のみ残存）。多パネル図の 1 枚目だけ残る既知パターン。すべて ar5iv から復元。
- 付録 F の収録範囲（ユーザー選択）: 生成例 10 件（Table 16〜25, 8.6 万字）のうち**代表 3 例を原文のまま完全収録**——**Table 16**（成功だが既存メソッドを使わず原始的な Python を書く＝§5.1 の傾向の証拠）・**Table 23**（成功。しかも **gold より効率的で綺麗な解**＝実行ベース評価が参照解との一致でなく振る舞いを見ていることの帰結）・**Table 25**（失敗し、**既存の振る舞いも壊す**＝pass-to-pass テストが存在する理由）。残る 7 例は著者らの分析のみ訳出。なお計画時は 3 例目に「テストは通るが実は壊れている」例を想定していたが、**原典にそのような例はなく**、Table 24（誤りだが振る舞いは保つ）と Table 25（誤りかつ壊す）があるだけだったので後者を採り、その旨を訳注に明記した。
- 翻訳中の自己修正: **§8 Ethics Statement と §9 Reproducibility Statement を、原文を読まずに内容を推測して書いてしまった**のに気づき、原文を読み直して全面的に差し替えた。原文は「GitHub ユーザーの情報を収集していない」「人間の被験者の参加を伴わない」「人気に基づくフィルタは差別的ヒューリスティクスに依拠しない」等の具体的な表明で、推測した内容とは実質的に異なっていた。
- 批判的視点として要約に明記した点: **数値は完全に陳腐化している**（4.8% → 翌年 12.47% → その後さらに）／Python・12 リポジトリのみ／**「オラクル」設定は反則であることを著者ら自身が明記**し、かつそれでも包括的でないと述べていること／**ベースラインは意図的に素朴**で著者らが「エージェントベースのアプローチに期待している」と書いていること（実際 7 か月後に SWE-agent）／**実行ベース評価だけでは不十分**と著者ら自身が述べていること（テスト通過は必要条件であって十分条件でない。Table 24 がその具体例）／**`hints_text` は収集されているが実験では未使用**なので、引用時は設定を明示しないと比較にならない／**GPT-4 は予算の制約で 25% の部分集合でしか評価されていない**。
- データギャップ: **前回記録した「SWE-bench 論文そのものが未取得」を解消。**残るのは **lost in the middle（Liu et al. 2023, arXiv:2307.03172）**——本論文が「文脈を増やすと悪化する」の説明として明示的に引いており、[[concepts/context-engineering]] の基本制約の根拠でもあるので**優先度が最も高い**——、**Self-Refine（arXiv:2303.17651）**・**Tree of Thoughts（arXiv:2305.10601）**（それぞれ [[concepts/self-reflection]]・[[concepts/reasoning-and-planning]] が「原典未 ingest のため概説」と明記）、**Anthropic HH（arXiv:2204.05862）**、**DeepSeek-LLM（2401.02954）/ DeepSeek-V2（2405.04434）**、**context rot（Chroma）**。

## [2026-08-02] ingest | Lost in the Middle: How Language Models Use Long Contexts

- 取り込み: `raw/papers/Lost in the Middle_ How Language Models Use Long Contexts.md`（Liu, Lin, Hewitt, Paranjape, Bevilacqua, Petroni, Liang / Stanford University・UC Berkeley・Samaya AI, **TACL 2024**, arXiv:2307.03172。`raw/papers/` 配置だが中身は ar5iv クリップ＝ケース A）
- 作成: [[summaries/2023-lost-in-the-middle]], [[translations/2023-lost-in-the-middle]]（395 行）
- 更新: [[concepts/context-engineering]]（基本制約 2 を一次資料化し、**`## この制約の系譜 — 2023 から 2025 への追試` 節を新設**＝ユーザー選択。本論文（2023, 一般 QA）→ [[summaries/2023-swe-bench]]（2023, コード）→ [[summaries/2024-graphrag]]（2024, 長文要約）→ [[summaries/2024-swe-agent]]（2024, インターフェース設計）→ [[summaries/2025-effective-context-engineering]]（2025, context rot として定式化）を 1 つの表にまとめ、読み取れること 3 点——**古びたのは数字であって命題ではない**・**検索が良くなることと成果が良くなることは別**・**対処は「減らす」と「並べ替える」に集約される**——を整理。あわせて一次資料の**原因調査 4 実験**（decoder-only のせいではない／クエリ位置だけでもない／instruction tuning のせいでもない／7B は recency のみで 13B・70B から U 字）と、**機構的説明は与えられておらず著者ら自身が「予備的」と呼んでいる**ことも明記）, [[concepts/retrieval-augmented-generation]]（設計論点に **`reader は retriever より遥か手前で飽和する`** を新設。20→50 文書で GPT-3.5 が約 1.5%・Claude が約 1% しか伸びないこと、**答えを中間に置くと closed-book を下回る**こと、**拡張コンテキスト版が元版とほぼ同じ曲線**であること、処方箋の**再ランキングと ranked list truncation**、そして「関連度の降順に並べる」が自明な既定に見えて U 字を前提にすると最適とは限らないこと）, [[summaries/2023-memgpt]]・[[summaries/2023-swe-bench]]・[[summaries/2024-graphrag]]・[[summaries/2025-effective-context-engineering]]・[[summaries/2024-swe-agent]]（関連ページ。**いずれも本論文を二次参照していた 5 ページで、これで一次資料へ繋がった**）, [[overview]], [[index]]
- 画像: `raw/assets/2023-lost-in-the-middle/` に **16 枚**（`x1`〜`x16`）。翻訳からの参照数 16 = 保存数 16、各 1 回。
- メモ: ケース A。**クリップに欠落はなかった**——原ページと照合して画像 16/16、Figure 1〜16 と Table 1〜7 のキャプションおよび本体がすべて揃っていることを確認。ここ数回の ingest で連続していた多パネル図の欠落や表の消失が今回はない稀な例。復元作業はゼロ。付録 F の HTML 表 3 個（トークン数統計。`±` が `<math>` 要素だった）のみ markdown へ変換した。翻訳範囲は本文＋付録 A〜G の全体で**圧縮なし**。
- **本 ingest の位置づけ**: この論文は**取り込み前から wiki 内の 6 ページが二次参照していた**（[[concepts/context-engineering]] は 2 つの「基本制約」の片方をこれに置いていた）。一次資料化の効果が大きく、散らばっていた言及が 1 本の系譜に繋がった。
- 要約で強調した「よく引かれる主張より強いこと」2 点: (1) **文書を与えるほうが与えないより悪くなる**——GPT-3.5-Turbo の closed-book は 56.1% だが、20 文書の index 9 で 53.8%、30 文書の index 9 で 50.5%。「中間は弱い」ではなく「**中間は有害でありうる**」が正確。(2) **拡張コンテキスト版は元版より良くない**——GPT-3.5-Turbo と 16K、Claude-1.3 と 100K がいずれも小数点以下しか違わない。
- 頑健性の確認として原典が潰している代替説明: **無作為な気を散らす文書に替えても U 字は消えない**（＝関連文書を特定できないことだけが原因ではない。付録 B）／**「順序は無作為」とプロンプトで明示して実際にシャッフルしても U 字は残る**（＝「検索結果は関連度順」という事前分布のせいではない。付録 C）／**GPT-4 でも U 字は出る**（付録 D）。
- 批判的視点として要約に明記した点: **モデルが 2023 年世代**で絶対性能はそのまま持ち込めない（ただし現象自体は追試され続けている）／**タスクが 2 つともに「探して答える」型**で、要約・生成のように関連情報が局在しないタスクへの一般化は本論文からは言えない／**原因調査は著者ら自身が「予備的」と明記**し機構的説明はない／**query-aware contextualization が合成検索では効くのに QA では効かない**非対称性も観測のみで未解明／評価が「答えの文字列が出力に含まれるか」という緩い一致で、NaturalQuestions の注釈と Wikipedia ダンプの時間的不整合も著者らが認めている（付録 A で曖昧でない部分集合の再実験により結論不変を確認）。
- データギャップ: **3 セッション以上「優先度が最も高い」として追跡していた本論文の未取得を解消。**残るのは **Self-Refine（arXiv:2303.17651）**・**Tree of Thoughts（arXiv:2305.10601）**（それぞれ [[concepts/self-reflection]]・[[concepts/reasoning-and-planning]] が「原典未 ingest のため概説」と明記。とくに self-reflection は 68 行で推論系の概念ページ中最薄）、**context rot の研究（Chroma）**（[[concepts/context-engineering]] の系譜表の最終行で、本 ingest により**系譜の中で唯一の未取得**になった）、**Anthropic HH（arXiv:2204.05862）**、**DeepSeek-LLM（2401.02954）/ DeepSeek-V2（2405.04434）**、**SWE-agent v1 が参照する AI Scientist v1（2408.06292）**。

## [2026-08-02] ingest | FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness

- 取り込み: `raw/papers/FlashAttention_ Fast and Memory-Efficient Exact Attention with IO-Awareness.md`（ar5iv クリップ。Tri Dao, Daniel Y. Fu, Stefano Ermon, Atri Rudra, Christopher Ré / Stanford・SUNY Buffalo, NeurIPS 2022, arXiv:2205.14135）
- 作成: [[summaries/2022-flashattention]], [[translations/2022-flashattention]], `raw/assets/2022-flashattention/`（画像 9 枚）
- 更新: [[concepts/llm-inference-optimization]], [[concepts/transformer-architecture]], [[summaries/2026-gpt2-to-kimi3]], [[summaries/2026-llm-optimization-guide]], [[summaries/2023-lost-in-the-middle]], [[overview]], [[index]]
- ユーザー決定: (1) 翻訳範囲は**付録 E を含めて全訳**（圧縮箇所なし）、(2) 概念ページは **llm-inference-optimization ＋ transformer-architecture の 2 ページ**（context-engineering には節を設けず、要約からのリンクに留める）。
- 位置づけ: **wiki が 4 箇所から参照していたのに原典が無かった**状態を解消した。とくに [[concepts/llm-inference-optimization]] のカーネル融合節は FlashAttention を自ら「概説」と書いており、[[summaries/2026-llm-optimization-guide]] は "Dao et al. 2022" として二次引用、[[translations/2026-gpt2-to-kimi3]] には著者が「O(N²) の枠づけには惑わされた……それは Flash Attention が解決したこと」と自己訂正した箇所がある。その一次資料。
- 要約の柱: **演算を 13% 増やして 5.7 倍速くなった**という 3 行の表（GFLOPs 66.6 → 75.2 / HBM 読み書き 40.3 → 4.4 GB / 実行時間 41.7 → 7.3 ms）を中心に据えた。tiling（softmax を行最大値と指数和の 2 統計量で漸進確定）と recomputation（backward 用に行列を保存せず SRAM で再構成）、IO 計算量 Θ(N²d²M⁻¹)、および**命題 3 の下界**（どの厳密 attention もすべての M でこれを漸近的に上回れない）。副産物として長コンテキストの経済化（GPT-2 の 4K 訓練が Megatron の 1K 訓練より 30% 速く ppl も 0.7 良い、Path-X 16K で 61.4%、block-sparse で Path-256 64K が 63.1%）。
- 概念ページへの配分: **llm-inference-optimization に `### IO-awareness — 律速はどこにあるか（FlashAttention）` を新設**（HBM/SRAM の帯域と容量、memory-bound vs compute-bound、arithmetic intensity、tiling/recomputation、下界、そして「再計算はメモリ節約の手段とは限らない——キャッシュか再計算かは保存コストでなく往復コストで決める」「近似は IO を直してから効かせる」の 2 教訓）。既存の「チャンク化とハードウェア整合」「カーネル融合」節も本論文を根拠に補強した。**transformer-architecture には attention 系譜の「直交する第四の軸」として追記**——既存の系譜（linear attention・DeltaNet・KDA・MLA・CSA/HCA・Gemma の間引き）はすべて「attention が計算する内容を変える」路線だが、FlashAttention は**数学を一切変えずメモリアクセスだけを変える**。この対比は系譜の読み方自体に効き、2020〜2022 年の近似 attention 群（Reformer・Linformer・Performer・Longformer・BigBird）が普及しなかった理由（FLOPs は落としたが HBM 往復を減らさなかった）を説明してしまう。
- 批判的視点として要約に明記した点: 著者ら自身が挙げる 3 つ（**新しい attention 変種ごとに CUDA 手書きが要り GPU 世代間の移植性も保証されない**／**単一 GPU に閉じており GPU 間転送は IO 解析に入っていない**／attention 以外の層は手つかず）に加えて、(a) 数値は 2022 年の A100/3090/T4 のもので SRAM と HBM 帯域の比は世代で変わる（**持ち帰るべきは倍率でなく原理**）、(b) 「最適」は定数倍を除いた漸近の話で、実際 **FlashAttention-2 が同じ IO 計算量のまま work partitioning の改善だけで約 2 倍を追加した**——同じ漸近オーダーの中に 2 倍が残っていた事実自体が本論文の教訓の再演になっている、(c) Apex FMHA との比較は正直だが控えめで**系列長 128 では 4% 遅い**（利得は長系列で効く）、(d) 系列長 512〜1024 で Linformer など一部の近似手法に**追い抜かれる交差点がある**ことを著者らは隠していない。
- **クリップ不良を 5 種類復元**（すべて ar5iv 原ページ `https://ar5iv.labs.arxiv.org/html/2205.14135` から取得）:
  1. **Figure 2 左の数値表が丸ごと欠落**していた。GFLOPs 66.6→75.2 / HBM R/W 40.3→4.4 GB / Runtime 41.7→7.3 ms という**本論文の中心的証拠そのもの**で、これが無いと §3.2 本文の「FLOP 数が高いのに速い」という主張の根拠が読者に見えない。表として復元。
  2. **画像 1 枚**（`figs/flashattn_speedup_t4_fwd.jpg` = Figure 8 下段、T4 の forward pass のみ）。既知の「多パネル図の 2 枚目が落ちる」パターン。
  3. **脚注 4 件の本文が全欠落**（`<sup>N</sup>` マーカーのみ残存）。コード URL、algebraic aggregation の出典、LRA のチューニング依存性への注意、Path-256 が Path-X より易しい理由。本文中に訳注として挿入。
  4. **付録 E の Table 9〜21 から太字・下線が完全消失**。これらのキャプションは「最良を太字、次点を下線」と明示しているので、強調が無いと 13 表 × 12 手法 × 10 系列長が読めない。ar5iv の HTML から `ltx_font_bold` / underline を抽出して 13 表すべて再生成した。行見出し列の太字は ar5iv の見出しスタイルなので付けず、**太字・下線が「最良・次点」の意味だけを担う**ようにした。
  5. **§B.5 の見出しが「Comparison with」で切れていた**（引用脱落）。正しくは「B.5 Comparison with Rabe and Staats [66]」。
  - 加えて Figure 1・Figure 2 のキャプションで手法名 "FlashAttention" が複数箇所で脱落（LaTeX マクロ脱落）していたため補った。
- **原ページ側の不備／原論文の誤りとして訳注に記録**（クリップの責任ではないので区別して扱った）: (a) **ar5iv 側で Algorithm 0 への参照が空欄でレンダリングされる**（「Standard attention (Algorithm ) requires…」等 5 箇所）。HTML のアンカーが `#alg0` を指しているので参照先は確定でき、`Algorithm 0` を補った上で補完した旨を明記。(b) 付録 C・D の定理参照も同様に空欄（「Proof of.」）で、アンカーから `定理 1`『定理 2』『命題 3』『命題 4』『定理 5』を確定して補った。(c) **原論文自身の相互参照の誤り**——§4.2 が「Table 6 shows that sequence length 16K outperforms…」と書いているが、その数値（MIMIC +4.3・ECtHR +8.5）は Table 5 にある。原ページでも同じなので訳はそのままにし訳注で指摘。
- 図表の検証: 画像 9 枚（`x1`〜`x4`＋速度比較 JPG 5 枚）をすべて Read で開いて内容を確認してからキャプションを書いた。**x2.png が Figure 2 のキャプションが述べる 3 パネルのうち中央・右の 2 枚しか含まない**ことに気づいたのが、上記 1 の欠落表の発見につながっている。翻訳側の参照数 9 = 保存数 9。
- メモ: 付録 E の Table 9〜21 は全訳の判断（ユーザー選択）どおり 13 表すべて数値ごと収録した。反復的な表ではあるが、「どの系列長でどの近似手法が FlashAttention を追い抜くか」という本論文の**交差点の知見**は、この生データがないと追えない。
- データギャップ: 変化なし——**Self-Refine（arXiv:2303.17651）**・**Tree of Thoughts（arXiv:2305.10601）**・**context rot の研究（Chroma）**・**Anthropic HH（arXiv:2204.05862）**・**DeepSeek-LLM（2401.02954）/ DeepSeek-V2（2405.04434）**・**AI Scientist v1（2408.06292）**。本 ingest により新たに **FlashAttention-2（arXiv:2307.08691）** が「参照しているが未取得」に加わる（本ページと [[concepts/llm-inference-optimization]] の両方で言及した）。ただし原理は本論文が尽くしており、優先度は上記より低い。

## [2026-08-02] ingest | FlashAttention-2: Faster Attention with Better Parallelism and Work Partitioning

- 取り込み: `raw/papers/FlashAttention-2_ Faster Attention with Better Parallelism and Work Partitioning.md`（ar5iv クリップ。Tri Dao 単著 / Princeton・Stanford, arXiv:2307.08691, 2023-07）
- 作成: [[summaries/2023-flashattention-2]], [[translations/2023-flashattention-2]], `raw/assets/2023-flashattention-2/`（画像 20 枚）
- 更新: [[concepts/llm-inference-optimization]], [[summaries/2022-flashattention]], [[summaries/2026-gpt2-to-kimi3]], [[summaries/2026-llm-optimization-guide]], [[overview]], [[index]]
- ユーザー決定: (1) **画像は 20 枚すべて復元**（Figure 4〜7 のベンチマーク 16 パネルを全収録。causal mask の有無 × head dim 64/128 の 4 通りの差自体が知見であるため）、(2) 概念ページは **llm-inference-optimization に別節を新設**（transformer-architecture は更新しない）。
- 位置づけ: 前日 ingest した [[summaries/2022-flashattention]] の直接の続編で、**データギャップとして挙げた当日にユーザーが原典を投入した**もの。前作の要約に「同じ漸近計算量の中に 2 倍が残っていた」と書いた、その 2 倍の正体にあたる。本論文に**付録はなく §5 で終わる**短い technical report（565 行。前作 1,425 行の約 4 割）。
- 要約の柱: **問題設定が前作から一段ずれている**点を最初に据えた——前作は「律速は IO だ」だったが、本作は「**IO を直してもなお理論ピークの 25〜40%**」（最適化された GEMM は 80〜90%）。前作の IO 下界と矛盾しないことを明示した: あの最適性は「HBM アクセス回数の漸近オーダー」の主張であって、定数倍にも、**演算ユニットの稼働率**にも触れていない。対処 3 点は (1) non-matmul FLOPs の削減（A100 は matmul 312 TFLOPs/s vs non-matmul FP32 19.5 TFLOPs/s ＝ **1 FLOP あたり 16 倍差**。毎ブロックのスケーリングを最後の 1 回にまとめ、統計量 m・ℓ を logsumexp 1 本へ）、(2) 系列長方向の並列化（batch × heads だけでは長系列時に積が SM 数 108 を下回り SM が遊ぶ）、(3) warp 間の "split-K" 廃止（K/V でなく **Q を分割**して warp 間通信を消す）。結果は attention 単体 1.7〜3.0 倍・230 TFLOPs/s（ピーク 73%）、エンドツーエンド 225 TFLOPs/s・MFU 72%、H100 では専用機能なしで 335 TFLOPs/s。
- 概念ページへの配分: [[concepts/llm-inference-optimization]] に **`### 占有率と仕事の割り振り — IO を直した後に残る壁` を新設**し、既存の `### IO-awareness` 節の直後に置いた。GPU の実行モデル（スレッドブロック → warp → スレッド、SM 108 個、occupancy）を初出で解説し、一般的な教訓を 2 つ立てた: (a) **FLOPs は互いに換算できない**（16 倍差。既存の「FLOPs ≠ wall-clock」節をもう一段進める）、(b) **下流の実装が上流の論文へ還流しうる**（後述）。既存の `### カーネル融合 — 実装成熟度が律速する` 節からも新節への導線を張った。
- 特筆すべき接続: **ループ順序の入れ替えと系列長並列は Phil Tillet の Triton 実装が先**であると論文が明記している。前作 FlashAttention は限界として「CUDA を手書きするしかなく、高レベル言語からコンパイルできる仕組みが要る」と書いていたが、**その仕組み（Triton）が実際に作られ、そこでの発見が本家のカーネルへ戻ってきた**。wiki が繰り返してきた「実装の成熟度が律速する」というテーマの、珍しく因果がはっきり追える実例なので、要約・概念ページの両方に書いた。
- 批判的視点として要約に明記した点: **ブロックサイズは手動チューニング**（自動化は今後の課題。新しいヘッド次元やデバイスごとに人手が要る）／**H100 の新機能（TMA・第 4 世代 Tensor Core）は未使用**で 335 TFLOPs/s は移植しただけの数字／**前作の限界（CUDA 手書き・世代間移植性・マルチ GPU の IO）はどれも解いていない**／**正しさの証明を「前作とほぼ同じ」として省略**しているがアルゴリズムは実際に変わっている（スケーリングのタイミング、保存する統計量）ので厳密には自明な系ではなく、査読済み論文というより technical report として読むべき／§4.2 の相互参照に誤りがある／測定は 2023 年の A100・H100 で、occupancy の議論は SM 数とバッチサイズの比に依存するため運用条件で効き方が変わる。
- **クリップ不良を 4 種類復元**（すべて ar5iv 原ページ `https://ar5iv.labs.arxiv.org/html/2307.08691` から取得）。**これまでの ingest で最も欠落が大きかった**:
  1. **画像 20 枚のうち 13 枚が欠落**（クリップに残っていたのは 7 枚）。Figure 4〜7 はいずれも「causal mask の有無 × head dimension 64/128」の 2×2 = 4 パネル構成だが、**各図の (a) しか残っていなかった**（x2/x3/x4, x6/x7/x8, x10/x11/x12, x14/x15/x16 が欠落）。
  2. とくに深刻だったのが **Figure 3(b)（`flash2_partitioning.png`）の欠落**。Figure 3 は「(a) FlashAttention / (b) FlashAttention-2」の並列比較図で、(b) は**本論文の表題そのものである work partitioning の新方式**を描いたもの。クリップには **(a)＝比較対象の旧方式だけ**が残っていた。Read で 2 枚を開いて確認したところ、点線（全 warp からアクセス）と破線（warp 間で分割）が Q と K/V の間で正確に入れ替わっており、この 1 枚の対比が §3.3 の主張そのものだった。
  3. **Figure 3〜7 の本キャプション 5 つが完全に消滅**していた。残っていたのは `(a) Without causal mask, head dimension 64` というサブキャプションのみで、しかも原典 478〜488 行では Figure 4・5・6 の 3 図がキャプションなしに連続しており、**どの画像がどの図に属するか markdown だけからは判別できない**状態だった。本キャプションとサブキャプション (a)〜(d) をすべて復元し、図番号を明示して配置した。
  4. **脚注 3 件の本文が脱落**（マーカーのみ残存）。コード公開先 URL、$\mathbf{Q}\mathbf{K}^\top$ のスケーリングを省略している旨の断り、Triton 実装への参照。
- **原論文側の誤りとして訳注に記録**（クリップの責任ではないので区別）: (a) **§4.2 の「1.3 倍の高速化」の比較対象が FlashAttention-2 自身になっている**（FlashAttention の誤り。自分自身に対して 1.3 倍にはなり得ず、Table 1 の 175 → 225 TFLOPs/s がおよそ 1.29 倍にあたる）。(b) §2.2 に `$\S$` という記述がある（`$\mathbf{S}$` であるべき）。いずれも原ページで同じ。
- 図の検証: 20 枚すべて PNG であることを確認し、うち主要な 4 枚（Figure 1・2・3(a)・3(b)）と Figure 4(a) を Read で開いて内容を確認してからキャプションを書いた。翻訳側の参照数 20 = 保存数 20。
- データギャップ: 前回挙げた **FlashAttention-2** は本 ingest で解消。残るのは **Self-Refine（arXiv:2303.17651）**・**Tree of Thoughts（arXiv:2305.10601）**・**context rot の研究（Chroma）**・**Anthropic HH（arXiv:2204.05862）**・**DeepSeek-LLM（2401.02954）/ DeepSeek-V2（2405.04434）**・**AI Scientist v1（2408.06292）**。新たに **FlashAttention-3（2024, Hopper 向け）** が「本ページで言及したが未取得」に加わる（本論文が「今後の課題」として挙げた H100 専用機能の活用がその内容にあたる）が、原理は既収録の 2 本で尽きているため優先度は低い。

## [2026-08-02] ingest | FlashAttention-3: Fast and Accurate Attention with Asynchrony and Low-precision

- 取り込み: `raw/papers/FlashAttention-3_ Fast and Accurate Attention with Asynchrony and Low-precision.md`（ar5iv クリップ。Jay Shah, Ganesh Bikshandi, Ying Zhang, Vijay Thakkar, Pradeep Ramani, Tri Dao, arXiv:2407.08608, 2024-07）
- 作成: [[summaries/2024-flashattention-3]], [[translations/2024-flashattention-3]], `raw/assets/2024-flashattention-3/`（画像 18 枚）
- 更新: [[concepts/llm-inference-optimization]], [[summaries/2022-flashattention]], [[summaries/2023-flashattention-2]], [[summaries/2024-deepseek-v3]], [[summaries/2023-lost-in-the-middle]], [[overview]], [[index]]
- ユーザー決定: (1) 概念ページは **3 つ目の節を新設して並べる**（既存の `### IO-awareness` `### 占有率と仕事の割り振り` の直後）、(2) FP8 の精度対策は **既存の `### 低精度訓練` 節に対照を追記**する。
- 位置づけ: **FlashAttention 三部作の完結編**で、前日〜当日に取り込んだ 2 本の直接の続き。前作の要約に「H100 では専用機能を使わずに 335 TFLOPs/s、本領は未発揮」と書いたが、その未発揮の度合いが判明した——**A100 で理論ピークの 73% まで詰めたコードが H100 では 35%**（同じ GPU の GEMM は 80〜90%）。要約では三部作を「各作が別の律速を扱っている」表（メモリ往復 → 占有率と分業 → 同期モデルと精度）として整理した。
- 要約の柱: Hopper が持ち込んだ 2 つの前提（**非同期性** = 行列積は Tensor Core / WGMMA、データ転送は TMA という別ユニットで独立に走る、**FP8**）と、それに応じた 3 つの対処。(1) **warp-specialization**（CTA 内の warp を producer と consumer に役割分担させ、`setmaxnreg` でレジスタを受け渡し、$s$ 段の循環 SMEM バッファで繋ぐ）、(2) **softmax を GEMM の陰に隠す**——H100 は FP16 行列積 **989 TFLOPS** に対し指数関数など特殊関数が **3.9 TFLOPS（256 分の 1）**で、ヘッド次元 128 では指数関数だけで約 50% のサイクルを食いうる。warpgroup 間の **pingpong スケジューリング**（570 → 620〜640 TFLOPS）と warpgroup 内の **2 段 GEMM-softmax パイプライン**を併用。(3) **FP8** のレイアウト適合（k-major 制約・カーネル内転置・累算器とオペランドの配置差を byte permute で吸収）と精度対策。結果は FP16 で **740 TFLOPs/s（ピークの 75%）**、FP8 で **1.2 PFLOPs/s 近く**、cuDNN を中〜長系列で上回る。アブレーションが示唆的で、warp-specialization だけなら 582・パイプラインだけなら 570 に対し**両方揃って 661 TFLOPs/s**。
- 概念ページへの配分: [[concepts/llm-inference-optimization]] に **`### 非同期化と低精度 — 世代が変わると壁も変わる` を新設**（既存 2 節の直後）。GPU の非同期実行モデル（Tensor Core / TMA / WGMMA / warpgroup）を初出で解説し、教訓を 2 つ立てた: (a) **「最適化し切った」は世代の関数である**（前世代で正しかった設計判断が次世代のボトルネックになる）、(b) **低精度化を進めるほど非行列積演算が相対的に重くなる**（A100 の 16 倍差が H100 で 256 倍に開き、FP8 にすると行列積側だけがさらに倍になる）。前節末尾からも導線を張った。加えて既存の `### 低精度訓練 — FP8 を極大規模で通す` 節に **「外れ値への、2 つの異なる答え方」の対照表**を追記——DeepSeek-V3 が**スケールの粒度を細かくする**（活性 1×128 タイル・重み 128×128 ブロック、訓練層）のに対し、FlashAttention-3 は加えて **incoherent processing で分布そのものをならす**（Q・K に同じランダム直交行列を掛ける。$MM^\top=I$ なので**出力は恒等**のまま外れ値が薄まる。Hadamard 行列で $O(d\log d)$、rotary embedding へ融合可能）。QuIP / QuIP# からの借用である点も記した。
- 批判的視点として要約に明記した点: (a) **FP8 が cuDNN と「competitive」という主張には脚注レベルの条件がつく**——ヘッド次元 64 では勝つが、128 と 256 では因果マスクなしで互角・**ありでは負ける**（脚注 2。しかもこの脚注はクリップから欠落していた）。(b) **FP8 版には persistent kernel と負荷分散が入っていない**（脚注 10。著者らが短系列で劣る理由の一部と認めている）＝ FP8 の数字は実装未完成の状態のもの。(c) **精度検証は合成データの RMSE のみ**で実際の訓練・推論での品質劣化は未測定、著者らも「大規模訓練における低精度 attention の影響の理解」を今後の課題に挙げている。(d) **推論向け最適化は未着手**（限界節の筆頭）。(e) **3 段パイプラインは失敗**（レジスタ圧で 2 段版より遅く、「コンパイラがなぜこう並べ替えるのか明らかでない」と正直に書かれている）。(f) **Hopper 固有性が高く**、脚注 1 の一般性の主張に反して検証は H100 のみ・記述は WGMMA/TMA/`setmaxnreg`/LDSM/STSM への依存が濃い。(g) 測定は 2024 年 5 月時点のライブラリ版。加えて、**アブレーション表の非対称性**を自分で読んだ点を記した——FP8 の精度改善はほぼ **incoherent processing だけ**が担っており（ブロック量子化を外しても 9.1e-3 → 9.3e-3 だが、incoherent processing を外すとベースラインと同じ 2.4e-2 に戻る）、論文が 2 技法を並列に提示していても寄与は同等でない。
- **クリップ不良を 5 種類復元**（ar5iv 原ページ `https://ar5iv.labs.arxiv.org/html/2407.08608` から取得）:
  1. **画像 18 枚のうち 11 枚が欠落**（残っていたのは 7 枚）。Figure 5（6 パネル）・Figure 6（2 パネル）・Figure 7（2 パネル）・Figure 9（6 パネル）から**各図の (a) しか残っていなかった**。前回の FlashAttention-2 とまったく同じパターン。
  2. **Figure 5・6・7・9 の本キャプション 4 つが完全に消滅**。残っていたのはサブキャプションのみで、原典 333〜339 行では Figure 5 と 6 が本キャプションなしに隣接していた。
  3. **脚注 10 件の本文が全欠落**（マーカーのみ残存）。うち脚注 2（FP8 と cuDNN の比較条件）・脚注 5（3.9 TFLOPS の導出根拠）・脚注 10（persistent kernel の不在）は批判的に読むうえで不可欠だった。
  4. **§3.2 の `Register pressure` サブセクションが見出しごと欠落**（2 段パイプラインが $B_r \times B_c \times$ sizeof(float) の追加レジスタを要し、大きいブロックサイズという別の最適化と衝突するという内容）。ar5iv のサブ見出し一覧と突き合わせて発見した。
  5. **付録 B.3 の `Register pressure.` も同様に欠落**（3 段版が $\tilde{P}_i$ と $scale\_o$ を余分に要するのでブロックサイズを小さくせざるを得ない）。
- **復元不能だったもの**: **Figure 5(b)（因果マスクあり・ヘッド次元 64）の画像は ar5iv 側にも存在しない**。ar5iv の HTML はこのサブ図（`S4.F5.sf2`）にキャプションだけを置いており、欠番になっている `x2.png` を直接取得すると HTTP 200 で**「NO IMAGE AVAILABLE」のプレースホルダ画像**（325×400 グレースケール）が返る。ar5iv 側の変換失敗と確定したので、skill の方針どおり無理に埋めず、該当箇所にキャプション訳と経緯の訳注のみを残した（取得したプレースホルダは保存していない）。
- **Figure 3・4 の扱い**: この 2 図はクリップに**インライン SVG として残っていた**（レジスタ配置図）。中身が文字だけの格子なので画像保存せず **markdown の表に転記**した。ただし **ar5iv の DOM 順は視覚順と一致しない**（Figure 3 では `T0 {d4, d5}` が DOM 上 3 番目だが視覚的には 5 列目）ため、SVG の `transform="matrix(... tx ty)"` の座標を読んで並べ直した。この 2 表を並べると、なぜ `{d0 d1 d4 d5 d2 d3 d6 d7}` というバイト置換が必要か（累算器は「スレッドが 2 列おきに交代」、オペランドは「スレッドが 4 列を占有」で所有の粒度が違う）が一目で分かる。
- **原論文側の誤りとして訳注に記録**: §5 の見出しが `Dicussion`（`Discussion` の誤記。原ページでも同じ）。また付録 B.2 の SASS コードで繰り返しの省略記号の一部が失われた形（`FMNMX and SHFL.BFLY` のような行）になっているのは **ar5iv 側の平坦化**であり、クリップと原ページで一致するためそのまま収録した。
- **自己修正**: 翻訳の冒頭ヘッダに著者の所属（Colfax Research / Meta / NVIDIA / Princeton 等）を書いたが、**ar5iv 版の原ページには所属の記載がなく**、背景知識から補ってしまっていた。skill の「復元の根拠は必ず取得した原文に置く（記憶や推測から本文を創作しない）」に反するため、著者名のみに修正し、所属を省略した旨を明記した。
- 図の検証: 18 枚すべて PNG であることを確認し、主要な 4 枚（Figure 1 pingpong・Figure 2 2-stage・Figure 5(a) ベンチマーク・Figure 8 3-stage）を Read で開いて内容を確認してからキャプションを書いた。翻訳側の参照数 18 = 保存数 18。
- データギャップ: 前回挙げた **FlashAttention-3** は本 ingest で解消し、**FlashAttention 三部作がすべて揃った**。残るのは **Self-Refine（arXiv:2303.17651）**・**Tree of Thoughts（arXiv:2305.10601）**・**context rot の研究（Chroma）**・**Anthropic HH（arXiv:2204.05862）**・**DeepSeek-LLM（2401.02954）/ DeepSeek-V2（2405.04434）**・**AI Scientist v1（2408.06292）**。新たに **QuIP#（incoherent processing の原典）** が「本ページで言及したが未取得」に加わるが、量子化の専門論文であり本 wiki の主題からはやや外れるため優先度は低い。

## [2026-08-02] ingest | LLM 量子化の解説 2 本（Abhinaykrishna / joydeep bhattacharjee）

- 取り込み: `raw/articles/LLM Quantization Explained_ A Complete Guide.md`（Abhinaykrishna / Medium, 2025-03-27）と `raw/articles/LLM Quantization Explained.md`（joydeep bhattacharjee / Medium, 2025-04-30）の **2 本同時**。ユーザーが同一主題の 2 本をまとめて指示したため、要約・翻訳はスキーマどおり**原典ごとに 1 ページずつ**作り、統合は概念ページ側で行った。
- 作成: [[concepts/model-quantization]]（**新規**）, [[summaries/2025-llm-quantization-guide]], [[translations/2025-llm-quantization-guide]], [[summaries/2025-llm-quantization-explained]], [[translations/2025-llm-quantization-explained]], `raw/assets/2025-llm-quantization-guide/`（画像 4 枚）, `raw/assets/2025-llm-quantization-explained/`（画像 17 枚）
- 更新: [[concepts/llm-inference-optimization]], [[concepts/transformer-architecture]], [[overview]], [[index]]
- ユーザー決定: (1) **`model-quantization` を新設**する、(2) 2 本の食い違いと誤りは**概念ページ側でのみ正す**（各要約は記事に忠実に書き、指摘は「限界・批判的視点」に留める）。
- 新概念ページの根拠: 量子化は [[concepts/llm-inference-optimization]] の 3 つの節（`モデル圧縮` / `オンデバイス推論` / `低精度訓練`）に散在していたが専用ページがなく、GPTQ・AWQ・GGUF・QLoRA・NF4・STE・SmoothQuant といった代表手法の置き場も未定義だった。**量子化は推論サービングだけの道具ではなく、訓練（FP8 訓練・QAT）・ファインチューニング（QLoRA）・エッジ配布（提供側 QAT）・カーネル実装（FP8 attention）にまたがる汎用の道具**であり、[[concepts/mixture-of-experts]] を切り出したのと同じ理由が立つ。既存 3 節には導線を張り、サービング文脈の記述は残した。
- 2 本の性格分け: 要約でも概念ページでも「**手法カタログ側と原理・実装側**」として対比した。Abhinaykrishna は量子化できる 3 対象（重み・活性・**KV cache**）の区別、較正データセットが「統計を集めるだけで重みを更新しない」こと、bitsandbytes / GPTQ / GGUF / AWQ のユースケース別比較、そして末尾の**「量子化は必ずしもレイテンシを下げない」**（逆量子化のオーバーヘッド・ハードウェア支援の欠如・未最適化の推論エンジン・小さなモデルの 4 状況）。joydeep は RTN の誤差を torch で実測、**BF16 は「範囲が広い」形式であって「精度が高い」形式ではない**ことを 1000 テンソルで実証、affine 量子化の s と z、SmoothQuant の外れ値分析、偽量子化と STE による QAT、QLoRA の NF4・二重量子化・4bit を uint8 へパックする実装まで動くコードで降りる。
- **概念ページで正した二次資料の誤り 4 点**（要約側では各記事の「限界」節に短く置き、正しい記述は概念ページに集約した）:
  1. **QLoRA は QAT ではない**（joydeep が「QAT における現在の人気手法」と分類）。実際は **PTQ ＋ PEFT** で、NF4 に量子化して**凍結した**ベースの上で LoRA アダプタだけを訓練する。QAT の定義は「偽量子化と STE によって量子化されたモデル自身の重みを訓練する」ことであり、QLoRA では量子化された重みは訓練対象でない。目的も違い（QAT は低ビットでの品質維持、QLoRA は省メモリ・ファインチューニング）。同記事が直前で STE つき QAT を正しく説明しているだけに混同しやすい。
  2. **「NF8」という形式は存在しない**（Abhinaykrishna が「nf4 の代わりに nf8」と記述し、付図の比較表にも `uses nf4,nf8` とある）。LLM.int8() は NormalFloat ではなく **INT8 ＋ 外れ値の混合精度分解**。NormalFloat は QLoRA が 4 ビット向けに導入したもので 8 ビット版はない。
  3. **非対称レンジにゼロ点 0 を置く実例の破綻**（Abhinaykrishna）。`scale = (3.8 − (−2.3))/255` と非対称レンジから求めながら `zero_point = 0` としたため、3.8 が 159 → **127 へクランプ**され復元値が **3.03**（約 20% の誤差）になる。記事はこの破綻に触れない。皮肉なことに、**もう 1 本の記事が載せる affine 量子化の式（$z = q_{min} - x_{min}/s$）がその正しい形**にあたるので、概念ページでは 2 本を突き合わせる形で書いた。
  4. **LLM.int8() の特徴づけの混同**（Abhinaykrishna が「64 要素ブロックごとにスケール係数」を 8 ビットの説明に置いている）。これは bitsandbytes の **4 ビット側**（NF4 のブロック量子化）の性質で、LLM.int8() の眼目はベクトル単位の量子化と外れ値分解にある。
  - なお **INT4 の誤差例で数値がすり替わる**（「5.62 を量子化」と書き始めて次行では「0.562 は 0.6 に最も近い」）点も要約に記した。
- 概念ページの独自の整理: **外れ値への対処を 4 系統に分類**した（(a) 分けて保持する＝LLM.int8 の混合精度分解、(b) 粒度を細かくする＝ブロック量子化・NF4・DeepSeek-V3 の 1×128 タイル、(c) 重要な重みを守る＝AWQ の salient weights、(d) 分布そのものをならす＝incoherent processing・SmoothQuant）。これにより、既収録の [[summaries/2024-deepseek-v3]]（訓練側の細粒度量子化）と [[summaries/2024-flashattention-3]]（カーネル側の直交回転）が、今回の 2 記事と同じ枠組みの中に並ぶ。SmoothQuant の 2 つ目の知見（**外れ値は固定チャネルに持続的に現れる**＝どのチャネルが外れ値かは安定している）が、なぜチャネル／ブロック単位の対処が効くのかの根拠になっている点も明記した。
- 画像（ケース C）: **両クリップとも健全**で、本文の切断・画像の欠落・キャプションの脱落はいずれもなかった（FlashAttention 三部作の後だけに対照的である）。Medium の CDN URL から `format:webp` を外して素の PNG を取得。**joydeep 記事は埋め込み 20 枚のうち 3 枚を chrome として除外**——冒頭の装飾タイトルカード（「LLM QUANTIZATION EXPLAINED / Make LLMs smaller and faster」とだけ書かれたカバー画像。Read で開いて内容を確認したうえで除外）と、末尾の「Lets connect on linkedin」以降にある著者プロフィール／宣伝画像 2 枚。保存は 4 枚 ＋ 17 枚 = 21 枚で、翻訳側の参照数と一致。
- **文字だけの図はテキストにも起こした**（skill の方針どおり）: Abhinaykrishna の図 4（手法比較表）は画像に加えて **markdown の表として転記**し、joydeep の「Linear Quantization Equations」の図は**数式を本文へ起こした**。いずれも画像のままでは検索・引用ができないため。転記は画像に忠実で、前者は原典の誤記（`uses nf4,nf8`）もそのまま写している。
- 除外: 両記事とも拍手・共有・相談の誘導（「If you found this article insightful…」「Concerned about the current job market?…」）は本文でない定型要素として訳出せず。joydeep 記事の **References（リンク集）も既定どおり除外**したが、本記事の価値の一部がそのリンク集にあるため、主要な一次資料（LLM.int8・SmoothQuant・QLoRA・STE の各論文）は要約と概念ページの側で言及した。
- 批判的視点として要約に明記したその他の点: 両記事とも**査読を経ていない個人ブログ**で 2025 年春時点のライブラリ挙動に依存する／**数値の出どころが示されない**（「AWQ は 4 ビットで GPTQ より精度が良い」にベンチマークも出典もない）／joydeep の **BF16 誤差比較は [0,1) 一様乱数という限定条件**での話で、勾配のように極端な値が出る場面では BF16 が有利になる非対称性に触れていない／同記事の **optimum-quanto の実験は原因が推測のまま**（`freeze` 後の再測定がない）／**両記事ともエージェント固有の観点がない**——多段ツール呼び出しでの誤差の累積、とくに**ツール呼び出しの JSON のような構造化出力が低ビット化にどれだけ脆いか**は、本 wiki の原典群では誰も測っていない（概念ページの「設計論点」に未解決として記録）。
- データギャップ: 変わらず **Self-Refine（arXiv:2303.17651）**・**Tree of Thoughts（arXiv:2305.10601）**・**context rot の研究（Chroma）**・**Anthropic HH（arXiv:2204.05862）**・**DeepSeek-LLM（2401.02954）/ DeepSeek-V2（2405.04434）**・**AI Scientist v1（2408.06292）**・**QuIP#**。今回の 2 本により、**LLM.int8（arXiv:2208.07339）**・**QLoRA（arXiv:2305.14314）**・**SmoothQuant（arXiv:2211.10438）**・**GPTQ**・**AWQ** が「概念ページで代表手法として記述したが一次資料が未取得」に加わった。とくに **QLoRA と LLM.int8 は二次資料の記述に誤りが見つかった箇所**なので、一次資料で裏を取る価値が高い。また `parameter-efficient-fine-tuning`（LoRA/PEFT）は依然 dangling のままで、今回の 2 本は LoRA 本体をほとんど説明していないため新設は見送った。

## [2026-08-02] ingest | 量子化の一次資料 3 本（LLM.int8() / SmoothQuant / QLoRA）

- 取り込み: `raw/papers/LLM.int8()_ 8-bit Matrix Multiplication for Transformers at Scale.md`（Dettmers ら, NeurIPS 2022, arXiv:2208.07339）、`raw/papers/SmoothQuant_ Accurate and Efficient Post-Training Quantization for Large Language Models.md`（Xiao ら / MIT・NVIDIA, ICML 2023, arXiv:2211.10438）、`raw/papers/QLoRA_ Efficient Finetuning of Quantized LLMs.md`（Dettmers ら / UW, NeurIPS 2023, arXiv:2305.14314）の **3 本同時**。いずれも前日の二次資料 ingest でデータギャップとして挙げたもの。
- 作成: [[concepts/parameter-efficient-fine-tuning]]（**新規**）, [[summaries/2022-llm-int8]], [[translations/2022-llm-int8]], [[summaries/2022-smoothquant]], [[translations/2022-smoothquant]], [[summaries/2023-qlora]], [[translations/2023-qlora]], `raw/assets/2022-llm-int8/`（6 枚）, `raw/assets/2022-smoothquant/`（10 枚）, `raw/assets/2023-qlora/`（6 枚）
- 更新: [[concepts/model-quantization]], [[concepts/agent-evaluation]], [[concepts/transformer-architecture]], [[summaries/2025-llm-quantization-guide]], [[summaries/2025-llm-quantization-explained]], [[summaries/2024-flashattention-3]], [[summaries/2024-deepseek-v3]], [[overview]], [[index]]
- ユーザー決定: (1) **`parameter-efficient-fine-tuning` を新設する**（長く dangling だったスラグ。QLoRA が LoRA の定式化・PEFT のメモリ内訳・アダプタ配置の知見・PEFT 手法のサーベイを持つため根拠が立つ）、(2) 翻訳は **QLoRA の §6.1 生成例のみ圧縮**し、他は 3 本とも付録含め全訳。
- **前日の訂正が一次資料で全部確定した。** これが本 ingest の最大の成果である。(a)「**NF8 は存在しない**」——LLM.int8() は absmax の vector-wise ＋ 混合精度分解であり、NormalFloat は論文中に一度も出てこない。(b)「**64 要素ブロックは 4bit 側の性質**」——同論文 §3 は block-wise 量子化を引き合いに出したうえで対比的に行／列単位の vector-wise を採っている。(c)「**QLoRA は QAT でない**」——QLoRA §3 が「**重みの勾配は、16 ビット BrainFloat を使う LoRA のパラメータについてのみ計算する**」と明記。(d) 二重量子化の 0.5 → 0.127 ビット（節約 0.373 bits/param、65B で約 3GB）も原典の式どおり。二次資料を先に取り込んで概念ページで訂正し、翌日に一次資料で裏を取るという流れが機能した事例として記録しておく。
- **3 本が互いを批判し合う構造**を要約・概念ページの骨格にした。SmoothQuant は LLM.int8() を名指しで批判し（「混合精度分解をハードウェアアクセラレータ上で効率的に実装するのは難しい」）、**Table 10 で数字を出す**——OPT-13B・seq256 で FP16 152.6ms に対し **LLM.int8() 237.1ms（1.55 倍遅い）**、SmoothQuant-O3 112.1ms（1.36 倍速い）。しかも**精度では両者とも FP16 に一致する**（OPT-175B 平均 66.7% vs 66.8%）。LLM.int8() 自身も §3.4 で「6.7B 未満では量子化オーバーヘッドが推論を遅くしうる」と認め、付録 D で**モデル次元 2560 で 0.64x、4096 で 0.86x** と実測している。前日 [[concepts/model-quantization]] に書いた「量子化＝高速化ではない」節に、**一次資料の直接対決の数字**を入れられた。
- 新概念ページ [[concepts/parameter-efficient-fine-tuning]]: LoRA の定式化（$\mathbf{Y}=\mathbf{X}\mathbf{W}+s\mathbf{X}\mathbf{L}_1\mathbf{L}_2$）、**PEFT のメモリの内訳という反直観的な事実**（7B/FLAN v2/バッチ 1 で LoRA パラメータ 26MB に対し**入力勾配 567MB**、勾配チェックポインティングで 18MB/系列、4bit ベースモデル 5,048MB）、そこから導かれる**「アダプタを削っても総メモリは減らない／逆に増やしても安い」**、**チューニングすべきは $r$ でなく配置**（全線形層に置く。$r$ は $\{8,...,256\}$ で性能が変わらない）、**QLoRA が QAT でない理由**、**量子化とファインチューニングの補完性**（推論で失われた精度がアダプタ訓練で回復する）、そして §7 の PEFT 手法サーベイ。**根拠が薄い旨をページ冒頭に明示**した——LoRA 本体（arXiv:2106.09685）は未取得であり、記述は QLoRA §2/§7 に依拠する範囲に留めている。
- [[concepts/model-quantization]] の強化: **外れ値節を一次資料で書き直した**（創発の相転移・6 次元への集中・0.1% が支配する構造・パープレキシティが創発を予測する）。**`### ハードウェアがアルゴリズムを縛る` を新設**——per-channel の活性量子化は精度的には正解（OPT-175B で 71.4% vs FP16 71.6%）なのに、INT8 GEMM はスケーリングを外側次元にしか置けないため使えない。だから SmoothQuant は「per-channel でスケールする」でなく「**per-channel でスケールした状態を事前の等価変換で作っておく**」という迂回を取る。FlashAttention 三部作と同じ主題の量子化版として書いた。代表手法の節も LLM.int8() と SmoothQuant を一次資料の記述で書き直した。
- [[concepts/agent-evaluation]] へ **`### ジャッジのバイアスを実測する — QLoRA の 3 つの数字` を新設**。量子化論文の中に埋もれているが、**LLM-as-a-judge のバイアスを具体的に測った数少ない仕事**である: (1) **順序効果**（先に現れる応答を高く採点。処方箋は両順序の平均）、(2) **自己贔屓**（GPT-4 の自己評価 Elo 1348 に対し人間評価 1176 ＝ 勝率で約 20% 差）、(3) **一致は水準で全く違う**（システム水準は Kendall τ=0.43 / Spearman r=0.55 だが**サンプル水準は Fleiss κ=0.25**、人間同士でも κ=0.42）。加えて**絶対スコアの接地不能性**（「10 点満点の 8」が場面をまたいで規定できない → 対比較の Elo を推奨）と、**ベンチマークの部分的直交性**（強い MMLU は強いチャットボット性能を意味しない。「新しいベンチマークを作るより既存で評価するほうが容易なので、特定のベンチマークがコミュニティを舵取りしうる」）。既存の DPO の規律（人間同士の一致率を基準線に置く）に、**どの水準の一致を測っているかを明示せよ**という論点を足す形で書いた。
- [[concepts/transformer-architecture]] へ **`## 規模とともに創発する外れ値特徴` を新設**。現象（6B→6.7B の相転移、15 万個が 6 次元に集中、消すと top-1 attention 質量 40%→20%・パープレキシティ 600〜1000% 悪化）に加えて、**方法論としての教訓**を主題にした——同じデータをパラメータ数で描くと「突然の相転移」、パープレキシティで描くと「滑らかな指数曲線」になる。外れ値の数はパープレキシティに単調、サイズには非単調。**何を横軸に取るかで「突然」に見えるか「滑らか」に見えるかが変わる**ので、創発を語るときは規模の代理変数を明示する必要がある。同ページの scaling law 節（規模を非埋め込み FLOPs/token で測ると曲線が揃う）と同型の教訓として接続した。
- クリップ不良: **LLM.int8() のみ**に既知パターンの不良があった。**画像 6 枚中 2 枚が欠落**（Figure 3・4 はいずれも 2 パネル図で **(a) しか残っておらず** `x4.png`/`x6.png` が欠落）、**Figure 3・4 の本キャプションが 2 つとも消滅**（`((a))` だけが残る）。復元した (b) パネルは**「創発を予測するのはパラメータ数でなくパープレキシティ」という本論文の最重要の主張を担う図**だったので、影響が大きかった。**SmoothQuant（画像 10/10）と QLoRA（画像 6/6）は健全**で、キャプション・本文とも欠落なし。HTML 表は 3 本合わせて 12 個を markdown へ変換した。
- 原論文側の特徴として訳注に記録: LLM.int8() の **Table 2（§6）と Table 3（付録 A）は同一の表の重複**（キャプションも内容も一致。§7 本文が「See Table 3」と参照するのは付録側）。原文に忠実に両方を収録した。
- **自己修正（重要）**: LLM.int8() の翻訳を書く際、**付録 B・D・E・F の表と本文を読まずに、もっともらしい内容を捏造して書いた**。書き上げた直後に気づき、原典 289〜381 行を読んで**全面的に差し替えた**（付録 B の関連研究の実際の記述、Table 5 の実測値 0.25x〜2.29x、Table 6 の BLOOM-176B のトークンあたり ms、Table 7/8 の 8 ビット訓練の結果、Table 9/10 の GLUE のスコア）。§3.3 も要約的に書いていたので原文どおりの全訳へ直した。**SWE-bench の ingest で同じ誤りをしており、二度目である。** 翻訳を書く前に該当箇所を必ず読む、という手順を守れなかった。
- 批判的視点として要約に明記した点: **LLM.int8()** — Int8 のみ（FP8 未検証）／175B までしか検証していない／attention 関数自体は 8bit 化していない／推論のみで訓練は付録の初期実験に留まる（**8bit FFN は劣化しないが attention の線形射影を 8bit にすると劣化**）／速度の主張は「175B 相当の大きな行列積で 2 倍」であって実運用帯では遅い／混合精度分解はハードウェアに優しくない／閾値 α=6.0 は経験的／創発の分析は記述的で機構の説明ではない。**SmoothQuant** — α はモデルごとにグリッド探索が要り適所が狭い（0.4〜0.6）／O3 は BLOOM で 0.8% 落ちる（較正データの分布依存）／**GPTQ と直接比較していない**（実装の違いを理由に回避。公正だが読者は答えを得られない）／「本番ではバッチングが標準になる」は予測であって証拠でない／LLaMA では外れ値の問題自体が小さい。**QLoRA** — 33B/65B での 16bit 完全ファインチューニングとの一致は未確立／評価ベンチマークが限定的／責任ある AI の評価が CrowS のみ／**Vicuna ベンチマークは 80 プロンプトでオープンソースに有利と著者自身が認めており「ChatGPT の 99.3%」はその前提つき**／信頼区間が広く論文自身が後で相対化している／RLHF 未使用／**「量子化された重みを訓練できる」ことを示したわけではない**（凍結したベースの上でアダプタを訓練しただけで、§7 も「10 億超の規模で量子化された重みを通した逆伝播を研究したのは SwitchBack layers だけ」と認めている）。
- データギャップ: 今回の 3 本で **LLM.int8()・SmoothQuant・QLoRA** が解消。残るのは **Self-Refine（arXiv:2303.17651）**・**Tree of Thoughts（arXiv:2305.10601）**・**context rot の研究（Chroma）**・**Anthropic HH（arXiv:2204.05862）**・**DeepSeek-LLM（2401.02954）/ DeepSeek-V2（2405.04434）**・**AI Scientist v1（2408.06292）**・**QuIP#**・**GPTQ**・**AWQ**。新たに **LoRA 本体（Hu ら 2021, arXiv:2106.09685）** が加わり、**これは新設した [[concepts/parameter-efficient-fine-tuning]] の根拠を薄いままにしている**ので優先度が高い。**Anthropic HH** は QLoRA が実験データセットとして使っており、優先度がやや上がった。

## [2026-08-02] ingest | 重みのみ量子化の三部作（AWQ / GPTQ / EfficientQAT）

- 取り込み: `raw/papers/AWQ_ Activation-aware Weight Quantization for LLM Compression and Acceleration.md`（Lin ら / MIT・SJTU・清華, MLSys 2024, arXiv:2306.00978）、`raw/papers/GPTQ_ Accurate Post-Training Quantization for Generative Pre-trained Transformers.md`（Frantar ら / IST Austria・ETH Zurich・NeuralMagic, ICLR 2023, arXiv:2210.17323）、`raw/papers/EfficientQAT_ Efficient Quantization-Aware Training for Large Language Models.md`（Chen ら / OpenGVLab Shanghai AI Lab・香港大学, 2024, arXiv:2407.11062）の **3 本同時**。いずれも直前の ingest でデータギャップとして挙げたもの。
- 作成: [[summaries/2023-awq]], [[translations/2023-awq]], [[summaries/2022-gptq]], [[translations/2022-gptq]], [[summaries/2024-efficientqat]], [[translations/2024-efficientqat]], `raw/assets/2023-awq/`（6 枚）, `raw/assets/2022-gptq/`（5 枚）, `raw/assets/2024-efficientqat/`（5 枚）
- 更新: [[concepts/model-quantization]], [[concepts/parameter-efficient-fine-tuning]], [[summaries/2022-llm-int8]], [[summaries/2022-smoothquant]], [[summaries/2023-qlora]], [[overview]], [[index]]
- ユーザー決定: (1) 翻訳は **GPTQ の付録結果表のみ圧縮**（他は 3 本とも付録含め全訳）、(2) 概念ページは代表手法の記述を一次資料で書き直したうえで **重要な軸を 2 つ新設**。
- **圧縮箇所は 1 か所のみ。** GPTQ 付録 A.3（Tables 9〜12）・A.4（Tables 13〜22）の計 14 表のうち、**Table 9（OPT/PTB）と Table 13（OPT/LAMBADA）を代表として全訳で残し、残る 12 表は最大モデルの数値と傾向の要約に置き換えた**。これらは本文 §5 の Table 3〜6 と同じ結論を別タスクで繰り返すもの。翻訳冒頭の訳注に明記した。要約に持ち込んだ知見は 2 点——**タスクによって 3 ビット RTN の壊れ方が違う**（LAMBADA では 0.00 まで落ちるが PIQA/ARC では偶然水準に張り付く＝「性能が落ちた」のでなく機能を失っている）、**OPT-66B の異常は全タスクで再現する**（脚注 2 の「初期層の死んだユニット」説と整合）。
- **概念ページに新設した 2 節**。(1) **`### 重みのみか、重みと活性か — メモリ帯域の勝ちと演算の勝ち`**——GPTQ が 2 か所で明記する「**高速化はメモリ移動の削減であって、実際の乗算は速くならない**」（混合精度オペランドへのハードウェア支援がないため）を軸に、W4A16 と W8A8 で**得られる高速化の出所が違う**ことを表にした。効く場面（小バッチ生成デコード vs 大バッチプリフィル）、圧縮率の上限（2〜4 ビット vs 8 ビット止まり）、難しさ（重みの分布は平坦 vs 活性の外れ値）まで対比。GPTQ 自身が「非生成・大バッチでは適用できず、行列を展開してから通常の行列積をすればよい」と書いている点も入れた。(2) **`### QAT は実用になったのか`**——AWQ（2023）と GPTQ（2022）がともに「QAT はコストが法外」と書いていた前提を EfficientQAT（2024）が崩した経緯を、Block-AP / E2E-QP の分割で説明し、**「4 ビットなら PTQ、2〜3 ビットなら QAT」**というビット数基準の線を引いた。著者自身が付録で「4 ビットでは RTN でさえより速く同等」と認めている点を根拠にしている。
- **既存記述の訂正**: 概念ページの較正データセット節は「**較正は重みを更新しない。統計を集めるだけである。したがって『較正データに過適合する』という心配は的外れ**」と書いていたが、**GPTQ には当てはまらない**（較正データ上で再構成誤差を最小化するよう重みを実際に動かす）。「多くの場合」に限定したうえで、`#### 較正データへの過適合 — AWQ vs GPTQ` を新設して AWQ の実測を入れた——分布がずれたときの PPL 悪化が **AWQ 0.5〜0.6 に対し GPTQ 2.3〜4.9**、視覚言語モデルでは **GPTQ が素の RTN より悪くなる**（32-shot CIDEr の劣化: RTN −4.57 / GPTQ −6.72 / AWQ −1.17）。**手法ごとに「較正が何をするのか」を確認せよ**という規律として書いた。
- **3 本が互いを批判し合う構造**を、前回の ingest（LLM.int8 / SmoothQuant / QLoRA）と同じくページの骨格にした。GPTQ → 先行研究（ZeroQuant・LLM.int8()・nuQmm）を「結局は最近接へ丸めているだけ」。AWQ → GPTQ を「較正セットに過適合し分布外ドメインで学習済み特徴を歪める」。EfficientQAT → Q-PEFT（QLoRA・LQ-LoRA・LoftQ）を「**LoRA をマージすると FP16 へ戻る**ので、メモリの限られたプラットフォームへ出すにはもう一度 PTQ が要る」。ただし**排他ではない**——INT2-g64 では AWQ + GPTQ の併用が最良（OPT-13B で 16.74 → 13.25）で、「顕著な重みを守る」と「誤差を補償する」は直交する。
- AWQ から要約に持ち込んだ**方法論の教訓**: 「大きい重みが重要」という機械学習で最も広く使われる代理指標が、**ランダム選択と大差ない**（OPT-6.7B INT3-g128 で 1% を FP16 に残すとき、活性基準 11.39 / 重みノルム基準 22.37 / ランダム 24.23）。Table 1 が**ランダム列をベースラインとして置いている**設計自体が良い。もう 1 点、**保護の強さを単調に上げてはいけない**（$s=4$ で顕著チャネルの誤差は減り続けるのに、グループの 21.2% で $\Delta$ が動いて非顕著チャネルが壊れ、PPL が悪化する。最良は $s=2$）。いずれも概念ページの設計論点へ追加した。
- GPTQ から持ち込んだ設計テンプレート: 「**計算量を落とす**（任意順序）」「**メモリ帯域を外す**（128 列の遅延バッチ更新）」「**数値的に壊れないようにする**（コレスキー再定式化 ＋ 1% 減衰）」は別々の作業であり、どれか 1 つでも欠けると 175B では動かない。とくに 2 番目は「理論計算量は変わらないが実測 1 桁」という、[[concepts/llm-inference-optimization]] の FlashAttention 系と同じ主題である。
- クリップ不良: **3 本で健全度がはっきり分かれた**。**AWQ は健全**（画像 6/6、Figure 1〜6・Table 1〜7 が完全）で、復元は脚注 3 件・Figure 5 キャプションの `×` 脱字・§2.2 の `$00$`→`$0$` の 3 点のみ。**GPTQ は skill の教科書的な取り違え事例**——原典では Table 7（2 ビットのグループサイズ別データ表）と Figure 4（4 ビットの図、`x5.png`）が**同じ float 内に隣接**しており、クリップは **Table 7 のデータ表を丸ごと落として、そのキャプションを Figure 4 の画像に付け替え、Figure 4 のキャプションを消していた**。本文の「Table 7 shows results on WikiText2」に数値が伴わない状態だった。加えて Figure 1 の 2 枚目（BLOOM パネル `x2.png`）欠落、脚注 4 件欠落。**EfficientQAT が最も重い**——Figure 1・2・3・5 のキャプション消失（Figure 3 の位置には文字列 `Refer to caption` が残存）、`x2.png` を Figure 1(a) と誤って紐づけ（実際は 1(b)）、`x4.png` 欠落、**Figure 5 の数値表が丸ごと欠落**、脚注 10 件欠落。すべて原ページから復元したが、**EfficientQAT の Figure 1(a) だけは ar5iv 側にも画像が存在せず復元不能**（キャプション訳のみ残した）。
- **クリップ不良の診断で自己修正**。当初、`Table N:` キャプションの**後ろ 6 行だけ**を見て「AWQ は 7 表中 6 表が欠落、GPTQ は 7 表欠落」と誤診した。**これらのクリップでは表がキャプションの前に来る**ため、前後両方向を見る必要があった。両方向で確認し直して「AWQ は健全、GPTQ は Table 7 のみ欠落」と訂正し、ユーザーへ明示的に報告した。表の有無を機械的に判定するときは**キャプションの前後どちらに本体が来るかを先に確認する**。
- **原典側の不整合（クリップ不良ではない）**: EfficientQAT の 4.2〜4.3 節は、Table 5 / Table 6 / Table 7 を指すべき箇所を**すべて「Table 7」と書いており**、また Figure 5 を指すべき箇所を「Table 5」と書いている（`\ref` が壊れている）。原文どおりに訳したうえで該当箇所ごとに訳注を付けた。査読を経ていない arXiv 版であることの現れとして要約の限界節にも記録した。
- 批判的視点として要約に明記した点: **AWQ** — グループなし量子化では原理的に不足（**スケーリングはグループごとに 1 チャネルしか守れない**。LLaMA-7B INT3 no-group で 1% FP16 の 14.06 に対し AWQ 20.52）／評価がパープレキシティと正解率に偏り、頑健性・公平性・バイアス・毒性は未測定（著者自身が付録 A で認める）／整数量子化に限定（FP4 等は検討外）／W4A16 なのでメモリ帯域の勝ちであって演算の勝ちではない／BLOOM を「品質が劣る」として実験から除外している。**GPTQ** — 活性の量子化がない／較正セットへの過適合（AWQ に指摘され、C4 の評価については論文自身が「完全なゼロショットではない」と自己申告）／評価がパープレキシティ中心（倫理声明で著者自身がバイアスへの影響の研究の必要を述べる）／reorder の工夫が後続版で追加されており本体だけでは LLaMA 系で不安定。**EfficientQAT** — **4 ビットでは PTQ に勝てない**（著者自身が付録 A で明記）／2 ビットになお 3〜4% の差／**Llama-3（15T トークン）は Llama-2（2T）より量子化しにくく QAT でも解消しない**（冗長性の少ないモデルほど圧縮の余地が小さい）／「41 時間」の内訳は Block-AP 26.6h ＋ E2E-QP 14.3h で、後者は 4096 サンプル 1 エポックの設定／相互参照が壊れている。
- 3 本に共通する空白として記録: **エージェント用途の検証がない**。長い trajectory・構造化出力（ツール呼び出しの JSON）・多段ツール呼び出しでの誤差の累積は、本 wiki の量子化原典 6 本のいずれも測っていない。[[concepts/model-quantization]] の設計論点に未解決として据え置いた。
- データギャップ: 今回の 3 本で **GPTQ・AWQ** が解消。残るのは **Self-Refine（arXiv:2303.17651）**・**Tree of Thoughts（arXiv:2305.10601）**・**context rot の研究（Chroma）**・**Anthropic HH（arXiv:2204.05862）**・**DeepSeek-LLM（2401.02954）/ DeepSeek-V2（2405.04434）**・**AI Scientist v1（2408.06292）**・**QuIP#**・**LoRA 本体（arXiv:2106.09685）**。新たに **OBQ（Frantar & Alistarh 2022）**・**BRECQ**・**BitNet b1.58**・**AQLM**・**PEQA / QA-LoRA** が「概念ページで言及したが一次資料が未取得」に加わった。**LoRA 本体**は [[concepts/parameter-efficient-fine-tuning]] の根拠を薄いままにしているので依然として最優先である。

## [2026-08-02] ingest | 訓練側の低精度 4 本（BFLOAT16 / FP8-LM / NVFP4 事前学習 / NVFP4 推論実測）

- 取り込み: `raw/papers/A Study of BFLOAT16 for Deep Learning Training.md`（Kalamkar ら / Intel Labs・Facebook, 2019, arXiv:1905.12322）、`raw/papers/FP8-LM_ Training FP8 Large Language Models.md`（Peng ら / Microsoft Azure・Microsoft Research, 2023, arXiv:2310.18313）、`raw/papers/Pretraining Large Language Models with NVFP4.md`（NVIDIA, 2025, arXiv:2509.25149）、`raw/articles/NVFP4_ Same Accuracy with 2.3x Higher Throughput for 4-Bit LLMs.md`（Benjamin Marie / The Kaitchup, Medium, 2025-08）の **4 本同時**。前 3 本は「訓練の精度」の系譜（BF16 → FP8 → FP4）、4 本目は同じ NVFP4 の推論側の実測。
- 作成: [[concepts/low-precision-training]]（**新規**）, [[summaries/2019-bfloat16]], [[translations/2019-bfloat16]], [[summaries/2023-fp8-lm]], [[translations/2023-fp8-lm]], [[summaries/2025-nvfp4-pretraining]], [[translations/2025-nvfp4-pretraining]], [[summaries/2025-nvfp4-inference]], [[translations/2025-nvfp4-inference]], `raw/assets/2019-bfloat16/`（7 枚）, `raw/assets/2023-fp8-lm/`（13 枚）, `raw/assets/2025-nvfp4-pretraining/`（15 枚）, `raw/assets/2025-nvfp4-inference/`（2 枚）
- 更新: [[concepts/model-quantization]], [[concepts/llm-inference-optimization]], [[concepts/transformer-architecture]], [[summaries/2024-deepseek-v3]], [[summaries/2024-flashattention-3]], [[summaries/2026-gemma-4]], [[summaries/2023-awq]], [[overview]], [[index]]
- ユーザー決定: (1) **`low-precision-training` を新設**（`model-quantization` 内での拡張や `llm-inference-optimization` への寄せでなく）、(2) **4 本とも全訳**（付録含め圧縮なし）。
- **新概念ページの冒頭に「低精度訓練と QAT は別のもの」という区別を立てた。** これが本 ingest で最も持ち帰る価値のある整理である——**低精度訓練は訓練コストを下げるもので成果物は高精度モデルでよく、QAT は低ビットで配れるモデルを作るもの**であり、両者は直交する。FP8-LM が FP8 で訓練した GPT-175B は FP8 のモデルではない。逆に [[summaries/2024-efficientqat]] は訓練自体が低精度である必要はないが 2 ビットで動くモデルを作る。混同すると論文の主張を取り違えるので、ページの最初の節に置いた。
- **BFLOAT16 論文は、wiki が二次資料の権威で書いていた主張の原典だった。** [[concepts/model-quantization]] の「BF16 は範囲に振った形式」は [[summaries/2025-llm-quantization-explained]]（査読なしの Medium 記事）に依拠していたが、その出所が本論文である。該当箇所に原典への引用ブロックを追加した。あわせて **BF16 が常に優れているわけではない**ことも明記——[[summaries/2023-fp8-lm]] は master weight のように「狭い範囲で細かい更新」を扱う場面で BF16 が FP16＋テンソルスケーリングに負けることを実測しており、**形式の優劣は用途で反転する**。
- **同じ直交回転が 3 か所に独立に現れていることを確認した。** [[summaries/2024-flashattention-3]] の incoherent processing（attention の FP8 化）、[[concepts/model-quantization]] の外れ値対処 4 系統のうち「分布そのものをならす」、そして [[summaries/2025-nvfp4-pretraining]] の Random Hadamard 変換（FP4 訓練）。**推論カーネル・推論量子化・訓練という異なる層で同じ道具が使われている**ので、概念ページに明示的に書いた。ただし NVFP4 側は適用先を厳しく選んでおり（Wgrad の入力のみ。Fprop / Dgrad へ適用するとむしろ劣化）、「万能の道具」ではない点も併記した。
- **「どこを低精度にしないか」が 6 年 3 世代で変わっていない**ことを表にした。BFLOAT16（バイアス項・重み更新のマスターコピー）、FP8-LM（master weight・二次モーメント・attention 内部）、DeepSeek-V3（埋め込み・出力ヘッド・MoE ゲーティング・正規化・attention）、NVFP4（上記＋最後の 8 ブロックと最初の 2 ブロックの線形層・オプティマイザ状態）。**共通するのは埋め込み・出力ヘッド・正規化・attention 内部・オプティマイザ状態**である。あわせて「**FP4 で訓練した**」の実際の適用範囲が**線形層の 84%** であることも明記した。
- **否定的結果を独立した節にまとめた。** 活性の勾配のブロック単位量子化で 16B が約 300B トークンで発散（DeepSeek-V3）、master weight を FP8 にすると劣化・二次モーメントも FP8 にすると発散（FP8-LM）、全線形層を FP4 にすると発散・確率的丸めを順伝播に適用すると発散・Hadamard 変換を Fprop/Dgrad に適用すると劣化（NVFP4）。低精度訓練は「うまくいった構成」より「壊れる構成」の情報のほうが実務で効く。
- **記事と論文の緊張関係を要約に書いた。** NVIDIA は NVFP4 を FP8 とだけ比較しているが、Marie が AWQ・AutoRound・bitsandbytes と並べると**精度では AWQ/AutoRound がわずかに上**（5901/5900 対 5858）、**サイズは NVFP4 が約 7GB 大きい**（グループサイズ 16 対 128 でスケールが 8 倍要る）。**勝ち筋は速度だけ**（INT4 比 2.35 倍）で、しかも**活性も量子化しないとその速度は出ない**（NVFP4A16 は 774 tok/s で INT4 の 723 とほぼ同じ）。これは [[concepts/model-quantization]] に前回立てた「重みのみか、重みと活性か」の軸の、**同一形式・同一ハードウェアでの実例**になったので、両ページから相互参照した。なお訓練論文は「本報告は実行時の効率ではなくアルゴリズムに関わる」として**速度を測っていない**ので、2 本は互いの空白を埋める関係にある。ただし訓練と推論では量子化の適用範囲が違う（訓練は線形層の 84% ＋ 勾配、推論は重みと活性）ため、**数字を直接つなげてはいけない**旨を要約に明記した。
- クリップ不良: **4 本で健全度が大きく分かれ、FP8-LM は本 wiki で扱った中で最悪だった。**
  - **FP8-LM**: **データ表 3 件が丸ごと欠落**（**Table 3** の SFT 比較と **Table 4** の RLHF 比較はキャプションごと消滅、**Table 6** の精度分離はキャプションだけ残って数値表が消え、**その位置に Figure 8 の画像（`x7.png`）が入り込んでいた**）、**図キャプション 4 件欠落**（Figure 4・7・8・9。Figure 4 は 3 パネル中 (a) のみ、Figure 7 も 3 パネル中 (a) のみ）、**画像 4 枚欠落**（`GPT-13b`・`GPT-175b`・`x5`・`x6`）、**脚注 8 件欠落**。すべて ar5iv から復元した。
  - **BFLOAT16**: **画像 7 枚中 3 枚欠落**（Figure 2・3・4 はいずれも 2 パネル図で**どれも (a) しか残っていない**）、**本キャプション 3 件が消滅**して `Refer to caption` だけが残存、**脚注 3 件欠落**。すべて復元した。
  - **NVFP4 事前学習**: 最も健全（画像 15/14、Figure 13/14、Table 5/5）。**Figure 6 の 2 枚目（`x7`）と図全体のキャプション**、脚注 1 件のみ復元。
  - **NVFP4 記事（Medium）**: 健全。画像 3 枚のうち**装飾写真 1 枚（Unsplash のヘッダー）を除外**し、著者作成のチャート 2 枚を保存（CDN URL から `format:webp/` を外して素の PNG を取得）。**別記事への誘導カードも除外**（skill 既定）。2 枚のチャートは**数値を読み取って `<figcaption>` へ転記**した（画像のままでは検索も引用もできないため）。
- **原典（ar5iv）側の変換不良を、クリップ不良と区別して記録した。** BFLOAT16 の Table 1 の指数表記が `3.401038`・`1.1710-38` のように **`×10^` の区切りを失った状態で ar5iv 自体に格納されている**（`alttext` を直接確認して確定）。数字列は完全に残っており、いずれも IEEE-754 の標準的な定数と一致するので桁を復元して訳出し、その旨を訳注に明記した。**「クリップが壊した」のか「原典がそうなっている」のかを区別する**作法の適用例である。
- 批判的視点として要約に明記した点: **BFLOAT16** — ほぼすべてエミュレーション（4.5 節の AVX512BF16 実訓練を除く）／**Transformer が対象外**で、LLM で BF16 が標準になった根拠は本論文にない（後年の Bloom / Gopher の実地経験による）／Intel Labs 主導で AVX512BF16 の正当化を兼ねる利害／**仮数 7 ビットの代償を定量していない**。**FP8-LM** — **175B は 40B トークンしか訓練していない**（著者が炭素排出とコストを理由に明記）、7B/13B も 100B トークンで現代の基準より 2 桁短い／理論値の 2 倍に対し実測は +75%／**MFU は BF16 より低い**（39.0% → 34.2%）——スループットは上がるが計算資源の利用効率は落ちている／attention 自体は FP8 化していない／多くの数値が単一実行で分散の報告がない／Azure の論文で比較対象の TE は NVIDIA 製という利害。**NVFP4 事前学習** — **速度を測っていない**（論文が明記。Blackwell の 2〜3 倍はハードウェアの仕様であってこの実行の実測ではない）／**まだ 16% が高精度**で attention・埋め込み・オプティマイザは最初から対象外／FP8 ベースラインとの比較が 1 本ずつで**乱数シードの分散が不明**（62.58 対 62.62 を「ほぼ一致」と読むのは解釈）／**コードタスクだけ 2.5〜3.2 ポイント劣化**して「評価のノイズかもしれない」で済ませている／**各技法の相対的な恩恵は追加の順序に依存する**と論文自身が認めており「どれが一番効くか」は答えられない設計／**アブレーションの多くが 1.2B で行われている**のに論文自身が「小規模の結論は大規模に外挿できない」と繰り返す／NVIDIA 独自形式で、MXFP4 が OCP のオープン標準である点は書かれていない。**NVFP4 記事** — 査読なしの個人ブログ、1 モデル・1 GPU・1 ベンチマーク群、分散の報告なし／著者自身が「70B では 4 ビットが既に完全精度に近いので差が出にくい」と限界を認める／**MXFP4 と比較できていない**（LLM Compressor 未対応）／「The Kaitchup Index」の中身が定義されておらず絶対値が解釈できない／Blackwell 世代限定／見出しの「2.3x」の分母は INT4 であって FP16 ではない。
- 4 本に共通する空白として記録: **エージェント用途の検証がない**。低精度訓練されたモデルが長い trajectory・多段のツール呼び出し・構造化出力でどう振る舞うかは測られていない。[[concepts/model-quantization]] に置いた同じ但し書きを [[concepts/low-precision-training]] にも据えた。
- データギャップ: 残るのは **Self-Refine（arXiv:2303.17651）**・**Tree of Thoughts（arXiv:2305.10601）**・**context rot の研究（Chroma）**・**Anthropic HH（arXiv:2204.05862）**・**DeepSeek-LLM（2401.02954）/ DeepSeek-V2（2405.04434）**・**AI Scientist v1（2408.06292）**・**QuIP#**・**LoRA 本体（arXiv:2106.09685）**・**OBQ**・**BRECQ**・**BitNet b1.58**・**AQLM**・**PEQA / QA-LoRA**。今回新たに **Micikevicius ら「Mixed Precision Training」（arXiv:1710.03740）**（FP16＋loss scaling の原典。BFLOAT16 論文が比較対象にしている）、**FP8 の標準仕様（Micikevicius ら 2022, arXiv:2209.05433）**、**MX 形式の仕様（OCP / arXiv:2310.10537）**、**Nemotron-H（arXiv:2504.03624）** が「概念ページで言及したが一次資料が未取得」に加わった。**LoRA 本体**は [[concepts/parameter-efficient-fine-tuning]] の根拠を薄いままにしているので依然として最優先である。

## [2026-08-02] ingest | LLM serving の 2 本（Zenn 技術記事 / Medium ライフサイクル記事）

- 取り込み: `raw/articles/LLM Servingを支える技術.md`（釜堀 / ワシントン大学・Kotoba Technologies, Zenn, 2025-07-21）、`raw/articles/Understanding LLM Serving_ How to Run Language Models Fast, Cheap, and Effectively.md`（Thanh Tung Vu, Medium, 2025-04-21）の **2 本同時**。同じ主題の両極——前者は機械学習システム研究者による体系的なサーベイ、後者はビジネス寄りのライフサイクル概観。
- 作成: [[concepts/llm-serving-systems]]（**新規**）, [[summaries/2025-llm-serving-techniques]], [[summaries/2025-understanding-llm-serving]], [[translations/2025-understanding-llm-serving]], `raw/assets/2025-llm-serving-techniques/`（16 枚）
- 更新: [[concepts/llm-inference-optimization]], [[concepts/model-quantization]], [[concepts/context-engineering]], [[concepts/tool-use-and-function-calling]], [[concepts/agent-observability]], [[concepts/reinforcement-learning-from-human-feedback]], [[concepts/mixture-of-experts]], [[summaries/2026-llm-optimization-guide]], [[summaries/2022-flashattention]], [[summaries/2022-gptq]], [[summaries/2024-deepseek-v3]], [[overview]], [[index]]
- ユーザー決定: (1) **`llm-serving-systems` を新設**（`llm-inference-optimization` の拡張でなく）、(2) Zenn 記事の図 27 枚のうち**論旨を担う図だけ選ぶ**、(3) **日本語記事は翻訳ファイルを作成しない**。
- **翻訳ファイルは英語記事のみ作成した。** 日本語記事（Zenn）はユーザーの指示により翻訳を作らず、要約ページが唯一の置き場になる。そのため要約ページは通常より厚く書き、図もそちらへ配置した。
- **小さな慣例を 1 つ導入した（本 wiki 初の日本語原典）。** 翻訳を持たない要約ページの frontmatter では `translation: null` を**明示する**ことにした。フィールドごと省くと「書き忘れ」と区別がつかないが、`null` なら「翻訳が存在しないことを確認済み」と読める。CLAUDE.md §2 はこのケースを定めていないので、日本語原典が増えるようならスキーマへ明文化することをユーザーに提案したい（本 ingest では既定の変更まではしていない）。
- **新概念ページの立て方**: 冒頭に **[[concepts/llm-inference-optimization]] との違いを表で置いた**——あちらは「1 回の前向き計算をどう速くするか」（カーネル・アーキテクチャ・精度、HPC/コンパイラに近い）、こちらは「**1 台の GPU で多数のクライアントをどう多重化するか**」（資源の多重化・スケジューリング・断片化・SLO、**OS/分散システムに近い**）。**FlashAttention を 3 倍速くしても、リクエストが 1 件しか来ていなければ GPU は 99% 遊んでいる**——この一文を分割の根拠として書いた。model-quantization と low-precision-training を切り出したのと同じ判断である。
- **本 wiki のサービング側の記述が、初めて一次の設計論理を持った。** これまで [[concepts/llm-inference-optimization]] の「### バッチングと KV cache 管理」は**箇条書き 6 行**で、しかも根拠が [[summaries/2026-llm-optimization-guide]] 経由の Introl 分析の孫引きだった。「continuous batching でトークン単価 −85%」という数字はあっても、**なぜバッチングがそこまで効くのか**が入っていなかった。Zenn 記事の Roofline による導出——decode の線形層は演算密度が $n/(1+n)\approx 1$ で、**1 個データを読むごとに 1 回しか計算していない**ので、**リクエストを 1 件から 2 件に増やすのは時間的にタダ**——を入れたことで、以降のほぼすべての技法（量子化がどこで効くか、GQA の有無で動作領域が切り替わること、投機的デコーディングがスループットを上げないこと）が同じ枠で読めるようになった。
- **量子化の Roofline 図が、前回 ingest で立てた軸の視覚的根拠になった。** [[concepts/model-quantization]] に 2026-08-02 の前の ingest で `### 重みのみか、重みと活性か` を新設したが、Zenn 記事が引く Atom 論文の図が**まさにその区別を Roofline 上で示している**——weight-activation は低ビット計算で**達成可能な FLOPS の上限そのものを上げる**ので compute-bound でも効き、weight-only は**演算密度を上げるだけ**なので memory-bound でしか効かない。「ローカル LLM なら weight-only、サービングなら weight-activation」という線引きが図から直接読める。該当節に引用ブロックを追加して相互参照した。
- **エージェント設計への含意を 2 つ引き出した。** (1) **prefix キャッシュはエージェントで最も効く**——多ターンの trajectory は定義上「共通 prefix ＋ 少しずつ伸びる末尾」であり、prefix キャッシュが最もよく効く形をしている。逆に**プロンプトの可変部分（タイムスタンプ・ランダム ID・シャッフルしたツール一覧）を先頭に置くとキャッシュが毎回無効になる**ので、「安定した接頭辞、変動する末尾」がコスト設計の規律になる（→ [[concepts/context-engineering]]）。(2) **構造的デコーディングは正しさの機能であると同時に高速化でもある**——制約で次のトークンが一意に確定する箇所では LLM を呼ぶ必要がなく、**ツール呼び出しの JSON はまさにその形**である（→ [[concepts/tool-use-and-function-calling]]）。いずれも原記事には「エージェント」という語は出てこないが、記事の論理から素直に出る含意なので概念ページと要約の両方に書いた。
- **serving system が RL 訓練の内部で使われている**という接続も記録した。RL では LLM に複数の文章を生成させて報酬を与えるので、**訓練システムであっても推論性能が要る**。OpenRLHF・verl・SkyRL はいずれも内部で vLLM を使う（→ [[concepts/reinforcement-learning-from-human-feedback]]）。「推論のインフラ」と「訓練のインフラ」が分離できなくなっている。
- **二次資料の誤りを、前回 ingest した一次資料で正せた。** 英語記事は「**GPTQ (Gradient Post-Training Quantization)**」としているが、原典（[[summaries/2022-gptq]]、前 ingest で取り込み）の**脚注 1 が「OPT モデルファミリの名と PTQ の合成」と明記している**。「Gradient」は入っておらず、手法自体も勾配でなく**ヘッセ行列の二次情報**を使う。要約の限界節に書き、[[summaries/2022-gptq]] からも逆参照した。同記事の「early exit」の説明も、**層単位の early exit（ネットワークの途中で抜ける）と生成の打ち切りを区別していない**点を指摘した。
- クリップ不良: **2 本とも健全**（本文・見出しに欠落なし）。
- **画像の扱いで 2 つの判断をした。**
  - **Zenn 記事は 27 枚のうち 16 枚を選抜した。** すべて他の論文・ブログからの引用図（出典リンク付き）で、記事の論旨を担うもの——Roofline・静的バッチング／continuous batching の対・PagedAttention・RadixAttention・NanoFlow・CUDA Graph・非同期スケジューリング・chunked prefill・P/D 分離・**量子化の Roofline**・投機的デコーディング・構造的デコーディング・DeepSeek の 3 枚——を残した。落としたのは、**既に wiki 内に同等の図がある**もの（FlashAttention・MQA/GQA/MLA・SWA）、**本文で足りる**もの（StreamingLLM・Wanda・FlashInfer の CSR レイアウト・Punica のカーネル図・vLLM V1 アーキテクチャ・large-scale EP・prefix 共有の説明図 2 枚）。
  - **Medium 記事は 5 枚すべて除外した。** 1 枚は記事タイトルを載せた装飾カバー、残り 4 枚は「Which strategy should be used?」のように**本文の箇条書きをアイコン付きで復唱するインフォグラフィック**である。ダイアグラム・フロー図・プロット・スクリーンショットのいずれでもなく本文を超える情報を持たないため、skill ケース C の除外規則（判断に迷う装飾画像は基本除外）に従った。**除外の判断は Read で中身を確認したうえで行った**（ファイル名や `0*`/`1*` の接頭辞だけで判断していない）。
- 批判的視点として要約に明記した点: **Zenn 記事** — 査読なしの企業ブログだが**記述のほぼすべてが一次文献へのリンクを伴い、出典の追跡可能性は本 wiki の他の二次資料より格段に高い**／**数値がほとんどない**（技術の地図であってベンチマークではない。定量値は [[summaries/2026-llm-optimization-guide]] で補う）／KV cache の見積もりが **GQA を持たない Llama 2 7B** で行われている（著者も脚注で断っている）／**トレードオフが「速くなる」側に寄っており**、各技法の精度への影響がほとんど扱われない（StreamingLLM の根拠も著者自身が「ヒューリスティクス」と書いている）／2025 年 7 月時点で**この層は半年で変わる**／**エージェント固有の観点はない**。**Medium 記事** — 技術的な誤り 1 件（GPTQ の名称）と曖昧な説明 1 件（early exit）／**数値が一切ない**／査読なしで出典がほとんどなく「OpenAI・Anthropic・Cohere がハイブリッドな手法を採る」にも根拠がない／**serving system の内部に踏み込まない**（continuous batching も PagedAttention もツールの機能として名前が出るだけで、なぜ効くのかは説明されない）／**カスケードの実務的な難所を扱わない**（「信頼度の閾値」をどう決めるか、**LLM の自己申告の信頼度が当てにならない**という既知の問題に触れていない）。
- データギャップ: 残るのは **Self-Refine（arXiv:2303.17651）**・**Tree of Thoughts（arXiv:2305.10601）**・**context rot の研究（Chroma）**・**Anthropic HH（arXiv:2204.05862）**・**DeepSeek-LLM（2401.02954）/ DeepSeek-V2（2405.04434）**・**AI Scientist v1（2408.06292）**・**QuIP#**・**LoRA 本体（arXiv:2106.09685）**・**OBQ**・**BRECQ**・**BitNet b1.58**・**AQLM**・**PEQA / QA-LoRA**・**Mixed Precision Training（1710.03740）**・**FP8 仕様（2209.05433）**・**MX 仕様（2310.10537）**・**Nemotron-H（2504.03624）**。今回新たに **vLLM / PagedAttention 論文（arXiv:2309.06180）**・**SGLang / RadixAttention 論文（arXiv:2312.07104）**・**Orca / continuous batching（OSDI'22）**・**Chunked Prefill（arXiv:2308.16369）**・**P/D 分離（arXiv:2401.09670）**・**NanoFlow（arXiv:2408.12757）**・**StreamingLLM（arXiv:2309.17453）**・**Atom（arXiv:2310.19102）**・**投機的デコーディング原典（arXiv:2211.17192）** が「概念ページで代表手法として記述したが一次資料が未取得」に加わった。**とくに PagedAttention と RadixAttention は [[concepts/llm-serving-systems]] の 2 本柱なので優先度が高い。** **LoRA 本体**は依然として [[concepts/parameter-efficient-fine-tuning]] の根拠を薄いままにしている。

## [2026-08-03] ingest | サービングの原典 2 本（PagedAttention / SGLang）

- 取り込み: `raw/papers/Efficient Memory Management for Large Language Model Serving with PagedAttention.md`（Kwon ら / UC Berkeley・Stanford・UCSD, SOSP 2023, arXiv:2309.06180）、`raw/papers/Efficiently Programming Large Language Models using SGLang.md`（Zheng ら / Stanford・UC Berkeley, 2023, arXiv:2312.07104）の **2 本同時**。**前日のログで「最優先」と書いたデータギャップをそのまま解消**した。
- 作成: [[concepts/llm-programming-systems]]（**新規**）, [[summaries/2023-pagedattention]], [[translations/2023-pagedattention]], [[summaries/2023-sglang]], [[translations/2023-sglang]], `raw/assets/2023-pagedattention/`（26 枚）, `raw/assets/2023-sglang/`（12 枚）
- 更新: [[concepts/llm-serving-systems]]（**大幅に書き直し**）, [[concepts/agent-frameworks]], [[concepts/llm-agents]], [[concepts/reasoning-and-planning]], [[concepts/tool-use-and-function-calling]], [[summaries/2022-react]], [[summaries/2025-llm-serving-techniques]], [[summaries/2025-understanding-llm-serving]], [[overview]], [[index]]
- ユーザー決定: (1) SGLang の言語処理系の半分は **`llm-programming-systems` を新設**して置く（`agent-frameworks` へ入れるのでも serving 内に留めるのでもなく）、(2) **原典版の図を保存し、概念ページはそちらへ差し替える**（Zenn 版の図は当該記事の要約ページに残す）。
- **前日に二次資料から作った [[concepts/llm-serving-systems]] が、一次の根拠を得た。** PagedAttention と RadixAttention の節を原典の数字で書き直し、**図も Zenn 経由の再ホスト版から原典版（`2023-pagedattention/x6.png`・`2023-sglang/x6.png`）へ差し替えた**。Zenn 記事の要約ページ側は再ホスト版のまま残してある（そちらは「記事が引用した図」として正しい）。
- **昨日の記述に訂正を入れた。** wiki は「PagedAttention → prefix キャッシュ」と滑らかに繋いでいたが、**vLLM 論文 §4.4 の共有 prefix は「提供者が事前定義した prefix のために物理ブロックを予約しておく」手動の仕組み**である。しかも SGLang 論文が「**vLLM の論文はこの機能を論じているが、信頼できる高性能なカーネルがないため公開されたコードは対応していない**」と明言している。**自動の prefix 再利用は SGLang の貢献**であり、概念ページに引用ブロックで訂正を置き、[[summaries/2023-pagedattention]] の限界節と [[summaries/2025-llm-serving-techniques]] の関連ページからも指摘した。**二次資料が機構を正しく紹介していても、系譜の繋ぎ方は原典に当たらないと確定できない**という例である。
- **vLLM の見どころは手法より診断である。** KV キャッシュの無駄を**予約（将来のトークンぶん。最終的には使うがその間ほかが使えない）・内部断片化（最大長への過剰供給。リクエストが終わって初めて分かる）・外部断片化（アロケータ由来。生成に決して使われない）**の 3 つに分解し、**既存システムが実際に使えているのは 20.4〜38.2% にすぎない**ことを測ってから手法を出す。この順序を概念ページと要約の双方で保った。
- **最も持ち帰る価値があるのは正直なアブレーションだった。** **PagedAttention の attention カーネル単体は FasterTransformer より 20〜26% 遅い**（block table へのアクセス・追加の分岐・可変系列長の処理という間接参照の税）。**それでも端から端までは 2〜4 倍速い**——より多くのリクエストが載るからである。前日 Zenn 記事から入れた Roofline の枠組み（**カーネルを速くするのでなく memory-bound の領域でバッチサイズを上げる**）が、一次資料でそのまま裏づけられた形になったので、概念ページに引用ブロックで明示した。
- **OS の比喩の「捨てどころ」を書いた。** vLLM は比喩を最後まで使い切る（ブロック=ページ、トークン=バイト、リクエスト=プロセス、block table=ページテーブル、参照カウント＋copy-on-write=fork、共有 prefix=共有ライブラリ、CPU メモリ=スワップ空間）一方で、**§8 でどこが OS と違うかを明示している**——**all-or-nothing の退避**（系列のブロックはまとめてアクセスされる）と、**再計算による復旧**（生成済みトークンを元のプロンプトへ連結すれば 1 回の prefill で作り直せる。**OS では不可能**）。さらに**適用すべきでない領域**（DNN 訓練は形状が静的、計算律速なサービングでは間接参照でむしろ悪化）まで書いている。**比喩を借りたらどこで捨てるかも決める**、という一般則として要約の示唆に置いた。
- **エージェントの主張に一次の数字が付いた。** 前日 Zenn 記事から「多ターンの trajectory は『共通 prefix ＋ 伸びる末尾』なので prefix キャッシュが最も効く」と書いたが、SGLang 論文が **ReAct エージェント（HotpotQA）で vLLM 比スループット 5.6 倍・レイテンシ 13%** を報告し、理由を「**エージェントが思考・行動・観測を続く LLM 呼び出しのプロンプトへ追記していく過程**」と明示している。対照的に **1 シミュレーション 1 呼び出しの generative agents では 30% に留まる**ので、**「エージェントだから速くなる」のではなく「状態を追記する形をしているから速くなる」**と書き分けた。[[summaries/2022-react]] からも逆参照した。
- **新概念ページ [[concepts/llm-programming-systems]]**: SGLang 論文の**半分は言語処理系**であり、RadixAttention だけを取ると論文の半分を落とすことになる。ページの骨格は論文自身が §2.4 で描いた層（**最上層 LangChain > 中間層 DSPy > SGLang > 最下層 LMQL/Guidance**）に置き、プリミティブ設計（5 つだけにして制御フローは宿主言語に任せる）、**プロンプトを非同期ストリームとして扱うインタプリタ**（「CUDA のカーネルを CUDA のストリームへ非同期に起動するのに似ている」）、計算グラフと IR、コンパイラ最適化、構造化出力が**正しさと速度の両方**であること、を並べた。**ページ末尾に「本 wiki における根拠の薄さ」節を置いた**——一次資料は SGLang 1 本だけで、LMQL・Guidance・DSPy の記述はすべて SGLang 論文の記述に依拠しており、**SGLang は自らを両者の改良として位置づけているので利害関係がある**旨を明記した。
- **GPT-4 にコンパイラ最適化をさせる**という着想を独立の項目にした。IR ノードを並べ替えて定数 prefix を長くする古典的なコード移動だが、**「自然言語の指示が含まれるため伝統的なプログラム解析では達成できない」**から GPT-4 に頼む。15 テンプレート中 12 件成功、共有可能 prefix が平均 +60 トークン。失敗の型も具体的（**積極的すぎて意味論を変える場合でも定数を前へ持ってくる**）。著者自身が「元の計算を厳密には保存しない積極的な最適化」と分類し留保しているので、**本番で自動適用するには早い**と要約に書いた。これは SGLang 固有でなく**プロンプトを含むコードベース全般**に当てはまる観察なので、概念ページの設計論点にも置いた。
- クリップ不良: **2 本で性格が違った。**
  - **vLLM**: **本文は完全**（長い段落もすべて末尾まで残っており、箇条書きに見えた箇所も原ページと照合して太字リード段落だと確認した）。失われたのは図とキャプションだけで、**画像 26 枚中 7 枚**（`x13`・`x15`・`x17`・`x19`・`x21`・`x25`・`x27`）と、**Figure 11・13・15・18・19 の本キャプション 5 件**、脚注 1 件。**Figure 11・12・13・14・15・18・19 はいずれも 2 パネル構成で、そのすべてが (a) だけを残して (b) を失う**——BFLOAT16・LLM.int8()・FP8-LM と同じ型である。
  - **SGLang**: **画像 12 枚すべてと Table 1・Algorithm 1 が揃っていたので当初「健全」と判断したが、誤りだった。** 本文が「These include:」で終わったまま次の段落へ飛んでおり、**§2.3 の箇条書き 5 項目（Caching・Batching・Sharing・Parallelism・Compilation）が丸ごと欠落**していた。これは論文が最適化の機会を整理した中核の一覧で、新概念ページの骨格にもなる部分である。原ページから復元した。
- **自己修正（点検手順の改善）**: SGLang を最初「完全に健全」とユーザーへ報告したが、**画像とキャプションの枚数照合だけでは本文の欠落を検出できない**。その後、**「`:` で終わる行の直後が箇条書きでない箇所」を両論文で機械的に洗い出す**点検を追加して §2.3 の欠落を見つけた。vLLM 側でも同じ検査が 8 件ヒットしたが、原ページと照合して**すべて数式または太字リード段落の偽陽性**と確定した。**図表の枚数照合に加えて、リード文と本体の対応も見る**——これを今後の既定の点検に加える。
- 批判的視点として要約に明記した点: **vLLM** — **共有 prefix は自動でなく手動**で、公開コードでは動かなかった／**Orca を自前で再実装**しており比較相手の性能が著者の実装に依存する（3 段階のベースライン Oracle/Pow2/Max を置いたのは緩和策だが Oracle は「実現不可能」と自認）／**チャットボットの実験で会話ラウンド間の KV キャッシュを保持していない**——つまり**エージェントの多ターンで最も効くはずの再利用を意図的に使っていない**（SGLang が示したのはここに 5 倍以上の余地があるということ）／**反復単位スケジューリング自体は Orca の貢献**で vLLM はそれを前提にメモリ側を解いている（**continuous batching を vLLM の発明と読まない**）／著者自身が §8 で適用範囲の限定を述べている／評価は OPT と LLaMA・A100 のみで **GQA を持つモデルが対象外**。**SGLang** — **コンパイラがデータ依存の制御フローに対応しない**（エージェントの制御フローは定義上そうなので、**冒頭で掲げた用途にコンパイラが使えない**という緊張がある。エージェントの成果はインタプリタ側のもの）／**文法制約付きデコーディング未実装**（[[concepts/llm-serving-systems]] が構造的デコーディングの根拠に引く compressed-FSM は**本論文より後**）／GPT-4 の最適化は 15 中 12 件で失敗は意味論を壊す／**ベースラインの状態に依存**（Guidance と LMQL は**バッチサイズ 1 しか支えない**、vLLM v0.2.2 は prefix 共有を持たない時点のもの）／長文書の実験で**計算を変える最適化**（parallel-context window）を使っている／ほとんどの実験が 7B・A10G 単体／**arXiv v1 で後に改題されている**。
- データギャップ: 今回の 2 本で **PagedAttention（2309.06180）と RadixAttention / SGLang（2312.07104）** が解消。残るのは **Self-Refine（2303.17651）**・**Tree of Thoughts（2305.10601）**・**context rot（Chroma）**・**Anthropic HH（2204.05862）**・**DeepSeek-LLM（2401.02954）/ DeepSeek-V2（2405.04434）**・**AI Scientist v1（2408.06292）**・**QuIP#**・**LoRA 本体（2106.09685）**・**OBQ**・**BRECQ**・**BitNet b1.58**・**AQLM**・**PEQA / QA-LoRA**・**Mixed Precision Training（1710.03740）**・**FP8 仕様（2209.05433）**・**MX 仕様（2310.10537）**・**Nemotron-H（2504.03624）**・**Orca / continuous batching（OSDI'22）**・**Chunked Prefill（2308.16369）**・**P/D 分離（2401.09670）**・**NanoFlow（2408.12757）**・**StreamingLLM（2309.17453）**・**Atom（2310.19102）**・**投機的デコーディング原典（2211.17192）**。新たに **LMQL（2212.06094）**・**Guidance**・**DSPy（2310.03714）** が「[[concepts/llm-programming-systems]] で言及したが一次資料が未取得」に加わった。**同ページの根拠は SGLang 1 本のみで、しかも比較記述が SGLang 側の自己申告**なので優先度が高い。**Orca** も、vLLM が「相補的」と位置づける相手として重要度が上がった。**LoRA 本体**は依然として [[concepts/parameter-efficient-fine-tuning]] の根拠を薄いままにしている。

## [2026-08-03] ingest | RoFormer: Enhanced Transformer with Rotary Position Embedding

- 取り込み: `raw/papers/RoFormer_ Enhanced Transformer with Rotary Position Embedding.md`（ケース A: ar5iv クリップ。arXiv:2104.09864 → Neurocomputing 568, 2024。Su, Lu, Pan, Murtadha, Wen, Liu / Zhuiyi Technology）
- 作成: [[summaries/2021-roformer]], [[translations/2021-roformer]], [[concepts/positional-encoding]]
- 更新: [[concepts/transformer-architecture]], [[concepts/llm-inference-optimization]], [[concepts/context-engineering]], [[summaries/2023-lost-in-the-middle]], [[overview]], [[index]]
- 画像: `raw/assets/2021-roformer/` に 4 枚（`x1`〜`x4`）
- 対話での決定: (1) 位置符号化は **`positional-encoding` を新設**（`transformer-architecture` に節を足すのでなく独立ページ）。理由は RoPE が本 wiki で**最も多く参照されながら一度も説明されていない技法**であり（8 ページ以上で言及）、しかも**派生形がすでに全部 wiki にある**（p-RoPE / 部分 RoPE / MLA の decoupled RoPE key / YaRN / 縮約 RoPE / 対照としての ALiBi）ため。(2) 要約では §3.4.1 の導出（約 210 行・式 30 本）を**論理の骨格として追う**（式変形の逐一展開はせず、翻訳側に全訳を置く）
- クリップの状態: **軽微**。本文・図表キャプション・数式に欠落なし。**画像 4 枚中 1 枚（`x4`）が欠落**——Figure 3 は 2 パネル図（左: BERT/RoFormer、右: PerFormer ± RoPE）で、既知の「多パネル図の 2 枚目が落ちる」型。**脚注 2 件の本文も欠落**（マーカーのみ残存）。いずれも原ページから復元
- 訂正の記録: 読解中に「本文 4 か所で文が途中で切れている」と一度判断したが、**それは私自身の表示側の切り詰め（`l[:900]`）であってクリップは無傷**だった。原ページ照合で確認して撤回した。長行を扱うときは切り詰め幅を疑うこと
- 翻訳の書式: ar5iv は整列数式を `\displaystyle` 付きの独立ブロックに分解して出力するため、**原典どおりの `aligned` 環境に戻して**掲載した（式の内容は不変）
- 原典側の表記の揺れ（クリップ不良ではない。訳注に記録）: $\theta_i$ の定義が式 (15) では $10000^{-2(i-1)/d}$、§3.3・§3.4.3 では $10000^{-2i/d}$／$g$ の第 3 引数が式 (11) では $m-n$、式 (21) では $n-m$／本文は事前学習「100k ステップ」だが図3 左の横軸は 250K まで伸びている／式 (16) の $\boldsymbol{x}^\intercal$ と式 (32) の $\boldsymbol{W}_q\boldsymbol{x}_n$ は添字の誤記
- メモ: **論文の実証は弱いのに手法は標準になった**という珍しい型の原典。GLUE は 6 タスク中 **3 勝 3 敗**（負け幅の方が大きい: MNLI −4.4/−3.6・SST-2 −2.8・QNLI −2.5 に対し勝ちは QQP を除けば最大 +1.2）なのに、本文は「3 つで有意に上回り、改善は相当なものである」としか書かず**負けに一切触れない**。唯一大きな勝ちの QQP +15.2 は **BERT 側の 71.2 が異常に低い**（通常 88〜91）ので単独で疑わしい。翻訳は +0.2 BLEU（単一シード・信頼区間なし）。さらに、現在 RoPE と最も結びつけられる**外挿（訓練長を超える系列）を一度も検証していない**——中国語実験の 1024 は 1536 まで含めて事前学習したモデルの評価である。著者自身が §4.5.5 で「なぜ速く収束するのか」「なぜ長文で強いのか」の**どちらも説明できない**と明記している
- メモ: 概念ページには **`## 本 wiki における根拠の所在`** 節を置き、一次資料あり／二次資料のみ（系譜 (b) の Shaw・Transformer-XL・T5・DeBERTa はすべて RoFormer の関連研究節経由＝**RoPE を提案する側による整理**）／原典なし（**YaRN・Position Interpolation・ALiBi・NoPE**）を区別した。`llm-programming-systems`（SGLang）で始めた作法の 2 例目
- データギャップ（次の取り込み候補）: **YaRN 原典（Peng ら 2023）**と **ALiBi 原典（Press ら 2021）**。長コンテキスト化は現在の主戦場なのに、本 wiki では「他の原典が使っている手法」としてしか登場していない。既存のギャップ（DeepSeek-LLM 2401.02954 / DeepSeek-V2 2405.04434 / Anthropic HH 2204.05862 / context rot（Chroma））は変わらず（**訂正 2026-08-03 lint**: この一覧に挙げた「lost in the middle」は、**2026-08-02 に [[summaries/2023-lost-in-the-middle]] として取り込み済み**であり未取得ではなかった。繰り越し時の確認漏れ）

## [2026-08-03] ingest | LlamaFirewall: An open source guardrail system for building secure AI agents

- 取り込み: `raw/papers/LlamaFirewall_ An open source guardrail system for building secure AI agents.md`（ケース A: ar5iv クリップ。arXiv:2505.03574, 2025-04。Meta / Joshua Saxe ほか 18 名）
- 作成: [[summaries/2025-llamafirewall]], [[translations/2025-llamafirewall]]
- 更新: [[concepts/agent-safety-and-guardrails]]（層 (2) と層 (3) を大幅加筆）, [[concepts/agent-evaluation]]（AgentDojo と防御評価の規律の節を新設）, [[concepts/coding-agents]], [[concepts/tool-use-and-function-calling]], [[summaries/2025-cot-faithfulness]], [[summaries/2024-llm-security-privacy-survey]], [[overview]], [[index]]
- 画像: `raw/assets/2025-llamafirewall/` に 8 枚（`image1`〜`image4`・`image6`〜`image8`・`x1`）
- 対話での決定: (1) **CoT モニタリングの衝突を対立として書き下す**（両論文とも主張していない統合である旨を明記したうえで、`agent-safety-and-guardrails` 層 (3) に小節を立てる）。(2) 付録 **C.5 は構造と要所に圧縮**（C.6 のシステムプロンプト全文は英語原文のまま全収録）
- クリップの状態: **付録 C.3「Recommended Use Cases」が見出しごと丸ごと欠落**、**§3.1 の「Threat Scenario」小見出しと攻撃設定の一文が欠落**（インジェクション本文だけが残っていた）。いずれも原ページから復元。画像 8 枚は健在。**引用が裸の bibkey なのは ar5iv 側に文献一覧が生成されていないため**でクリップ不良ではない（`ltx_bibitem` 0 件・`ltx_cite` 64 件を確認）
- 新概念ページは作らなかった。本論文が埋めるのは既存の `agent-safety-and-guardrails` の対策 4 層のうち **(2) 入出力のガードレール**と **(3) 監視**であり、同ページの骨格（4 層）は変えずに中身を厚くするのが適切と判断した。**層 (2) はこれまで LLM セキュリティサーベイの手法カタログ（二次・汎用）と building-effective-agents の指針しか根拠がなく、実装され測られた系の一次資料はこれが最初**である
- 埋まった「約束された穴」: `agent-safety-and-guardrails` の層 (2) は「ASR だけでなく通常タスクの性能をどれだけ損なったかを併記する規律が要る（→ agent-evaluation）」と書いて `agent-evaluation` を指していたが、**`agent-evaluation` 側にその話は一切なかった**（攻撃系ベンチマークはゼロ、`AgentDojo` は wiki 全体で 0 件）。今回 `agent-evaluation` に `#### 防御を評価するときは、指標を必ず対にする` を新設して閉じた
- メモ（本論文の設計則）: **(1) 文脈を知らない安いモデルに文脈依存の判断をさせない**——PromptGuard は 1 → 2 で「広い goal hijacking 検出」を捨てて「明示的ジェイルブレイク」に**スコープを狭めた**ことが改善の中身だった（広く取ると偽陽性が爆発する）。**(2) 検査器に攻撃者由来の生テキストを渡さない**——AlignmentCheck はツール出力を直接受けず入力を PromptGuard で事前走査する。ガードレール LLM 自身が攻撃面（guardrail injection）である。**(3) カスケードは入れ子になる**——PromptGuard → AlignmentCheck と、CodeShield 内部の 60ms → 300ms が同じ形
- メモ（批判点）: **評価が防御なしで生成された静的トレースへの後付け適用**である。しかも「**攻撃が最初は成功しても後でフラグが立てばそのトレースを『防御された』に再分類する**」と明記されており、**流出後の検知を阻止に数えている**。utility の基準は 47.7%（防御なしでも半分以上失敗）で、そこからの 5.05pt 減は**相対 10.6%** ——論文の "modest" は甘い。CodeShield の**再現率 79%**（危険なコードの 5 本に 1 本は通る、CI はおよそ 0.61〜0.97）。そして**論文自身の図5 が、ガードレールなしの基準 ASR がベースモデル間で 2.8%（claude-3-7-sonnet）〜39.4%（gpt-4-turbo）と 14 倍開くことを示しながら、モデル間の差に一言も触れていない**——モデルの選択は一次のセキュリティ統制である
- メモ（原典内の矛盾・誤り。すべて訳注に記録）: CodeShield の対応言語が §1・§2.2 で **8**、§4.4 で **7**／**存在しない「付録 E」への参照が 2 か所**（実体は付録 C）／CodeShield の結果を「Figure 4 に示す」とあるが実際は **Figure 3**／**Figure 4 のキャプションは「攻撃成功率」だが中身は評価データの件数分布**で、図中の構成（悪性 289・良性 0・None/Benign 576 除外・攻撃タイプ 6 種）は本文の「600 シナリオ（良性 300・悪性 300）・7 技法」と**どの数字も合わない**／§4.1 の番号つきリストが 3 戦術を挙げながら 2 項目しかない
- メモ（**Figure 6 の軸ラベル誤りの検証**）: Figure 6 は縦軸ラベルが `ASR (%)`・図題も "ASR at Various Utility Reductions" だが、**プロットされている量は攻撃防止率（基準 ASR からの削減ポイント）** である。キャプション本文の記述とは一致する。図1 の値と数値照合して確定した——有用性低下 3% の点で PromptGuard 2 86M は約 14.5 を示し、17.63 − 3.3 = 14.33 に一致。ProtectAI PI detector も 17.63 − 13.7 = 3.9 に対し約 3.9、Deepset は 17.63 − 15.3 = 2.33 に対し約 2.4。**ラベルどおり「ASR」と読むと最良のスキャナが最悪に見えるという反転が起きる**
- データギャップ（次の取り込み候補）: **AgentDojo 原典**（Debenedetti ら 2024）——本 wiki で初めて言及されたが、本論文経由の二次記述しかない。プロンプトインジェクション防御の系列（**CaMeL**（Debenedetti ら 2025）・**Spotlighting**（Hines ら 2024）・**Instruction Hierarchy**（Wallace ら 2024））も、いずれも「LlamaFirewall が比較対象として言及するもの」としてしか登場していない。既存のギャップ（YaRN / ALiBi / DeepSeek-LLM / DeepSeek-V2 / Anthropic HH / context rot）は変わらず（**訂正 2026-08-03 lint**: この一覧に挙げた「lost in the middle」は、**2026-08-02 に [[summaries/2023-lost-in-the-middle]] として取り込み済み**であり未取得ではなかった。繰り越し時の確認漏れ）

## [2026-08-03] ingest | ガードレールの原典 2 本（Llama Guard / NeMo Guardrails）

- 取り込み: `raw/papers/Llama Guard_ LLM-based Input-Output Safeguard for Human-AI Conversations.md`（ケース A。arXiv:2312.06674, 2023-12, Meta）、`raw/papers/NeMo Guardrails_ A Toolkit for Controllable and Safe LLM Applications with Programmable Rails.md`（ケース A。arXiv:2310.10501, EMNLP 2023 System Demonstrations, NVIDIA）
- 作成: [[summaries/2023-llama-guard]], [[translations/2023-llama-guard]], [[summaries/2023-nemo-guardrails]], [[translations/2023-nemo-guardrails]]
- 更新: [[concepts/agent-safety-and-guardrails]]（**軸の新設と層 (2) の再構成**）, [[concepts/llm-programming-systems]], [[concepts/tool-use-and-function-calling]], [[summaries/2025-llamafirewall]], [[summaries/2024-llm-security-privacy-survey]], [[overview]], [[index]]
- 画像: `raw/assets/2023-llama-guard/` に 3 枚、`raw/assets/2023-nemo-guardrails/` に 10 枚（原典は図 11 個だが `jailbreak_rail_v2` を図4 と図7 で再掲しているため実体は 10 種）
- 対話での決定: (1) **概念ページに軸を立てて層 (2) も再構成する**。冒頭近くに `## もう一つの軸 — 制約はいつ書かれるか（embedded / programmable）` を新設し、層 (2) を **(2a) 内容を判定する検査器**（ガードレール＝モデル）/ **(2b) 会話と行動の形を制約するプログラム**（ガードレール＝プログラム）/ **2 つを統合した系（LlamaFirewall）** の 3 段に組み直した。(2) NeMo は 2023 年のトーキット論文なので **原典のみを扱い、冒頭に鮮度注記を置く**（現状の Web 裏取りはしない。古びた記述の検出は lint に回す）
- クリップの状態: **両方とも本文・見出し・図表は完全**（Llama Guard は Figure 1〜3・Table 1〜6、NeMo は Figure 1〜11・Table 1〜3）。**脚注の本文が全滅**（Llama Guard 7 件・NeMo 11 件）しており、すべて原ページから復元した。大半は URL だが **Llama Guard の脚注 4 は読解に必要な用語の断り書き**（本論文の「prompt」はエージェントへのプロンプトを指し、Llama Guard 自身へのものは「Llama Guard prompt」と呼ぶ）。Llama Guard の図2・図3 のキャプション内で引用参照が裸の数字に化けていた箇所も復元
- なぜ新概念ページを作らなかったか: 2 本とも既存の `agent-safety-and-guardrails` の対策 4 層のうち **(2) 入出力のガードレール**に属する。ただし**同じ層に属しながら、やっていることが別種**（内容の判定 vs 会話の形の制約）だったので、ページを増やす代わりに**層の内部を割った**。加えて NeMo が提示する embedded / programmable は 4 層と直交する軸なので、4 層の手前に独立節として立てた
- メモ（Llama Guard）: 中心的発明は **安全ポリシーを重みでなくプロンプトで渡した**こと。ポリシー変更が再学習でなくテキスト編集になる。実装の勘所として **`safe`/`unsafe` を SentencePiece の単一トークンに選び、最初のトークンの確率をそのまま分類器スコアにする**、**prompt 分類と response 分類を同一モデルへの指示文の書き分けだけで分ける**、**「カテゴリを削ってラベルも合わせる」データ拡張で条件付き判定を教える**の 3 点が転用できる。訓練は Llama2-7b・13,997 件・500 ステップと小規模
- メモ（Llama Guard の批判点）: 自社テストセットの 0.945/0.953 は**自社タクソノミー・自社レッドチームのラベル・自社テストセット**上のもので、比較対象の API は完全にオフポリシー。カテゴリ別の 0.798 対 0.035 は**品質差でなくカテゴリの有無**である（論文自身が §4 で説明してはいる）。外部データセット ToxicChat の **0.626** が素の実力に近い。ただし**評価プロトコルは自分に 1-vs-all（厳しい側）・ベースラインに 1-vs-benign（自ら「楽観的な結果を生みうる」と認める側）** を当てており、この点は公正。データはプロンプトが Anthropic hh-rlhf 由来・**応答は社内 Llama チェックポイントの生成**なので「安全でない応答」の分布が Llama 自身のもの。英語のみ
- メモ（Llama Guard 付録の表が最も情報量が多い）: しきい値 0.5 固定で **OpenAI Mod API は適合率 0.874 だが再現率 0.250**、Perspective は 0.219——**既定設定では安全でないプロンプトの 4 分の 3 を見逃す**。逆に **GPT-4 をゼロショット判定器にすると過剰遮断**で、Guns & Illegal Weapons は適合率 0.052/再現率 0.971、Regulated Substances は 0.176/1.000、Self-Harm は 0.039/1.000 と**ほぼ全部を unsafe と言う**。「強いモデルにゼロショットで判定させればよい」への具体的な反証であり、2 年後の [[summaries/2025-llamafirewall]] が Llama 3.2 1B で見せた過剰遮断と同じ失敗モード
- メモ（Llama Guard §6）: **「LLM として、Llama Guard は意図された用途を改変あるいは迂回しうるプロンプトインジェクション攻撃に対して脆弱でありうる」と 2023-12 に明記されている。緩和策はない。** 実装で答えるのは 1 年半後の LlamaFirewall（入力を CoT と行動に限定 ＋ PromptGuard で事前走査）。「LLM をガードレールにすると LLM の脆弱性を継承する」という問題は、この系譜の最初から自覚されていた
- メモ（NeMo）: 最大の貢献は手法でなく **embedded / programmable という軸の提示**。機構はタスク指向対話（Rasa・DialogFlow の NLU＋DM）の再発明で、**閉じた intent 集合を canonical form（LLM が生成するので閉じていない）に置き換え、ベクトル DB から類似例を引いて誘導する**。**topical rails は内容を判定せず「可能な会話の形」を制約するので、実質的には層 (1) の行動空間設計に近い**——本 wiki で層 (2) に置いているのは実装上同じツールキットに同居しているからにすぎない、と明記した
- メモ（NeMo の批判点）: 3 段の CoT が逐次連鎖する（第 2 段の入力が第 1 段の出力）ので**バッチ不可・通常の約 3 倍のレイテンシとコスト**。**§7.1 が「とりわけ安全性に固有のレールについては単独の解決策として用いるべきではない」と明記**し、付録 A も「そのまま使える安全機能としてではない」と書く。評価はデモ規模（topical 231 サンプル / モデレーション 200 サンプル / hallucination **20 問**）。**検査器と被検査者が同じモデルでありうる**（gpt-3.5-turbo が生成し gpt-3.5-turbo が検査する構成を含む）
- メモ（NeMo で最も示唆的な結果）: **中核機構の精度で gpt-3.5-turbo が 4 モデル中の最下位**（Banking 38%/45%、text-davinci-003 は 77%/83%、falcon-7b-instruct 70%/76%、llama2-13b-chat 76%/77%）。理由は能力不足でなく「**少数ショットを使ってさえより多様な canonical form を生成する**」こと。付録 G 表3 が裏付けで、**完全一致 0.38 → 類似度マッチ（0.6）0.73** とほぼ倍増する（dolly-v2-3b も 0.32→0.62）。**出力を後段の機械が消費する系では、指示に創造的に応じる能力がそのまま不利になる**——構造化出力・制約デコーディングの問題そのもの（→ [[llm-programming-systems]]）。もう 1 つ、**canonical form ごとにベクトル DB へ最低 3 例**（k=all 0.77 / k=3 0.65 / k=1 0.50）
- メモ（NeMo の数字の決着とコードのバグ）: hallucination rail について**本文 §5.2 は「95% まで押し上げる」、図6 は 90%、付録 G.2.3 は「25% 押し上げる」** と 3 つの数字が併存する。**65% ＋ 25 ポイント = 90% なので図と付録が整合し、本文の 95% が誤り**と確定できる。§5.1 の「上位 3 つのモデルを図5 に示す」も**図には 4 モデル**。そして**付録 E.2 に印刷された jailbreak rail のコードが動かない**——`jailbreak_chain.apredict(bot_response=bot_response)` を呼んでいるがテンプレート変数は `user_input` で `input_variables` も `["user_input"]`、しかも `bot_response` はそのスコープで未定義（E.1 からの写し間違い）。判定も `if "no" in check` の部分文字列一致で脆い。**論文に印刷されたガードレールのコード自体が壊れている**のは、2023 年時点のこの分野の成熟度の記録として読める
- メモ（系譜の整理）: 2023 年秋〜冬に**同じ問いへ正反対の 2 つの答え**（ガードレール＝モデル / ガードレール＝プログラム）が出て、**2 年後の LlamaFirewall が両方を取った**——PromptGuard と AlignmentCheck は Llama Guard の子孫、統一ポリシーエンジン（カスタムのパイプライン・条件つき是正戦略・差し込める検出器）は NeMo の子孫。さらに **AlignmentCheck の構造は Llama Guard の構造そのもの**である——判定の基準（Llama Guard はタクソノミー、AlignmentCheck はユーザーの目標）を**プロンプトで渡すことで単一のモデルを任意の基準の判定器にする**。「何を違反と呼ぶかを実行時パラメータにする」という発明は、対象を変えて生き延びた
- データギャップ（次の取り込み候補）: **Anthropic hh-rlhf（2204.05862）** ——Llama Guard のプロンプトの出所であり NeMo のモデレーション評価データでもある。既存のギャップ一覧に載っていたが、これで**2 本の原典が同時に依存する**状態になったので優先度が上がった。**AgentDojo 原典**（Debenedetti ら 2024）、**SelfCheckGPT**（Manakul ら 2023、NeMo の hallucination rail の下敷き）、**ToxicChat**（Lin ら 2023）、**OpenAI Moderation の原典**（Markov ら 2023）も未取得。CaMeL / Spotlighting / Instruction Hierarchy は前回から変わらず

## [2026-08-03] lint | wiki 全体の健康診断と記録の不整合の修正

- 走査: wiki 配下 165 ページ（concepts 28 / summaries 67 / translations 66 / questions 1 / root 3）・約 396 万字
- **健全だった点**: 孤立ページ **0 件**／dangling link は実質 **1 件**（`web-agents`。他 2 件は index・overview のテンプレ行）／index.md への掲載漏れ **0 件**／log.md の見出し形式 **63/63 準拠**／frontmatter の wikilink 引用符違反 **0 件**／どの概念ページからも参照されない summary **0 件**／個別手法・製品・ベンチマーク・人物/組織の専用ページ **0 件**（CLAUDE.md §1 の粒度規約どおり）。陳腐化しやすい記述も、確認範囲ではすべて時点が明記されていた（「K2.5 評価時点（2026 年初頭）」「当時（2025）の SOTA」等）
- **修正 1** — `index.md` の「未作成の想定スラグ」行が古かった。`parameter-efficient-fine-tuning` は 2026-08-02 に作成済みなのに未作成扱いのままだったので除去し、残る `web-agents` には「本 wiki に原典が 1 件もないテーマ」である旨を併記した
- **修正 2** — 概念ページの frontmatter `summaries` に、**本文で一度も引用していない原典が 11 件**残っていた。`summaries` は「この概念の根拠となる原典」（CLAUDE.md §2）なので、宣言だけで本文に現れないのは規約違反にあたる。**原因ははっきりした型で、分割の残骸**である——量子化を `model-quantization` へ、MoE を `mixture-of-experts` へ、位置符号化を `positional-encoding` へ切り出した際に、親ページの出典欄を刈らなかった。内訳と処置:
	- 削除 10 件: `llm-inference-optimization`（2025-llm-quantization-guide / -explained / 2025-moe-survey）、`transformer-architecture`（2023-moe-explained / 2025-moe-survey / 2025-deepseek-r1）、`mixture-of-experts`（2026-deepseek-v4 / 2026-kimi-k2.5）、`llm-agents`（2026-ai-scientist）、`research-agents`（2026-sakana-fugu）
	- **本文に引用を足したもの 1 件**: `llm-inference-optimization` の `2021-roformer`。本文（FlashAttention-3 の非干渉化処理の節）が「直前の rotary embedding へ融合すれば追加コストもない」と論じているのに出典を引いていなかっただけなので、削除でなく本文側を直した——**RoPE がブロック対角の回転だけで実質メモリ帯域律速だからこそ、そこが「ついでに何かを混ぜ込める場所」になる**という接続を明記し、[[positional-encoding]] と [[summaries/2021-roformer]] へリンクした
	- なお `2021-roformer` の登録は 2026-08-03 の RoFormer ingest で私が入れたもので、**同じ型をその場で作っていた**
- **修正 3** — 2026-08-03 の **2 つの** ingest（RoFormer・LlamaFirewall）のデータギャップ欄に、**前日 2026-08-02 に取り込み済みの lost in the middle（Liu et al. 2023）を「未取得」として繰り越していた**。両方の行に訂正を追記した。同じ一覧を毎回コピーして持ち回るやり方の弱点で、**繰り越す前に実在を確認する**手順が要る
- **検出のみ（未対応）— 根拠が薄い概念ページ**:
	- `computer-use-agents`: 根拠原典 **1 本**（Kimi K2.5 のみ）かつ `updated` が **2026-07-28 で全概念ページ中最古**。CUA は最も変化が速い領域で、OSWorld 63.3% / WebArena 58.9% といった数値が単一原典に乗っている
	- `agent-observability`: ページ自身が「可観測性そのものを主題とする一次資料はまだ ingest していない」と明記（5 本の summaries はすべて他テーマからの間借り）
	- `model-context-protocol`: 根拠 **2 本、いずれも二次資料**。**MCP の仕様そのものが未取得**で、版を重ねる領域なので陳腐化リスクが高い
	- `llm-programming-systems`: 2 本（ページ自身が根拠の薄さを明記済み・許容範囲）
- **検出のみ（未対応）— 分割圧力**: `agent-safety-and-guardrails` と `reinforcement-learning-from-human-feedback` がともに **24k 字**で最大。前者は 2026-08-03 に 2 回加筆し層 (2) が 3 段構造になった。即分割ではないが、次にガードレール系を 1〜2 本入れると検討域に入る
- **データギャップ（優先度つきで再整理）**:
	- **高（複数の既存原典が同時に依存）**: **Anthropic hh-rlhf（2204.05862）** ——Llama Guard のプロンプト出所 ＋ NeMo のモデレーション評価データ ＋ RLHF 系譜の基点の 3 方向／**AgentDojo（Debenedetti ら 2024）** ——`agent-evaluation` に防御評価の節まで作ったのに一次資料なし／**YaRN（Peng ら 2023）** ——`positional-encoding` が明示的にギャップ宣言
	- **中**: ALiBi（Press ら 2021）・SelfCheckGPT（Manakul ら 2023）・ToxicChat（Lin ら 2023）・OpenAI Moderation（Markov ら 2023）・CaMeL / Spotlighting / Instruction Hierarchy
	- **繰り越し**: Tree of Thoughts・Self-Refine・LoRA 本体・DeepSeek-LLM / V2・context rot（Chroma）・AI Scientist v1
	- **テーマとしての空白**: **web agents**（CLAUDE.md §1 が想定スラグに挙げ overview がリンクしているのに原典ゼロ）
- **修正 4（computer-use-agents の鮮度照合）** — ユーザー選択により、根拠 1 本・`updated` 最古だった `computer-use-agents` の記述を Web で照合した。**古くなっている記述を 1 件確認**:
	- 旧記述は「人間は 70%+ とされ、63〜66% のフロンティアモデルはこれに接近しつつある」だった。しかし **2026-08 時点の OSWorld-Verified リーダーボードの上位は 85% 前後**（Claude Mythos 5 / Claude Fable 5 が 85%、Claude Opus 4.8 が 83.4%）で、**原典（Xie ら, NeurIPS 2024, arXiv:2404.07972）が報告する人間の基準線 72.36% をすでに 10 ポイント以上上回っている**。つまり「人間に追いつけるか」という枠組み自体が失効し、問いは「**このベンチマークが飽和したあと何を測るか**」へ移っている
	- 同原典から**タスク数 369**・**当時の最良モデル 12.24%** も確定できたので、曖昧だった「数百タスク」を修正した
	- 処置: K2.5 レポート由来の 63.3%（2026 年初頭の一次資料）はそのまま残し、**日付と出所を明記した鮮度注記ブロック**を併置した。**リーダーボード集計サイト由来で本 wiki の一次資料ではない**旨も明記している
	- **`updated` を 2026-07-28 → 2026-08-03 へ**
- **データギャップに追加**: **OSWorld 原典（arXiv:2404.07972, Xie ら, NeurIPS 2024）**。人間基準線 72.36%・369 タスク・評価スクリプトの設計といった、`computer-use-agents` と `agent-evaluation` の双方が依拠する数値の出所が二次経由のままである。上記「高」優先の 3 本（hh-rlhf / AgentDojo / YaRN）に次ぐ候補
- ユーザーの指示: 「記録の不整合 3 件を直す」＋「computer-use-agents を補強する」を選択。**両方とも本エントリで完了**

## [2026-08-03] ingest | 攻撃と選好データの原典 2 本（PAIR / Anthropic HH）

- 取り込み: `raw/papers/Jailbreaking Black Box Large Language Models in Twenty Queries.md`（ケース A。arXiv:2310.08419, 2023-10, UPenn）、`raw/papers/Training a Helpful and Harmless Assistant with Reinforcement Learning from Human Feedback.md`（ケース A。arXiv:2204.05862, 2022-04, Anthropic）
- 作成: [[summaries/2023-pair]], [[translations/2023-pair]], [[summaries/2022-anthropic-hh]], [[translations/2022-anthropic-hh]], **[[concepts/llm-red-teaming]]（新設）**
- 更新: [[concepts/agent-safety-and-guardrails]]（**攻撃側を切り出して防御側に集中**）, [[concepts/reinforcement-learning-from-human-feedback]]（HH の 4 つの寄与を追加）, [[concepts/agent-evaluation]], [[concepts/self-reflection]], [[concepts/tool-use-and-function-calling]], [[summaries/2022-instructgpt]], [[summaries/2022-rlhf-illustrated]], [[summaries/2023-llama-guard]], [[summaries/2023-nemo-guardrails]], [[summaries/2025-llamafirewall]], [[summaries/2025-cot-faithfulness]], [[summaries/2024-llm-security-privacy-survey]], [[overview]], [[index]]
- 画像: `raw/assets/2023-pair/` に 11 枚、`raw/assets/2022-anthropic-hh/` に **71 枚**
- **lint で最優先に挙げた Anthropic hh-rlhf（2204.05862）のギャップが解消**した。本 wiki の 3 方向（Llama Guard のプロンプト出所・NeMo Guardrails のモデレーション評価データ・RLHF 系譜の基点）が依拠していた一次資料である
- 対話での決定: (1) **`llm-red-teaming` を新設して攻撃側を切り出す**（lint で「次にガードレール系を 1〜2 本入れると分割検討域」と判定していた `agent-safety-and-guardrails` が 24k 字だったため）。**分量だけが理由ではない**——レッドチーミングは**防御の前段・訓練データの生産工程・評価の一部**の 3 つを兼ねており、片方に最適な設計が他方を壊す、という主題が独立に立つ。(2) HH の付録 C（PALMS / InstructGPT / LaMDA プロンプトへのモデル出力のサンプル集、約 300 行）は**位置づけとプロンプトを訳し、モデルの応答本体は英語原文のまま**収録（プロンプトと同じく一字一句が挙動の証拠であるため）
- **クリップの状態（HH は本 wiki で最悪）**: **画像 71 枚中 29 枚が欠落**（`x4, x6, x8, x11, x13, x15, x17, x21, x24, x28, x30-32, x34, x36, x38, x42, x44, x50, x52, x54, x57, x60, x62, x64, x66, x68, x70, x71`。多パネル図の 2 枚目以降が系統的に脱落する既知の型が大規模に起きたもの）／**§1.1「Dialogue Preference Datasets」が見出しだけで中身が空**（箇条書き 2 項目が丸ごと消滅）／**Figure 37 のキャプションが消滅**／**脚注 20 件がマーカーごと痕跡なく消滅**／§4.3 の見出しが LaTeX 崩れ。**全 71 枚を取得し直し、欠落部はすべて原ページから復元**した。復元した脚注には **hh-rlhf の URL（脚注 3）** と**符号反転バグの逸話（脚注 20）** が含まれ、いずれも本文の理解に必要である。なお ar5iv 上の `x19.png` は実在するがどこからも参照されておらず、保存対象から外した（代わりに `x72.png` が Figure 43 で使われている）
- クリップの状態（PAIR）: **本文・見出し・表はすべて完全**。**画像 11 枚中 1 枚（`x5`＝Figure 4 の右パネル）と脚注 6 件**のみ欠落し、原ページから復元した
- メモ（HH の最重要点）: **本 wiki が 3 回別々に観測してきた過剰遮断の機構**が §4.4 に書かれていた。無害性データでクラウドワーカーに**より有害な**応答を選ばせた結果、**データセットは「何をすべきでないか」しか教えず**、しかも「**レッドチーミングのプロンプトで良いスコアを得るには『それには答えられません』で十分**——これは分類さえできればよいので有用性より学習が容易」。実測でも方策の無害性スコアは分布の上側の裾へ出て（過剰最適化）、有用性はオンディストリビューションのまま（過少最適化）。著者らは望ましい挙動を「**hostage negotiator**」と名付け、**「無害な対話モデルの訓練に取り組む他の人には、ユーザーが会話をより有益な方向へ動かす応答を主に選ぶ形でデータを集めることを推奨する」と自分たちの設計を名指しで否定している**。既存の観測（[[summaries/2023-llama-guard]] のゼロショット GPT-4 が適合率 0.039〜0.052／[[summaries/2025-llamafirewall]] の Llama 3.2 1B が ASR 最良で有用性 1/9／[[summaries/2023-nemo-guardrails]] が有用な要求まで遮断）に**機構による説明**が付いた
- メモ（HH のその他）: **アラインメント税がモデルサイズで符号を変える**（10B 未満は税、13B/52B はほぼ全評価でボーナス。脚注 5 が「**小さいモデルだけのアラインメント研究は素朴に外挿すると誤った結論を導きうる**」）／**√D_KL と報酬がおおよそ線形**でモデルサイズをまたいで平行、機構の説明つき（$D_{KL}$ の展開が 2 次から始まる）。含意として「**RLHF は新しい技能を教えているのか、既存挙動の部分分布に絞っているだけなのか**」をこの線形領域として定義しなおせるかもしれない／**PM は高スコア域で較正も頑健性も崩れる**（train/test PM 分割で 15 万サンプル付近から乖離、大きい PM ほど頑健）→ 反復オンライン RLHF、効果は**データ量とハイパーパラメータを揃えた統制実験**で確認／**OOD 検出は有害例 10 個で AUROC 0.94、13M モデル＋10 例（0.86）が 52B の最良（0.85）を上回る**
- メモ（HH の批判点・自己申告）: **平均ケースしか見ていない**と §7.1 で明記し、最悪ケースは Perez ら 2022 に委ねる（→ PAIR）／「**表面的にアラインしているだけかもしれない**」／ジェンダーバイアスは **RLHF 後も基盤 LM と強く相関**／BBQ-Lite は著者自ら「有意義である可能性は低い」と警告／評価書式（Gopher 形式）が「**誤解を招きうる grok 曲線の外観**」を作る／PM は**モデル生成サンプルのみで訓練したため敵対的に頑健でない**（人間が書いた分布外テキストに騙される）／脚注 13「**PM スコアをそのまま報酬に使う良い理由はない**」・$\lambda_{KL}=0.001$ は「**まったく不要かもしれない**」／研究者とクラウドワーカーの一致率**約 63%**（InstructGPT の 72.6% より低いと自認）、**約 20 名が全データの 80%**、選別は一致度でなく**執筆の洗練度**（未検証の推測）、付録 D.3 が**ゴールドラベル方式が使えなかった理由**を正直に書き「洗練されたデータ品質の統制なしに結果を達成できたことは注記に値する」
- メモ（PAIR）: **HH §7.1 が指した最悪ケースの頑健性を攻撃側から測った**仕事。**平均 11.9〜33.8 クエリ・約 5 分** vs GCG の **256,000 クエリ・約 150 分**。成功率 Vicuna 100%・PaLM-2 72%・GPT-4 62%・GPT-3.5 60% に対し **Llama-2 10%・Claude 6%**、転移では **Claude が全設定で 0%**。**アブレーションが最も示唆的**——攻撃例を消しても 72%→70% しか落ちないのに、**`improvement`（前回なぜ失敗したかを書かせる CoT）を消すと 72%→56%**。**探索を駆動しているのは例示でなく自己反省**である（→ [[self-reflection]] に追記）。**弱いモデルのほうが良い攻撃者**（Vicuna 72% > GPT-3.5 58%。安全策を欠く／システムプロンプトへの忠実度／出力形式の制御）——**防御の評価に自社のアラインされたモデルを攻撃者に使うと脅威を過小評価する**。深さは 2 で足り、K>50 では生成ループに落ちる
- メモ（PAIR の批判点）: **評価がすべて GPT-4 の判定に乗っているのに、その判定器を検証していない**（根拠は別タスクについての先行研究。本 wiki は [[summaries/2023-dpo]] から「人間同士の一致率を基準線に置く」規律を持っている）／標的の生成を **150 トークンで打ち切って**判定しており「**冒頭が破られた**」と「**有害な成果物が完成した**」を区別していない／評価用 50 件は**著者自身が AdvBench から選別**（520 件中 24 件が爆弾製造という重複が理由で、動機は妥当だが選別者と提案者が同一）／温度 0 の決定論的生成／モデル世代が 2023 年秋で完全に古い
- メモ（本 wiki で 2 度目のパターン）: 「**より整合し／より創造的に応じる能力が、限定された役割を果たす系では不利に働く**」。PAIR では GPT-3.5 が攻撃者として Vicuna に負け（アラインメント自体が原因）、[[summaries/2023-nemo-guardrails]] では gpt-3.5-turbo が対話フローの中核機構で 4 モデル中最下位だった（出力形式の揺れが原因）。**現象は同型だが原因は異なる**ので、両ページに相互参照を張った
- 概念ページの再構成: `agent-safety-and-guardrails` の `## 脅威モデルの俯瞰`（2,906 字）を `llm-red-teaming` へ移し、防御ページ側には**エージェント固有の脅威を 3 つに束ねた最小限の導入**（外部由来の乗っ取り／権限過多と誤操作／モデル内在の逸脱）と誘導だけを残した。**24k → 22k 字**。`llm-red-teaming` にはさらに「なぜジェイルブレイクは消えないのか（2 つの失敗モード）」「攻撃の 2 系統」「攻撃の自動化と 3 つの設計則」「攻撃データを訓練へ戻すと過剰拒否を作る」「攻撃側から見た評価の規律」「本 wiki における根拠の所在」を新設した
- データギャップ: **Anthropic hh-rlhf（2204.05862）を解消。** 新たに攻撃側の空白が明確になった——**GCG（Zou ら 2023）** は本 wiki のトークンレベル攻撃の記述がすべて PAIR 経由であり、**Perez ら 2022（Red teaming language models with language models）** は HH が明示的に指し PAIR の直接の先行研究であるのに未取得、**Ganguli ら 2022（Anthropic のレッドチーミング論文）** は HH が「今後の刊行物で議論する」と述べた先。いずれも `llm-red-teaming` の骨格が依拠している。既存のギャップ（**AgentDojo** / YaRN / ALiBi / OSWorld 原典 / SelfCheckGPT / ToxicChat / OpenAI Moderation / DeepSeek-LLM / DeepSeek-V2 / context rot / Tree of Thoughts / Self-Refine / LoRA 本体 / AI Scientist v1）は変わらず

## [2026-08-03] ingest | 攻撃を「測る」側の原典 2 本（HarmBench / JailbreakBench）

- 取り込み: `raw/papers/HarmBench_ A Standardized Evaluation Framework for Automated Red Teaming and Robust Refusal.md`（ケース A。arXiv:2402.04249, ICML 2024, Center for AI Safety・UIUC・CMU）、`raw/papers/JailbreakBench- An Open Robustness Benchmark for Jailbreaking Large Language Models.pdf`（**ケース B**。arXiv:2404.01318v5, NeurIPS 2024 D&B, UPenn・ETH Zurich・EPFL・Sony AI）
- 作成: [[summaries/2024-harmbench]], [[translations/2024-harmbench]], [[summaries/2024-jailbreakbench]], [[translations/2024-jailbreakbench]]
- 更新: [[concepts/llm-red-teaming]]（**評価の標準化を扱う新節**・第 3 の攻撃系統・R2D2・設計論点 4 件追加）, [[concepts/agent-evaluation]]（**測定手続きのハイパーパラメータ**節と**判定器を「選ぶ」手続き**節を新設）, [[concepts/agent-safety-and-guardrails]]（**層 (2c) 入力を加工する軽量防御**を新設）, [[summaries/2023-pair]]（**訂正ブロックを追加**）, [[overview]], [[index]]
- 画像: `raw/assets/2024-harmbench/` に 17 枚（`x1`〜`x17`、すべて ar5iv から取得）、`raw/assets/2024-jailbreakbench/` に 2 枚（`fig1.png`＝埋め込み画像、`fig2.png`＝PyMuPDF の領域レンダリング）
- 対話での決定: (1) **[[summaries/2023-pair]] に訂正ブロックを置く**——PAIR が検証しないまま使った判定器は後に同じ著者たちが検証して**一致率 90.3% で妥当だった**こと、150 トークン問題は HarmBench が**ASR を最大 30% 変える**と定量化したこと、そして PAIR 自身の ASR が JBB 設定では GPT-4 で 62% → 34% になることを、限界節の冒頭に日付つきで追記した。(2) **HarmBench は付録 E（X-Risk Sheet）も全訳する**——安全性研究が能力向上に寄与してしまう度合い（safety-capabilities balance）を著者に自己申告させる様式自体が、本 wiki に無かった視点であるため
- クリップの状態（HarmBench）: **本 wiki がこれまで扱った中で最も状態が良いクリップの 1 つ**。ar5iv の 84 見出しと照合して、**画像 17/17・図表キャプション 29/29・表 12/12 がすべて残存**。欠落は**付録 B.5.2「Copyright Classifier」が見出しごと 1 節まるごと**のみで、原ページから復元した（著作物の重なり合うチャンクをハッシュ化し MinHash でソフトマッチを捉える判定器。**なぜ著作権だけ判定基準を変えたか**という論文の設計の要になる節なので、欠けたままにはできなかった）
- 原典の状態（JailbreakBench）: PDF なのでケース B の手順。本文は `pdftotext -layout`、図はキャプション位置を境界とした領域レンダリング。**fig2 の切り出しは 3 回やり直した**（1 回目はグラフ上端が切れ、2 回目は左段の本文が混入、3 回目 `Rect(299,119,512,247)` dpi 220 で確定）。**表 11 個のうち 5 個はシステムプロンプト**なので英語原文のままコードブロックへ収録。付録 G〜I（NeurIPS チェックリスト・データカード）は定型の提出書式なので訳出していない。Figure 3 は 2 プロットからなる図だが本文記述で完全に代替できるため数値のみ本文へ収録した
- メモ（最重要 — 判定器の実測）: JailbreakBench が **300 例（攻撃 200 ＋ XS-Test の境界的な良性 100）に 3 名の専門家がラベル付け（相互一致率およそ 95%）** して 6 判定器を比較した表が、本 wiki のこれまでの記述をいくつも裏づけ・修正した。**文字列照合は一致率 56.0%・偽陽性 64.2%**（初期文献の ASR は額面で読めない）／**[[summaries/2023-llama-guard]] は偽陰性 60.9%**（安全性分類器を成功判定に転用すると 6 割見逃す。Llama Guard 2 では 10.9% まで改善）／**HarmBench 判定器は偽陽性 26.8%** で、内訳は XS-Test の境界的な良性例に集中する——**HarmBench 自身の前提資格試験は「ランダムな良性段落」を 98.0 で通しているので、ランダムな良性は弾けるが有害に見える良性は弾けない**。**判定器の頑健性はテストした軸についてしか保証されない**という教訓は、2 本を並べて初めて見えた
- メモ（HarmBench の要点）: **測定手続きのパラメータが比較不能の犯人だった**——生成トークン数だけで ASR が最大 30% 動く（$N=512$ へ標準化。JailbreakBench は 150 なので**2 つの標準の値が違い、両者の ASR も直接は比較できない**）。**機能カテゴリ**（Standard 200 / Copyright 100 / **Contextual 100** / Multimodal 110）は意味カテゴリと直交する軸で、文脈的挙動は原典自身が「アシスタントまたは自律的なハッキングエージェント」を模したと述べており、**エージェントに有害要求が来る形はこれ**である（検索可能率 0%、AdvBench 50%・MaliciousInstruct 55%）。**挙動と判定器の両方を validation/test に分ける**（「研究者が挙動ごとに手作業でハイパラを調整するなら、その手法はもはや自動ではない」）。**18 攻撃 × 33 LLM** から: **どの攻撃も防御も一様には効かない**・**頑健性はモデルサイズと独立**（6 ファミリー・7B〜70B。効くのは訓練データと手続き。例外は著作権挙動で、そこだけ逐語再現の能力が要る）
- メモ（R2D2）: GCG は A100 で 1 テストケース 20 分と遅いので、**永続的テストケースのプール（180 個、毎回 8 個を抜いて GCG を 5 ステップだけ進める）** で敵対的訓練する。損失は away ＋ toward ＋ SFT の 3 本立て。**8×A100 16 時間**で Zephyr 7B の GCG ASR を **5.9**（Llama 2 7B Chat 31.8・13B Chat 30.2）まで落とし、MT-Bench 6.0（Mistral 7B Instruct v0.2 が 6.5）。ただし**PAIR・TAP など性質の違う攻撃には効きが薄い**——「1 種類の攻撃に敵対的訓練して汎化を期待する」戦略は成り立たない
- メモ（JailbreakBench の要点）: **ジェイルブレイク・アーティファクトの公開リポジトリ**が中核の貢献であること自体が状況を物語る（大半の攻撃論文はプロンプトを公開せず、集積サイトはオフラインになった）。**有害 100 に話題が 1 対 1 で対応する良性 100** を持ち、これで測ると **Llama-2 7B は防御なしで良性挙動の 65% を拒否する**——HarmBench が「最も頑健」と評価したそのモデルである。**[[summaries/2022-anthropic-hh]] の hostage negotiator 問題の、最も明快な実測値**。攻撃側では **Prompt with RS（テンプレート＋ランダム探索）が Llama-2 を 90%・GPT-4 を 78%、Vicuna では平均 2 クエリ**で破り、**パープレキシティフィルタは自然言語攻撃に無力**（Llama-2 で 73% 通す）——**「入力が不自然か」を見る防御は、攻撃が自然言語になった瞬間に無力化する**
- メモ（2 本の設計思想の対立）: **攻撃の実装を標準化するか否か**。JailbreakBench は「テスト時防御を正しく測るには**その防御に合わせた適応的攻撃**が要る」から攻撃を固定しない。HarmBench は**同じ困難**を理由に、システムレベル防御の評価そのものを見送った（「本当に頑健なのか、単にテストが甘いだけなのかを判定できない」）。**同じ壁に対して、片方は評価対象から外し、もう片方は攻撃を自由にして受け入れる**という逆の判断をしている。結果として**実務で使われる防御の大半（フィルタ・ラッパー）は HarmBench の枠外**にある
- メモ（X-Risk Sheet という様式）: CAIS / Hendrycks 系に特徴的な付録で、著者に「**この研究が一般的能力の向上に寄与し、かえってリスクの到来を早める道筋はあるか**」を自己申告させる。HarmBench の回答は率直で、**自動レッドチーミングのツールは AI の信頼性を高め、より自律的な設定で配備する経済的誘因を強めうる**（防御の脆弱性でなく通常タスクの失敗事例の探索にも転用できる）。また Q7「人間の信頼性への依存」に **☒** をチェックし、「**有害挙動の選定を含むパイプライン全体の自動化は未達で、何を有害とみなすかは文脈依存なので完全な自動化は困難あるいは望ましくないかもしれない**」と補足している
- データギャップ: **AdvBench・TDC の挙動データセットは 2 本経由で間接的に押さえられた**。**GCG（Zou ら 2023）** は HarmBench 付録 C.1 の手法記述と R2D2 での実装により以前より薄さは緩和されたが、依然として未取得。**Perez ら 2022（Red teaming language models with language models）** も未取得のまま。新たに **Andriushchenko ら 2024（Prompt with RS）** が「本 wiki で最強の攻撃として扱っているのに JailbreakBench 経由でしか知らない」文献として加わった

## [2026-08-03] ingest | TAP — Tree of Attacks with Pruning（PAIR の一般化）

- 取り込み: `raw/papers/Tree of Attacks_ Jailbreaking Black-Box LLMs Automatically.md`（ケース A。arXiv:2312.02119, 2023-12, Yale University・Robust Intelligence・Google Research）
- 作成: [[summaries/2023-tap]], [[translations/2023-tap]]
- 更新: [[concepts/llm-red-teaming]]（**PAIR 節の後ろに TAP 節を新設**——木にする理由・枝刈りは配分の機構・失敗の伝播・評価器の偽陽性が探索を殺す・Llama-2 の拒否優先）, [[concepts/reasoning-and-planning]]（**ToT 節に「木は鎖より本当に強いのか」小節を新設**）, [[concepts/agent-evaluation]]（判定器の欠陥が使う場所で症状を変える話を判定器選定の節へ追記）, [[overview]], [[index]]
- 画像: `raw/assets/2023-tap/jailbreak-v2.png` 1 枚のみ（本論文の図は Figure 1 だけが画像で、Figure 2〜15 はすべてインライン SVG のテキスト箱）
- 対話での決定: (1) **付録 B のジェイルブレイク実例（SVG テキスト箱 75 個、Figure 3〜15）を全訳する**——skill 既定どおり付録も全訳し、SVG テキスト箱は画像化せずテキストに起こす方針にも合致する。攻撃プロンプトと標的の応答が両方読め、後から grep して引用できる。(2) **概念ページは `llm-red-teaming` に加えて `reasoning-and-planning` も更新する**——**ToT 原典（Yao et al. 2023）が未 ingest で概説しかなかったところに、木探索が鎖に勝つことの本 wiki 初の統制された実測値が入る**ため
- クリップの状態: **見出し 31・図表キャプション 25 件（Figure 1〜15・Table 1〜9・Algorithm 1）・表 12 個は完全**、SVG テキスト箱も 78 個中 75 個が残存。欠落は 3 種類でいずれも原ページから復元した——**(1) §1 の「3 つの key properties」箇条書きが丸ごと欠落**（Automatic / Black-box / Interpretable。本論文が扱う攻撃の定義そのもので、直後の「上記の性質をすべて満たす手法に焦点を当てる」が受ける先が消えていた）、**(2) 脚注 7 件が全滅**（マーカーが参考文献番号 `[^N]` に化けていた。脚注 1 は GCG の非可読サフィックスの実例、脚注 5 は「**PAIR の成功率が原論文と違う理由**」の自己申告＝成功例だけで平均を取るか全件で取るか、で内容として重要）、**(3) Figure 13〜15 の「木の枝（矢印）」を描く SVG 3 個が欠落**
- メモ（クリップ不良の新しいパターン）: (3) は本 wiki で初めて見る形の不良である。**箱（ノード）はすべて残っているのに、それらを木に繋ぐ矢印だけが消えている**——つまり「**これは一本の鎖ではなく木である**」という図の主眼そのものが失われる。原ページの `<path d="M x y L x y">` 座標から分岐構造（Fig13 は根から 3 分岐 / Fig14 は 2 分岐 / Fig15 は直線＋2 分岐）を確認し、各図の冒頭に訳注として記した。**「何が欠けたか」だけでなく「欠けたことで何が読めなくなるか」を見る**必要がある例
- メモ（本論文の構造の良さ）: 著者らが「**TAP は分岐数 b=1 かつ話題ずれの枝刈りを無効にすると PAIR に一致する**」と形式的に述べているので、**TAP と PAIR の差分がそのままアブレーションになっている**。手法論文としては理想的な提示の仕方で、本 wiki の他の原典にはあまりない形
- メモ（アブレーション 3 件が本体）: **(a) 枝刈りを外すとクエリが 2.5 倍（22.5 → 55.4）になるうえ成功率も 12 ポイント落ちる**（84% → 72%）。理由は**木の幅上限 $w$ が固定なので話題ずれが on-topic を押し出す（crowd out）**——**枝刈りは節約でなく配分の機構**である。**(b) 分岐数 1（＝ CoT ＋枝刈り）を 40 回繰り返してクエリ予算で有利にしてもなお 84% → 48%**。**(c) 評価器を GPT-4 → GPT-3.5-Turbo で 84% → 4.2% に崩壊**（まだ破れていないのに破れたと誤判定して早期停止。平均クエリも 22.5 → 4.4）
- メモ（本 wiki にとって最大の収穫）: (b) が **Tree of Thoughts が chain-of-thought に勝つことの、本 wiki 唯一の統制された実測値**である。ToT 原典は未 ingest で `reasoning-and-planning` には主張しかなかった。しかも著者らが特定した理由が通常の ToT の説明（行き詰まったら戻れる）と違う——**独立に繰り返しても全部が同じ会話履歴から始まるので、第 1 反復のプロンプトがほぼ同一の戦略に収束してしまう**。**効いているのは「戻れること」でなく「途中から分岐できること」**。この観察は **self-consistency（CoT-SC）の多数決が効く条件**にも同じ形で効く
- メモ（判定器の使う場所）: (c) は [[summaries/2024-jailbreakbench]] が測った判定器の偽陽性が、**評価ではなく探索ループの内部**で効くとどうなるかの実例。**同じ欠陥が、評価に使えば ASR を水増しし、探索に置けば探索を殺す**。判定器の一致率を 1 つ測って「使える」と結論する前に、**どこに置くのかを決めてから測る**必要がある。皮肉なことに、**本論文の (c) こそ「判定器の質が結果を左右する」ことの最強の証拠なのに、その論理は自分たちの GPT-4 判定器には向けられていない**（PAIR から引き継いだ弱点。事後に JailbreakBench が一致率 90.3% と検証した）
- メモ（履歴に何を残すか）: **話題ずれは伝播する**——直前が off-topic だと次も off-topic になる確率が最大 50% 高い。だから TAP は**失敗したプロンプトそのものは履歴に残さず、攻撃者が書いた `improvement`（なぜ失敗したかの言語化）だけを残す**。[[self-reflection]] の「失敗を振り返らせると次が良くなる」と一見矛盾するが、**残しているものが違う**（反省か、失敗そのものか）。[[summaries/2023-pair]] の設計則「例示より自己反省が効く」と同じ線上にある
- メモ（Llama-2 の頑健性の正体）: TAP でも Llama-2-Chat-7B だけ 4%。著者らは手作りプロンプトで検証し、**「与えられた指示に従うことよりも拒否を優先する」**（「応答にネガティブな語を使うな」という指示を無視して拒否する）と報告。持ち帰りは「**より有能な LLM のほうが破りやすい**」。**この性質の代償は 1 年後に [[summaries/2024-jailbreakbench]] が測った——同じ Llama-2 7B は防御なしで良性挙動の 65% を拒否する。** 攻撃側から見た「頑健」と利用者から見た「使えない」が同じ性質の裏表であることの、最も明快な組み合わせ
- メモ（原典自体の誤り 4 件、クリップ不良ではない）: **(1) §4.2 の第 3 実験と Table 4 のキャプションが「attacker」と書いているが、変えているのは evaluator**（表の行ラベルは "as evaluator"）。**(2) §4.3「GPT-4 由来のほうが PaLM-2 へよく転移する（27% 対 27%）」——Table 5 の実際の値は 27% 対 24%** で本文が誤り（そのままでは主張が成立しない）。**(3) §4.3「jailbreaks found by PAIR and GCG transfer at a higher rate than the jailbreaks found by GCG」——「GCG が GCG より高い」という自己矛盾で、TAP and PAIR の誤り**。**(4) §1「keeping the details of an LLM secret does prevent attacks」——does not の脱字**。加えて **Figure 1 の帯ラベルが "Braching"**（正しくは Branching）。[[summaries/2025-llamafirewall]] の Figure 6 軸ラベル誤りと同じ扱いで、訳注に明示した
- メモ（測定日の記録）: 本論文は「評価はすべて **2023 年 11 月 18 日から 30 日までの 12 日間**に収集した」と明記している。GCG の転移率が PAIR 論文の報告値より落ちていることを「OpenAI チームによるモデルの継続的な更新」と推測しており、**ブラックボックス API を測る仕事では測定日がないと数値が再現も反証もできない**という作法の実例
- データギャップ: 変化なし。**GCG（Zou ら 2023）** と **Perez ら 2022** は依然未取得。**Tree of Thoughts（Yao ら 2023, arXiv:2305.10601）** は本 ingest で間接的な実測値が入ったので優先度は下がったが、[[reasoning-and-planning]] が依拠している原典であることに変わりはない

## [2026-08-03] ingest | Dive into Claude Code — 本番エージェント系のソースレベル解剖

- 取り込み: `raw/papers/Dive into Claude Code_ The Design Space of Today’s and Future AI Agent Systems.md`（ケース A。arXiv:2604.14228, 2026, VILA Lab / MBZUAI・University College London）
- 作成: [[summaries/2026-dive-into-claude-code]], [[translations/2026-dive-into-claude-code]]
- 更新: [[concepts/coding-agents]]（**2 節を新設**——「Claude Code という設計点」と「短期の増幅と長期の損耗」）, [[concepts/agent-loop]]（`queryLoop()` の解剖節を新設）, [[concepts/context-engineering]]（5 層 compaction の節を新設）, [[concepts/agent-memory]]（CLAUDE.md 階層と「指針と強制の分離」の節を新設）, [[concepts/agent-safety-and-guardrails]]（7 層の権限・承認疲れ・独立性の仮定・信頼前の窓の節を新設）, [[concepts/agent-frameworks]]（3 つの注入点と 4 機構のコスト階層の節を新設）, [[concepts/multi-agent-systems]]（subagent 委譲の本番実装を類型 (f) として追加）, [[concepts/model-context-protocol]]（本番実装と ACP の節を新設）, [[concepts/agent-observability]]（静かな失敗の節を新設）, [[overview]], [[index]]
- 画像: `raw/assets/2026-dive-into-claude-code/` に 8 枚（`x1`〜`x8`）
- 対話での決定: (1) **Claude Code のアーキテクチャ記述を関係する概念ページへ分散する**——スキーマ上、製品専用ページは作れない。coding-agents に設計点の俯瞰、agent-loop に queryLoop、context-engineering に 5 層 compaction、agent-safety-and-guardrails に 7 層の権限、agent-frameworks に 4 拡張機構、agent-memory に CLAUDE.md、multi-agent-systems に subagent、model-context-protocol に MCP 実装。**7 ページの予定が、agent-observability を足して 8 ページになった**。(2) **長期的な人間能力についての批判的証拠は coding-agents に新節を立てる**——本 wiki の coding-agents は ACI・ハーネス・検証・評価と「どう作るか」だけで構成されており、**使った結果人間側に何が起きるか**の記述が 1 つも無かった
- クリップの状態: **見出し 66・図表キャプション 17 件（Figure 1〜9・Table 1〜8）・markdown 表 8 個は完全**。欠落は 4 種類でいずれも原ページから復元した——**(1) §1 の番号付きリスト 3 項目が丸ごと欠落**（Design-space analysis / OpenClaw との対比 / 6 つの未解決方向。直前の「then organize the analysis in three parts:」が受ける先が消えていた。**TAP と同じ enumerate 脱落パターン**）、**(2) §2.3 の箇条書き 1 項目**（Safety, Security, and Privacy の行だけ。他の 4 項目は残存）、**(3) 脚注 4 件が全滅**（脚注 3 は **CVE 番号 4 件**、脚注 4 は**複雑度 +40.7%・速度スパイク +281%** という数値そのもので、内容として重要）、**(4) Figure 4 と Figure 8 に埋め込まれた設計原則の表 2 つ**（ar5iv には `<table>` として存在する）
- メモ（Figure 5 は不良ではない）: 図 5 だけ画像が無いが、**原典自体が擬似コード＋3 つの表で構成された図**である（ar5iv の画像も x1〜x8 の 8 枚のみで、Figure 1〜4・6〜9 に対応）。擬似コードと 3 表はテキストとして本文に収録した。**「画像が 1 枚足りない」だけで欠落と判断しないこと**の例
- メモ（本論文の性格）: **Anthropic の刊行物ではない。** 第三者が公開 npm パッケージから抽出した TypeScript ソース（v2.1.88、約 1,884 ファイル・51.2 万行）を読んで書いたもので、著者らは冒頭で原著作権が Anthropic に帰属することを明記している。**証拠を Tier A（公式ドキュメント）/ B（コード検証済み）/ C（再構成・推論）の 3 階層でラベル付けする**方法論を採っており、リバースエンジニアリング系としては珍しく誠実。**有名な「1.6% / 98.4%」は Tier C（コミュニティ解析からの推定）**であり、Tier B の主張と同じ強さでは読めない
- メモ（最大の収穫 1 — 一般則）: 「**あらゆるエージェントループは 3 つの注入点を持つ。`assemble()` はモデルが何を見るかを、`model()` は何に手を伸ばせるかを、`execute()` は行動が実行されるか／どう実行されるかを制御する**」。Claude Code の 4 拡張機構（フック 0・スキル低・プラグイン中・MCP 高）はこの 3 点に配置され、**コンテキストコストが挿入点と対応している**。自作エージェントの拡張点設計のチェックリストとしてそのまま使える
- メモ（最大の収穫 2 — 指針と強制の分離）: **CLAUDE.md はシステムプロンプトでなく user メッセージとして届く**ので「**これらの指示へのモデルの遵守は保証されたものではなく確率的である**」。決定論的な強制は権限ルールが担う。**守らせたい制約をプロンプトに書くのは設計上の誤り**であることを、届け方の構造として述べている
- メモ（最大の収穫 3 — 多層防御の独立性）: **層状の安全性は独立性の仮定に立つが、Claude Code の層は共通の性能・経済上の制約を共有している**（分類器は追加の LLM 呼び出し、AST チェックは解析遅延）。実例——**50 超のサブコマンドを持つコマンドは、解析が UI をフリーズさせたため deny チェックを飛ばして単一の汎用承認プロンプトへフォールバックする**。評価基準は「個々の層が迂回されうるか」でなく「**いくつの独立した層が同時に失敗しなければならないか、そしてそれらが失敗モードを共有しているか**」。本 wiki の 4 層構成そのものへの批判として読める
- メモ（最大の収穫 4 — 時間軸）: **権限のパイプライン図は安全性チェックの*空間的*な順序を描くが、*時間的*な次元を捉えていない。** プロジェクト初期化中に走るコード（フック、MCP サーバー接続、設定ファイル解決）は**信頼ダイアログの提示より前**に実行され、**deny-first の評価パイプラインの外側に落ちる**。これが **CVE-2025-59536（CVSS 8.7）・CVE-2026-21852（CVSS 5.3）・CVE-2025-54794・CVE-2025-54795**（全てパッチ済み）の根本原因。「**拡張性は組み合わせの複雑さだけでなく初期化の順序を通じても攻撃面を作る**」
- メモ（compaction の実装細部）: **層の順序に理由がある**——budget reduction が microcompact の前なのは、**microcompact が `tool_use_id` だけで動作し内容を見ないから**（きれいに合成する）。**計測器の罠**——snip の節約は主要トークンカウンタから見えない（カウンタは直近アシスタントメッセージの `usage` から導出し、そのメッセージは snip を生き延びて snip 前の値を保つ）ので `snipTokensFreed` を明示的に配管している。**間接的に導出された指標は、導出元が圧縮されると壊れる**。**context collapse だけは保存履歴を変更しない**（読み取り時の射影。ソースのコメント「何も yield されない。折り畳まれたビューは REPL の完全な履歴に対する読み取り時の射影である」）
- メモ（数字）: 承認疲れ **93%**（対応は警告増ではなく**決定の数を減らす**——サンドボックス化で権限プロンプト **−84%**。自動承認率は 50 セッション未満 20% → 750 セッション 40% 超で**習慣化により勾配を渡る**）／エージェントチームは標準セッションの**約 7 倍のトークン**／**最大 54 ツール**（19 無条件・35 条件つき。`CLAUDE_CODE_SIMPLE` では 3 つだけ）／**27 のフックイベント**（5 が権限フロー、15 がイベント固有の出力スキーマ）／`MAX_OUTPUT_TOKENS_RECOVERY_LIMIT = 3`／compaction のキャッシュ実験「false のパスは 98% キャッシュミスで、フリートの `cache_creation` の約 0.76% のコスト」
- メモ（本 wiki に完全に無かった軸）: **短期の増幅と長期の損耗。** **経験豊富な開発者 16 名・246 タスクの RCT で AI 使用時に 19% 遅くなったのに、本人たちは 20% 速くなったと知覚していた**（becker2025measuring）／Cursor 導入 807 リポジトリで**複雑度 +40.7%（p<0.001）**、初速 +281% は 3 か月で消え複雑度は残り将来の速度低下と比例（he2026cursor）／AI コミット **30.4 万件・6,275 リポジトリ**の監査で**AI 由来の問題の約 1/4 が最新版まで持続**、セキュリティ関連はさらに高率（liu2026techdebt）／理解度テスト **−17%**（shen2026skill）／EEG 54 名で神経結合の弱まりが AI 除去後も持続（kosmyna2025brain）／初級技術職の採用 **−25%**（rak2025aihiring）。**論文はこれを 5 価値と対等な設計価値ではなく「評価的レンズ」として扱う**——理由も明記されており、**長期的な人間の能力の保全はアーキテクチャにも Anthropic の表明された価値にも設計の駆動因として反映されていない**から
- メモ（測れないという問題）: 上の実証はすべてセッションから数か月の尺度で動作するが、**ハーネスは理解や規約のドリフトについてセッションごとの信号を一切露出していない**。著者らは (a) セッション粒度で測定可能にすること と (b) 測定が存在したときアーキテクチャがそれに応答できるか を別々の未解決問題として分け、**「ハーネスがそもそもその行動の正しい場所なのか（IDE、組織、人間の開発ループではなく）はアーキテクチャの分析では裁定できない」**と留保している
- メモ（OpenClaw と ACP）: 比較対象の OpenClaw は約 2 ダースのメッセージング表面を繋ぐ local-first の WebSocket ゲートウェイで、**正反対の賭け**をしている（信頼境界はゲートウェイの境界／エージェントループは制御平面の 1 コンポーネント／プラグインはゲートウェイ全体の能力面を拡張）。**そして 2 つは合成できる——OpenClaw は ACP（Agent Client Protocol）を通じて Claude Code・Codex CLI・Gemini CLI を外部ハーネスとしてホストする。** 「**エージェントの設計空間は平坦な分類ではなく層状のものであり、ゲートウェイ水準の系とタスク水準のハーネスが合成できる**」。なお **Agent Client Protocol と Agent Communication Protocol は略称 ACP が衝突している**ので index / model-context-protocol で区別を明記した
- データギャップ: **本論文が引用する外部の実証はどれも未取得**である。とくに **becker2025measuring（16 名の RCT）・he2026cursor（807 リポジトリの因果分析）・liu2026techdebt（30.4 万コミットの監査）** は、本 wiki が新設した「短期の増幅と長期の損耗」節の骨格が全面的に依拠しているのに二次情報でしか知らない。**hou2025mcpsurvey（MCP の脅威 4 カテゴリ 16 シナリオ）** も model-context-protocol の攻撃面の記述が依拠している。従来からの未取得（GCG / Perez ら 2022 / Tree of Thoughts）は変化なし

## [2026-08-03] ingest | AHE — ハーネスの自動進化、および concepts/harness-engineering の新設

- 取り込み: `raw/papers/Agentic Harness Engineering_ Observability-Driven Automatic Evolution of Coding-Agent Harnesses.md`（ケース A。arXiv:2604.25850, 2026, 復旦大学・北京大学・上海奇迹智锋）
- 作成: [[summaries/2026-agentic-harness-engineering]], [[translations/2026-agentic-harness-engineering]], **[[concepts/harness-engineering]]（新規）**
- 更新: [[concepts/coding-agents]]（**ハーネス系 3 節を新ページへ移設**し、要約＋導線の 1 節に置き換え。38KB → 35KB）, [[concepts/agent-frameworks]]（**ハーネス系 2 小節と長い段落を移設**し、3 点の導線に置き換え。24KB → 17KB）, [[concepts/agent-observability]]（「可観測性を設計原理に格上げする」節を新設）, [[concepts/agent-loop]], [[concepts/context-engineering]], [[concepts/agent-memory]], [[concepts/agent-evaluation]]（いずれも相互参照）, [[overview]], [[index]]
- 画像: `raw/assets/2026-agentic-harness-engineering/` に 9 枚（`x1`〜`x9`）
- 対話での決定: (1) **`harness-engineering` を新設し、既存 2 ページから移設する**——ユーザーからの提案を受けて検討し、賛成した。理由は 3 つで、**(a) ハーネスを主題とする原典が既に 7 本**（effective-harnesses / harness-design / meta-harness / managed-agents / dive-into-claude-code / swe-agent / 本論文）、**(b) `coding-agents` が 38KB で lint の分割検討域を超え、`agent-frameworks` の「フレームワーク観」も 5.7K 字がハーネス論だった**、**(c) ハーネスの議論はコーディングに固有でない**（Managed Agents のランタイム分離も OpenClaw もコーディング限定でない）。**スラグ名も `context-engineering` という先例があるので営みの名前で問題ない**と判断した。(2) **付録 B のプロンプト 4 本（約 45K 字）を全文収録する**
- **クリップの状態（新種の不良を 1 件発見）**: 見出し 66・図表キャプション 17 件・SVG テキスト箱 21 個・表 4 つはすべて残存。欠落は 3 種類——
  1. **画像 9 枚中 3 枚（`x5`・`x7`・`x9`）が欠落**。いずれも **2 パネル図の右パネル**（Figure 4 の regression パネル、Figure 11・12 の recall パネル）で、**本 wiki が何度も見てきた既知のパターン**。とくに `x5` には本論文の中核の数字（regression precision 11.8%・recall 11.1% とランダム基準 5.6%・5.4%）が入っていた
  2. **脚注 2 件**（タイトルページの equal contributions / 連絡先著者 / インターン先 / コード URL と、terminal-bench-2 リーダーボードの URL）
  3. **付録 B のプロンプトの空白が系統的に壊れていた——これは本 wiki で初めて見る形の不良である。** ar5iv は `lstlisting` の描画で**明示的な空白を `class="ltx_lst_space"` の span として符号化している**が、**Web Clipper が `class` 属性を落としたためその空白 span が消え、代わりに隣接する span の間へ一律で空白が挿入された**。結果、`non-interactive` → `non - interactive`、`` `run_shell_command` `` → `` ` run_shell_command ` ``、`setting. Your` → `setting.Your` に化けていた。**プロンプトは一字一句が挙動に効くので、付録 B の 4 本はすべて ar5iv から `ltx_lst_space` を手掛かりに空白を復元し直して収録した**（`lstnumberxN` の id 接頭辞から行境界も再構成）
- メモ（クリップ不良の教訓）: 3 番目は **「本文は全部あるのに、意味が変わっている」**という種類の不良である。これまでの不良（画像欠落・脚注欠落・リスト脱落）はすべて**何かが無くなる**形だったが、今回は**残っているものが静かに壊れている**。**プロンプトやコードのように空白が意味を持つ内容を扱うときは、「あるか無いか」ではなく「同一か」を確認する必要がある**
- メモ（本論文の中核）: 主張は「**ボトルネックはエージェントの能力ではなく可観測性である**」。**3 本の柱**——**❶ コンポーネント可観測性**（ハーネスを 7 種の直交コンポーネントに分解して**すべてファイルとして固定マウント点へ**置く。編集＝git の 1 コミット、ファイル粒度でロールバック。**種は意図的に最小**——「ベンチマークに適合させた種を使うと以後のあらゆる編集の帰属が汚染される」）、**❷ 経験可観測性**（数百万トークンの生トレースを**「1 メッセージ 1 ファイル」のナビゲート可能な環境**として探索させ、タスク別レポート → ベンチマーク概観へ蒸留。〜10M → 〜10K トークン）、**❸ 決定可観測性**（**すべての編集に「直すタスク」と「壊すタスク」を自己申告させ、次ラウンドの実測差分と突き合わせ、外れたらファイル粒度で巻き戻す**）
- メモ（最大の収穫 — 層別の価値）: **本 wiki でハーネスの層別の価値を定量化した唯一の資料。** 種に 1 つだけ差し替えると **長期記憶 +5.6 / ツール +3.3 / ミドルウェア +2.2 pp に対し、システムプロンプトは −2.3 pp（唯一の後退）**。「**事実としてのハーネス構造は転移するが、散文レベルの戦略は転移しない**」。**これは [[summaries/2026-dive-into-claude-code]] の「CLAUDE.md は user メッセージとして届くので遵守は確率的」という構造的な観察を、性能の数字で裏づけたもの**として読める。プロンプトのみの自己進化（ACE・TF-GRPO）が SWE-bench-verified で**手つかずの種を下回りながら 11〜29% 多くトークンを使う**のも同じ理由
- メモ（非加法性）: **3 つの正の単独利得の和 +11.1 pp に対し全部入りは +7.3 pp。Hard では記憶のみが全部入りを上回る。** 似た方向の是正（closure 流の検証）が重なると長期ホライズンの予算を冗長な再チェックに食う。加えて**最適化の目的関数の構成比が収束先の性格を決める**（55 件の Medium が支配する集計を最適化すると Medium 寄りに落ちて Hard を返上する）
- メモ（自己改善ループの限界）: **fix の予測はランダムの約 5 倍（精度 33.7%・再現率 51.4%）だが、regression の予測はわずか約 2 倍（11.8% / 11.1%）。9 ラウンドで 43 件予測して的中 5 件、予見できなかった後退が 40 件。** 「エージェントはある編集がなぜ役立つはずかを正当化できるが、その同じ編集が壊そうとしているタスクを名指しすることはできない」。**[[self-reflection]] が扱う前向きの反省とは別に、後ろ向きの「何を壊すか」は働いていない**
- メモ（独立に収束した 7 コンポーネント）: 新ページの冒頭に置いた観察。**NexAU（本論文）と Claude Code v2.1.88（[[summaries/2026-dive-into-claude-code]]）が、系譜的な繋がりがないのにほぼ同じ 7 コンポーネント分類へ着地している**（システムプロンプト／ツール記述／ツール実装／ミドルウェア＝フック／スキル／サブエージェント設定／長期記憶）。**ハーネスという設計対象がある程度まで自然な関節を持っていることの、弱いが有用な証拠**だと思う
- メモ（批判点）: 10 反復のキャンペーンは **1 回しか走っていない**（best-so-far 報告なので実行間分散が見えない。進化曲線が非単調であること自体が分散の大きさを示唆する）／**Hard では人手設計の Codex-CLI に負けている**（53.3% 対 56.7%）／**進化そのもののコストが時間しか報告されていない**（10 反復 × 89 タスク × 2 ロールアウト、約 32 時間。加えて Agent Debugger と Evolve Agent の呼び出し）／**3 つの役割エージェントが同じモデルを共有**するので Evolve Agent が Code Agent と同じ盲点を持つ可能性が検討されていない／著者ら自身が「**完全に成熟した自律的な自己改善システムではなく、制御された研究のプロトタイプ**」と明記
- 概念ページの再構成: `coding-agents` から「長時間タスクのハーネス」「generator / evaluator / planner」「コーディングエージェントがハーネスを書く」の 3 節を、`agent-frameworks` から「ハーネスの自動探索」「ランタイムの分離」の 2 小節とフレームワーク観のハーネス段落を移設。**両ページには要約＋導線を残した**（coding-agents はコーディング固有の話——ACI・検証の設計・Claude Code という設計点・短期の増幅と長期の損耗——に、agent-frameworks は製品としてのフレームワークと 3 つの注入点に集中する）
- データギャップ: 本論文が引く自動最適化の系譜——**ACE（zhangAgenticContextEngineering2025a）・TF-GRPO（caiTrainingFreeGroupRelative2025）・GEPA（agrawalGEPAReflectivePrompt2025a）・DSPy（khattabDSPyCompilingDeclarative2023）**——はいずれも未取得で、本 wiki はそれらを比較対象としてしか知らない。**Agent Debugger（linAgentDebuggerUnderstanding2026）** も、本論文の柱 ❷ の実装を担うのに二次情報のみ。従来からの未取得（GCG / Perez ら 2022 / Tree of Thoughts / becker2025measuring 等の人間側の実証）は変化なし
