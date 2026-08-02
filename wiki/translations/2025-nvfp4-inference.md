---
type: translation
source_path: raw/articles/NVFP4_ Same Accuracy with 2.3x Higher Throughput for 4-Bit LLMs.md
source_page: "[[summaries/2025-nvfp4-inference]]"
original_language: en
translated_to: ja
translated_at: 2026-08-02
---

# NVFP4: 4 ビット LLM で同じ精度を 2.3 倍のスループットで

> 原題: NVFP4: Same Accuracy with 2.3x Higher Throughput for 4-Bit LLMs
> 副題: How to quantize LLMs with NVFP4
> 著者: Benjamin Marie
> 出典: Medium（Data Science Collective）, 2025-08-28

> **訳注（クリップの状態と除外したもの）**
> - 底本は Medium 記事の Web Clipper クリップ。本文・コード・見出しに欠落はなく、**クリップは健全**であった。
> - **画像 3 枚のうち 2 枚を保存し、1 枚を除外した。** 除外したのは冒頭の装飾写真（Unsplash の "Photo by Igor Omilaev"）で、記事内容と無関係なヘッダー画像であるため（ingest skill のケース C・除外規則）。保存した 2 枚は著者自身が作成した測定結果のチャートである。Medium の CDN URL から `format:webp/` を外して素の PNG を取得した。
> - **本文でない定型要素は訳出しない**（skill 既定）。具体的には、著者の別記事への誘導カード（"The Impact of the Calibration Dataset for AutoRound and AWQ Quantization" の見出しとその紹介文・ドメイン名）を除外した。
> - コードブロックは原文のまま残している（原典では `hs` という言語指定になっているが、内容は Python とシェルコマンドである）。

## How to quantize LLMs with NVFP4（NVFP4 で LLM を量子化する方法）

大規模言語モデル（LLM）がサイズと複雑さにおいて成長し続けるにつれ、とくにコンシューマ向けおよびエンタープライズ級のハードウェア上で推論をより効率的にするために、量子化は本質的な技法になってきた。新興の量子化形式の中でも、NVIDIA の **NVFP4** は Blackwell の GPU との緊密な統合と、大きな精度上の代償なしに顕著な高速化をもたらすという約束で際立っている。

*NVFP4 は、AWQ・AutoRound・bitsandbytes といった広く用いられる 4 ビット量子化の手法と比べてどうなのか。Blackwell の GPU を持っているなら、体系的に NVFP4 のモデルを使うべきなのか。*

本記事では NVFP4 を検証にかけ、公開されているモデルといくつかの独自に量子化した変種を用いて、精度・モデルサイズ・推論スループットといった主要な次元にわたって評価する。

また、vLLM で NVFP4 のモデルを使ううえでの実践的な助言と、**なぜ活性の量子化が NVFP4 の性能上の優位を保つのに決定的なのか**も共有する。

## NVFP4: FP4 Quantization with Dual-Scaling（NVFP4: 二重スケーリングを伴う FP4 量子化）

### How NVFP4 Works（NVFP4 の仕組み）

本節は NVP4 の仕組みの簡略化した概観を示す。より詳細と図解については NVIDIA のブログ記事を参照のこと: [Introducing NVFP4 for Efficient and Accurate Low-Precision Inference](https://developer.nvidia.com/blog/introducing-nvfp4-for-efficient-and-accurate-low-precision-inference/)

NVIDIA の Blackwell GPU アーキテクチャは、FP64 と FP32/TF32 から FP16/BF16、INT8/FP8、FP6、FP4 に至る広範囲の数値形式への支援を導入し、開発者が精度をワークロードの要求に合わせる柔軟性を与えた。

Blackwell における最も先進的な FP4 の実装が **NVFP4** であり、4 ビット精度でモデルの精度を保つための新しいマイクロ浮動小数点の標準として設計されている。

NVFP4 は **E2M1** と呼ばれる非常にコンパクトな 4 ビット浮動小数点形式に基づいており、各数値はわずか 4 ビットで格納される:

- 符号に 1 ビット（正か負か）
- 指数に 2 ビット（スケールすなわち大きさを制御する）
- 仮数に 1 ビット（数字の精度を制御する）

このレイアウトでは、数値はおおよそ -6 から +6 という限られた範囲しかカバーできない。それ自体では非常に制約が強い。

これを LLM にとって使えるものにするため、NVFP4 は**二重スケーリング（dual-scaling）** と呼ばれるものを導入する。スケーリングとは、小さな 4 ビットの数値がより大きな、あるいはより精密な値を表現できるように乗数を適用することを意味する。NVFP4 はこれを 2 段階で行う:

1. **マイクロブロックのスケーリング（局所的な調整）**: すべての数値に個別のスケールを与える（これは高価になる）代わりに、NVFP4 は値を **16 個の小さな集合**——**マイクロブロック**と呼ばれる——にまとめる。ブロック内の 16 個の値はすべて同じスケーリング係数を共有する。しかし重要なのは、NVFP4 が **E4M3 形式の FP8 のスケーリング係数**を用いる点である。FP8 は 8 ビット浮動小数点を意味し、E4M3 は具体的に指数 4 ビットと仮数 3 ビットを用いる。MXFP4 のように 2 の冪によるスケーリング（2 倍や半分など）しか許さないより単純な形式とは異なり、**E4M3 は小数のスケーリング（1.5 倍、2.5 倍など）を支援する**。これによりスケーリングははるかに精密になり、格納される数値が元のデータにより近く一致する。
2. **テンソル水準のスケーリング（大域的な調整）**: 精密なブロックスケーリングをもってしても、一部のデータセットは非常に広い範囲の値にまたがる。これを扱うため、NVFP4 はもう一つの層を加える——**テンソル全体（モデルのある層のすべての数値の大きな集まり）にわたって適用される高精度の FP32 のスケーリング係数**である。この大域的なスケーリングは、数値の大きさが劇的に変わるときでも各マイクロブロックがその FP8 のスケーリングを効果的に使えることを保証する。

16 個の値ごとの細粒度な FP8 のスケーリングと、テンソル全体の大域的な FP32 のスケールを組み合わせることで、NVFP4 は丸め誤差を減らし、重要な細部を保ち、以前の FP4 形式よりはるかに小さい精度損失でモデルを 4 ビット精度で動かせるようにする。

**マイクロブロックのサイズが MXFP4 の 32 個から NVFP4 の 16 個へ減った**ことは、異質なテンソル値への適応性をさらに改善し、大きな数値が小さいが重要な変動を圧倒することを防ぐ。実際には、ブロック内の各 4 ビット符号化値（*xq*）は $x = x_q \times s$ として再構成される。ここで *s* はブロックの誤差を最小化するよう動的に計算された E4M3 のスケールである。これにより NVFP4 は、より高精度の型と比べてメモリと計算の単純さを保ちながら、4 ビット精度で精度を保つことができる。

## NVFP4's Accuracy（NVFP4 の精度）

NVIDIA の精度ベンチマークは NVFP4 の有効性を裏づけている。DeepSeek-R1–0528 モデルを FP8 から NVFP4 へ訓練後量子化すると、7 つの評価タスクにわたってほとんど劣化がないことが示された——MMLU-Pro・GPQA Diamond・LIVECODEBENCH その他のベンチマークで精度差は 1% 以内に留まり、SCICODE と Math-500 は同一のままだった。AIME 2024 のベンチマークでは NVFP4 が FP8 を 2% 上回ったが、**AIME の実行間には大きな分散があるのでこれは有意ではない**。単に同程度の性能であることを意味するだけである。

## NVFP4's Memory Efficiency and Throughput（NVFP4 のメモリ効率とスループット）

NVFP4 では、16 個の値の各ブロックが要素あたり 1 つの 4 ビットの数値・1 つの共有 FP8 スケーリング係数・テンソルあたり単一の FP32 スケーリング係数を格納し、**値あたり平均で約 4.5 ビット**になる。これは**しばしばブロックサイズ 128 を使う標準的な INT4 量子化モデルより多い**。FP16 と比べると **3.5 倍小さいメモリ使用量**、FP8 と比べると約 **1.8 倍の節約**になる。

NVFP4 の鍵となる利点は、**NVIDIA Blackwell の GPU 上でハードウェアによってネイティブに加速される**ことである。INT4 の量子化では、モデルは 4 ビットの値の上で直接動作できない。代わりに推論の間、INT4 の重みは計算を進める前に**逆量子化され、一時的に 16 ビットの数値へ戻される**必要がある。この追加のステップはオーバーヘッドを加え速度を制限する（もっとも今では SGLang や vLLM のような推論フレームワークで極度に最適化されてはいる）。

NVFP4 はこのボトルネックを避ける。Blackwell の Tensor Core が NVFP4 の演算を直接扱うよう設計されているので、**重みと活性の双方が NVFP4 で量子化されている限り**、テンソルは推論を通じてコンパクトな 4 ビット形式のままである。逆量子化の必要がなく、NVFP4 の演算はハードウェアで加速されるので、計算がはるかに速く走ることを意味する。実際には、NVFP4 のモデルは INT4 のモデルより高いスループットを達成し、その INT4 自体が既に標準的な 16 ビットのモデルより大幅に速かった。

ソフトウェアの側では、NVFP4 は既にエコシステムへ統合されている。開発者は [**llm-compressor**](https://github.com/vllm-project/llm-compressor) を使ってモデルを NVFP4 形式へ量子化し、その後 NVFP4 モデルの実行を支援する [**vLLM**](https://github.com/vllm-project/vllm) で効率的に実行できる。このワークフローが実際にどう動くのかを見ていこう。

## NVFP4 Quantization with LLM-Compressor（LLM-Compressor による NVFP4 の量子化）

私が試したのは RTX 6000 Pro のみである。前世代の GPU（Hopper、Ada など）では動作しないと予想する。

LLM Compressor をインストールする:

```hs
pip install llmcompressor
```

NVIDIA はいくつかのモデルを NVFP4 へ量子化して公開している。比較のために、私は[彼らが既にこの形式で公開しているモデル: Llama 3.3 Instruct](https://huggingface.co/nvidia/Llama-3.3-70B-Instruct-FP4) も自分で量子化した。

自分の量子化を始めるには、モデルを読み込む必要がある。重要なのは、**モデルが完全に GPU 上に置かれる必要はない**ことである。量子化なしなら Llama 3.3 は 80 GB の GPU が 2 台必要になる。しかし NVFP4 の量子化には、GPU メモリに収まらない部分を保持するのに十分な CPU RAM があれば、**RTX 6000 Pro（94 GB VRAM）1 台で十分**である。

```hs
MODEL_ID = "meta-llama/Llama-3.3-70B-Instruct"
model = AutoModelForCausalLM.from_pretrained(MODEL_ID, torch_dtype="auto")
tokenizer = AutoTokenizer.from_pretrained(MODEL_ID)
```

次にキャリブレーションデータセットが必要である。少数のデータサンプルしか要らない。私は 512 を使ったが、大きな違いに気づくことなく 128 まで下げられる。より多くのサンプルを使うと（理論上は）量子化の精度がわずかに改善しうるが、1024 を超えると利得はおそらく無視できる。

とはいえ、**2048 より短い系列長を使うことは推奨しない**。理想的には、量子化されたモデルが長コンテキストの推論に対して適切に較正されることを保証するために、はるかに長い系列を使うべきである。しかし量子化のコストは系列が長くなるにつれて大幅に増えるので、較正の品質と計算効率の間にトレードオフがある点に注意されたい。

```hs
from datasets import load_dataset
NUM_CALIBRATION_SAMPLES=512
MAX_SEQUENCE_LENGTH=2048
# Load dataset.
ds = load_dataset("HuggingFaceH4/ultrachat_200k", split=f"train_sft[:{NUM_CALIBRATION_SAMPLES}]")
ds = ds.shuffle(seed=42)

# Preprocess the data into the format the model is trained with.
def preprocess(example):
    return {"text": tokenizer.apply_chat_template(example["messages"], tokenize=False,)}
ds = ds.map(preprocess)
# Tokenize the data (be careful with bos tokens - we need add_special_tokens=False since the chat_template already added it).
def tokenize(sample):
    return tokenizer(sample["text"], padding=False, max_length=MAX_SEQUENCE_LENGTH, truncation=True, add_special_tokens=False)
ds = ds.map(tokenize, remove_columns=ds.column_names)
```

そして量子化は次のように実行する:

```hs
# Configure the quantization algorithm to run.
recipe = QuantizationModifier(targets="Linear", scheme="NVFP4", ignore=["lm_head"])
# Apply quantization.
oneshot(
    model=model,
    dataset=ds,
    recipe=recipe,
    max_seq_length=MAX_SEQUENCE_LENGTH,
    num_calibration_samples=NUM_CALIBRATION_SAMPLES,
)
```

NVFP4 の量子化の方式は、**推論中に重みと活性の双方が量子化された**モデルを生成する。これは、活性が同じデータ型なので重みが NVFP4 形式のままでいられることを意味する。逆量子化が必要ないので、推論のスループットははるかに高くなる。

活性を量子化したくない場合（例: 精度を保つため）は、**NVFP4A16** の方式に切り替えられる。この場合、重みだけが量子化されるので、キャリブレーションデータセットは一般に必要ない。

```hs
recipe = QuantizationModifier(targets="Linear", scheme="NVFP4A16", ignore=["lm_head"])
# Apply quantization.
oneshot(model=model, recipe=recipe)
```

重みは INT4 のモデルで起こることと同様に**実行時に逆量子化される**。その結果、推論のスループットは大幅に低下し、次節で確認するとおり **NVFP4 を使う性能上の恩恵のほとんどが事実上失われる**。

## Inference with NVFP4 Models Using vLLM（vLLM を用いた NVFP4 モデルの推論）

私は vLLM v0.10.0 で試したが、（ほぼ）そのまま動く。NVFP4 のモデルは他のどのモデル型とも同じように読み込める。

しかしテスト中に、vLLM の推論でサンプリングを加速するのに使われるライブラリ **FlashInfer** の問題に遭遇した。検出されると既定で有効になるが、NVFP4 のモデルではクラッシュを引き起こした。きれいに無効化する方法がおそらくあるのだろうが、この実験のためには単にアンインストールした（`pip uninstall flashinfer-python`）。

FlashInfer が修正されれば、NVFP4 モデルでの推論はさらに速くなる可能性がある。

私が遭遇したもう一つの問題は、標準の `pip install vllm` が **Blackwell の GPU 向けに vLLM を正しくインストールしない**ことである。これは正直なところ驚きで、私が何かを見落としている可能性はあるものの、正しい手順に従ったとかなり確信している。なぜ pip が Blackwell の GPU 向けに vLLM を正しくインストールできないのか分からない。

動くようにするには、次の一連のコマンドを使ってソースから vLLM をコンパイルする必要があった:

```hs
git clone https://github.com/vllm-project/vllm.git 
cd vllm
python use_existing_torch.py
pip install -r requirements/build.txt
pip install setuptools_scm
mkdir ./tmp
MAX_JOBS=10 CCACHE_DIR=./tmp python setup.py develop
```

## Accuracy, Memory Consumption, and Throughput of NVFP4 LLMs（NVFP4 の LLM の精度・メモリ消費・スループット）

[彼らのブログ記事](https://developer.nvidia.com/blog/introducing-nvfp4-for-efficient-and-accurate-low-precision-inference/)で、**NVIDIA は NVFP4 を FP8 とだけ比較しており**、ほとんどのタスクでわずかに劣るだけであることを示している。しかし NVFP4 は、AWQ・bitsandbytes・AutoRound といった、ほとんどの推論フレームワークで既に支援されている他の確立された量子化の手法と比べてどうなのか。

これを確かめるため、公開されているモデルと、私が自分で量子化した 2 つの独自の NVFP4 の変種——1 つは NVFP4A16 を、もう 1 つは NVIDIA が公開している公式の NVFP4 モデルと似ていると想定される NVFP4 の方式を用いたもの——を並べて、The Kaitchup Index で一連の比較を行った。

<figure>

![](../../raw/assets/2025-nvfp4-inference/accuracy-size-table.png)

<figcaption>図1: The Kaitchup Index における量子化された Llama 3.3 の比較（指示追従・世界知識・多言語能力のベンチマーク）。青がスコア、緑がサイズ（GB）。AWQ 5901 / 37.09GB、AutoRound INT4 5900 / 36.83GB、bitsandbytes 4bit 5814 / 39.81GB、NVIDIA 公式 FP4 5858 / 43.82GB、独自 NVFP4 5854 / 43.79GB、独自 NVFP4A16 5878 / 39.88GB、AutoRound INT2 5488 / 24.41GB。訳注: 数値は図から読み取って転記した。</figcaption>
</figure>

AWQ と AutoRound の 4 ビットモデル（OPEA が公開しているものなど）は、**NVIDIA の公式のものを含む NVFP4 のモデルよりわずかに良い性能を示す**。特筆すべきは、NVFP4 が活性も量子化しているにもかかわらず精度が大きく変わらないことで、とくに活性を 16 ビットに保つ NVFP4A16 と比べてそうである。

NVFP4 のモデルはまた**ほぼ 7 GB 大きい**。これは主に **NVFP4 がはるかに小さいグループサイズ（128 に対して 16）を使うため、格納すべきスケールが大幅に多くなる**からである。このサイズの増加は、NVFP4 が AWQ や AutoRound と比べてより低いビット幅のスケール（FP8）を使うという事実によっていくらか相殺されている。

とはいえ、私は NVFP4 と INT4 のモデルの間に劇的な精度差を期待していたわけではない。主な理由は、**Llama 3.3 のような大きなモデルでは 4 ビット量子化が既に完全精度に非常に近い性能を示す**からである。実際の差は**より小さなモデル（例: 100 億パラメータ未満）でテストするとより明らかになるかもしれず**、それは非常に興味深い後続の研究になるだろう。

また、GPT-OSS のモデルで使われている MXFP4 のような他の FP4 形式と NVFP4 を比較したいとも思っていたが、残念ながら LLM Compressor は現在 MXFP4 を支援していないので、NVFP4 が今日利用できる真に最良の FP4 量子化の手法なのかどうかは評価できなかった。

NVFP4 が圧縮率や精度の点で際立たない一方、**一つの主要な領域で優れている——推論の速度**である。NVFP4 のモデルは、私がテストした他のどの量子化モデルより、大きな差で著しく速い:

<figure>

![](../../raw/assets/2025-nvfp4-inference/throughput-chart.png)

<figcaption>図2: The Kaitchup Index における量子化された Llama 3.3 の推論スループット（vLLM v0.10.0、RTX 6000 Pro）。青が入力速度、緑が出力速度（いずれも tokens/sec）。AWQ 1431/720、AutoRound INT4 1437/723、bitsandbytes 4bit 1150/585、NVIDIA 公式 FP4 3342/1692、独自 NVFP4 3358/1693、独自 NVFP4A16 1534/774、AutoRound INT2 1222/659。訳注: 数値は図から読み取って転記した。</figcaption>
</figure>

Blackwell GPU による NVFP4 のデータ型の加速のおかげで、それらは **INT4 のモデルより 2.35 倍速い**。この結果はまた、**活性の量子化がこの高速化を保つのに本質的である**ことも際立たせている。重みだけを量子化する NVFP4A16 のモデルは、INT4 のモデルよりわずかに速いだけだからである。

## Conclusion（結論）

Blackwell の GPU にアクセスできるなら、**NVFP4 の量子化を強く推奨する**。精度は十分以上であり、推論のスループットは劇的に良い。

NVFP4 の強い性能と、現在この形式で利用できるモデルがごく少ないという事実を踏まえて、私は自分でいくつかの NVFP4 量子化モデルを公開しようと考えている。量子化のコストはさほど高くなく、より大きな組織が体系的に NVFP4 でモデルを公開し始めるまで続けるかもしれない。私が NVFP4 で量子化したモデルのいくつかは既にここで見つけられる:

- [NVFP4 Collection](https://huggingface.co/collections/kaitchup/nvfp4-68ac613bb02570bd89a78137)

### QLoRA Fine-Tuning with NVFP4 Models（NVFP4 モデルでの QLoRA ファインチューニング）

可能なのか。可能である。**NVFP4 は単なるデータ型であり量子化形式である。** QLoRA はどんな形式・データ型で量子化されたモデルにも適用できる（[これは『LLMs on a Budget』の第 4 章で詳述されている](https://benjaminmarie.gumroad.com/l/llms-on-a-budget)）。しかし私の知る限り、**現在 NVFP4 のモデルに対する QLoRA を支援するフレームワークはない**。Hugging Face の TRL と PEFT には他の量子化形式への支援が既に統合されているので、実装するのは実に容易である。おそらく近いうちにこの可能性が得られるだろう。
