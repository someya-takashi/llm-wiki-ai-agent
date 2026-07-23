# AI Agent LLM Wiki — スキーマ

このリポジトリは Andrej Karpathy が提唱した「LLM Wiki」パターンに基づく、**AI Agent（AI エージェント, 大規模言語モデルを推論エンジンとして、目標を与えられると自ら計画を立て、ツールを呼び出し、環境からの結果を観測しながら多段階のタスクを遂行するシステム）** 領域のパーソナル・ナレッジベースです。エージェント本体だけでなく、その土台となる LLM（Large Language Model, 大規模言語モデル）と開発基盤まで幅広く扱います。具体的には agent loop（観測→思考→行動を繰り返す実行ループ）や ReAct 型の推論・行動連鎖、tool use / function calling（モデルが外部ツールを構造化された引数で呼び出す仕組み）、planning（計画立案）と self-reflection（自己反省による軌道修正）、memory（短期・長期記憶）、RAG（Retrieval-Augmented Generation, 検索により外部知識を注入して生成する手法）、multi-agent systems（複数エージェントの協調・分業）、coding agent（コーディングエージェント）や computer use / web agent といった応用、MCP（Model Context Protocol, ツールやデータソースをモデルに接続する標準プロトコル）などの連携基盤、エージェント評価（SWE-bench, GAIA, WebArena 等のベンチマーク）、prompt / context engineering（プロンプト・コンテキスト設計）、post-training（RLHF や検証可能報酬による強化学習）、推論の高速化・サービング、安全性とガードレール、observability（トレーシング・可観測性）までを含みます。LLM（あなた）が wiki を**読み・書き・更新する側**、ユーザーは**情報源のキュレーションと質問**を担当します。ユーザーは wiki を直接編集することはほぼありません。

このファイルは「あなた（LLM）」のための運用ルール書（スキーマ）です。ingest / query / lint の各オペレーションは skill として切り出してあり（§3）、それらの skill は本ファイルのスキーマ規約に従って実行してください。

---

## 1. ディレクトリ構成

```
AI Agent/
├── CLAUDE.md                ← このファイル（スキーマ）
├── .claude/skills/          ← オペレーション skill（ingest / query / lint）
├── raw/                     ← 原典（immutable, LLM は読むだけ）
│   ├── papers/              ← 論文 PDF（arXiv 等）
│   ├── articles/            ← ブログ記事・Webクリップ等の markdown
│   ├── images/              ← 記事に紐づかない単独画像
│   └── assets/              ← 記事内画像のローカル保存先
└── wiki/                    ← LLM が完全所有する markdown 群
    ├── index.md             ← カタログ（全ページの一覧）
    ├── log.md               ← 時系列ログ（append-only）
    ├── overview.md          ← AI Agent 全体の総括ページ（随時更新）
    ├── summaries/           ← 原典 1 件につき 1 ページの要約・解説
    ├── translations/        ← 原典の全文翻訳（要約せず一文ずつ正確に）
    ├── concepts/            ← 概念・カテゴリ（抽象レベル）の解説ページ
    └── questions/           ← query で得た成果物（比較表・分析等）を保存
```

### 命名規約

- ファイル名は `kebab-case.md`（例: `tool-use-and-function-calling.md`、`retrieval-augmented-generation.md`）
- 原典の wiki ページ（summaries/translations）は元ファイル名にあわせる（例: `raw/papers/2022-react.pdf` → `wiki/summaries/2022-react.md`, `wiki/translations/2022-react.md`）
- **概念ページ（concepts/）は「抽象的な概念・カテゴリ」を単位にする**。スラグは概念の英語正式名称（略称ではなくフルネーム）。
  - 例: `llm-agents`, `agent-loop`, `tool-use-and-function-calling`, `reasoning-and-planning`, `self-reflection`, `agent-memory`, `retrieval-augmented-generation`, `multi-agent-systems`, `context-engineering`, `model-context-protocol`, `agent-frameworks`, `coding-agents`, `computer-use-agents`, `web-agents`, `agent-evaluation`, `agent-safety-and-guardrails`, `agent-observability`, `reinforcement-learning-from-human-feedback`, `test-time-compute`, `llm-inference-optimization`, `parameter-efficient-fine-tuning`, `transformer-architecture`。
- **landmark な個別手法・モデル・製品は専用ページを作らず、属する概念ページ内に「代表手法」として記述する**。
  - 例: ReAct・Chain-of-Thought・Tree of Thoughts は `[[reasoning-and-planning]]`、Reflexion は `[[self-reflection]]`、Toolformer・Gorilla は `[[tool-use-and-function-calling]]`、MemGPT は `[[agent-memory]]`、AutoGPT・LangChain / LangGraph・AutoGen・CrewAI・Claude Agent SDK は `[[agent-frameworks]]`、SWE-agent・Devin・Claude Code・Cursor は `[[coding-agents]]`、GPT-4・Claude・Llama・Gemini など個別モデルは `[[transformer-architecture]]` や該当する応用の概念ページ内で扱う。
  - 仕組み・実験・限界は概念ページ内の該当手法の項に、俯瞰と位置づけは概念ページ冒頭に書く。ある手法の言及が増えて 1 ページに収まらなくなったら、ユーザーに相談のうえ概念ページの分割や新設を検討する（§7）。
- **主要なベンチマーク・データセット（例: SWE-bench, GAIA, WebArena, AgentBench, τ-bench, HumanEval, MMLU）や評価指標（pass@k, resolve rate, success rate, tool-call accuracy, コスト／レイテンシ等）も専用ページは作らず**、それを評価に使う概念ページ（多くは `[[agent-evaluation]]`）、または該当する `summaries/` ページ内で説明・言及する。
- **個別の人物・組織・企業（OpenAI, Anthropic, Google DeepMind 等）の専用ページは作らない**。それらは関連する concept、または該当する `summaries/` ページ内で言及する。
- 略称（LLM, RAG, MCP, CoT, ToT, RLHF, SFT, LoRA, API, SDK, HITL 等）は専用ページを作らず、対応する正式名称の概念ページ（retrieval-augmented-generation, model-context-protocol, reasoning-and-planning, reinforcement-learning-from-human-feedback, parameter-efficient-fine-tuning 等）にリダイレクトする位置づけで `index.md` に併記する。

### 言語ポリシー

- **wiki 内の解説文（summaries / concepts / questions / overview / index / log）は日本語**で書く
- **translations のみ、原典が英語であれば日本語訳**を作成する（原典が日本語であれば翻訳は作成しない）
- 固有名詞・術語は無理に和訳せず、初出時に「Retrieval-Augmented Generation（RAG, 外部知識を検索してプロンプトに与え、それを根拠に生成させる手法）」のように原語＋略称＋短い注釈を添える

### リンク規約

- 内部リンクは Obsidian の `[[wikilink]]` 記法を使う（`[[tool-use-and-function-calling]]` のように slug を直接書く）
- まだ存在しないページへのリンク（dangling link）も許容する。`lint` 時に未作成ページとして検出する
- 原典への参照は `[[summaries/2022-react]]` のようにディレクトリ込みで書く（同名スラグの衝突を避けるため）

---

## 2. Frontmatter 規約

すべての wiki ページに YAML frontmatter を付与する。Obsidian Dataview で集計できるようにする。

**重要（Obsidian 互換）**: frontmatter 内で `[[wikilink]]` を値に使うときは**必ず引用符で囲む**（`"[[...]]"`）。裸の `[[...]]` は YAML では入れ子配列と解釈され、Obsidian で「無効なプロパティ」になる。リンクが複数あるフィールド（`related` / `summaries` / `summaries_used`）は **YAML ブロックリスト**（各行 `  - "[[...]]"`）にする。単一リンク（`translation` / `source_page`）は `key: "[[...]]"` と引用符付きの 1 行で書く。本文（frontmatter 外）の `[[wikilink]]` は引用符不要（通常どおり）。

### summaries/*.md

```yaml
---
type: summary
source_path: raw/papers/2022-react.pdf
source_kind: paper            # paper | article | blog | docs | video | podcast
title: "ReAct: Synergizing Reasoning and Acting in Language Models"
authors: [Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, Yuan Cao]
year: 2022
venue: ICLR 2023
ingested: 2026-07-23
tags: [llm-agents, reasoning-and-planning, tool-use-and-function-calling, react]
translation: "[[translations/2022-react]]"
---
```

`source_kind` の `docs` は、公式ドキュメント（フレームワークのガイド、プロトコル仕様、モデルの system card 等）を取り込んだ場合に使う。

### translations/*.md

```yaml
---
type: translation
source_path: raw/papers/2022-react.pdf
source_page: "[[summaries/2022-react]]"
original_language: en
translated_to: ja
translated_at: 2026-07-23
---
```

### concepts/*.md

```yaml
---
type: concept
aliases: [RAG, 検索拡張生成]
tags: [retrieval-augmented-generation, agent-memory, llm-agents]
related:
  - "[[agent-memory]]"
  - "[[context-engineering]]"
summaries:                      # この概念の根拠となる原典（summaries/）
  - "[[summaries/2020-rag]]"
  - "[[summaries/2022-react]]"
updated: 2026-07-23
---
```

### questions/*.md

```yaml
---
type: question
asked: 2026-07-23
question: "ReAct と Reflexion は、失敗からの回復という観点でどう違うか？"
summaries_used:
  - "[[summaries/2022-react]]"
  - "[[summaries/2023-reflexion]]"
---
```

---

## 3. オペレーション（ingest / query / lint）

主要オペレーションは **skill として切り出してある**。実行時は対応する skill の手順に従うこと（手順の本体は各 SKILL.md にあり、本ファイルには重複させない）。

- **ingest（原典の取り込み）** — `.claude/skills/ingest/SKILL.md`
  raw/ に置かれた論文・記事を読み、翻訳ファイル（translations）・要約ページ（summaries）・関連する概念ページ（concepts）を作成／更新し、index/log を更新する。**翻訳・要約・画像の具体テンプレと書式もこの skill 内に置いてある**。
- **query（質問への回答）** — `.claude/skills/query/SKILL.md`
  index から関連ページを辿り、`[[wikilink]]` 引用付きで回答する。必要なら成果物を `questions/` として保存する。
- **lint（健康診断）** — `.claude/skills/lint/SKILL.md`
  矛盾・古い記述・孤立ページ・dangling link・欠落クロスリファレンス・概念⇔要約間のリンク不備・データギャップを点検し、一覧で提示する。

共通方針：

- ingest は基本 **1 件ずつ**、ユーザーと対話しながら進める（1 件の ingest で 3〜8 ページが触られるのが普通）。
- 下記 §4（「機械的なまとめ」にしないルール）は、`translations/` を除くすべてのページ生成、および query の回答に **常時適用**する。

---

## 4. 「機械的なまとめ」にしない（最重要ルール）

`wiki/translations/` 以外のすべてのページ（summaries / concepts / questions / overview）および query の回答で守ること：

1. **略称は必ず初出時に展開する**。`RAG` → `RAG（Retrieval-Augmented Generation, 外部知識を検索してプロンプトに与え、それを根拠に生成させる手法）` のように、**展開＋短い意味付け**をセットで書く。AI エージェント領域で頻出する略称の例：LLM（Large Language Model, 大規模言語モデル）, RAG, MCP（Model Context Protocol, ツールやデータソースをモデルに接続する標準プロトコル）, CoT（Chain-of-Thought, 思考過程を明示的に出力させて推論精度を上げる手法）, ToT（Tree of Thoughts, 思考を木構造で探索する手法）, RLHF（Reinforcement Learning from Human Feedback, 人間の選好を報酬にした強化学習）, RLVR（Reinforcement Learning with Verifiable Rewards, 正誤を機械的に判定できる報酬を使う強化学習）, SFT（Supervised Fine-Tuning, 教師ありファインチューニング）, PEFT/LoRA（Parameter-Efficient Fine-Tuning / Low-Rank Adaptation, 少数パラメータのみ更新する微調整）, MoE（Mixture of Experts, 入力ごとに一部の専門家サブネットのみを使う構造）, HITL（Human-in-the-Loop, 人間が途中で承認・介入する運用）, KV cache（Key-Value cache, 生成済みトークンの中間状態を再利用する高速化機構）, TTFT/TPS（Time To First Token / Tokens Per Second, 応答遅延とスループットの指標）, API / SDK。
2. **難概念は補足を入れる**。専門用語（例: agent loop（エージェントループ, 観測→思考→行動を繰り返す実行ループ）, tool call / function calling（モデルが JSON スキーマに沿った引数でツールを呼ぶ仕組み）, trajectory / rollout（エージェントが辿った行動列）, scratchpad（中間の思考を書き溜める作業領域）, context window（コンテキストウィンドウ, モデルが一度に読める最大トークン数）, embedding（埋め込み, テキストを意味を保った数値ベクトルに変換したもの）, chunking / reranking（文書分割・再ランキング）, orchestrator-worker（親エージェントが子エージェントに分業させる構成）, subagent（サブエージェント）, verifier / reward model（出力の正しさを判定するモデル）, prompt injection（外部入力に埋め込まれた指示でエージェントを乗っ取る攻撃）, sandboxing（サンドボックス化, 危険な操作を隔離環境に閉じ込めること）, guardrails（ガードレール, 入出力を検査して逸脱を防ぐ仕組み）, test-time compute（推論時に計算量を増やして精度を上げる考え方））が出てきたら、その場で 1 文の補足説明を付ける。リンクで概念ページに飛べる場合でも、文脈で必要なら補足を残す。
3. **初学者の読者を想定する**。学部高学年〜修士 1 年程度の機械学習初学者、あるいは LLM アプリ開発を始めたばかりのエンジニアが読んで「何のことを言っているか分からない」段落を作らない。逆に、自明な内容を冗長に説明するのも避ける。
4. **原典の章立てをそのままコピーしない**。要約ページの構造は ingest skill の要約テンプレートに従い、原典の構造を「再解釈」した形にする。
5. **「研究の意義」を自分の言葉で説明する**。原典の Abstract をなぞるだけにしない。「なぜこの結果が AI エージェント／LLM 応用にとって重要なのか」を 1〜2 文加える。
6. **既存 wiki との接続を明示する**。「SWE-agent は [[coding-agents]] において、[[tool-use-and-function-calling]] のツール設計をエージェント向けに作り直す（agent-computer interface）ことで、[[agent-evaluation]] の代表ベンチマークである SWE-bench の解決率を大きく引き上げた」のように、既存知識（概念・代表手法）と結びつける一文を必ず入れる。

逆に、`wiki/translations/` では上記ルールは **適用しない**。翻訳ファイルでは補足・解釈・接続づけは一切せず、原典に忠実に訳すことに専念する。

---

## 5. index.md と log.md の運用

### index.md

カテゴリ別の全ページカタログ。1 行 = 1 ページ。フォーマット：

```markdown
- [[<slug>]] — <一行の説明>
```

セクション：
- Overview
- Summaries（さらに paper / article / docs 等に分けてよい）
- Translations
- Concepts
- Questions

略称リダイレクト（例: `RAG → [[retrieval-augmented-generation]]`、`MCP → [[model-context-protocol]]`、`CoT → [[reasoning-and-planning]]`、`RLHF → [[reinforcement-learning-from-human-feedback]]`）は対応するセクション内に併記してよい。ingest / query で新規ページを作るたびに必ず更新する。

### log.md

時系列の append-only ログ。

```markdown
## [YYYY-MM-DD] ingest | <タイトル>

- 取り込み: `raw/papers/2022-react.pdf`
- 作成: [[summaries/2022-react]], [[translations/2022-react]], [[concepts/reasoning-and-planning]]
- 更新: [[concepts/tool-use-and-function-calling]], [[overview]], [[index]]
- メモ: ...
```

`grep "^## \[" log.md | tail -10` で直近の動きを追えるよう、必ずこのプレフィックス形式を守る。スキーマ変更は `## [YYYY-MM-DD] schema-update | <要点>` で記録する（§7）。

---

## 6. ツール

- **Obsidian**：wiki の閲覧・グラフビュー確認。ユーザーが裏側で開いている。
- **Obsidian Web Clipper**：記事を markdown 化して `raw/articles/` に保存。
- **Marp**：スライド出力が必要な質問への回答に使う（任意）。
- **Dataview**：frontmatter ベースの動的集計（任意）。

検索は規模が小さいうちは index.md ベースで十分。ページ数が増えてきたら `qmd` 等の導入を検討する。

---

## 7. このスキーマ自体について

このスキーマはユーザーと共進化する。運用していく中で「このカテゴリが必要」「このルールは緩めたい」となったら、ユーザーに提案してこの CLAUDE.md（およびオペレーション手順を持つ `.claude/skills/` 配下の SKILL.md）を更新する。スキーマ変更は log.md にも `## [YYYY-MM-DD] schema-update | <要点>` として記録する。

AI エージェント領域は動きが速く、用語も陳腐化しやすい（例: 「AutoGPT 的な自律ループ」から「ツール＋検証ループ」へ、あるいは MCP のような新プロトコルの登場）。**古くなった記述を見つけたら、そのページを更新するか、`lint` の結果としてユーザーに提示する**こと。
