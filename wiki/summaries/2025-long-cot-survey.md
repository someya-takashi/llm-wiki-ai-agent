---
type: summary
source_path: "raw/papers/Towards Reasoning Era_ A Survey of Long Chain-of-Thought for Reasoning Large Language Models.md"
source_kind: paper
title: "Towards Reasoning Era: A Survey of Long Chain-of-Thought for Reasoning Large Language Models"
authors: [Qiguang Chen, Libo Qin, Jinhao Liu, Dengyun Peng, Jiannan Guan, Peng Wang, Mengkang Hu, Yuhang Zhou, Te Gao, Wanxiang Che]
year: 2025
venue: "arXiv:2503.09567"
ingested: 2026-07-26
tags: [test-time-compute, reasoning-and-planning, self-reflection, reinforcement-learning-from-human-feedback, long-cot, survey]
translation: "[[translations/2025-long-cot-survey]]"
---

# Towards Reasoning Era: A Survey of Long CoT（Chen et al., 2025）

> 原典: [[translations/2025-long-cot-survey]] ・ `raw/papers/Towards Reasoning Era_ A Survey of Long Chain-of-Thought for Reasoning Large Language Models.md`
> 著者・年・出典: Qiguang Chen ほか（ハルビン工業大学・中南大学ほか）・2025・arXiv:2503.09567（813 文献の大型サーベイ）

## 一言まとめ

o1 / R1 が実践で見せた「長く考えるモデル」を、**Long CoT（長い思考連鎖）という独立した推論パラダイム**として初めて体系化したサーベイ。Long CoT を **deep reasoning（深い推論）・extensive exploration（広範な探索）・feasible reflection（実行可能な反省）の 3 特性**で形式的に定義し、6 つの経験的現象（創発・推論境界・overthinking・test-time scaling・PRM vs ORM・aha moment）と、膨大な手法群の分類法を与えた——推論モデル時代の**地図**にあたる原典である。

## 背景と問題意識

CoT（[[summaries/2022-chain-of-thought]]）以来、「考えてから答える」手法は爆発的に増えたが、o1 / R1（[[summaries/2025-deepseek-r1]]）の登場で状況が質的に変わった: 思考が数千〜数万トークンに伸び、反省や探索を**ひとつの生成の中で**行うようになった。しかしこの「Long CoT」が従来の Short CoT と何が違うのか、共通の定義はなく、「test-time scaling は有効だ」対「overthinking は有害だ」という議論が、用語の整理のないまま並走していた。本サーベイはこの混乱に、**形式的定義・現象の整理・手法の分類法**という 3 点セットで秩序を与えようとする。

## 提案手法 / 主張

### Long CoT の形式的定義 — 3 つの制約緩和

<figure>

![](../../raw/assets/2025-long-cot-survey/x3.png)

<figcaption>図2（再掲）: Short CoT との違いを規定する 3 特性。deep reasoning（推論境界の拡張）・extensive exploration（並列分岐）・feasible reflection（フィードバックと洗練）を統合したものが Long CoT。</figcaption>
</figure>

Short CoT を「論理ノードの列が (a) 上限 $\mathcal{B}_s$ 以下、(b) 一直線、(c) 再訪なし」という 3 制約つきの生成として形式化し、Long CoT を**それぞれの制約の緩和**として定義する:

1. **Deep Reasoning** — ノード数上限の拡張（$\mathcal{B}_s \ll \mathcal{B}_l$）。深さが足りないと未解決・幻覚に落ちる。
2. **Extensive Exploration** — 分岐の許容（並列に $m$ 個のノードを探索）。不確実・多解の問題で決定的。
3. **Feasible Reflection** — 再訪の許容（フィードバックで誤り箇所へ戻り、refinement で修正）。

重要な論点として、**ToT や GoT は Long CoT ではない**——ToT は探索だけ、GoT は探索＋反省だが深い推論を欠く。3 特性が**単一の生成システムに統合**されて初めて Long CoT と呼ぶ、という線引きが与えられる（CoT → ToT → Reflexion と個別に発達した能力の統合として推論モデルを見る史観）。

### 6 つの現象 — test-time compute の経験則

<figure>

![](../../raw/assets/2025-long-cot-survey/x4.png)

<figcaption>図4（再掲）: Long CoT の外的挙動の 6 現象。創発・推論境界・overthinking・test-time scaling・PRM vs ORM・aha moment。</figcaption>
</figure>

- **創発**: Long CoT は事前学習中に埋め込まれ、プロンプト・デコーディング・ルールベース RL で「活性化」される。検証・バックトラック・部分目標・バックリンクの 4 認知挙動を持つモデル（Qwen）はルール RL だけで引き出せるが、持たないモデル（Llama）は例示ベースの RL が要る——**RL は能力を作るのでなく引き出す**という見方。
- **推論境界（reasoning boundary）**: モデルには推論容量の上限があり、超えると性能が落ちる。
- **overthinking**: 長さと性能は単調増加ではなく、**閾値を超えると低下**する。推論境界超過が有力な説明。
- **test-time scaling の 2 型**: **垂直**（1 本の推論を長く。境界に制約される）と**並列**（サンプル数を増やして検証・集約。Pass@k を超えられない上界がある。誤差下界は計算量 $N$ に対し $\log N$ でスケール）。
- **PRM vs ORM**: プロセス報酬は直観的に有利だが、軌跡カバレッジ問題と reward hacking を抱える。理論的には「データカバレッジ仮定の下で、結果監督はプロセス監督より統計的に難しくない（多項式因子を除く）」という結果があり、**結果報酬で十分**という R1 の実践を理論側から補強する。
- **aha moment**: R1 が報告した自己反省の創発には**反証も出ている**——自己反省パターンはベースモデル（epoch 0）から存在し（superficial self-reflection）、応答長の増加は反省でなく報酬最適化の結果だとする分析（Liu et al.）。aha moment を鵜呑みにしない批判的整理がなされている。

### 手法の分類法

3 特性それぞれに手法群を整理する（詳細は翻訳の図 3・§4〜6）: deep reasoning は**形式**（自然言語／構造化言語＝コード・記号／潜在空間）と**学習**（模倣＝蒸留・LIMO/s1 の少数サンプル活性化／自己学習＝STaR 系・木探索系）。reflection は**フィードバック**（ORM／ルール抽出／critic モデル、PRM／プロセス critic）と**refinement**（プロンプト型／SFT 模倣型／RL 学習型）。exploration は**スケーリング**（垂直／並列＝検証最適化・サンプリング最適化）・**内的**（RL アルゴリズム: PPO→GRPO→REINFORCE++ 系譜、ルール報酬 vs モデル報酬）・**外的**（人間駆動: ToT/GoT/Forest-of-Thought、モデル駆動: Beam/A*/MCTS 系）。§7 は再現用のオープンソース資源（訓練フレームワークとデータセット）を、§8 は 6 つのフロンティア（マルチモーダル・多言語・エージェント・効率・知識拡張・安全性）をまとめる。

## 実験結果と知見

サーベイのため独自実験はないが、性能比較表 6 点（Table 1〜6）と訓練データ統計（Table 7）を集約している。エージェント wiki の観点で拾うべき知見:

- **並列スケーリングの実力**: MCTS ベースの最適スケーリングで **1B モデルが 405B モデルを複雑タスクで上回る**報告がある一方、検証最適化は Best-of-N を超えられない——並列化は「検証器の質」で頭打ちになる。
- **SFT は記憶、RL は汎化**: SFT が出力形式を安定させ、RL が汎化を与える（学習効率最大 8 倍）という分業の整理は、[[summaries/2025-deepseek-r1]] の 4 段パイプラインの理論的裏付け。
- **効率化の系譜**: 短い推論への圧縮・トークン予算・適応的な深さ調整（考える/考えないの切り替え）・潜在空間推論——overthinking への実務的対処のカタログ。
- **agentic への拡張はフロンティア扱い**: エージェント的 Long CoT（§8.3）は木探索・環境相互作用・マルチエージェント協調が主要アプローチだが、不確実環境での頑健性とマルチエージェントのスケーラビリティが未解決と明記。

## 限界・批判的視点

- **鮮度の宿命**: 2025 年前半までの文献整理であり、この分野の速度では各論はすぐ古びる。ただし 3 特性の定義・6 現象・分類の骨格は持ちが良い。
- **形式化の緩さ**: 「論理ノード」の定義は操作的でなく、$\mathcal{B}_s, \mathcal{B}_l$ も実測可能な量として与えられていない。定義は思考の整理としては有効だが、定理を導く数学ではない。
- **サーベイ固有のバイアス**: 網羅性を優先するため個々の手法の再現性・比較条件の統制は検証されていない。性能表は各論文の自己申告値の転記であり、[[agent-evaluation]] の「provider-reported の数字は測定条件とセットでしか意味を持たない」がそのまま当てはまる。
- **安全性の節は overview 水準**: Long CoT への攻撃（OverThink 攻撃・jailbreak）に触れるが、[[summaries/2025-cot-faithfulness]] が示した「思考の不忠実性」の問題——監視の信頼性——は本サーベイの射程外にある。

## 実装・運用上の示唆

- **「長く考えさせれば良くなる」は条件付き**: 推論境界と overthinking を前提に、タスク難易度に応じた適応的な思考の深さ（budget forcing・トークン予算・考える/考えないの切り替え）を設計する。
- **並列サンプリングを足す前に検証器を疑う**: 並列スケーリングの上限は Pass@k と検証器の質で決まる。サンプル数を増やす投資と検証器を磨く投資は別物。
- **報酬設計はまずルールベースから**: PRM の reward hacking・注釈コストを踏まえると、検証可能な結果報酬＋書式報酬から始めるのが実務の定跡（R1 の再現フレームワーク群も同じ構成）。
- **オープンソース資源の入口**: §7 のフレームワーク（OpenRLHF, OpenR1, SimpleRL, TinyZero 等）とデータセット一覧（Table 7）は、推論モデルの再現・自作の出発点カタログとして使える。

## 用語と略称

- **Long CoT / Short CoT**（長い/短い思考連鎖——深い推論・探索・反省を統合した生成 vs 浅く線形な生成）
- **RLLM** = Reasoning Large Language Model（推論大規模言語モデル。o1, R1 等）
- **deep reasoning / extensive exploration / feasible reflection**（Long CoT を定義する 3 特性）
- **test-time scaling**（テスト時スケーリング, 推論時に計算量を増やして精度を上げること）／ **垂直・並列スケーリング**（1 本を長く vs 本数を増やす）
- **overthinking**（考えすぎ, 閾値を超えた長考による性能低下）／ **reasoning boundary**（推論境界, モデル固有の推論容量の上限）
- **aha moment**（RL 中に自己反省が創発したとされる瞬間。反証あり——superficial self-reflection）
- **PRM / ORM** = Process / Outcome Reward Model（過程/結果に報酬を与えるモデル）
- **CoT** = Chain-of-Thought ／ **ToT** = Tree of Thoughts ／ **GoT** = Graph of Thoughts（連鎖/木/グラフ状の思考構造）
- **MCTS** = Monte Carlo Tree Search（モンテカルロ木探索）／ **Best-of-N**（N 本生成して最良を選ぶ）
- **SFT / RL / DPO / PPO / GRPO**（教師ありファインチューニング／強化学習／直接選好最適化／近接方策最適化／グループ相対方策最適化）
- **STaR / self-consistency / budget forcing**（自己生成推論での自己学習／多数決サンプリング／思考長の強制制御）
- **latent space reasoning**（潜在空間推論, 思考をトークンとして書かず隠れ状態で行う）
- **Pass@k / Cons@k / Best-of-N / EM**（k 回試行成功率／一貫性／N 本選抜／厳密一致——評価指標）
- **reward hacking**（報酬の抜け穴を突く学習）／ **LLM** = Large Language Model

## 関連ページ

- [[test-time-compute]] — 本サーベイが主根拠となる概念ページ（垂直/並列・境界・overthinking）
- [[reasoning-and-planning]] — CoT 系譜の俯瞰。Long CoT は「3 能力の統合」という位置づけを与える
- [[self-reflection]] — reflection の外付け（Reflexion）から Long CoT 内への統合まで
- [[reinforcement-learning-from-human-feedback]] — PRM vs ORM・ルール報酬・GRPO の文脈
- [[summaries/2025-deepseek-r1]] — 本サーベイが体系化した実践側の代表原典
- [[summaries/2025-cot-faithfulness]] — 「長い思考」の中身の信頼性という、本サーベイの射程外の問題
