---
type: concept
aliases: [tool use, function calling, ツール利用, ツール呼び出し, Toolformer]
tags: [tool-use-and-function-calling, llm-agents]
related:
  - "[[agent-loop]]"
  - "[[reasoning-and-planning]]"
  - "[[model-context-protocol]]"
  - "[[retrieval-augmented-generation]]"
summaries:
  - "[[summaries/2022-react]]"
updated: 2026-07-24
---

# Tool Use and Function Calling（ツール利用とツール呼び出し）

LLM（Large Language Model, 大規模言語モデル）が**外部のツール——検索 API、計算機、コード実行環境、ブラウザなど——を呼び出して、自分の知識と能力の限界を補う**仕組み。モデル単体では最新情報を知らず、正確な計算が苦手で、外部世界に作用できない。ツール利用はこの 3 つの限界を一度に破る手段であり、[[agent-loop]] における「行動（action）」の具体的な中身にあたる。

歴史的には、(1) プロンプトでテキスト形式の行動を出力させる時代（ReAct 等）、(2) モデル自身にツール呼び出しを学習させる時代（Toolformer 等）、(3) API レベルで構造化された function calling が標準機能になる時代、と発展してきた。

## 代表手法

### ReAct の Wikipedia API — プロンプト時代のツール定義

[[summaries/2022-react]]（2022）は、ツールを **`search[entity]` / `lookup[string]` / `finish[answer]` の 3 つだけ**に絞った Wikipedia API を設計し、few-shot プロンプトの例示だけでモデルにその使い方を教えた。特筆すべきは、このツールが**意図的に弱く**作られていることである。検索は正確なページ名でしか引けず（最先端の検索器よりはるかに弱い）、それゆえモデルは「見つからなかったから類似候補の film の方を検索しよう」のように**言語での推論によって検索を導く**ことを強制される。ツールの弱さが推論を引き出すというこの設計は、ツールとエージェントを一体で設計する後の agent-computer interface の考え方（SWE-agent 等 → [[coding-agents]]）の源流にある。

得られた教訓も具体的である: 検索が有益な情報を返さないことがエラーの 23% を占め、悪い検索結果は推論全体を脱線させる。**ツールの出力の質がエージェントの上限を決める**。

### Toolformer ほか — ツール呼び出しの学習

モデル自身に「どこでどのツールを呼ぶべきか」を自己教師的に学習させる方向（Schick et al., 2023。原典未 ingest のため概説のみ）。プロンプトによる教示から、モデルの重みへのツール利用能力の埋め込みへ、という流れを作った。

### Function calling — 構造化されたツール定義

現在の主流は、**ツールを JSON スキーマ（名前・説明・引数の型）で宣言し、モデルが構造化された引数付きの呼び出しを生成する** API レベルの機構である（OpenAI function calling、Anthropic tool use 等。原典未 ingest のため概説のみ）。ReAct の `search[entity]` のような文字列規約と比べ、引数の型検証・複数ツールの並列呼び出し・呼び出し失敗のハンドリングが体系化された。ツールとデータソースの接続を標準化するプロトコルとしては [[model-context-protocol]]（MCP）が登場している。

## 設計上の論点

- **ツールの粒度と数**: ReAct は 3 ツールで足りたが、ツールが増えるほどモデルの選択誤りも増える。ツール説明文の書き方は一種のプロンプト設計である。
- **検索ツールとの関係**: 「検索して結果を根拠に答える」というツール利用の最頻出パターンは、[[retrieval-augmented-generation]]（RAG）をエージェントのループに埋め込んだものと見なせる。ReAct の「内部知識で足りるか、検索すべきか」の切り替えはその原型。
- **安全性**: ツールは外部世界への作用を意味するので、危険な行動を行動空間に含めない設計（ReAct は閲覧のみで Wikipedia を編集できない）、権限の最小化、実行前の人間の承認が基本になる → [[agent-safety-and-guardrails]]。

## 関連ページ

- [[agent-loop]] — ツール呼び出しが埋め込まれる実行ループ
- [[reasoning-and-planning]] — どのツールをいつ呼ぶかを決める推論
- [[model-context-protocol]] — ツール接続の標準プロトコル（未作成）
- [[retrieval-augmented-generation]] — 検索ツール利用の体系化（未作成）
- [[summaries/2022-react]] — 本ページの主要な根拠原典
