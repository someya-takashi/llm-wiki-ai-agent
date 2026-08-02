---
type: summary
source_path: "raw/papers/ReAct_Synergizing Reasoning and Acting in Language Models.md"
source_kind: paper
title: "ReAct: Synergizing Reasoning and Acting in Language Models"
authors: [Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, Yuan Cao]
year: 2022
venue: ICLR 2023
ingested: 2026-07-24
tags: [llm-agents, reasoning-and-planning, agent-loop, tool-use-and-function-calling, react]
translation: "[[translations/2022-react]]"
---

# ReAct: Synergizing Reasoning and Acting in Language Models

> 原典: [[translations/2022-react]] ・ `raw/papers/ReAct_Synergizing Reasoning and Acting in Language Models.md`（ar5iv クリップ, arXiv:2210.03629）
> 著者・年・会議: Shunyu Yao ほか（Princeton / Google Brain）・2022・ICLR 2023

## 一言まとめ

LLM（Large Language Model, 大規模言語モデル）に「考える（thought）」と「行動する（action）」を**交互に生成させる**だけで、推論の幻覚（hallucination, もっともらしいが事実でない内容の生成）を外部知識で抑え、行動の迷走を推論で正せることを示した論文。thought → action → observation を繰り返す枠組みは、その後のほぼすべての LLM エージェントの [[agent-loop]]（エージェントループ）の原型になった。

## 背景と問題意識

2022 年当時、LLM の「推論」と「行動」は別々の研究の系譜だった。

- **推論のみ**: CoT（Chain-of-Thought, 思考過程を明示的に出力させて推論精度を上げるプロンプト手法）は算術や常識推論で成果を上げていたが、モデルの内部知識だけで完結する**静的なブラックボックス**であり、外部世界に接地（grounding, 生成内容を実世界の情報に結びつけること）されていない。そのため事実の幻覚や、一度間違えるとそのまま連鎖する誤り伝播が起きる。
- **行動のみ**: WebGPT や SayCan など、LLM に環境内での行動を予測させる研究はあったが、高レベルの目標について抽象的に推論したり、進捗を覚えておくワーキングメモリを言語で維持したりはしていなかった。また多くは高価な人間フィードバックによる強化学習に依存していた。

人間は料理をしながら「切り終わったから次は湯を沸かす」と頭の中で進捗を整理し、「生地の作り方が分からないから検索しよう」と行動に切り替える。この**推論と行動の相乗**を LLM で再現できないか、というのが問題意識である。

## 提案手法

<figure>

![](../../raw/assets/2022-react/x1.png)

<figcaption>図1（再掲）: 上段は HotpotQA の質問に対する (1a) Standard・(1b) CoT・(1c) Act のみ・(1d) ReAct の比較。CoT は内部知識だけで推論して幻覚し（赤）、Act のみは検索はできるが観測を統合できず失敗する。ReAct は思考で検索を導き、観測から得た事実（緑）で思考を更新して正答に至る。下段は ALFWorld で、思考を持つ (2b) だけが部分目標を追跡してタスクを完了する。</figcaption>
</figure>

### 定式化: 行動空間を「言語」で拡張する

エージェントを、観測 $o_t$ を受けて方策 $\pi(a_t|c_t)$（$c_t$ はそれまでの観測・行動の履歴＝コンテキスト）で行動 $a_t$ を選ぶものとして定式化し、行動空間を $\hat{\mathcal{A}} = \mathcal{A} \cup \mathcal{L}$ に拡張する。$\mathcal{L}$ は言語空間で、ここに属する「行動」が **thought（思考）**。thought は環境に作用せず観測も返さない代わりに、コンテキストに追記されて以降の推論・行動を支える。つまり**「考えること」自体を行動の一種として同じループに乗せた**のが核心のアイデアで、後の LLM エージェントで言う scratchpad（中間の思考を書き溜める作業領域）に相当する。

### エージェントの構成要素

- **ツール**: Wikipedia API のわずか 3 つ——`search[entity]`（ページ冒頭 5 文か類似候補を返す）・`lookup[string]`（ページ内 Ctrl+F 相当）・`finish[answer]`（回答して終了）。意図的に弱いツールにして、モデルが**言語での推論によって検索を導く**ことを強制している（→ [[tool-use-and-function-calling]]）。
- **ループ**: 知識タスク（HotpotQA / FEVER）では thought-action-observation を毎ステップ交互に行う **dense thought**。意思決定タスク（ALFWorld / WebShop）では行動数が多いので、thought をいつ挟むかを**モデル自身に決めさせる sparse thought**。この密度の使い分けは実務でも重要な設計判断になる（→ [[agent-loop]]）。
- **学習**: 凍結した PaLM-540B に few-shot の in-context 事例（人手で書いた 1〜6 個の軌跡）を与えるだけ。勾配更新なし。
- **人間の介入点**: 付録 A.3 で、失敗した軌跡の thought を人間が 2 箇所書き換えるだけでエージェントが成功に転じることを示した。行動を逐一指示する代わりに**思考を編集して方策を直す**という、HITL（Human-in-the-Loop, 人間が途中で承認・介入する運用）の先駆的なデモである。

## 実験結果と知見

**表1（原典 Table 1 より）**: PaLM-540B でのプロンプティング結果。

| 手法 | HotpotQA (EM) | Fever (Acc) |
| --- | --- | --- |
| Standard | 28.7 | 57.1 |
| CoT | 29.4 | 56.3 |
| CoT-SC | 33.4 | 60.4 |
| Act（思考なし） | 25.7 | 58.9 |
| ReAct | 27.4 | 60.9 |
| CoT-SC → ReAct | 34.2 | **64.6** |
| ReAct → CoT-SC | **35.1** | 62.0 |
| 教師あり SoTA | 67.5 | 89.5 |

- **幻覚の激減が最大の成果**。人手での失敗分析（原典 Table 2）で、CoT の失敗の 56% が幻覚だったのに対し ReAct は **0%**。成功事例中の「たまたま当たった」false positive も 14% → 6% に減った。外部知識への接地が効いている。
- **ただし ReAct 単体は HotpotQA で CoT にわずかに負ける**（27.4 vs 29.4）。構造の制約で推論の柔軟性が落ち、同じ思考・行動を繰り返して抜け出せなくなる reasoning error（失敗の 47%）や、検索が有益な情報を返さないことによる脱線（23%）が起きるため。**最良は両者の組み合わせ**（ReAct が規定ステップで解けなければ CoT-SC へ、CoT-SC の多数決が割れたら ReAct へ切り替えるフォールバック）で、CoT-SC 21 サンプル分の性能に 3〜5 サンプルで到達する。
- **意思決定タスクでは圧勝**。ALFWorld 成功率 71%（Act 45%、10⁵ 軌跡で訓練した模倣学習 BUTLER 37%）、WebShop 成功率 40.0%（模倣+強化学習 28.7%）。**1〜2 個の in-context 事例だけ**で、大量データで訓練した IL/RL（Imitation Learning / Reinforcement Learning, 模倣学習／強化学習）を絶対値 34% / 10% 上回った。
- **thought の中身が重要**という対照実験も示した。環境状態の復唱に限定した Inner Monologue 風の thought（ReAct-IM）では 53% にとどまり、目標分解・部分目標の完了判定・常識推論を含む自由な thought（71%）に大きく劣る。
- **ファインチューニングでは ReAct が最良**。自己生成した正答軌跡 3,000 件で小型モデルを訓練すると、ReAct 形式の PaLM-8B が PaLM-62B の全プロンプティング手法を上回る。「事実を暗記させる」より「調べ方を教える」方が汎化する、という後の agentic 訓練につながる知見。
- 実務コスト面: HotpotQA では 7 ステップ・FEVER では 5 ステップを打ち切り上限とした（それ以上は性能が伸びない）。GPT-3（text-davinci-002）では PaLM-540B より高性能（ALFWorld 78.4%）で、指示追従の訓練がエージェント性能に効くことを示唆する。

## 限界・批判的視点

- **反復ループ問題**: 同じ thought と action を繰り返す failure mode は ReAct 固有で、貪欲デコードが一因と著者らも認める。これは現在のエージェント開発でも loop detection として対策され続けている問題の初出の一つ。
- **再現性**: 主実験の PaLM-540B は非公開モデルで、プロンプト全文（付録 C）と GPT-3 版コードの公開で補っているが、結果はモデル・デコード設定に強く依存する。ベンチマークの HotpotQA はラベル自体が古くなっている事例がある（付録 A.2）ことも著者ら自身が指摘しており、評価の鮮度という [[agent-evaluation]] 的な問題を先取りしている。
- **安全性**: 行動空間は Wikipedia と研究用 WebShop という閉域に限定されており、著者らも Ethics Statement で「LLM を外部環境に接続することには危険が伴う」と明記している。開かれた環境での prompt injection（外部入力に埋め込まれた指示でエージェントを乗っ取る攻撃）や権限管理は本論文の範囲外で、後続の課題として残った。
- ツールが 3 種の固定 API であり、現代的な function calling（JSON スキーマでツールと引数を構造的に宣言する方式）以前の素朴な形。行動空間が大きいタスクではデモ数がコンテキスト長を超えてしまう、と著者ら自身が結論で認めている。

## 実装・運用上の示唆

- **thought の 4 用途**（目標の分解／部分目標の完了追跡／次の部分目標の決定／常識による所在推論）はそのまま現代のエージェントのシステムプロンプト設計に流用できる分類。
- 単純な「行動のみ」エージェントが状態を見失って同じ操作を繰り返す、という Act の失敗（付録 D.2.2）は、ツール呼び出しの合間に**現在地と次の一手を言語化させる**だけで大きく減る——ReAct が現在も多くのフレームワークの既定ループである理由。
- 内部知識と外部検索の**フォールバック戦略**（ReAct ⇄ CoT-SC）は、「まず検索し、ダメなら内部知識」「自信がなければ検索」という現在の RAG（Retrieval-Augmented Generation, 外部知識を検索して生成の根拠にする手法）ルーティングの原型として読める。
- 失敗軌跡は行動でなく **thought を編集して直す**（付録 A.3）——エージェントのデバッグ・操縦の実践知として今も有効。

## 用語と略称

- **ReAct** = Reason + Act。思考と行動を交互に生成するプロンプトパラダイム
- **LLM** = Large Language Model（大規模言語モデル）
- **CoT** = Chain-of-Thought（思考連鎖。推論過程を明示的に出力させる手法）
- **CoT-SC** = CoT with Self-Consistency(複数の CoT サンプルの多数決で答えを決める手法)
- **EM** = Exact Match（生成答とラベルの完全一致率）
- **IL / RL** = Imitation Learning / Reinforcement Learning（模倣学習／強化学習）
- **IM** = Inner Monologue（環境フィードバックの復唱に限定された「内的独白」を使う先行手法）
- **HITL** = Human-in-the-Loop（人間が介入する運用形態）
- **HotpotQA / FEVER** = マルチホップ質問応答／事実検証のベンチマーク
- **ALFWorld / WebShop** = テキストベースの家庭内タスク／模擬 EC サイトでの購買タスクのベンチマーク
- **PaLM** = Pathways Language Model（Google の LLM。本論文の主実験は 540B 版）

## 関連ページ

- [[summaries/2024-swe-agent]] — thought + action の形式を実務的な ACI へ発展させた例

- [[reasoning-and-planning]] — ReAct が属する概念。CoT との対比の本拠地
- [[agent-loop]] — thought-action-observation ループの定式化
- [[tool-use-and-function-calling]] — Wikipedia API を初期のツール利用として位置づけ
- [[agent-safety-and-guardrails]] — HITL 介入（人間による thought 編集）の最初期の実例
- [[translations/2022-react]] — 全文翻訳
