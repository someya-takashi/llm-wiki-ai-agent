---
type: concept
aliases: [agent training, エージェント訓練, エージェントの訓練, agentic RL, multi-turn RL, 多ターン強化学習, RFT, rejection sampling fine-tuning, rejection fine-tuning, 棄却サンプリングファインチューニング, 棄却ファインチューニング, filtered behavior cloning, trajectory distillation, 軌跡の蒸留, self-improvement, 自己改善, verifier training, 検証器の訓練, ORM, outcome-supervised reward model, SWE agent training, DAPO, per-instance capping]
tags: [agent-training, reinforcement-learning-from-human-feedback, agent-environments, coding-agents, test-time-compute, llm-agents]
related:
  - "[[reinforcement-learning-from-human-feedback]]"
  - "[[agent-environments]]"
  - "[[coding-agents]]"
  - "[[test-time-compute]]"
  - "[[agent-evaluation]]"
  - "[[agent-cost-efficiency]]"
  - "[[agent-computer-interface]]"
  - "[[tool-use-and-function-calling]]"
  - "[[parameter-efficient-fine-tuning]]"
  - "[[harness-engineering]]"
summaries:
  - "[[summaries/2024-swe-gym]]"
  - "[[summaries/2025-swe-rl-long-context]]"
  - "[[summaries/2023-swe-bench]]"
  - "[[summaries/2025-deepseek-r1]]"
  - "[[summaries/2024-deepseekmath]]"
  - "[[summaries/2025-kimi-k2]]"
  - "[[summaries/2026-kimi-k2.5]]"
  - "[[summaries/2026-deepseek-v4]]"
  - "[[summaries/2026-kimi-k3]]"
  - "[[summaries/2023-toolformer]]"
  - "[[summaries/2025-swe-effi]]"
  - "[[summaries/2024-agentless]]"
updated: 2026-10-25
---

# Agent Training（エージェントの訓練）

**エージェントとして働く力を、プロンプトや足場（scaffold, モデルの周りでツールとループを提供するプログラム）ではなく、モデルの重みそのものに入れる**方法を扱うページ。ここでの「エージェントとして働く力」とは、ツールを正しい書式で呼び、観測（コマンドの出力・テストのログ）を読み、数十ターンにわたって一貫して作業を進める力である。

2024 年ごろまで、SWE（Software Engineering）エージェントの進歩はほぼ**プロプライエタリなモデル＋足場の工夫**によるもので、オープンなモデルは素のままではほとんど解けなかった（SWE-bench Verified で 1 桁台）。その状況を変えたのが、**実行できる訓練環境**（→ [[agent-environments]]）と、その上で回す訓練の段取りである。どの方法も、次の 3 つの問いへの答えの組み合わせとして読める。

1. **誰の軌跡（trajectory, エージェントが辿った行動と観測の列）から学ぶか**——強い教師モデルか、自分自身か。
2. **どう採点するか**——テストの成否（0/1）、LLM ジャッジのルーブリック、学習した検証器。
3. **どう更新するか**——成功例をまねる教師あり学習か、良し悪しの差から学ぶ強化学習（RL）か。

### 近隣ページとの境界

| ページ | 守備範囲 |
|---|---|
| **本ページ** | **エージェントを訓練する段取り**——データの出所（蒸留・自己生成）、成功例の選び方、多ターン RL に固有の落とし穴、検証器の訓練 |
| [[reinforcement-learning-from-human-feedback]] | 学習則そのもの（PPO・GRPO・DAPO・OPD）と事後訓練の系譜。本ページはその上で「エージェントだと何が変わるか」を扱う |
| [[agent-environments]] | 訓練に使う**環境**（タスク合成・サンドボックス・報酬の接地） |
| [[test-time-compute]] | 検証器を**推論時に**使って候補から選ぶ側 |
| [[coding-agents]] / [[agent-computer-interface]] | 訓練されたモデルが動く**足場とツール** |

## 3 つのやり方 — 教師をまねる、自分の成功をまねる、差から学ぶ

| やり方 | データ | 採点の使い方 | 代表例（本 wiki） |
|---|---|---|---|
| **(1) 教師の軌跡の蒸留**（SFT） | 強いモデルの軌跡 | 成功したものだけ残す（ことが多い） | SWE-Gym：GPT-4o・Claude 3.5 Sonnet の成功軌跡 491 本 → 32B が 7.0% → 20.6% |
| **(2) 棄却サンプリングファインチューニング**（RFT） | **自分の**軌跡 | 成功したものだけ残して SFT | 72B が 11.4% → 20.5%（[[summaries/2025-swe-rl-long-context]]）、Moatless 7B が 7.0% → 10.0%（[[summaries/2024-swe-gym]]） |
| **(3) 多ターン RL** | 自分の軌跡（毎反復で引き直す） | 成功も失敗も使い、グループ内の差を advantage にする | 72B が 20.5% → 39.0%（DAPO の改変）、Kimi K2 の agentic rollout、DeepSeek-V4・Kimi K3 の専門家 RL |

**(1) 蒸留**は最も安く、効き目も大きい。SWE-Gym は数百本の成功軌跡でオープンモデルの解決率を 3 倍にした。推論の領域でも、DeepSeek-R1 は「小さなモデルに RL を直接かけるより、大きなモデルが RL で見つけた推論を蒸留したほうが強い」（Qwen-32B で 72.6 対 47.0）と報告している（[[summaries/2025-deepseek-r1]]）。ただし**教師より上には行きにくく**、教師がプロプライエタリならデータの出所に縛られる。さらに、SWE-bench の SWE-Llama は「編集すべきファイルが文脈に全部入っている」設定で訓練したため、検索で文脈を作る設定に移すと 3.00% → 0.70% に落ちた（[[summaries/2023-swe-bench]]）。**訓練時と使用時で文脈の作り方が違うと、まねた振る舞いは崩れる**。

**(2) RFT**（rejection sampling fine-tuning, filtered behavior cloning とも）は、「自分に何回も解かせ、成功した軌跡だけを学習データにする」方法である。成功を判定できさえすれば教師モデルが要らない。考え方は古く、Toolformer の「損失を下げたツール呼び出しだけを訓練例に残す」フィルタ（[[summaries/2023-toolformer]]）や、R1 の第 3 段（RL チェックポイントから正解だけを 60 万件集める）、Kimi K2 の「LLM ジャッジがルーブリックで合格とした多ターン軌跡だけ残す」大規模なツール利用データ合成（[[summaries/2025-kimi-k2]]）も同じ形をしている。エージェントでは **RL の前の準備運動**として効く——素のモデルの失敗の多くがツール呼び出しの書式違反なら、まず RFT で書式を覚えさせるだけで大きく伸びる。

**(3) 多ターン RL**は、成功した軌跡だけでなく**失敗した軌跡からも学ぶ**。同じタスクで複数（例: 10 本）の軌跡を引き、グループ内の平均より良かったものの確率を上げ、悪かったものの確率を下げる（GRPO・DAPO の考え方 → [[reinforcement-learning-from-human-feedback]]）。RFT では捨てていた失敗が負の信号になるので、RFT が頭打ちになったところからさらに伸びる。[[summaries/2025-swe-rl-long-context]] では、伸びの大半（20.5 → 35.7）が RL の第 1 段階で生まれた。

## 1 回答えて終わる RL と何が違うか

数学の RL（R1・DeepSeekMath）は、問題に 1 回答えると採点が返る。エージェントの RL は、**状態を持つ環境と数十ターンやり取りした最後に、ようやく 1 ビットの採点が返る**。[[summaries/2025-swe-rl-long-context]] はこれを部分観測マルコフ決定過程（POMDP, 環境の本当の状態は見えず、観測の履歴から次の手を決める問題設定）として書き、次の違いを挙げる。

- **軌跡が長い**：1 本が数十ステップ・数万〜十数万トークンになる。文脈長そのものが訓練の設定値になり、65k → 131k のように段階的に広げる。
- **観測が情報を持つ**：テストのログやエラーを読み、次の手に反映する力が要る。
- **信用割当が難しい**：成功の信号は最後にしか来ないので、数十の行動のどれが効いたかを割り当てにくい。
- **1 回の採点が高く、ノイズを含む**：環境を立ち上げ、テストを走らせるたびにコストがかかり、不安定なテスト（flaky test）は報酬そのものを汚す。
- **ロールアウトのインフラが重い**：環境待ちで GPU が遊ぶ、軌跡の長さがばらつく。Kimi K2 の partial rollout（長い軌跡を一時停止して次の反復で再開する）や、DeepSeek-V4 の DSec（数十万の並行サンドボックス）は、この問題への大規模な回答である（[[summaries/2025-kimi-k2]]・[[summaries/2026-deepseek-v4]]）。足場の速さが訓練の試行回数を決める点は [[agent-cost-efficiency]] も参照。

## 実測された落とし穴

本 wiki の原典が実際に踏んだ失敗を、対策と組みにして並べる。

| 落とし穴 | 何が起きたか | 対策 |
|---|---|---|
| **簡単なタスクへの偏り** | 何回も解かせると、成功は簡単なタスクに集中する。成功軌跡を全部使うと訓練データが簡単なタスクで埋まる（[[summaries/2024-swe-gym]] 図 3） | **1 タスクあたりの本数に上限**（SWE-Gym では上限 2 が最良） |
| **学習信号のないタスク** | 10 本全部成功・全部失敗のタスクは、グループ内に差がなく advantage が 0 になる | advantage 0 のサンプルを捨てる（DAPO）。累積解決率が **2/3 超と 0 のタスクを除いて**、中くらいの難しさに絞る（[[summaries/2025-swe-rl-long-context]] で 7,249 → 2,028 タスク） |
| **自己改善で悪化する** | 教師の軌跡で SFT したモデルに、自分の成功軌跡 868 本を足して訓練し直すと Lite で **15.3% → 8.7%**（SWE-Gym・OpenHands）。手順の決まった足場でも 32B は 19.0 → 19.7 で頭打ち | 成功例をまねる RFT の限界と見て、失敗からも学ぶ RL に進む（後続の研究が DAPO で 39% に到達） |
| **成功軌跡の中の失敗した手までまねる** | 成功した軌跡にも、書式エラーを起こしたターンが混じる | **書式エラーのターンを損失から外す**（マスク） |
| **長すぎる軌跡を捨てるとループが増える** | 文脈長を超えた軌跡はたいていループに陥ったものなので、捨てるとループが罰されなくなる | 捨てずに**長さの罰**で抑える |
| **サンプリング設定のずれ** | vLLM の更新で既定の top_k・min_p が有効になり、5〜10 反復後に性能が落ちた。重要度サンプリング比が前提とする「旧方策から引いた」が崩れたため | ロールアウトは温度 1.0 だけにし、他のデコーディング設定を切る。同じ系統の問題として、K2.5 は訓練側と推論側の数値の不一致にトークン単位の対数比クリップで、K3 は量子化をロールアウトと訓練でそろえることで対処している |
| **報酬のノイズ** | 外部サービスや浮動小数点誤差で、テストの結果が実行ごとに揺れる | **テストを 50 回走らせて揺れるタスクを除く** |

## 検証器を訓練する — 生成と選別を分ける

テストで採点できる環境では、**成功・失敗のラベルつきの軌跡**が副産物としてたまる。これで**検証器**（verifier, 軌跡や出力が正しいかを判定するモデル）を訓練すると、推論時に K 本の候補から 1 本を選べる（→ [[test-time-compute]]）。SWE-Gym の検証器は ORM（Outcome-supervised Reward Model, 最終結果だけを見て成功確率を出す報酬モデル）で、軌跡を読ませて YES／NO を答えさせ、YES の確率をスコアにする。

指標は 2 つを並べて見る。**Pass@K**（K 本中に 1 本でも正解があるか、選び方が完璧なときの上限）と **Best@K**（検証器が選んだ 1 本が正解か）である。

| 原典 | 1 本だけ | K 本から検証器で選ぶ | K 本中に正解がある（上限） |
|---|---|---|---|
| SWE-Gym・32B（Verified） | 20.6% | 32.0%（Best@16） | 42.8%（Pass@16） |
| SWE-Gym・Moatless 32B（Lite） | 19.7% | 26.3%（8 本から） | — |
| RL 論文・72B（Verified） | 39.0% | —（検証器なし） | 58.4%（Pass@10） |

**上限と実際の差は「よい候補は出ているのに選べていない」部分**であり、そこを埋めるのは検証器の仕事である。DeepSeekMath の「RL は Maj@K（多数決）を上げるが Pass@K を上げない」という観察（[[summaries/2024-deepseekmath]]）と合わせると、**RL で 1 本目の精度を上げることと、検証器で選ぶことは、同じ上限を取り合う関係にある**とも読める。

SWE-Gym が見つけた検証器の訓練のこつは次のとおり。

- **教師の軌跡と自分の軌跡を混ぜ、成功と失敗を同数にする**。教師の軌跡だけだと 22% 前後で頭打ちになり、失敗例を 2 倍にすると検証器が慎重になりすぎる。
- **LoRA のほうが全パラメータより良かった**（Best@8 で 29.8% 対 27.2%。正則化として効いたと推測 → [[parameter-efficient-fine-tuning]]）。
- **検証器も大きいほうが伸び続ける**（7B は 4 本で頭打ち、32B は 8 本でも伸びる）。

検証器は推論時の選別に使うだけでなく、そのまま RL の報酬モデルにもなる。Kimi K2 の自己批評ルーブリックや DeepSeek-V4 の「actor 自身が GRM（Generative Reward Model, 文章で評価を生成する報酬モデル）を兼ねる」設計は、判定する力と生成する力を同じモデルの中で育てる方向である（→ [[reinforcement-learning-from-human-feedback]]）。

## SWE エージェントの訓練の到達点（2025 年夏時点）

[[summaries/2025-swe-rl-long-context]] の表 2 が、同時期の SWE 特化のオープンモデルをまとめている（SWE-bench Verified、Pass@1）。

**表**: 訓練した SWE エージェントの比較（原典の表 2 を再構成）

| エージェント | 基盤モデル | 訓練前 | 訓練後 | 教師からの蒸留 |
|---|---|---|---|---|
| DeepSWE-32B | Qwen3-32B | 23% | **42.2%** | なし |
| SWE-agent-LM-32B（SWE-smith） | Qwen2.5-Coder-32B-Instruct | 14.3% | 40.2% | あり |
| Golubev ら（RFT → DAPO） | Qwen2.5-72B-Instruct | 11.4% | 39.0% | なし |
| Skywork-SWE-32B | Qwen2.5-Coder-32B-Instruct | 6.4% | 38.0% | あり |
| R2EGym-Agent-32B | Qwen2.5-Coder | 7.0% | 34.4% | あり |
| SWE-Fixer-72B | Qwen2.5-72B-Base | – | 30.2% | あり |
| SWESynInfer-72B | Qwen2.5-72B-Instruct | 25.4% | 30.2% | あり |
| SkyRL-Agent-14B-v0 | Qwen3-14B | 18.0% | 21.6% | なし |
| SWE-Gym-32B | Qwen2.5-Coder-32B-Instruct | 7.0% | 20.6% | あり |

読み取れることは 2 つある。第一に、**半年で 20% 台から 40% 前後まで上がった**。第二に、**蒸留ありとなしで到達点に大差がない**——教師に頼らない RL でも、強いモデルの蒸留と同じ水準に届くようになった。なお、同じ時期のフロンティアのプロプライエタリモデルや後の世代のモデルは SWE-bench Verified で 60〜80% 台にあり（[[coding-agents]] の評価節）、この表はあくまで「オープンな中規模モデルを訓練でどこまで持ち上げられるか」の記録である。

> 注: DeepSWE・SkyRL・SWE-smith・Skywork-SWE・R2E-Gym・SWE-rebench の原典は wiki に未取得で、上の数字は [[summaries/2025-swe-rl-long-context]] を通した二次情報である。Kimi K3 の評価表にある「DeepSWE」はベンチマーク名で、この表のモデルとは別物である。

## 設計論点

- **教師に頼るか、自力で育てるか**。蒸留は安く強い（R1 の 72.6 対 47.0、SWE-Gym の 491 本で 3 倍）が、教師の能力と利用条件に縛られる。自力の RL は高い（H200 ノード 16 台・100 反復超）が、出発点のモデルがある程度解ける（72B で 11%）なら教師なしで蒸留と同じ水準に届く。実際の大規模モデルの訓練（DeepSeek-V4・Kimi K3）は、**専門家を RL で育て、それを 1 体に蒸留する**という両方の組み合わせに収束している（→ [[reinforcement-learning-from-human-feedback]]）。
- **足場と訓練の相性**。手順の決まった足場（Moatless・Agentless → [[summaries/2024-agentless]]）は素のモデルでも点が高いが、訓練で伸びにくい。汎用の足場（OpenHands・SWE-agent 型）は素のモデルでは弱いが、訓練でよく伸びる（[[summaries/2024-swe-gym]]）。SWE-Effi の「弱いモデルは手順を固定したほうが安定し、強いモデルには制御を渡したほうが伸びる」（[[summaries/2025-swe-effi]]）と合わせると、**訓練でモデルを強くする計画があるなら、最初から汎用の足場で訓練データを作る**ほうが筋がよい（→ [[coding-agents]]）。
- **何が律速か**。SWE-Gym は訓練軌跡を 491 本の 25%・50%・100% と増やしても飽和せず、タスク数やリポジトリの多様性ではなく**軌跡をサンプリングする計算量**が律速だと結論した。RL の側でも、ロールアウトの速さが訓練の試行回数を決める（→ [[agent-cost-efficiency]]）。
- **環境の作り方がデータの偏りを決める**。SWE-Gym は依存関係を手で設定して 2,438 件にとどまり、並行研究（Nebius の 2 本の連続したブログ記事、Badertdinov ら・Golubev ら 2024。後者は [[summaries/2025-swe-rl-long-context]] と同じ著者群）は汎用のセットアップスクリプトで 6,415 件を集めたが、セットアップに失敗した（依存の難しい）タスクは捨てている。**自動化して数を取るか、手間をかけて偏りを避けるか**の選択である（→ [[agent-environments]]）。
- **評価の汚染を分けて見る**。SWE-bench Verified だけで訓練の効果を語ると過大評価になりうる。[[summaries/2025-swe-rl-long-context]] では、第 2 段階の RL の +3.3 ポイントが、訓練データより後に集めた SWE-rebench の月次分割では再現しなかった（→ [[agent-evaluation]]）。
- **報酬が 1 ビットだけであることの副作用**。成功／失敗しか返さない報酬では、「確信がなくても何かを提出する」振る舞いが育ちやすく、途中のどの行動が効いたかも学びにくい。プロセス報酬や途中の検証は、この限界に対する今後の論点として残っている。

## 関連ページ

- [[reinforcement-learning-from-human-feedback]] — PPO・GRPO・DAPO・OPD など学習則の系譜
- [[agent-environments]] — 訓練に使う環境（SWE-Gym・SWE-rebench、K3 の AgentENV）
- [[coding-agents]] — 訓練されたモデルが動く SWE の足場
- [[test-time-compute]] — 検証器で候補から選ぶ推論時スケーリング
- [[agent-evaluation]] — SWE-bench と汚染のない月次分割
- [[agent-cost-efficiency]] — ロールアウトのコストと、高くつく失敗
- [[agent-computer-interface]] — 訓練で使うツールの設計（RL 論文はシェル・edit・submit の 3 つ）
- [[tool-use-and-function-calling]] — ツール利用データの合成（K2）
- [[parameter-efficient-fine-tuning]] — 検証器の訓練で LoRA が効いた件
- [[summaries/2024-swe-gym]] — 実行環境つきの訓練データ・蒸留・自己改善・検証器
- [[summaries/2025-swe-rl-long-context]] — 教師なしの RFT → 多ターン DAPO
- [[summaries/2025-deepseek-r1]] — 単一ターンの RL と蒸留の比較
- [[summaries/2024-deepseekmath]] — GRPO と Maj@K／Pass@K の分析
- [[summaries/2023-swe-bench]] — SWE-Llama（文脈の分布シフトに脆い SFT）
