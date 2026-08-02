---
type: concept
aliases: [自己反省, 自己内省, verbal reinforcement, Reflexion, Self-Refine, self-correction]
tags: [self-reflection, llm-agents, reasoning-and-planning]
related:
  - "[[llm-red-teaming]]"
  - "[[reasoning-and-planning]]"
  - "[[agent-loop]]"
  - "[[agent-memory]]"
  - "[[agent-evaluation]]"
  - "[[test-time-compute]]"
summaries:
  - "[[summaries/2023-reflexion]]"
  - "[[summaries/2025-deepseek-r1]]"
  - "[[summaries/2025-long-cot-survey]]"
  - "[[summaries/2026-harness-design]]"
updated: 2026-07-31
---

# Self-Reflection（自己反省）

LLM（Large Language Model, 大規模言語モデル）エージェントが**自分の失敗を言語で振り返り、その反省を次の試行に活かす**仕組みの総称。重み更新を伴わない、in-context（文脈内）での方策改善であり、「エージェントは一発勝負ではなく試行錯誤で学べるか」という問いへの最初の実用的な答えである。単発の出力を磨く self-correction（自己修正）から、試行をまたいで記憶を蓄える verbal reinforcement（言語的強化）までがこの概念に含まれる。

前提となる観察は 2 つ。(1) 単なるリトライは学習ではない——[[summaries/2023-reflexion]] の実測では、ReAct も CoT も temperature を上げて再試行するだけでは初回に失敗したタスクを 1 問も解けなかった。(2) スカラー報酬は貧しすぎる——「失敗した」という信号だけでは、長い trajectory（軌跡）のどこが悪かったのか（信用割当問題）が伝わらない。自己反省は、この貧しい信号を**LLM 自身に因果分析つきの文章へ増幅させる**ことで両方を解く。

## 代表手法

### Reflexion — 試行間の言語的強化

[[summaries/2023-reflexion]]（NeurIPS 2023）が確立した標準形。**方策を「LLM ＋ 記憶」と再定義**し、学習で更新されるのはテキストの記憶だけ、という設計。

- 構成は 3 役: **Actor**（ReAct/CoT 等の既存エージェントをそのまま使う）＋ **Evaluator**（完全一致・ヒューリスティクス・LLM 判定・自己生成単体テスト）＋ **Self-Reflection モデル**（「どこで・なぜ失敗し・次はどうするか」を一人称で生成）。
- 記憶は二層: 軌跡＝短期記憶、反省文＝長期記憶（上限 1〜3 件のスライディングウィンドウ）→ [[agent-memory]]。
- 成果: AlfWorld 130/134（ReAct 単体は幻覚率 22% で頭打ち）、HumanEval pass@1 91%（当時の GPT-4 単体 80%）。既存エージェントに**後付けできるラッパー**である点が実装上の美点。

### Self-Refine — 単発生成の自己改善

生成→自己フィードバック→改稿を繰り返す反復枠組み（Madaan et al., 2023。原典未 ingest のため概説）。Reflexion との違いは原典の比較表が端的で、**反復はするが試行をまたぐ記憶を持たず**、意思決定タスクや二値報酬を扱わない。「その場で磨く」Self-Refine と「失敗を覚えて出直す」Reflexion、という対比で覚えるとよい。

### 内生する反省 — RL からの創発

Reflexion が反省を**プロンプト工学で外付け**したのに対し、[[summaries/2025-deepseek-r1]]（2025）は、検証可能な報酬だけの大規模 RL の中で **reflection・自己検証・手順の再評価が誰にも教えられずに創発する**ことを示した（「aha moment」——中間チェックポイントが "Wait, wait. Wait." と書いて自分の数式変形を再評価し始めた記録）。テンプレートは「考えてから答えよ」という構造しか指定しておらず、内省の義務づけは一切ない。つまり自己反省には、(a) プロンプトとループで外付けする経路（Reflexion）と、(b) 報酬から内生させる経路（R1-Zero）の 2 つがあり、後者の登場で「反省できるモデル」自体が製造可能になった → [[reinforcement-learning-from-human-feedback]]。

### Long CoT への統合 — 「feasible reflection」

Long CoT サーベイ（[[summaries/2025-long-cot-survey]], 2025）は、自己反省を推論モデルの定義の一部に組み込んだ: Long CoT とは deep reasoning・extensive exploration・**feasible reflection**（誤り箇所へ戻るフィードバックと、修正する refinement）の 3 特性を**単一の生成に統合**したものであり、反省を欠く ToT は Long CoT ではない、という線引きである。Reflexion が試行の**外側**のループで行った反省は、推論モデルでは 1 本の思考の**内側**に畳み込まれた——外付け（Reflexion）→ 内生（R1-Zero）→ 定義への統合（Long CoT）という 3 段の系譜になる。同サーベイは refinement の手法群も 3 分類で整理する（プロンプト型＝Self-Refine 系・SFT 模倣型・RL 学習型。SFT だけでは自己洗練は促進されず、分布ミスマッチのため RL が要るという SCoRe の知見を含む）。

重要な留保も同サーベイが集約している: **aha moment（反省の創発）には反証がある**。自己反省のパターン自体はベースモデル（epoch 0）の段階から存在し（superficial self-reflection）、それは必ずしも正答につながらず、R1-Zero 訓練での応答長の増加は反省の深化でなく報酬最適化の帰結だとする分析である。「RL が反省を生んだ」ではなく「RL が既存の反省パターンを有効な形で引き出した」という読みが安全になりつつある。

### 検証つき反省 — 何を評価信号にするか

反省の質は評価信号の質で決まる。Reflexion の選択肢は (a) 環境の二値報酬、(b) 手書きヒューリスティクス（**同一行動×3 回 or 30 行動超**というループ検出だけで AlfWorld の空回りをほぼ捕捉——安価で強い）、(c) LLM による判定、(d) **自己生成単体テスト**（コード。もっとも接地された自己評価）→ [[agent-evaluation]]。

## 設計論点と限界

- **盲目的リトライは効かない**。Reflexion のアブレーションでは、反省の言語化を省くとエラーが見えても修正に翻訳されず改善ゼロ、テスト（終了判定）を省くとベースライン以下に劣化。**「誤りの特定」と「修正」の橋渡しには言語化が要る**。
- **偽陽性は即死、偽陰性は回復可能**。自己テストが誤って合格を出すと誤答を確信して提出してしまう。テストは厳しすぎる方向に倒すのが安全という非対称性。
- **創発能力である**。starchat-beta では効果が完全にゼロで、GPT-4 ≫ GPT-3.5 の順に効く。小さいモデルに反省を書かせても学習にならない——CoT の創発性（[[summaries/2022-chain-of-thought]]）と同型。
- **探索は生めない**。反省は「同じ山をより上手に登る」ことしかできず、局所解からの脱出（WebShop での失敗が実例）には別の仕掛け——多様なサンプリング、[[multi-agent-systems]] の独立試行、[[reasoning-and-planning]] の木探索——が要る。
- 内在的な自己修正（外部信号なしで自分の答えを直す）の効果は後続研究で論争があり、**外部の実行結果・報酬と組み合わせた場合**の有効性として理解するのが安全。実務側の証言が Anthropic のハーネス実験（[[summaries/2026-harness-design]]）にある: 自分の仕事の評価を求めると、エージェントは品質が明らかに凡庸でも**自信を持って称賛**する。そして「**generator を自己批判的にするより、独立した evaluator を懐疑的にチューニングする方がはるかに扱いやすい**」——自己反省の甘さは、同じモデルでも**役割を分離した別インスタンス**に懐疑を担わせることで回避できるという、reflection の限界への現在の実務解である。
- 反省の蓄積は [[test-time-compute]]（推論時計算を積んで精度を買う）の一形態でもあり、試行回数×トークンのコスト増と引き換えである。

## 関連ページ

- [[reasoning-and-planning]] — 推論の系譜（CoT → ReAct → Reflexion）の第 3 幕
- [[agent-loop]] — episode 内ループの外側の「試行間ループ」
- [[agent-memory]] — 反省文の保存・取捨選択が次の課題
- [[agent-evaluation]] — 評価信号の設計・FP/FN 非対称
- [[summaries/2023-reflexion]] — 本ページの主要な根拠原典
- [[llm-red-teaming]] — 同じ構造が攻撃側にも現れる。[[summaries/2023-pair]] のアブレーションでは、攻撃例を消しても 72%→70% しか落ちないのに、**「前回なぜ失敗したか」を書かせる欄を消すと 72%→56%** に落ちた——探索を駆動しているのは例示でなく自己反省である
