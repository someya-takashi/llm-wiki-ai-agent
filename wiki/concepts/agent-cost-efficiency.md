---
type: concept
aliases: [コスト効率, cost efficiency, エージェントのコスト, トークン経済, token economics, 予算, budget, futility detection, 不毛さの検知, Token Snowball, トークンの雪だるま, expensive failures, 高くつく失敗, EuTB, EuCB, SWE-Effi]
tags: [agent-cost-efficiency, agent-evaluation, context-engineering, test-time-compute, llm-agents]
related:
  - "[[agent-evaluation]]"
  - "[[context-engineering]]"
  - "[[test-time-compute]]"
  - "[[llm-inference-optimization]]"
  - "[[llm-serving-systems]]"
  - "[[multi-agent-systems]]"
  - "[[harness-engineering]]"
  - "[[coding-agents]]"
  - "[[agent-environments]]"
  - "[[agent-observability]]"
summaries:
  - "[[summaries/2025-swe-effi]]"
  - "[[summaries/2024-swe-agent]]"
  - "[[summaries/2025-swe-bench-sonnet]]"
  - "[[summaries/2025-multi-agent-research-system]]"
  - "[[summaries/2025-manus-context-engineering]]"
  - "[[summaries/2026-harness-design]]"
  - "[[summaries/2026-dive-into-claude-code]]"
  - "[[summaries/2026-meta-harness]]"
  - "[[summaries/2026-kimi-k2.5]]"
  - "[[summaries/2026-kimi-k3]]"
  - "[[summaries/2025-kimi-k2]]"
  - "[[summaries/2026-agent-orchestration-guide]]"
updated: 2026-10-25
---

# Agent Cost Efficiency（エージェントのコスト効率）

**エージェントが成果を出すために使う資源——トークン・金額・時間・計算——を、成功率と同格の評価軸として扱う**ためのページ。エージェントは観測→思考→行動のループ（→ [[agent-loop]]）を何十回も回し、そのたびに履歴を読み直すので、1 回の質問応答とは桁の違う資源を使う。同じ解決率でも、1 タスクに 10 セントかかるのか 10 ドルかかるのか、数分か数時間かで、使える場面はまったく変わる。

2026-10-25 に、解決率だけでなく予算あたりの解決を測る [[summaries/2025-swe-effi]] を機に新設した。それまで wiki の各所に散らばっていたコストの記述——Research 機能の「チャット比 15 倍のトークン」、SWE-agent の「1 件 $1.59」、Manus の「入力:出力 100:1」など——もここに集めている。

### 近隣ページとの境界

| ページ | 扱うこと |
|---|---|
| **本ページ** | エージェントの**ふるまい**として資源がどこで使われ、どう測り、どう抑えるか |
| [[llm-inference-optimization]] / [[llm-serving-systems]] | 1 回の推論を速く安くする（カーネル、量子化、バッチング、KV cache の管理） |
| [[test-time-compute]] | 推論時に計算を足して精度を買う考え方と、その上限 |
| [[context-engineering]] | コンテキストに何をどれだけ載せるか（コストの最大の発生源の 1 つ） |
| [[agent-evaluation]] | 評価の方法論全般。本ページはそのうちコスト軸を掘り下げる |

## 規模感 — wiki に記録されている数字

原典ごとに条件がまったく違うので、横に並べて比べるためのものではない。**どこにどれだけかかりうるかの目安**として読む。

| 原典 | 数字 | 何と比べてか |
|---|---|---|
| [[summaries/2025-multi-agent-research-system]] | トークンは**チャットの約 15 倍**（マルチエージェント）、エージェント一般で約 4 倍 | 通常の対話 |
| [[summaries/2026-dive-into-claude-code]] | エージェントチームは標準セッションの**約 7 倍**のトークン | plan モードの標準セッション |
| [[summaries/2024-swe-agent]] | SWE-bench 1 件あたり平均 **$1.59**（予算上限 $4） | 非対話の RAG は $0.13 |
| [[summaries/2025-swe-bench-sonnet]] | 成功した実行の多くが**数百ターン・10 万トークン超** | 例示の成功例は 12 ステップ |
| [[summaries/2025-swe-effi]] | SWE-agent＋GPT-4o-mini で平均**入力 814 万トークン**・呼び出し 181 回 | 同じ足場の Qwen3-32B は 44 万トークン・35.5 回 |
| [[summaries/2025-manus-context-engineering]] | 入力:出力トークン比 **100:1**、キャッシュの有無で単価 **10 倍**、1 タスク平均 50 ツール呼び出し | — |
| [[summaries/2026-harness-design]] | 単独 $9・20 分 に対し、ハーネス v1 $200・6 時間、v2 $124.70・3 時間 50 分 | 同じプロンプトでの比較 |
| [[summaries/2026-kimi-k3]] | BrowseComp で 91.2% を **$2.03/タスク** | 他モデルとの散布図 |

## コストはどこから来るか

エージェントの 1 タスクのコストは、おおまかに次の積で決まる。

**（LLM 呼び出しの回数）×（1 回あたりに読む入力の長さ）＋（出力・思考トークン）＋（ツールの実行時間）**

### (1) 回数 — 往復が多いほど高くつく

[[summaries/2025-swe-effi]] の観察では、**総コストを決めるのは相互作用の回数で、1 回の重さではない**。推論の強いモデル（Qwen3-32B）は呼び出しあたりのトークンが多くても、往復が少ないので総トークンは減った（AutoCodeRover で 15 回対 38 回）。賢いモデルが簡潔な計画を立てると、無駄なやり取りが消える。

### (2) 1 回あたりの入力 — Token Snowball

多くの足場は、応答と観測をそのまま次のプロンプトに付け足す。すると**進展がなくても呼び出しのたびに入力が伸び**、コストは回数に対して 2 乗に近い形で増える。SWE-Effi はこれを **Token Snowball（トークンの雪だるま）**と名付けた。文脈が古い情報で散らかるとモデルが手がかりを見失い、往復がさらに増える悪循環にもなる。精度の側から同じ現象を見たのが SWE-bench 原論文の「文脈を増やすと解決率が下がる」である（→ [[context-engineering]]）。

### (3) 出力と思考

出力は入力よりずっと高い。SWE-Effi の回帰式では、出力 1 トークンの時間は入力 1 トークンの約 117 倍だった（デコードはプレフィルと違い 1 トークンずつしか進まないため。→ [[llm-inference-optimization]]）。思考（thinking）を長くするほど精度が上がるとは限らず、推論境界を超えると下がる（overthinking。→ [[test-time-compute]]）。

### (4) ツールの実行

テストやビルドを回す足場では、LLM の待ち時間より CPU 時間が支配的になる。SWE-Effi では、Agentless と OpenHands は実行時間の大半が CPU で、Agentless は再現テストと回帰テストを毎回回すため平均 727.9 秒の CPU 時間を使った。**トークン効率が良くても時間効率が悪い**ことがある。

## 最も高くつくのは失敗である

複数の原典が、独立に同じ傾向を報告している。

- **SWE-agent**（[[summaries/2024-swe-agent]]）：解決は中央値 $1.21・12 ステップ、失敗は平均 $2.52・21 ステップ。解決の 93.0% は予算を使い切る前に提出していた。著者らは「予算やトークン上限を上げても性能はさほど伸びない」と結論した。
- **SWE-Effi**（[[summaries/2025-swe-effi]]）：5 足場 × 3 モデルのほぼすべてで、失敗した試行のほうが資源を多く使った。SWE-agent＋GPT-4o-mini では失敗が成功の 4 倍超（887 万対 187 万トークン、658 秒対 167 秒）。
- **Claude 3.5 Sonnet の SWE-bench 実行**（[[summaries/2025-swe-bench-sonnet]]）：コンテキストを使い切るまで試行を続けたタスクがあった。

つまり、**うまくいく試行は早く終わり、行き詰まった試行は予算の上限まで走る**。外部の上限だけで止めると、失敗のたびに上限まで払うことになる。SWE-Effi はこれを**不毛さの検知（futility detection）の欠如**と呼び、停滞の兆候を見分けて早めに中止・方向転換する能力を今後の課題に挙げた。

**実装されている止め方**：

- **呼び出しの深さと実行時間の上限**。エージェント同士がループ状に呼び合ったり無限にリトライしたりする暴走に備える（[[summaries/2026-agent-orchestration-guide]]）。
- **回復の回数制限**。Claude Code は出力上限のエスカレーションを 1 ターン 3 回、reactive compaction を 1 ターン 1 回に制限している（[[summaries/2026-dive-into-claude-code]]）。
- **ループ検出**。Reflexion の AlfWorld 実験は「同じ行動と同じ応答が 3 回を超える」か「30 行動を超える」を失敗とみなした（→ [[self-reflection]]）。
- **予算超過を失敗として数える評価**。Kimi K2.5 は上限を超えた試行を切り詰めでなく失敗として扱うと評価条件に明記した（[[summaries/2026-kimi-k2.5]]）。

## 「トークンを使うほど良くなる」との関係

一見、逆の報告もある。Anthropic の Research 機能では、**BrowseComp の性能分散の 80% をトークン使用量が説明**した（[[summaries/2025-multi-agent-research-system]]）。SWE-Effi では、**多くのトークンを使った試行ほど失敗している**。

これは矛盾ではなく、見ている比較が違う。前者は**構成をまたいで**「より多く考えさせる設計は良くなるか」を見ており、後者は**同じ構成の中で結果別に**「行き詰まった試行がどれだけ食うか」を見ている。両方を合わせると、**トークンを足すべき場面（並列で別の筋を探る、難しい問題を深く考える）と、止めるべき場面（同じ所を回っている）を見分けること**が、コスト効率の中心的な問題になる。

もう 1 つの注意点は、**モデルの能力とトークン予算が掛け算で効く**ことである。Research 機能の報告では、Sonnet 3.7 のトークン予算を 2 倍にするより Sonnet 4 に上げるほうが効いた。SWE-Effi でも、足場の良し悪しはモデルとの相性で大きく変わり、弱いモデルに作り込んだエージェント型の足場を渡すと、トークンを大量に使ったうえで解けなかった。

## 測り方

- **解決率と並べてコストを報告する**。Kimi K2.5 は各ベンチマークのスコアに平均出力トークンを併記した（AIME 96.1% を 25k トークンで、前世代は 94.5% に 30k）。Kimi K3 はスコアとタスクあたりの推論コスト（USD）を散布図にし、コスト効率のフロンティア上にいるかを見せた（→ [[agent-evaluation]]）。
- **予算のもとでの解決を 1 つの数にする**。SWE-Effi は、横軸に資源の消費量、縦軸に累積の解決率をとった曲線の下の面積（AUC）を正規化し、トークン・金額・CPU 時間・推論時間の 4 指標（EuTB・EuCB・EuCTB・EuITB）にした。少ない資源で早く解けるほど高い。
- **時間はハードウェアの揺れを除いて測る**。API の実時間はネットワークやプロバイダで揺れるので、SWE-Effi はトークン数から時間を予測する回帰式（固定 1.457 秒＋入力・出力トークンの一次式）で正規化した。ただしすべてのモデルに同じ式を当てるので、モデルごとの速度差も消える。
- **成功と失敗に分けて測る**。平均だけでは、失敗が成功の数倍かかっていることが見えない。
- **価格は時点つきで書く**。トークン単価はプロバイダと時期で大きく変わる（SWE-Effi は 2025-07-11 時点の openrouter.ai の最安値を使った）。

## 抑え方

| 手段 | 何を減らすか | 原典 |
|---|---|---|
| 古い観測の折り畳み・compaction・段階ごとのリセット | 1 回あたりの入力（Token Snowball） | [[summaries/2024-swe-agent]]、[[summaries/2026-dive-into-claude-code]] → [[context-engineering]] |
| プレフィックスを安定させ、追記だけにする | キャッシュミス（単価 10 倍） | [[summaries/2025-manus-context-engineering]] → [[llm-serving-systems]] |
| 大きな観測はファイルに置いて参照だけ渡す | 1 回あたりの入力 | [[summaries/2025-manus-context-engineering]] |
| 探索をサブエージェントに隔離し、要約だけ返す | メインの文脈の伸び（ただし総トークンは増える） | [[summaries/2025-multi-agent-research-system]] → [[multi-agent-systems]] |
| ツール応答の上限・ページネーション・簡潔版の選択 | 1 回あたりの入力 | [[summaries/2025-writing-effective-tools]] → [[agent-computer-interface]] |
| 停滞の検知と打ち切り、回復の回数制限 | 失敗の試行の長さ | [[summaries/2025-swe-effi]]、[[summaries/2026-dive-into-claude-code]] |
| 思考の予算を訓練で仕込む（K2 の予算制御、K2.5 の Toggle で出力 −25〜30%、K3 の Reasoning Effort RL） | 出力・思考トークン | [[summaries/2025-kimi-k2]]、[[summaries/2026-kimi-k2.5]]、[[summaries/2026-kimi-k3]] → [[test-time-compute]] |
| 足場とモデルの組み合わせを選び直す（弱いモデルには手続き型） | 往復の回数 | [[summaries/2025-swe-effi]] → [[coding-agents]] |
| ハーネスの部品を、効いているか確かめて剥がす | 不要な評価・往復 | [[summaries/2026-harness-design]] → [[harness-engineering]] |

## 訓練のコスト — RL のロールアウト

コストは推論時だけの問題ではない。SWE タスクの強化学習では、1 回のロールアウトごとに環境を準備し、推論し、コードを実行し、テストで検証する。**足場が遅い・高いと、訓練全体の試行回数が頭打ちになる**。SWE-Effi は、DeepSWE が低レイテンシの独自足場を作り、SkyRL が OpenHands を大きく最適化し、SWE-RL が軽量な Agentless-Mini を使ったうえで実行フィードバックを避けた例を挙げ、**高くつく失敗を避ける軽い足場のほうが RL の土台として「進化させやすい」**と主張した。失敗したロールアウトが上限まで走る問題は、推論時と同じく訓練でもコストを押し上げる。Kimi K3 がチェックポイントと再開の速い microVM サンドボックスを大量に作ったのも、この訓練側のコストへの対処である（→ [[agent-environments]]）。

## 設計論点

- **何に上限をかけるか**。トークン・金額・時間・呼び出し回数のどれで止めるかで、どの失敗が切られるかが変わる。SWE-Effi では、SWE-agent に金額の上限をかけたため、単価の安い GPT-4o-mini だけが 181 回まで呼べた。**上限の単位は、モデルの単価によって公平さが変わる**。
- **トークン効率と時間効率は別物**。テストを多く回す足場はトークンが少なくても遅い。使う場面（毎日の開発支援か、大量に回す RL か）で優先する軸が変わる。
- **キャッシュを考えた単価**。入力の大半がキャッシュに当たるなら、入力トークン数の多さはそれほど高くつかない。逆にプロンプトの先頭を毎回変えると、同じトークン数でも単価が跳ね上がる。トークン数だけでコストを比べると、この差が見えない。
- **提供元の数字は条件しだい**。報告されたコストは、プロバイダの価格、キャッシュの有無、並列化の有無、打ち切り条件に強く依存する。横比較する前に条件をそろえる必要がある（→ [[agent-evaluation]] の provider-reported の問題）。
- **証拠はまだ薄い**。コストを主題にした一次資料は wiki では SWE-Effi だけで、それも 50 問・安価な 3 モデルの探索的な研究である。他の数字は各原典の副産物として報告されたものにとどまる。

## 関連ページ

- [[agent-evaluation]] — 解決率とコストを並べて報告する評価
- [[context-engineering]] — Token Snowball と文脈の圧縮
- [[test-time-compute]] — 計算を足して精度を買う考え方と上限
- [[llm-inference-optimization]] ・ [[llm-serving-systems]] — 1 回の推論を速く安くする層
- [[multi-agent-systems]] — 並列化とトークンの経済
- [[harness-engineering]] — ハーネスの部品とコスト
- [[coding-agents]] — 手続き型とエージェント型の足場
- [[agent-environments]] — RL のロールアウトのコスト
- [[summaries/2025-swe-effi]] — 本ページの主な一次資料
