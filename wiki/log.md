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
