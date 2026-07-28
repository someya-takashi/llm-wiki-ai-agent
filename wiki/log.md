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
