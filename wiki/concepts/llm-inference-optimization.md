---
type: concept
aliases: [推論最適化, inference optimization, KV cache, TTFT, TPS, serving, サービング, デコードスループット]
tags: [llm-inference-optimization, llm-foundations]
related:
  - "[[transformer-architecture]]"
  - "[[test-time-compute]]"
  - "[[context-engineering]]"
  - "[[multi-agent-systems]]"
summaries:
  - "[[summaries/2026-gpt2-to-kimi3]]"
  - "[[summaries/2025-multi-agent-research-system]]"
updated: 2026-07-28
---

# LLM Inference Optimization（LLM 推論の高速化・サービング）

LLM（Large Language Model, 大規模言語モデル）の**推論を速く・安く捌く**ための技術群。[[test-time-compute]] が「推論時に計算を積んで賢くする」側なら、本ページは「その計算を速く安く実行する」側であり、両者は対をなす。エージェントにとっては、**多ターン・長 trajectory・並列サブエージェント**のすべてが推論コストの掛け算になるため、この層の性質がエージェント設計の経済性（→ [[multi-agent-systems]] の「チャット比 15 倍」）を直接規定する。

## 基本概念

- **prefill / decode の 2 相**: 推論は、プロンプト全体を一括処理する **prefill**（並列性が高く計算律速）と、1 トークンずつ生成する **decode**（逐次的でメモリ帯域律速）に分かれる。
- **TTFT / TPS**: Time To First Token（最初のトークンまでの遅延。主に prefill が決める）と Tokens Per Second（生成スループット。主に decode が決める）が基本指標。
- **KV cache**: decode で過去トークンの key/value を再計算しないための保存領域。系列長に対して **O(N) で成長**し、長コンテキストでは GPU の HBM（High Bandwidth Memory）への読み書きがボトルネックになる——「LLM の長文が高い・遅い」ことの物理的な正体（[[summaries/2026-gpt2-to-kimi3]] が実装レベルで解説）。

## 高速化のアプローチ

### 状態を固定サイズに畳む — アーキテクチャ側の解

linear attention 系（→ [[transformer-architecture]] の系譜）は、KV cache の O(N) 成長そのものを、固定 D×D 状態への畳み込みで解消する。デコードコストは系列長によらず O(1) になり、Kimi Linear は full attention 比**最大 6 倍のデコードスループット**を主張する（自己申告の管理下比較）。代償は完全検索の放棄で、実運用のモデル（Kimi K3）は固定状態層と完全 attention 層のハイブリッドでバランスを取る。

### チャンク化とハードウェア整合 — FLOPs ≠ wall-clock

DeltaNet 系のチャンク並列化では、チャンクサイズ **C=1 が FLOPs 最小だが、実効速度では C=64/128 が速い**——GPU のテンソルコア（行列積ユニット）に仕事が乗る粒度だからである（[[summaries/2026-gpt2-to-kimi3]]）。「演算数の削減」と「wall-clock の短縮」は別の目的関数であり、ハードウェアへの写像を考えない FLOPs 比較は実務では当てにならない。

### カーネル融合 — 実装成熟度が律速する

Kimi K3 の SiTU 活性化は、**融合カーネル（fused kernel, 複数の演算を 1 つの GPU カーネルにまとめてメモリ往復を省く実装）なしでは元のパスの約 3 倍遅い**。新しいアーキテクチャ要素の速度は、数学ではなくカーネル実装の成熟度で決まることが多い——FlashAttention（softmax attention を N×N 行列を実体化せずに計算するカーネル。概説）が softmax attention の寿命を延ばしたのも同じ構図である。

### バッチングほか — サービング側の解

複数リクエストをまとめて GPU 稼働率を上げる continuous batching、KV cache のページ管理（vLLM 系）、量子化・投機的デコーディングなどのサービング技術群（専用原典は未 ingest のため概説）。エージェント文脈では、[[summaries/2025-multi-agent-research-system]] の並列サブエージェント・並列ツール呼び出し（調査時間 −90%）が、この層のスループットを前提にした設計である。

## エージェント設計への含意

- **コスト構造を物理から理解する**: 長いコンテキストの持ち回りは KV cache の帯域コスト、多ターンの再 prefill はキャッシュヒットの問題。[[context-engineering]] の「参照渡し」「フェーズ要約」は、この物理コストへの応答でもある。
- **レイテンシ予算の配分**: エージェントの体感速度は TTFT（計画・最初の応答）と TPS（長い生成）の両方で決まり、並列化（サブエージェント・並列ツール呼び出し）はスループットで遅延を買う手段。
- **モデル選定の軸**: 固定状態ハイブリッドの普及で「長コンテキストのデコード単価」がモデル間で大きく異なる時代になりつつある。トークン単価だけでなく、長系列でのスループット特性を見る。

## 関連ページ

- [[transformer-architecture]] — コスト構造を決めるアーキテクチャの側
- [[test-time-compute]] — 推論時計算を「賢さ」に使う側（本ページはそれを安く捌く側）
- [[context-engineering]] — コンテキスト積載のコスト意識
- [[multi-agent-systems]] — 並列化とトークン経済
- [[summaries/2026-gpt2-to-kimi3]] — 本ページの主要な根拠原典
