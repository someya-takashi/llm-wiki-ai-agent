---
type: concept
aliases: [LLM エージェント, LLM-based agents, AI エージェント, AI agents, 自律エージェント, autonomous agents]
tags: [llm-agents]
related:
  - "[[agent-loop]]"
  - "[[llm-programming-systems]]"
  - "[[reasoning-and-planning]]"
  - "[[tool-use-and-function-calling]]"
  - "[[agent-memory]]"
  - "[[multi-agent-systems]]"
  - "[[agent-frameworks]]"
  - "[[agent-evaluation]]"
  - "[[agent-safety-and-guardrails]]"
summaries:
  - "[[summaries/2026-ai-scientist]]"
  - "[[summaries/2023-llm-agents-survey]]"
  - "[[summaries/2022-react]]"
  - "[[summaries/2024-building-effective-agents]]"
updated: 2026-08-02
---

# LLM Agents（LLM ベースエージェント）— 総論

**LLM（Large Language Model, 大規模言語モデル）を推論エンジン（脳）として、目標を与えられると自ら計画を立て、ツールを呼び出し、環境からの結果を観測しながら多段階のタスクを遂行するシステム**の総称。この wiki 全体の主題であり、本ページは各概念ページへのハブとなる総論である。主要な根拠原典は、分野の語彙を最初に体系化した Fudan NLP のサーベイ（[[summaries/2023-llm-agents-survey]], 2023, 686 文献）。

## 定義と系譜

「エージェント」の概念は哲学（アリストテレス・ヒュームの「欲求・信念・意図を持ち行動する実体」）から AI へ輸入され、AI では実用的に**周囲を知覚し、意思決定し、行動する人工的実体**と定義される。技術系譜は 記号エージェント（論理規則）→ 反応エージェント（sense-act ループ）→ 強化学習エージェント（AlphaGo・DQN）→ 転移・メタ学習 と進んだが、いずれもタスク特化で「汎用の出発点となるモデル」を欠いていた。LLM がその空白を埋める——知識・指示理解・汎化・計画・推論を 1 つのモデルで備え、**自律性・反応性・能動性・社会的能力**というエージェントの 4 性質を（程度の差はあれ）示す最初の基盤である、というのが 2023 年サーベイの中心宣言であり、現在まで有効な出発点になっている。

単一エージェントの能力面の系譜は wiki の他原典が担う: 推論の創発（[[summaries/2022-chain-of-thought]]）→ 推論と行動の統合（[[summaries/2022-react]]）→ 失敗からの試行間学習（[[summaries/2023-reflexion]]）→ 記憶の階層管理（[[summaries/2023-memgpt]]）。

## 構成要素 — brain / perception / action

2023 年サーベイの 3 モジュール枠組みは、エージェントの解剖図として定着した。[[agent-loop]]（観測→思考→行動の反復）をモジュール側から見た分解でもある。

| モジュール | 役割 | 各論ページ |
| --- | --- | --- |
| **brain**（脳） | 制御中枢。自然言語相互作用・知識・記憶・推論と計画・転移と汎化 | [[reasoning-and-planning]]・[[self-reflection]]・[[agent-memory]]・[[context-engineering]] |
| **perception**（知覚） | テキスト・視覚・聴覚などの入力をモデルが扱える表現へ変換 | [[computer-use-agents]]（スクリーンショット観測）・[[transformer-architecture]]（マルチモーダル拡張） |
| **action**（行動） | テキスト出力・ツール使用・身体化された行動（embodied action） | [[tool-use-and-function-calling]]・[[coding-agents]]・[[computer-use-agents]] |

設計上の要点は「**どの能力をモデル内在で持ち、どれを外装で持つか**」の切り分けにある。2023 年時点は「brain の能力は所与、知覚と行動を外装で拡張」が前提だったが、その後 (a) ツール使用・並列協調を**訓練で作り込む**路線（[[summaries/2025-kimi-k2]]・[[summaries/2026-kimi-k2.5]]、RLVR による推論創発は [[summaries/2025-deepseek-r1]]）と、(b) 外装を**ハーネス**として体系設計・自動探索する路線（[[agent-frameworks]]・[[summaries/2026-meta-harness]]）が並走し、内外の境界はモデル世代ごとに動く変数になった。

## 応用の 3 形態

1. **単一エージェント** — タスク指向（Web・生活タスク）・**イノベーション指向（科学探究）→ [[research-agents]]**・ライフサイクル指向（オープンワールドでの生涯学習。Voyager のスキルライブラリが代表）。現在の主戦場は [[coding-agents]]（実行フィードバックが速く機械可読）と [[computer-use-agents]]（GUI 直接操作）、agentic search（[[retrieval-augmented-generation]]）。
2. **マルチエージェント** — 協調的相互作用（無秩序＝自由発言と多数決／秩序＝役割対・SOP）と敵対的相互作用（討論）。幻覚の相互増幅・誤った合意への収束という 2023 年時点の警告は、後の失敗分類（[[summaries/2025-masft]]）と本番運用の経済学（[[summaries/2025-multi-agent-research-system]]）で実証・条件づけられた → [[multi-agent-systems]]。
3. **人間–エージェント** — **instructor-executor パラダイム**（人間が指示とフィードバックを与える。現在の実務の大半）と **equal partnership パラダイム**（共感的コミュニケータ・対等な参加者）。人間関与の根拠は解釈可能性と制御可能性の保証 → HITL の実装は [[agent-safety-and-guardrails]]。

## エージェント社会と評価・安全

複数エージェント＋環境（テキスト／仮想サンドボックス／物理）で構成される**シミュレート社会**は open / persistent / situated / organized の 4 性質で特徴づけられ、Generative Agents（Stanford の 25 人町）が代表。社会科学の実験装置・合成データ生成・政策シミュレーションへの応用と、予期せぬ社会的害・ステレオタイプ増幅・プライバシー・擬人化への過度の依存というリスクが表裏をなす → [[multi-agent-systems]]。

評価は 2023 年サーベイが utility・sociability・values・継続進化の 4 観点を提示し、その後ベンチマーク群（SWE-bench・OSWorld 等）・トレース分析・LLM-as-a-judge へ具体化した → [[agent-evaluation]]。安全は「**行動空間を持つエージェントでは敵対的攻撃がテキストの誤りでなく破壊的行動になる**」という質的転換が出発点 → [[agent-safety-and-guardrails]]。

## 実務の設計原則

実務側の標準語彙は [[summaries/2024-building-effective-agents]]（workflow と agent の区別・「まず最も単純な解から」）が与え、設計パターンとフレームワーク・ハーネスの層は [[agent-frameworks]] が扱う。長時間・本番運用の実践知（構造化 artifact による引き継ぎ・generator/evaluator 分離・ハーネスの縮小と自動探索）は [[coding-agents]] と [[summaries/2025-effective-harnesses]] 以降の系譜を参照。

## 関連ページ

- [[research-agents]] — 調査・研究そのものを担うエージェント（知識の集約 ⇄ 知識の生成）

- [[agent-loop]] — 実行ループとしての見方（本ページの 3 モジュールと表裏）
- [[reasoning-and-planning]] / [[self-reflection]] / [[agent-memory]] / [[context-engineering]] — brain の各論
- [[tool-use-and-function-calling]] / [[coding-agents]] / [[computer-use-agents]] — action と応用の各論
- [[multi-agent-systems]] — 協調・敵対・エージェント社会
- [[agent-frameworks]] — 設計パターンとハーネス層
- [[agent-evaluation]] / [[agent-safety-and-guardrails]] / [[agent-observability]] — 評価・安全・運用
- [[summaries/2023-llm-agents-survey]] — 本ページの主要な根拠原典
