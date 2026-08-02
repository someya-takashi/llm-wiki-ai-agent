---
title: "LLM Quantization Explained: A Complete Guide"
source: "https://medium.com/@abhinaykrishna/llm-quantization-in-depth-1fa65ac24f2a"
author:
  - "[[Abhinaykrishna]]"
published: 2025-03-27
created: 2026-08-02
description: "More"
tags:
  - "clippings"
---
## WHAT is Quantization?

- Quantization is a compression technique that maps high-precision values to lower-precision ones.
- In LLMs, this involves reducing the precision of weights and activations, making the model more memory-efficient.

## WHY Quantization?

- Quantization lowers the precision of model weights (e.g., from 16-bit to 4-bit), significantly reducing memory requirements.
- For example, a 7B parameter model that requires 28GB in FP16 can fit into just 7GB using 4-bit quantization, making it possible to run on consumer GPUs.
- Lower-precision computations require fewer hardware resources and can improve inference speed on supported hardware.

## QUANTIZATION BASICS

A floating-point number consists of bits structured into three key components:

- **Sign Bit** — Determines whether the number is positive (0) or negative (1).
- **Exponent** — Defines the scaling factor by raising the base (typically 2 in binary) to a specific power, enabling representation of both extremely large and tiny values.
- **Significand (Mantissa)** — Holds the meaningful digits of the number, with its bit-length directly influencing numerical precision.
![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*--fmt1Z2CTSJllei.png)

FP32, using four bytes, is considered full precision, whereas BF16 and FP16, which use fewer bits, fall under half precision.

- Consider a large language model (LLM) with 1 billion parameters, where weights are typically stored in FP32 (32-bit format). The memory requirements for different precision levels can be calculated as follows:
- **FP32:** 1 billion parameters × 4 bytes = 4.0 GB
- **INT8:** 1 billion parameters × 1 byte = 1.0 GB

Quantization reduces memory usage by converting a continuous range of values into a limited set of discrete values, enabling more efficient storage and computation.

## How Does Quantization Introduce Error?

For example, consider the number **5.62** undergoing **INT4 quantization** (4-bit precision).

1. INT4 Representation: A 4-bit format can represent 16 discrete values (since 2 ⁴ = 16).
2. Mapping to a Limited Range: The possible values might be something like \[-1, -0.6, -0.3, …, 0.6, 0.8, 1\] (this is just an approximation).
3. Rounding During Quantization: Since 0.562 is closest to 0.6, it is mapped to an index (e.g., 14).
4. Dequantization Error: When converting back, 14 maps to 0.6, causing a small error of 0.6–0.562

## Example: Quantizing an FP32 Tensor to INT8

Quantization maps floating-point (FP32) values to integer (INT8) values using a **scale factor** (which adjusts the range) and a **zero-point** (which shifts values for proper alignment).

Below is a step-by-step example demonstrating this process of both quantization and dequantization.

We start with a tensor containing FP32 values:

```c
X_fp32 = [0.2, 1.5, -2.3, 3.8]
```

The INT8 format represents values in the range:

```c
[-128, 127]
```

**Step 1:Compute Scale and Zero-Point**

The **scale factor** ensures that the FP32 range fits within the INT8 range:

```c
scale = (max(X_fp32) - min(X_fp32)) / (127 - (-128))
```

Substituting values:

```c
scale = (3.8 - (-2.3)) / 255  # 6.1 / 255 ≈ 0.0239
```

For simplicity, we assume:

```c
zero_point = 0
```

**Step 2: Convert FP32 to INT8**

Using the quantization formula:

```c
X_int8 = round(X_fp32 / scale)
```

Applying this to each value:

```c
round(0.2 / 0.0239)  # 8
round(1.5 / 0.0239)  # 63
round(-2.3 / 0.0239) # -96
round(3.8 / 0.0239)  # 159 (clamped to 127)
```

Final INT8 values:

```c
X_int8 = [8, 63, -96, 127]
```

**Step 3: Convert Back to FP32 (Dequantization)**

To approximate the original values, we use:

```c
X_fp32_recovered = X_int8 * scale
```

Applying to each value:

```c
8    * 0.0239  # 0.191
63   * 0.0239  # 1.51
-96  * 0.0239  # -2.29
127  * 0.0239  # 3.03
```

Recovered FP32 values:

```c
X_fp32_recovered = [0.19, 1.51, -2.29, 3.03]
```

This is a simplified example to help illustrate the concept of quantization. In practice, real quantization techniques are much more advanced.

## Two types of LLM quantization

## Post-training quantization (PTQ)

- Quantization is performed after the model has been fully trained.
- It involves converting weights and potentially activations from higher precision to lower precision. Common methods include static and dynamic quantization.
![](https://miro.medium.com/v2/resize:fit:1100/format:webp/0*_TEzCcck87nWH_5j.png)

## Quantization-aware training (QAT)

- The model is trained while simulating low-precision arithmetic so that it learns to compensate for quantization errors.
- During training, the model simulates lower precision operations, enabling it to adapt to the effects of quantization.
- This typically offers better performance than PTQ, as the model learns to reduce quantization errors during training.
![](https://miro.medium.com/v2/resize:fit:1100/format:webp/0*RMDurTeAg-BRDTvG.png)

## What Can Be Quantized in an LLM?

When people say an LLM is **“4-bit quantized”**, they are usually referring to **weight quantization**. However, different parts of a model can be quantized depending on the deployment scenario.

## 1\. Weight Quantization (Most Common)

Weight quantization compresses the model’s learned parameters, significantly reducing the model size and VRAM required to load it.

- Used by **GPTQ, AWQ, GGUF, and BitsAndBytes**
- Best when the model itself is too large to fit into GPU memory
- This is the type of quantization most people refer to when discussing LLM quantization.

## 2\. Activation Quantization (Less Common)

Activation quantization reduces the precision of intermediate values produced during inference.

- Mainly used in **Quantization-Aware Training (QAT)** or specialized inference engines.
- More difficult than weight quantization because activations change for every input.
- Less commonly used in day-to-day LLM deployment.

## 3\. KV Cache Quantization (Increasingly Popular)

During text generation, transformer models store previous Keys and Values in memory so they don’t need to recompute them for every new token.

As conversations become longer, this **KV Cache** can consume several gigabytes of GPU memory.

Quantizing the KV Cache:

- Reduces memory usage during inference.
- Enables longer context windows.
- Allows serving more concurrent users on the same GPU.

This has become increasingly popular in modern inference frameworks such as **vLLM** and **TensorRT-LLM**, especially for long-context models.

## Calibration Dataset

Several post-training quantization techniques, such as **GPTQ** and **AWQ**, require a small **calibration dataset** before quantization.

A calibration dataset consists of representative samples that are passed through the model to collect activation statistics. These statistics help determine optimal scaling factors and identify important weights that should be preserved during quantization.

Unlike fine-tuning, calibration **does not update the model weights**. It simply gathers information that enables more accurate quantization. In practice, a few hundred to a few thousand representative samples are usually sufficient.

> *Calibration collects statistics only — it does* ***not*** *update or retrain the model.*

## Techniques for LLM quantization

Each quantization technique is optimized for a different deployment scenario.

## 1\. BITS AND BYTES QUANTIZATION

## a) 8-bit Quantization (LLM.int8)

- Method Used: INT8 Quantization (LLM.int8())
- Instead of quantizing individual elements, it divides weights into small blocks (e.g., 64 elements per block).
- Each block gets its own scaling factor to maintain numerical accuracy.
- During computation, weights are dequantized on the fly when needed.
- Uses mixed-precision calculations (int8 × fp16) to reduce errors.
- Some outlier weights are stored in full precision (fp16) to prevent large accuracy loss.
- Keeps activations in fp16/bf16 to maintain stability.
- LLM.int8() does not degrade model accuracy much.

## b) 4-bit Quantization ( LLM.int4)

- 4x memory reduction over fp16 models.
- Minimal performance loss compared to full precision.
- Technique is mostly similar to 8bit,uses nf4 instead of nf8 except that in 8-bit quantization, outlier values are processed in fp16 to maintain accuracy, while 4-bit quantization relies on the NF4 data type’s design to represent weight distributions effectively.
```c
import torch
from bitsandbytes import BitsAndBytesConfig
# 4-bit Quantization Configuration
bnb_4bit_config = BitsAndBytesConfig(
 load_in_4bit=True,
 bnb_4bit_use_double_quant=True,
 bnb_4bit_quant_type='nf4')
# 8-bit Quantization Configuration
bnb_8bit_config = BitsAndBytesConfig(
 load_in_8bit=True,
)
```

**Bnb\_4bit\_use\_double\_quant**

- A process of quantizing the quantization constants for additional memory savings. Weights are quantized in blocks of 64, and while this facilitates precise 4-bit quantization, you also have to account for the scaling factors of each block — which increases the amount of memory required. DQ addresses this issue by performing a second round of quantization on the scaling factors for each block.
- The 32-bit scaling factors are compiled into blocks of 256 and quantized to 8-bit. Consequently, where a 32-bit scaling factor for each block of previously added 0.5 bits per weight, DQ brings this down to only 0.127 bits.
- Though seemingly insignificant, when combined in a 65B LLM, for example, this saves 3 GB of memory.

Bitsandbytes is mostly used for finetuning on less GPU memory using the famous techniques LORA and QLORA

## 2\. GPTQ (general pre-trained transformer quantization)

GPTQ is designed to reduce model size by applying layer-wise quantization, optimizing quantized weights to minimize output error.

**Mechanism**:

- Quantizes the model one layer at a time, adjusting weights in batches and minimizing the mean squared error (MSE) between the original and quantized layers.
- Uses four-bit integers for quantized weights while maintaining activations in 16-bit float (FP16) precision. Weights are dequantized during inference for computation in FP16.
- Requires calibration set
- First, all the model’s weights are converted into a matrix, which is worked through in batches of 128 columns at a time through a process called lazy batch updating.
- This involves quantizing the weights in batch, calculating the MSE, and updating the weights to values that diminish it.
- After processing the calibration batch, all the remaining weights in the matrix are updated in accordance with the MSE of the initial batch — and then all the individual layers are re-combined to produce a quantized model.

## 3\. GGML/GGUF (Georgi Gerganov Machine Learning / GPT-generated unified format)

GGUF (**GPT-generated Unified Format**) is an advanced model file format built upon **GGML** (**Georgi Gerganov Machine Learning**). It optimizes model execution for **CPU inference**, allowing **quantized models** to run efficiently on resource-limited devices.

**Mechanisms**:

- **k-Quant System:** Divides model weights into blocks and quantizes them using various bit-width methods depending on importance (e.g., q2\_k, q5\_0, q8\_0).
- **GGUF:** Extends GGML to support a broader range of models and is backward-compatible**.**

**Advantages**: Optimized for CPU execution, supports a wide range of models.

Example: How q3\_k Works

1. **Block-wise Quantization** → The model divides weights into fixed-size blocks.
2. **3-bit Weight Storage** → Each weight is stored using **3 bits per parameter**.
3. **Grouped Scaling** → Scaling factors are applied at the block level to retain numerical precision.

To run GGML or GGUF models, however, you need to use a C/C++ library called llama.cpp — which was also developed by GGML’s creator Georgi Gerganov.

llama.cpp is capable of reading models saved in the.GGML or.GGUF format and enables them to run on CPU devices as opposed to requiring GPUs.

## 4\. AWQ quantization (W4A16)

- Activation-Aware Weight Quantization (AWQ) is a technique designed to compress large language models (LLMs) by focusing on the importance of weights during inference.
- Unlike traditional methods that quantize all weights uniformly, AWQ identifies and protects a small subset of ‘salient’ weights — those most critical to the model’s performance — typically comprising about 1% of the total weights.
- To identify these salient weights, AWQ collects activation statistics from the model using a calibration dataset. Instead of skipping quantization for these weights, AWQ applies per-channel scaling to them before quantization. This approach enhances accuracy while maintaining hardware efficiency.
- The remaining 99% of the weights are then quantized to lower-bit representations, such as INT3 or INT4, effectively reducing the model’s memory footprint without significantly compromising performance.
- This method allows LLMs to be deployed more efficiently on resource-constrained devices, balancing computational efficiency with model accuracy

## Choosing the Right Quantization Method

There is no single “best” quantization technique. The right choice depends on your hardware, use case, and whether you prioritize inference speed, memory efficiency, or model accuracy.

**• If you want to run an LLM on a CPU:** Use **GGUF**, which is optimized for efficient CPU inference through llama.cpp.

**• If you want to run an LLM on an NVIDIA GPU:**Choose **GPTQ** or **AWQ** for fast and memory-efficient inference.

**• If you want to fine-tune an LLM with limited GPU memory:**Use **BitsAndBytes** with **LoRA** or **QLoRA**.

**• If maximum model accuracy is your priority:**Run the model in **FP16** or **BF16** without quantization.

**• If your primary goal is reducing memory usage:**Use **4-bit quantization**, which offers up to a 4× reduction in model size with only a small loss in accuracy.

As a general rule, **GPTQ** provides an excellent balance between speed and accuracy for GPU inference, **AWQ** focuses on preserving model quality, **GGUF** is ideal for CPU deployments, and **BitsAndBytes** remains the standard choice for memory-efficient fine-tuning.

## Comparision of various quantization methods

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*I-l43zhb-N4fuVOc.png)

## When Should You Avoid Quantization?

Although quantization significantly reduces memory usage, it is **not always the best choice**.

Avoid aggressive quantization when:

- **Maximum model accuracy is critical.**
- **You are training a model from scratch.**
- **Your hardware has sufficient memory** to run the model in FP16/BF16.
- **Even a small drop in model quality is unacceptable** for your application.

## Does Quantization Always Reduce Latency?

**No. Lower memory usage does not always mean faster inference.**

Latency may increase in the following situations:

- **Dequantization Overhead:**Many quantization methods store weights in INT4 or INT8 but perform computations in FP16/BF16. During inference, the weights are first **dequantized**, adding extra computation.
- **Lack of Hardware Support:**If the GPU or CPU does not have dedicated low-precision (INT4/INT8) acceleration, the hardware cannot fully benefit from quantized weights.
- **Unoptimized Inference Engines:** Some frameworks dequantize weights before every matrix multiplication instead of performing native quantized operations, increasing inference time.
- **Small Models:**For smaller models, the dequantization overhead can outweigh the memory savings, resulting in little or no speed improvement.

## Key Takeaway

- Quantization **almost always reduces memory usage.**
- It enables **larger models to run on limited hardware.**
- **Lower latency is not guaranteed** — it depends on the hardware, inference engine, and quantization method being used.

For the best performance, use optimized inference frameworks such as **TensorRT-LLM, vLLM, llama.cpp, or ONNX Runtime**, which are designed to execute quantized models efficiently.

\===============================================================

If you found this article insightful, please clap, share, or leave a comment with your thoughts.

You can reach me out at linkedin: [https://www.linkedin.com/in/abhinay23/](https://www.linkedin.com/in/abhinay23/)