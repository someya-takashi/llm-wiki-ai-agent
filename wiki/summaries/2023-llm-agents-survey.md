---
type: summary
source_path: raw/papers/The Rise and Potential of Large Language Model Based Agents- A Survey.pdf
source_kind: paper
title: "The Rise and Potential of Large Language Model Based Agents: A Survey"
authors: [Zhiheng Xi, Wenxiang Chen, Xin Guo, Wei He, Yiwen Ding, et al. (Fudan NLP Group)]
year: 2023
venue: "arXiv 2309.07864"
ingested: 2026-08-01
tags: [llm-agents, agent-loop, tool-use-and-function-calling, agent-memory, multi-agent-systems, agent-evaluation, agent-safety-and-guardrails, survey]
translation: "[[translations/2023-llm-agents-survey]]"
---

# LLM ベースエージェントの興隆と可能性 — 分野の語彙を定めた古典サーベイ

> 原典: [[translations/2023-llm-agents-survey]] ・ `raw/papers/The Rise and Potential of Large Language Model Based Agents- A Survey.pdf`
> 著者・年: Xi et al.（Fudan NLP Group, 著者 30 名）・2023-09（arXiv:2309.07864v3, 本文 45 ページ・686 文献）

## 一言まとめ

LLM（Large Language Model, 大規模言語モデル）を「脳」に据えたエージェントという研究領域を、**brain（脳）/ perception（知覚）/ action（行動）の 3 モジュール枠組み**・**応用 3 分類（単一・マルチ・人間協調）**・**エージェント社会**という 3 層で最初に体系化した包括サーベイ。この wiki が扱う領域全体の見取り図を 2023 年秋時点で描いた「総論」の古典であり、以後の研究（この wiki の他の原典群を含む）はこの地図の各区画を深掘り・更新してきたと読める。

## 背景と問題意識

2023 年時点の診断はこうである: AI エージェント研究は記号エージェント → 反応エージェント → 強化学習（RL, Reinforcement Learning）エージェント → 転移・メタ学習と進んできたが、いずれも**特定タスク・特定能力の強化**に留まり、「多様なシナリオに適応できる汎用の出発点となるモデル」を欠いていた。そこに LLM が現れ、知識・指示理解・汎化・計画・推論の汎用能力を 1 つのモデルで示した——ならば LLM をエージェントの地位に引き上げ、知覚空間と行動空間を拡張すれば、汎用エージェントへの道が開けるのではないか。World Scope（NLP→汎用 AI の 5 段階: Corpus → Internet → Perception → Embodiment → Social）の語彙で言えば、「第 2 レベルにいる LLM を第 3〜5 レベルへ引き上げる」宣言である。

エージェント概念そのものは哲学（アリストテレス・ヒュームの「欲求・信念・意図を持ち行動する実体」）から AI へ輸入されたもので、本サーベイは「LLM は条件付き確率モデルにすぎず志向性を持たない」という反論と「文脈からエージェントの信念・欲求・意図の近似表現を推測しうる」という擁護の両論を丁寧に併記した上で、実用側の定義——**周囲を知覚し、意思決定し、行動する人工的実体**——を採る。LLM が脳に適する根拠は 4 性質: **自律性**（詳細指示なしのタスク遂行）・**反応性**（マルチモーダル知覚・ツールで環境に即応。ただし「考えてから動く」中間ステップの遅延つき）・**能動性**（推論・計画・目標指向の行動）・**社会的能力**（自然言語による協調・役割演技）。

## 提案する枠組み

### brain / perception / action の 3 モジュール

エージェントのワークフローは「perception が環境のマルチモーダル情報を表現へ変換 → brain が保存（知識・記憶）と意思決定（推論・計画）→ action がツール・身体で実行し環境に痕跡を残す → フィードバックを得て反復」。この wiki の語彙では [[agent-loop]]（観測→思考→行動）のモジュール分解版である。

- **brain**（§3.1）: 自然言語相互作用（多輪会話・意図と含意の理解）／知識（言語・常識・専門ドメイン。幻覚と知識陳腐化・モデル編集が課題）／**記憶**（過去の観測・思考・行動の系列。長文脈化・要約・ベクトル/DB 圧縮の 3 戦略と、Recency・Relevance・Importance の重みつき想起——後者は Generative Agents 由来で [[agent-memory]] の標準語彙になった）／**推論と計画**（CoT（Chain-of-Thought, 思考連鎖）系の計画定式化と、フィードバックによる計画内省——[[reasoning-and-planning]] と [[self-reflection]] の 2023 年時点の整理）／転移と汎化（zero-shot 汎化・ICL（in-context learning, 文脈内学習）・継続学習）。
- **perception**（§3.2）: テキスト・視覚（キャプション化 vs ViT（Vision Transformer）＋ Q-Former/射影層の整合）・聴覚（LLM をハブにしたツールのカスケード呼び出し・音声スペクトログラムの画像扱い）・その他（ジェスチャ・Lidar・GPS 等）。
- **action**（§3.3）: テキスト出力／**ツール使用**（ツールの理解 → デモとフィードバックからの使い方学習 → 自給自足のためのツール**作成**まで。専門性・解釈可能性・頑健性の補強としてのツール、という動機づけは [[tool-use-and-function-calling]] の原型）／**embodied action**（身体化された行動——観測・操作・航行。RL の限界（サンプル効率・汎化・報酬設計）を LLM の内的知識で補う路線。SayCan・PaLM-E・Voyager が代表）。

### 応用 3 分類（§4）

1. **単一エージェント**: タスク指向（Web シナリオ＝web navigation・生活シナリオ）・イノベーション指向（科学探究——ChemCrow 等。違法薬物合成の危険の警告つき）・**ライフサイクル指向**（Minecraft でのオープンワールド生存。スキルライブラリと反復プロンプトで生涯学習する Voyager が到達点）。
2. **マルチエージェント**: **協調的相互作用**を無秩序（自由発言＋統合役/多数決）と秩序だった協調（CAMEL の役割対・MetaGPT の SOP（標準作業手順）符号化）に二分し、**敵対的相互作用**（討論・tit-for-tat）を対置。幻覚が相互作用で増幅する危険・長い討論のコンテキスト限界・「誤った合意への収束」という 3 課題を 2023 年時点で明記している——後の [[summaries/2025-masft]] の失敗分類を先取りする指摘。
3. **人間–エージェント**: **instructor-executor パラダイム**（人間が指示・定量/定性フィードバック。教育・医療・メンタルヘルスの応用）と **equal partnership パラダイム**（共感的コミュニケータ・人間レベルの参加者——交渉ゲーム等）。人間関与の理由を「解釈可能性の保証（エージェント同士は理解不能な言語を生みうる）」と「制御可能性の保証」に整理。

### エージェント社会（§5）

個の振る舞い（入力・内在化・出力）と集団の振る舞い（正・中立・負——エージェントが効率のために他者や環境を破壊する観察まで）・人格（認知・感情知能・性格。CRT や MBTI での測定）を整理し、環境をテキスト・仮想サンドボックス・物理の 3 類型に分ける。シミュレート社会は **open / persistent / situated / organized** の 4 性質で特徴づけ、Generative Agents（Stanford の 25 人町。バレンタインパーティーの自発的組織）を筆頭に、社会ネットワークの情報伝播・倫理的意思決定（人狼・ゲーム理論）・政策シミュレーションへの応用と、予期せぬ社会的害・ステレオタイプ・プライバシー・過度の依存（Bing「Sydney」への愛着と FreeSydney 請願）というリスクを併記する。

## 評価と安全（§6）

- **評価 4 観点**: utility（タスク完了成功率＋効率）・sociability（言語堪能さ・協力交渉・ロールプレイング）・values（誠実・無害・文化適応）・**継続的に進化する能力**（継続学習・自己目的的学習・新環境への汎化）。「成功率だけでなく効率・価値観・進化まで測れ」という枠は、後の評価論（→ [[agent-evaluation]]）の先駆け。
- **安全 3 論点**: 敵対的頑健性（テキストの誤りで済む LLM と違い、**行動空間を持つエージェントでは敵対的攻撃が破壊的行動に直結**する）・信頼性（キャリブレーション・バイアス・幻覚）・その他（悪用・失業・人類の福祉への脅威）→ [[agent-safety-and-guardrails]]。
- **スケールアップ**（§6.4）: エージェント数の事前決定型 vs 動的スケーリング。通信網の複雑化・幻覚の増幅・協調の困難という課題つき。
- **未解決問題**（§6.5）: LLM は AGI への道かの賛否両論・仮想→物理の乖離・集合知・**Agent as a Service**（エージェントのクラウドサービス化）。

## 限界・批判的視点 — 2023 年の地図として読む

このサーベイの記述は **2023 年 9 月時点**のものであり、この wiki の観点では「歴史的な基準面」として読むのが正しい。以後の展開との主なずれ:

- **「エージェント＝プロンプトで組む」時代の前提**。当時の代表例は AutoGPT・CAMEL・MetaGPT で、能力はプロンプト設計と外装で引き出すものだった。その後、[[summaries/2025-kimi-k2]]/[[summaries/2026-kimi-k2.5]] のようにツール使用・並列協調を**訓練（データ合成＋ RL）でモデル内に作り込む**路線と、[[summaries/2025-deepseek-r1]] の RLVR（Reinforcement Learning with Verifiable Rewards, 検証可能報酬の強化学習）による推論の創発が主流化し、「brain の能力は所与、外装で拡張」という本サーベイの前提は半分だけ残った。
- **ハーネスという層の不在**。本サーベイの action・応用の記述には、ループ・コンテキスト管理・検証を束ねる実行基盤（harness）の設計論（[[summaries/2025-effective-harnesses]]・[[summaries/2026-harness-design]]・[[summaries/2026-meta-harness]]）がまだない。むしろ「LLM がどう効率的に行動を計画・活用するかは未解決」（§6.1）という指摘が、後のハーネス工学で埋められた空白を正確に指している。
- **MCP（Model Context Protocol）以前・computer use 以前**。ツール接続の標準化や GUI 直接操作（→ [[computer-use-agents]]）は射程外。web navigation の記述（HTML を読む・RL で操作を真似る）は 2 世代前の姿である。
- **評価の具体性**。評価 4 観点は枠組みとしては生きているが、SWE-bench・OSWorld・TerminalBench のような後発の標準ベンチマーク群やコスト併記の慣行（→ [[agent-evaluation]]）は未登場（AgentBench への言及が最初期の例）。
- サーベイ固有の限界として、686 文献の網羅性と引き換えに個々の手法の実験的裏づけは薄く、「〜できる」の多くが当時のデモ・単発研究に基づく。マルチエージェントとエージェント社会の記述は特に、その後の実測（MASFT の失敗分類・Anthropic の 15 倍トークン経済）で条件づけられた。

## 実装・運用上の示唆

いま読む価値は「何を作るか」の指示ではなく**語彙と分類の原点**にある。(1) brain/perception/action の分解は、エージェント設計を「どの能力を LLM 内在で・どれを外装で持つか」へ切り分ける最初の道具として今も有効。(2) 協調の「無秩序/秩序」・「協調/敵対」の二軸、人間関与の「instructor-executor/equal partnership」の二分は、[[multi-agent-systems]] のパターン整理の下敷きになる。(3) §6.3 の「行動空間を持つエージェントでは敵対的攻撃が破壊的行動になる」という指摘と、幻覚の相互作用増幅・誤った合意への収束の警告は、現在のガードレール設計（→ [[agent-safety-and-guardrails]]）でもそのまま通用する。

## 用語と略称

- **LLM** = Large Language Model（大規模言語モデル）／ **AGI** = Artificial General Intelligence（汎用人工知能。Strong AI とも）
- **brain / perception / action** = 本サーベイの 3 モジュール（制御中枢／マルチモーダル入力の変換／テキスト・ツール・身体による出力）
- **World Scope（WS）** = NLP→汎用 AI の 5 段階（Corpus・Internet・Perception・Embodiment・Social）
- **CoT** = Chain-of-Thought（思考連鎖。答えの前に理由づけを生成させる）→ [[reasoning-and-planning]]
- **ICL** = In-Context Learning（文脈内学習。few-shot 例示からの学習）
- **RL / DRL** = （深層）強化学習／ **HRL** = 階層的強化学習
- **embodied action** = 身体化された行動（観測・操作・航行）。**Embodiment hypothesis** = 知能は環境との相互作用から生じるという仮説
- **instructor-executor / equal partnership** = 人間–エージェント関与の 2 パラダイム（指示者と実行者／対等な参加者）
- **disordered / ordered cooperation** = 無秩序な協調（自由発言）／秩序だった協調（規則・逐次発言）
- **open / persistent / situated / organized** = シミュレート社会の 4 性質
- **CRT / MBTI / Big Five** = 認知反射テスト／性格類型指標（エージェントの人格測定に流用）
- **AaaS / LMaaS** = Agent as a Service / Language Model as a Service
- **Q-Former** = 視覚エンコーダと LLM を整合させる学習可能クエリの Transformer（BLIP-2）
- 代表システム: **AutoGPT**（自律ループの初期実装）・**Voyager**（Minecraft の生涯学習・スキルライブラリ）・**CAMEL**（役割対の協調）・**MetaGPT**（SOP 符号化）・**Generative Agents**（シミュレート町）・**SayCan / PaLM-E**（ロボット接地）

## 関連ページ

- [[llm-agents]] — 本サーベイを主要根拠とする総論ハブ（本 ingest で新設）
- [[agent-loop]] / [[reasoning-and-planning]] / [[self-reflection]] — brain の各論
- [[tool-use-and-function-calling]] / [[agent-memory]] / [[context-engineering]] — action・記憶の各論
- [[multi-agent-systems]] — 協調/敵対・エージェント社会の発展先
- [[agent-evaluation]] — 評価 4 観点の後継
- [[agent-safety-and-guardrails]] — §6.3 の発展先
- [[summaries/2022-react]] / [[summaries/2023-reflexion]] / [[summaries/2023-memgpt]] — 本サーベイが「部品」として整理した同時代の一次研究
