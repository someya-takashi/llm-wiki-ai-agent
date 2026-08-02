---
title: "NVFP4: Same Accuracy with 2.3x Higher Throughput for 4-Bit LLMs"
source: "https://medium.com/data-science-collective/nvfp4-same-accuracy-with-2-3x-higher-throughput-for-4-bit-llms-03518ecba108"
author:
  - "[[Benjamin Marie]]"
published: 2025-08-28
created: 2026-08-02
description: "How to quantize LLMs with NVFP4"
tags:
  - "clippings"
---
## How to quantize LLMs with NVFP4

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*MolE_Mhaye8_8G5V)

Photo by Igor Omilaev on Unsplash

As large language models (LLMs) continue to grow in size and complexity, quantization has become an essential technique for making inference more efficient, especially on consumer and enterprise-grade hardware. Among the emerging quantization formats, NVIDIA’s NVFP4 stands out for its tight integration with Blackwell GPUs and promise of significant speedups without major accuracy trade-offs.

*How does NVFP4 compare to widely used 4-bit quantization methods such as AWQ, AutoRound, and bitsandbytes? Should you systematically use NVFP4 models if you have a Blackwell GPU?*

In this article, I put NVFP4 to the test, evaluating it across key dimensions like accuracy, model size, and inference throughput, using publicly available models as well as a few custom-quantized variants.

I also share practical tips on using NVFP4 models with vLLM, and why activation quantization proves to be critical for maintaining NVFP4’s performance edge.

## NVFP4: FP4 Quantization with Dual-Scaling

### How NVFP4 Works

This section presents a simplified overview of how NVP4 works. For more details and illustrations, check NVIDIA’s blog post:

[Introducing NVFP4 for Efficient and Accurate Low-Precision Inference](https://developer.nvidia.com/blog/introducing-nvfp4-for-efficient-and-accurate-low-precision-inference/)

NVIDIA’s Blackwell GPU architecture introduced support for a wide range of numeric formats, from FP64 and FP32/TF32 down to FP16/BF16, INT8/FP8, FP6, and FP4, giving developers flexibility to match precision with workload requirements.

The most advanced FP4 implementation in Blackwell is NVFP4, designed as a new micro floating-point standard to preserve model accuracy at 4-bit precision.

NVFP4 is based on a very compact 4-bit floating-point format called E2M1, where each number is stored using only 4 bits:

- 1 bit for the sign (positive or negative),
- 2 bits for the exponent (which controls the scale or magnitude),
- 1 bit for the mantissa (which controls the precision of the digits).

With this layout, numbers can only cover a limited range, roughly from -6 to +6. On its own, that’s very restrictive.

To make this usable for LLMs, NVFP4 introduces something called a dual-scaling system. Scaling means applying a multiplier so that the small 4-bit numbers can represent larger or more precise values. NVFP4 does this in two steps:

1. Micro-block scaling (local adjustment): Instead of giving every single number its own scale (which would be expensive), NVFP4 groups values into small sets of 16, called *micro-blocks*. All 16 values in a block share the same scaling factor. But importantly, NVFP4 uses an FP8 scaling factor in the E4M3 format. FP8 means 8-bit floating-point, and E4M3 specifically uses 4 exponent bits and 3 mantissa bits. Unlike simpler formats that only allow scaling by powers of two (like doubling or halving), like MXFP4, E4M3 supports fractional scaling (like 1.5x, 2.5x, etc.). This makes the scaling far more precise, so the stored numbers more closely match the original data.
2. Tensor-level scaling (global adjustment): Even with precise block scaling, some datasets span a very wide range of values. To handle this, NVFP4 adds another layer: a high-precision FP32 scaling factor applied across the entire tensor (the large collection of all numbers in a layer of the model). This global scaling makes sure that even when numbers vary dramatically in size, each micro-block can still use its FP8 scaling effectively.

By combining fine-grained FP8 scaling per 16-value block with a global FP32 scale for the entire tensor, NVFP4 reduces rounding errors, preserves important details, and allows the models to run in 4-bit precision with far less accuracy loss than earlier FP4 formats. I explained in more detail the role of the scaling factors in this article:

## [The Impact of the Calibration Dataset for AutoRound and AWQ Quantization](https://kaitchup.substack.com/p/the-impact-of-the-calibration-dataset?source=post_page-----03518ecba108---------------------------------------)

### Explore how calibration datasets influence LLM quantization accuracy with methods like GPTQ, AWQ, and AutoRound, plus…

kaitchup.substack.com

The micro-block size reduction from 32 values in MXFP4 to 16 in NVFP4 further improves adaptability to heterogeneous tensor values, preventing large numbers from overwhelming smaller but important variations. In practice, each 4-bit encoded value (*xq*) within a block is reconstructed as *x = xq × s*, where *s* is the dynamically computed E4M3 scale chosen to minimize block error. This allows NVFP4 to preserve accuracy at 4-bit precision while maintaining memory and compute simplicity compared to higher precision types.

## NVFP4’s Accuracy

NVIDIA’s accuracy benchmarks validate NVFP4’s effectiveness. Post-training quantization of the DeepSeek-R1–0528 model from FP8 to NVFP4 showed almost no degradation across seven evaluation tasks: accuracy differences stayed within 1% on MMLU-Pro, GPQA Diamond, LIVECODEBENCH, and other benchmarks, with SCICODE and Math-500 remaining identical. In the AIME 2024 benchmark, NVFP4 outperformed FP8 by 2%, but this is not significant as there is a high variance between AIME runs. It just means that they perform similarly.

## NVFP4’s Memory Efficiency and Throughput

In NVFP4, each 16-value block stores one 4-bit number per element, one shared FP8 scaling factor, and a single FP32 per tensor scaling factor, averaging ~4.5 bits per value. It’s higher than standard INT4 quantized models, which often use a block size of 128. Compared to FP16, this results in a 3.5x smaller memory footprint, and compared to FP8, about 1.8x savings.

The key advantage of NVFP4 is that it is natively accelerated in hardware on NVIDIA Blackwell GPUs. With INT4 quantization, models cannot operate directly on 4-bit values. Instead, during inference, the INT4 weights must be dequantized, temporarily converted back into 16-bit numbers, before computation can proceed. This extra step adds overhead and limits speed (even though, now, this is extremely optimized in inference frameworks like SGLang and vLLM).

NVFP4 avoids this bottleneck. Because Blackwell Tensor Cores are designed to handle NVFP4 operations directly, tensors remain in their compact 4-bit format throughout inference, as long as both weights and activations are quantized in NVFP4. There is no need for dequantization, and NVFP4 operations are hardware-accelerated, which means computations run much faster. In practice, NVFP4 models achieve higher throughput than INT4 models, which themselves were already significantly faster than standard 16-bit models.

On the software side, NVFP4 is already integrated into the ecosystem. Developers can use [**llm-compressor**](https://github.com/vllm-project/llm-compressor) to quantize models into NVFP4 format, and then run them efficiently with [**vLLM**](https://github.com/vllm-project/vllm), which supports execution of NVFP4 models. Let’s walk through how this workflow operates in practice.

## NVFP4 Quantization with LLM-Compressor

I only tried with the RTX 6000 Pro. I don’t expect it to work with the previous generation of GPUs (Hopper, Ada, etc.).

Install LLM Compressor:

```hs
pip install llmcompressor
```

NVIDIA has released several models quantized to NVFP4. For comparison, I also quantized a model they [had already published in this format: Llama 3.3 Instruct](https://huggingface.co/nvidia/Llama-3.3-70B-Instruct-FP4).

To start our own quantization, we need to load the model. Importantly, it doesn’t need to reside fully on the GPU. Without quantization, Llama 3.3 would require two 80 GB GPUs. However, for NVFP4 quantization, a single RTX 6000 Pro (94 GB VRAM) is sufficient, provided you also have enough CPU RAM to hold the parts of the model that don’t fit into GPU memory.

```hs
MODEL_ID = "meta-llama/Llama-3.3-70B-Instruct"
model = AutoModelForCausalLM.from_pretrained(MODEL_ID, torch_dtype="auto")
tokenizer = AutoTokenizer.from_pretrained(MODEL_ID)
```

Next, we need a calibration dataset. Only a small number of data samples is required. I used 512, but you can go as low as 128 without noticing much difference. Using more samples may improve quantization accuracy slightly (in theory), but beyond 1024, the gains are likely negligible.

That said, I don’t recommend using sequence lengths shorter than 2048. Ideally, you should use much longer sequences to ensure the quantized model is properly calibrated for long-context inference. However, be aware that the quantization cost increases significantly with longer sequences, so there’s a trade-off between calibration quality and compute efficiency.

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

And the quantization runs as follows:

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

The NVFP4 quantization scheme produces a model where both weights and activations are quantized during inference. This means the weights can remain in the NVFP4 format throughout, as the activations are in the same data type. Because no dequantization is required, inference throughput is much higher.

If you prefer not to quantize activations, e.g., to preserve accuracy, you can switch to the NVFP4A16 scheme. In this case, only the weights are quantized, so a calibration dataset is generally not necessary.

```hs
recipe = QuantizationModifier(targets="Linear", scheme="NVFP4A16", ignore=["lm_head"])
# Apply quantization.
oneshot(model=model, recipe=recipe)
```

The weights will be dequantized at runtime, similar to what happens with an INT4 model. As a result, inference throughput is significantly reduced, and most of the performance benefits of using NVFP4 are effectively lost, as we’ll confirm in the next section.

## Inference with NVFP4 Models Using vLLM

I tested with vLLM v0.10.0, and it (almost) works out of the box. You can load NVFP4 models just like any other model type.

However, during testing, I ran into an issue with FlashInfer, a library used to accelerate sampling in vLLM inference. It’s enabled by default if detected, but with NVFP4 models, it caused crashes. There’s probably a way to disable it cleanly, but for the sake of this experiment, I simply uninstalled it (`pip uninstall flashinfer-python`).

Once FlashInfer is fixed, it’s possible that inference with NVFP4 models could be even faster.

Another issue I encountered is that the standard `pip install vllm` still doesn’t install vLLM properly for Blackwell GPUs. This is honestly surprising, and while it’s possible I’m missing something, I’m fairly confident I followed the correct steps. I don’t know why pip can’t install vLLM correctly for Blackwell GPUs.

To get it working, I had to compile vLLM from source, using this set of commands:

```hs
git clone https://github.com/vllm-project/vllm.git 
cd vllm
python use_existing_torch.py
pip install -r requirements/build.txt
pip install setuptools_scm
mkdir ./tmp
MAX_JOBS=10 CCACHE_DIR=./tmp python setup.py develop
```

## Accuracy, Memory Consumption, and Throughput of NVFP4 LLMs

In [their blog post](https://developer.nvidia.com/blog/introducing-nvfp4-for-efficient-and-accurate-low-precision-inference/), NVIDIA only compared NVFP4 against FP8, showing that it performs only slightly worse across most tasks. But how does NVFP4 stack up against other well-established quantization methods already supported by most inference frameworks, such as AWQ, bitsandbytes, and AutoRound?

To find out, I ran a series of comparisons on The Kaitchup Index, using publicly available models alongside two custom NVFP4 variants I quantized myself: one using NVFP4A16 and the other using the NVFP4 scheme, which I assume is similar to the official NVFP4 model released by NVIDIA.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*AoP8_z53CbFUep9l.png)

Image by the author

AWQ and AutoRound 4-bit models (such as those released by OPEA) perform slightly better than the NVFP4 models, including the official one from NVIDIA. Notably, NVFP4 also quantizes activations, yet accuracy remains largely unchanged, especially compared with NVFP4A16, which keeps activations at 16-bit.

NVFP4 models are also almost 7 GB larger, primarily because NVFP4 uses a much smaller group size (16 vs. 128), requiring significantly more scales to be stored. This size increase is somewhat offset by the fact that NVFP4 uses lower bitwidth scales (FP8) compared to AWQ and AutoRound.

That said, I wasn’t expecting a dramatic difference in accuracy between NVFP4 and INT4 models, mainly because 4-bit quantization already performs very close to full-precision for large models like Llama 3.3. The real differences might become more apparent when testing on smaller models (e.g., under 10B parameters), which would make for a very interesting follow-up study.

I was also eager to compare NVFP4 with other FP4 formats, such as MXFP4, used in GPT-OSS models, but unfortunately, LLM Compressor currently doesn’t support MXFP4, so I couldn’t evaluate whether NVFP4 is truly the best FP4 quantization method available today.

While NVFP4 doesn’t stand out in terms of compression ratio or accuracy, it excels in one key area: inference speed. NVFP4 models are significantly faster than any other quantized models I’ve tested, by a wide margin:

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*d875cIaYOtGldgSc.png)

Image by the author

They are 2.35x faster than INT4 models, thanks to Blackwell GPU acceleration for the NVFP4 data type. The results also highlight that activation quantization is essential for preserving this speedup, since the NVFP4A16 model (which only quantizes weights) is only slightly faster than the INT4 models.

## Conclusion

If you have access to a Blackwell GPU, I highly recommend using NVFP4 quantization. The accuracy is more than sufficient, and the inference throughput is dramatically better.

Given NVFP4’s strong performance and the fact that very few models are currently available in this format, I’m considering releasing some NVFP4-quantized models myself. The quantization cost isn’t too high, and I may continue doing it until larger organizations begin systematically releasing models in NVFP4. You can already find some of the models I quantized with NVFP4, here:

- [NVFP4 Collection](https://huggingface.co/collections/kaitchup/nvfp4-68ac613bb02570bd89a78137)

### QLoRA Fine-Tuning with NVFP4 Models

Is it possible? Yes! NVFP4 is just a data type and a quantization format. QLoRA can be applied to models quantized with any format and data type ([this is detailed in Chapter 4 of LLMs on a Budget](https://benjaminmarie.gumroad.com/l/llms-on-a-budget)). However, as far as I know, no frameworks currently support QLoRA for NVFP4 models. It is really easy to implement for Hugging Face in TRL and PEFT, as they have already integrated support for other quantization formats. We will probably get this possibility soon.