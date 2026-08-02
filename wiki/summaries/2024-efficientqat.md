---
type: summary
source_path: raw/papers/EfficientQAT_ Efficient Quantization-Aware Training for Large Language Models.md
source_kind: paper
title: "EfficientQAT: Efficient Quantization-Aware Training for Large Language Models"
authors: [Mengzhao Chen, Wenqi Shao, Peng Xu, Jiahao Wang, Peng Gao, Kaipeng Zhang, Yu Qiao, Ping Luo]
year: 2024
venue: arXiv:2407.11062
ingested: 2026-08-02
tags: [model-quantization, parameter-efficient-fine-tuning, qat, efficientqat, llm-inference-optimization]
translation: "[[translations/2024-efficientqat]]"
---

# EfficientQAT: 効率的な量子化を意識した訓練

> 原典: [[translations/2024-efficientqat]] ・ `raw/papers/EfficientQAT_ Efficient Quantization-Aware Training for Large Language Models.md`
> 著者: Mengzhao Chen, Wenqi Shao, Peng Xu, Jiahao Wang, Peng Gao, Kaipeng Zhang, Yu Qiao, Ping Luo（OpenGVLab, Shanghai AI Laboratory / 香港大学）
> 2024 年（arXiv:2407.11062）

## 一言まとめ

**QAT（Quantization-Aware Training, 量子化を意識した訓練）は「精度は最良だがコストが法外」というのが 2024 年までの常識だったが、それを「Llama-2-70B の 2 ビット化を A100 1 枚・41 時間・訓練メモリ 34.2GB」まで下げた論文**である。訓練を「ブロックごとに全パラメータを訓練する段（Block-AP）」と「量子化パラメータだけを端から端まで訓練する段（E2E-QP）」に分けたことが鍵で、結果として PTQ・QAT・Q-PEFT の 3 系統すべてを同じ土俵で上回る。

## 背景と問題意識

LLM を低ビット化する手法は 3 系統あり、それぞれに固有の欠陥があった。

| 系統 | 代表 | 欠陥 |
|---|---|---|
| **PTQ**（訓練後量子化） | [[summaries/2022-gptq]], [[summaries/2023-awq]], OmniQuant, AutoRound | 低ビット（2〜3 ビット）で精度が落ちる |
| **QAT** | BitNet b1.58, LLM-QAT | 事前学習データ全体での再訓練が要る。BitNet b1.58 は**3B モデル・1,000 億トークンでしか検証していない** |
| **Q-PEFT** | [[summaries/2023-qlora]], QA-LoRA, PEQA | **ファインチューニング後に LoRA を統合するとモデルが FP16 へ戻る** |

3 番目の指摘が本論文の批判のうち最も鋭い。**QLoRA 系は「4 ビットで訓練できる」が「4 ビットで配れる」わけではない**。アダプタを量子化重みへマージすると FP16 に戻るので、メモリの限られたプラットフォームへ出すにはもう一度 PTQ をかける必要があり、そこで性能が落ちる。この問題に取り組んだ PEQA（RTN で量子化してステップサイズだけをファインチューニング）が最も近い先行研究だが、**PTQ 初期化が貧弱**なため QLoRA や QA-LoRA に見劣りしていた。

つまり、**低ビットのまま訓練でき、低ビットのまま出荷でき、かつ現実的なコストで回るもの**が空白だった。

## 提案手法 — 2 段階に分ける

素朴な QAT は完全精度の重み $\mathbf{W}$ と量子化パラメータ $s$（ステップサイズ）・$z$（ゼロ点）を**端から端まで同時に**訓練するので、メモリが跳ね上がる。EfficientQAT はこれを 2 段に割る。

量子化の定義自体は標準的な一様量子化である:

$$\mathbf{W}_{int}=\mathrm{clamp}\left(\left\lfloor\frac{\mathbf{W}}{s}\right\rceil+z,\,0,\,2^{N}-1\right),\qquad \widehat{\mathbf{W}}=(\mathbf{W}_{int}-z)\cdot s$$

丸めが微分不可能なので **STE**（Straight-Through Estimator, 直通推定器）で勾配を通す（付録 C に $s$・$z$・$\mathbf{W}$ それぞれの勾配式がある）。

### 第 1 段: Block-AP（全パラメータのブロック単位訓練）

**トランスフォーマブロック 1 つずつ**、ブロック単位の再構成損失で、**$s$・$z$・$\mathbf{W}$ のすべて**を訓練する。BRECQ の知見——事前学習済みモデルが与えられていれば、ブロック単位訓練のほうが端から端までより速く収束し、必要な時間・データ・メモリが少ない——に乗っている。GPTQ が量子化をブロック単位でストリーミングしたのと同じ構造を、**訓練**に持ち込んだ形である。

ここで反直感的なのが**「全部訓練する」が最良だった**という結果である。従来手法は過適合を防ぐために訓練対象を絞ってきた（丸めパラメータのみ、クリッピング閾値のみ、ステップサイズのみ）が、Table 6 が示すのは:

| 訓練対象 | パラメータ数 | メモリ | Avg. PPL | Avg. Acc |
|---|---|---|---|---|
| clipping | 6.3M | 6.4GB | 11.28 | 53.20 |
| $s$, $z$ | 6.3M | 6.4GB | 10.26 | 55.20 |
| round | 202.4M | 8.6GB | 15.50 | 45.32 |
| $s$, $z$, round | 208.7M | 9.3GB | 9.17 | 57.14 |
| **$s$, $z$, $\mathbf{W}$** | 208.7M | **8.5GB** | **8.65** | **58.94** |

丸めパラメータを訓練する手法は整数重みの更新範囲を $(-1,+1)$ に制約する正則化として働くが、**それが解空間を狭めて最終性能を頭打ちにしている**。EfficientQAT は制約を外し、代わりに**訓練サンプルを 128 → 4096 に増やすだけで過適合に対処する**（訓練損失と検証損失の差が 1.07 → 0.06、平均正解率 57.14% → 58.99%）。しかも $\mathbf{W}$ を直接訓練するほうが**メモリは小さい**——丸め操作の訓練は丸めパラメータの複製をもう 1 つ要るからである。

### 第 2 段: E2E-QP（量子化パラメータの端から端までの訓練）

Block-AP で得た量子化重み $\mathbf{W}_q$ を**固定**し、**ステップサイズ $s$ だけ**を目標データセット上で端から端まで訓練する。ここには式 (1) の量子化はなく、式 (2) の逆量子化だけがあるので、勾配は $\partial\widehat{w}/\partial s = w_q - z$ という単純な形になる。

訓練対象はグループサイズ 64 で**全体の約 1.5%** しかない。$s$・$z$・両方のいずれを訓練しても性能はほぼ同じだが（Table 7）、$z$ の訓練は低ビット表現を FP16 へ変換するので平均ビット数が増える。ゆえに既定は $s$ のみ。

**この段が Q-PEFT の役割も兼ねる。** 訓練データセットを差し替えるだけで継続的事前学習にも指示チューニングにも使え、**しかも終始低ビットのまま**である。QLoRA のように FP16 へ戻る必要がない。

## 実験結果と知見

### 量子化としての性能（Table 1・3）

4 ビットでは RTN でさえ健闘するので EfficientQAT の優位はわずか（PPL で約 0.02）。**ビット数が下がるほど差が開く。**

- **3 ビット**: AutoRound に対しゼロショット正解率で約 0.5%、OmniQuant に対し PPL で 0.14〜0.43 の改善
- **2 ビット**: Llama-2-7B で AutoRound を**正解率 5%** 上回る（54.50 → 59.50）
- **一様量子化のままベクトル量子化に並ぶ**: 2 ビットで AQLM（2x8 コードブック）を上回り、AQLM（1x16）と QuIP# にわずかに劣るのみ。ただし後者はハードウェア上のオーバーヘッドが大きく、**推論速度をむしろ落としうる**（Figure 2(a)）

QAT 手法との直接比較（Table 2）では、Llama-2-7B の w2g128 で BitDistiller に PPL 0.89 の差、Llama-3-8B では DB-LLM に対し C4 PPL で 5.98、平均正解率で **8.57 ポイント**の差をつける。

### 指示チューニングとしての性能（Table 4）

Llama-1 を Alpaca でチューニングし 5-shot MMLU で評価。4 ビットでは既存手法と伯仲だが、**2 ビットで差が開く**——7B で QA-LoRA を 5.1%、PEQA を 4.5% 上回り、13B ではそれぞれ 4.0%、8.7%。

### 効率（Table 8・9）

| Llama-2 | Block-AP | E2E-QP | 合計時間 | E2E-QP のメモリ（4/3/2 ビット） |
|---|---|---|---|---|
| 7B | 3.3h / 8.5GB | 〜1.5h | **4.8h** | 7.0 / 6.4 / 5.6GB |
| 13B | 5.6h / 10.3GB | 〜2.9h | 8.5h | 11.7 / 10.6 / 9.1GB |
| 70B | 26.6h / 29.9GB | 〜14.3h | **40.9h** | 48.4 / 42.0 / **34.2GB** |

すべて **A100 1 枚**である。推論側は BitBLAS の INT2 カーネルで線形層が 2.9〜4.4 倍高速化する。一様量子化なので MLC-LLM・AWQ・BitBLAS といった既存ツールボックスにそのまま乗る。

### 視覚言語モデルへの拡張（付録 G）

LLaVA-1.5 のパイプラインでファインチューニング手法だけを差し替える。**2 ビットの LLaVA-1.5-13B（平均 59.9）が、LoRA で訓練した FP16 の LLaVA-1.5-7B（59.6）を上回る**。低ビット化して大きなモデルを使うほうが、高精度で小さいモデルを使うより良いという、QLoRA と同じ含意（→ [[parameter-efficient-fine-tuning]]）がマルチモーダルでも再現している。

## 限界・批判的視点

- **4 ビットでは意味がない。** 著者自身が付録 A で書いている——「4 ビットのグループ単位量子化では、GPTQ・AWQ、さらには最も単純な RTN が同等の性能をより速く達成する」。**EfficientQAT は 2〜3 ビットのための道具である。**
- **2 ビットにはまだ 3〜4% の差が残る。** 「ほぼ無損失」ではない。
- **Llama-3 は量子化しにくい。** 15T トークンで訓練された Llama-3 は 2T の Llama-2 より劣化が大きく、これは EfficientQAT でも解消しない。**訓練が十分に行き渡ったモデルほど冗長性が少なく、量子化の余地が小さい**という一般的な問題を示唆している。
- **相互参照が壊れている。** 原典の 4.2〜4.3 節が Table 5 / 6 / 7 を指すべき箇所をすべて「Table 7」と書き、Figure 5 を指すべき箇所を「Table 5」と書いている。内容から復元できるが、査読を通っていない arXiv 版であることは意識しておくべきである（翻訳側に訳注を付けた）。
- **「41 時間」の内訳に注意。** Block-AP に 26.6 時間、E2E-QP に 14.3 時間。E2E-QP は 4096 サンプル・1 エポックの設定であり、実際に下流タスクへ適応させるならここは増える。
- **評価はゼロショット正解率とパープレキシティ中心。** エージェント用途で問題になる長い trajectory・構造化出力・多段ツール呼び出しでの挙動は測られていない（本 wiki の量子化原典に共通する空白）。

## 実装・運用上の示唆

1. **「訓練対象を絞れば過適合しない」を疑う。** Block-AP の最良設定は**制約を外してデータを増やす**ことだった。正則化を設計する前に、**データを 32 倍にできないか**を先に考える価値がある。
2. **段を分けると、各段の制約が別々に緩む。** Block-AP は「全パラメータを訓練する自由」をブロック単位のスコープで買い、E2E-QP は「端から端までの視野」を訓練対象 1.5% で買う。**同時に満たせない 2 つの要求を、時間軸で分離する**という一般的な設計パターンである。
3. **配布形式まで含めて手法を選ぶ。** 「4 ビットで訓練できる」と「4 ビットで配れる」は別である。QLoRA 系がマージで FP16 へ戻る問題は、訓練メモリだけを見ていると見落とす。
4. **一様量子化を捨てない。** ベクトル量子化のほうが精度は出るが、ハードウェア上のオーバーヘッドで**推論が遅くなりうる**。[[model-quantization]] の「ハードウェアがアルゴリズムを縛る」がここでも効いている。
5. **QAT の適用判断はビット数で切る。** 4 ビットなら PTQ で十分、2〜3 ビットなら QAT を検討する、という明快な線が引ける。

## 用語と略称

- **QAT** = Quantization-Aware Training（量子化を意識した訓練。偽量子化を訓練グラフに挿し、量子化誤差を織り込んで学習する）
- **PTQ** = Post-Training Quantization（訓練後量子化）
- **Q-PEFT** = Quantized Parameter-Efficient Fine-Tuning（量子化されたモデルに対する、パラメータ効率のよいファインチューニング。QLoRA が代表）
- **Block-AP** = Block-wise training of All Parameters（全パラメータのブロック単位訓練）
- **E2E-QP** = End-to-End training of Quantization Parameters（量子化パラメータの端から端までの訓練）
- **STE** = Straight-Through Estimator（丸めのような微分不可能な演算を、恒等関数だったかのように勾配を通す近似）
- **ステップサイズ $s$ / ゼロ点 $z$** = アフィン量子化のスケールと切片（→ [[model-quantization]]）
- **w2g64** = 重み 2 ビット・グループサイズ 64
- **ベクトル量子化（vector quantization）** = 重みを個別にでなくベクトル単位でコードブックへ写す方式。AQLM・QuIP# が代表
- **LVLM** = Large Vision-Language Model（大規模視覚言語モデル）
- **PPL** = Perplexity（パープレキシティ）

## 関連ページ

- [[model-quantization]] — 本論文が属する概念ページ。「QAT は実用になったのか」の節が本論文に依拠している
- [[parameter-efficient-fine-tuning]] — 本論文が批判する Q-PEFT（QLoRA・QA-LoRA・PEQA）の系統
- [[summaries/2023-qlora]] — 「マージで FP16 へ戻る」という批判の主要な対象
- [[summaries/2022-gptq]] / [[summaries/2023-awq]] — PTQ 側のベースライン。4 ビットでは本論文よりこちらが速い
- [[llm-inference-optimization]] — 一様量子化 vs ベクトル量子化のハードウェア効率の話
