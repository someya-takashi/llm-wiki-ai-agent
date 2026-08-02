---
type: summary
source_path: raw/articles/Understanding LLM Serving_ How to Run Language Models Fast, Cheap, and Effectively.md
source_kind: article
title: "Understanding LLM Serving: How to Run Language Models Fast, Cheap, and Effectively"
authors: [Thanh Tung Vu]
year: 2025
venue: Medium, 2025-04-21
ingested: 2026-08-02
tags: [llm-serving-systems, llm-inference-optimization, agent-observability, model-quantization, parameter-efficient-fine-tuning]
translation: "[[translations/2025-understanding-llm-serving]]"
---

# LLM Serving を理解する — 速く・安く・効果的に動かす

> 原典: [[translations/2025-understanding-llm-serving]] ・ `raw/articles/Understanding LLM Serving_ How to Run Language Models Fast, Cheap, and Effectively.md`
> 著者: Thanh Tung Vu ・ Medium, 2025-04-21

## 一言まとめ

**LLM serving を「モデルの適応 → 推論の最適化 → ルーティング → 量子化 → 監視 → ツールスタック」という 6 段のライフサイクルとして俯瞰した、ビジネス寄りの入門記事**である。技術的な深さは [[summaries/2025-llm-serving-techniques]] に大きく劣るが、**あちらが扱わない 2 つの層——モデル間のルーティング／カスケードと、実際のツールスタック**を埋める。

## 背景と問題意識

同時期の Zenn 記事が「serving system の中で何が起きているか」を扱うのに対し、本記事は「**serving 層をどう組織として設計・運用するか**」を扱う。冒頭で読者を 3 種類に分けているのが記事の性格をよく表している——CTO には費用効率の良いインフラ、ML/プラットフォームエンジニアにはレイテンシとスループット、プロダクトマネージャには応答性の高い機能。

主張は素直である。**「最適化の不十分な serving 構成は高額なクラウド請求・一貫性のないユーザー体験・システム停止につながり、うまく調整されたスタックはリアルタイムの知性と高トラフィックの優雅な処理を可能にする」。**

## 提案手法 / 主張

### 1. PEFT を「serving の技法」として位置づける

記事の最初の判断が面白い。**完全なファインチューニング・指示チューニング・継続的事前学習は展開の前に終わるが、PEFT だけが serving に関係する**——なぜなら PEFT は**ホットスワップ可能な低コストのカスタマイズ**を支えるからである。

そこから 2 つの展開モードを立てる。

| | **統合適応（merged）** | **モジュラー適応（modular）** |
|---|---|---|
| やること | 適応した重みを基本モデルへ統合してから展開 | 基本モデルを凍結し、用途に応じて LoRA / アダプタを動的に読み込む |
| 長所 | 速く単純。レイテンシとメモリのオーバーヘッドが最小 | モデルを複製せずマルチテナント・マルチドメインを実現 |
| 短所 | 柔軟性に欠ける | 動的読み込みの機構が要る |
| 使いどころ | 単一の専門用途 | 複数の用途・ドメイン・クライアント |

**例が具体的である**——医療と法務の両方を扱うカスタマーサービスのチャットボットが、リクエストごとに適切な LoRA を読み込むことで、複数の完全なモデルを保持せずに済む。

> これは Zenn 記事の **Multi-LoRA（Punica・S-LoRA）** と同じ話を、運用者の言葉で述べたものである。あちらは CUDA カーネルの実装、こちらは展開モードの選択として書いている。

### 2. 推論時の最適化（4 つ）

投機的デコーディング・KV キャッシング・**早期終了（early exit）**・バッチング＆並列デコーディングを、それぞれ「いつ使うか」と「利点」で整理する。

**早期終了は本 wiki に無かった項目である**が、記事の説明は「モデルが結果に確信を持った時点で生成を打ち切る」「分類型のタスクや構造化出力で有効」というもので、**層単位の early exit（ネットワークの途中で抜ける）と生成の打ち切りを区別していない**。用語としては要注意である。

### 3. ルーティングとカスケード — 本記事の独自価値

**「すべてのユーザーのリクエストを 1 兆パラメータのモデルで提供するのは無駄である」**——ここから 2 つの戦略を立てる。

- **モデルカスケード**: 能力とコストの順にモデルを並べる。**小さいモデルで受け、信頼度の閾値を満たせば返し、満たさなければ大きいモデルへ引き上げる**。例として「7B で日常的な質問に答え、信頼度が低いときだけ GPT-4 へ」。
- **ルータ**: 軽量な分類器やルールで、クエリの特徴（トピック・長さ・複雑さ）から**最初に**適切なモデルへ振り分ける。例として「法的な方針についての質問を法律特化モデルへ」。

**使い分けの基準が明確である。** カスケードは「単純なクエリと複雑なクエリが混在する」場合に平均コストとレイテンシを下げる。ルータは「ドメイン固有のモデルや利用階層がある」場合に、**冗長な計算を避けるために前もって決める**。

> **本 wiki でこの層は空白だった。** 既存の「ルーティング」の記述は MoE のエキスパートルーティング・エージェントのタスク振り分け・RAG の検索先選択で、**モデル間のコスト階層化**とは別物である。[[llm-inference-optimization]] の「モデルファミリーの階層化——定型クエリは小モデル、高価値タスクは大モデルへ」という一文が最も近いが、機構の記述はなかった。

### 4. 量子化（4 手法）

QLoRA・AWQ・GPTQ・GGUF を「いつ使うか」で並べる。詳細は [[model-quantization]] にあるので繰り返さないが、**記事に 1 つ誤りがある**（後述）。

### 5. 監視と評価

**3 つの手法**を挙げる。

- **合成評価（synthetic evaluations）**: LLM を評価者に使う。人間評価が高価すぎるとき、開発中に複数のモデル版を素早く比較するのに向く（→ [[agent-evaluation]] の LLM-as-a-judge。**バイアスの実測値は [[summaries/2023-qlora]] にある**）。
- **シャドー展開（shadow deployments）**: 候補モデルを本番と**並行して動かし、ユーザー体験に影響を与えずに**出力を静かに比較する。完全な展開の前に退行を捕まえる。
- **ログとダッシュボード**: レイテンシ・トークン使用量・エラー率・ハルシネーション頻度を継続的に可視化する。**最初から実装すべき**と明記している。

追跡すべき指標は **レイテンシ（最初のトークンと全体）・コスト（トークンあたり／リクエストあたり）・正しさ（正解率・ハルシネーション率・指示追従）**。

### 6. ツールスタック

本記事で最も実用的な部分である。層ごとに具体名を挙げる。

| 層 | ツール |
|---|---|
| **serving エンジン** | `vLLM`（paged attention・continuous batching）、`Triton`（複数バックエンド・動的バッチング・モデルのバージョン管理） |
| **ルーティング** | `Ray Serve`（パイプラインの動的合成・シャーディング）、`BentoML`（マイクロサービス化・REST API・K8s 統合） |
| **展開プラットフォーム** | Hugging Face Inference Endpoints、OpenRouter（価格・レイテンシ・提供者でルーティング）、Replicate（サーバレス）、オンプレミスのクラスタ |
| **監視・実験管理** | `MLflow`、`LangSmith`（LLM 特化のトレーシング・評価）、`Weights & Biases` |
| **クラウド** | AWS（SageMaker・Bedrock）、Azure（Azure ML・OpenAI Service）、GCP（Vertex AI） |

**業界のベストプラクティスとして「ハイブリッド」を挙げる**——柔軟性と制御のためにモジュラーなスタック（Ray Serve ＋ vLLM / Triton）、スケーラビリティとコンプライアンスのためにクラウドのマネージド基盤。加えて**コストを意識したモデル選択の内部ルーティング層**と**可観測性のパイプライン**を自前で持つ、と述べている。

## 限界・批判的視点

- **技術的な誤りが 1 つある。** **「GPTQ (Gradient Post-Training Quantization)」は誤りである。** 原典（[[summaries/2022-gptq]]）の脚注 1 が明記しているとおり、**GPTQ の名は「OPT モデルファミリの名」と「PTQ（post-training quantization）の略称」の合成**である。「Gradient」は入っていない。手法自体は勾配でなく**ヘッセ行列の二次情報**を使うので、記述としても紛らわしい（記事本文の「二次の最適化を用いる」は正しい）。
- **early exit の説明が曖昧である。** 「モデルが結果に確信を持った時点で生成を打ち切る」は、層単位の early exit（ネットワークの途中の層で出力を確定させる）と、生成の停止条件を区別していない。
- **数値が一切ない。** 「速くなる」「安くなる」は述べるが、どの手法がどれだけ効くかの定量値がない。本 wiki の [[summaries/2026-llm-optimization-guide]] や [[summaries/2025-llm-serving-techniques]] と併読しないと、判断の材料にならない。
- **査読なしの個人ブログである。** 出典・引用がほとんどなく（自身の別記事へのリンクを除く）、主張の裏取りができない。「OpenAI・Anthropic・Cohere がハイブリッドな手法を採る」という記述にも出典がない。
- **serving system の内部に踏み込まない。** continuous batching も PagedAttention も**ツールの機能として名前が出るだけ**で、なぜ効くのかは説明されない。「バッチング」を推論時最適化の 1 項目として挙げるが、**バッチングが serving の第一戦略である理由**（memory-bound だから）には触れない。
- **図がすべて装飾である。** 5 枚あるが、1 枚は記事タイトルのカバー、残り 4 枚は本文の箇条書きをアイコン付きで復唱するインフォグラフィックで、本文を超える情報を持たない（ingest 時に全部除外した）。
- **カスケードの実務的な難所を扱わない。** 「信頼度の閾値」をどう決めるか、**LLM の自己申告の信頼度が当てにならない**という既知の問題（→ [[self-reflection]]）には触れていない。

## 実装・運用上の示唆

1. **PEFT を「展開前の作業」でなく「serving の設計変数」として扱う。** 統合適応かモジュラー適応かは、**マルチテナントで運用するかどうか**で決まる。この分け方は実務でそのまま使える。
2. **モデル間のコスト階層を設計に入れる。** カスケード（後から引き上げる）とルータ（先に振り分ける）は排他ではなく、**冗長な計算を避けたいならルータ、判定が難しいならカスケード**という使い分けになる。
3. **shadow deployment を評価の型として持つ。** 本番トラフィックで新モデルを静かに走らせて比較するのは、オフライン評価では捕まらない退行を捕まえる。エージェントのように**入力分布が多様で長い**系では特に効く（→ [[agent-observability]]）。
4. **ダッシュボードは最初から作る。** 記事が唯一「should be implemented from the start」と強い言い方をしているのがここである。
5. **ベンダの名前リストは陳腐化する。** 2025 年 4 月時点のツールスタックであり、この層は最も速く変わる。**層の分け方**（エンジン／ルーティング／展開／監視／クラウド）のほうが持ちが良い。

## 用語と略称

- **LLM serving** = 訓練済みモデルを本番システムでの推論に利用可能にする過程
- **SLA** = Service Level Agreement（サービス水準合意）
- **PEFT** = Parameter-Efficient Fine-Tuning（→ [[parameter-efficient-fine-tuning]]）
- **LoRA** = Low-Rank Adaptation（低ランク適応）
- **merged / modular adaptation** = 適応した重みを事前に統合するか、動的に読み込むか
- **model cascade（モデルカスケード）** = 小さく安いモデルから始め、信頼度が足りなければ大きいモデルへ引き上げる構成
- **router（ルータ）** = クエリの特徴から最初に適切なモデルへ振り分ける軽量な分類器またはルール
- **early exit（早期終了）** = 確信が得られた時点で計算・生成を打ち切ること
- **speculative decoding（投機的デコーディング）** = 小さなドラフトモデルの提案を大きなモデルで検証する高速化
- **shadow deployment（シャドー展開）** = 候補モデルを本番と並行して動かし、ユーザーに影響を与えずに比較すること
- **synthetic evaluation（合成評価）** = LLM を評価者として使う評価
- **QLoRA / AWQ / GPTQ / GGUF** = 量子化の代表手法（→ [[model-quantization]]）
- **vLLM / Triton / Ray Serve / BentoML** = serving エンジンとルーティングのフレームワーク
- **MLflow / LangSmith / Weights & Biases** = 実験追跡と可観測性のプラットフォーム

## 関連ページ

- [[llm-serving-systems]] — 本記事が属する概念ページ。ルーティング／カスケードとツールスタックの節が本記事に依拠している
- [[llm-programming-systems]] — 本記事の「ツールスタック」の一段下、LLM プログラムをどう書くかの層
- [[summaries/2025-llm-serving-techniques]] — 同じ主題の技術側。本記事が名前だけ挙げる機構（continuous batching・PagedAttention）の中身はそちらにある
- [[summaries/2022-gptq]] — 本記事の「Gradient Post-Training Quantization」という誤りを正す原典
- [[model-quantization]] — 量子化 4 手法の詳細
- [[parameter-efficient-fine-tuning]] — LoRA とアダプタ。本記事の「統合適応 vs モジュラー適応」の接点
- [[agent-observability]] — シャドー展開・ログ・ダッシュボード。LangSmith もこちらで扱う
- [[agent-evaluation]] — 合成評価（LLM-as-a-judge）とそのバイアス
- [[llm-inference-optimization]] — 「モデルファミリーの階層化」という本記事のカスケードに対応する記述
