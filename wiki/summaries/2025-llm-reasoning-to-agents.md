---
type: summary
source_path: raw/papers/From LLM Reasoning to Autonomous AI Agents_ A Comprehensive Review.md
source_kind: paper
title: "From LLM Reasoning to Autonomous AI Agents: A Comprehensive Review"
authors: [Mohamed Amine Ferrag, Norbert Tihanyi, Merouane Debbah]
year: 2025
venue: "arXiv 2504.19678"
ingested: 2026-08-01
tags: [llm-agents, agent-evaluation, model-context-protocol, agent-frameworks, multi-agent-systems, survey]
translation: "[[translations/2025-llm-reasoning-to-agents]]"
---

# LLM 推論から自律 AI エージェントへ — ベンチマーク・フレームワーク・プロトコルの横断サーベイ

> 原典: [[translations/2025-llm-reasoning-to-agents]] ・ `raw/papers/From LLM Reasoning to Autonomous AI Agents_ A Comprehensive Review.md`
> 著者・年: Ferrag, Tihanyi, Debbah（Guelma Univ / TII / ELTE / Khalifa Univ）・2025-04（arXiv:2504.19678）

## 一言まとめ

LLM（Large Language Model, 大規模言語モデル）と自律 AI エージェントを、**評価ベンチマーク（約 60 個・8 カテゴリ分類）・エージェントフレームワーク（LangChain〜Agents SDK）・11 応用分野・エージェント間プロトコル（ACP/MCP/A2A）・6 つの未解決課題**という 5 軸で横断的に整理した統合サーベイ。断片化していたこの領域を「単一のロードマップ」にまとめることを狙った 2025 年時点のカタログで、この wiki では特に**エージェント間通信プロトコルを扱う初めての原典**である。

## 背景と問題意識

2025 年前半、LLM エージェントの評価ベンチマーク・フレームワーク・協調プロトコルは急増したが、統一された分類法も横断サーベイもなく、状況は断片化していた（既存サーベイは「評価だけ」「RAG だけ」「マルチエージェントの通信だけ」と 1〜2 側面に偏る、と Table I で対比）。本サーベイは、ベンチマーク・フレームワーク・応用・プロトコル・課題を **1 本に統合した最初のサーベイ**を標榜する。エージェントを「推論と行動を組み合わせて複雑なデジタル環境と相互作用する」ものと捉え、ReAct・MCTS・データ合成（Learn-by-Interact・AgentGen・AgentTuning）・強化学習といった強化手法の系譜を踏まえたうえで、評価と接続基盤を地図化する。

## 内容の 5 軸

### 1. ベンチマーク（§III・約 60 個を 8 カテゴリに分類）

2019〜2025 年のベンチマークを、**学術・一般知識推論／数学的問題解決／コードとソフトウェア工学／事実の接地と検索／ドメイン固有／マルチモーダル・embodied／タスク選択／agentic・対話的**の 8 カテゴリに分類（図2）。個々の紹介では「現在の最先端でも解けない」ことを繰り返し強調する: HLE（Humanity's Last Exam, 専門家級 3,000 問）は SOTA でも 10% 未満、GAIA（汎用アシスタント）は人間 92% vs GPT-4+plugins 15%、τ-bench（会話エージェント）は GPT-4o でも 50% 未満、SWE-Lancer（実支払つき SE タスク）は Claude 3.5 Sonnet でも 26.2%。評価方法論の新形態として **Agent-as-a-Judge**（エージェントがエージェントを評価。人間と 90% 一致・コスト 2.29%）も紹介。→ [[agent-evaluation]]

### 2. フレームワーク（§IV-A）

LangChain・LlamaIndex・CrewAI・Swarm・GUI Agent（Claude Computer Use）・Agentic Reasoning・OctoTools・Agents SDK を、中核概念・ワークフロー・利点で整理（Table V）。あわせて **Agentic RAG** を「RAG の事実接地 × エージェントの動的適応性」の統合として位置づけ、LLM 事前学習／事後訓練／RAG／AI エージェント／Agentic RAG の 5 段階を自律性・学習・信頼性の軸で比較する（Table VI）。→ [[agent-frameworks]]

### 3. 応用（§IV-B・11 分野）

医療（診断・カウンセリング・創薬）・材料科学・生物医学・研究自動化・ソフトウェア工学・合成データ生成・金融・化学・数学・地理・マルチメディアの各分野で、既存のエージェントシステムを大量に列挙する（各分野に表と図）。カタログ性が強く、「どの分野でもエージェント化が進んでいる」ことの網羅的な証拠集。

### 4. プロトコル（§IV-C・この wiki で初出の話題）

エージェント間通信の 3 プロトコルを比較（Table XII）:
- **MCP（Model Context Protocol）**: Anthropic が 2024 年後半に導入。「AI システムと外部ツール・データソースの接続の **USB-C**」。Language Server Protocol 着想の client-server 設計で、ホストアプリが複数の軽量サーバーに接続する。ツール・データを **LLM に文脈として渡す**標準インターフェース。
- **ACP（Agent Communication Protocol）**: IBM が 2025 年に提案（BeeAI プラットフォームの中核）。MCP 着想で、ローカルファーストな**複数エージェントのオーケストレーション**（発見・委譲・テレメトリ）に焦点。
- **A2A（Agent-to-Agent Protocol）**: Google が 2025 年に導入。HTTP/SSE/JSON-RPC ベースで、**異フレームワーク・異ベンダーのエージェント間の相互運用**（記憶・ツールを共有せず、"Agent Card" で能力発見）に焦点。

三者は棲み分ける——**MCP=ツール接続（縦）、A2A=エージェント間協調（横）、ACP=ローカル編成**。→ [[model-context-protocol]]（本 ingest で新設）

### 5. 課題（§V・6 論点）

(1) 推論（Meta-CoT——CoT が明かさない潜在的な認知過程を捉える）、(2) **マルチエージェントの失敗**（Pan et al. の 14 失敗モード・3 カテゴリ。150 タスク・5 フレームワークの分析で、役割定義やオーケストレーションの調整だけでは不十分——[[summaries/2025-masft]] と同一の MAST 研究）、(3) 自動科学発見（データ汚染・仮説の妥当性）、(4) 動的ツール統合（Chain-of-Tools——凍結 LLM で未見ツールを CoT に組み込む）、(5) **RL による統合検索**（ReSearch——検索を「トークン化された行動」として推論連鎖に織り込み RL で訓練）、(6) **プロトコルの脆弱性**（MCP は分散設計ゆえ中央権威・標準認証・ロギングの欠如が攻撃面になる）。

## 既存 wiki との接続

- **[[model-context-protocol]] 新設の根拠原典**。この wiki は MCP・A2A・ACP を扱う原典を持たず、多数のページが `[[model-context-protocol]]` に dangling していた。本サーベイの §IV-C がその空白を埋める——特に「MCP=ツール接続・A2A=エージェント間・ACP=ローカル編成」の三分と、プロトコルのセキュリティ脆弱性（§V-F）は、他の原典にない一次的な整理。
- **[[agent-evaluation]] のベンチマーク地図**。この wiki の評価ページは個別ベンチマーク（SWE-bench・GAIA・τ-bench 等）を扱うが、本サーベイの 8 カテゴリ分類と「約 60 個の並列比較表」は、評価ランドスケープの俯瞰を与える。Agent-as-a-Judge（エージェントによる評価）は [[agent-evaluation]] の LLM-as-a-judge の発展形。
- **MAST の別名での再確認**。§V-B の Pan et al. の 14 失敗モード・3 カテゴリ（設計/仕様・エージェント間不整合・検証と終了）は、[[summaries/2025-masft]]（Cemri et al.）と**同一の MAST 研究**（同じ 5 フレームワーク・150 タスク・14 モード）。複数のサーベイが同じ失敗分類を引くことは、[[multi-agent-systems]] の失敗論が定着したことの傍証。
- **Agentic RAG は [[retrieval-augmented-generation]] の発展**。「RAG に自律エージェント（計画・内省・ツール利用・協調）を埋め込む」枠組みは、[[summaries/2020-rag]] の静的検索から、エージェントループ内での動的検索（[[summaries/2022-react]]・ReSearch）への系譜に位置づく。
- **フレームワーク一覧は [[agent-frameworks]] の補完**。LangChain/CrewAI/Swarm/Agents SDK は同ページが「顔ぶれは変わり続ける」とした個別ライブラリの 2025 年時点のスナップショット。

## 限界・批判的視点

- **サーベイであり一次実験はない**。すべての数値（HLE 10% 未満・GAIA 15% 等）は引用元の値で、評価条件は揃っていない。カタログ性が強く、特に §IV-B の応用は個別研究の網羅列挙で、批判的分析は薄い。
- **2025 年 4 月時点のスナップショット**。ベンチマークもプロトコルも動きが速く、MCP/A2A/ACP の仕様・採用状況・エコシステムは陳腐化しやすい（本サーベイ自体、MCP 登場からわずか半年後の記述）。
- **本文に重複・崩れが散見**（同じ研究の記述が 2 回現れる箇所、見出し番号の非連続 II-D2 欠番など）。査読前 arXiv 版のサーベイに typical な粗さ。
- **プロトコルの記述は概説レベル**。MCP/A2A/ACP の具体的なメッセージ形式・認証フロー・実装は本サーベイでは深掘りされず、「USB-C のようなもの」という比喩と役割分担の整理に留まる。より深い技術仕様は各プロトコルの一次ドキュメント（原文脚注の GitHub/公式サイト）を要する。
- 分類の網羅性と引き換えに、**個々の主張の裏づけの深さは犠牲**——「約 60 ベンチマーク」の分類は俯瞰として有用だが、各カテゴリの境界や選定基準は必ずしも明確でない。

## 実装・運用上の示唆

- **プロトコルは目的で選ぶ**: LLM にツール・データを繋ぐなら MCP、異フレームワークのエージェントを協調させるなら A2A、ローカルで複数エージェントを編成するなら ACP。三者は競合でなく補完（縦・横・ローカル）。
- **ベンチマークは「解けなさ」で選ぶ**: 現行モデルが飽和したベンチマーク（MMLU 等）でなく、HLE・GAIA・τ-bench・SWE-Lancer のような「人間と大差がある」評価を使うと、エージェントの実力差が見える。エージェント評価では agentic 対応（Table IV の "Agentic AI" 列）を確認する。
- **エージェント評価はエージェントで**: Agent-as-a-Judge は、成果だけでなく中間過程に細粒度のフィードバックを与え、人間評価の 2.29% のコストで 90% の一致を得る——コーディング等の複雑タスクの評価自動化の一手。
- **プロトコル導入時はセキュリティを設計に含める**: MCP の分散設計は中央権威・標準認証・ロギングの欠如という攻撃面を持つ（§V-F）。認証・監査ログ・状態一貫性を最初から組み込む → [[agent-safety-and-guardrails]]。

## 用語と略称

- **MCP** = Model Context Protocol（Anthropic, 2024。AI とツール・データ接続の標準。"USB-C" 比喩）
- **A2A** = Agent-to-Agent Protocol（Google, 2025。異フレームワークのエージェント間相互運用。Agent Card で能力発見）
- **ACP** = Agent Communication Protocol（IBM, 2025。BeeAI のローカルファーストなエージェント編成）
- **Agentic RAG** = 自律エージェント（計画・内省・ツール利用・協調）を RAG に埋め込んだ発展形
- **RAG** = Retrieval-Augmented Generation（検索拡張生成）
- **Meta-CoT** = CoT の潜在的な認知過程（探索・反復推論）を捉える推論の枠組み
- **Agent-as-a-Judge** = エージェントがエージェントを評価する方法論（中間フィードバック・低コスト）
- **MAST** = マルチエージェントシステムの 14 失敗モード分類（§V-B の Pan et al.、[[summaries/2025-masft]] と同一研究）
- **HLE / GAIA / τ-bench / SWE-Lancer / BFCL / MMLU** = 代表ベンチマーク（専門家学術／汎用アシスタント／会話／フリーランス SE／関数呼び出し／マルチタスク知識）
- 主要フレームワーク: **LangChain・LlamaIndex・CrewAI・Swarm・OctoTools・Agents SDK**

## 関連ページ

- [[model-context-protocol]] — 本サーベイを主要根拠に本 ingest で新設したプロトコル概念ページ
- [[agent-evaluation]] — ベンチマーク 8 カテゴリ分類・Agent-as-a-Judge
- [[agent-frameworks]] — フレームワーク一覧の 2025 年スナップショット
- [[multi-agent-systems]] — §V-B の MAST 失敗分類
- [[retrieval-augmented-generation]] — Agentic RAG・ReSearch の系譜
- [[llm-agents]] — 領域全体の総論（本サーベイは 2 年後の別視点の総論）
- [[summaries/2023-llm-agents-survey]] — 同種の包括サーベイ（brain/perception/action 版）
- [[summaries/2025-masft]] — §V-B が引く MAST 研究の一次原典
