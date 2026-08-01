---
type: concept
aliases: [エージェントフレームワーク, agentic design patterns, workflow patterns, LangGraph, AutoGen, Claude Agent SDK]
tags: [agent-frameworks, llm-agents]
related:
  - "[[agent-loop]]"
  - "[[tool-use-and-function-calling]]"
  - "[[multi-agent-systems]]"
  - "[[model-context-protocol]]"
  - "[[agent-evaluation]]"
summaries:
  - "[[summaries/2024-building-effective-agents]]"
  - "[[summaries/2025-effective-harnesses]]"
  - "[[summaries/2026-harness-design]]"
  - "[[summaries/2026-meta-harness]]"
  - "[[summaries/2025-llm-reasoning-to-agents]]"
updated: 2026-08-01
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

なお [[summaries/2025-masft]] の分析対象（ChatDev・MetaGPT・AG2 等）は、フレームワークの上に組まれた MAS が実際にどう失敗するかの実測を与えており、「フレームワークがあること」と「正しく設計されていること」が別問題であることを示す。

フレームワークの一段外側の層が **harness（ハーネス）**——ループ・ツール・コンテキスト管理・プロンプトを束ねた実行基盤——である。Anthropic の実務系譜は連作として読める: 設計パターンの語彙（[[summaries/2024-building-effective-agents]], 2024）→ 並列分業の本番運用（[[summaries/2025-multi-agent-research-system]], 2025）→ **長時間タスクのハーネス設計**（[[summaries/2025-effective-harnesses]], 2025）→ その発展と**縮小**（[[summaries/2026-harness-design]], 2026）。3 本目は、数日級のプロジェクトをセッションの連鎖でやり遂げるための二部構成（initializer agent が環境の足場を作り、coding agent が毎セッション 1 単位ずつ進めてクリーンに終了する → 詳細は [[coding-agents]]）を開示した。特筆すべきは復元した脚注の設計情報で、**initializer と coding は「別エージェント」ではなく、同一ハーネスで初期プロンプトだけが違う**——新しい問題（セッション境界）に対して、マルチエージェント化でもフレームワーク追加でもなく「最初のウィンドウにだけ別プロンプト」という最小の介入で応えており、下記 simplicity 原則の実践例そのものである。

4 本目はこの原則の**運用形**を与えた: 「**ハーネスのすべての部品は『モデルが単独でできないこと』についての仮定を符号化しており、その仮定は誤りうるし、モデルの改善で陳腐化する**」。だから新モデルが出たらハーネスを再点検し、もはや荷重を支えていない（load-bearing でない）部品を剥がす——ただし一気に削ると何が効いていたか分からなくなるため、**1 部品ずつ外して影響を確認**する。実例として Opus 4.5→4.6 で context reset とスプリント分解が実際に不要になった一方、planner と evaluator は残存価値を保った（evaluator は「タスクがモデルの単独信頼境界の外にあるときだけコストに見合う」動的な判断になる）。「ハーネスの面白い組み合わせ空間はモデル改善で縮まず、移動する」という結語は、フレームワーク・ハーネス層の仕事の性格を要約している。

### ハーネスの自動探索 — 第三の道

人手設計（作る）→ 仮定の棚卸し（剥がす）に続く第三段階が、**ハーネス設計そのものの自動化**である。Meta-Harness（[[summaries/2026-meta-harness]], Stanford/MIT/KRAFTON, 2026）は、コーディングエージェント（Claude Code）を proposer（提案役）に据え、過去の全候補ハーネスの**ソースコード・スコア・実行トレースをファイルシステム経由で無制限に検分**させて新しいハーネスコード（単一ファイルの Python プログラム）を提案→評価→記録する外側ループを回す。テキスト分類で人手最先端の ACE を +7.7 ポイント（コンテキスト 1/4）、TerminalBench-2 で Terminus-KIRA 超え（Haiku 4.5 では全エージェント中 1 位）、数学検索では発見された単一ハーネスが**未見の 5 モデルに転移**して平均 +4.7 ポイント——「新モデルごとに人手で剥がして作り直す」作業自体が、探索で代替されうることの最初の広域実証である。

既存の自動化（プロンプト最適化の GEPA・OPRO、プログラム進化の AlphaEvolve/OpenEvolve）との分岐点は探索の有無ではなく**フィードバックの渡し方**にある。既存手法はスカラースコア・短い要約・固定テンプレートに圧縮するが、ハーネスは長ホライズンで作用するため、下流の失敗を上流の設計判断に遡る信用割当（credit assignment）には生の実行トレースが要る。アブレーションでは「スコア＋LLM 要約」の最良候補を、生トレースつき条件の**中央値候補**が上回った——要約は診断情報を消すことで害にすらなる。ただし探索セットへの過適合は常在リスクで（TerminalBench-2 は探索と評価が同一の 89 タスク）、proposer も Claude Code 1 種でしか検証されていない。上記の Anthropic 連作が人手でやった「仮説→分離検証→部品の追加・削除」のサイクルをエージェントが自律再現している様子（交絡の診断ログ）は原典の付録 A に詳しく、「このワークフローは 2026 年初頭のコーディングエージェント能力の改善で初めて実用的になった」という脚注の証言とあわせて、[[coding-agents]] の成熟がハーネス層の仕事を変えた記録になっている。

## 実装原則（simplicity / transparency / ACI）

1. **Simplicity** — 設計を単純に。
2. **Transparency** — 計画ステップを明示的に見せる（[[reasoning-and-planning]] の thought を隠さない、に対応）。
3. **ACI**（Agent-Computer Interface）— ツール定義・文書をプロンプト本体と同格に磨く → 詳細は [[tool-use-and-function-calling]]。

## 関連ページ

- [[agent-loop]] — agent パターンの実行構造
- [[tool-use-and-function-calling]] — ACI・ツール設計
- [[multi-agent-systems]] — orchestrator-workers の発展形と失敗モード
- [[model-context-protocol]] — augmented LLM の接続標準（未作成）
- [[agent-evaluation]] — 「足した複雑さを測る」ための方法論
- [[summaries/2024-building-effective-agents]] — 本ページの主要な根拠原典
