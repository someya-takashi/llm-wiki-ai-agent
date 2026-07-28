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
updated: 2026-07-26
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

**Group Relative Policy Optimization**。同じ質問に対して G 個の出力をサンプリングし、**グループ内の報酬の平均・標準偏差から advantage を計算**することで、方策と同サイズになりがちな critic（価値推定）モデルを丸ごと排した。計算コストの安さから RLVR の標準装備になった。wiki 内では利用例が 2 つあり、対比が面白い——[[summaries/2025-deepseek-r1]] は**モデルの推論**を GRPO で育て、[[summaries/2026-sakana-fugu]] の Fugu-Ultra は**エージェントのオーケストレーション**を同じ GRPO で育てた。「検証可能な報酬＋グループ相対 advantage」は、対象がトークン列でもワークフローでも機能する汎用レシピである。

### 蒸留 — 発見と転写の分業

R1 の重要な副次的知見: 小型モデルに RL を直当てするより、**大型モデルが RL で発見した推論パターンを蒸留（出力での SFT）する方が圧倒的に安くて強い**（Qwen-32B で 47.0 vs 72.6）。推論パターンの「発見」には大きなベースモデルと大規模 RL が要るが、「転写」は 80 万サンプルの SFT で足りる。

## 設計論点

- **報酬の設計＝訓練の設計**。検証器（テスト・照合器）が書ける領域では RLVR が人手アノテーションを置き換える。逆に、検証器のない領域は選好報酬モデルに戻るしかなく、そこでは **reward hacking**（報酬モデルの穴を突く学習）が常在リスクになる——R1 がニューラル報酬モデル（PRM 含む）を意図的に避けたのはこのため。[[agent-evaluation]] の「judge の信頼性を先に検証せよ」と同根の問題。
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
