---
type: concept
aliases: [エージェントフレームワーク, agentic design patterns, workflow patterns, LangGraph, AutoGen, Claude Agent SDK]
tags: [agent-frameworks, llm-agents]
related:
  - "[[agent-loop]]"
  - "[[llm-programming-systems]]"
  - "[[tool-use-and-function-calling]]"
  - "[[multi-agent-systems]]"
  - "[[model-context-protocol]]"
  - "[[agent-evaluation]]"
  - "[[coding-agents]]"
  - "[[context-engineering]]"
  - "[[harness-engineering]]"
summaries:
  - "[[summaries/2024-building-effective-agents]]"
  - "[[summaries/2025-effective-harnesses]]"
  - "[[summaries/2026-harness-design]]"
  - "[[summaries/2026-meta-harness]]"
  - "[[summaries/2025-llm-reasoning-to-agents]]"
  - "[[summaries/2026-agent-orchestration-guide]]"
  - "[[summaries/2026-managed-agents]]"
  - "[[summaries/2026-dive-into-claude-code]]"
  - "[[summaries/2026-agentic-harness-engineering]]"
updated: 2026-08-03
---

# Agent Frameworks（エージェントフレームワークと設計パターン)

LLM（Large Language Model, 大規模言語モデル）エージェントを**どう組み立てるか**——設計パターンの選択と、それを支えるフレームワーク・SDK——を扱う概念。個々のライブラリの API は陳腐化が速いため、このページは**パターン（長持ちする）とフレームワーク観（変わりうる）を分けて**記述する。

## 出発点: workflow と agent の区別

実務の標準語彙は Anthropic「Building Effective Agents」（[[summaries/2024-building-effective-agents]], 2024）が確立した:

- **workflow** — LLM とツールを**事前定義されたコードパス**でオーケストレートする。予測可能・一貫。
- **agent** — LLM が**自らプロセスとツール使用を動的に決める**。オープンエンドな問題向きだが、高コストで誤りが複利する。

最重要の指針は「**まず最も単純な解（単一 LLM 呼び出し＋検索＋文脈内例）を試し、複雑さは成果を実証的に改善するときだけ足す**」。agentic 化そのものが目的化しがちな領域での、最も引用される歯止めである。

## 設計パターンのカタログ

基礎ブロックは **augmented LLM**（検索・ツール・記憶で拡張した LLM。接続の標準は [[model-context-protocol]]）。その上に積む代表パターン（詳細と図は [[summaries/2024-building-effective-agents]]）:

1. **Prompt chaining** — 直列分解＋中間 gate（プログラム検査）。固定サブタスク向き。
2. **Routing** — 入力分類→特化プロンプト/モデルへ（易問は小型モデルへ、のコスト最適化を含む）。
3. **Parallelization** — sectioning（独立サブタスクの並列）/ voting（多数決）。ガードレールの分離実行や自動評価に使う。
4. **Orchestrator-workers** — 中心 LLM が動的に分解・委譲・統合。サブタスクが予測不能なとき → [[multi-agent-systems]]。
5. **Evaluator-optimizer** — 生成役と評価役のループ。明確な評価基準があるとき → [[agent-evaluation]] の LLM-as-a-judge と同根。
6. **Agent** — ツール＋ループ＋環境フィードバック（＋停止条件と人間のチェックポイント）→ [[agent-loop]]。

パターンは規範ではなく合成の素材であり、測定と反復が前提。この「パターンを足すごとに測る」姿勢は、[[summaries/2025-masft]] が示した「プロンプト改善やトポロジー変更だけでは +14 ポイント止まり」という実測と併読すると立体的になる——**構造は効くが、効いたかは測らないと分からない**。

## フレームワーク観

代表的なフレームワーク・SDK には Claude Agent SDK、Strands Agents SDK（AWS）、LangGraph・AutoGen（AG2）・CrewAI、GUI 系の Rivet・Vellum などがある（顔ぶれと API は変わり続けるので、個別の使い方は各公式 docs を一次資料とする）。横断サーベイ（[[summaries/2025-llm-reasoning-to-agents]], 2025）の整理では、これらは大きく **(1) 汎用オーケストレーション系**（LangChain / LangGraph——チェーンとグラフでツール・記憶・分岐を組む）、**(2) データ／RAG 指向系**（LlamaIndex——文書接続と検索を軸に据える）、**(3) マルチエージェント協調系**（CrewAI・AutoGen（AG2）・Microsoft Semantic Kernel——役割分担した複数エージェントの会話・協調を組む）、**(4) 軽量・低抽象系**（OpenAI Swarm / Agents SDK・OctoTools——最小限の足場でエージェントループを書く）に類型化できる。同サーベイは、抽象度の高いフレームワークほど素早く組めるが内部挙動が見えにくく、低抽象なものほど自由度と引き換えに実装量が増える、というトレードオフを指摘する。近年はこれらフレームワーク間・エージェント間の相互接続を担う標準プロトコル（[[model-context-protocol]] の MCP／A2A／ACP）の層が立ち上がりつつあり、「どのフレームワークで書くか」と「エージェント同士をどう繋ぐか」は別の設計軸になった。

実務指針として引用され続けているのは次の 3 点（[[summaries/2024-building-effective-agents]]）:

- **まず LLM API を直接使う**。多くのパターンは数行で書ける。
- フレームワークの**抽象化はプロンプトと応答を見えにくくし、デバッグを難しくする**。使うなら中身を理解する。誤った内部仮定は顧客の誤りの常連。
- **本番に向かうほど抽象化を剥がす**。立ち上げの速さと運用の可観測性はトレードオフ → [[agent-observability]]。

自前で組むのでなく**オーケストレーション・プラットフォーム**を買う場合の評価軸は、実務ガイド（[[summaries/2026-agent-orchestration-guide]]）が挙げる 4 点に集約される: **状態とメモリの永続化**（エージェントの相互作用をまたいで何がどう保持されるか。製品ごとの差が最も大きい）、**複数のエージェントフレームワークへのネイティブ対応**、**組み込みの可観測性**、**ガバナンス制御**（監査ログ・ロールベースアクセス・データレジデンシー）。どれか 1 つでも欠ければ、その穴は結局チームが自前で埋めることになる。加えて **ベンダーロックイン**——ワークフローが単一のモデルプロバイダや独自のエージェント形式に固定されていないか、モデルの差し替えと他プラットフォームへの移行がどれだけ容易か——を選定時に確かめる。ただし同ガイドはベンダー自身が書いたものであり、自社製品を「あるべき機能の例」として織り込んでいる点は割り引いて読む必要がある。

なお [[summaries/2025-masft]] の分析対象（ChatDev・MetaGPT・AG2 等）は、フレームワークの上に組まれた MAS が実際にどう失敗するかの実測を与えており、「フレームワークがあること」と「正しく設計されていること」が別問題であることを示す。

**フレームワークの一段外側の層が harness（ハーネス）** ——ループ・ツール・コンテキスト管理・記憶・プロンプトを束ねた実行基盤である。**両者の違いは、フレームワークが「エージェントを組み立てる道具立て（製品）」であるのに対し、ハーネスは「モデルを包む実行基盤そのもの」**であること。本 wiki はハーネスの原典が 7 本に達したので、**その設計・縮小・自動探索・自動進化・仮想化は 2026-08-03 に [[harness-engineering]] へ切り出した**。本ページからの主要な導線は次の 3 つである。

- **Anthropic の実務連作**——設計パターンの語彙（[[summaries/2024-building-effective-agents]]）→ 並列分業の本番運用（[[summaries/2025-multi-agent-research-system]]）→ 長時間タスクのハーネス（[[summaries/2025-effective-harnesses]]）→ その発展と**縮小**（[[summaries/2026-harness-design]]）→ **ハーネスの下に敷くインターフェース**（[[summaries/2026-managed-agents]]）。**「ハーネスのすべての部品は『モデルが単独でできないこと』への仮定であり、モデルの改善で陳腐化する」**という原則と、**「ハーネスの面白い組み合わせ空間はモデル改善で縮まず、移動する」**という結語が、この層の仕事の性格を要約している。
- **設計の担い手が変わりつつある**——[[summaries/2026-meta-harness]] は**ハーネスコードそのものを探索で発見**させ、[[summaries/2026-agentic-harness-engineering]] は**観測された失敗から編集を導く自動進化**を実装した。後者の測定は本ページにも直接効く——**ツール・ミドルウェア・記憶に符号化した挙動は転移するが、システムプロンプトの散文は単独では後退する（−2.3 pp）**。
- **出荷された実装の解剖**——[[summaries/2026-dive-into-claude-code]] は Claude Code v2.1.88 のソースから、**コードベースのうち意思決定ロジックは推定 1.6%、残る 98.4% が運用ハーネス**であることを示した（Tier C の推定）。**フレームワークの抽象度の議論は、この比率を前提に読み直す価値がある。**

**「meta-harness」の語義の衝突**（探索の外側ループ 対 差し替え可能にする下位基盤）も [[harness-engineering]] に整理した。

## 拡張点は 3 つしかない — 本番系から抽出した一般則

[[summaries/2026-dive-into-claude-code]]（2026）が Claude Code v2.1.88 の公開ソースから抽出した一般化は、**フレームワークを作る側・選ぶ側の双方にとって最も実用的な整理**だと思う。

> **あらゆるエージェントループは 3 つの注入点を持つ。`assemble()` はモデルが*何を見るか*を、`model()` は*何に手を伸ばせるか*を、`execute()` は行動が*実行されるか／どう実行されるか*を制御する。**

```python
while not stopped:
    context = assemble(tool_schemas, history, hook_additions)  # (a) 何を見るか
    action  = model(context, tools)                            # (b) 何に手を伸ばせるか
    if action.is_text_only():
        stopped = run_stop_hooks(action); continue
    if not permitted(action): continue                         # (c) 実行されるか
    action = run_pre_tool_hooks(action)
    result = execute(action)
    result = run_post_tool_hooks(result)
    history.append(action, result)
```

**この 3 点に、拡張機構が過不足なく配置できる。**

### 4 機構をコンテキストコストで階層化する

Claude Code は単一の統一された拡張 API を持たず、**4 つの機構をコンテキストコストの順に並べている**。

| 機構 | 独自の能力 | **コンテキストコスト** | 注入点 |
|---|---|---|---|
| **フック** | ライフサイクル介入・イベント駆動の自動化 | **ゼロ**（既定） | `execute()` |
| **スキル** | ドメイン固有の指示・メタツール呼び出し | **低**（frontmatter の記述のみ） | `assemble()` |
| **プラグイン** | 複数コンポーネントの梱包＋配布 | **中**（内容による） | 3 点すべて |
| **MCP サーバー** | 外部サービス統合（マルチトランスポート） | **高**（ツールスキーマ） | `model()` |

**「なぜ 4 つも要るのか」への答えが、フレームワーク設計の一般則になっている。**

> **異なる種類の拡張性は、コンテキストウィンドウに異なるコストを課す。単一の機構では、ゼロコンテキストのライフサイクルフックからスキーマの重いツールサーバーまでの全範囲を、拡張の作者に不要なトレードオフを強いることなく張れない。**

**段階的なコストの順序づけが意味するのは「安い拡張はコンテキストを使い果たさずに広くスケールでき、高価なものは本当に新しいツール面を必要とする場合のために取っておける」ことである。** 単一の tool-only API を提供する枠組みは、**ゼロコンテキストで済むはずのライフサイクル介入にもツールスキーマのコストを払わせてしまう**。

**代償も明記されている**——**4 機構は、ある統合タスクにどれを使うか決めるときの学習曲線を増やす**。そして組み合わせが創発的な挙動を生む（プラグインが PreToolUse フックでツール入力を修正する、パススコープのルールが会話の途中で遅延ロードされて分類器の挙動を変える等）。**「単一の設定ファイルからは予測が難しい」**。

### メタツール — 拡張をツールとして露出する

Claude Code の基本ツールプールには **2 つのメタツール**が並ぶ。**両者は根本的に異なる**。

- **SkillTool** — 名前でスキルを起動し、**その指示を現在のコンテキストウィンドウへ注入する**。
- **AgentTool** — **新しい隔離されたコンテキストウィンドウを生成する**（→ [[multi-agent-systems]]）。

**だから前者は低コスト、後者は高コストになる。** 「拡張を追加のツールとして見せる」という同じ形をとりながら、**コンテキストへの影響がまったく違う 2 つを区別している**のが設計として重要である。

ツールプールの組み立て（`assembleToolPool()`）は 5 段——基本ツールの列挙（**最大 54。19 は無条件、35 は機能フラグ・環境・ユーザー種別に依存**）→ モードによるフィルタ（**`CLAUDE_CODE_SIMPLE` モードでは Bash・Read・Edit の 3 つだけ**）→ deny ルールによる事前フィルタ → MCP ツールの統合 → **重複除去（組み込みが MCP に優先）**。**すべての実行パス（REPL と AgentTool の両方）が同じ関数を呼ぶ**ので、組み立ては一貫する。

### フレームワークは層をなす — Claude Code と OpenClaw の合成

**本ページ冒頭の「フレームワーク観」に対する 2026 年時点の追記として重要な観察**が、同論文の OpenClaw との対比から出ている。

OpenClaw は約 2 ダースのメッセージング表面をエージェントランタイムへ繋ぐ **local-first の WebSocket ゲートウェイ**で、Claude Code とは正反対の賭けをしている——**エージェントループは「系の中心」ではなく「制御平面の中の 1 コンポーネント」**であり、**プラグインは単一エージェントのコンテキストではなくゲートウェイ全体の能力面を拡張する**（12 の能力型を中央レジストリへ登録する）。

**そして 2 つは排他的ではない。**

> **OpenClaw は ACP（Agent Client Protocol）を通じて Claude Code・OpenAI Codex・Gemini CLI を外部のコーディングハーネスとしてホストできる。**（中略）**エージェントの設計空間は平坦な分類ではなく層状のものであり、ゲートウェイ水準の系とタスク水準のハーネスが合成できる。**

**「どのフレームワークを選ぶか」ではなく「どの層のフレームワークを、どの層と組み合わせるか」という問いになりつつある**、というのがこの対比の含意である。→ [[model-context-protocol]]（MCP と ACP の関係）

## 実装原則（simplicity / transparency / ACI）

1. **Simplicity** — 設計を単純に。
2. **Transparency** — 計画ステップを明示的に見せる（[[reasoning-and-planning]] の thought を隠さない、に対応）。
3. **ACI**（Agent-Computer Interface）— ツール定義・文書をプロンプト本体と同格に磨く → 詳細は [[tool-use-and-function-calling]]。

## 関連ページ

- [[agent-loop]] — agent パターンの実行構造
- [[tool-use-and-function-calling]] — ACI・ツール設計
- [[multi-agent-systems]] — orchestrator-workers の発展形と失敗モード
- [[model-context-protocol]] — augmented LLM の接続標準
- [[agent-evaluation]] — 「足した複雑さを測る」ための方法論
- [[agent-observability]] — 抽象度と可観測性のトレードオフ、障害の局在化
- [[summaries/2024-building-effective-agents]] — 本ページの主要な根拠原典
- [[summaries/2026-managed-agents]] — ランタイム分離（brain / session / hands）の根拠原典
- [[summaries/2026-dive-into-claude-code]] — 3 つの注入点・4 機構のコンテキストコスト階層・メタツール・ACP による層の合成の根拠原典
- [[harness-engineering]] — モデルを包む実行基盤そのもの。設計・縮小・自動探索・自動進化・仮想化（本ページから 2026-08-03 に切り出した）
