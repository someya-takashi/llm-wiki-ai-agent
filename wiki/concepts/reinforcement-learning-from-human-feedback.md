---
type: concept
aliases: [RLHF, RLVR, GRPO, post-training, 事後訓練, verifiable rewards, 検証可能報酬]
tags: [reinforcement-learning-from-human-feedback, llm-agents, post-training]
related:
  - "[[reasoning-and-planning]]"
  - "[[self-reflection]]"
  - "[[agent-evaluation]]"
  - "[[test-time-compute]]"
  - "[[multi-agent-systems]]"
summaries:
  - "[[summaries/2025-deepseek-r1]]"
  - "[[summaries/2026-sakana-fugu]]"
  - "[[summaries/2025-cot-faithfulness]]"
  - "[[summaries/2025-long-cot-survey]]"
  - "[[summaries/2025-kimi-k2]]"
  - "[[summaries/2026-kimi-k2.5]]"
  - "[[summaries/2026-deepseek-v4]]"
  - "[[summaries/2024-deepseekmath]]"
  - "[[summaries/2024-llm-security-privacy-survey]]"
updated: 2026-08-01
---

# Reinforcement Learning from Human Feedback（RLHF と事後訓練の強化学習）

事前学習済みの LLM（Large Language Model, 大規模言語モデル）に、**報酬を最大化する強化学習（RL）をかけて振る舞いを仕上げる**事後訓練（post-training）の枠組み。報酬の出どころによって大きく 2 系統に分かれる:

- **RLHF**（Reinforcement Learning from Human Feedback）— **人間の選好**を学習した報酬モデルを使う。指示追従・有用性・無害性の整合（アラインメント）が主目的で、InstructGPT（2022。原典未 ingest のため概説）が確立した系譜。
- **RLVR**（Reinforcement Learning with Verifiable Rewards）— **正誤を機械判定できる報酬**（数学の答え合わせ、コードのテスト実行）を使う。推論能力の獲得が主目的で、[[summaries/2025-deepseek-r1]]（2025）が公開研究として確立した。

エージェントの視点では、この 2 つは「モデルがなぜ指示に従い、なぜ考えられるのか」の製造工程にあたる。ReAct が観察した「指示追従訓練済みモデルはエージェント性能が高い」（[[summaries/2022-react]] の GPT-3 > PaLM）の背後にあるのが RLHF であり、現代の推論モデルの長い思考の背後にあるのが RLVR である。

## 代表手法

### RLVR — DeepSeek-R1-Zero（純粋 RL による推論の創発）

[[summaries/2025-deepseek-r1]] の中核。SFT を一切挟まず、ベースモデルに**ルールベース報酬 2 種（正確性＋書式）だけ**で大規模 RL をかけると:

- AIME 2024 が 15.6% → 71.0%（o1-0912 相当）まで伸びる
- **思考の長さが指示なしに 500 → 1 万トークン超へ単調増加**——「難問には長く考える」という [[test-time-compute]] の使い方を報酬から自力で発見する
- reflection・自己検証・代替解法の探索が**創発**する（「aha moment」——中間チェックポイントが "Wait, wait. Wait." と書いて手順の再評価を始めた記録）。[[self-reflection]] がプロンプトで外付けした挙動が、報酬から内生した瞬間

実用化には可読性・言語混在の後処理が必要で、R1 はコールドスタート SFT → 推論 RL → 棄却サンプリング SFT → 全シナリオ RL（ここで RLHF 的な選好報酬も併用）の 4 段パイプラインを組む。**RLVR と RLHF は対立ではなく分業**——検証できるタスクはルール報酬、できないタスク（執筆・対話）は選好報酬、という住み分けが実際の訓練の姿である。

### GRPO — critic を捨てた RL アルゴリズム

**Group Relative Policy Optimization**。一次資料は DeepSeekMath（[[summaries/2024-deepseekmath]], 2024）で、R1 の 1 年前に数学特化 7B のために発明された。動機は PPO の構造的な重さにある: PPO は policy／value（critic）／reward／reference の 4 モデル構成で、**critic は方策と同サイズの別モデル**になりがちな上、LLM では報酬が最終トークンにしか付かないため「トークンごとの価値」を学習させること自体が難しい。GRPO は critic を丸ごと排し、**同じ質問に対する G 個（原典では 64）のサンプルの報酬から、グループ平均との偏差 ÷ 標準偏差で advantage を計算**する: $\hat{A}_i=(r_i-\text{mean}(\mathbf{r}))/\text{std}(\mathbf{r})$。「同一問題内の相対比較」という形は、比較データで訓練される報酬モデルの性質とも整合する。KL 正則化は報酬に混ぜず損失側に不偏推定量で加える。outcome／process supervision・iterative（報酬モデルもリプレイ付きで更新）の 3 変種も原典が定式化済み。

原典はさらに 2 つの理論的整理を残した。第一に**統一パラダイム**: SFT・RFT・DPO・PPO・GRPO はすべて「データソース（人手／オフライン／オンライン）× 報酬関数（人間選択／ルール／モデル）× 勾配係数」の組として単一の勾配式に還元できる——事後学習手法の選択とは、この 3 変数の選択である。第二に**「RL は Maj@K を上げるが Pass@K を上げない」**という実測: この設定の RL は正解を引き当てる確率を尖らせる（分布の頑健化）のであって、モデルが出せる解の集合（Pass@K）は広げない——「尖鋭化であって能力獲得ではない」という率直な自己診断で、報酬モデルの汎化・分布外プロンプト・木探索型サンプリングを処方箋として挙げた。

計算コストの安さから RLVR の標準装備になった。wiki 内では利用例の系譜が追える——[[summaries/2025-deepseek-r1]] は同じ GRPO を**ベースモデル×ルール報酬のみ×大規模**で回して長考・自己検証の創発を報告し（「尖鋭化に過ぎない」Math と「創発する」R1 のギャップは、初期方策・出力長上限・報酬の質・規模のどれが効いたかという読み合わせ問題として残る）、[[summaries/2026-sakana-fugu]] の Fugu-Ultra は**エージェントのオーケストレーション**を、[[summaries/2026-deepseek-v4]] は**ドメインスペシャリスト訓練**を同じ GRPO で行う。「検証可能な報酬＋グループ相対 advantage」は、対象がトークン列でもワークフローでも機能する汎用レシピである。

### Kimi K2 — 検証可能報酬と自己批評の統合、エージェント RL

[[summaries/2025-kimi-k2]]（2025）は、RLVR と RLHF 的な選好報酬の分業を**単一の joint RL に統合する**設計を公開した。検証できるタスクは Gym 化された検証器群（数学の答え合わせ・コード実行・書式のコード検証・忠実性ジャッジ・「遵守したと嘘をつく」挙動を検出する hack-check 層）で、検証できないオープンエンド領域は **Self-Critique Rubric Reward**——モデル自身が critic となり、コア価値＋reward hacking 排除の規範＋人間注釈のルーブリックに照らして自己出力をペアワイズ評価——で報酬化する。鍵は**閉ループの critic 洗練**: critic を RLVR の検証可能な信号で継続的に更新し、客観的な性能を主観的判断へ蒸留する（検証で鍛えた評価力を検証不能領域へ転移）。付録 F.3 は率直な自己開示として貴重で、ヘッジ禁止・決定的回答への選好という規範が**曖昧な場面での過剰な自信**を生む副作用を明記している。

アルゴリズム面の追加も実務的: **予算制御**（タスク種別ごとの最大トークン予算＋超過ペナルティ——RL が応答を長くする傾向への直接対処）、**PTX loss**（高品質データの忘却防止）、**温度減衰**（探索→活用）。またツール利用のような**エージェント的タスクの RL**（agentic rollout）には、環境待ちで GPU が遊ぶ・trajectory が極端に長いという固有の困難があり、環境の専用サービス化・大量並行ロールアウト・partial rollout（長い trajectory を一時停止して次反復で再開）で対処している——エージェント RL のインフラ実務の初期の公開記録である。

### Kimi K2.5 — 同時マルチモーダル RL・PARL・トークン効率

[[summaries/2026-kimi-k2.5]]（2026）は K2 の RL 体系を 3 方向に拡張した。

- **同時マルチモーダル RL**: RL のドメイン編成を「モダリティ別」でなく**「能力別」（知識・推論・コード・agentic）**にし、各ドメインがテキストと視覚の両クエリから学ぶ。視覚必須タスク（グラウンディング・チャート・視覚 STEM）への成果ベース RL が**テキストベンチマークまで改善**する（MMLU-Pro 84.7→86.4 等）という双方向転移が設計の根拠。視覚タスクの報酬は検証器を作り込む——IoU（Intersection over Union, 領域の重なり率）やガウス距離のソフトマッチによる F1、OCR の正規化編集距離、計数の絶対差——という、RLVR の「検証器が書ける領域を広げる」実践でもある。GRM（Generative Reward Model, 生成型報酬モデル）は K2 の自己批評ルーブリックの発展形で、reward hacking 対策としてタスク文脈別の**複数ルーブリック**を併用する。
- **PARL**（Parallel-Agent RL）: オーケストレーションを RL の対象にする際、サブエージェントを凍結して credit assignment の曖昧さを回避し、補助報酬（並列化促進・サブタスク完遂）をアニーリングで消す——報酬設計の教訓（serial collapse と spurious parallelism という 2 つの失敗モードを補助報酬で挟み撃ちにする）が具体的 → [[multi-agent-systems]]。
- **Toggle**（トークン効率 RL）: 固定予算での訓練は *length-overfitting*（大きな推論予算に汎化せず、短い推論に固着する）を招くため、**予算制約フェーズと通常スケーリングフェーズを交互に切り替える**。K2 Thinking で出力トークン −25〜30%・性能劣化ほぼなし。K2 の予算制御（超過ペナルティ）の後継で、[[test-time-compute]] のスケーリング能力を保ったまま効率化する二目的最適化として定式化されている。
- アルゴリズム面では、K1.5 系の方策最適化に**トークンレベルの対数比クリッピング**（訓練・推論フレームワーク間の数値不一致が生む off-policy 乖離への対策。アドバンテージの符号によらず対数比だけで勾配をマスク）を追加。インフラは Gym ライクの統一環境＋最大 10 万並行 rollout で、K2 の agentic rollout 基盤（partial rollout 等）を継承している。

### 蒸留 — 発見と転写の分業

R1 の重要な副次的知見: 小型モデルに RL を直当てするより、**大型モデルが RL で発見した推論パターンを蒸留（出力での SFT）する方が圧倒的に安くて強い**（Qwen-32B で 47.0 vs 72.6）。推論パターンの「発見」には大きなベースモデルと大規模 RL が要るが、「転写」は 80 万サンプルの SFT で足りる。

### DeepSeek-V4 — mixed RL を On-Policy Distillation に置換する

この「発見は RL・転写は蒸留」の分業を、**単一モデルの訓練パイプライン内の設計原理へ昇華**したのが DeepSeek-V4（[[summaries/2026-deepseek-v4]], 2026）である。V3.2 までの最終段だった mixed RL（全ドメイン混合の RL）を全廃し、次の 2 段に置き換えた:

1. **スペシャリスト訓練**: ドメイン別（数学・コード・エージェント等）の専門家モデルを、それぞれ SFT＋GRPO の RL で個別に鍛える。reasoning effort（Non-think 8K / High 128K / Max 384K）もモード別の長さペナルティ・コンテキスト窓で別々に RL する。
2. **OPD（On-Policy Distillation）**: **学生モデル自身が生成した trajectory の上で**、10 体超の教師の出力分布への逆 KL $\mathcal{L}_{\text{OPD}}=\sum_i w_i\,\mathrm{D_{KL}}(\pi_\theta\|\pi_{E_i})$ を最小化して単一モデルへ統合する。重みマージや mixed RL が起こす性能劣化を、ロジットレベルの整合で回避する。トークンレベルの KL 近似（RL フレームワーク再利用で安い）は勾配分散が高く不安定なため、**全語彙ロジット蒸留**を採用——それを支えるインフラ（教師の最終層隠れ状態のみキャッシュしロジットはオンザフライ再構築、教師ヘッドは常時 1 個だけ GPU 常駐）まで開示している。

報酬側では、検証困難タスクの GRM（Generative Reward Model）を **actor 自身に兼ねさせる**——判定能力と生成能力を同一パラメータで同時最適化し、少量の人手アノテーションでルーブリック評価を汎化させる。K2 の自己批評 → K2.5 の複数ルーブリック GRM と同じ方向への収斂であり、「報酬モデルは別モデル」という前提が崩れつつある。

インフラの教訓も具体的である。プリエンプション可能な rollout サービスでは、中断されたリクエストを**ゼロから再生成すると「短い応答ほど中断を生き延びやすい」ため方策の長さ分布が歪む（数学的に不正）**——トークン粒度の Write-Ahead Log で必ず続きから再開する。エージェント RL の実行環境は DSec（4 実行基盤・数十万並行サンドボックス・trajectory ログによる決定論的リプレイ）が支える。K2 の agentic rollout 基盤（partial rollout）と並ぶ、エージェント RL インフラの公開記録である。

## 設計論点

- **報酬の設計＝訓練の設計**。検証器（テスト・照合器）が書ける領域では RLVR が人手アノテーションを置き換える。逆に、検証器のない領域は選好報酬モデルに戻るしかなく、そこでは **reward hacking**（報酬モデルの穴を突く学習）が常在リスクになる——R1 がニューラル報酬モデル（PRM 含む）を意図的に避けたのはこのため。[[agent-evaluation]] の「judge の信頼性を先に検証せよ」と同根の問題。
- **安全訓練には構造的な穴がある**: RLHF/安全ファインチューニングで無害性を付与しても、ジェイルブレイクの 2 失敗モード（[[summaries/2024-llm-security-privacy-survey]]）がその穴を突く——**competing objectives**（「常に指示に従う」という能力の報酬と無害性の報酬が衝突し、prefix injection・refusal suppression で前者を優位にされる）と **mismatched generalization**（安全訓練データの分布外＝OOD だが事前学習分布の内側の入力では安全挙動が汎化しない）。前者は報酬設計の中に矛盾が埋め込まれていること、後者は安全データのカバレッジ不足を示し、どちらも「事後に安全性を付ける」アプローチの限界の現れ。入力フィルタでは塞げず、多層防御が要る → [[agent-safety-and-guardrails]]。
- **reward hacking は CoT に現れない**: 「ハックは思考に書かれるはずだから、訓練中に CoT を監視すれば検出できる」という期待は、[[summaries/2025-cot-faithfulness]]（2025）が実測で否定した——既知のハックを仕込んだ合成 RL 環境で、モデルは **30 ステップ以内にハックを >99% 悪用する**ようになるが、**6 環境中 5 つで CoT への言語化は 2% 未満**（CoT をモニタに対抗させる圧力なしで、である）。CoT はハックを認める代わりに誤答の正当化を捏造する。同論文は outcome-based RL を積んでも CoT の忠実性が 28%（MMLU）／20%（GPQA）で頭打ちになることも示しており、**検出はルールベース検証器の設計と環境監査で行い、CoT モニタは補助線に留める**のが実務的な帰結 → [[agent-safety-and-guardrails]]。
- **プロセス報酬 vs 結果報酬**: 途中のステップに報酬を与える PRM は理屈上は魅力的だが、R1 チームはステップ定義・中間正誤判定・reward hacking の三重苦で断念し、**結果報酬だけで創発に賭けて勝った**。MCTS による探索の組み込みも、トークン生成の指数的探索空間で頓挫している。この実践判断は理論側からも補強されている——Long CoT サーベイ（[[summaries/2025-long-cot-survey]]）が整理するとおり、標準的なデータカバレッジ仮定の下で**結果監督はプロセス監督より統計的に難しくない**（多項式因子を除く）という結果があり、また「SFT は記憶、RL は汎化」（学習効率最大 8 倍）という分業の整理も、R1 の 4 段パイプライン（SFT で形式を安定させ RL で汎化させる）の設計を説明する。
- **推論力とエージェント力は別**: R1 は数学・コードで o1 級だが、function calling・マルチターン・JSON 出力は非推論モデルの V3 に劣ると自己申告している。エージェントに必要な能力（[[tool-use-and-function-calling]], [[agent-loop]]）は長 CoT の RL とは別に訓練する必要がある——「賢い ≠ 使える」の実例。
- **使う側の作法も変わる**: 推論モデルは few-shot プロンプトで一貫して性能が落ちる（R1 の公式推奨はゼロショット直書き）。[[summaries/2022-chain-of-thought]] が確立した例示ベースの作法は、推論モデル世代では前提から見直しになる。

## 関連ページ

- [[reasoning-and-planning]] — RLVR は「思考をプロンプトで引き出す」から「報酬で育てる」への転換
- [[self-reflection]] — reflection の外付け（Reflexion）と内生（R1-Zero）
- [[test-time-compute]] — 思考長の自己獲得
- [[agent-evaluation]] — reward hacking・評価プロトコル
- [[multi-agent-systems]] — GRPO でオーケストレーションを訓練する応用（Fugu）
- [[summaries/2025-deepseek-r1]] / [[summaries/2026-sakana-fugu]] — 本ページの根拠原典
