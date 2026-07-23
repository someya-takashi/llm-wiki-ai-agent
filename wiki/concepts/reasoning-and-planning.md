---
type: concept
aliases: [CoT, Chain-of-Thought, ToT, Tree of Thoughts, ReAct, 推論と計画]
tags: [reasoning-and-planning, llm-agents, prompting]
related:
  - "[[agent-loop]]"
  - "[[tool-use-and-function-calling]]"
  - "[[self-reflection]]"
  - "[[test-time-compute]]"
summaries:
  - "[[summaries/2022-react]]"
updated: 2026-07-24
---

# Reasoning and Planning（推論と計画）

LLM（Large Language Model, 大規模言語モデル）に**答えへ一足飛びに向かわせず、途中の思考過程や計画を明示的に生成させる**ことで問題解決能力を引き上げる手法群。AI エージェントにおいては「次に何をすべきか」を決める頭脳部分にあたり、[[agent-loop]]（観測→思考→行動のループ）や [[tool-use-and-function-calling]]（ツール呼び出し）と組み合わさってエージェントの中核をなす。

この分野の基本的な緊張関係は次の 2 つである。

1. **内部知識 vs 外部接地** — モデルの中だけで推論すると速くて柔軟だが、幻覚（hallucination, もっともらしいが事実でない内容の生成）に弱い。外部情報に接地（grounding）すると事実性は上がるが、推論の柔軟性が落ちる。
2. **柔軟性 vs 構造** — 自由に考えさせると脱線し、形式を強制すると考えの幅が狭まる。

## 代表手法

### Chain-of-Thought（CoT, 思考連鎖）

「Let's think step by step」に代表される、**推論の途中経過を言語として出力させる**プロンプト手法（Wei et al., 2022。原典は未 ingest）。算術・常識・記号推論で劇的な改善を示し、この分野の出発点となった。派生として、複数の推論経路をサンプリングして多数決を取る **self-consistency（CoT-SC）**、複雑な課題を簡単な部分問題から順に解く least-to-most プロンプティング、例示なしで動く zero-shot CoT がある。

弱点は、推論がモデルの内部知識だけで完結する**静的なブラックボックス**であること。[[summaries/2022-react]] の失敗分析では、HotpotQA（マルチホップ質問応答ベンチマーク）における CoT の失敗の **56% が幻覚**であり、正解した事例にも「推論は幻覚だがたまたま当たった」ものが 14% 含まれていた。

### ReAct（Reason + Act）

**思考（thought）と行動（action）を交互に生成させ、行動の結果（observation）で思考を接地する**パラダイム（[[summaries/2022-react]], 2022）。行動空間を言語空間で拡張する（Â = A ∪ L）という定式化により、「考えること」を行動の一種として同じループに乗せた。

- 幻覚を 56% → **0%** に抑制（HotpotQA 失敗分析）。外部の Wikipedia API への接地が効く。
- 意思決定タスクでは、目標の分解・部分目標の追跡・常識推論を担う疎な thought を挟むだけで、ALFWorld 成功率 71%（思考なし 45%、10⁵ 軌跡で訓練した模倣学習 37%）と大差をつけた。
- 一方、思考-行動-観測の**構造的制約が推論の柔軟性を奪い**、純粋な推論精度では CoT にわずかに劣ることもある（HotpotQA 27.4 vs 29.4）。同じ思考と行動を繰り返して抜け出せなくなる**反復ループ**は ReAct 固有の失敗モードで、現在のエージェント開発でも対策され続けている。

### 内部推論と外部接地の組み合わせ

[[summaries/2022-react]] は、ReAct と CoT-SC を**フォールバックで組み合わせる**のが単体より強いことも示した（ReAct が規定ステップで解けなければ CoT-SC へ、CoT-SC の多数決が割れたら ReAct へ）。「まず調べ、自信があれば内部知識で答える」というこの発想は、後の [[retrieval-augmented-generation]]（RAG, 検索で外部知識を注入する手法）のルーティング設計の原型と見なせる。

### Tree of Thoughts（ToT）ほか

思考を一本の連鎖でなく**木構造で探索**し、行き詰まったら別の枝に戻る手法（Yao et al., 2023。原典未 ingest のため概説のみ）。探索・バックトラックを持ち込む点で CoT の直線性を補う。このほか、推論に使う計算量を推論時に増やして精度を上げる方向は [[test-time-compute]] として発展し、失敗を言語で振り返って次の試行に活かす方向は [[self-reflection]]（代表: Reflexion）として発展した。

## エージェント設計への含意

- thought の役割は少なくとも 4 つある: **目標の分解／部分目標の完了追跡／次の部分目標の決定／常識による補完**（ReAct の ALFWorld プロンプト設計より）。環境状態の復唱だけに限定した thought（Inner Monologue 風）では性能が大きく落ちる——「何を考えさせるか」の設計がエージェントの質を決める。
- thought の**密度**も設計変数である。毎ステップ考える dense thought（知識タスク向き）と、要所でだけ考える sparse thought（行動の多い意思決定タスク向き）の使い分け → 詳細は [[agent-loop]]。

## 関連ページ

- [[agent-loop]] — 推論を組み込む実行ループの側
- [[tool-use-and-function-calling]] — 推論を接地させる行動の側
- [[self-reflection]] — 失敗からの学習（未作成）
- [[test-time-compute]] — 推論時計算のスケーリング（未作成）
- [[summaries/2022-react]] — 本ページの主要な根拠原典
