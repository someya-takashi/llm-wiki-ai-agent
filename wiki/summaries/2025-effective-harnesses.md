---
type: summary
source_path: "raw/articles/Effective harnesses for long-running agents.md"
source_kind: blog
title: "Effective harnesses for long-running agents"
authors: [Justin Young]
year: 2025
venue: "Anthropic Engineering Blog"
ingested: 2026-07-31
tags: [coding-agents, context-engineering, agent-frameworks, agent-loop, long-running-agents, harness, claude-agent-sdk]
translation: "[[translations/2025-effective-harnesses]]"
---

# Effective harnesses for long-running agents（Anthropic, 2025）

> 原典: [[translations/2025-effective-harnesses]] ・ `raw/articles/Effective harnesses for long-running agents.md`
> 著者・年・出典: Justin Young（Anthropic）・2025・Anthropic Engineering Blog

## 一言まとめ

**数時間〜数日かかるタスクを、記憶を持たないセッションの連鎖でやり遂げるためのハーネス（エージェントを動かす外側の実行基盤）設計**を、claude.ai クローン構築（200 超の機能）を題材に開示した実務記事。解は「**initializer agent が初回に環境の足場（機能リスト JSON・進捗ファイル・init.sh・git）を作り、coding agent が毎セッション『状況把握 → 基本テスト → 1 機能だけ → クリーンに終了』を繰り返す**」という二部構成。[[summaries/2024-building-effective-agents]]（設計原則）→ [[summaries/2025-multi-agent-research-system]]（並列の分業）に続く Anthropic 実務系原典の第 3 弾で、こちらは**時間方向の分業（セッション間の引き継ぎ）**を扱う。[[coding-agents]] の初出典。

## 背景と問題意識

エージェントの 1 セッションはコンテキストウィンドウ（モデルが一度に読める最大トークン数）に縛られ、複雑なプロジェクトは 1 ウィンドウでは終わらない。記事の比喩が的確で、これは「**引き継ぎ記録が一切ないシフト勤務のエンジニアでプロジェクトを回す**」問題である。compaction（履歴を要約して文脈を空ける圧縮機構）があれば理論上は無限に働けるはずだが、実際には不十分——**要約は次のエージェントへの指示として不完全**だからだ。Opus 4.5 クラスでも、素のループ＋高レベルプロンプト（「claude.ai のクローンを作って」）では本番品質に届かず、失敗は 2 つの形をとる:

1. **一発完成を狙って自滅**: 一度に作りすぎ、実装の途中で文脈が尽き、「半実装・記録なし」の状態を次セッションに残す。次のエージェントは状況の推測と復旧に時間を溶かす。
2. **早期勝利宣言**: プロジェクト後半、途中参加のインスタンスが「進捗がある」ことを見て、残タスクがあるのに完了を宣言する。

## 提案手法 / 主張

### 二部構成: initializer と coding — ただし「同一ハーネス、プロンプトだけ違う」

復元した脚注が重要な設計情報を含む: initializer と coding は**別エージェントではなく、初期ユーザープロンプトが違うだけで、システムプロンプト・ツール・ハーネスは同一**である。「最初のウィンドウにだけ別プロンプト」という最小の介入で解いている点が、[[summaries/2024-building-effective-agents]] の simplicity 原則の実践になっている。

**Initializer agent（初回のみ）**が作る環境の足場は 4 点:

1. **feature list JSON** — ユーザーの一文プロンプトを **200 超の end-to-end 機能記述**に展開し、全件 `"passes": false` で列挙（各機能に検証手順 steps つき）。「完成とは何か」の輪郭を先に固定することで、一発完成も早期勝利宣言も構造的に防ぐ。
2. **claude-progress.txt** — 歴代エージェントの作業ログ。
3. **init.sh** — 開発サーバーを立ち上げるスクリプト（毎セッションの「アプリの動かし方の再発見」を排除）。
4. **初期 git commit** — 以後、git 履歴そのものが引き継ぎ記録・チェックポイント・ロールバック手段を兼ねる。

**Coding agent（毎セッション）**の型:

- **冒頭の状況把握（bearings）**: `pwd` → progress ファイルと git log を読む → 機能リストから未完了の最優先 1 件を選ぶ。
- **着手前の基本 E2E テスト**: サーバーを起動し、チャット送受信などの基幹動線を確認。壊れていたら**新機能より先に修復**（壊れた土台の上に積むと悪化する）。
- **1 機能だけ**実装・検証。
- **クリーンに終了**: 説明的なメッセージで git commit＋progress 更新。「クリーン」の定義は「**main ブランチにマージできる状態**」——バグなし・整理済み・文書化済みで、次の開発者（＝次のセッションの自分）が片付けなしに着手できること。

### 実務知見の粒

- **JSON は Markdown より改変されにくい**: 機能リストを Markdown で持つとモデルが不適切に書き換え・上書きしやすい。フォーマット選択が「モデルに対する権限制御」として機能するという発見。あわせて「テストの削除・編集は容認不可」という**強い文言**で passes フィールド以外の編集を禁じる。
- **E2E テストを明示しないと「動くつもり」で完了マークする**: 単体テストや `curl` では通るのに end-to-end では動かない、を認識できない。**Puppeteer MCP（ブラウザ自動化ツールを MCP 経由で提供）で人間のユーザーと同じ操作をさせる**と劇的に改善——コードからは見えないバグを見つけて直せるようになる（→ ブラウザ操作という点で [[computer-use-agents]] の検証用途での利用例）。
- **検証ツールの盲点はそのままバグの巣になる**: Puppeteer MCP からはブラウザネイティブの alert モーダルが見えず、それに依存する機能はバグが残りがち——「エージェントが検証できない場所に品質は担保されない」という一般則の具体例。

<figure>

![](../../raw/assets/2025-effective-harnesses/puppeteer-testing.gif)

<figcaption>（再掲）Puppeteer MCP を通じて claude.ai クローンを E2E テストする Claude のスクリーンショット。</figcaption>
</figure>

### 失敗モードと対処の対応表

| 失敗モード | initializer 側の仕込み | coding 側の型 |
| --- | --- | --- |
| 早期の全体勝利宣言 | 機能リスト JSON | 冒頭で読み、1 機能を選ぶ |
| バグ・未記録のまま終了 | 初期 git＋進捗ファイル | 冒頭で読み基本テスト、末尾で commit＋更新 |
| 未検証の完了マーク | 機能リスト（steps つき） | 注意深いテスト後にのみ passing 化 |
| 起動方法の再発見に浪費 | init.sh | 冒頭で init.sh を読む |

## 実験結果と知見

定量ベンチマークはなく、**失敗モードの定性的観察と対処の有効性**という形の報告である（「劇的に改善」「決定的に重要」といった記述レベル）。素材としての価値は、典型セッションの実トレース（bearings → git log → サーバー起動 → 基幹確認 → 機能着手）と feature JSON の実例が逐語で公開されている点にある。コード例は付属の quickstart（anthropics/claude-quickstarts/autonomous-coding）で再現可能。

## 限界・批判的視点

- **定量評価の欠如**: 対照実験（ハーネスあり/なしの成功率・所要セッション数・トークンコスト）は示されない。「効いた」の根拠は内部実験の観察に留まる。
- **Web アプリ特化**: E2E 検証の設計（ブラウザ自動化）はフルスタック Web 開発に最適化されており、著者も科学研究・金融等への一般化は今後の課題と明言。検証手段が弱いドメイン（実行結果を機械確認できないタスク）では feature list の「self-verify で passing 化」が機能しにくい——[[summaries/2023-reflexion]] が示した**自己検証の偽陽性（誤った解をテストが通すと確信して即提出）**と同じ罠が、feature list の自己採点にもそのまま潜む。
- **単一 vs マルチエージェントは未解決**: テスト・QA・クリーンアップの特化エージェント分業が勝る可能性を自認しつつ未検証。[[summaries/2026-kimi-k2.5]] の Agent Swarm（並列分業を RL で学習）とは、同じ問いへの「ハーネス設計で解く/訓練で解く」の対照になっている。
- **Claude Agent SDK 前提**: compaction・MCP 等の機能を持つハーネスが前提で、素の API ループにそのまま移植できるとは限らない。

## 実装・運用上の示唆

- **「継ぐ」のは要約ではなく構造化 artifact**: [[summaries/2023-memgpt]] の recursive summary（要約で継ぐ）に対し、本記事は **progress ファイル＋git 履歴＋機能リスト**という検査可能な構造物で継ぐ。要約は劣化するが、git log と JSON は劣化しない——長時間タスクの引き継ぎ設計の現在の実務解。
- **完成の定義を先に外部化する**: 「何ができたら終わりか」をエージェント自身の判断に残すと早期勝利宣言が起きる。feature list はこれを初回に固定し、以後は読み取り専用（passes のみ可変）にする——[[agent-evaluation]] の終了状態評価を、評価者でなく**エージェント自身の作業管理**に埋め込んだ形。
- **セッションの型を決める**: 「状況把握 → 健全性確認 → 1 単位の作業 → クリーンに終了」は、コーディングに限らず長時間タスクの汎用テンプレートとして流用できる。
- **フォーマットも権限設計**: 改変されたくない状態は JSON に、強い禁止文言とセットで置く。

## 用語と略称

- **ハーネス（harness）**（エージェントを動かす外側の実行基盤——ループ・ツール・コンテキスト管理・プロンプトの総体）
- **長時間走行エージェント（long-running agent）**（複数のコンテキストウィンドウ＝セッションをまたいで単一のプロジェクトを進めるエージェント）
- **コンテキストウィンドウ**（モデルが一度に読める最大トークン数）／ **compaction**（履歴を要約して文脈を空ける圧縮機構 → [[context-engineering]]）
- **initializer agent / coding agent**（初回の環境構築役／毎セッションの漸進役。実体は同一ハーネスでプロンプトのみ異なる）
- **feature list**（全機能を "passes": false で列挙した JSON。完成の定義の外部化）／ **claude-progress.txt**（セッション間の作業ログ）／ **init.sh**（開発サーバー起動スクリプト）
- **bearings（状況把握）**（セッション冒頭の pwd・ログ読み・機能選択の手順）／ **クリーンな状態**（main にマージできる品質でセッションを終えること）
- **E2E（end-to-end）テスト**（ユーザー操作の動線全体を通す検証）／ **Puppeteer MCP**（ブラウザ自動化ツール Puppeteer を MCP 経由でエージェントに提供するサーバー）
- **MCP** = Model Context Protocol（ツールやデータソースをモデルに接続する標準プロトコル）／ **Claude Agent SDK**（Anthropic のエージェント構築キット → [[agent-frameworks]]）
- **LLM** = Large Language Model（大規模言語モデル）

## 関連ページ

- [[coding-agents]] — 本記事を初出典として新設した概念ページ
- [[context-engineering]] — compaction の限界とセッション間の状態外部化
- [[agent-frameworks]] — Anthropic 実務 3 部作の 3 本目（harness の語彙）
- [[agent-loop]] — セッション内ループの外側の「セッション間ループ」
- [[agent-evaluation]] — 完成定義の外部化＝終了状態評価の作業管理への埋め込み
- [[computer-use-agents]] — E2E 検証手段としてのブラウザ操作
- [[summaries/2024-building-effective-agents]] / [[summaries/2025-multi-agent-research-system]] — 同じ Anthropic 実務系譜（設計原則→並列分業→時間方向の引き継ぎ）
- [[summaries/2023-memgpt]] — 「要約で継ぐ」から「構造化 artifact で継ぐ」への対比
- [[summaries/2023-reflexion]] — 自己検証の偽陽性という共通の罠
