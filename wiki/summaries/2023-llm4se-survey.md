---
type: summary
source_path: "raw/papers/A Survey on Large Language Models for Software Engineering.md"
source_kind: paper
title: "A Survey on Large Language Models for Software Engineering"
authors: [Quanjun Zhang, Chunrong Fang, Yang Xie, Yaxin Zhang, Yun Yang, Weisong Sun, Shengcheng Yu, Zhenyu Chen]
year: 2023
venue: "arXiv:2312.15223（v1 2023-12, v2 2024-09）"
ingested: 2026-10-28
tags: [coding-agents, agent-evaluation, transformer-architecture, parameter-efficient-fine-tuning, llm-red-teaming, survey, llm4se, program-repair, data-leakage]
translation: "[[translations/2023-llm4se-survey]]"
---

# A Survey on Large Language Models for Software Engineering — エージェント以前の「LLM でソフトウェア工学」の全体地図

> 原典: [[translations/2023-llm4se-survey]] ・ `raw/papers/A Survey on Large Language Models for Software Engineering.md`
> 著者・年・会議: Quanjun Zhang, Chunrong Fang, Yang Xie, Yaxin Zhang, Yun Yang, Weisong Sun, Shengcheng Yu, Zhenyu Chen（南京大学 ソフトウェア新技術国家重点実験室・南洋理工大学・スウィンバーン工科大学）・2023・arXiv:2312.15223（v1 2023-12-23、v2 2024-09-08）。継続更新のリポジトリ: https://github.com/iSEngLab/AwesomeLLM4SE
> 底本の版: `raw/` のクリップは v2（2024 年 8 月までの論文を対象にした版）。翻訳と本ページも v2 に基づく

## 一言まとめ

LLM（Large Language Model, 大規模言語モデル）をソフトウェア工学（SE, Software Engineering, ソフトウェアの要求定義・設計・開発・テスト・保守・管理を体系的に行う分野）に使った研究 1,009 本を集め、3 つの観点で整理したサーベイ。(1) コード向けの LLM 62 個をアーキテクチャ・事前学習の目的・下流タスク・公開状況で分類し、(2) 947 本の応用研究を SE の 5 つの工程と 112 のタスクに振り分け、(3) 評価・安全性・微調整・軽量化という「SE に組み込むときの論点」をまとめた。対象は 2024 年 8 月までで、**エージェントが SE の中心に来る直前の地図**として読める。

## 背景と問題意識

2018 年ごろから、BERT や T5、GPT のような事前学習モデル（大量の無ラベルのデータで汎用の表現を学び、少量のラベル付きデータで個別のタスクに合わせる方式）が自然言語処理を大きく変えた。同じ方式はすぐコードにも持ち込まれ、CodeBERT・CodeT5・Codex といったコード向けの LLM が生まれた。2022 年末の ChatGPT 以降は、SE の研究者が手元のタスクに LLM を試す論文が急増した。

著者らによれば、この流れは追いかけるのが難しい。SE の工程は長く（要求 → 設計 → 開発 → テスト → 保守）、タスクの種類も多い（脆弱性の検出、欠陥の位置の特定、プログラム修復など）。そこに、アーキテクチャも訓練法も公開状況も違うモデルが次々に加わる。たとえばプログラム修復だけでも、モデルの種類（公開か非公開か）、大きさ、使い方（ファインチューニング、少数の例を見せる few-shot、例なしの zero-shot）の違う研究が並んでいる。先行のサーベイは深層学習一般を対象にするか（2022 年）、自然言語からコードを書くタスクだけに絞るか（NL2Code）で、SE 全体を LLM の側から見た整理はなかった。

## 提案手法 / 主張

### 文献の集め方

- **3 つの研究課題（RQ）**: RQ1「SE を支える LLM にはどんなものがあるか」（LLM の観点）、RQ2「LLM はどの SE タスクに使われているか」（SE の観点）、RQ3「LLM を SE に組み込むときの論点は何か」（統合の観点）。
- **検索**: SE 側のキーワード（コード生成、テスト生成、プログラム修復など）と LLM 側のキーワード（LLM、BERT、Codex、ChatGPT など）の両方を含む論文を、Google Scholar・ACM・IEEE で 2024 年 8 月まで検索し、32,560 本を得た。
- **選別**: Transformer の登場（2017 年）より前の論文、7 ページ未満の論文、重複を自動で除き、残りを 2 人の著者が独立に読んで判定した（意見が割れたら 3 人目が決める）。さらに参考文献を辿る雪だるま式の検索（snowballing）で補い、**最終的に 1,009 本**を得た。

<figure>

![](../../raw/assets/2023-llm4se-survey/fig1.png)

<figcaption>図1（再掲）: 年ごとの収集論文数。コード向け LLM の論文（薄い紫）、LLM を使った SE の論文（濃い紫）、その合計（水色）。合計は 2020 年 13 本、2021 年 32 本、2022 年 91 本、2023 年 376 本、2024 年（8 月まで）496 本。</figcaption>
</figure>

論文の 4 割近く（382/1,009）は査読を経ていない arXiv のプレプリントで、著者らはこれを ChatGPT 以降の研究の速さの表れと見ている。掲載先の上位は ICSE・FSE・ACL・ASE・ISSTA で、SE と自然言語処理の両方の会議にまたがる。

### RQ1: コード向けの LLM（62 個）

<figure>

![](../../raw/assets/2023-llm4se-survey/fig2.png)

<figcaption>図2（再掲）: RQ1 の分類。RQ1.1 はアーキテクチャ（エンコーダのみ・エンコーダ–デコーダ・デコーダのみ）ごとのモデル、RQ1.2 は事前学習の目的の 4 分類、RQ1.3 は入出力の型（コード→コード、文→コード、コード→文、コード→ラベル）による下流タスクの分類。</figcaption>
</figure>

**3 つのアーキテクチャ**（→ [[transformer-architecture]]）。

| 型 | 仕組みと得意なこと | 代表例 |
|---|---|---|
| エンコーダのみ（encoder-only） | 入力全体を双方向に読んで固定長の表現を作る。生成はしない。コード検索や脆弱性の検出のような「理解」のタスク向き | CuBERT、CodeBERT、GraphCodeBERT（データフローのグラフを使う） |
| エンコーダ–デコーダ（encoder-decoder） | 入力を表現に圧縮し、別の系列を生成する。コードの要約やプログラム修復のような「系列から系列へ」の変換向き | PyMT5、PLBART、CodeT5、CodeT5+、AlphaCode |
| デコーダのみ（decoder-only） | 前のトークンから次のトークンを 1 つずつ生成する（自己回帰）。生成全般 | Codex、CodeGen、StarCoder、Code Llama、DeepSeek-Coder |

著者らの観察では、初期は NLP（自然言語処理）の基盤モデル（BERT・T5・GPT）をコードに移したものが多く、資源を持つ大企業が主導した。ChatGPT 以降はモデルが大きくなり、**デコーダのみの型が主流になりつつある**。

**事前学習の目的**は 4 つの系統、15 種類に分けられる。

- **コード列のモデル化と予測**: 次のトークンを予測する因果的言語モデル化（CLM, Causal Language Modeling）、隠した単語を当てるマスク言語モデル化（MLM, Masked Language Modeling）、隠した区間を当てる MSP（Masked Span Prediction）、変数名などの識別子だけを隠して当てる MIP（Masked Identifier Prediction）など 7 種類
- **コードと自然言語の双方向の理解と生成**: コード→説明と説明→コードを両方学ぶ BDG（Bimodal Dual Generation）、メソッド名の生成
- **コードの構造と関係の理解**: 識別子かどうかの判定、データフローの辺の予測、構文木（AST, Abstract Syntax Tree, ソースコードの構造を木で表したもの）との対応づけなど
- **モダリティをまたぐ表現学習**: コードと自然言語の対照学習など

著者らは、目的が NLP 由来の汎用のもの（CLM・MLM）から、識別子や構造を使う**コードを意識したもの**へ移ってきたとまとめる。

**公開状況**: 62 個のうち 54 個が何らかの成果物（コード・データ・モデル）を公開し、ほぼすべてが GitHub と Hugging Face に置いている。一方、訓練に要る計算資源、ハイパーパラメータや乱数の種の設定、高品質のデータを揃えるのは難しく、再現性は従来の深層学習の研究以上に課題だとしている。

### RQ2: SE の工程ごとの応用（947 本・112 タスク）

5 つの工程ごとの論文数（表 V）と、論文の多いタスクは次のとおり。

| 工程 | 論文数（表 V の合計） | 論文の多いタスク |
|---|---|---|
| 要求と設計 | 43 | 要求の分類（7）、要求の聞き取り（7）、曖昧さの検出（5） |
| 開発 | 341 | **コード生成（155）**、コードの要約（41）、コード補完（34）、コード翻訳（24） |
| テスト | 240 | **脆弱性の検出（76）**、テスト生成（46）、ファジング（18）、欠陥の位置の特定（17） |
| 保守 | 211 | **プログラム修復（66）**、コードレビュー（24）、ログ解析（20） |
| 管理 | 5 | 工数見積もり（2）など |

著者らは、開発とテストに応用が集中し、要求・設計・管理はほとんど手つかずだと指摘する。理由として、これらのタスクの多くがもともと LLM の下流タスク（コードを入力してコードを出す系列変換）として扱いやすい形をしていることを挙げる。

本 wiki の主題に近い**プログラム修復**（バグのあるコードから正しいパッチを自動で作るタスク）では、LLM の使われ方が段階的に変わってきたと整理している。

1. **LLM の下流タスクとして**: CodeT5 や CodeBERT の原論文で、バグのあるコードを正しいコードに「翻訳」する課題として評価された。
2. **既存の修復手法の部品として**: CURE（2021）は GPT を事前学習して既存の修復手法に組み込んだ。
3. **修復用にファインチューニング**: 過去のバグ修正の対で T5 や BART を訓練する（最も研究が多い）。
4. **ファインチューニングなしの穴埋め**: AlphaRepair（2022）は、バグのある行をマスクに置き換えて CodeBERT に埋めさせた。修復を「翻訳」から「穴埋め」に変える転換である。
5. **従来の修復手法との組み合わせ**: 修正のテンプレートをマスクのパターンにして LLM に埋めさせる、など。

これと並んで ChatGPT を使った実証研究が大量にあり、最後に「実際のイシュー」「修復のエージェント」「対話的な修復」も 1 行ずつ挙げられている。コード生成の節でも、エージェントによる生成とリポジトリ単位の生成は「その他」の最先端の話題として 1 文で触れられるだけである。

### RQ3: SE に組み込むときの論点

- **評価とベンチマーク**: HumanEval・HumanEval-X・EvalPlus・CodeXGLUE などの整理に加え、**データ漏れ（data leakage）** を中心の課題に挙げる。LLM は公開リポジトリをほぼすべて読んで訓練されるので、評価に使う問題が訓練データに入っていないことを保証できない。対策として、ChatGPT の訓練の締め切りより後の問題で作った EvalGPTFix のような新しいベンチマークが作られている。
- **安全性と信頼性**: コード向けの LLM への攻撃を 3 つに分ける。意味の同じコードに小さな変更を加えて誤判定させる**敵対的攻撃**（ALERT・CodeAttack）、訓練データに引き金（トリガー）を仕込んで特定の入力のときだけ誤動作させる**バックドア攻撃**（BADCODE など）、API への問い合わせだけで能力を盗み出す**模倣攻撃**である（→ [[llm-red-teaming]]）。
- **領域への合わせ込み**: パラメータの一部だけを更新する効率的なファインチューニング（アダプタ・プレフィックス・LoRA → [[parameter-efficient-fine-tuning]]）と、新しいデータで学ぶと前の知識を忘れる破滅的忘却（catastrophic forgetting）を防ぐ継続学習。
- **圧縮と蒸留**: CodeBERT と GraphCodeBERT を 3 MB まで縮めた Compressor のように、IDE（統合開発環境）や個人の端末で動かすための軽量化。

### 課題と機会（6 節）

著者らは今後の方向として 7 つを挙げる。モデルの大きさと効果の両立、特定のタスク向けの領域モデル、**汚染のない評価データ**、未開拓の SE タスクへの応用、多タスク・多次元のベンチマーク（正解率以外に時間効率や頑健性も測る）、画像を扱う SE（GUI のテストなど）、説明可能性である。

汚染のない評価データについては、具体的な事例と作り方が書かれている。プログラム修復の標準ベンチマーク Defects4J のコードの一部は、CodeT5・CodeBERT・UniXcoder の訓練に使われた CodeSearchNet に含まれていた。ChatGPT は LeetCode の問題番号を言うだけで問題文と解答を返した。汚染のないデータの出どころとして、(1) HumanEval のように人手で新しく書く、(2) EvalGPTFix のように最近公開されたものを使う、(3) 企業内の非公開プロジェクトを使う、の 3 つを挙げ、モデルが答えを記憶しているだけかを確かめる技術も要るとしている。

## 実験結果と知見

サーベイなので新しい実験はない。数字として拾えるのは文献の集計と、引用した結果の表である。

- **論文の急増**: 年ごとの論文数は 2022 年の 91 本から 2023 年の 376 本へ約 4 倍になり、2024 年は 8 月までで 496 本に達した（図 1）。
- **タスクの偏り**: 947 本のうち、コード生成だけで 155 本ある。工程別では開発 341 本、テスト 240 本に対し、要求と設計は 43 本、管理は 5 本しかない（表 V）。
- **プログラム修復の初期の水準**（表 VI, BFP-small）: CodeBERT・CodeT5・GraphCodeBERT・PLBART などで、BLEU（生成文と正解の語の重なりの指標）は 77〜80% だが、正解と完全に一致した割合は 16〜22% にとどまった。BLEU が高くても正しく直せているとは限らない。
- **公開状況**: 62 個中 54 個が成果物を公開している。

## 限界・批判的視点

- **エージェントと実リポジトリのイシューがほぼ入っていない**。2024 年 8 月までを対象にしながら、SWE-bench（→ [[summaries/2023-swe-bench]]）は付録 B の分類図の「コード生成」の欄に参考文献番号 [612] として載るだけで、本文では論じていない。SWE-agent（→ [[summaries/2024-swe-agent]]）、Agentless（→ [[summaries/2024-agentless]]）、OpenHands（→ [[summaries/2024-openhands]]）は参考文献にもない（本 wiki で参考文献の一覧を検索して確かめた）。エージェントは「エージェントによる生成」（AgentCoder・CodeAgent など）「修復のエージェント」（RepairAgent など）として参考文献番号つきで 1 行ずつ挙がるだけである。同じ時期に、本 wiki の [[coding-agents]] が扱う「リポジトリを探索し、編集し、テストを走らせるエージェント」が SE の中心に移りつつあった。本論文は、**関数やコード片を入出力とする LLM4SE の到達点**の記録として読むのが妥当である。キーワード検索の SE 側の語（コード生成、修復など）が、エージェントの論文をうまく拾えなかった可能性もある。
- **数の記載が合わない**。応用研究は「947 本」とされるが、表 V の 5 工程の合計は 43 + 341 + 240 + 211 + 5 = 840 本である（本 wiki の計算）。選別の記述では SE の論文は 912 + 36 = 948 本になり、こちらも 947 と 1 本ずれる。図 1 の各年の合計は 1,008 本で、本文の 1,009 本と 1 本違う。下流タスクも要旨は 16 種類、3.3 節は「15 の代表的な下流タスク」としている。
- **表の誤りがある**。表 II（コード向け LLM の一覧）は、行ごとの値がモデル名とずれているように見える（CodeBERT が ICML・Google の、BLOOM がエンコーダ–デコーダ型の Salesforce のモデルとして載っている。翻訳の訳注に記録）。表 I では ICPC（International Conference on Program Comprehension）が「International Collegiate Programming Contest」と書かれている。個々のモデルの属性は原論文で確かめる必要がある。
- **分類は網羅的だが浅い**。1 タスクあたりの記述は数文で、手法どうしを同じ条件で比べた数字はほとんどない（表 VI だけ）。「どの手法が良いか」を判断する材料にはならず、文献を探す索引として使うものである。
- **査読前の論文が 4 割**。382 本が arXiv のプレプリントで、品質の判定は 2 人の著者の判断に任されている。

## 実装・運用上の示唆

- **用途で型を選ぶ発想は今も使える**。検索や分類（重複コードの検出、脆弱性の判定）にはエンコーダのみの小さなモデル、生成にはデコーダのみのモデル、という整理は、エージェントの中で埋め込みの検索や分類器を部品として使うときの目安になる（→ [[retrieval-augmented-generation]]）。
- **評価データの出どころを確かめる**。公開ベンチマークで測るなら、その問題が訓練データに入りうるかを先に考える。新しく書く・締め切り後のものを使う・非公開のものを使う、の 3 つの作り方は、のちの SWE-rebench の月次分割や SWE-bench Verified の退役の議論（→ [[agent-evaluation]]、[[summaries/2026-swe-bench-verified-retired]]）と同じ方向である。
- **BLEU のような文字列の一致で修復を測らない**。表 VI のとおり、BLEU は 8 割でも完全一致は 2 割である。SWE-bench がテストの実行で採点するようになったのは、この限界への答えと読める。
- **LLM を既存の手順の部品にする**。著者らは、複雑なタスクでは LLM を end-to-end の解法にするより、既存の手順の一部に組み込むほうが現実的だと勧める。手順を固定して LLM を部品として使う Agentless の考え方（→ [[coding-agents]] の「手続き型とエージェント型」）と同じ方向の提言である。

## 用語と略称

- **LLM**（Large Language Model）: 大規模言語モデル
- **SE**（Software Engineering）: ソフトウェア工学
- **LLM4SE**: LLM をソフトウェア工学に使う研究分野の通称
- **NLP**（Natural Language Processing）: 自然言語処理
- **NL2Code**: 自然言語の説明からコードを生成するタスク
- **事前学習（pre-training）／ファインチューニング（fine-tuning）**: 大量の無ラベルのデータで汎用の表現を学ぶ段階／少量のラベル付きデータで個別のタスクに合わせる段階
- **下流タスク（downstream task）**: 事前学習したモデルを合わせ込んで解かせる個別のタスク
- **few-shot / zero-shot**: プロンプトに少数の例を見せる使い方／例を見せずに指示だけで解かせる使い方
- **CLM**（Causal Language Modeling）: 前のトークンから次のトークンを予測する事前学習の目的
- **MLM**（Masked Language Modeling）: 一部を隠した入力から隠した語を当てる事前学習の目的
- **MSP**（Masked Span Prediction）: 連続した区間を隠して当てる目的
- **MIP**（Masked Identifier Prediction）: 変数名や関数名などの識別子だけを隠して当てる目的
- **BDG**（Bimodal Dual Generation）: コード→説明と説明→コードの両方向の生成を学ぶ目的
- **AST**（Abstract Syntax Tree）: 抽象構文木。ソースコードの構造を木で表したもの
- **APR**（Automated Program Repair）: プログラムの自動修復
- **BLEU**（Bilingual Evaluation Understudy）: 生成した文と正解の文の語の重なりを測る指標
- **PEFT / LoRA**（Parameter-Efficient Fine-Tuning / Low-Rank Adaptation）: パラメータの一部や低ランクの追加行列だけを更新する効率的なファインチューニング（→ [[parameter-efficient-fine-tuning]]）
- **破滅的忘却（catastrophic forgetting）**: 新しいデータで訓練すると、前に学んだ知識を失う現象
- **データ漏れ（data leakage）／汚染（contamination）**: 評価に使う問題や正解が、モデルの訓練データに含まれていること
- **IDE**（Integrated Development Environment）: 統合開発環境
- **GUI**（Graphical User Interface）: 画面とマウス操作で使う利用者向けのインターフェース
- **XAI**（Explainable AI）: 判断の理由を説明できるようにする AI の技術

## 関連ページ

- [[coding-agents]] — 本論文が描いた「関数・コード片単位の LLM4SE」の後に来た、リポジトリ単位のエージェント
- [[agent-evaluation]] — データ漏れと汚染のない評価データの作り方
- [[transformer-architecture]] — エンコーダのみ・エンコーダ–デコーダ・デコーダのみの 3 つの型
- [[parameter-efficient-fine-tuning]] — コード向け LLM の効率的なファインチューニング
- [[llm-red-teaming]] — コード向け LLM への敵対的攻撃・バックドア攻撃・模倣攻撃
- [[summaries/2023-swe-bench]] — 本論文が扱わなかった、実際のイシューを実行で採点するベンチマーク
- [[summaries/2024-agentless]] ・ [[summaries/2024-swe-agent]] ・ [[summaries/2024-openhands]] — 同時期のエージェント型と手続き型の SE の足場
- [[summaries/2026-swe-bench-verified-retired]] — 汚染を理由にベンチマークを退役させた後年の事例
- [[summaries/2025-llm-reasoning-to-agents]] — LLM の推論からエージェントまでを整理した別のサーベイ
