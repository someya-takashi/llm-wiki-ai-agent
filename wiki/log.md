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
