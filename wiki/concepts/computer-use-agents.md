---
type: concept
aliases: [CUA, computer use, GUI エージェント, GUI agents, コンピュータ操作エージェント]
tags: [computer-use-agents, llm-agents, multimodal]
related:
  - "[[tool-use-and-function-calling]]"
  - "[[agent-loop]]"
  - "[[agent-evaluation]]"
  - "[[agent-safety-and-guardrails]]"
  - "[[multi-agent-systems]]"
summaries:
  - "[[summaries/2026-kimi-k2.5]]"
updated: 2026-07-28
---

# Computer-Use Agents（コンピュータ操作エージェント）

**画面のスクリーンショットを観測し、マウス・キーボード操作を出力して、人間向けの GUI（Graphical User Interface）をそのまま操作する**エージェントの総称。CUA（Computer-Use Agent）とも呼ばれる。[[tool-use-and-function-calling]] が「ソフトウェアの API 側の入り口」を使うのに対し、computer use は **API が存在しない・使えないソフトウェアも、人間と同じ入り口（画面と入力デバイス）から操作できる**のが本質的な価値である。代償として、観測は構造化データでなくピクセル、行動は関数呼び出しでなく座標クリックになり、視覚認識・GUI の状態変化への追従・長い操作列の一貫性という固有の難しさを抱える。

## エージェントループの形

CUA は [[agent-loop]] の一種で、典型的には次を反復する:

1. **観測**: 現在の画面のスクリーンショット（＋タスク指示、直近の操作履歴）
2. **思考**: 画面の状態を読み、次の一手を決める
3. **行動**: 低レベル操作（クリック座標・キー入力・スクロール・待機）を構造化した形式で出力し、実行環境が再生する

行動空間には 2 つの流儀がある。**低レベル操作型**は pyautogui のような座標ベースの汎用操作で任意のアプリを扱う。**専用ツール型**はブラウザ操作など対象を限定した高レベルツール（「リンクをクリック」「フォームに入力」）を使う——確実だが対象外のソフトは扱えない。評価では両者を区別しないと比較にならない（後述の K2.5 の例では、Claude を computer use ツールのみに制限して条件を揃えている）。

## 代表例 — Kimi K2.5・Operator・Claude

現時点の wiki 内の主要な根拠原典は Kimi K2.5（[[summaries/2026-kimi-k2.5]], 2026）で、**汎用マルチモーダルモデルが専用フレームワークなしで CUA として機能する**水準を示した:

- **構成**（OpenCUA 構成に準拠）: コンテキストは「直近 3 枚の履歴スクリーンショット＋完全な思考履歴＋タスク指示」。システムプロンプトは各ステップに `{thought}` → `## Action:` → `## Code:` の形式を要求し、Code 部は **pyautogui コードまたは `computer.wait` / `computer.terminate`** の 2 関数（付録 E.7 に全文）。エピソードあたり最大 100 ステップ。
- **結果**: OSWorld-Verified **63.3%**（GUI 操作のみ）——オープンソースの Qwen3-VL-235B-A22B（38.1%）、OpenAI の CUA フレームワーク **Operator**（o3 ベース, 42.9%）を大きく上回り、主導的 CUA である **Claude Opus 4.5**（66.3%）に肉薄。WebArena **58.9%** は Operator（58.1%）超え。
- **能力の出どころ**: 事前学習段階でデスクトップ・モバイル・Web の **GUI スクリーンショットと操作 trajectory**（人手デモ含む）を混ぜ、OS スクリーンショットを joint 事前学習のデータレシピに含めている。GUI 操作は「読む」（アイコン・ボタンの認識＝視覚知覚）と「当てる」（正確な座標出力＝グラウンディング）の複合であり、K2.5 が視覚 RL で鍛えた位置特定・計数・OCR（→ [[reinforcement-learning-from-human-feedback]]）はこの基礎体力にあたる。

Anthropic の Claude computer use（2024〜）と OpenAI の Operator は、この領域をプロダクトとして先行させた代表例である（それぞれの技術詳細の原典は未 ingest。上記スコアは K2.5 レポートの比較評価による）。

## 評価 — OSWorld と WebArena

- **OSWorld（-Verified）**: 実 OS（Linux デスクトップ）上の実アプリ（LibreOffice, GIMP, ブラウザ等）を操作する数百タスクのベンチマーク。成功判定はタスクごとの検証スクリプトによる**終了状態評価**（→ [[agent-evaluation]] の end-state evaluation と同じ原理）。人間は 70%+ とされ、63〜66% のフロンティアモデルはこれに接近しつつある。
- **WebArena**: 自己ホスト型の実 Web アプリ群（EC サイト・フォーラム・GitLab 等）での GUI ブラウジングタスク。K2.5 の評価では判定スクリプトの誤り修正や fuzzy_match のジャッジモデル指定など、**ハーネスの手直しが結果に効く**ことも記録されており、スコアは測定条件とセットで読む必要がある。

## 設計論点

- **視覚グラウンディングが律速する**: 「どのボタンを押すべきか分かる」と「そのボタンの座標を正しく出せる」は別の能力で、後者の失敗（僅かなズレ・誤クリック）は GUI では即座に状態を壊す。視覚 RL の位置特定・計数タスク（IoU/距離ベース報酬）はこの能力への直接投資である。
- **長い horizon とエラー回復**: 1 タスク最大 100 ステップ級の操作列では、途中の 1 誤操作が後続をすべて狂わせる。ダイアログの予期しない出現・読み込み待ち（`computer.wait` の存在理由）・画面解像度やテーマの差など、環境の非決定性への頑健さが実タスク成功率を分ける。
- **安全性の露出面が最大級**: CUA は「画面に映るものすべて」が入力になるため、Web ページに仕込まれた指示による prompt injection（外部入力に埋め込まれた指示でエージェントを乗っ取る攻撃）の攻撃面が広い。しかも行動空間は実マシンの任意操作であり、誤動作・乗っ取りの被害半径が大きい。サンドボックス化（隔離 VM での実行）・不可逆操作（購入・送信・削除）前の人間の確認（HITL, Human-in-the-Loop）が実運用の必須装備 → [[agent-safety-and-guardrails]]。K2.5 のプロンプトが `The password of the computer is {password}.` と資格情報を平文で渡している点は、この領域の権限設計がまだ素朴であることの例でもある。
- **API があるなら API を使う**: computer use は万能の入り口だが、ピクセル経由の操作は API 呼び出しより遅く・脆く・高い（毎ステップ画像を積むためトークンも重い）。[[tool-use-and-function-calling]] が使える対象には API を使い、GUI しかない残余に CUA を充てるのが経済的な使い分けである。
- **マルチエージェントとの合流**: 動画分割解析のような大規模視覚タスクでは、CUA 的な視覚エージェントを [[multi-agent-systems]] の並列オーケストレーション（K2.5 の Agent Swarm）で束ねる構成が現れている——「1 画面を操作する」から「多数の視覚ワークロードを分業する」への拡張線。

## 関連ページ

- [[tool-use-and-function-calling]] — API 側の入り口。CUA はその補集合（GUI しかない対象）を埋める
- [[agent-loop]] — スクリーンショット観測→思考→操作のループ構造
- [[agent-evaluation]] — OSWorld / WebArena と終了状態評価
- [[agent-safety-and-guardrails]] — prompt injection・サンドボックス・HITL
- [[multi-agent-systems]] — 視覚エージェントの並列オーケストレーション
- [[summaries/2026-kimi-k2.5]] — 本ページの根拠原典（初出典）
