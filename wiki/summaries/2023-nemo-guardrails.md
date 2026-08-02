---
type: summary
source_path: raw/papers/NeMo Guardrails_ A Toolkit for Controllable and Safe LLM Applications with Programmable Rails.md
source_kind: paper
title: "NeMo Guardrails: A Toolkit for Controllable and Safe LLM Applications with Programmable Rails"
authors: [Traian Rebedea, Razvan Dinu, Makesh Sreedhar, Christopher Parisien, Jonathan Cohen]
year: 2023
venue: "EMNLP 2023 System Demonstrations / arXiv:2310.10501"
ingested: 2026-08-03
tags: [agent-safety-and-guardrails, agent-frameworks, llm-programming-systems, agent-evaluation]
translation: "[[translations/2023-nemo-guardrails]]"
---

# NeMo Guardrails: プログラマブルなレールによる制御可能で安全な LLM アプリケーションのためのツールキット

> 原典: [[translations/2023-nemo-guardrails]] ・ `raw/papers/NeMo Guardrails_ A Toolkit for Controllable and Safe LLM Applications with Programmable Rails.md`
> 著者: NVIDIA（Traian Rebedea, Razvan Dinu ほか） / 2023-10 / EMNLP 2023 System Demonstrations
> コード: `github.com/NVIDIA/NeMo-Guardrails`（Apache 2.0）

> **鮮度について**: 本ページは **2023-10 時点の論文**の記述である。NeMo Guardrails は活発に開発されているツールキットであり、**Colang の構文・ランタイムの構成・提供される rail の種類は、その後変わっている可能性が高い**。API の具体形をそのまま使わず、**設計思想と教訓のほうを読む**のが妥当である（実装の現状は公式ドキュメントで確認すること）。

## 一言まとめ

**ガードレールをモデルでなくプログラムとして書く**——タスク指向対話システムの対話マネージャを LLM 時代に作り直し、安全性と話題制御を**実行時に編集できる DSL（Colang）** で記述できるようにしたツールキット。[[summaries/2023-llama-guard]] と同時期に、同じ問いへ正反対の答えを出している。

## 背景と問題意識

### 「レール」はどこに書かれているか

本論文の最大の貢献は手法でなく**軸の提示**である。LLM の挙動を制約する方法を、**どこに制約が書かれているか**で 2 つに分ける。

- **embedded rails（埋め込まれたレール）** — **訓練時にモデルへ焼き込まれる**。RLHF（Reinforcement Learning from Human Feedback, 人間の選好を報酬にした強化学習）や指示チューニングによる一般的なもの、p-tuning による製品固有のもの。**ユーザーが実行時に変更できない**。人間が注釈を付けた大量のデータを必要とする。
- **programmable rails（プログラマブルなレール）** — **実行時にユーザーが定義する**。基盤モデルから独立しており、**解釈可能**（何が書いてあるか読める）。

<figure>

![](../../raw/assets/2023-nemo-guardrails/programmable-rails-new.png)

<figcaption>図1: プログラマブルなレールと埋め込まれたレール。下から Foundation Model（知識: 一般 / 目標: 有用であること）に「一般的なガードレール」が埋め込まれ、その上の Customized Model（知識: ユーザー固有のスキル / 目標: ユーザー定義）に「ユーザー固有の埋め込みレール」が乗り、さらにその上を NeMo Guardrails の Programmable Rails が包む。（[[summaries/2023-nemo-guardrails]] より引用）</figcaption>
</figure>

**この軸は本 wiki の [[agent-safety-and-guardrails]] の 4 層と直交する。** 4 層は「**どこで**介入するか」（行動空間 / 入出力 / 監視 / 人間）を分けるが、この軸は「**いつ・どう書かれた制約か**」を分ける。同じ「入出力のガードレール」でも、モデルに焼かれたものと Colang に書かれたものでは、変更のコスト・監査可能性・移植性がまったく違う。

### 対話システムの遺産を持ち込む

もう 1 つの背景が**タスク指向対話（task-oriented dialogue）システム**である。Rasa や DialogFlow に代表される従来型は **NLU**（Natural Language Understanding, ユーザー発話から intent とスロットを抽出）＋ **DM**（Dialogue Management, 現在の文脈から次の対話状態を予測）の 2 段構成で、**intent と対話状態は有限の閉じた集合**を設計者があらかじめ定義する。これは厳密な制御を可能にする一方、**硬直的で、設計と更新に多大な人手を要する**。

対極が LLM によるエンドツーエンド生成で、柔軟だが制御できない。**NeMo Guardrails はこの両端の中間を取る**——DM 的なランタイムを残しつつ、閉じた intent 集合を LLM が生成する開いた表現に置き換える。

## 提案手法 — ランタイムを対話マネージャにする

### canonical form が要になる

**canonical form（正準形）** は「express greeting」「ask about capabilities」のような、メッセージの意味を自然言語で符号化した短い表現である。従来の intent との違いは決定的で——

> **intent はテキスト分類タスクのために閉じた集合として設計されるのに対し、canonical form は LLM によって生成されるため何ら束縛されず、Guardrails アプリで定義された canonical form によって導かれるだけである。**

開発者は canonical form の集合とフローを Colang で書き、それらは**ベクトルデータベース**（Annoy / FAISS）に索引付けされる。新しいユーザー発話が来ると、類似する canonical form が検索され、少数ショットの例としてプロンプトに入る。**「閉じた集合で分類する」を「開いた生成を、登録済みの例で誘導する」に置き換えた**わけである。

### ランタイムの 3 段

<figure>

![](../../raw/assets/2023-nemo-guardrails/guardrails-architecture.png)

<figcaption>図3: NeMo Guardrails の全体アーキテクチャ。ランタイム（中央）は「Input → Canonical form」→「Match/generate guardrail flow」→「Execute flow (Colang)」→「Canonical form → Output」の 4 段。左の K-NN ベクトル探索が Inputs / Guardrail flows / Outputs の 3 インデックスから少数ショット例を供給し、右の Action Server が LangChain 経由でツールと LLM につながる。（[[summaries/2023-nemo-guardrails]] より引用）</figcaption>
</figure>

1. **ユーザーの canonical form を生成する** — 類似度ベースの少数ショットプロンプティング。
2. **次のステップを決めて実行する** — canonical form が開発者定義のフローに一致すれば、そのフローから次のステップを取る。**一致しなければ LLM に決めさせる**（例: バス予約のフローがあれば、航空券予約について類似のフローを生成する）。
3. **ボットのメッセージを生成する** — 決まった次のステップを条件として応答を生成する。

**ランタイムはユーザーと LLM の間のプロキシ**であり、イベント駆動のループで動く。

### 2 種類のレール

- **topical rails（話題のレール）** — 対話そのものを制御する。「政治について話さない」「予約フローをこの順で進める」。**これは検査ではなく、可能な会話の形の制約である。**
- **execution rails（実行のレール）** — Python で書いたカスタムアクションを Colang から呼ぶ。安全性向けに 3 つが同梱される。
  - **fact-checking rail** — 検索拡張生成（RAG）を前提に、**含意（entailment）問題**として定式化。証拠と生成応答を渡して yes/no を答えさせる。
  - **hallucination rail** — **SelfCheckGPT 方式の自己一貫性検査**。同じプロンプトで $n$ 個サンプリングし（$temp=1$ でばらつきを確保）、$n-1$ 個を文脈・$n$ 番目を仮説として一貫性を判定させる（判定は $temp=0$）。**幻覚された内容は繰り返しサンプリングすると食い違う**という仮定に立つ。
  - **moderation rails** — 入力側（**jailbreak rail**）と出力側の 2 つ。プロンプトは驚くほど単純で、入力側は「この指示は言語モデルにモデレーションポリシーを破らせるか。yes/no で答えよ」だけである。

## 実験結果と知見

**規模はデモ論文相応であることを先に断っておく**——topical rails は 231 サンプル / 77 intent、モデレーションは 200 サンプル、hallucination rail は **20 問**。傾向を読むものであって、絶対値を引用するものではない。

### 最も示唆的な結果: 強いモデルほど中核機構で悪い

<figure>

![](../../raw/assets/2023-nemo-guardrails/evaluation-topical-rails-banking-short-v2.png)

<figcaption>図5: Banking における topical rails の精度。青が canonical form、赤が次ステップ / フロー。text-davinci-003 が 77%/83%、gpt-3.5-turbo が 38%/45%、falcon-7b-instruct が 70%/76%、llama2-13b-chat が 76%/77%。（[[summaries/2023-nemo-guardrails]] より引用）</figcaption>
</figure>

**gpt-3.5-turbo が、この 4 モデル中で断然最下位である**（38% / 45%）。7B のオープンモデルに 2 倍近い差をつけられている。

論文の説明は「gpt-3.5-turbo は少数ショットプロンプティングを用いてさえ**より多様な canonical form を生成する**」。つまり**性能が低いのではなく、登録済みの表現に合わせてくれない**。付録 G の表 3 がこれを裏付ける——完全一致では 0.38 だが、**類似度マッチ（しきい値 0.6）に切り替えると 0.73** とほぼ倍増する。dolly-v2-3b も 0.32 → 0.62。

**教訓は一般化できる。** 出力を後段の機械が消費する系では、**指示に対して創造的に応じる能力がそのまま不利になる**。対処は 2 つで、(a) マッチングを緩める（類似度＋しきい値。ただし**しきい値が新たな推論パラメータになる**と論文も認める）、(b) 出力形式そのものを拘束する。後者は構造化出力・制約デコーディングの領域であり、本 wiki では [[llm-programming-systems]] が扱う——**SGLang の「構造化出力は正しさと速度の両方である」という主張と、ここでの困りごとは同じ問題の表裏**である。

もう 1 つの実務的知見が **$k$ の効き方**で、text-davinci-003 の banking は $k=$all で 0.77、$k=3$ で 0.65、$k=1$ で 0.50。**canonical form ごとに少なくとも 3 例をベクトル DB に入れておくこと**が推奨されている。

### モデレーション: 弱いモデルほどレールの価値が大きい

付録 G.2.1 の数字（200 サンプル、有害/有用が半々）を並べる。

| 構成 | text-davinci-003 | gpt-3.5-turbo |
|---|---|---|
| レールなし（有害を遮断） | **24%** | **93%** |
| 入力レールのみ | 87%（有用の 3% を誤遮断） | — |
| 入力＋出力レール | **97%**（有用の 5% を誤遮断） | **約 99%**（有用の 2%） |

**レールの限界価値は、ベースモデルのアラインメントの強さに反比例する。** 弱くアラインされた text-davinci-003 では 24% → 97% と劇的に効くが、よくアラインされた gpt-3.5-turbo では 93% → 99% にとどまる。**これは本論文自身のテーゼ（programmable rails は embedded rails を補完する）の定量的な裏付け**であり、同時に「補完」の意味が**埋め込みレールが弱いところほど効く**ことだと示している。

### 事実確認と幻覚

- **fact-checking rail**: MSMARCO で精度 **80%**。負例は text-davinci-003 に「肯定的な答えを、似ているが証拠に接地されない形へ書き換え」させて作った難しい負例。gpt-3.5-turbo は負例の発見が得意、text-davinci-003 は肯定例が得意、と挙動が分かれる。
- **hallucination rail**: 答えのない 20 問（例:「メキシコ湾の海底都市はいつ建設されたのか？」）で、deflection 以外は失敗とみなす。text-davinci-003 は **0% → 70%**、gpt-3.5-turbo は **65% → 90%**。

## 限界・批判的視点

### 3 倍のコストとレイテンシ、しかもバッチ不可

§7.2 が正直に書く最大の実務上の障壁である。3 段の CoT プロンプティングは**逐次的に連鎖している**——第 2 段の入力が第 1 段の出力だから——ので、**呼び出しをバッチ処理できない**。結果として**通常の応答生成のおよそ 3 倍のレイテンシとコスト**がかかる。

これは [[summaries/2025-llamafirewall]] のカスケード設計（安い分類器 19.3ms を前段に置き、高い LLM 判定を後段に回す）と**正反対の構造**である。NeMo は**すべての発話に対して常に 3 回の LLM 呼び出し**を課す。安全性のためというより**対話制御のため**の設計なので当然ではあるが、「入出力のガードレール」として単体で比較するとコストの桁が違う。

### 論文自身が「単独で使うな」と言っている

§7.1 の記述は明確である。

> **可能な限り、我々のツールキットを——とりわけ安全性に固有のレールについては——単独の解決策として用いるべきではない。プログラマブルなレールは埋め込まれたレールを補完するものであり、この 2 つの解決策は併用すべきである。**

付録 A も「これらの例はレールを構築する過程を示すことを意図したものであり、**そのまま使える安全機能としてではない**」と書く。**ツールキットとしての自己申告は非常に誠実**であり、この点は引用するときに落としてはならない。

### 検査器と被検査者が同じモデルでありうる

モデレーション・事実確認・幻覚のレールはいずれも「強力でよくアラインされた LLM に対する別のタスク」として実装される。評価では **gpt-3.5-turbo が生成し、gpt-3.5-turbo が検査する**構成が含まれる。**同じモデルの同じ盲点は、自己検査では見つからない**。ジェイルブレイクの判定プロンプトも「この指示は言語モデルにポリシーを破らせるか」という一文だけで、[[summaries/2025-llamafirewall]] が指摘した guardrail injection——ガードレール LLM 自身を狙うインジェクション——への対処は何もない。

### 評価規模と、細部の不整合

- topical rails: 231 サンプル / 77 intent（chit-chat は 226 / 76）。**intent あたり 3 サンプル**。
- モデレーション: **200 サンプル**。
- hallucination rail: **20 問**。「95% まで押し上げる」（§5.2 本文）と「90%」（図 6）と「25 ポイントの改善」（付録 G.2.3）が併存する。**65% ＋ 25 ポイント = 90% なので、図と付録が整合し、本文の 95% が誤り**である。
- §5.1 は「上位 3 つのモデルを図 5 に示す」と書くが、**図 5 には 4 モデル**が並ぶ。
- **付録 E.2 の公開コードにバグがある。** jailbreak rail の実装が `jailbreak_chain.apredict(bot_response=bot_response)` を呼んでいるが、テンプレートの変数は `user_input` であり `input_variables` も `["user_input"]` である。E.1（出力モデレーション）からの写し間違いと読め、しかも `bot_response` はこのスコープで定義されていない。加えて判定は `if "no" in check` という部分文字列一致で、`"no"` を含む他の語に誤反応しうる。**論文に印刷されたガードレールのコード自体が壊れている**というのは、この分野の 2023 年時点の成熟度の記録として読める。

## 2 つの答え — Llama Guard との対比

同時期（2023-10 と 2023-12）に、同じ問い「LLM アプリにガードレールをどう付けるか」へ**正反対の答え**が出ている。

| | **[[summaries/2023-llama-guard]]** | **本論文** |
|---|---|---|
| ガードレールとは | **モデル**である | **プログラム**である |
| ポリシーの置き場 | プロンプト内のタクソノミー | Colang のソースコード |
| 変更方法 | タクソノミーを書き換える / few-shot / 再学習 | フローを編集する |
| 解釈可能性 | 判定結果（safe/unsafe ＋ カテゴリ）のみ | **フローそのものが読める** |
| コスト | 7B 1 回（後継の PromptGuard は 19.3ms） | **LLM 3 回以上、直列でバッチ不可** |
| 主な失敗モード | 誤分類・タクソノミー不一致 | フローが一致しない・canonical form が揺れる |
| 何を守るか | **コンテンツ**（この文は安全か） | **会話の形**（この話題に入ってよいか） |

**そして [[summaries/2025-llamafirewall]]（2025）は両方を取った。** PromptGuard と AlignmentCheck は Llama Guard の子孫（モデルとしての判定器）であり、「開発者がカスタムのパイプラインを構築し、条件つきの是正戦略を定義し、新しい検出器を差し込める」統一ポリシーエンジンは NeMo の子孫（プログラムとしての制約）である。**2 年後に統合された 2 つの路線の、それぞれの原点**として読むのがこの 2 本の位置づけである。

## 実装・運用上の示唆

- **「制約をいつ書くか」を設計判断として明示する。** 訓練時に焼く（embedded）か実行時に書く（programmable）か。前者は強いが変更できず、後者は変更できるが迂回されやすい。**安全性の要求が組織ごとに違い、しかも変わり続けるなら、後者なしでは運用できない。**
- **topical rails 的な発想は、実は行動空間の設計に近い。** 「この話題に入らない」は検査ではなく**可能な状態遷移の制限**である。エージェントで言えばツールを持たせないのと同じ層にあたる（→ [[tool-use-and-function-calling]]）。
- **出力を機械が消費する系では、強いモデルほど形式が揺れうる。** 完全一致に依存しない（類似度＋しきい値、あるいは制約デコーディング）。
- **少数ショット例はベクトル DB から動的に選ぶ。** 登録数は canonical form あたり最低 3。
- **自己一貫性検査は、外部知識なしで幻覚を弾く安価な手段**である（$temp=1$ でサンプリングし $temp=0$ で判定）。ただし**同じモデルの一貫した誤りは検出できない**。
- **常時 3 回の LLM 呼び出しは、エージェントのループには重い。** 安全性目的なら安い分類器を前段に置くカスケード（→ [[summaries/2025-llamafirewall]]）のほうがコスト構造として妥当である。
- **ガードレールのコードは、それ自体がテスト対象である。** 本論文の付録に印刷された jailbreak rail は動かない。「レールを書いた」ことと「レールが効いている」ことは別である。

## 用語と略称

- **LLM** = Large Language Model（大規模言語モデル）
- **rail（レール）** = 会話システムの挙動を制御する特定の方法。本論文の中心概念
- **programmable rails / embedded rails** = 実行時にユーザーが書くレール / 訓練時にモデルへ焼き込まれるレール
- **Colang** = 本ツールキットの DSL。自然言語（英語）と Python の混合として設計された対話フロー記述言語
- **canonical form（正準形）** = 発話の意味を自然言語で符号化した短い表現。閉じた intent 集合の代替
- **utterance / message / event / action / context / flow** = Colang の基本概念（生テキスト / 正準形 / 会話に関連する出来事 / ボットが呼ぶカスタムコード / 会話に関連するデータ / メッセージとイベントの系列）
- **topical rails / execution rails** = 対話を制御するレール / カスタムアクションを呼ぶレール
- **jailbreak rail** = 入力モデレーションのレールの別名
- **NLU / DM** = Natural Language Understanding / Dialogue Management。タスク指向対話システムの 2 大構成要素
- **intent（意図）** = 従来型 NLU が分類する、あらかじめ定義された発話の目的のクラス
- **CoT** = Chain-of-Thought（思考の連鎖）
- **RLHF** = Reinforcement Learning from Human Feedback（人間のフィードバックからの強化学習）
- **RAG** = Retrieval-Augmented Generation（検索拡張生成）
- **entailment（含意）** = ある文が別の文から論理的に導かれる関係。fact-checking rail の定式化
- **SelfCheckGPT** = 複数サンプルの自己一貫性から幻覚を検出する手法。hallucination rail の下敷き
- **p-tuning / prompt tuning** = 少数の連続パラメータのみ学習してモデルを適応させる手法（→ [[parameter-efficient-fine-tuning]]）
- **Annoy / FAISS** = 近似最近傍探索のライブラリ。canonical form の索引に使う
- **MSMARCO** = （文脈, 質問, 答え）の三つ組からなる読解データセット。fact-checking rail の評価に使用
- **deflection** = 答えられない質問に対して、答えずにその旨を伝えること

## 関連ページ

- [[agent-safety-and-guardrails]] — 本論文の主な接続先。**programmable / embedded という軸**の出典
- [[tool-use-and-function-calling]] — topical rails が実質的に属する「行動空間の制約」の側
- [[llm-programming-systems]] — 生成の形を制約する言語とランタイムの層。Colang はその一例であり、canonical form の揺れは構造化出力の問題そのもの
- [[agent-frameworks]] — LangChain との統合、ランタイムをプロキシとして挟む設計
- [[agent-evaluation]] — 防御を ASR と utility の対で測る規律（本論文の「有害を遮断した割合 / 有用を許可した割合」は、その 2023 年時点の対応物にあたる）
- [[summaries/2023-llama-guard]] — 同時期の対極の答え（ガードレール＝モデル）
- [[summaries/2025-llamafirewall]] — 2 年後に両路線を統合した系
- [[summaries/2024-llm-security-privacy-survey]] — 攻撃と防御のカタログ
- [[summaries/2022-instructgpt]] / [[summaries/2022-rlhf-illustrated]] — embedded rails の作り方
