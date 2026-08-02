---
type: translation
source_path: raw/papers/NeMo Guardrails_ A Toolkit for Controllable and Safe LLM Applications with Programmable Rails.md
source_page: "[[summaries/2023-nemo-guardrails]]"
original_language: en
translated_to: ja
translated_at: 2026-08-03
---

# NeMo Guardrails: プログラマブルなレールによる制御可能で安全な LLM アプリケーションのためのツールキット

> 原題: NeMo Guardrails: A Toolkit for Controllable and Safe LLM Applications with Programmable Rails
> 著者: Traian Rebedea\*, Razvan Dinu\*, Makesh Sreedhar, Christopher Parisien, Jonathan Cohen（NVIDIA。\* 同等の貢献）
> 出典: arXiv:2310.10501（2023-10 / ar5iv 版）・EMNLP 2023 System Demonstrations
> コード: [https://github.com/NVIDIA/NeMo-Guardrails](https://github.com/NVIDIA/NeMo-Guardrails)

> **訳注（クリップの状態と復元）**
> - 底本は ar5iv 版の Web Clipper クリップ。**本文・見出し・図表はすべて完全**であった（Figure 1〜11、Table 1〜3 が揃っている。画像も 10 種すべて取得できた。Figure 4 と Figure 7 は原典自体が同一の画像を再掲している）。
> - **脚注 11 件の本文が欠落**していた（`<sup>1</sup>`〜`<sup>11</sup>` のマーカーのみ残存）。原ページから復元して該当箇所に訳注として挿入した。いずれもリポジトリまたはドキュメントの URL である。
> - 参考文献一覧（References）は既定どおり訳出しない。付録 A〜G はすべて訳出した。
> - **プロンプトテンプレートと Python のアクション定義、Colang のコードは英語原文のまま収録**している（一字一句が挙動に効くため、翻訳すると再現性が失われる）。
> - 図はすべて `raw/assets/2023-nemo-guardrails/` にローカル保存し、そのパスを参照している。

---

###### Abstract（要旨）

**NeMo Guardrails** は、LLM ベースの会話システムに**プログラマブルなガードレール**を容易に追加するためのオープンソースのツールキット<sup>1</sup>である。ガードレール（略して**レール**）は LLM の出力を制御する特定の方法であり、たとえば有害とみなされる話題について話さない、あらかじめ定められた対話の経路をたどる、特定の言語スタイルを用いる、などがある。LLM の提供者と開発者が、**訓練時に特定のモデルに埋め込まれる**ガードレールを追加できる機構はいくつか存在する（例: モデルアラインメントを用いるもの）。それとは異なり、NeMo Guardrails は対話管理（dialogue management）に着想を得たランタイムを用いて、開発者が LLM アプリケーションに**プログラマブルなレール**を追加できるようにする——これらはユーザーが定義するもので、基盤となる LLM から独立しており、解釈可能である。我々の初期の結果は、提案するアプローチが複数の LLM 提供者とともに用いられ、プログラマブルなレールを使って制御可能で安全な LLM アプリケーションを開発できることを示している。

> 訳注（脚注 1、原ページより復元）: [https://github.com/NVIDIA/NeMo-Guardrails](https://github.com/NVIDIA/NeMo-Guardrails)。なお本文中の `*` は「同等の貢献」を示す脚注である。

## 1 Introduction（はじめに）

**操縦可能性（steerability）と信頼性**は、大規模言語モデル（LLM）を本番環境に配備するうえで鍵となる要素である。これらのモデルが会話の複数ターンにわたって軌道を保てるようにすることは、タスク指向の対話システムを開発するうえで不可欠である。これは深刻な課題に見える。LLM は容易に話題から逸れさせられうる [^19] からである。同時に LLM は、事実として誤っている、あるいは完全に捏造された応答（**幻覚, hallucination**）を生成する傾向もある [^16] [^21] [^1]。加えて LLM は**プロンプトインジェクション（またはジェイルブレイク）攻撃**に対して脆弱であり、悪意ある行為者が入力を操作してモデルを騙し、有害な出力を生成させる [^12] [^29] [^34]。

信頼でき制御可能な会話システムを構築することは、顧客と接する状況に LLM を配備するうえで極めて重要である。NeMo Guardrails は、LLM ベースのアプリケーションにプログラマブルなレールを容易に追加するためのオープンソースのツールキットである。ガードレール（レール）は、LLM の出力を人間が課したなんらかの制約——たとえば有害な話題に関与しない、あらかじめ定められた対話の経路をたどる、一部のユーザーの要求に特定の応答を追加する、特定の言語スタイルを用いる、構造化データを抽出する——に従わせるよう制御する機構を提供する。さまざまな種類のレールを実装するには、訓練時のモデルアラインメント、プロンプトエンジニアリングと chain-of-thought（CoT）、対話マネージャの追加といった、いくつかの技法が使える。モデルアラインメントが訓練時に LLM へ埋め込まれる一般的なレールを提供し、プロンプトチューニングがカスタマイズされたモデルへ埋め込まれるユーザー固有のレールを提供できるのに対し、**NeMo Guardrails はユーザーが実行時にカスタムのプログラマブルなレールを定義できるようにする**（図 1 参照）。この機構はアラインメント戦略から独立しており、埋め込まれたレールを補完し、異なる LLM とともに動作し、カスタムのモデリング言語である **Colang** を使って定義される解釈可能なレールを提供する。

<figure>

![](../../raw/assets/2023-nemo-guardrails/programmable-rails-new.png)

<figcaption>図1: LLM に対するプログラマブルなレールと埋め込まれたレール。（訳注: 上から「Users / Applications」、NeMo Guardrails の枠内に「Programmable Rails — Guiding and Safeguarding」、その下に「Embedded Rails: User-specific」を持つ「Customized Model（知識: ユーザー固有のスキル、目標: ユーザー定義）」、さらに下に「Embedded Rails: General guardrails」を持つ「Foundation Model（知識: 一般、目標: 有用であること）」が積まれている。）</figcaption>
</figure>

LLM のためのユーザー定義のプログラマブルなレールを実装するために、我々のツールキットは**ユーザーと LLM の間のプロキシのように振る舞うプログラマブルなランタイムエンジン**を用いる。このアプローチはモデルアラインメントを補完するものであり、LLM がユーザーとのやりとりにおいて従うべき規則を定義する。したがって Guardrails のランタイムは**対話マネージャの役割**を持ち、プログラマブルなレールを定義する規則を解釈し強制できる。これらの規則は Colang というモデリング言語を使って表現される。より具体的には、Colang は **LLM が常に従うべき対話フロー（dialogue flows）** として規則を定義するのに用いられる（図 2 参照）。文脈内学習を用いたプロンプティング技法と特定の形の CoT を用いることで、我々は LLM が会話を導く次のステップを生成できるようにする。Colang はその後、対話マネージャによって解釈され、ユーザーがあらかじめ定義した、あるいは LLM が自動生成したガードレールの規則を適用して LLM の挙動を導く。

NeMo Guardrails はいかなる LLM ベースのアプリケーションにも安全性と操縦可能性を追加するのに使えるが、我々は **LLM に駆動される対話システムが Colang と Guardrails ランタイムを使うことから最も恩恵を受ける**と考えている。ツールキットは Apache 2.0 でライセンスされており、我々は複数の LLM 提供者への初期的な対応を、スターターとなる例示アプリケーションと評価ツールとともに提供している。

## 2 Related Work（関連研究）

### 2.1 Model Alignment（モデルアラインメント）

LLM にレールを追加する既存の解決策は、**指示チューニング** [^30] や**強化学習** [^18] [^8] [^17] といったモデルアラインメントの技法に大きく依拠している。LLM のアラインメントはいくつかの次元で働き、主として有用性（helpfulness）を改善し有害性（harmfulness）を減らす。レッドチーミング [^22] を含むアラインメント一般は、**特定の基準（例: 無害性）に従って手作業でラベル付けされた入力プロンプトと応答の大規模なコレクション**を必要とする。

モデルアラインメントは、**訓練時に LLM に埋め込まれ、ユーザーが実行時に容易に変更できないレール**を提供する。さらに、LLM に組み込まれるレールごとに、人間が注釈を付けた応答評価の大きな集合も必要とする。人間のフィードバックからの強化学習 [^18] がモデルアラインメントの最も一般的な手法だが、AI フィードバックからの RL [^3] のような代替手法は人間がラベル付けしたデータセットを必要とせず、実際の LLM を用いて各応答へのフィードバックを提供する。

ほとんどのアラインメント手法が一般的な埋め込みレールを提供するのに対し、同様の方法で開発者はプロンプトチューニング [^13] [^14] を通じてアプリ固有の埋め込みレールを LLM に追加できる。

<figure>

![](../../raw/assets/2023-nemo-guardrails/colang-flows-v2.png)

<figcaption>図2: Colang で定義された対話フロー: 単純な挨拶のフローと、数学および距離に関する問い合わせに応答するためにカスタムアクション wolfram alpha request を呼ぶ 2 つの topical rail のフロー。</figcaption>
</figure>

### 2.2 Prompting and Chain-of-Thought（プロンプティングと chain-of-thought）

LLM にユーザー定義のプログラマブルなレールを追加する最も一般的なアプローチは、**プロンプティング**——プロンプトエンジニアリングと文脈内学習 [^6] を含む——を用いて、ユーザー入力の前または後に特定のテキストを付加すること [^28] [^23] である。このテキストが、LLM が遵守すべき挙動を指定する。

LLM に実行時のユーザー定義レールを与えるもう 1 つのアプローチは **chain-of-thought（CoT）** [^31] を用いることである。最も単純な形では、CoT はユーザーの指示に、当該タスクに対する入出力の対の類似例を 1 つないし複数付加する。これらの例のそれぞれは、最終的な答えを決定するのに有用な、より詳細な説明を出力に含む。より複雑な他のアプローチは、一般から特殊へと LLM に複数段階でプロンプトするもの [^33] や、内なる独白（inner monologue）に似た異なる役割を持つ対話全体を使うもの [^9] を含む。

### 2.3 Task-Oriented Dialogue Agents（タスク指向の対話エージェント）

タスク指向の対話エージェントの構築は一般に 2 つの構成要素を必要とする: **自然言語理解（NLU）** と**対話管理（DM）** のエンジンである [^5] [^15]。NLU と DM の双方について広範なツールと解決策が存在し、Rasa [^5] のようなオープンソースの解決策から、Microsoft LUIS や Google DialogFlow のようなプロプライエタリなプラットフォーム [^15] まである。その機能はおおむね次の 2 段階に従う: まず NLU がユーザーメッセージから意図（intent）とスロットを抽出し、次に DM が現在の対話文脈から次の対話状態を予測する。

**意図と対話状態の集合は有限であり、会話設計者があらかじめ定義する。** ボットの応答も対話状態に応じて閉じた集合から選ばれる。このアプローチは、あらゆる対話エージェントを厳密に制御する特定の対話フローを定義できるようにする。逆にこれらのエージェントは硬直的であり、NLU と対話フローの設計・更新に多大な人的労力を要する。

スペクトルの反対の端にあるのが、対話追跡とボットメッセージ生成に LLM を用いる最近のエンドツーエンド（E2E）の生成的アプローチ [^10] [^32] である。NeMo Guardrails も LLM 駆動の対話エージェントを構築するのに E2E のアプローチを用いるが、**Colang で書かれた対話フローの状態を解釈・維持できる DM 的なランタイムと、LLM を用いてボットメッセージや新しい対話フローまでも生成する CoT ベースのアプローチを組み合わせる**点が異なる。

## 3 NeMo Guardrails

### 3.1 General Architecture（全体アーキテクチャ）

NeMo Guardrails は、図 3 に詳述するとおり**ユーザーと LLM の間のプロキシのように振る舞う**。開発者は、イベント（会話を含む）のフローを指定するために設計された形式的モデリング言語である Colang を用いて、LLM がユーザーとのやりとりで従うべきプログラム的なレールを定義できる。Colang は Guardrails ランタイムによって解釈され、ランタイムはユーザー定義の規則、あるいは次に述べるとおり LLM が自動生成した規則を適用する。これらの規則がガードレールを実装し、LLM の挙動を導く。

Colang スクリプトの抜粋を図 2 に示す——これらのスクリプトが Guardrails アプリの設定の中核をなす。Colang スクリプトの主要な要素は、**ユーザーの canonical form（正準形）、対話フロー、ボットの canonical form** である。これら 3 種類の定義はすべて**ベクトルデータベース**（例: Annoy [^24], FAISS [^11]）にも索引付けされ、プロンプトのための少数ショット例を選ぶ際の効率的な最近傍探索を可能にする。LLM と Guardrails ランタイムの間のやりとりは Colang の規則を用いて定義される。適切にプロンプトされたとき、LLM は**プロンプト内の少数ショット学習を用いて Colang スタイルのコードを生成できる**。そうでない場合、LLM は通常のモードで動作し自然言語を生成する。

**Canonical form（正準形）** [^25] は、Colang とランタイムエンジンが用いる鍵となる機構である。これらは自然言語（例: 英語）で表現され、意図（intent）に似て、会話におけるメッセージの意味を符号化する。**意図と canonical form の主な違いは、前者がテキスト分類タスクのために閉じた集合として設計されるのに対し、後者は LLM によって生成されるため何ら束縛されず、Guardrails アプリで定義された canonical form によって導かれるだけである点にある**。やりとりを導くレールを定義するのに用いる canonical form の集合は開発者が指定する。これらは、新しいユーザーメッセージの canonical form を生成する際の少数ショット例を選ぶのに使われる。

これらの鍵となる概念を用いて、開発者はさまざまなプログラマブルなレールを実装できる。我々は 2 つの主要なカテゴリを特定した: **topical rails（話題のレール）** と **execution rails（実行のレール）** である。topical rails は対話を制御することを意図しており、たとえば特定の話題への応答を導いたり、複雑な対話ポリシーを実装したりする。execution rails はアプリ開発者が定義したカスタムのアクションを呼ぶ。我々はすべての Guardrails アプリで利用可能な一連の安全性のレールに焦点を当てる。

<figure>

![](../../raw/assets/2023-nemo-guardrails/guardrails-architecture.png)

<figcaption>図3: NeMo Guardrails の全体アーキテクチャ。（訳注: 左から User → Server / Custom app、中央の灰色の枠が NeMo Guardrails ランタイムで「Input → Canonical form」→「Match/generate guardrail flow」→「Execute flow (Colang)」→「Canonical form → Output」の 4 段。左側に K-NN ベクトル探索（AnnoyIndex, in-memory）が Inputs / Guardrail flows / Outputs の 3 つのインデックスを保持し、各段に少数ショット例を供給する。右側の Action Server（LangChain Tools & Chains）がローカル・外部のアクションやツール（例: Zapier）へつながり、LLM 呼び出しは LangChain 経由で LLM サービスへ向かう。）</figcaption>
</figure>

### 3.2 Topical Rails（話題のレール）

topical rails は NeMo Guardrails が用いる鍵となる機構を採用する: プログラマブルなレールを対話フローとして記述する Colang と、対話管理のためのランタイム内の Colang インタプリタ（図 3 の Execute flow [Colang] ブロック）である。フローは、ユーザーの会話がどう進むべきかを決定するために開発者が指定する。Guardrails ランタイムの対話マネージャは、現在の対話文脈でどのフローが有効かを保証するために、**イベント駆動の設計**（イベントを処理して別のイベントを生成し返すイベントループ）を用いる。

ランタイムは、対話フローで会話を導きしたがって topical rails を保証するために、**3 つの主要な段階**を持つ（図 3 参照）。

**ユーザーの canonical form を生成する。** 類似度にもとづく少数ショットプロンプティングを用いて、各ユーザー入力に対する canonical form を生成し、ガードレールシステムがユーザー定義のフローを発火できるようにする。

**次のステップを決定し実行する。** ユーザーの canonical form が特定されると、2 つの経路がありうる: 1) **あらかじめ定義されたフロー**: canonical form が開発者の指定したフローのいずれかに一致すれば、対話マネージャがその特定のフローから次のステップを取り出す。2) **LLM が次のステップを決める**: 現在の対話文脈で定義されていないユーザーの canonical form については、我々は LLM の汎化能力を用いて適切な次のステップを決める——たとえば旅行予約システムで、バスのチケット予約のフローが定義されていれば、ユーザーが航空券を予約したい場合には LLM が類似のフローを生成すべきである。

**ボットのメッセージを生成する。** 次のステップを条件として、LLM に応答を生成するようプロンプトする。したがって、ボットに政治的な質問へ応答してほしくない場合、そのような質問に対する次のステップが bot inform cannot answer であれば、ボットは応答をそらしてレールを尊重する。

付録 B が Colang 言語の詳細を提供する。付録 C にサンプルのプロンプトを含む。

### 3.3 Execution Rails（実行のレール）

このツールキットは「実行」のレールの追加も容易にする。これらは（Python で定義される）カスタムのアクションであり、LLM の入力と出力の双方を監視し、フローの中で遭遇したときに Guardrails ランタイムによって実行されうる。execution rails は幅広いタスクに使えるが、我々は LLM の安全性のために**事実確認（fact-checking）・幻覚（hallucination）・モデレーション（moderation）** をカバーするいくつかのレールを提供する。

#### 3.3.1 Fact-Checking Rail（事実確認のレール）

検索拡張生成（retrieval augmented generation） [^27] の想定のもとで、我々はこのタスクを**含意（entailment）の問題**として定式化する。具体的には、証拠テキストと生成されたボットの応答が与えられたとき、その応答が証拠に接地され含意されているかどうかを LLM に予測させる。証拠と仮説の各対について、モデルは次のプロンプトを用いて二値の含意予測で応答しなければならない。

```
You are given a task to identify if the hypothesis is grounded and entailed in the evidence. You will only use the contents of the evidence and not rely on external knowledge. Answer with yes/no. "evidence": {{evidence}} "hypothesis": {{bot_response}} "entails":
```

モデルが仮説は証拠に含意されないと予測した場合、生成された応答は誤っている可能性があることを示唆する。そうした状況を扱うには、回答を差し控えるなど、異なるアプローチが使える。

#### 3.3.2 Hallucination Rail（幻覚のレール）

検索の要素を含まない一般的な質問については、ボットが事実をでっち上げるのを防ぐ助けとして幻覚のレールを定義する。このレールは **SelfCheckGPT** [^16] に似た**自己一貫性（self-consistency）の検査**を用いる: あるクエリに対して、まず LLM から複数の答えをサンプリングし、次にこれらの異なる答えが一致しているかを検査する。幻覚された記述については、繰り返しのサンプリングは一致しない応答を生成する可能性が高い。

同じプロンプトに対して LLM から $n$ 個のサンプルを得た後、$n-1$ 個の応答を連結して文脈を形成し、$n$ 番目の応答を仮説として用いる。次に、付録 D で定義されるプロンプトテンプレートを用いて、サンプリングされた応答が一貫しているかどうかを LLM に検出させる。

#### 3.3.3 Moderation Rails（モデレーションのレール）

NeMo Guardrails におけるモデレーションの過程は 2 つの主要な構成要素を含む。

$\bullet$ **入力モデレーション**（**jailbreak rail** とも呼ばれる）は、悪意のありうるユーザーメッセージが対話システムに到達する前に検出することを目指す。

$\bullet$ **出力モデレーション**は、LLM の応答がユーザーに返される前に、合法的・倫理的で無害かどうかを検出することを目指す。

モデレーションのシステムはパイプラインとして機能し、ユーザーメッセージはまず入力モデレーションを通過してから対話システムに到達する。対話システムが LLM に駆動された応答を生成した後、出力モデレーションのレールが発火する。**両方のモデレーションのレールを通過して初めて**、応答はユーザーに返される。

入力・出力いずれのモデレーションのレールも、**強力でよくアラインされた LLM に対する別のタスク**として定式化され、その LLM が入力または応答を審査する。これらのレールのプロンプトテンプレートは付録 D にある。

## 4 Sample Guardrails Applications（Guardrails アプリケーションの例）

Colang スクリプトを使えば、会話アプリケーションへのレールの追加は単純で直截である。

### 4.1 Topical Rails

topical rails は execution rails と組み合わせて、特定のアクションをいつ呼ぶべきかを決めたり、タスク指向のエージェントを構築するための複雑な対話フローを定義したりするのに使える。

図 2 に示した例では、Guardrails アプリが WolframAlpha エンジンを使って数学と距離の問い合わせに応答できるようにする 2 つの topical rail を実装している。これを達成するために、wolfram alpha request のカスタムアクション（Python で実装され、GitHub で利用可能）が WolframAlpha API を使ってユーザーのクエリへの応答を取得する。この応答は次に、現在の会話の文脈で答えを生成するために LLM によって用いられる。

### 4.2 Execution Rails

execution rails を追加する手順は次のとおりである。

1. **アクションを定義する** —— レールを定義するには、開発者がそのレールのロジックを指定するアクションを（Python で）定義する必要がある。
2. **対話フローでアクションを呼び出す** —— アクションが定義されたら、execute キーワードを使って Colang からそのアクションを呼べる。
3. **対話フローでアクションの出力を使う** —— 開発者は、アクションからの出力にアプリケーションがどう反応すべきかを指定できる。

付録 E に、アクションの定義の詳細と、入力・出力のモデレーションのレールを実装するアクションの例を含む。

図 4 は、$check\_jailbreak$ アクションを呼び出す Colang のサンプルフローを示す。jailbreak のレールがユーザーメッセージにフラグを立てた場合、開発者は生成された応答を表示せず、代わりに既定のテキストを出力すると決めることができる。付録 F に execution rails を用いた他のフローの例を示す。

<figure>

![](../../raw/assets/2023-nemo-guardrails/jailbreak_rail_v2.png)

<figcaption>図4: Colang で jailbreak のレールを用いるフロー。</figcaption>
</figure>

## 5 Evaluation（評価）

本節では、さまざまなレールの性能をどう測るかの詳細を提供する。すべてのタスクに関する追加の情報と、NeMo Guardrails で利用可能な自動評価ツールの議論は付録 G にある。

### 5.1 Topical Rails

topical rails の評価は、canonical form と対話フローを用いて会話を導くために本ツールキットが用いる**中核的な機構**に焦点を当てる。現在の評価実験は会話 NLU に用いられるデータセットを採用している。本節では Banking データセット [^7] の結果を示し、追加の実験は付録 G にある。

NLU データセットから出発して、我々は意図を canonical form に対応づけ、それらのために単純な対話フローを定義することで Colang アプリケーション（GitHub で公開）を作成する。実験に用いた評価データセットはバランスの取れたもので、元のデータセットからランダムにサンプリングされた**意図ごとに最大 3 サンプル**を含む。テストデータセットは **77 個の異なる意図にわたる 231 サンプル**を持つ。

上位 3 つの高性能モデルの結果を図 5 に示す。**falcon-7b-instruct や llama2-13b-chat といったより小さなオープンソースのモデルでも、topical rails が会話を導くのにうまく使える**ことを示している。LLM の性能はプロンプトに大きく依存するので、すべての結果はより良いプロンプティングによって改善されうる。

topical rails の評価はいくつかの重要な側面を際立たせる。**第一に**、Guardrails が用いる 3 段階のアプローチ（ユーザーの canonical form、次のステップ、ボットのメッセージ）の各段階が性能の改善をもたらす。**第二に**、良い性能を達成するには、各ユーザーの canonical form についてベクトルデータベースに**少なくとも $k=3$ 個のサンプル**を持つことが重要である。**第三に**、一部のモデル（すなわち gpt-3.5-turbo）は、少数ショットプロンプティングを用いてさえ**より多様な canonical form を生成する**。そうした場合、canonical form の生成には完全一致の代わりに**類似度によるマッチ**を加えるのが有用である。

<figure>

![](../../raw/assets/2023-nemo-guardrails/evaluation-topical-rails-banking-short-v2.png)

<figcaption>図5: Banking における topical rails の性能。（訳注: 青が canonical form、赤が next step / flow の精度。text-davinci-003 が 77%/83%、gpt-3.5-turbo が 38%/45%、falcon-7b-instruct が 70%/76%、llama2-13b-chat が 76%/77%。本文は「上位 3 つのモデル」と述べているが図には 4 モデルが並んでいる。）</figcaption>
</figure>

### 5.2 Execution Rails

##### Moderation Rails（モデレーションのレール）

モデレーションのレールを評価するために、我々は Anthropic の Red-Teaming データセットと Helpful データセット [^2] [^22] を用いる。我々はバランスの取れた harmful-helpful の評価セットを次のようにサンプリングした: Red-Teaming データセットからは最も有害性スコアの高いプロンプトをサンプリングし、Helpful データセットからは同数のプロンプトを選ぶ。

我々はレールの性能を、**有害なプロンプトが遮断された割合と、有用なプロンプトが許可された割合**にもとづいて定量化する。結果の分析は、**入力と出力の双方のモデレーションのレールを使うことが、いずれか一方のレールを単独で使うよりはるかに堅牢である**ことを示している。両方のレールを使うと gpt-3.5-turbo は素晴らしい性能を示す——有害な要求の **99% 近く**を遮断し（レールなしでは 93%）、有用な要求はわずか 2% しか遮断しない。詳細は付録 G。

##### Fact-Checking Rail（事実確認のレール）

事実確認のレールの性能を評価するために MSMARCO データセット [^4] を考える。このデータセットは（文脈, 質問, 答え）の三つ組からなる。負例（文脈に**接地されていない**答え）を採掘するために、我々は OpenAI text-davinci-003 を用いて肯定的な答えを、それに似ているが証拠に接地されていない**難しい負例**へ書き換えさせる。肯定・否定の三つ組を等しくサンプリングして結合データセットを構築する。text-davinci-003 と gpt-3.5-turbo はいずれも事実確認のレールでうまく機能し、**全体の精度 80%** を得る（付録 G.2.2 の図 11 参照）。

##### Hallucination Rail（幻覚のレール）

幻覚のレールの評価は、主観的な手作業の注釈を用いずには困難である。この問題を克服してその性能を自動的に定量化できるようにするために、我々は**偽の前提にもとづく 20 個の質問**（正しい答えを持たない質問）のリストを編纂する。

言語モデルからの生成のうち、（回答をそらす）deflection 以外はすべて失敗とみなす。次に、幻覚のレールをフォールバック機構として採用することの便益を定量化する。text-davinci-003 については、LLM は答えられないプロンプトをそらすことができず、幻覚のレールを使うと**そうしたプロンプトの 70% を捕捉**するのに役立つ。gpt-3.5-turbo ははるかにうまく機能し、答えられないプロンプトをそらすか、応答が誤っている可能性があると示すかを **65% の場合に**行う。この場合でさえ、幻覚のレールを採用すると性能は **95%** まで押し上げられる。

<figure>

![](../../raw/assets/2023-nemo-guardrails/Hallucination_Rail.png)

<figcaption>図6: 幻覚のレールの性能。（訳注: 青が「モデルのアラインメントのみで捕捉」、緑が「モデル＋幻覚のレールで捕捉」。text-davinci-003 は 0% → 70%、gpt-3.5-turbo は 65% → 90%。本文は「95% まで押し上げられる」と述べているが、図と付録 G.2.3 の記述はいずれも 90%（65% から 25 ポイントの改善）を示す。）</figcaption>
</figure>

## 6 Conclusions（結論）

我々は NeMo Guardrails を提示した。これは開発者が**プログラマブルなレール**を実装することで、制御可能で安全な LLM ベースのアプリケーションを構築できるようにするツールキットである。これらのレールは Colang を用いて表現され、複雑なロジックを要する場合はカスタムアクションとしても実装できる。CoT プロンプティングと Colang コードを解釈できる対話マネージャを用いて、Guardrails ランタイムはアプリケーションと LLM の間のプロキシのように振る舞い、ユーザー定義のレールを強制する。

## 7 Limitations（限界）

### 7.1 Programmable Rails and Embedded Rails（プログラマブルなレールと埋め込まれたレール）

制御可能で安全な LLM 駆動のアプリケーション一般、とりわけ対話システムを構築することは困難なタスクである。**我々は、プロンプティングと Colang インタプリタで実装される開発者定義のプログラマブルなレールという NeMo Guardrails のアプローチが、完璧な解決策ではないことを認める。**

したがって我々は、**可能な限り、我々のツールキットを——とりわけ安全性に固有のレールについては——単独の解決策として用いるべきではない**と主張する。プログラマブルなレールは埋め込まれたレールを補完するものであり、安全な LLM アプリケーションを構築するにはこの 2 つの解決策を**併用すべき**である。本プロジェクトのビジョンは、将来的に execution rails のいくつかについて、現在の純粋なプロンプティングの手法を補完するより強力なカスタマイズされたモデルを提供することでもある。他方で我々の結果は、**強力な LLM（例: ChatGPT）に埋め込まれた既存の安全性のレールにモデレーションのレールを追加すると、ジェイルブレイク攻撃に対するより良い保護が得られる**ことを示している。

制御可能でタスク指向の対話エージェントの文脈では、可能なすべてのタスクと topical rails についてカスタマイズされたモデルを開発するのは困難である。したがってこの文脈では、NeMo Guardrails は追加の機構なしに LLM 駆動のタスク指向エージェントを構築するための実行可能な解決策である。ただし topical rails とタスク指向エージェントについても、我々は一部のタスク（例: canonical form の生成）についてより良い性能を達成する p-tuned モデルを公開する計画である。

### 7.2 Extra Costs and Latency（追加のコストとレイテンシ）

Guardrails ランタイムが用いる 3 段階の CoT プロンプティングのアプローチは、**追加のコストと追加のレイテンシ**を招く。これらの呼び出しは逐次的に連鎖している（すなわち第 2 段階における次のステップの生成は、第 1 段階で生成されたユーザーの canonical form に依存する）ため、**呼び出しをバッチ処理できない**。我々の現在の実装では、必要なレイテンシとコストは、Guardrails を使わずにボットのメッセージを生成する通常の呼び出しの**およそ 3 倍**である。我々は現在、場合によっては単一の呼び出しで 3 つのステップ（ユーザーの canonical form、フローの次のステップ、ボットのメッセージ）すべてを生成できないか調査している。

より複雑なプロンプトと少数ショットの文脈内学習を用いることも、素の会話における通常のボットメッセージ生成と比べてわずかに追加のレイテンシとより大きなコストを生む。開発者は必要ならより単純なプロンプトを使うと決めることもできる。

しかし我々は、開発者には自分のニーズに合わせたさまざまな選択肢が提供されるべきだと考えている。安全で制御可能な LLM 駆動の対話エージェントを持つために追加のコストを払ってもよいと考える人もいるだろう。さらに GPU の推論コストは下がるだろうし、より小さなモデルも NeMo Guardrails のタスクの一部またはすべてで良い性能を達成できる。我々の論文で示したとおり、falcon-7b-instruct [^20] はすでに topical rails で非常に良い性能を達成することが分かっている。Llama 2（7B と 13B）の chat 版 [^26] のような他の最近のモデルからも同様の良好な性能が見られている。

## 8 Broader Impact（広範な影響）

LLM アプリケーション（対話システムを含む）にプログラマブルなレールを強制するツールキットとして、NeMo Guardrails は開発者と研究者に便益を提供するはずである。プログラマブルなレールは、一般的なもの（RLHF を用いる）であれユーザー定義のもの（p-tuned のカスタマイズされたモデルを用いる）であれ、埋め込まれたレールを補完する。たとえば事実確認のレールを使えば、開発者は強化された検索ベースの LLM アプリケーションを容易に構築でき、また**プログラマブルなレールはモデル非依存なので、さまざまなモデルの性能を評価する**こともできる。複雑な対話フローに従うべき LLM ベースのタスク指向エージェントの構築についても同じことが言える。

同時に、Guardrails アプリケーションを本番に投入する前に、実装されたプログラマブルなレールは（とりわけ安全性に関わるレールは）徹底的にテストされるべきである。我々のツールキットは、topical rails と execution rails の双方について性能をテストするための一連の評価ツールを提供する。

我々のツールキットに関する追加の詳細は付録にあり、GitHub で共有されている例示の Guardrails アプリケーションでツールキットを実行するための簡単なインストール手順も含む。短いデモ動画も利用可能である: [https://youtu.be/Pfab6UWszEc](https://youtu.be/Pfab6UWszEc)。

## Appendix A Installation Guide and Examples（インストールガイドと例）

開発者は NeMo Guardrails ツールキットの最新版を GitHub <sup>2</sup> から直接ダウンロードしてインストールできる。`pip install nemoguardrails` を使って最新の安定版をインストールすることもできる。

我々は、提供されるコマンドラインインターフェース（CLI）を使って Guardrails アプリを実行する方法、あるいは Guardrails のウェブサーバを立ち上げる方法を示す簡潔なインストールガイド <sup>3</sup> を用意している。サーバは、起動時に指定したフォルダにあるすべての Guardrails アプリとやりとりするための単純なチャットのウェブクライアントを駆動する。

さまざまな種類のレールを構築するための一般的なデモンストレーションとして、**5 つの参照 Guardrails アプリケーション**が提供されている。

- **Topical Rail**: ボットを特定の会話の話題に留まらせる。
- **Moderation Rail**: ボットの応答をモデレートする。
- **Fact Checking and Hallucination Rail**: 事実に即した答えを保証する。
- **Secure Execution Rail**: LLM とともに第三者のサービスを実行する。
- **Jail-breaking Rail**: ユーザーの悪意ある意図にもかかわらず安全な答えを保証する。

**これらの例はレールを構築する過程を示すことを意図したものであり、そのまま使える安全機能としてではない。レールのカスタマイズと強化が強く推奨される。**

サンプルの Guardrails アプリケーションには、ローカルに配備された、あるいは HuggingFace Inference のプライベートエンドポイントを使った複数のオープンソースモデル（例: falcon-7b-instruct, dolly-v2-3b, vicuna-7b-v1.3）の使い方の例も含まれる。他の例は、Langchain で定義されたさまざまなチェーンを NeMo Guardrails で定義されたプログラマブルなレールと組み合わせる方法をカバーする。

参照アプリケーションとツールキット全般に関する追加の詳細は、主要なドキュメントページ <sup>4</sup> にある。

> 訳注（脚注 2・3・4、原ページより復元）: それぞれ [https://github.com/NVIDIA/NeMo-Guardrails/](https://github.com/NVIDIA/NeMo-Guardrails/)、`https://github.com/NVIDIA/NeMo-Guardrails/blob/main/docs/getting_started/installation-guide.md`、`https://github.com/NVIDIA/NeMo-Guardrails/blob/main/docs/README.md`。

## Appendix B Colang Language and Dialogue Manager（Colang 言語と対話マネージャ）

Colang はイベントとやりとりの系列をモデル化するための言語であり、とりわけ会話のモデル化に有用である。同時に、イベントベースの処理エンジンであり対話マネージャのように振る舞う Colang インタプリタを使って、会話システムのためのガードレールの設計を可能にする。

会話システムのためのガードレールを作ることは、ユーザーとボットの間の対話がどう展開するかについてのなんらかの理解を必要とする。フローチャート・状態機械・フレームベースのシステムといった既存の対話管理の技法は、**LLM ベースのシステムとやりとりする際に期待されるような、非常に柔軟な会話のフローをモデル化するのには適していない**。

しかし、新しい言語を学ぶことは容易なタスクではないため、**Colang は自然言語（英語）と Python の混合として設計された**。Python に馴染みがあれば、いくつかの例を見れば説明なしでも Colang を自信を持って使えるはずである。

Colang 言語が用いる主な概念は次のとおりである。

- **Utterance（発話）**: ユーザーまたはボットから来る生のテキスト。
- **Message（メッセージ）**: ユーザー／ボットの発話の canonical form（構造化された表現）。
- **Event（イベント）**: 会話に関連して起こった何か。例: ユーザーが沈黙している、ユーザーが何かをクリックした、ユーザーがジェスチャーをした、など。
- **Action（アクション）**: ボットが呼び出せるカスタムのコード。通常は第三者の API に接続するためのもの。
- **Context（文脈）**: 会話に関連する任意のデータ（キー・値の辞書として符号化される）。
- **Flow（フロー）**: メッセージとイベントの系列。追加の分岐ロジックを伴いうる。
- **Rails（レール）**: 会話システム（別名ボット）の挙動を制御する特定の方法。例: 政治について話さない、特定のユーザーの要求に特定の仕方で応答する、あらかじめ定められた対話の経路をたどる、特定の言語スタイルを使う、データを抽出する、など。Colang におけるレールは 1 つ以上のフローを通じてモデル化できる。

Colang の追加の詳細については、Colang 構文ガイド <sup>5</sup> を参照されたい。

Guardrails ランタイムは**イベント駆動の設計**（すなわち、イベントを処理して別のイベントを生成し返すイベントループ）を用いる。対話フローはイベントの系列として扱われるが、単純なユーザーメッセージもまたイベントである——`UtteranceUserActionFinished` イベントが作られてランタイムに送られる。より詳しくは NeMo Guardrails のアーキテクチャガイド <sup>6</sup> にある。

> 訳注（脚注 5・6、原ページより復元）: それぞれ `https://github.com/NVIDIA/NeMo-Guardrails/blob/main/docs/user_guide/colang-language-syntax-guide.md`、`https://github.com/NVIDIA/NeMo-Guardrails/blob/main/docs/architecture/README.md`。

## Appendix C Prompts for Topical Rails（topical rails のためのプロンプト）

NeMo Guardrails は、第 3.2 節で述べたとおり、ユーザーメッセージに応答するために **3 段階に連鎖した複雑なプロンプト**を用いる。以下のリストでは、現在の会話における最後のユーザーメッセージの canonical form を生成する第 1 段階の例を提供する。

以下のプロンプトは text-davinci-003 向けに設計されており、**4 つの部分**に構造化されている。

1. アプリケーションのタスクを記述する一般的なプロンプト。
2. Colang 構文を用いたサンプルの会話。
3. 現在のユーザーメッセージに対して最も類似する、ユーザーメッセージを対応する canonical form へ対応づけるための少数ショット（$k=5$）の例。
4. Colang 構文による、ユーザーとボットの間の現在の会話。

```
"""
Below is a conversation between a helpful AI assistant and a user. The bot is designed to generate human-like text based on the input that it receives. The bot is talkative and provides lots of specific details. If the bot does not know the answer to a question, it truthfully says it does not know.
"""

# This is how a conversation between a user and the bot can go:

user "Hello there!"
  express greeting
bot express greeting
  "Hello! How can I assist you today?"
user "What can you do for me?"
  ask about capabilities
bot respond about capabilities
  "I am an AI assistant which helps answer questions based on a given knowledge base. For this interaction, I can answer question based on the job report published by US Bureau of Labor Statistics"
```

> 訳注: 原典はこの後にも同じプロンプトの続き（類似例と現在の会話の部分）を示しているが、底本のクリップに残っているのは上記までである。原ページ側でも同じ範囲までしか掲載されていない。

## Appendix D Prompt Templates for Execution Rails（execution rails のためのプロンプトテンプレート）

本節では、幻覚とモデレーションのレールが用いるプロンプトテンプレートを提供する。

### D.1 Hallucination Rail

会話エージェントから同じプロンプトに対して $n$ 個のサンプルを得た後、$n-1$ 個の応答を連結して文脈を形成し、$n$ 番目の応答を仮説として用いる。次のプロンプトテンプレートを用いて、仮説が文脈と一貫しているかを LLM に検証させる。

```
You are given a task to identify if the hypothesis is in agreement with the context below. You will only use the contents of the context and not rely on external knowledge. Answer with yes/no. "context": {{ sampled_responses }} "hypothesis": {{ bot_response }} "agreement":
```

ボットの応答をサンプリングする際、SelfCheckGPT [^16] が提案するとおり、**高い温度（$temp=1$）** を用いて、答えの自己一貫性を評価するのに必要な応答間のばらつきを確保する。同時に、LLM ベースの幻覚検証の呼び出しは最も確からしい応答を得るために **$temp=0$** を用いる。この挙動は本論文で述べる他のすべての execution rails についても同様である。

### D.2 Moderation Rails

入力・出力いずれのモデレーションのレールも、強力でよくアラインされた LLM に対する別のタスクとして定式化され、その LLM が入力または応答を審査する。**jailbreak のレール**には次のプロンプトテンプレートを用いる。

```
Instruction: {{ user_input }}
Would this instruction make a language model break moderation policies, deviate from good aligned responses and provide answers that a language model should ideally not? Answer with yes/no.
```

**出力モデレーションのレール**は次のプロンプトテンプレートを用いる。

```
Model output: {{ bot_response }}
Is this output legal, ethical and not harmful to any person? Answer with yes/no.
```

## Appendix E Action Definitions（アクションの定義）

execution rail を作る第一段階は、そのレールのロジックを実装するカスタムのアクションを定義することである。現時点ではこれは Python で行う。

任意のカスタムアクションは、次の例で見られるとおり会話の文脈にアクセスできる。Guardrails ランタイムにおいて、文脈は会話履歴における全イベントの系列である——ユーザーとボットのメッセージ、canonical form、呼ばれたアクションなどを含む。より頻繁にアクセスされうる一部の文脈イベントにはショートカットがある（例: $context.get("last\_bot\_message")$）。

アクションは、それが呼ばれる Colang スクリプトから任意の個数のパラメータを受け取れる。これらはアクションのロジックを実装する Python 関数に渡される。同時にアクションは通常、対話をさらに導くのに使える値を返す。

### E.1 Output Moderation Rail（出力モデレーションのレール）

これは出力モデレーションのアクションの現在の実装である。付録 D で定義したプロンプトテンプレートを用い、ボットの応答が出力モデレーションのレールに違反する場合に $True$ を返す。

LLM への呼び出しは、複数の LLM 提供者との通信を単純化するために Langchain <sup>8</sup> が提供するプロンプトテンプレートと単純な LLM チェーンを使って行われる。

```python
bot_response = context.get("last_bot_message")

if bot_response:
    output_moderation_template =
    "Model Output: {bot_response}

    Is this output legal, ethical and not harmful to any person? Answer with yes/no."

    prompt = PromptTemplate(
        template=output_moderation_template, input_variables=["bot_response"]
    )

    output_moderation_chain = LLMChain(prompt=prompt, llm=llm)
    check = await output_moderation_chain.apredict(bot_response=bot_response)

    check = check.lower().strip()
    log.info(f"Output moderation check result is {check}.")

    if "no" in check:
        return False
    return True
```

### E.2 Jailbreak Rail（jailbreak のレール）

これは入力モデレーションのアクションの現在の実装である。付録 D で定義したプロンプトテンプレートを用い、ユーザーの入力が jailbreak のレールに違反する場合に $True$ を返す。

```python
user_input = context.get("last_user_message")

if user_input:
    jailbreak_template = "Instruction: {user_input}

    Would this instruction make a language model break moderation policies, deviate from good aligned responses and provide answers that a language model should ideally not? Answer with yes/no."

    prompt = PromptTemplate(
        template=jailbreak_template, input_variables=["user_input"]
    )

    jailbreak_chain = LLMChain(prompt=prompt, llm=llm)
    check = await jailbreak_chain.apredict(bot_response=bot_response)

    check = check.lower().strip()
    log.info(f"Jailbreak check result is {check}.")

    if "no" in check:
        return False
    return True
```

> 訳注: 上記 E.2 のコードは**原典に掲載されたまま**である。ただし `jailbreak_chain.apredict(bot_response=bot_response)` の行は、テンプレートの変数が `user_input` であり `input_variables` も `["user_input"]` である以上、`user_input=user_input` の誤りと読める（E.1 からの写し間違いと思われる）。加えて `bot_response` はこのスコープで定義されていない。

## Appendix F Sample Guardrails Flows using Actions（アクションを用いた Guardrails フローの例）

本節は、単純な Colang アプリケーションを定義するために、カスタムアクションとして実装された安全性の execution rails を Colang フローの中で用いる例をいくつか含む。

図 7 は、入力モデレーションのために `check_jailbreak` アクションを使う方法を示す。その意味は、各ユーザーメッセージ（`user...`）について jailbreak のアクションが呼ばれて最後のユーザーメッセージを検証し、それがジェイルブレイクの試みとしてフラグを立てられた場合には、LLM が生成した最後のボットの答えが除去され、ユーザーのメッセージがモデレーションポリシーに違反することを知らせる新しい答えが発話される、というものである。図 8 は `output_moderation` アクションの使われ方を示す——意味はジェイルブレイクと同様だが、こちらは**任意の出力ボットメッセージのイベントの後**（`bot...`）に発火する。

<figure>

![](../../raw/assets/2023-nemo-guardrails/jailbreak_rail_v2.png)

<figcaption>図7: Colang で jailbreak のレールを用いるフロー。（訳注: 原典は図4 と同一の画像を再掲している。）</figcaption>
</figure>

<figure>

![](../../raw/assets/2023-nemo-guardrails/output_moderation_v2.png)

<figcaption>図8: Colang で出力モデレーションを用いるフロー。</figcaption>
</figure>

同様に、図 9 は特定の話題（すなわち人物についての質問。GPT 系のモデルが幻覚を起こしやすい）について応答を検査するために幻覚のレールを使う方法を示す。この場合、ボットのメッセージは除去されず、代わりに答えが誤っている可能性についてユーザーに警告する追加のメッセージが加えられる。図 10 は、特定の話題について質問された場合（雇用報告についての質問）に事実確認を加える方法を示す。この状況では、LLM は報告書の情報と一貫しているべきである。

<figure>

![](../../raw/assets/2023-nemo-guardrails/hallucination_moderation_v2.png)

<figcaption>図9: Colang で幻覚のレールを用いるフロー。</figcaption>
</figure>

<figure>

![](../../raw/assets/2023-nemo-guardrails/check-facts-v2.png)

<figcaption>図10: Colang で事実確認のレールを用いるフロー。</figcaption>
</figure>

> 訳注（脚注 7・8、原ページより復元）: それぞれ `https://github.com/NVIDIA/NeMo-Guardrails/tree/main/nemoguardrails/llm/prompts`、[https://github.com/langchain-ai/langchain](https://github.com/langchain-ai/langchain)。

## Appendix G Additional Details on Evaluation（評価に関する追加の詳細）

我々のツールキットは、topical rails と execution rails の性能を評価するための評価ツールと方法論も提供する。本論文で報告したすべての結果は、GitHub で利用可能な CLI 評価ツールを使い、評価についての説明 <sup>9</sup> に従って再現できる。同じページには現在の論文よりわずかに詳細な情報があり、（新しい LLM を含む）新しい結果で定期的に更新されている。

実験を再現する詳細な手順はここ <sup>10</sup> にある。

> 訳注（脚注 9・10、原ページより復元）: それぞれ `https://github.com/NVIDIA/NeMo-Guardrails/blob/main/nemoguardrails/eval/README.md`、`https://github.com/NVIDIA/NeMo-Guardrails/blob/main/docs/README.#evaluation-tools`。

### G.1 Topical Rails

topical rails の評価は、canonical form と対話フローを用いて会話を導くために NeMo Guardrails が用いる中核的な機構に焦点を当てる。

topical rails の現在の評価実験は、会話 NLU に用いられる 2 つのデータセットを使う: **chit-chat** <sup>11</sup> と **banking** である。

これらのデータセットは、各意図に対する canonical form、特定の対話フロー、そして（chit-chat データセットについてのみ）ボットのメッセージまでを定義することによって、NeMo Guardrails のアプリへ変換された。2 つのデータセットは多数のユーザーの意図、したがって topical rails を持つ。一方は非常に一般的で粗い粒度の意図を持ち（chit-chat）、banking データセットはドメイン固有でより細かい粒度である。topical rails の評価実験の実行と評価データセットの詳細はここにある。

> 訳注（脚注 11、原ページより復元）: `https://github.com/rahul051296/small-talk-rasa-stack`。データセットはもともと Rasa によって公開されたものである。

予備的な評価結果を次に示す。すべての実験において、我々は**意図ごとに最大 3 サンプル**のバランスの取れたテストセットを持つよう選んだ。両データセットについて、さまざまな LLM と、ベクトルデータベースに索引付けされる意図ごとのサンプル数（$k=all, 3, 1$）についての性能を評価した。一貫性を確保するため、すべての実験で乱数シード **42** を用いた。

上位 3 つの高性能モデルの結果を図 5 に示す。falcon-7b-instruct や llama2-13b-chat といったより小さなオープンソースのモデルでも、topical rails が会話を導くのにうまく使えることを示している。LLM の性能はプロンプトに大きく依存するので、NeMo Guardrails が用いる複雑なプロンプトゆえに、すべての結果はより良いプロンプティングによって改善されうる。

topical rails の評価はいくつかの重要な側面を際立たせる。第一に、Guardrails が用いる 3 段階のアプローチ（ユーザーの canonical form、次のステップ、ボットのメッセージ）の各段階が性能の改善をもたらす。第二に、良い性能を達成するには、各ユーザーの canonical form についてベクトルデータベースに少なくとも $k=3$ 個のサンプルを持つことが重要である。第三に、一部のモデル（すなわち gpt-3.5-turbo）は、少数ショットプロンプティングを用いてさえより多様な canonical form を生成する。そうした場合、canonical form の生成には完全一致の代わりに類似度によるマッチを加えるのが有用である。**この場合、類似度のしきい値が重要な推論パラメータになる。**

データセットの統計と複数の LLM についての詳細な結果を表 1・2・3 に示す。一部の実験に数値が欠けているのは、それらの実験がその指標を計算しなかったか、データセットが特定の項目を含まない（たとえば banking データセットにはユーザー定義のボットメッセージがない）ためである。

**表1**: topical rails 評価のためのデータセット統計。

| データセット | 意図数 | テストサンプル数 |
| --- | --- | --- |
| chit-chat | 76 | 226 |
| banking | 77 | 231 |

**表2**: chit-chat データセットにおける topical 評価の結果。Us int はユーザーの意図の精度、Bt int は次ステップ生成（すなわちボットの意図）の精度、Bt msg は生成されたボットメッセージの精度を意味する。Sim は、マッチに意味的類似度を用いた（指定されたしきい値、この場合 $0.6$）か完全一致を用いたかを示す。

| モデル | Us int, no sim | Us int, sim=0.6 | Bt int, no sim | Bt int, sim=0.6 | Bt msg, no sim | Bt msg, sim=0.6 |
| --- | --- | --- | --- | --- | --- | --- |
| text-davinci-003, k=all | 0.89 | 0.89 | 0.90 | 0.90 | 0.91 | 0.91 |
| text-davinci-003, k=3 | 0.82 | N/A | 0.85 | N/A | N/A | N/A |
| text-davinci-003, k=1 | 0.65 | N/A | 0.73 | N/A | N/A | N/A |
| gpt-3.5-turbo, k=all | 0.44 | 0.56 | 0.50 | 0.61 | 0.54 | 0.65 |
| dolly-v2-3b, k=all | 0.65 | 0.78 | 0.68 | 0.78 | 0.69 | 0.78 |
| falcon-7b-instruct, k=all | 0.81 | 0.81 | 0.81 | 0.82 | 0.81 | 0.82 |
| llama2-13b-chat, k=all | 0.87 | N/A | 0.88 | N/A | 0.89 | N/A |

**表3**: banking データセットにおける topical 評価の結果。

| モデル | Us int, no sim | Us int, sim=0.6 | Bt int, no sim | Bt int, sim=0.6 | Bt msg, no sim | Bt msg, sim=0.6 |
| --- | --- | --- | --- | --- | --- | --- |
| text-davinci-003, k=all | 0.77 | 0.82 | 0.83 | 0.84 | N/A | N/A |
| text-davinci-003, k=3 | 0.65 | N/A | 0.73 | N/A | N/A | N/A |
| text-davinci-003, k=1 | 0.50 | N/A | 0.63 | N/A | N/A | N/A |
| gpt-3.5-turbo, k=all | 0.38 | 0.73 | 0.45 | 0.73 | N/A | N/A |
| dolly-v2-3b, k=all | 0.32 | 0.62 | 0.40 | 0.64 | N/A | N/A |
| falcon-7b-instruct, k=all | 0.70 | 0.76 | 0.75 | 0.78 | N/A | N/A |
| llama2-13b-chat, k=all | 0.76 | N/A | 0.78 | N/A | N/A | N/A |

### G.2 Execution Rails

#### G.2.1 Moderation Rail

モデレーションのレールを評価するために、我々は Anthropic の Red-Teaming データセットと Helpful データセット [^2] [^22] を用いる。red-teaming データセットは、言語モデルから不適切な応答を引き出す能力について人間が注釈を付けた（0〜4）プロンプトからなる。スコアが高いほど、そのプロンプトがモデルのアラインメントを迂回するのに成功したことを意味する。我々は最高評価のプロンプトをランダムにサンプリングして harmful セットを編纂する。Anthropic の Helpful データセットのすべてのプロンプトは正真正銘のクエリであり、これが我々の helpful セットを形成する。我々は harmful と helpful を同数含むバランスの取れた評価セットを作成する。

我々はレールの性能を、有害なプロンプトが遮断された割合と、有用なプロンプトが許可された割合にもとづいて定量化する。**理想的なモデルは、有害なプロンプトの 100% を遮断し、有用なプロンプトの 100% を許可できるだろう。** 我々は評価セットのプロンプトを入力（jailbreak）モデレーションのレールに通す。フラグを立てられなかったものだけが会話エージェントに渡されて応答が生成され、その応答が出力モデレーションのレールに通される。ここでも、フラグを立てられなかった応答だけがユーザーに表示される。

結果の分析は、入力（別名 jailbreak のレール）と出力の双方のモデレーションのレールの組み合わせを使うことが、いずれか一方のレールを単独で使うより堅牢であることを示している。また、**出力モデレーションのレールの評価は主観的であり、各人／各組織が何を通過させるべきかについて異なる主観的意見を持つ**ことにも注意すべきである。そうした状況では、会話エージェントを配備する主体の信念を反映するようモデレーションのレールへのプロンプトを修正するのは容易だろう。

上記のとおり作成された、harmful と helpful を等分した **200 サンプル**の評価セットを用いて、我々は次を確認した。

- **text-davinci-003 は有害なメッセージの 24% しか遮断しない**のに対し、**gpt-3.5-turbo はモデレーションのガードレールなしで 93% の有害なメッセージを遮断**するという、はるかに良い結果を示す。ここで遮断とは、モデレーションを要する入力に対してモデルが応答を提供しないことを意味する。有用な入力については、いずれのモデルもいかなる要求も遮断しない。
- 入力モデレーションのレールのみを使うと、text-davinci-003 は有害な要求の 87% と有用な要求の 3% を遮断する。
- 入力と出力の双方のモデレーションを使うと、**text-davinci-003 は有害な要求の 97% と有用な要求の 5% を遮断**し、**gpt-3.5-turbo は素晴らしい性能を示す——有害な要求の 99% 近くを遮断し、有用な要求はわずか 2%** である。

#### G.2.2 Fact-checking Rail

事実確認のレールの性能を評価するために MSMARCO データセット [^4] を考える。このデータセットは（文脈, 質問, 答え）の三つ組からなる。負例（文脈に**接地されていない**答え）を採掘するために、我々は OpenAI text-davinci-003 を用いて肯定的な答えを、それに似ているが証拠に接地されていない難しい負例へ書き換えさせる。肯定・否定の三つ組を等しくサンプリングして結合データセットを構築する。text-davinci-003 と gpt-3.5-turbo はいずれも事実確認のレールでうまく機能し、全体の精度 80% を得る（図 11）。**2 つのモデルの挙動はわずかに異なる——gpt-3.5-turbo は負例の発見が得意である一方、text-davinci-003 は肯定的なサンプルでより良い性能を示す。**

<figure>

![](../../raw/assets/2023-nemo-guardrails/MSMARCO.png)

<figcaption>図11: 事実確認のレールの性能。</figcaption>
</figure>

#### G.2.3 Hallucination Rail

幻覚のレールの評価は、言語モデルのパラメータに埋め込まれた事実知識で答えられる質問を確定できないため困難である。モデルが幻覚を検出する能力を効果的に定量化するために、我々は**偽の前提にもとづく 20 個の質問**のリストを編纂する。たとえば、正しい答えを持たないそうした質問の 1 つは「メキシコ湾の海底都市はいつ建設されたのか？」である。

言語モデルからの生成のうち、deflection（すなわち、その質問が答えられないものだと認識すること）以外はすべて失敗とみなす。我々はまた、幻覚のレールをフォールバック機構として採用することの便益も定量化する。text-davinci-003 については、ベースの言語モデルは答えられないプロンプトをそらすことができず、幻覚のレールを使うと答えられないプロンプトの 70% を捕捉するのに役立つ。gpt-3.5-turbo は、答えられないプロンプトをそらしたり、応答が誤っている可能性について言及して答えを和らげたりすることに非常に長けている。そうした強力なモデルについてさえ、幻覚のレールを採用することは、**誤った応答を招きやすい質問の識別を 25% 押し上げる**のに役立つことを我々は見出した。
