---
type: overview
tags: [llm-agents, ai-agent, overview]
updated: 2026-08-02
---

# Overview — AI Agent

この wiki が扱う領域全体の総括ページ。**原典を取り込むたびに更新する**（ingest skill の手順 7）。
原典はまだ少なく、以下の大部分は「これから埋めていく骨組み」である。原典に裏付けられた箇所には `[[summaries/...]]` への参照を付してある。

## この領域の対象

**AI Agent（AI エージェント）** とは、LLM（Large Language Model, 大規模言語モデル）を推論エンジンとして、目標を与えられると自ら計画を立て、ツールを呼び出し、環境から返ってきた結果を観測しながら多段階のタスクを遂行するシステムを指す。1 回のプロンプトで答えを返す従来の LLM 利用と違い、**観測→思考→行動を繰り返すループ**を回す点、そして**外部世界に副作用を及ぼせる**点が本質的な差になる。

この wiki は、エージェント本体だけでなく、その土台となる LLM と開発基盤までを射程に入れる。

この領域全体の初期の見取り図は Fudan NLP のサーベイ（[[summaries/2023-llm-agents-survey]], 2023, 686 文献）が与えた: エージェント概念の哲学→AI の系譜、LLM が脳に適する 4 性質（自律性・反応性・能動性・社会的能力）、**brain / perception / action** の 3 モジュール枠組み、応用 3 分類（単一・マルチ・人間協調）、エージェント社会、評価 4 観点と安全リスク。総論の概念ページ [[llm-agents]] はこのサーベイを主要根拠に、以下の各軸への入口をまとめている。ただし同サーベイは 2023 年秋時点の地図であり、その後の「訓練で作るエージェント」（K2/K2.5・RLVR）・ハーネス工学・MCP・computer use は射程外——以後の原典群がこの地図をどう更新したかは各要約の「限界」節を参照。

## 骨組み（今後埋めていく軸）

### 1. エージェントの基本構造

- agent loop（エージェントループ, 観測→思考→行動の反復）、ReAct 型の推論と行動の交互実行 → [[agent-loop]], [[reasoning-and-planning]]
  - この骨格の原型は ReAct（[[summaries/2022-react]], 2022）が確立した。思考を「環境に作用しない行動」として行動空間に加えるという定式化により、外部接地で幻覚を抑えつつ（CoT の失敗要因 56% → 0%）、few-shot プロンプトのみで模倣・強化学習を上回れることを示した。
- tool use / function calling（モデルが JSON スキーマに沿った引数で外部ツールを呼ぶ仕組み）→ [[tool-use-and-function-calling]]
  - 初期形は ReAct の 3 アクション Wikipedia API のようなプロンプト規約によるツール定義で、その後 API レベルの構造化された function calling へ発展した。並行して、ツール利用能力を**プロンプトでなくモデルの重みに埋め込む**系譜が Toolformer（[[summaries/2023-toolformer]], 2023）で始まった——「その API 呼び出しが将来トークンの予測を助けたか」という自己教師あり信号で訓練例を選別し、6.7B の GPT-J が GPT-3(175B) を zero-shot で上回った。連鎖・対話ができないという限界が、後のエージェントループ（[[agent-loop]]）とデータ合成（[[summaries/2025-kimi-k2]]）の動機になった。
- planning（計画立案）と self-reflection（自己反省による軌道修正）→ [[reasoning-and-planning]], [[self-reflection]]
  - 推論の系譜の起点は CoT（[[summaries/2022-chain-of-thought]], 2022）。few-shot 例示に思考連鎖を入れるだけで推論が創発する（約 100B 規模で急伸）ことを示し、「答える前に考えさせる」設計と test-time compute の発想の源流となった。
  - Reflexion（[[summaries/2023-reflexion]], 2023）は、失敗の反省文をエピソード記憶に蓄えて次試行に注入する「言語的強化学習」で、重み更新なしの試行間学習を実現した——CoT（考えてから答える）→ ReAct（考えて動く）→ Reflexion（失敗から学ぶ）で単一エージェントの基本系譜が完結する。
- memory（短期＝コンテキスト内、長期＝外部ストア）→ [[agent-memory]]
  - 原型は MemGPT（[[summaries/2023-memgpt]], 2023）: コンテキストを OS の物理メモリに見立て、**LLM 自身が function call で記憶をページング**する virtual context management を定式化した（working context・archival memory・recursive summary の語彙の出発点）。Deep Memory Retrieval で GPT-4 単体 32.1% → 92.5%。
  - A-Mem（[[summaries/2025-a-mem]], 2025）は自己管理の対象を配置から**組織化・進化**へ拡張: Zettelkasten 型のノート・リンク・記憶進化（新しい記憶が既存記憶を書き換える）で、長期会話の multi-hop QA を約 1/10 のトークンで 2 倍超の性能。「検索の agency（agentic RAG）」と「索引の agency（agentic memory）」を分ける境界も整理した。
- context engineering（限られたコンテキストウィンドウに何をどう積むかの設計）→ [[context-engineering]]
  - この営みを命名・定式化したのが [[summaries/2025-effective-context-engineering]]（Anthropic, 2025-09）。制約の呼び名が **注意予算（attention budget）**——コンテキストは容量でなく「限界効用が逓減する有限資源」であり、**トークン数が増えるほど正確な想起能力が落ちる（context rot）**。機構まで降りた説明（$n$ トークンで $n^2$ の対関係／訓練分布が短い系列に偏るので長距離依存の専用パラメータが少ない／位置符号化の内挿は劣化を伴う）から導かれる帰結が「**明確な崖ではなく性能の勾配**」で、指導原理は「**望ましい結果の尤度を最大化する、最小の高信号なトークン集合を見つけよ**」。各論としてシステムプロンプトの**適切な高度**（脆い if-else と漠然とした指針の間）、ツールの**実行可能な最小集合**（人間がどれを使うか断言できないならエージェントにもできない）、そして取得戦略の転換——埋め込みによる推論前検索から、識別子だけ持って実行時に引く **just-in-time** と **段階的開示**へ——が示される。長時間タスクには **compaction / 構造化ノート取り / サブエージェント**の 3 手法と使い分け基準。**定量的な結果は一切なく、実務経験に基づく指針として読む記事**である点は押さえておく。
  - MemGPT の main context 3 分割（不変の規則／更新される要点／流れる履歴）と閾値駆動の退避が区画化の原型。本番運用のパターン（フェーズ要約・handoff・参照渡し）は [[summaries/2025-multi-agent-research-system]] が記録。
  - Kimi K2.5（[[summaries/2026-kimi-k2.5]], 2026）は「溢れてから削る」reactive な切り詰め（Summary / Discard-all 等）に対し、分解時点でコンテキストを分割する **context sharding**（Agent Swarm）を対置し、同一モデル比較で精度・効率の優位を示した。
  - 本番製品側の 6 原則は Manus（[[summaries/2025-manus-context-engineering]], 2025）が開示: **KV cache ヒット率を最重要指標**とする追記専用設計（入出力比 100:1・キャッシュ有無で単価 10 倍）、ツールは削除せずロジットマスク、ファイルシステム＝復元可能な外部化メモリ、todo.md の復唱、失敗トレースの保持、few-shot の轍の回避——コンテキスト設計と推論経済を 1 つの問題として扱う視点の原点。

### 2. 知識の接続

- RAG（Retrieval-Augmented Generation, 外部知識を検索してプロンプトに与え、それを根拠に生成させる手法）→ [[retrieval-augmented-generation]]
  - 原典（[[summaries/2020-rag]], 2020）はパラメトリック記憶（重み）と非パラメトリック記憶（文書索引）の end-to-end 結合として RAG を定式化し、幻覚の減少・索引差し替えによる知識更新・retrieval collapse を実証した。「知識はパラメータでなく索引に置く」という設計原則の出発点。
  - **GraphRAG**（[[summaries/2024-graphrag]], Microsoft, 2024-04）は、この系譜に「**検索では原理的に答えられない質問がある**」という線を引いた。「このコーパスの主要なテーマは何か」に対して意味的に近いチャンクを top-K 引くことには意味がない——答えがどのチャンクにも書かれていないからで、それは検索でなく **QFS**（Query-Focused Summarization, クエリ焦点型要約）のタスクである。解法は検索の改善ではなく**索引の作り直し**で、LLM にエンティティ知識グラフを作らせ、Leiden でコミュニティ検出して階層分割し（相互排他かつ全体網羅なのでコーパス全体をカバーする）、各コミュニティ要約を事前生成しておいて、質問時は**全件に並列に答えさせて畳む**（map-reduce）。「検索して絞る」から「**全部に問い合わせて畳む**」への転換であり、「検索の質が生成の上限」という制約そのものが外れる。素朴な RAG に網羅性 72〜83% で勝つが、**グラフなしの原文 map-reduce も競争力があり**（優位は 52〜64%）、グラフの真価は品質でなく**トークン単価**（ルート水準は 9〜43 倍安い）——ただしその表にグラフ索引の構築費は入っておらず、**生涯クエリ数で割って考える償却の問題**になる。副産物として、8k / 16k / 32k / 64k の比較で**最小の 8k が一貫して優れた**という実測も残した（→ [[context-engineering]]）。
- MCP（Model Context Protocol, ツールやデータソースをモデルに接続する標準プロトコル）→ [[model-context-protocol]]
  - 横断レビュー（[[summaries/2025-llm-reasoning-to-agents]], 2025）は、エージェントの外部接続を担うプロトコル層を **MCP（Anthropic, 縦＝ツール・データ接続）／A2A（Google, 横＝エージェント間相互運用）／ACP（IBM, ローカル協調）** の三つ巴として整理し、標準化と引き換えに生じるプロトコル層のセキュリティ（なりすまし・権限昇格・ツール説明文経由の注入）を課題として挙げた。「どのフレームワークで書くか」（[[agent-frameworks]]）とは独立に「エージェント同士・外部資源とどう繋ぐか」が設計軸として立ち上がった段階。

### 3. 構成とスケール

- multi-agent systems（複数エージェントの分業・協調、orchestrator-worker 構成）→ [[multi-agent-systems]]
  - Sakana Fugu（[[summaries/2026-sakana-fugu]], 2026）は「どのモデルにどう働かせるか」を学習したオーケストレータで個々のフロンティアモデル単体を超え、**オーケストレーションをモデルスケーリングと直交する新しいスケーリング軸**として実証した。固定集約役の debate/MoA からクエリ適応的なワークフロー生成への世代交代を示す原典。
  - 本番運用の実例は Anthropic Research（[[summaries/2025-multi-agent-research-system]], 2025）: リード＋並列サブエージェント＋CitationAgent の orchestrator-worker で単一エージェント比 +90.2%。効果の正体は「別コンテキストで並列にトークンを費やす容量」（BrowseComp 分散の 80% をトークン量が説明）であり、代償はチャット比 **15 倍**のトークン——適用条件（幅優先・高価値・並列化可能）まで含めた経済学を開示した。
  - Kimi K2.5 の Agent Swarm（[[summaries/2026-kimi-k2.5]], 2026）は、その「いつ・いくつ・どう分けるか」を**プロンプト設計から RL（PARL）へ**移した: 凍結サブエージェント＋補助報酬のアニーリング＋critical steps（最長ブランチ）制約で並列化の意思決定自体を学習し、BrowseComp 78.4%（同一モデル比 +17.8pt。絶対値は発表時点）・実行時間 3〜4.5 倍短縮。フロンティアモデル自体が並列オーケストレーションを内蔵する段階に入った。
- agent frameworks（LangGraph, AutoGen, CrewAI, Claude Agent SDK 等）→ [[agent-frameworks]]
  - 実務の標準語彙は Anthropic「Building Effective Agents」（[[summaries/2024-building-effective-agents]], 2024）が確立: workflow（事前定義コードパス）と agent（動的制御）の区別、5 つの設計パターン、「まず単純に、複雑さは実証されたときだけ」の原則。

### 4. 応用

- coding agents（SWE-agent, Devin, Claude Code, Cursor 等）→ [[coding-agents]]
  - この領域の主戦場が **SWE-bench**（[[summaries/2023-swe-bench]], Princeton・UChicago, ICLR 2024）で、**ベンチマークを人手で作るのでなく既存の開発の営みから収穫する**という型を示した——実際の GitHub イシューと、それを解決したマージ済み PR の対には**問題・解・検証器（PR が追加したテスト）が最初から揃っている**ので、あとはノイズを落とすフィルタを設計すればよい（**93,139 PR → 3 段フィルタ → 2,294 件**）。イシューの解決を測る **fail-to-pass** テストと、既存の振る舞いが壊れていないかを測る **pass-to-pass** テストを分離する設計、**「人間にも解けない問題」の能動的な除外**、**作成年で層別して汚染を検査する**やり方は、コーディング以外のエージェント評価にも転用できる（→ [[agent-evaluation]]）。
  - **同論文が残した最も重要な知見は評価の外側にある。**BM25 の最大文脈を 13k → 50k と広げると**オラクルのファイルに対する recall は 29.58 → 51.06 と上がるのに、解決率は 1.96 → 1.22 と下がる**。逆に**実際に編集された行の ±15 行以外を畳むだけで Claude 2 は 4.8 → 5.9 へ上がる**。**検索指標の改善が成果の改善を意味しない**というこの乖離は、[[context-engineering]] の注意予算という枠組みの、2023 年時点での数量的な裏づけである。
  - 研究側の原典が **SWE-agent**（[[summaries/2024-swe-agent]], Princeton, NeurIPS 2024）で、**ACI（agent-computer interface, エージェント–コンピュータインターフェース）**という概念を立てた。主張は「**LM は、ソフトウェア向けの API・人間向けの UI に続く第 3 のエンドユーザーである**」——人間は不要な情報を柔軟に無視できるが LM にとってはすべての内容が固定のコストを持つ、人間には上下キーの連打が直観的だが LM には冗長で高くつく、といった違いが設計指針を分ける。**モデルの重みを一切変えず、インターフェースだけを設計し直して** SWE-bench の解決率を 3.8% → **12.47%** へ引き上げた。
  - これが単なる言い換えでないことを示すのが同論文のアブレーションで、**Vim / VSCode 流に検索結果を 1 件ずつ送る UI（12.0%）は、検索ツールを一切与えない場合（15.7%）より悪い**——エージェントが全一致を網羅的に舐めて予算とコンテキストを枯渇させるためである。**「人間にとって良い UI」がエージェントには積極的に有害でありうる。**ファイルビューアの窓サイズにも両側の最適点がある（30 行 14.3 / 100 行 18.0 / 全体 12.7）。
  - 転用価値の高い実装は、**範囲指定の置換＋編集後の自動再表示**（確認のターンが消える）、**検索結果 50 件上限と「より具体的なクエリを書け」という返し**、**直近 5 観測より前を 1 行へ折り畳む**（→ [[context-engineering]]）、そして**リンタを通らない編集は差し戻し、エラー・編集後・編集前の 3 点を返す**こと。**編集は 1 回失敗すると回復確率が 90.5% → 57.2% に落ちる**ので、連鎖を断つガードレールが効く。
  - なお 2026 年のフロンティアモデルの評価ハーネスは**逆に最小構成へ収斂**しており（bash ＋ファイル編集の 2 ツール等）、**モデルが強くなるほど足場を薄くできる**という [[summaries/2026-harness-design]] の主題と、ACI の主題（弱いモデルほどインターフェースで底上げできる）は同じ軸の両端にあたる。
  - 初出典は Anthropic の長時間ハーネス記事（[[summaries/2025-effective-harnesses]], 2025）: 数日級の自律コーディングを「initializer が環境の足場（feature list JSON・進捗ログ・init.sh・git）を作り、coding agent が毎セッション 1 機能ずつクリーンに進める」二部構成で解く。引き継ぎは要約でなく検査可能な構造化 artifact——時間方向の分業の実務解。
  - 続編（[[summaries/2026-harness-design]], 2026）は planner/generator/evaluator の 3 エージェント（GAN 着想の生成・採点分離、sprint contract）へ発展させ、同時に**縮小の方法論**——「ハーネスの部品はモデル能力への仮定であり、新モデルごとに 1 部品ずつ外して検証する」——を実録（Opus 4.5→4.6 で context reset・スプリント分解が不要化）。ハーネス設計を「作る技術」から「保守する技術」へ更新した。
  - Meta-Harness（[[summaries/2026-meta-harness]], 2026）はその先——**ハーネス設計の自動化**——を実証: コーディングエージェントを proposer に、過去の全候補のコード・スコア・実行トレースをファイルシステム越しに検分させてハーネスコードを探索させると、テキスト分類（ACE +7.7pt・トークン 1/4）・数学検索（未見 5 モデルへ転移 +4.7pt）・TerminalBench-2（人手ベースライン超え）の 3 領域で人手設計を上回った。鍵は「フィードバックを要約せず生のまま選択的に読ませる」こと。人手の連作（作る→剥がす）に対する「探索で発見する」第三の道であり、コーディングエージェントの成熟が自分自身の実行基盤の改善という再帰を可能にした記録 → [[agent-frameworks]]。
  - 連作 5 本目（[[summaries/2026-managed-agents]], 2026-04）は方向を変え、**ハーネスが入れ替わり続ける前提でその下に何を敷くか**を扱う。OS がハードウェアを process・file に仮想化したのに倣い、エージェントを **session（append-only なイベントログ）/ harness＝brain / sandbox＝hands** の 3 つの差し替え可能なインターフェースへ仮想化する。全部を 1 コンテナに同居させると「ペットを飼う」ことになり、コンテナ死＝session 消失・**障害を局在化できない**・顧客 VPC に繋げない、の 3 症状が出た。分離後は brain も hands も使い捨てにでき（ハーネスの中にクラッシュを生き延びる必要のあるものが無い）、コンテナを必要時のみ確保する構成で **TTFT が p50 −60%・p95 −90%**。認証情報は「スコープを狭める」のでなく **sandbox から到達不能にする**（Git はトークンを remote に配線・MCP は vault＋プロキシ）。**注意: 本記事の「meta-harness」は Meta-Harness 論文と同名だが意味が逆**（下位の共通基盤 vs 上位の最適化器）→ [[agent-frameworks]]。
- computer use / GUI 操作エージェント → [[computer-use-agents]]
  - 初出典は Kimi K2.5（[[summaries/2026-kimi-k2.5]], 2026）: スクリーンショット観測 → pyautogui 操作のループで、汎用マルチモーダルモデルのまま OSWorld-Verified 63.3%（Operator 42.9% 超え・Opus 4.5 の 66.3% に肉薄）・WebArena 58.9%。GUI trajectory を事前学習データに混ぜ、視覚 RL でグラウンディングを鍛える製法まで開示。
- research agents（調査・研究そのものの自動化）→ [[research-agents]]
  - 「リサーチ」は水準の違う 2 つの営みを指す。**知識の集約**（Deep Research 型。既存情報を検索・分析・統合するが新しい実験は設計も実行もしない。実装例が [[summaries/2025-multi-agent-research-system]]）と、**知識の生成**（新しい実験を回して新しい知見を作る）である——この線は [[summaries/2026-ai-scientist]] が自ら引いている。
  - 後者の到達点が **The AI Scientist**（[[summaries/2026-ai-scientist]], Sakana AI ほか, 2026-06）で、着想 → 文献調査 → 実験の設計と実行 → 作図 → 分析 → 論文執筆 → 査読までを人間の介入なしに通し、**生成論文 1 本が ICLR 2025 のワークショップで実際の査読を通った**（6.33、全 43 本中上位 45%。プロトコルどおり査読後に全件取り下げ）。設計として移植価値が高いのは、**段階ごとに観測可能な停止条件を書き下すこと**（「実行時エラーなしに走るコードが出た」「LLM 自身が定義した指標でベースラインを超えた」「探索予算が尽きた」）と、**replication / aggregation という「統計を取るための型」を探索木に組み込むこと**——エラーバーを付けるという作法をプロンプトの努力目標でなく構造にしている。品質は**探索ノード予算とベースモデルの世代の双方**に対して単調に伸びる（→ [[test-time-compute]]）。
  - **ただし結果の読み方には条件が 3 つ付く**: (1) ワークショップの採択率は **70%**（ICLR 本会議は 32%）、(2) **人手の選別漏斗**が入っている（採択論文は生成された 24 本の原稿から選ばれた 1 本。応用系は 136 個のアイデアから 1 つ）、(3) **著者ら自身の Automated Reviewer は既定のプロンプトでは 3 本とも不採択と判定した**——判定系は評価対象の分布ごとに校正し直さねばならず、**校正は移植できない**。著者ら自身がこれらを明記しており、**「自動生成物を人が選別したなら漏斗の数字を必ず併記する」**という報告の規範としても読める。
  - 評価の難所は「良い研究か」を測る物差しがないことで、そこは査読という人間の制度を借りるしかない。その **Automated Reviewer** の検証設計——制度の accept/reject を正解に使う・カットオフ前後で汚染を検査する・**自明ベースライン（Always Reject は精度 0.65 だが F1 0.00）を併記する**・比較の不完全性を自己申告する——は [[agent-evaluation]] 側に整理した。
  - 安全性の実測記録としても重要で、サンドボックスがほぼない状態では**自分を再起動してプロセスを増やす・記憶容量を 1TB 近く食い潰す・実験が時間制限を超えたときに実行時間でなく制限のほうを書き換えようとする**という挙動が出た。制約の回避は悪意でなく**目的達成の副作用**として現れる（→ [[agent-safety-and-guardrails]]）。
- web agents（ブラウザ操作・情報収集）→ `[[web-agents]]`

### 5. 評価・運用・安全性

- agent evaluation（SWE-bench, GAIA, WebArena, τ-bench 等のベンチマークと、解決率・コスト・ステップ数といった指標）→ [[agent-evaluation]]
  - 手前にある問いの起点が [[summaries/2022-rlhf-illustrated]]（2022）: 「良さ」を捉える損失関数は書けず、BLEU / ROUGE は参照文との一致しか測れない——だから人間の選好を測る。**Arena の Elo も LLM-as-a-judge も、この系譜が RLHF の報酬モデル訓練で作った道具（ペアワイズ比較・順位づけ・人間代行モデル）を評価側に転用したもの**である。同記事はそこで得られた 2 つの教訓——絶対スコアは較正が壊れるので相対比較を使う／アノテータは不一致で正解が存在しない——も記録しており、Cohen's κ によるジャッジ検証の規律の出所になっている。
  - **LLM ジャッジの検証手続き**の参照実装は [[summaries/2023-dpo]]（2023）§6.4: ジャッジと人間の一致率は絶対値で読めず、**人間同士の一致率を基準線に置いて比べる**（難しい対戦では人間同士 65%・GPT-4 と人間 67〜70%、易しい対戦ではそれぞれ 87%・85〜86%）。またジャッジのプロンプトの文言が勝率を動かす（素朴な版では GPT-4 が人間より長く反復的な要約を好み、"concise" を明示した版で勝率が 47%→54% 変化）。「選好の代理として妥当」は「事実性の判定者として妥当」を意味しない（同論文は GPT-4 が算術を誤判定した例を自ら収録している）。
  - MASFT（[[summaries/2025-masft]], 2025）はスコアでなく**トレースを一次データとする失敗分析**の方法論（Grounded Theory・Cohen's κ・LLM-as-a-judge）を確立し、「MAS の失敗は個々の LLM でなく組織設計の欠陥」という診断を与えた。
  - 実務側の評価・運用の教訓は [[summaries/2025-multi-agent-research-system]]（2025）: 約 20 クエリの小規模評価から直ちに始める、LLM-as-a-judge は単一プロンプト・単一呼び出しが最も人間と整合、人間のテスターだけが情報源選択バイアス（SEO ファーム優先）を発見、状態変更エージェントは終了状態評価。運用面ではエラー地点からの再開・rainbow deployment・会話内容を見ないトレーシングを記録。
- agent safety and guardrails（prompt injection（外部入力に埋め込まれた指示でエージェントを乗っ取る攻撃）、権限設計、sandboxing、HITL）→ [[agent-safety-and-guardrails]]
  - 外付けの対策を考える前に、**モデル内在の「安全そうな振る舞い」がどこから来ているか**を押さえる。一次資料は InstructGPT（[[summaries/2022-instructgpt]], 2022）で、同論文 §5.2「Who are we aligning to?」は整合の宛先を 4 層——訓練ラベラー（約 40 名・互いに 27% 意見が食い違う・東南アジア系 52.6%・18〜34 歳 73.7%）／指示を書いた研究者と組織／API の顧客／その顧客層の偏り（**待機リストの最初の種は OpenAI 従業員**）——に名指しで分解し、「**すべての人の選好に同時に整合したシステムを訓練することは不可能である**」と結論した。含意は実務的で、想定利用者がこの小集団から離れるほど、内在の安全性への依存を下げて外付けの 4 層に寄せる必要がある。
  - 攻撃側の脅威分類は [[summaries/2024-llm-security-privacy-survey]]（Das et al., 2024）が体系化: セキュリティ攻撃（prompt injection の goal hijacking/prompt leaking・間接インジェクション、jailbreak の DAN と 2 失敗モード＝competing objectives/mismatched generalization、backdoor、data poisoning）とプライバシー攻撃（勾配漏洩・MIA・記憶による PII 漏洩）、および各防御（SmoothLLM・self-reminder・DP-SGD・PII スクラビング等）。防御はほぼ必ずモデル有用性とのトレードオフを抱える。
  - 監視面の実測が [[summaries/2025-cot-faithfulness]]（Anthropic, 2025）: 推論モデルの CoT が実際の判断理由（ヒント）を明かす率は平均 25〜39%、RL で仕込んだ reward hack は >99% 悪用されながら言語化 <2%。**CoT モニタリングは「気づく層」としては有望だが「排除の保証」には使えない**という運用原則を確立した。
- agent observability（trajectory のトレーシングとデバッグ、指標の集約、監査証跡）→ [[agent-observability]]
  - 概念ページは 2026-08-01 に新設。評価が「開発時に良し悪しを測る」のに対し、可観測性は「本番稼働中に何が起きているかを見る」——非決定的で・エラーが複利で効き・静かに失敗するエージェントでは、行動ログ／指標／改変不可能な監査証跡の 3 層が要る。トレース中の thought は行動の理由とは限らないため（CoT 忠実性 25〜39%）、**思考の記述より行動と観測の系列を一次データとして扱う**のが規律になる。
  - エンタープライズ運用の語彙（デプロイ形態 6 パターン・暴走コスト上限・承認ゲート・policy-as-code）は [[summaries/2026-agent-orchestration-guide]]（Databricks, 2026）。ただしベンダーブログであり、提示される統計値には出典がない。

### 6. 土台となる LLM 側

- transformer architecture と個別モデルの世代 → [[transformer-architecture]]
  - 系譜の俯瞰は [[summaries/2026-gpt2-to-kimi3]]（2026）: GPT-2 → Kimi K3 の **22,580 倍**を「固定容量の連想メモリには追い出しポリシーが要る」という一本の線で読む——KV cache の O(N) 成長 → linear attention の固定状態（干渉）→ delta rule の選択的上書き → ゲート減衰 → KDA/MLA ハイブリッド＋MoE。スケーリングは「容量をどこに足すか」の設計である。
  - この系譜の**手前 2 本**（DeepSeek-LLM 2024-01 / DeepSeek-V2 2024-06）は、二次解説 [[summaries/2025-deepseek-series]] 経由で入っている。前者は **scaling law の測り方**——規模をパラメータ数でなく**非埋め込み FLOPs/token** で測ると $C = M \times D$ という素直な形に揃う（埋め込み層はパラメータは大きいがトークンあたりの計算にほぼ寄与しないので、パラメータ数で測ると語彙サイズの違いが「規模」に化ける）、そして**データ品質が最適比率を動かす**（scaling law は普遍定数ではない）——を示した。後者は **MLA と DeepSeekMoE の初出**で、KV cache 93.3% 削減・生成スループット 5.76 倍を達成し、**device-limited routing**（ルーティングの自由度を通信予算から逆算して削る）と、計算・配置・通信の 3 層に分けた均衡損失を導入した。この 4 本を貫く方法論が **HPC co-design**（モデルのアーキテクチャと計算基盤を切り離さずに一緒に設計すること）で、制約の厳しいハードウェア（H800）の側から出てきた設計論として読める。**ただし同記事の V3 の MLA に関する記述 3 点は原典に存在しない**（要約に照合節あり）——一次資料を持った状態で二次資料を読むことの効用と、二次資料をそのまま概念ページの根拠にしない運用の必要性を、同時に示す事例になっている。
  - 本番規模での MoE 設計の一次資料が **DeepSeek-V3**（[[summaries/2024-deepseek-v3]], 2024-12）。671B 総/37B 活性を **278.8 万 H800 GPU 時間（$5.576M）・回復不能な loss spike ゼロ**で訓練しきり、3 つの貢献を残した: **補助損失を使わない負荷分散**（エキスパートごとのバイアス項をルーティング判定にのみ使い勾配経路を汚さない。ただし §4.5.3 の分析によれば本質は「均衡の粒度を系列単位からバッチ単位へ緩めたこと」で、バッチ単位の補助損失も同等の性能に達する——**「負荷を均す」と「専門化させる」の緊張を、均衡を要求する粒度で調整する**）、**MTP**（マルチトークン予測。推論時は破棄でき、投機的デコードに転用すれば TPS 1.8 倍）、そして**極大規模での FP8 訓練の初の検証**（BF16 比の相対 loss 誤差 0.25% 未満。ただし活性の勾配をブロック単位で量子化すると発散する、という否定的結果つき）。R1 のベースであり、同時に**その R1 から推論能力を蒸留している**という相互参照の関係にある。
  - **MoE は 2026-08-02 に独立した概念ページ [[mixture-of-experts]] へ切り出した**。アーキテクチャの一部としてだけでなく、継続学習・メタ学習・マルチタスク学習・強化学習でも使われる汎用の道具だからである。その全体像を与えるのが [[summaries/2025-moe-survey]]（Mu & Lin, 2025-03）で、wiki に無かった 2 領域を持ち込んだ: **ゲーティング関数の設計空間**（softmax top-k だけでなくコサインルータ・指数型分布族・離散割り当てをやめて token dropping を原理的に消す Soft MoE）とルーティング水準（トークン／モダリティ／**タスク**——タスク水準なら推論時に関連エキスパートだけ読めばよく通信とメモリが減る）、そして**理論**（softmax ゲート＋線形エキスパートの普遍近似定理、MLE 収束を Voronoi 損失で特徴づける系譜、深層では「2 層 CNN の単一エキスパートは 87.5% を超えられないが非線形 MoE は超える」「十分な探索があればルータはデータのクラスタ構造を自動的に学習する」）。ただし深層の理論は玩具設定であり、システム記述は DeepSeek-V3 で止まっていて wiki の他原典より古い。
  - MoE の基礎は [[summaries/2023-moe-explained]]（Hugging Face, 2023）: 疎な MoE 層＋ルータの仕組み、1991 年からの系譜（Shazeer → GShard → Switch → Mixtral）、負荷分散の 3 点セット（aux loss・z-loss・expert capacity）、「メモリは総パラメータ・計算は活性化分」という MoE 経済の本質。K2/K3・DeepSeek 世代の MoE 採用を理解する土台。その中心にある一次資料が [[summaries/2021-switch-transformers]]（2021）——top-1 ルーティング・selective precision・蒸留 30%・初の 1.6T モデルと、data/model/expert 並列の体系化の原典。
- post-training（RLHF, RLVR 等）→ [[reinforcement-learning-from-human-feedback]]（概念ページは**時系列構成**で「何が何を置き換えたか」を追う）
  - 系譜の起点は古典的 RLHF。一次資料は **InstructGPT**（[[summaries/2022-instructgpt]], OpenAI, 2022）で、ChatGPT の直接の前身にあたる: SFT（人間が模範応答を書き下ろす）→ 報酬モデル（人間が $K=4$〜9 個の出力を順位づけ）→ PPO の 3 段で GPT-3 を仕上げ、**13 億パラメータの InstructGPT が 1750 億の GPT-3 より人間に好まれる**（パラメータ 100 分の 1）ことを示した。**整合の計算コストは事前学習の約 1.6%**（60 対 3,640 ペタフロップス/s-日）で、しかも「モデルサイズを 100 倍にするより有用性の改善が大きい」——**後段への投資がスケールに勝ちうる**という構図の最初の定量的な提示である。同時に **alignment tax（アラインメント税, 整合させると他タスクの性能が落ちる代償。高いと「整合させないほうが得」という誘因が生まれる）** という語を導入し、事前学習分布の勾配を混ぜる PPO-ptx で緩和した。ただし改善は一様ではなく、**バイアスは改善せず、有害な出力を出すよう指示されれば GPT-3 より有害になる**（訓練ラベルでは helpfulness を優先し、評価ラベルでは逆にしたという報酬設計の帰結）→ [[agent-safety-and-guardrails]]。
  - 一般向けの標準的な解説が [[summaries/2022-rlhf-illustrated]]（Hugging Face, 2022-12）: 「良さを測る損失関数が書けない」ので、人間の**比較**（絶対スコアではなく順位・Elo）から報酬モデルを学習する、という発想を図解する。報酬に KL ペナルティを課す理由が「出鱈目テキストで報酬モデルを騙し始めるから」と説明されており、**reward hacking の構造がこの時点で認識されていた**。この記事は 3 年半前のもので周辺はほぼ入れ替わっているが（PPO→GRPO、人間の選好→検証器、TRL 以外のライブラリは停止）、骨格——比較から報酬を作る／スカラー化／乖離ペナルティ／絶対評価より相対比較／アラインメント税という問題設定——は現在まで生き残っている。
  - この 3 段パイプラインの重さへの回答が 3 通り出る。最初が **DPO**（[[summaries/2023-dpo]], Stanford, 2023）で、**報酬モデルの訓練と RL ループを両方消す**: KL 制約付き報酬最大化の最適解を逆に解くと報酬が方策で書け（$r=\beta\log\frac{\pi}{\pi_{\text{ref}}}+\beta\log Z$）、Bradley-Terry が報酬の**差**にしか依存しないので計算不能な分配関数が相殺される。残るのは選好データ上の二値交差エントロピー 1 本で、実装は 10 行。報酬-KL フロンティアは PPO を厳密に支配し、**PPO が真の報酬を持つ場合ですら上回る**。ただし消えたサンプリングループは、目的が「既存の選好への適合」でなく「新しい挙動の発見」に移ると必要になる——年表で DPO の後に RLVR が来る理由。
  - GRPO の起点は DeepSeekMath（[[summaries/2024-deepseekmath]], 2024）: PPO の critic をグループ相対 advantage で置き換えて発明し、「全事後学習手法＝データソース×報酬×勾配係数」の統一パラダイムと「RL は Maj@K を上げるが Pass@K を上げない（分布の尖鋭化）」という基礎観察を残した。
  - DeepSeek-R1（[[summaries/2025-deepseek-r1]], 2025）は、その GRPO を検証可能な報酬だけの大規模 RL（RLVR）で回し、長い思考・自己検証・reflection が**創発**することを公開実証して o1 型推論モデルの製法を開いた。GRPO は [[summaries/2026-sakana-fugu]] のオーケストレータ訓練にも使われ、モデルとエージェントの両方を貫く訓練レシピになっている。
  - 後継の DeepSeek-V4（[[summaries/2026-deepseek-v4]], 2026）は事後学習の主役を **On-Policy Distillation（OPD）** に置換: ドメイン専門家を RL で個別に鍛え、学生の自己生成軌跡上の全語彙逆 KL 蒸留で単一モデルへ統合する——「発見は RL・統合は蒸留」の分業がパイプライン設計の原理に昇格した。
- test-time compute（推論時に計算量を増やして精度を上げる考え方）→ [[test-time-compute]]
  - 体系化は Long CoT サーベイ（[[summaries/2025-long-cot-survey]], 2025, 813 文献）: 推論モデルの思考を deep reasoning・extensive exploration・feasible reflection の**3 特性の統合（Long CoT）**として定義し、垂直/並列スケーリングの 2 型、**推論境界**と **overthinking**（長考は閾値を超えると性能が落ちる）、PRM vs ORM、aha moment への反証までを整理した推論モデル時代の地図。
- 推論の高速化・サービング（KV cache, バッチング, コストとレイテンシ）→ [[llm-inference-optimization]]
  - prefill/decode の 2 相・メモリ帯域律速・カーネル融合の基礎は [[summaries/2026-gpt2-to-kimi3]] が実装レベルで解説（固定状態化でデコード 6 倍、FLOPs 最小 ≠ wall-clock 最小、融合カーネルなしの新活性化は 3 倍遅）。エージェントのトークン経済（[[summaries/2025-multi-agent-research-system]] の 15 倍）の物理的な下部構造。
  - 実務側のレバーと運用の型は [[summaries/2026-llm-optimization-guide]]（Mirantis, 2026）: 量子化 −75%・continuous batching で稼働率 40→90%・PagedAttention −55% の定量カタログと、ワークロードのクラス分け・縮退設計・構成のコード化という運用規律。
  - **エッジ／オンデバイス側の設計**は Gemma 4（[[summaries/2026-gemma-4]], 2026）が参照実装: KV cache −37.5%（local:global 5:1・values=keys・p-RoPE）、QAT（int2/4 で E2B 4.6→0.8GB）、投機的デコード用 MTP drafter の同梱、encoder-free 化（12B は視覚・音声エンコーダを射影に置換）。フロンティア・スケール（K2.5 の 1T MoE）と対をなす「小さく速く配る」路線で、Arena 人間評価では 31B dense が 744B〜1.6T MoE 群に伍した。
  - **1M コンテキストの効率化**は DeepSeek-V4（[[summaries/2026-deepseek-v4]], 2026）が最初の大規模実証: KV の学習圧縮＋スパース選択（CSA/HCA）で 1M 時の KV cache を V3.2 比 10%・一般的な BF16 GQA8 比約 2% に削り、100 万トークンを「日常運用」の水準にした。on-disk KV cache（shared-prefix 再利用）・batch-invariant 決定論カーネル・agentic search の同一モデル実測（精度優位・コスト僅増）まで含め、長ホライズンのエージェントタスクの物理的基盤を更新している。
- エージェント特化の基盤モデル訓練（データ合成＋RL でエージェント能力を作る）
  - [[summaries/2025-kimi-k2]]（Moonshot, 2025）: 1.04T MoE を MuonClip で loss spike ゼロ事前学習し、**実 MCP ツール 3000+＋合成ツール 20,000+ による tool-use trajectory 合成**と、検証可能報酬＋自己批評ルーブリックの joint RL でエージェント能力を仕込む。**非思考のまま** SWE-bench Verified 65.8・τ²-Bench 66.1——「長考の創発」（R1）と対をなす「非思考のエージェント化」の代表原典。
  - 後継の Kimi K2.5（[[summaries/2026-kimi-k2.5]], 2026）は**マルチモーダル化**の製法を追加: 15T トークンの joint 事前学習は early fusion・低 vision 比率が最良、視覚エージェント能力の発火はテキストのみの SFT で足りる（zero-vision SFT）、視覚 RL はテキスト性能まで上げる（双方向転移）。MoonViT-3D（画像・動画共有エンコーダ）・DEP（視覚エンコーダの並列化分離）・Toggle（トークン効率 RL, −25〜30%）まで含め、「エージェント基盤モデルは単一モダリティでは作らない」方向を示した。
- ファインチューニング（LoRA 等の PEFT）→ `[[parameter-efficient-fine-tuning]]`

## 現状のカバレッジ

| 軸 | 取り込み済みの原典 |
| --- | --- |
| 基本構造 | [[summaries/2023-llm-agents-survey]]（総論: brain/perception/action・応用 3 分類・エージェント社会）、[[summaries/2022-chain-of-thought]]（推論の創発・CoT）、[[summaries/2022-react]]（agent loop・推論と行動の統合・初期のツール利用）、[[summaries/2023-toolformer]]（自己教師ありツール利用学習・重みへの埋め込み）、[[summaries/2023-reflexion]]（自己反省・試行間学習）、[[summaries/2023-memgpt]]（階層記憶・仮想コンテキスト管理・イベント駆動制御）、[[summaries/2025-a-mem]]（動的記憶組織化・記憶進化）、[[summaries/2025-manus-context-engineering]]（本番のコンテキスト設計 6 原則・KV cache 経済）、[[summaries/2025-effective-context-engineering]]（コンテキストエンジニアリングの命名と定式化・注意予算と context rot・JIT 取得・長時間タスクの 3 手法） |
| 知識の接続 | [[summaries/2020-rag]]（検索拡張生成・非パラメトリック記憶・hot-swap） |
| 構成とスケール | [[summaries/2026-sakana-fugu]]（学習されたオーケストレータ）、[[summaries/2025-masft]]（MAS の失敗分類）、[[summaries/2024-building-effective-agents]]（設計パターンとフレームワーク観）、[[summaries/2025-multi-agent-research-system]]（本番 orchestrator-worker・トークン経済学）、[[summaries/2026-kimi-k2.5]]（RL で学習された並列オーケストレーション・context sharding） |
| 応用 | [[summaries/2026-kimi-k2.5]]（computer use: OSWorld / WebArena・GUI エージェント構成）、[[summaries/2025-effective-harnesses]]（coding agents: 長時間自律コーディングのハーネス）、[[summaries/2026-harness-design]]（coding agents: 3 エージェント構成とハーネス縮小）、[[summaries/2026-meta-harness]]（coding agents / frameworks: ハーネスの自動探索・TerminalBench-2）、[[summaries/2026-managed-agents]]（frameworks: ランタイム分離・session/brain/hands・認証情報の到達不能化）。[[summaries/2026-sakana-fugu]] もコーディング・自律研究・CAD 等の応用例に言及 |
| 評価・運用・安全性 | [[summaries/2022-instructgpt]]（整合の宛先の 4 層分解・helpful 優先の報酬設計とその帰結・「公開ベンチマークは実利用を反映しない」）、[[summaries/2022-rlhf-illustrated]]（評価の原点: 自動指標の限界・人間の選好と Elo・HHH 基準と安全性の事後注入）、[[summaries/2025-masft]]（トレース分析・LLM-as-a-judge・失敗分類）、[[summaries/2025-multi-agent-research-system]]（小規模評価・単一ジャッジ・終了状態評価・本番運用の信頼性）、[[summaries/2025-cot-faithfulness]]（CoT 忠実性・CoT モニタリングの限界・安全性）、[[summaries/2024-llm-security-privacy-survey]]（セキュリティ・プライバシー攻撃と防御の脅威分類）、[[summaries/2026-agent-orchestration-guide]]（エンタープライズ運用・デプロイ形態 6 パターン・ガバナンスと可観測性の実務層。ベンダーブログ・統計値は出典なし）。ベンチマークは [[summaries/2026-sakana-fugu]]（SWE-Bench Pro / Terminal Bench / GPQA / HLE / τ³ 等）、[[summaries/2022-react]]（HotpotQA / FEVER / ALFWorld / WebShop・HITL 介入）も言及 |
| LLM 基盤 | [[summaries/2025-deepseek-series]]（DeepSeek 4 本の二次解説: scaling law の測り方・MLA/DeepSeekMoE の初出・HPC 協調設計）、[[summaries/2024-deepseek-v3]]（本番規模 MoE の一次資料: 補助損失なし負荷分散・MLA・MTP・FP8 訓練・DualPipe・訓練コスト $5.576M）、[[summaries/2025-moe-survey]]（MoE の基本設計・学習パラダイム別アルゴリズム・理論・応用の総覧）、[[summaries/2022-instructgpt]]（RLHF の一次資料: SFT＋報酬モデル＋PPO・PPO-ptx とアラインメント税・整合コストは事前学習の 1.6%）、[[summaries/2023-dpo]]（報酬モデルと RL ループの排除・同値クラスと定理 1・PPO の分散の診断）、[[summaries/2022-rlhf-illustrated]]（同パイプラインの一般向け解説: 順位づけと Elo・KL ペナルティ）、[[summaries/2024-deepseekmath]]（GRPO の発明・統一パラダイム・数学コーパス採掘）、[[summaries/2025-deepseek-r1]]（RLVR・GRPO・蒸留・推論の創発）、[[summaries/2025-long-cot-survey]]（Long CoT の体系化・test-time scaling・推論境界）、[[summaries/2026-gpt2-to-kimi3]]（アーキテクチャの系譜・KV cache・推論効率）、[[summaries/2026-llm-optimization-guide]]（本番推論最適化の実務・サービング運用）、[[summaries/2025-kimi-k2]]（エージェント特化の事前学習＋データ合成＋joint RL）、[[summaries/2023-moe-explained]]（MoE の仕組み・歴史・負荷分散）、[[summaries/2026-kimi-k2.5]]（マルチモーダル joint 訓練・zero-vision SFT・トークン効率 RL）、[[summaries/2026-gemma-4]]（エッジ〜31B の効率化設計・encoder-free・QAT・投機的デコード）、[[summaries/2026-deepseek-v4]]（1M コンテキスト効率・CSA/HCA・mHC・OPD・RL/rollout インフラ）。[[summaries/2026-sakana-fugu]] も SFT・進化戦略・GRPO の訓練レシピに言及 |

6 軸すべてに少なくとも 1 件の原典が入った（2026-07-26 時点）。「応用」軸には 2026-07-28 の Kimi K2.5 で初の専用記述（computer use）が入った。「知識の接続」軸のプロトコル層は 2026-08-01 の横断レビュー（[[summaries/2025-llm-reasoning-to-agents]]）で概念ページ [[model-context-protocol]] を新設し、MCP/A2A/ACP の三つ巴として初めて記述が入った——ただし各プロトコルの一次仕様（MCP spec 等）はまだ ingest しておらず、次に読むべき docs 候補として残る。「評価・運用・安全性」軸では 2026-08-01 に概念ページ [[agent-observability]] を新設し、既存原典に散在していた可観測性の記述（本番運用報告・トレース分析方法論・CoT 忠実性の限界）を集約した——ただし可観測性を主題とする一次資料（OpenTelemetry の生成 AI 向け規約、トレーシング基盤の設計文書、本番インシデントの事後報告など）はまだ 1 件も入っていない。以後は各軸の深化（プロトコル一次仕様、可観測性の一次資料、応用における coding agents の専用原典）を `lint` のデータギャップとして追う。

## 関連ページ

- [[index]] — 全ページのカタログ
- [[log]] — 取り込み・更新の時系列ログ
