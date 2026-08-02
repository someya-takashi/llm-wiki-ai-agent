---
type: summary
source_path: raw/papers/Security and Privacy Challenges of Large Language Models_ A Survey.md
source_kind: paper
title: "Security and Privacy Challenges of Large Language Models: A Survey"
authors: [Badhan Chandra Das, M. Hadi Amini, Yanzhao Wu]
year: 2024
venue: "arXiv 2402.00888"
ingested: 2026-08-01
tags: [agent-safety-and-guardrails, llm-agents, prompt-injection, jailbreaking, privacy, survey]
translation: "[[translations/2024-llm-security-privacy-survey]]"
---

# LLM のセキュリティ・プライバシー攻撃と防御 — 脅威分類のサーベイ

> 原典: [[translations/2024-llm-security-privacy-survey]] ・ `raw/papers/Security and Privacy Challenges of Large Language Models_ A Survey.md`
> 著者・年: Badhan Chandra Das, M. Hadi Amini, Yanzhao Wu（Florida International University）・2024-02（arXiv:2402.00888）

## 一言まとめ

LLM（Large Language Model, 大規模言語モデル）への攻撃を **セキュリティ攻撃（プロンプトハッキング＝プロンプトインジェクション・ジェイルブレイク、敵対的攻撃＝バックドア・データポイズニング）** と **プライバシー攻撃（勾配漏洩・メンバーシップ推論・PII 漏洩）** に体系分類し、それぞれの代表手法・防御機構・応用ドメイン別リスク・研究ギャップを一望した早期のサーベイ。この wiki の [[agent-safety-and-guardrails]] が扱う脅威の「攻撃側カタログ」を提供する原典である。

## 背景と問題意識

2024 年初頭、ChatGPT/GPT-4 の普及で LLM の実運用が一気に広がった一方、その脆弱性はセキュリティ・プライバシーの両面で体系的に整理されていなかった。既存のサーベイは断片的で（表2 の比較表——ジェイルブレイクのみ、プライバシーのみ、等）、攻撃・防御・応用リスクを横断して 1 枚の地図にした文献がなかった。本サーベイは、学習データ側（ポイズニング・勾配漏洩・記憶による PII 漏洩）とユーザー側（プロンプト経由の攻撃）の両方を、HIPAA・GDPR・CCPA といった法規制の文脈も添えて統合する。

前提の整理も明快で、**セキュリティ＝システムの保護**（無許可アクセス・改変・誤作動・サービス拒否の防止）、**プライバシー＝個人情報の制御**（誰がアクセスできるかを個人が決める）と定義し、攻撃が目標を共有しうること（バックドアとポイズニングは共に「誤作動の誘発」、プロンプトインジェクションとジェイルブレイクは共に「機密情報の奪取や制約回避」）を重なり合う集合として図示する（図2）。

## 脅威分類

<figure>

![](../../raw/assets/2024-llm-security-privacy-survey/x2.png)

<figcaption>図2（再掲）: LLM の脆弱性の分類。大枠「汎用 LLM 脆弱性」の中に、Security Attack（Adversarial Attack＝Backdoor・Poisoning、Prompt Hacking＝Prompt Injecting・Jailbreaking）と Privacy Attack（PII Leakage・Membership-inference・Gradient Leakage）が配置され、重なり合う領域が攻撃間で共有される目標を示す。</figcaption>
</figure>

### A. セキュリティ攻撃

**プロンプトハッキング**——入力プロンプトを操作して出力を誘導する。
- **プロンプトインジェクション**: モデルの出力を掌握する攻撃。基本形は「以前の指示を無視せよ」で、**goal hijacking（目標乗っ取り: 元の目標を差し替えて特定フレーズを吐かせる）**と **prompt leaking（プロンプト漏洩: システムプロンプトそのものを再現させる）**に分かれる。HOUYI（Web インジェクション着想のブラックボックス攻撃、3 要素構成）、AutoPrompt（勾配誘導で自動生成）、P2SQL インジェクション（Langchain 経由で DB を操作）が代表。**アプリ統合型 LLM が外部 Web の汚染テキストを読む「間接インジェクション」**の危険が指摘される——これは現在のエージェントの最大の実務脅威（→ [[agent-safety-and-guardrails]]）の一次的な整理。
- **ジェイルブレイク**: モデルに課された安全制約を回避する。元は iOS 等の「脱獄」の転用語。先駆は **DAN（Do Anything Now）**。プロンプトのパターンは **pretending（ふりをする・ロールプレイ）・attention shifting（物語生成へ注意そらし）・privilege escalation（制約を迂回でなく破らせる）**の 3 型。理論的な整理として Wei et al. の **2 つの失敗モード**が重要——**competing objectives（「常に指示に従う」能力と安全目標の衝突。prefix injection・refusal suppression）**と **mismatched generalization（入力が事前学習分布の内側だが安全訓練データの外＝OOD に落ちる）**。自動化（MASTERKEY 成功率 21.58%、遺伝的アルゴリズム、PAIR は 20 クエリ未満でブラックボックス達成）・マルチモーダル（視覚敵対例）・多段（多段ジェイルブレイクは防御下でも PII を漏らす）へ発展。

**敵対的攻撃**——入力を意図的に摂動して誤出力させる（$x' = x + \eta$）。
- **バックドア攻撃**: 汚染サンプルで隠れた機能を仕込み、良性入力では正常・トリガー入力で異常動作。トリガー源で input/prompt/instruction/demonstration の 4 型。BadPrompt・BadGPT（RL ファインチューニングにトリガー語「cf」）・ProAttack（外部トリガー不要のクリーンラベル）・BGMAttack・CBA（複合トリガーで隠密性向上）。重要な知見: **LLM のバックドア化は固定能力の分類器より難しい**（どうプロンプトされてもトリガーで発動しつつ他タスクへの影響を最小化する必要がある）。
- **データポイズニング攻撃**: 訓練データに毒を混ぜて意思決定を歪める。TROJANLM・TROJANPUZZLE（静的解析を逃れる巧妙な毒）。LLM は単語レベル（タイポ）・文レベル（気散らし）の敵対的入力に脆弱。

### B. プライバシー攻撃

- **勾配漏洩攻撃**: 連合学習（FL）等で共有される勾配から訓練データを再構成。**TAG**（Transformer ベース LM から最大 88.9% のトークンを復元）・**LAMP**（補助 LM で自然文へ誘導、バッチサイズ >1 で初成功）。多くは画像モデル向けで LM 研究は少ない。
- **メンバーシップ推論攻撃（MIA）**: あるサンプルが訓練データに含まれたかを判定。過適合による損失値の低さを突く（LOSS 攻撃）。**shadow training**（標的の振る舞いを模したシャドウモデルで攻撃分類器を訓練）が Shokri et al. の原型。近傍攻撃・尤度比ベースなど。医療・金融データで深刻。
- **PII 漏洩攻撃**: 個人識別情報の抽出。TAB attack・ProPILE（PII を尋ねると尤度で答える）・Carlini et al. の**記憶（memorization）による訓練データ抽出**（GPT-2 から逐語系列を復元）。文レベル差分プライバシーでも約 3% の PII が漏れる。

### 防御機構（§6）

- **プロンプトインジェクション**: 予防（前処理で注入指示除去・言い換え・再トークン化・データ分離）＋検出（応答ベース／プロンプトベース。パープレキシティ検出）。P2SQL には DB 権限強化・Auxiliary LLM Guard。
- **ジェイルブレイク**: 内容フィルタ・**system-mode self-reminder**（成功率 67.21%→19.34%）・レッドフラグ語検出・**SmoothLLM**（入力を複製摂動して出力を集約、成功率 1/100〜1/50）。
- **バックドア**: Fine-mixing・CUBE（HDBSCAN クラスタリング）・RAP・ONION・**MDP**（マスクへの感度差で毒を検出、ただし新型攻撃には非力）。多くはホワイトボックス前提でブラックボックス防御が欠落。
- **プライバシー**: 差分プライバシー（DP-SGD）・ノイズ挿入・準同型暗号（勾配漏洩）／ドロップアウト・モデルスタッキング・正則化（MIA）／重複除去・PII スクラビング（NER でタグ付け）（PII 漏洩）。**共通の限界: ほぼすべての防御がモデル有用性とのトレードオフを抱える**。

## 応用ドメイン別リスク（§7）

交通（事故報告のバイアス・自動運転からの個人データ漏洩）・医療（誤った治療推奨・確率的性質による再現性欠如で「現時点で統合は非推奨」）・教育（誤情報・過度の依存）・ガバナンス（GPT-4 で議員情報を抽出しスピアフィッシング）・科学（Galactica が虚偽生成でローンチ直後に停止・査読の質低下）。「参入障壁がコーディングからプロンプトエンジニアリングへ移る」という指摘も。

## 既存 wiki との接続

- **[[agent-safety-and-guardrails]] の攻撃側カタログ**。この wiki の安全性ページは Anthropic の CoT 忠実性・DSec sandboxing など**防御・監視**側が厚かったが、本サーベイは**攻撃の分類体系**（何から守るのか）を供給する。2023 年サーベイ（[[summaries/2023-llm-agents-survey]] §6.3）が「行動空間を持つエージェントでは敵対的攻撃が破壊的行動になる」と概念的に警告したのに対し、本サーベイは個々の攻撃手法（DAN・HOUYI・MASTERKEY・TAG・ProPILE 等）まで降りて具体化した。
- **prompt injection の一次的整理**。この wiki が lint のデータギャップとして挙げていた「prompt injection の一次原典不足」を埋める。特に「アプリ統合型 LLM が外部の汚染テキストを読む間接インジェクション」は、[[computer-use-agents]]（画面全部が入力）・[[retrieval-augmented-generation]]（検索結果をコンテキストに戻す）・[[tool-use-and-function-calling]]（ツール出力の取り込み）のすべてに刺さる攻撃面である。
- **ジェイルブレイクの失敗モード理論**は [[reinforcement-learning-from-human-feedback]] の安全訓練（RLHF）の限界と直結する——competing objectives は「指示追従と無害性の報酬の衝突」、mismatched generalization は「安全訓練データの分布外」であり、どちらも事後訓練で安全性を付与することの構造的限界を示す。[[summaries/2025-cot-faithfulness]] の「reward hack を >99% 悪用しつつ CoT に出さない」という監視の限界と併読すると、攻撃・監視の両側から RLHF 安全化の穴が見える。
- **記憶（memorization）による PII 漏洩**は [[agent-memory]] の裏面——「モデルが訓練データを覚える」ことは能力であると同時に漏洩経路である。エージェントが会話履歴やファイルに PII を蓄える設計（[[context-engineering]] のファイルシステム外部化）は、この攻撃面を運用側にも広げる。

## 限界・批判的視点

- **2024 年 2 月時点のスナップショット**。対象は ChatGPT/GPT-4・BERT 系 PLM が中心で、記述の多くが「攻撃・防御は小さな LM で評価済みだが LLM では未検証」という留保つき（著者自身が繰り返し明言）。以後 2 年で登場した推論モデル・エージェント・MCP・computer use 固有の攻撃面（間接インジェクションの実運用被害）は射程外。
- **サーベイであり一次実験はない**。数値（MASTERKEY 21.58%・self-reminder 67→19%・TAG 88.9% 等）はすべて引用元の値で、評価条件は揃っていない。
- **エージェント固有の脅威への言及が薄い**。ツール権限の悪用・マルチエージェントでの汚染伝播・自律ループの暴走といった、行動空間を持つエージェント特有のリスクは、応用リスク（§7）で断片的に触れるに留まる。この wiki では [[agent-safety-and-guardrails]] がその差分（DSec の 4 段サンドボックス・CUA の資格情報平文渡し等）を補う。
- 防御の**共通の弱点＝有用性とのトレードオフ**が繰り返し確認されるが、そのトレードオフを定量化する統一指標がない（パープレキシティくらいしかない、と著者も認める）ことが、この分野の評価の未成熟を示す。

## 実装・運用上の示唆

- **攻撃を分類軸で持つ**: 「プロンプト経由か学習データ経由か」「セキュリティ（誤作動）かプライバシー（漏洩）か」の 2 軸で脅威を棚卸しすると、防御の抜けが見える。エージェントでは特にプロンプト経由の**間接インジェクション**（ツール出力・検索結果・ファイル内容）を第一に想定する。
- **ジェイルブレイクは根絶できない前提で設計する**: competing objectives と mismatched generalization は事後訓練の構造的穴であり、入力フィルタ（レッドフラグ語）だけでは巧妙なプロンプトを止められない。SmoothLLM のような入力摂動＋集約や、self-reminder のような system prompt 側の防御を多層で重ねる。
- **PII は「覚えさせない」が第一線**: 重複除去・PII スクラビング（NER）で訓練前に落とし、出力側に検出モジュール（機密検出で拒否/マスク）を置く。差分プライバシーは効くが有用性を削る。
- **防御は必ず有用性コストとセットで評価する**: 「攻撃成功率をどれだけ下げたか」だけでなく「通常タスクの性能・レイテンシをどれだけ損なったか」を併記する（→ [[agent-evaluation]] の「スコアとコストの併記」と同じ規律）。

## 用語と略称

- **PII** = Personally Identifiable Information（個人識別情報。名前・電話・SSN・医療記録等）
- **プロンプトインジェクション（PI）** = 入力プロンプトでモデル出力を掌握する攻撃。**goal hijacking**（目標差し替え）／**prompt leaking**（システムプロンプト再現）
- **ジェイルブレイク** = 安全制約の回避。**DAN**（Do Anything Now）／**pretending・attention shifting・privilege escalation**（プロンプトパターン）／**competing objectives・mismatched generalization**（Wei et al. の 2 失敗モード）
- **バックドア攻撃 / データポイズニング攻撃** = 隠れ機能の埋め込み／訓練データへの毒混入（敵対的攻撃の 2 型）
- **勾配漏洩攻撃** = 共有勾配から訓練データを再構成（TAG・LAMP）。**FL** = Federated Learning（連合学習）
- **MIA** = Membership Inference Attack（あるサンプルが訓練データにあったかを判定）。**shadow training**（シャドウモデルで攻撃分類器を訓練）
- **memorization** = モデルが訓練例を逐語的に覚える現象（PII 漏洩の経路）
- **DP / DP-SGD** = Differential Privacy / 差分プライバシー確率的勾配降下法（プライバシー防御の主力）
- **perplexity** = パープレキシティ（言語モデルの予測の不確かさ。プロンプト危殆化やバックドア毒の検出に使う）
- **SmoothLLM / self-reminder / MDP / CUBE / RAP / ONION** = 代表的な防御手法
- **HIPAA / GDPR / CCPA** = 米医療・EU・カリフォルニアのプライバシー法規
- **NER** = Named Entity Recognition（固有表現認識。PII スクラビングに使う）
- **PLM / MLM** = Pretrained / Masked Language Model
- **secure multi-party computation** = 安全なマルチパーティ計算（将来の防御候補）

## 関連ページ

- [[agent-safety-and-guardrails]] — 本サーベイを攻撃側カタログの主要根拠とする安全性の総論
- [[reinforcement-learning-from-human-feedback]] — ジェイルブレイクの 2 失敗モードが突く安全訓練の限界
- [[tool-use-and-function-calling]] / [[computer-use-agents]] / [[retrieval-augmented-generation]] — 間接インジェクションの攻撃面
- [[agent-memory]] / [[context-engineering]] — 記憶・外部化が広げる PII 漏洩面
- [[summaries/2023-llm-agents-survey]] — エージェント側の安全性の概念的整理（§6.3）
- [[summaries/2025-cot-faithfulness]] — 監視（CoT モニタリング）側から見た安全化の限界
- [[summaries/2025-llamafirewall]] — 本サーベイが列挙した防御手法を、実際に 1 つの系へ組み上げて本番投入し測定した側の一次資料
