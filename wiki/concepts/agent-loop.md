---
type: concept
aliases: [エージェントループ, thought-action-observation loop, 観測-思考-行動ループ]
tags: [agent-loop, llm-agents]
related:
  - "[[reasoning-and-planning]]"
  - "[[tool-use-and-function-calling]]"
  - "[[multi-agent-systems]]"
  - "[[agent-memory]]"
  - "[[context-engineering]]"
summaries:
  - "[[summaries/2022-react]]"
  - "[[summaries/2026-sakana-fugu]]"
  - "[[summaries/2025-masft]]"
updated: 2026-07-24
---

# Agent Loop（エージェントループ）

LLM（Large Language Model, 大規模言語モデル）エージェントの実行の骨格となる、**観測（observation）→ 思考（thought）→ 行動（action）を繰り返すループ**。1 回のプロンプトで答えを返す通常の LLM 利用と違い、エージェントは行動の結果を環境から受け取り、それを踏まえて次の一手を決める。この「結果を見てから考え直せる」性質が、幻覚（hallucination）の抑制と多段タスクの遂行を可能にする。

## 定式化

[[summaries/2022-react]]（ReAct, 2022）による定式化が現在も標準的である。時刻 $t$ でエージェントは環境から観測 $o_t$ を受け取り、方策 $\pi(a_t|c_t)$ に従って行動 $a_t$ を選ぶ。ここで $c_t = (o_1, a_1, \cdots, o_{t-1}, a_{t-1}, o_t)$ は**それまでの履歴すべて＝コンテキスト**であり、LLM エージェントでは文字通りコンテキストウィンドウ（モデルが一度に読める最大トークン数）に積まれたテキストがこれにあたる。

ReAct の核心は、行動空間を $\hat{\mathcal{A}} = \mathcal{A} \cup \mathcal{L}$（$\mathcal{L}$ は言語空間）に拡張したことである。言語空間に属する「行動」= **thought** は環境に作用せず観測も返さないが、コンテキストに追記されて以後の判断を支える。thought はいわば**書き出されたワーキングメモリ**（scratchpad）であり、これが [[agent-memory]] の最も原始的な形になっている。エージェントが辿った観測・思考・行動の列は trajectory（軌跡）と呼ばれ、デバッグ・評価・ファインチューニングの基本単位となる。

## 設計変数

### thought の密度: dense vs sparse

- **dense thought**: 毎ステップ思考を挟む。1 手ごとの判断が重い知識集約タスク（マルチホップ QA 等）向き。
- **sparse thought**: 要所でだけ思考し、挟むタイミングはモデル自身に決めさせる。行動数が多い意思決定タスク（ALFWorld のような 50 ステップ級の環境）向き。

ReAct はこの両モードを示し、sparse thought だけでも「目標の分解／部分目標の完了追跡／次の部分目標の決定／常識による所在推論」が働けば成功率が激変することを実証した（ALFWorld で思考なし 45% → あり 71%）。

### 停止条件

ループには打ち切りが要る。ReAct では `finish[answer]` という明示的な終了行動と、タスクごとのステップ上限（HotpotQA 7 / FEVER 5。それ以上は性能が伸びないため）を併用し、上限に達したら別手法（CoT-SC）へフォールバックした。**「解けないと判断したら戦略を切り替える」**という設計は現在のエージェントでも重要である。

## 典型的な失敗モード

[[summaries/2022-react]] の失敗分析から、ループ構造に固有の失敗が整理できる。

- **反復ループ**: 直前と同じ思考・行動を繰り返して抜け出せなくなる（ReAct の reasoning error の主要因）。貪欲デコードが一因とされ、現在も loop detection として対策される。マルチエージェントでも消えない——[[summaries/2025-masft]] の実測では「ステップの反復」（FM-1.3, 11.5%）と「会話履歴の喪失」（FM-1.4）が MAS の主要失敗モードとして残った。
- **状態の見失い**: 思考を持たない行動のみのループ（Act）は、「自分はいまナイフを持っている」「まだ流しに行っていない」といった状態を追跡できず、無効な行動を繰り返す。thought はこれを防ぐ最小の仕掛けである。
- **観測の質への依存**: 検索が有益な情報を返さないとその後の推論全体が脱線する（ReAct のエラーの 23%）。ループは自己修復もするが、悪い観測に引きずられもする。

## 位置づけと発展

ReAct のループは「LLM 呼び出し 1 回 = 1 ステップ」の最小構成であり、後のエージェントはこの骨格の上に、ツールの構造化（[[tool-use-and-function-calling]]）、長期記憶（[[agent-memory]]）、コンテキストへの情報の積み方の最適化（[[context-engineering]]）、失敗の振り返り（[[self-reflection]]）、複数エージェントへの分業（[[multi-agent-systems]]）を積み増していった。さらに、ループの各ターンで「**どのモデルがそのステップを担当するか**」自体を学習対象にする段階にも到達している——[[summaries/2026-sakana-fugu]] の Fugu は、ハーネスの相互作用状態からターンごとに担当ワーカーを選び直すオーケストレータで、「GPT が構築し、決定的なデバッグ地点で Opus に交代する」といった役割交代を学習で獲得した。人間がループの途中に介入する点（HITL, Human-in-the-Loop）としては、ReAct が示した「thought を編集して軌道修正する」方式が先駆けである。

## 関連ページ

- [[reasoning-and-planning]] — ループ内の「思考」の設計
- [[tool-use-and-function-calling]] — ループ内の「行動」の設計
- [[agent-memory]] — コンテキストを超える記憶（未作成）
- [[context-engineering]] — コンテキストの積み方（未作成）
- [[summaries/2022-react]] — 本ページの主要な根拠原典
