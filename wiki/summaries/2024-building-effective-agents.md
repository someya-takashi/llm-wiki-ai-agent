---
type: summary
source_path: "raw/articles/Building Effective AI Agents.md"
source_kind: blog
title: "Building Effective AI Agents"
authors: ["Erik S., Barry Zhang（Anthropic）"]
year: 2024
venue: "Anthropic Engineering Blog（初出 2024-12。取り込んだのは Claude Agent SDK 言及を含む改訂版）"
ingested: 2026-07-24
tags: [agent-frameworks, agent-loop, tool-use-and-function-calling, multi-agent-systems, llm-agents]
translation: "[[translations/2024-building-effective-agents]]"
---

# Building Effective AI Agents

> 原典: [[translations/2024-building-effective-agents]] ・ `raw/articles/Building Effective AI Agents.md`
> 著者・年: Erik S., Barry Zhang（Anthropic）・2024（改訂版）

## 一言まとめ

数十の顧客チームとの実務経験から Anthropic がまとめた、エージェント設計の**事実上の標準的な実務指針**。「最も成功した実装は複雑なフレームワークではなく**単純で合成可能なパターン**を使っていた」という一文に集約される。workflow（事前定義されたコードパス）と agent（LLM が自らプロセスを決める）の区別、5 つの workflow パターン、simplicity / transparency / ACI の 3 原則を提示し、[[summaries/2025-masft]] が MAS 設計原則の根拠として引用するなど、研究側からも参照される一次資料になった。

## 背景と問題意識

2024 年末、エージェントブームの中でフレームワークと抽象化レイヤーが乱立し、「とりあえずエージェント化する」実装が量産されていた。本記事はそれへのカウンターで、出発点は徹底して保守的である——**「可能な限り最も単純な解を探し、複雑さは成果を実証的に改善するときだけ足す」**。多くのアプリは検索と文脈内例つきの単一 LLM 呼び出しで足り、agentic 化はレイテンシとコストを性能と引き換えにする明示的なトレードオフだと位置づける。

## 主張

### 区別: workflow vs agent

- **workflow** = LLM とツールを**事前定義されたコードパス**でオーケストレートするシステム。予測可能性・一貫性が欲しい定型タスク向き。
- **agent** = LLM が**自らプロセスとツール使用を動的に決める**システム。ステップ数を予測できないオープンエンドな問題向き。ただし高コストで誤りが複利的に積み重なるため、サンドボックスとガードレールが前提。

この二分法は「自律性は目的でなく設計上の選択肢」という含意を持ち、wiki の [[agent-loop]]・[[agent-frameworks]] の記述の基礎になる。

### 基礎ブロックと 5 つの workflow パターン

基礎は **augmented LLM**（検索・ツール・記憶で拡張した LLM。接続の標準として [[model-context-protocol]] に言及）。その上に:

| パターン | 何をするか | 使いどころ |
| --- | --- | --- |
| **Prompt chaining** | タスクを直列のステップに分解し、中間に gate（プログラム検査）を挟む | きれいに固定サブタスクへ分解できるとき |
| **Routing** | 入力を分類して特化プロンプト/モデルへ振り分け | 明確なカテゴリがあるとき（易問→Haiku、難問→Sonnet 等） |
| **Parallelization** | sectioning（独立サブタスクの並列）と voting（同一タスクの多数決） | 速度、または多視点による信頼度が欲しいとき |
| **Orchestrator-workers** | 中心 LLM が動的にタスクを分解しワーカーへ委譲・統合 | サブタスクを事前に予測できないとき（コーディング等） |
| **Evaluator-optimizer** | 生成役と評価役をループさせる | 明確な評価基準があり反復洗練が効くとき |

<figure>

![](../../raw/assets/2024-building-effective-agents/fig7.png)

<figcaption>図7（再掲）: 自律エージェントの本質。「環境フィードバックに基づきループの中でツールを使う LLM」——Human との対話でタスクを確定し、Environment との Action/Feedback ループを回し、Stop 条件で終わる。</figcaption>
</figure>

### agent の定義と 3 原則

agent は「**環境フィードバックに基づいてループの中でツールを使う LLM に過ぎない**」——各ステップで環境から ground truth（ツール実行結果・コード実行）を得て進捗を評価することが決定的で、停止条件（最大反復数）とチェックポイントでの人間の介入を組み込む。実装原則は:

1. **Simplicity** — 設計を単純に保つ
2. **Transparency** — 計画ステップを明示的に見せる
3. **ACI**（agent-computer interface）— **ツールの定義と文書に、プロンプト本体と同等のエンジニアリングを注ぐ**。HCI に注ぐのと同じ労力を ACI へ。具体策: モデルが「考える」余地のある形式を選ぶ（diff の行数勘定や JSON エスケープのような書式オーバーヘッドを排す）、ジュニア開発者向け docstring のつもりで書く、workbench で誤用を観察して反復、Poka-yoke（ポカヨケ, 誤りを構造的に犯しにくくする治具化）

## 知見（実務からの一般化）

- **フレームワーク論**: Claude Agent SDK・Strands・Rivet・Vellum 等は立ち上げを速くするが、抽象化がプロンプトと応答を見えにくくしデバッグを難しくする。**まず LLM API を直接使い、フレームワークを使うなら中身を理解せよ。本番に向かうほど抽象化を剥がせ**。
- **エージェントが価値を出す条件**（付録 1）: 会話と行動の両方を要し、成功基準が明確で、フィードバックループが可能で、人間の監督を組み込めるタスク。実例はカスタマーサポート（解決成功時のみ課金する事業者が出るほど成功が測れる）とコーディング（テストで検証可能・SWE-bench Verified を PR 記述だけで解く）。ただし**自動テストが通っても人間レビューは要る**。
- ツール形式の選択がエージェントの成否を分ける、という ACI の主張は、ReAct の「意図的に弱いツールが推論を強制した」（[[summaries/2022-react]]）とは逆向きの実務判断で、面白い対照——研究では推論を引き出すためにツールを絞り、本番では誤りを減らすためにツールを磨く。

## 限界・批判的視点

- **定量データがない**。「数十チームの経験」に基づく主張で、パターン間の比較実験や数値はない。処方の妥当性は事後的に [[summaries/2025-masft]] のような研究が部分的に裏づけた（「自明な修正では不十分」「モジュール性・検証・終了条件が効く」は MASFT の知見と符合）が、体系的検証は外部に委ねられている。
- ベンダーの一次発信であり、フレームワーク回避の推奨と自社 API・Claude Agent SDK・MCP への誘導が同居する点は割り引いて読む必要がある。
- 単一エージェント中心の視点で、multi-agent の失敗（orchestration collapse、エージェント間の情報抱え込み → [[multi-agent-systems]]）は射程外。orchestrator-workers を workflow として一括りにしているが、[[summaries/2026-sakana-fugu]] のようにオーケストレーション自体を学習する方向はこの分類の外にある。
- 改訂版のため一部の記述（モデル名・SDK 名）は初出版と異なる。参照時は版に注意（frontmatter に記載）。

## 実装・運用上の示唆

本記事自体が示唆の塊なので、wiki の他ページとの対応だけ整理する:

- 「agent = ツール＋ループ＋環境フィードバック」→ [[agent-loop]] の定式化の実務版
- ACI・ツール形式論 → [[tool-use-and-function-calling]] の設計論点の一次資料
- orchestrator-workers → [[multi-agent-systems]] の orchestrator-worker 構成の出所
- evaluator-optimizer / voting → [[agent-evaluation]] の LLM-as-a-judge・self-consistency と同根の発想
- 「複雑さは実証されたときだけ」→ [[summaries/2025-masft]] の「戦術的修正では不十分、だが構造も測って足せ」と併読すると設計判断の軸になる

## 用語と略称

- **agentic システム** = workflow と agent の総称（本記事の用語法）
- **workflow / agent** = 事前定義コードパスによる協調／LLM 自身による動的制御
- **augmented LLM** = 検索・ツール・記憶で拡張した LLM（本記事の基礎ブロック）
- **ACI** = Agent-Computer Interface（エージェントとツール群の接面。HCI のもじり）
- **HCI** = Human-Computer Interface（人間-コンピュータインターフェース）
- **gate** = ワークフロー中間のプログラムによる検査点
- **ground truth** = 環境から得る客観的な実行結果（ツール出力・テスト結果）
- **Poka-yoke** = ポカヨケ。製造業由来の、誤りを構造的に防ぐ治具設計の考え方
- **MCP** = Model Context Protocol（ツール・データソース接続の標準プロトコル）
- **SWE-bench (Verified)** = 実 GitHub イシュー修正のベンチマーク（人手検証済みサブセット）
- **Claude Agent SDK / Strands / Rivet / Vellum** = 本記事が挙げるエージェント構築フレームワーク・ツール群

## 関連ページ

- [[agent-frameworks]] — 本原典が主要根拠となる概念ページ
- [[agent-loop]] — agent 定義（ツール＋ループ＋環境フィードバック）の実務版
- [[tool-use-and-function-calling]] — ACI・ツール形式設計の一次資料
- [[multi-agent-systems]] — orchestrator-workers パターンの出所
- [[summaries/2025-masft]] — 本記事を設計原則の根拠として引用した研究側の対応物
- [[translations/2024-building-effective-agents]] — 全文翻訳
