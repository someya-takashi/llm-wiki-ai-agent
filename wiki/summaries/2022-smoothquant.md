---
type: summary
source_path: raw/papers/SmoothQuant_ Accurate and Efficient Post-Training Quantization for Large Language Models.md
source_kind: paper
title: "SmoothQuant: Accurate and Efficient Post-Training Quantization for Large Language Models"
authors: [Guangxuan Xiao, Ji Lin, Mickael Seznec, Hao Wu, Julien Demouth, Song Han]
year: 2022
venue: ICML 2023 (arXiv:2211.10438)
ingested: 2026-08-02
tags: [model-quantization, llm-inference-optimization, w8a8, activation-outliers, migration-strength]
translation: "[[translations/2022-smoothquant]]"
---

# SmoothQuant: 大規模言語モデルのための正確かつ効率的な訓練後量子化

> 原典: [[translations/2022-smoothquant]] ・ `raw/papers/SmoothQuant_ Accurate and Efficient Post-Training Quantization for Large Language Models.md`
> 著者: Guangxuan Xiao, Ji Lin, Mickael Seznec, Hao Wu, Julien Demouth, Song Han
> 年・会議: 2022-11・ICML 2023（arXiv:2211.10438）

## 一言まとめ

**外れ値を分離するのでなく、活性から重みへ「移す」。** 数学的に等価な per-channel のスケーリングで活性の外れ値をならし、重みと活性の両方を INT8 にする（W8A8）。[[summaries/2022-llm-int8]] が精度は保つが**FP16 より遅くなる**という弱点を、正面から突いた仕事である。

## 背景と問題意識

**量子化には 2 つの目的がある。** 重みだけを INT8 にすればメモリは半減するが、**推論を速くするには重みと活性の両方を INT8 にして整数カーネル（INT8 GEMM）を使わなければならない**。これが W8A8 である。

ところが LLM の活性は量子化が難しい。6.7B を超えると体系的な外れ値が創発するからである（[[summaries/2022-llm-int8]] の発見）。既存手法の状況は次のようだった。

- **ZeroQuant** — per-token の動的活性量子化と group-wise の重み量子化。効率的だが **OPT-175B の精度を保てない**。
- **LLM.int8()** — 混合精度分解で外れ値を FP16 に残す。精度は保つが、**その分解をハードウェアアクセラレータ上で効率的に実装するのが難しい**。

本論文はこの 2 番目を出発点にする。「**効率的で、ハードウェアに優しく、できれば訓練不要な方式**」が未解決だ、というのが問題設定である。

### なぜ活性が難しいのか——3 つの観察

著者らは OPT-13B の線形層を可視化して 3 点を挙げる。これが手法の設計をそのまま導く。

1. **重みは平坦で一様、活性は違う。** 重みは INT8 どころか INT4 でも精度が落ちないことが先行研究で分かっている。
2. **外れ値は他の値の約 100 倍。** per-tensor 量子化では外れ値が最大絶対値の測定を支配し、**外れ値でないチャネルには実効的に 2〜3 段階しか量子化水準が残らない**。チャネル $i$ の実効水準は $2^8 \cdot m_i/m$ と定量化される。
3. **外れ値は固定チャネルに持続的に現れる。** トークン間のチャネルの分散は大きいが、**あるチャネルのトークンをまたいだ分散は小さい**。どのチャネルが外れ値かは安定している。

### そして最も重要な「できない」の証明

3 番目の観察からすれば、**per-channel の活性量子化をすればよい**。実際 Table 1 がそれを裏づける。

| OPT- | 6.7B | 13B | 30B | 66B | 175B |
|---|---|---|---|---|---|
| FP16 | 64.9% | 65.6% | 67.9% | 69.5% | 71.6% |
| INT8 per-tensor | 39.9% | 33.0% | 32.8% | 33.1% | 32.3% |
| INT8 per-token | 42.5% | 33.0% | 33.1% | 32.9% | 31.7% |
| **INT8 per-channel** | **64.8%** | **65.6%** | **68.0%** | **69.4%** | **71.4%** |

**per-channel なら FP16 とほぼ同一**。per-token はほとんど役に立たない（per-tensor よりわずかに良いだけ）。

**しかし per-channel は使えない。** INT8 の GEMM カーネルは Tensor Core の高スループットな演算列に依存しており、そこへ低スループットの命令（変換や CUDA Core の FMA）を挟むことを許さない。**スケーリングは行列積の「外側の」次元——活性のトークン次元 $T$、重みの出力チャネル次元 $C_o$——にしか置けず、行列積が終わった後にしか適用できない**。活性の入力チャネル $C_i$ は内側の次元なので、原理的に届かない。

**これは wiki 内で繰り返し現れる主題の、量子化版である**——ハードウェアの構造がアルゴリズムの選択肢を切り落とす。[[summaries/2024-flashattention-3]] で FP8 WGMMA が k-major しか受け付けないためにカーネル内転置が要る、という話と同型である。

## 提案手法

per-channel が使えないなら、**難しさそのものを動かす**。活性を per-channel の平滑化係数 $\mathbf{s}$ で割り、重みを逆方向に掛ける:

$$\mathbf{Y}=(\mathbf{X}\,\text{diag}(\mathbf{s})^{-1})\cdot(\text{diag}(\mathbf{s})\mathbf{W})=\hat{\mathbf{X}}\hat{\mathbf{W}}$$

**これは数学的に等価な変換である**（出力は変わらない）。しかも $\mathbf{X}$ は通常それ以前の線形層や layer norm から来るので、**平滑化係数を先行層のパラメータへオフラインで融合できる**。実行時のカーネル呼び出しのオーバーヘッドはゼロになる。

<figure>

![](../../raw/assets/2022-smoothquant/x2.png)

<figcaption>図2（再掲）: SmoothQuant の直観。外れ値が量子化の範囲を引き伸ばして大半の値に有効なビットを残さない状態から、スケールの分散を活性 X から重み W へ移すことで、両方を量子化しやすくする。</figcaption>
</figure>

### migration strength $\alpha$ — 難しさをどれだけ移すか

素朴には $\mathbf{s}_j = \max(|\mathbf{X}_j|)$ とすれば全チャネルの最大値が揃うが、これは**難しさをすべて重みへ押しつけてしまう**。そこでハイパーパラメータ **migration strength（移行強度）** $\alpha$ を導入する:

$$\mathbf{s}_{j}=\max(|\mathbf{X}_{j}|)^{\alpha}/\max(|\mathbf{W}_{j}|)^{1-\alpha}$$

$\alpha = 0.5$ が OPT と BLOOM の適所で、対応するチャネルの重みと活性が似た最大値を共有する。**活性の外れ値がより激しいモデルには大きな $\alpha$**（GLM-130B は約 30% が外れ値なので 0.75、LLaMA は 0.8）。アブレーション（Figure 10）では **0.4〜0.6 が適所**で、小さすぎると活性が、大きすぎると重みが量子化しにくくなる。

平滑化係数は事前学習データ（Pile）からの **512 文で一度だけ較正**し、以後すべての下流タスクへ同じモデルを適用する。

### 効率の 3 水準

SmoothQuant は量子化方式に直交なので、O1 → O3 と段階的に効率を上げられる。

| | Weight | Activation |
|---|---|---|
| SmoothQuant-O1 | per-tensor | per-token 動的 |
| SmoothQuant-O2 | per-tensor | per-tensor 動的 |
| **SmoothQuant-O3** | per-tensor | **per-tensor 静的** |

粗いほど速い。静的量子化は実行時にステップサイズを計算しないぶん大きく加速する。

## 実験結果と知見

### 精度

OPT-175B（7 つの zero-shot ベンチマークの平均と WikiText パープレキシティ）:

| | Average ↑ | WikiText ↓ |
|---|---|---|
| FP16 | 66.9% | 10.99 |
| W8A8（素朴） | 35.5% | **93080** |
| ZeroQuant | 35.8% | **84648** |
| Outlier Suppression | 36.0% | **96151** |
| LLM.int8() | 66.7% | 11.10 |
| **SmoothQuant-O3** | **66.8%** | 11.17 |

素朴な W8A8 は**パープレキシティが 4 桁**に飛ぶ（完全な破壊）。精度を保つのは LLM.int8() と SmoothQuant だけである。OPT-175B・BLOOM-176B・GLM-130B の 3 つ、さらに **MT-NLG 530B**、指示チューニング済みの OPT-IML-30B、LLaMA 7B〜65B でも成立する。

### そして速度——本論文の主張の核心

| OPT-13B, seq 256 | レイテンシ |
|---|---|
| FP16 | 152.6 ms |
| **LLM.int8()** | **237.1 ms** |
| SmoothQuant-O1 | 124.5 ms |
| SmoothQuant-O2 | 120.5 ms |
| **SmoothQuant-O3** | **112.1 ms** |

**LLM.int8() は FP16 より 1.55 倍遅い。SmoothQuant-O3 は 1.36 倍速い。** 論文は「LLM.int8() はほとんど常に FP16 のベースラインより遅く、これは混合精度の活性表現の大きなオーバーヘッドによる」と明記する。

PyTorch で最大 **1.51 倍**、FasterTransformer で最大 **1.56 倍**、メモリは約 **1.96 倍**削減。最も実務的に効くのは **GPU 台数を半分にできる**ことで、OPT-66B が 2 台→1 台、OPT-175B が 8 台→4 台、そして **MT-NLG 530B が 16 台→8 台**（＝単一ノードに収まる）。デコード段でも最大 1.42 倍。

## 限界・批判的視点

1. **$\alpha$ はモデルごとにグリッド探索が要る。** 0.5 が「一般的な適所」とされるが、GLM-130B は 0.75、LLaMA は 0.8 と実際にはばらつく。手法は訓練不要だが**完全にハイパーパラメータフリーではない**。適所の幅（0.4〜0.6）も狭い。
2. **O3（per-tensor 静的）は BLOOM で 0.8% 落ちる。** 著者らはこれを「静的に収集した統計と実際の評価サンプルの活性統計の差異」に帰しており、**較正データの分布依存性**という一般的な弱点がそのまま出ている。
3. **W8A8 で止まっている。** 付録 A が認めるとおり、重みのみの量子化（GPTQ）とは設定が異なり**直接比較していない**。実装の違い（GPTQ の低ビットカーネルはバッチサイズ 1 の生成段のみ対応、しかも OPT-175B 専用に最適化）を理由に比較を避けている——公正な態度だが、**読者は「どちらが良いか」の答えを得られない**。
4. **バッチングが標準になるという前提に依存する。** 著者らは「GPTQ はメモリ律速の小バッチで有利、SmoothQuant はバッチ設定やコンテキスト段で有利。本番ではバッチングが将来の標準になると信じる」と述べる。**これは予測であって証拠ではない。**
5. **LLM.int8() への批判は速度に限られる。** 精度では両者とも FP16 に一致しており、SmoothQuant の優位は純粋に実装効率にある。LLM.int8() 側も自ら「6.7B 未満では遅くなりうる」と認めていたので、**争点は「どれだけ遅いか」であって「遅いかどうか」ではない**。
6. **LLaMA では問題自体が小さい。** 著者らは「LLaMA は OPT や BLOOM と比べて活性の外れ値の問題が概して深刻でない」と述べている。**外れ値の深刻さはアーキテクチャと訓練に依存する**ので、手法の必要性もモデルによって変わる。
7. **2022〜2023 年の実装環境**（FasterTransformer、CUTLASS INT8 GEMM）に依存する数値である。

## 実装・運用上の示唆

- **「精度を保つ」と「速くする」を分けて評価する。** 本論文の最大の教訓はこれである。LLM.int8() と SmoothQuant は**精度では同等**（OPT-175B で 66.7% vs 66.8%）だが、**速度では正反対**（0.64x vs 1.36x）。量子化手法の比較表を読むときは、精度の列だけを見てはいけない。
- **ハードウェアの制約から逆算して設計する。** per-channel が精度的には正解でも INT8 GEMM に載らないなら、載る形（per-tensor）に問題を変換するほうが実用になる。**「理想的な手法を諦めて、等価変換で問題を動かす」**という発想は移植性が高い。
- **等価変換で数値的性質だけを変える手がある。** 出力を変えずに量子化しやすさだけを改善するという点で、これは [[summaries/2024-flashattention-3]] の incoherent processing（直交回転で外れ値をならす）と同じ系統に属する（→ [[model-quantization]] の外れ値対処の 4 系統のうち「分布そのものをならす」）。
- **較正は一度でよいが、分布は合わせる。** 512 文で一度較正して全下流タスクに使えるのは実務上大きい。ただし O3 の 0.8% の劣化が示すとおり、**較正データと本番の活性分布のずれはそのまま精度に出る**。
- **エージェント実務への含意。** 付録 A の 3 番目の論点が重要である——**チャットボットのような長コンテキスト・バッチ運用では KV cache が支配的になる**（バッチ 512・コンテキスト 2048 で 3TB＝モデル重みの 3 倍）。活性の量子化は KV cache の削減にも効くので、多ターンのエージェントでは重みのみの量子化より W8A8 のほうが効く場面がある。

## 用語と略称

- **W8A8** = 重み 8 ビット・活性 8 ビットの量子化。整数 GEMM カーネルを使うために必要
- **PTQ** = Post-Training Quantization（訓練後量子化。再訓練しない）
- **per-tensor / per-token / per-channel** = テンソル全体／トークンごと／チャネルごとにスケール係数を持つ粒度
- **group-wise 量子化** = per-channel の粗い版。チャネル群ごとにスケールを持つ
- **静的 / 動的量子化** = 較正データで事前にスケールを決める／実行時に決める
- **migration strength（移行強度）$\alpha$** = 量子化の難しさを活性から重みへどれだけ移すかを制御するハイパーパラメータ
- **実効的な量子化水準（effective quantization levels）** = あるチャネルが実際に使える量子化の段階数。$2^8 \cdot m_i/m$
- **GEMM / BMM** = General Matrix Multiply（汎用行列積）／Batched Matrix Multiply（attention 内の行列積）
- **FasterTransformer** = NVIDIA の本番向けトランスフォーマー提供フレームワーク
- **CUTLASS** = NVIDIA の GEMM カーネルテンプレートライブラリ
- **Outlier Suppression** = 非スケーリング LayerNorm とトークンごとクリッピングで外れ値を扱う先行手法。BERT では成功するが LLM では失敗する
- **MT-NLG 530B** = Microsoft/NVIDIA の 5300 億パラメータモデル

## 関連ページ

- [[model-quantization]] — 本論文が主要な根拠となる概念ページ。外れ値対処の 4 系統のうち「分布そのものをならす」に属する
- [[summaries/2022-llm-int8]] — 本論文が名指しで批判する先行研究。外れ値の創発を発見した側
- [[summaries/2023-qlora]] — 同時期のもう一つの答え。こちらは推論でなくファインチューニングを狙う
- [[llm-inference-optimization]] — ハードウェアの構造がアルゴリズムを縛るという主題
- [[summaries/2024-flashattention-3]] — 等価変換で数値的性質だけを変える incoherent processing、および FP8 のレイアウト制約という同型の問題
- [[summaries/2025-llm-quantization-explained]] — 本論文の外れ値分析を二次資料として紹介している記事
- [[context-engineering]] — 付録 A の「長コンテキスト・バッチ運用では KV cache が支配的」という論点の接続先
