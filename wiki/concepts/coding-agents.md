---
type: concept
aliases: [コーディングエージェント, coding agent, autonomous coding, SWE-agent, Claude Code, ACI, agent-computer interface]
tags: [coding-agents, llm-agents]
related:
  - "[[agent-loop]]"
  - "[[tool-use-and-function-calling]]"
  - "[[agent-evaluation]]"
  - "[[context-engineering]]"
  - "[[agent-frameworks]]"
  - "[[computer-use-agents]]"
summaries:
  - "[[summaries/2025-effective-harnesses]]"
  - "[[summaries/2026-harness-design]]"
updated: 2026-07-31
---

# Coding Agents（コーディングエージェント）

**コードを書き・実行し・その結果を観測して修正する**ループを回すエージェントの総称。[[agent-loop]] の応用形のうち最も成熟した領域で、環境は「リポジトリ＋実行環境（シェル・テスト・開発サーバー）」、行動は「ファイル編集・コマンド実行」、観測は「コンパイル結果・テスト出力・実行ログ」である。コーディングが応用の先頭を走る理由は、**環境からのフィードバックが速く・機械可読で・ほぼ無料**なこと——書いたものが動くかを即座に検証できるため、[[reinforcement-learning-from-human-feedback]] の検証可能報酬とも、実行時の自己修正ループとも相性がよい。代表製品・システムに Claude Code・SWE-agent・Devin・Cursor など（各詳細は概説。専用原典は未 ingest）。

## 構成要素

- **ツール（ACI）**: エージェントが使うツールは人間向け CLI の流用では最適でない。ツール群をエージェントの認知に合わせて設計し直す考え方が **ACI**（agent-computer interface）で、SWE-agent（2024, 概説）が確立した。興味深いことに、フロンティアモデルの評価ハーネスは最小構成に収斂している——[[summaries/2026-deepseek-v4]] の SWE 評価は「bash＋ファイル編集」の 2 ツール、[[summaries/2026-kimi-k2.5]] は bash / create_file / insert / view / str_replace / submit の 6 ツールで、豊富なツールより**少数の直交したツール**が好まれる（→ [[tool-use-and-function-calling]] の「ツールの弱さが推論を引き出す」の現代形）。
- **実行フィードバック**: 単体テスト・型検査・リンタ・開発サーバー・ブラウザ自動化。どの層まで検証するかが品質を決める（後述）。
- **状態管理**: **git が天然のチェックポイント機構**を提供する——コミットは進捗記録・ロールバック手段・次セッションへの引き継ぎ記録を兼ねる。エージェント設計で外部メモリをゼロから作る前に、リポジトリに既にある仕組みを使うのが定跡。

## 長時間タスクのハーネス — initializer / coding の二部構成

単発のバグ修正を超えて**数時間〜数日のプロジェクト**（アプリをまるごと構築する等）を任せると、セッション（コンテキストウィンドウ）の境界が支配的な問題になる。本ページの主要な根拠原典である Anthropic の実務記事（[[summaries/2025-effective-harnesses]], 2025）は、claude.ai クローン構築（200 超の機能）でこの問題を解いたハーネスを開示した:

- **失敗の 2 形態**: (1) 一発完成を狙って文脈切れし「半実装・記録なし」を残す、(2) 途中参加のセッションが進捗を見て**早期勝利宣言**する。compaction（要約圧縮）では防げない——要約は次セッションへの指示として不完全だから。
- **initializer agent**（初回のみ・実体は同一ハーネスでプロンプトだけ別）: 環境の足場を作る——**feature list JSON**（全機能を `"passes": false` で列挙し「完成の定義」を外部化。JSON は Markdown より改変されにくく、「テスト削除は容認不可」の強い文言で守る）・進捗ログファイル・`init.sh`（起動手順の固定）・初期 git commit。
- **coding agent**（毎セッション）: **状況把握**（pwd → 進捗ファイル・git log → 未完了の最優先機能を 1 つ選ぶ）→ **基本 E2E テスト**（壊れていたら新機能より先に修復）→ **1 機能だけ**実装・検証 → **クリーンに終了**（「main にマージできる状態」で commit＋進捗更新）。
- 位置づけ: [[summaries/2025-multi-agent-research-system]] が**空間方向の分業**（並列サブエージェント）なら、これは**時間方向の分業**（直列セッションの引き継ぎ）である。引き継ぎ媒体を「要約」でなく**検査可能な構造化 artifact**（git 履歴・JSON・ログ）にした点が [[summaries/2023-memgpt]] の recursive summary からの進化 → [[context-engineering]]。

## generator / evaluator / planner — 分業とハーネスの縮小

initializer/coding ハーネスの続編（[[summaries/2026-harness-design]], 2026）は、この構成を **3 エージェント**へ発展させ、そして**縮小する方法論**まで開示した:

- **planner**: 1〜4 文のプロンプトを完全な仕様へ拡張（一文 → 16 機能・10 スプリント）。**成果物レベルで縛り技術詳細は指定しない**——細かい技術指定の誤りは下流へ連鎖するため。外すと generator は仕様化を省いて under-scope する。
- **generator / evaluator の分離**: 自己評価は構造的に甘い（自分の仕事を自信満々に承認する）が、**独立した evaluator を懐疑的にチューニングする方が、generator を自己批判的にするよりはるかに扱いやすい**。evaluator は Playwright で実操作 QA を行い、基準ごとの硬い閾値で不合格判定＋具体的フィードバックを返す。着手前には **sprint contract**——「このチャンクの完了とは何か」への事前合意——を generator と交渉する。
- **実測の対比**（Opus 4.5, レトロゲームメーカー）: solo 20 分/$9 は見た目それらしいが**ゲームが動かない**。フルハーネス 6 時間/$200 は動く。v2（Opus 4.6, DAW）は 3h50m/$124.70 で、QA は合計 $10 程度のコストで「表示だけの機能」（録音がスタブ・クリップ移動不可）級の欠陥を捕捉し続けた。
- **ハーネスの縮小**: 「**各部品は『モデルが単独でできないこと』への仮定であり、モデルの改善で陳腐化する**」。一気に削ると何が load-bearing か分からなくなるため、**1 部品ずつ外して影響を確認**する。実際に Opus 4.5→4.6 で context reset（context anxiety の消滅）とスプリント分解（2 時間超を分解なしで一貫走行）が不要になった。**evaluator は固定の yes/no でなく、タスクがモデルの単独信頼境界の外にあるときだけコストに見合う**——モデルが強くなるたびに境界は動く。

## 検証の設計 — 「コードだけ見て完了宣言」問題

コーディングエージェントの品質は**どこまで検証させるか**で決まる。実務で繰り返し観測される失敗が「単体テストや curl は通したが、end-to-end では動かないのに完了とマークする」であり、対策は**人間のユーザーと同じ経路での検証**——Web アプリならブラウザ自動化（Puppeteer 等。ブラウザ操作という意味で [[computer-use-agents]] の技術の検証用途への転用）で実際に操作させる。ただし 2 つの罠が残る:

1. **検証ツールの盲点は品質の穴になる**: Puppeteer MCP から見えない alert モーダルに依存する機能はバグが残った（[[summaries/2025-effective-harnesses]]）——エージェントが観測できない場所は検証されない。
2. **自己検証の偽陽性は致命傷**: [[summaries/2023-reflexion]] の実測どおり、誤った実装を通してしまうテスト（偽陽性）は「確信を持った誤り」を生む。feature list の self-verify（自分で passing にする）も同じ構造を持ち、検証手段が弱いドメインでは機能しにくい。評価側の対応物は [[agent-evaluation]] の終了状態評価。

## 評価

標準は **SWE-bench 系**（実 GitHub イシューの解決率。Verified / Pro / Multilingual）・Terminal Bench（端末操作）・LiveCodeBench（競技・汚染対策つき）など → 詳細は [[agent-evaluation]]。実務上の注意として、同じモデルでもハーネスでスコアが大きく動く（ハーネス差・step 上限・コンテキスト管理の開示が比較の前提）ことと、フロンティアの到達点（2026 年時点で SWE-bench Verified 80 前後 → [[summaries/2026-deepseek-v4]] 表6）に対し、**長時間の自律構築タスクには標準ベンチマークがまだ無い**（Anthropic の記事も定量評価なし）ことが挙げられる。

## 設計論点

- **単一汎用 vs 特化分業**: テスト専任・QA・クリーンアップ等の特化エージェント分業が単一 coding agent に勝るかは**未解決**（[[summaries/2025-effective-harnesses]] が明示的にオープンクエスチョンとした）。[[multi-agent-systems]] の一般論（依存の密なコーディングは並列分業に不向き——Anthropic Research の自己申告）と、Agent Swarm 系（並列化を学習で解く）の間で、コーディング領域の答えはまだ出ていない。
- **エージェントの書きやすさに環境を寄せる**: init.sh・機能リスト・進捗ログは「エージェントのための開発者体験（DX）」であり、人間の新人オンボーディングと同じ投資が効く。モデル訓練側でも同じ力学が働く——[[summaries/2025-kimi-k2]] のツール利用データ合成や [[summaries/2026-deepseek-v4]] の R&D 実タスク評価は、エージェントが働く環境・課題の整備がモデル性能と同じくらい結果を左右することを示す。
- **セッションの型化**: 「状況把握 → 健全性確認 → 1 単位 → クリーン終了」はコーディング以外の長時間タスク（研究・分析）にも流用可能な汎用テンプレート。
- **安全性**: コード実行はサンドボックス内で（→ [[agent-safety-and-guardrails]] の DSec が本番規模の実例）。リポジトリ書き込み・デプロイ権限は最小化し、不可逆操作（force push・本番反映）は人間の承認を挟む。

## 関連ページ

- [[agent-loop]] — 基礎となる観測→思考→行動ループ
- [[tool-use-and-function-calling]] — ツール設計（ACI・最小ツール構成）
- [[agent-evaluation]] — SWE-bench 系・終了状態評価
- [[context-engineering]] — セッション間の状態外部化
- [[agent-frameworks]] — ハーネスという設計層
- [[computer-use-agents]] — E2E 検証手段としてのブラウザ操作
- [[summaries/2025-effective-harnesses]] — 本ページの主要な根拠原典（長時間ハーネス）
- [[summaries/2026-harness-design]] — 続編（3 エージェント構成・ハーネス縮小の方法論）
