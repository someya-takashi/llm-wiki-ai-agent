---
title: "EfficientQAT: Efficient Quantization-Aware Training for Large Language Models"
source: "https://ar5iv.labs.arxiv.org/html/2407.11062"
author:
published:
created: 2026-08-02
description: "Large language models (LLMs) are integral to modern natural language processing and artificial intelligence. However, they face challenges in managing their significant memory requirements. Although quantization-aware …"
tags:
  - "clippings"
---
<sup>†</sup>

Mengzhao Chen <sup>1</sup>, Wenqi Shao <sup>†1</sup>, Peng Xu <sup>1,2</sup>, Jiahao Wang <sup>1,2</sup>,  
Peng Gao <sup>1</sup>, Kaipeng Zhang <sup>1</sup>, Yu Qiao <sup>1</sup>, Ping Luo <sup>†1,2</sup>  
<sup>1</sup> OpenGVLab, Shanghai AI Laboratory <sup>2</sup> The University of Hong Kong

###### Abstract

Large language models (LLMs) are integral to modern natural language processing and artificial intelligence. However, they face challenges in managing their significant memory requirements. Although quantization-aware training (QAT) offers a solution by reducing memory consumption through low-bit representations with minimal accuracy loss, it demands substantial training resources to optimize model weights and quantization parameters. To address this, we propose Efficient Quantization-Aware Training (EfficientQAT), a novel quantization technique for compressing LLMs. EfficientQAT involves two consecutive phases: Block-wise training of all parameters (Block-AP) and end-to-end training of quantization parameters (E2E-QP). Block-AP sequentially conducts quantization-aware training for all parameters in each transformer block with block-wise reconstruction, maintaining efficiency by avoiding training the entire LLM. Initialized with quantized model, E2E-QP then trains only quantization parameters (step sizes) end-to-end, enhancing efficiency with a fixed quantized backbone and reduced trainable parameter count. Extensive experiments demonstrate that EfficientQAT outperforms previous quantization methods across a range of models, including base LLMs, instruction-tuned LLMs, and multimodal LLMs, with scales from 7B to 70B parameters at various quantization bits. For instance, EfficientQAT obtains a 2-bit Llama-2-70B model on a single A100-80GB GPU in 41 hours, with less than 3% accuracy degradation compared to the full precision (69.48 vs. 72.41). Notably, this INT2 quantized 70B model obtains a 1.67 accuracy gain over the Llama-2-13B model (69.48 vs. 67.81) while requiring less memory (19.2GB vs. 24.2GB). Code is available at [https://github.com/OpenGVLab/EfficientQAT](https://github.com/OpenGVLab/EfficientQAT).

## 1 Introduction

Recent advancements in large language models (LLMs) [^57] [^6] [^12] [^63] [^66] have demonstrated impressive capabilities in diverse language tasks such as reasoning [^14] [^13] [^70], cognitive processing [^23] [^63], and agent-based applications [^48] [^49]. However, these models are characterized by their extensive parameters, which pose significant challenges for memory footprint and bandwidth [^30] [^62].

Quantization-aware training (QAT), one of the most effective quantization techniques, works by minimizing quantization errors through training with quantization constraints. Although QAT can compress LLMs effectively without significant performance loss, it requires training the whole LLM on a large corpus, resulting in enormous training costs. For instance, the QAT method BitNet b1.58 [^45] can achieve nearly lossless ternary quantization but requires retraining LLMs from scratch using the full pre-trained dataset, which is impractical for extremely large models <sup>*</sup>.

In pursuit of efficient quantization for large language models (LLMs), techniques such as post-training quantization (PTQ) and quantized parameter-efficient fine-tuning (Q-PEFT) have been developed. PTQ [^37] [^22] [^52] [^8] [^20] minimizes memory footprint during inference by converting pre-trained LLM weights from 16-bit to formats like 2-bit without retraining. While PTQ is suitable for quick deployment with minimal resources, it may compromise accuracy, especially in low-bit regimes, as shown in Figure 1(a). Q-PEFT [^16] [^64] [^35] [^25] [^29], on the other hand, uses parameter-efficient fine-tuning methods like LoRA [^27] to train quantized models, significantly reducing memory requirements during training. For instance, QLoRA [^16] converts the model from FP16 to NF4 (a 4-bit float format) and then applies a LoRA module, allowing fine-tuning on consumer-grade GPUs without sacrificing performance. However, most Q-PEFT techniques [^16] [^25] [^35] revert to FP16 for integrating additional LoRA modules post fine-tuning. This necessitates another round of PTQ for deployment on memory-limited platforms, which can degrade performance. Although solutions like PEQA [^29] and QA-LoRA [^64] attempt to address this issue, their performance remains suboptimal in low-bit scenarios (2-bit and 3-bit) due to poor quantized model initialization (Figure 1(b)).

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2407.11062/assets/x2.png)

(a) 2-bit quantization comparisons

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2407.11062/assets/x3.png)

(a) inference speed comparisons

To tackle the above challenges, we introduce a novel quantization technique named EfficientQAT, to compress LLMs effectively and efficiently. EfficientQAT surpasses existing PTQ [^52] [^20], Q-PEFT [^64] [^16] and QAT [^43] methods in performance while maintaining memory efficiency during training and inference, as illustrated in Figure 2(a) & 2(b). To improve quantization efficiency and accuracy, EfficientQAT involves two consecutive phases: Block-wise training of all parameters (Block-AP) and end-to-end training of quantization parameters (E2E-QP). Block-AP sequentially conducts quantization-aware training for all parameters including the original full-precision weights and quantization parameters (step sizes and zero points) in each transformer block with block-wise reconstruction. While Block-AP trains all parameters, it preserves memory efficiency by avoiding training the entire LLM. Moreover, potential overfitting issues in Block-AP can be mitigated by simply increasing the training samples from 128 to 4096 (see Figure 5). Notably, our fully trainable paradigm of Block-AP outperforms other trainable variants (see Table 7), such as training rounding [^46] [^11], clipping thresholds [^52], and step sizes [^21] [^18].

Building on the robust initialization provided by Block-AP, E2E-QP fixes the quantized weights and trains only the quantization parameters (step sizes) end-to-end, further enhancing the quantization performance. Moreover, E2E-QP enables instruction tuning of quantized models for specific tasks in a quantization-aware and memory-efficient manner, as trainable quantization parameters take only a small fraction of the total network (*e.g.*, 1.5% with a quantization group size of 64). It works like previous quantized parameter-efficient fine-tuning (Q-PEFT)techniques such as QA-LoRA [^64]. Therefore, EfficientQAT is general enough to compress various models including base LLMs (Llama [^57] [^56], instruction-tuned LLMs (Vicuna [^12]), and multimodal LLMs (LLaVA [^39]).

Thanks to the integration of the Block-AP and E2E-QP, EfficientQAT characterizes itself as a fast-converging, memory-efficient, and high-performing quantization technique. For instance, EfficientQAT can obtain a 2-bit Llama-2-70B model on a single A100-80GB GPU in just 41 hours, with less than 3% zero-shot accuracy degradation compared to its full-precision counterpart (69.48 vs. 72.41). We also evaluate EfficientQAT across scenarios involving model compression and instruction-tuning. In model compression, as illustrated in Figure 1(a), EfficientQAT significantly outperforms other uniform quantization methods by approximately 5 points on accuracy in the challenging 2-bit quantization setting. It also matches the performance of vector quantization methods [^20] [^58] in this scenario. Furthermore, Figure 2(a) demonstrates that the uniform quantization implemented by EfficientQAT is more hardware-efficient than vector quantization, which introduces a considerable overhead. In terms of instruction tuning, as shown in Figure 1(b), EfficientQAT consistently outperforms existing Q-PEFT methods, including QLoRA [^16], QA-LoRA [^64], and PEQA [^29].

## 2 Related Works

Post-Training Quantization of LLMs. PTQ is a pivotal technique for accelerating and deploying LLMs. Quantization approaches generally fall into two categories: weight-only quantization [^22] [^17] [^31] [^30] [^33] [^10] and weight-activation quantization [^61] [^41] [^60] [^59] [^68] [^71] [^1] [^34] [^2]. Weight-only quantization focuses on compressing weights into low-bit formats, reducing memory demands and enhancing the efficiency of memory-bounded computations in LLMs [^38] [^69]. Conversely, weight-activation quantization compresses both weights and activations, thus further decreasing the overhead associated with matrix multiplications [^38]. Recent advancements in weight-only quantization include the introduction of vector quantization methods by QUIP# [^58] and AQLM [^20]. These methods have shown promising performance but also introduce significant overhead [^24]. Our research continues to explore uniform quantization, which is preferred for its compatibility with hardware implementations.

Quantization-Aware Training of LLMs. QAT can enhance the performance of quantized models beyond what PTQ offers. However, QAT has been less explored in LLMs due to the significant training costs involved. Studies such as LLM-QAT [^43] and BitDistiller [^19] investigate the application of knowledge distillation within QAT contexts. Techniques like BitNet b1.58 [^45] and OneBit [^65] employ QAT to achieve extreme binary or ternary quantization levels. Although BitNet b1.58 demonstrates near-lossless performance on models up to 3 billion parameters and 100 billion training tokens with ternary quantization, its applicability to larger models or datasets remains uncertain due to prohibitive training expenses.

Quantized Parameter-Efficient Fine-Tuning of LLMs. Techniques like QLoRA [^16], INT2.1 [^7], LQ-LoRA [^25], and LoftQ [^35] quantize model parameters to low-bit representations followed by the addition of LoRA [^27] modules for fine-tuning. However, these methods require merging the LoRA modules into quantized weights, resulting in the model reverting to the FP16 format. Addressing this issue, QA-LoRA [^64] redesign the LoRA module to merge seamlessly into the zero points. To our knowledge, the work most closely to our approach is PEQA [^29], which employs a simple round-to-nearest (RTN) method for low-bit quantization and fine-tunes the step sizes for downstream task adaptation. Despite this, the suboptimal PTQ initialization technique utilized by PEQA leads to performance that falls short when compared to QLoRA [^16] and QA-LoRA [^64].

## 3 EfficientQAT

### 3.1 Method Overview

In this section, we introduce EfficientQAT, a novel quantization-aware training framework for LLMs that enhances memory efficiency. As illustrated in Figure 3, traditional QAT approaches train the full-precision weights $\mathbf{W}$ and quantization parameters $s$ (step sizes) and $z$ (zero points) simultaneously in an end-to-end manner, which significantly increases the memory requirements due to the large number of parameters involved. To address this issue, EfficientQAT adopts a two-stage strategy: block-wise training of all parameters (Block-AP) and end-to-end training of quantization parameters (E2E-QP). In the Block-AP phase, model parameters and quantization parameters are trained block-by-block using reconstruction loss, which not only allows for precise calibration with full training but also reduces memory consumption [^36] [^52] by block-wise training. Following this, the E2E-QP phase fixes the quantized weights and trains the step sizes exclusively on target datasets, thus achieving quantization-aware training in a memory-efficient way. Details on Block-AP and E2E-QP are further described in Sections 3.2 and 3.3, respectively.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2407.11062/assets/x5.png)

Refer to caption

### 3.2 Block-Wise Training of All Parameters

In this section, we introduce the Block-Wise Training of All Parameters (Block-AP) approach, designed to efficiently provide an effective initialization for following end-to-end training.

Quantization and Dequantization. Specifically, Block-AP begins with a standard uniform quantization method:

$$
\mathbf{W}_{int}=\mathrm{clamp}(\lfloor\frac{\mathbf{W}}{s}\rceil+z,0,2^{N}-1),
$$

where $\lfloor\cdot\rceil$ represents the rounding operation. $N$ is the target bit number. $\mathbf{W}_{int}$ and $\mathbf{W}$ denote the quantized integer and full-precision weights, respectively. $s$ is the scaling factor and $z$ is the zero point. In the forward propagation, the quantized weights are converted back to full precision as follows:

$$
\widehat{\mathbf{W}}=(\mathbf{W_{int}}-z)\cdot s.
$$

Here, $\widehat{\mathbf{W}}$ refers to the dequantized weights used in the forward computation. The processes of quantization (Eq.(1)) and dequantization (Eq.(2)) are integrated within the computation graph and can be optimized through gradient descent in a quantization-aware manner.

Blcok-wise Quantization-aware Training. Traditional QAT methods [^45] [^21] [^43] train the entire network using Eq.(1) and Eq.(2) in an end-to-end fashion, which typically requires substantial computational resources and extensive data to prevent overfitting. Here we aim to enhance the training efficiency of QAT. Previous studies, such as BRECQ [^36], have demonstrated that block-wise training achieves faster convergence and requires less training time, data, and memory than end-to-end training given a pre-trained model. Following the methodologies in BRECQ [^36] and OmniQuant [^52], Block-AP sequentially conducts quantization-aware training within one transformer block before moving on to the next under a block-wise reconstruction framework.

Full Training of Model Weights and Quantization Parameters. Unlike previous methods which optimize several quantization parameters such as rounding parameters [^46] [^11] [^32], clipping parameters [^52], and step sizes [^21] [^18], Block-AP behaves like QAT, training all inherent parameters from Eq.(1) and Eq.(2), including scaling factor $s$, zero point $z$, and model weights $\mathbf{W}$.

In our Block-AP approach, a straightforward full-training regimen outperforms existing partial-training variants [^46] [^36] [^18] with intricate designs. Traditional training methods involving rounding parameters [^46] [^36] [^18] serve as regularization techniques, constraining the update range of integral weights to $(-1,+1)$ to mitigate overfitting. However, this approach limits the solution space, potentially hindering the final performance of quantized models. By simply scaling the training samples from $128$ to $4096$, Block-AP effectively addresses the overfitting issue (see Figure 5 for details) while improving performance compared to training with rounding parameters. Our empirical findings demonstrate the superiority of full training within our Block-AP over existing partial-training variants [^46] [^36] [^18], as shown in Table 7.

Following block-wise training, we obtain the quantized model which includes quantized weights $\mathbf{W}_{q}$, step sizes $s$, and zero points $z$ for each quantization group. The weights $\mathbf{W}_{q}$ and zero points $z$ are stored in a low-bit format, while step sizes $s$ are stored in FP16. Note that $s$ and $z$ are shared within their respective quantization groups and constitute only a small fraction of the model’s parameters, approximately 1.6% for a group size of 64. Moreover, the model’s memory footprint is substantially reduced by transitioning from full-precision 16-bit weights to 2/3/4-bit quantized weights.

### 3.3 End-to-End Training of Quantization Parameters

We further introduce the End-to-End Training of Quantization Parameters (E2E-QP), aimed at efficiently training the entire quantized model on target datasets.

End-to-End Training of step sizes. Unlike traditional Quantization-Aware Training (QAT) methods [^43] [^45] that train full-precision weights, E2E-QP begins with $\mathbf{W}_{q}$ initialized via Block-AP and focuses solely on the training of quantization parameters ($s$ and $z$). Our findings indicate that training $s$, $z$, or both yields similar performance (see Table 7 for details). However, since training $z$ involves converting it from a low-bits format to full-precision, we typically train only $s$ by default unless specified otherwise to avoid additional memory overhead.

Additionally, within E2E-QP, there is no quantization process as per Equation (1); only the dequantization process occurs as described in Equation (2). Thus, the gradient of the trainable parameter $s$ is computed as $\frac{\partial\widehat{w}}{\partial s}=w_{q}-z$.

Overall, the memory usage for training in E2E-QP is drastically reduced due to the reduced trainable parameter count. Detailed memory footprints for various model sizes and bits under E2E-QP are listed in Table 8. For instance, the Llama-2-70B model can complete 2-bit QAT through E2E-QP using only 34.2GB of memory. Equipped with E2E-QP, EfficientQAT is adaptable to different scenarios by simply changing the training datasets, which includes applications such as continual pre-training and instruction-tuning [^53].

Table 1: Llama 2 & 3 average zero-shot accuracy on 5 common-sense reasoning tasks ($\uparrow$). Detailed number of each task can be found at Table 13 (4-bit), Table 14 (3-bit), and Table 15 (2-bit). “Group” indicates group size for uniform quantization and codebook scheme for vector quantization.

| Method | Bits | Type | Group (code) | 2-7 | 2-13 | 2-70 | 3-8 | 3-70 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| FP16 | 16 | \- | 16 | 64.86 | 67.81 | 72.41 | 68.58 | 75.33 |
| RTN | 4 | uniform | 128 | 64.52 | 67.50 | 72.26 | 67.79 | 73.98 |
| GPTQ | 4 | uniform | 128 | 64.24 | 67.27 | 72.39 | 67.80 | 74.74 |
| AWQ | 4 | uniform | 128 | 64.54 | 67.61 | 72.44 | 68.24 | 74.77 |
| OmniQ | 4 | uniform | 128 | 64.52 | 67.10 | 72.39 | \- | \- |
| AutoRound | 4 | uniform | 128 | 64.39 | 67.36 | 72.47 | \- | \- |
| QuIP# | 4 | vector | \- | 64.48 | 67.28 | 72.17 | \- | \- |
| EfficientQAT | 4 | uniform | 128 | 64.27 | 67.52 | 72.62 | 68.43 | 74.57 |
| RTN | 3 | uniform | 128 | 62.06 | 65.77 | 70.83 | 58.72 | 65.29 |
| GPTQ | 3 | uniform | 128 | 62.48 | 66.18 | 71.47 | 60.58 | 71.28 |
| AWQ | 3 | uniform | 128 | 62.82 | 66.14 | 71.41 | 64.82 | 73.65 |
| OmniQ | 3 | uniform | 128 | 62.42 | 66.18 | 71.07 | \- | \- |
| AutoRound | 3 | uniform | 128 | 63.72 | 66.68 | 71.24 | \- | \- |
| QuIP# | 3 | vector | \- | 63.52 | 66.26 | 72.13 | \- | \- |
| EfficientQAT | 3 | uniform | 128 | 64.02 | 67.28 | 71.76 | 67.35 | 72.42 |
| OmniQ | 2 | uniform | 128 | 46.98 | 53.56 | 54.87 | \- | \- |
| AutoRound | 2 | uniform | 128 | 54.50 | 60.72 | 67.70 | \- | \- |
| EfficientQAT | 2 | uniform | 128 | 59.50 | 63.88 | 68.93 | 59.37 | 67.57 |
| AQLM | 2 | vector | 2x8 | 57.61 | 62.22 | 69.85 | \- | \- |
| AQLM | 2 | vector | 1x16 | 61.85 | 64.95 | 70.84 | 64.10 | 70.10 |
| QuIP# | 2 | vector | \- | 60.61 | 64.44 | 70.91 | \- | \- |
| EfficientQAT | 2 | uniform | 64 | 60.14 | 64.48 | 69.48 | 60.76 | 67.89 |

## 4 Experiments

This section presents extensive experiments to verify our proposed EfficientQAT. Secition 4.1 and Sec 4.2 present the comparisons with quantization methods and Q-PEFT methods respectively. Section 4.4 details the training cost and inference speed-up of the proposed EfficientQAT. Section 4.3 presents the comprehensive ablation studies of the proposed EfficientQAT.

### 4.1 EfficientQAT for LLMs Quantization

Training. We conduct experiments on the Llama-2 and Llama-3 models. For Block-AP, we use 4096 samples from RedPajama [^15] with a context length of 2048. We train each block with batch size as 2 and epochs as 2, setting the learning rate of quantization parameters as $1\text{\times}{10}^{-4}$, and the learning rate of weights as $2\text{\times}{10}^{-5}$ for 2-bit and $1\text{\times}{10}^{-5}$ for 3/4-bits. For E2E-QP, we also employ 4096 samples from RedPajama [^15] but with a context length of 4096. We train the entire model with batch size as 32 and epoch as 1, and set the learning rate of step size as $2\text{\times}{10}^{-5}$ for 2-bit and $1\text{\times}{10}^{-5}$ for 3/4-bits.

Evaluation. We assess the zero-shot accuracy of five common-sense reasoning tasks using the v0.4.2 lm-evaluation-harness <sup>†</sup>. The tasks include WinoGrande [^50], PIQA [^5], HellaSwag [^70], Arc-Easy [^14], and Arc-Challenge [^14]. We also measure the perplexity of Wikitext2 and C4 with a 2048 context length, as done in previous studies [^22] [^52].

PTQ Baseline. We compare our results with PTQ methods from uniform quantization such as GPTQ [^22], AWQ [^37], OmniQ [^52], and AutoRound [^11], and vector quantization including QuIP# [^58] and AQLM [^20].

Table 2: Comparison with QAT methods on Llama-2 & 3 models.

<table><thead><tr><th rowspan="2">Model</th><th rowspan="2">Method</th><th rowspan="2">Bits</th><th rowspan="2">Group</th><th colspan="2">PPL</th><th>Average</th></tr><tr><th>Wiki</th><th>C4</th><th>Accuracy</th></tr></thead><tbody><tr><td>2-7B</td><td>LLM-QAT</td><td>3</td><td>128</td><td>6.02</td><td>-</td><td>-</td></tr><tr><td>2-7B</td><td>BitDistiller</td><td>3</td><td>128</td><td>5.97</td><td>-</td><td>-</td></tr><tr><td>2-7B</td><td>EfficientQAT</td><td>3</td><td>128</td><td>5.81</td><td>7.34</td><td>64.02</td></tr><tr><td>2-7B</td><td>LLM-QAT</td><td>2</td><td>128</td><td>9.30</td><td>-</td><td>-</td></tr><tr><td>2-7B</td><td>BitDistiller</td><td>2</td><td>128</td><td>8.08</td><td>-</td><td>-</td></tr><tr><td>2-7B</td><td>EfficientQAT</td><td>2</td><td>128</td><td>7.19</td><td>8.79</td><td>59.50</td></tr><tr><td>3-8B</td><td>PB-LLM</td><td>2</td><td>128</td><td>24.70</td><td>79.20</td><td>38.80</td></tr><tr><td>3-8B</td><td>DB-LLM</td><td>2</td><td>128</td><td>13.60</td><td>19.20</td><td>51.80</td></tr><tr><td>3-8B</td><td>EfficientQAT</td><td>2</td><td>128</td><td>9.80</td><td>13.22</td><td>59.37</td></tr></tbody></table>

PTQ Results. The accuracy and perplexity results are presented in Table 1 and Table 3, respectively. It seems that minimal performance degradation with 4-bit group quantization, and even simple round-to-nearest (RTN) quantization achieves respectable results. Therefore, EfficientQAT offers a slight perplexity improvement ($\sim$ 0.02) in 4-bit uniform quantization and comparable zero-shot accuracy with existing methods. As the number of quantization bits decreases, EfficientQAT’s performance advantage increases. In 3-bit quantization, EfficientQAT surpasses the optimization-based method AutoRound by approximately 0.5% in zero-shot accuracy and outperforms OmniQuant by 0.14 to 0.43 in perplexity. Notably, in 2-bit quantization, EfficientQAT outperforms AutoRound [^11] by 5% in zero-shot accuracy for Llama-2-7b. EfficientQAT also exceeds previous vector quantization methods like AQLM [^20] using a 2x8 codebook. While AQLM [^20] with a 1x16 codebook and QuIP# [^58] perform slightly better in 2-bit scenarios, they introduce notable computational overhead [^24] and can even reduce inference speed, as shown in Figure 2(a). Thus, EfficientQAT successfully pushes the limits of uniform quantization in LLMs, making hardware-friendly uniform quantization competitive with more complex vector quantization methods. However, we can also find that Llama-3 models suffer more performance degeneration after quantization [^28], which may caused by the full training with 15T tokens other than 2T tokens.

QAT Baseline. We also compare our results with existing QAT methods, including LLM-QAT [^43], BitDistiller [^19], PB-LLM [^51] and DB-LLM [^9].

QAT Results. As demonstrated in Table 2, EfficientQAT markedly surpasses other QAT methods in performance. Specifically, EfficientQAT reduces perplexity by 0.89 compared to BitDistiller [^19] when applied to the Llama-2-7B model with w2g128 quantization. In the more challenging scenario of Llama-3 quantization [^28], EfficientQAT exceeds the performance of DB-LLM [^9] by 5.98 in C4 perplexity and by 8.57 points in average accuracy with the w2g128 quantization.

Table 3: Llama 2 & 3 Wikitext2 and C4 perplexity ($\downarrow$), context length 2048.

<table><tbody><tr><th></th><th></th><th></th><th></th><td colspan="5">Wikitext 2</td><td colspan="5">C4</td></tr><tr><th>Method</th><th>Bits</th><th>Type</th><th>Group</th><td>2-7</td><td>2-13</td><td>2-70</td><td>3-8</td><td>3-70</td><td>2-7</td><td>2-13</td><td>2-70</td><td>3-8</td><td>3-70</td></tr><tr><th>FP16</th><th>16</th><th>-</th><th>16</th><td>5.47</td><td>4.88</td><td>3.32</td><td>6.14</td><td>2.85</td><td>6.97</td><td>6.47</td><td>5.52</td><td>8.88</td><td>6.73</td></tr><tr><th>GPTQ</th><th>4</th><th>uniform</th><th>128</th><td>5.61</td><td>4.98</td><td>3.42</td><td>6.53</td><td>3.38</td><td>7.13</td><td>6.56</td><td>5.58</td><td>9.35</td><td>7.02</td></tr><tr><th>AWQ</th><th>4</th><th>uniform</th><th>128</th><td>5.62</td><td>4.97</td><td>3.41</td><td>6.55</td><td>3.26</td><td>7.13</td><td>6.56</td><td>5.58</td><td>9.41</td><td>6.98</td></tr><tr><th>OmniQ</th><th>4</th><th>uniform</th><th>128</th><td>5.58</td><td>4.95</td><td>3.40</td><td>-</td><td>-</td><td>7.12</td><td>6.56</td><td>5.58</td><td>-</td><td>-</td></tr><tr><th>QuIP#</th><th>4</th><th>vector</th><th>-</th><td>5.56</td><td>4.95</td><td>3.38</td><td>-</td><td>-</td><td>7.07</td><td>6.54</td><td>5.56</td><td>-</td><td>-</td></tr><tr><th>EfficientQAT</th><th>4</th><th>uniform</th><th>128</th><td>5.53</td><td>4.93</td><td>3.39</td><td>6.47</td><td>3.17</td><td>7.07</td><td>6.54</td><td>5.58</td><td>9.26</td><td>6.94</td></tr><tr><th>GPTQ</th><th>3</th><th>uniform</th><th>128</th><td>6.29</td><td>5.42</td><td>3.85</td><td>9.58</td><td>5.25</td><td>7.89</td><td>7.00</td><td>5.85</td><td>11.66</td><td>8.64</td></tr><tr><th>AWQ</th><th>3</th><th>uniform</th><th>128</th><td>6.24</td><td>5.32</td><td>3.74</td><td>8.16</td><td>4.69</td><td>7.84</td><td>6.94</td><td>5.81</td><td>11.49</td><td>7.91</td></tr><tr><th>OmniQ</th><th>3</th><th>uniform</th><th>128</th><td>6.03</td><td>5.28</td><td>3.78</td><td>-</td><td>-</td><td>7.75</td><td>6.98</td><td>5.85</td><td>-</td><td>-</td></tr><tr><th>QuIP#</th><th>3</th><th>vector</th><th>-</th><td>5.79</td><td>5.10</td><td>3.56</td><td>-</td><td>-</td><td>7.32</td><td>6.72</td><td>5.67</td><td>-</td><td>-</td></tr><tr><th>EfficientQAT</th><th>3</th><th>uniform</th><th>128</th><td>5.81</td><td>5.12</td><td>3.61</td><td>7.09</td><td>4.19</td><td>7.34</td><td>6.73</td><td>5.71</td><td>10.06</td><td>7.43</td></tr><tr><th>OmniQ</th><th>2</th><th>uniform</th><th>128</th><td>11.06</td><td>8.26</td><td>6.11</td><td>-</td><td>-</td><td>15.02</td><td>11.05</td><td>8.52</td><td>-</td><td>-</td></tr><tr><th>EfficientQAT</th><th>2</th><th>uniform</th><th>128</th><td>7.19</td><td>6.08</td><td>4.61</td><td>9.80</td><td>6.38</td><td>8.79</td><td>7.75</td><td>6.48</td><td>13.22</td><td>9.53</td></tr><tr><th>AQLM</th><th>2</th><th>vector</th><th>2x8</th><td>7.24</td><td>6.06</td><td>4.49</td><td>-</td><td>-</td><td>8.96</td><td>7.80</td><td>6.36</td><td>-</td><td>-</td></tr><tr><th>AQLM</th><th>2</th><th>vector</th><th>1x16</th><td>6.34</td><td>5.59</td><td>4.06</td><td>7.76</td><td>5.10</td><td>8.06</td><td>7.27</td><td>6.04</td><td>10.89</td><td>8.31</td></tr><tr><th>QuIP#</th><th>2</th><th>vector</th><th>-</th><td>6.66</td><td>5.74</td><td>4.16</td><td>-</td><td>-</td><td>8.35</td><td>7.45</td><td>6.12</td><td>-</td><td>-</td></tr><tr><th>EfficientQAT</th><th>2</th><th>uniform</th><th>64</th><td>6.86</td><td>5.96</td><td>4.52</td><td>9.41</td><td>6.07</td><td>8.50</td><td>7.59</td><td>6.38</td><td>12.77</td><td>9.23</td></tr></tbody></table>

### 4.2 EfficientQAT for Instruction Tuning

Training and Evaluation. Following existing works [^64] [^47], we train Llama-1 models on the Alpaca dataset [^53] and assess their performance by measuring average 5-shot MMLU [^26] accuracy works [^64] [^47]. The training hyperparameters are identical to those described in Section 4.1, except we replace the RedPajama dataset [^15] with Alpaca. In line with QLoRA’s methodology [^16], we adjust the source context length to 384 and the target context length to 128, training for 10,000 steps with a batch size of 16.

Baseline. We benchmark EfficientQAT against several leading methods, including QLoRA [^16], QA-LoRA [^64], PEQA [^29], and IR-QLoRA [^47], across quantization setting of 2, 3, and 4 bits. Consistent with QA-LoRA [^64], we also employ GPTQ [^22] to quantize the fine-tuned QLoRA models into a low-bit format without FP16 LoRA for equitable comparison.

Results. Both Table 7 and Figure 1(b) indicate that EfficientQAT significantly outperforms existing Q-PEFT methods. For instance, in channel-wise quantization (group size of -1), EfficientQAT achieves more than 3% higher accuracy than PEQA [^29]. In the 2-bit quantization scenario, the superiority of EfficientQAT is even more pronounced, surpassing QA-LoRA [^64] by 5.1% and 4.0% in 7B and 13B models, respectively, and outperforming PEQA by 4.5% and 8.7% in the same models. Moreover, Table 7 also demonstrates that EfficientQAT outperforms both QA-LoRA and QLoRA with GPTQ in smaller model memory footprint (larger group size).

Table 4: Llama-1 average MMLU accuracy (5-shot) about instruction-tuning on Alpaca dataset.

| Method | Bits | Group | 7B | 13B |
| --- | --- | --- | --- | --- |
| \- | 16 | \- | 34.6 | 46.3 |
| PEQA | 4 | \-1 | 35.8 | 45.0 |
| EfficientQAT | 4 | \-1 | 38.8 | 48.2 |
| QLoRA | 4+16 | \- | 38.4 | 48.4 |
| QLoRA w/GPTQ | 4 | 32 | 36.0 | 48.0 |
| QA-LoRA | 4 | 32 | 39.4 | 49.2 |
| PEQA | 4 | 64 | 39.4 | 47.4 |
| IR-QLoRA | 4 | 64 | 40.8 | 49.3 |
| EfficientQAT | 4 | 64 | 41.2 | 49.5 |
| QLoRA w/ GPTQ | 3 | 32 | 34.0 | 46.1 |
| QA-LoRA | 3 | 32 | 37.4 | 47.3 |
| IR-QLoRA | 3 | 64 | 38.4 | \- |
| PEQA | 3 | 64 | 38.5 | 46.3 |
| EfficientQAT | 3 | 64 | 40.0 | 48.2 |
| QLoRA w/ GPTQ | 2 | 32 | 25.8 | 30.9 |
| QA-LoRA | 2 | 32 | 27.5 | 36.9 |
| IR-QLoRA | 2 | 64 | 27.8 | \- |
| PEQA | 2 | 64 | 28.1 | 32.2 |
| EfficientQAT | 2 | 64 | 32.6 | 40.9 |

Table 5: Effectiveness of each component on Llama-2-7B w2g64 quantization.

| Block-AP | E2E-QP | Avg. PPL | Avg. Accuracy |
| --- | --- | --- | --- |
| ✗ | ✗ | 453.49 | 40.69 |
| ✓ | ✗ | 8.53 | 58.99 |
| ✗ | ✓ | 9.33 | 55.71 |
| ✓ | ✓ | 7.68 | 60.14 |

Table 6: W2g64 Llama-2-7B performance with different trainable parameters in the block-wise training (w/o E2E-QP). “# Param.” indicates trainable parameters count in a block.

| Param. | \# Param. | Memory | Avg. PPL | Avg. Accuracy |
| --- | --- | --- | --- | --- |
| clipping | 6.3M | 6.4GB | 11.28 | 53.20 |
| $s$,$z$ | 6.3M | 6.4GB | 10.26 | 55.20 |
| round | 202.4M | 8.6GB | 15.50 | 45.32 |
| $s$,$z$,round | 208.7M | 9.3GB | 9.17 | 57.14 |
| $s$,$z$,$\mathbf{W}$ | 208.7M | 8.5GB | 8.65 | 58.94 |

Table 7: Llama-2-7B w2g64 quantization with different trainable parameters for E2E-QP (w/ Block-AP).

| Param. | Avg. Bits | Avg. PPL | Avg. Accuracy |
| --- | --- | --- | --- |
| $s$ | 2.28 | 7.68 | 60.14 |
| $z$ | 2.50 | 7.69 | 60.08 |
| $s,z$ | 2.50 | 7.68 | 60.18 |

### 4.3 Ablation Analysis

The EfficientQAT algorithm is comprised of two main components: Block-AP and E2E-QP. This section evaluates the effectiveness, trainable parameters, and training sample requirements of each component. We present the average perplexity for WikiText2 and C4 datasets, and the average accuracy for five zero-shot reasoning tasks, similar to Table 1.

Effectiveness of each component. As indicated in Table 7, both the Block-AP and E2E-QP components significantly enhance performance, with their combination yielding the best results. Notably, Block-AP outperforms E2E-QP, aligning with findings from BRECQ [^36].

Trainable parameters of Block-AP. Block-AP trains all parameters, including original weights and quantization parameters. Previous methods have introduced various training strategies to mitigate overfitting, such as trained rounding [^46] [^11], clipping thresholds [^52], and step sizes [^21] [^18]. We compare Block-AP with these methods by modifying only the trainable parameters of Block-AP. As shown in Table 7, Block-AP (training $s$, $z$, $\mathbf{W}$) performs best with an acceptable training cost. Additionally, the memory footprint of directly training $\mathbf{W}$ is even smaller than that of training the rounding operation, which requires an additional copy of rounding parameters.

Trainable parameters of E2E-QP. We further examine the trainable parameters within E2E-QP. Table 7 shows that training $s$, $z$, or both yields similar performance. However, given that converting $z$ from an original low-bit representation to a trainable FP16 format increases the average bit count, we opt to train only $s$ by default.

Samples number of Block-AP. We assess the number of training samples for Block-AP, noting that E2E-QP trains all parameters, which may lead to overfitting. To address this, we introduce an additional 64 unseen samples from ReadPajama to evaluate the overfitting issue. We adjust the training epochs to ensure a similar total training time, allowing for fair comparisons across different sample sizes. As illustrated in Figure 5, increasing the number of training samples significantly reduces the gap between training loss and validation loss from 1.07 to 0.06. This reduction corresponds to an increase in the average accuracy for zero-shot tasks from 57.14% to 58.99%. Consequently, we set the default number of training samples for E2E-QP at 4096, as this maintains a minimal gap between training and validation losses.

Samples number of E2E-QP. In the E2E-QP, we train the model for 1 epoch to avoid over-fitting. Our examination of the training sample sizes for E2E-QP, detailed in Table 5, reveals that average perplexity consistently improves as sample sizes increase from 128 to 32,674. However, there is no significant improvement in average accuracy beyond 4096 samples. Therefore, we set the training sample size for E2E-QP at 4096 by default to balance efficiency and performance. Nonetheless, it is possible to further enhance the performance of EfficientQAT by increasing the sample size.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2407.11062/assets/x6.png)

Figure 4: Illustration of training loss, validation loss and average accuracy of w2g64 Llama-2-7b with different training samples size for Block-AP (w/o E2E-QP).

Table 8: The detailed training time and training memory of EfficientQAT across different model size and quantization bits on a single A100-80GB GPU.

<table><tbody><tr><th rowspan="2">Llama-2</th><th colspan="2">Block-AP</th><th colspan="2">E2E-QP</th><td></td></tr><tr><th>Time</th><th>Memory</th><th>Time</th><th>Memory (4-/3-/2-bits)</th><th>Total Time</th></tr><tr><td>7B</td><td>3.3h</td><td>8.5GB</td><td><math><semantics><mo>∼</mo> <csymbol>similar-to</csymbol> <annotation>\sim</annotation></semantics></math> 1.5h</td><td>7.0/6.4/5.6GB</td><td>4.8h</td></tr><tr><td>13B</td><td>5.6h</td><td>10.3GB</td><td><math><semantics><mo>∼</mo> <csymbol>similar-to</csymbol> <annotation>\sim</annotation></semantics></math> 2.9h</td><td>11.7/10.6/9.1GB</td><td>8.5h</td></tr><tr><td>70B</td><td>26.6h</td><td>29.9GB</td><td><math><semantics><mo>∼</mo> <csymbol>similar-to</csymbol> <annotation>\sim</annotation></semantics></math> 14.3h</td><td>48.4/42.0/34.2GB</td><td>40.9h</td></tr></tbody></table>

Table 9: Speed of the FP16 linear layer matrix-vector multiplication in PyTorch, and relative INT2 speedups in BitBLAS [^54]. Testing on A100-80GB GPU.

<table><thead><tr><th>Llama-2</th><th colspan="2">7B</th><th colspan="2">13B</th><th colspan="2">70B</th></tr><tr><th>size (out_c <math><semantics><mo>×</mo> <annotation>\times</annotation></semantics></math> in_c)</th><th>4096x4096</th><th>11008x4096</th><th>5120x5120</th><th>13824x5120</th><th>8192x8192</th><th>28672x8192</th></tr></thead><tbody><tr><td>FP16</td><td>25 us</td><td>61 us</td><td>38 us</td><td>90 us</td><td>91 us</td><td>286 us</td></tr><tr><td>INT2</td><td>9 us</td><td>21 us</td><td>11 us</td><td>26 us</td><td>24 us</td><td>67 us</td></tr><tr><td>Speedup</td><td>3.1x</td><td>2.9x</td><td>3.6x</td><td>3.5x</td><td>3.9x</td><td>4.4x</td></tr></tbody></table>

### 4.4 Efficiency of EfficientQAT

Training Efficiency Table 8 illustrates the required memory and time for training Lllama-2 models using EfficientQAT. The results indicate that the model completes training rapidly, taking 4.8 hours for the 7B model and 40.9 hours for the 70B model. Furthermore, the adoption of a quantized backbone in the end-to-end quantization process (E2E-QP) significantly reduces memory usage due to the lower bit width. Consequently, EfficientQAT accomplishes 2-bit training for the 70B model using only 34.2GB of memory.

Inference Efficiency Weight-only quantization reduces the memory requirements of LLMs and accelerates inference, as LLMs are predominantly memory-bound [^69]. Quantization modifies the computation in the linear layers while other operations remain unchanged. To evaluate this, we calculate the relative speedup in the linear layers’ processing. According to Table 9, INT2 quantization enhances the forward-pass speed by approximately 2.9x to 4.4x. Furthermore, due to the leverage of standard uniform quantization, the quantized models of EfficientQAT can also achieve speedup through other toolbox, such as MLC-LLM [^55], AWQ [^37], and BitBLAS [^54].

## 5 Conclusion

In this study, we introduce EfficientQAT, a novel method that completes QAT with improved efficiency in both memory usage and training time. Through comprehensive testing, EfficientQAT proves superior to existing PTQ, QAT, and Q-PEFT methods in terms of versatility and performance across various models and quantization levels. Additionally, EfficientQAT leverages a standard uniform quantization, which simplifies deployment using popular toolboxes. We anticipate that EfficientQAT will stimulate further research and improve the compression of Large Language Models (LLMs), making them more efficient and widely accessible.

## References

## Appendix A Limitations

In terms of limitations, EfficientQAT demonstrates marked superiority in low-bit scenarios, such as 3-bit and 2-bit quantization. However, in the 4-bit group-wise quantization, existing PTQ methods like GPTQ [^22], AWQ [^37], and even the simplest RTN achieve comparable performance more quickly. Additionally, the Llama-3 quantization experiences significant performance degradation [^28], a phenomenon also observed in EfficientQAT. Moreover, the 2-bit performance of EfficientQAT still exhibits a small gap (3% to 4% accuracy) compared to full-precision models. Addressing these issues to achieve nearly lossless performance with INT2 quantization is a key focus of our future work.

## Appendix B Broader Impact

This paper presents work whose goal is to advance the compression and acceleration of large language models. There are many potential societal consequences of our work, none of which we feel must be specifically highlighted here.

## Appendix C Gradient of Trainable Parameters in Block-AP

Block-AP, aligned with LSQ+ [^4], uses a straight-through estimator (STE) [^3] to facilitate gradient computation through the rounding operation. The gradients of scaling factor $s$ are computed as follows:

$$
\frac{\partial\widehat{w}}{\partial s}=\left\{\begin{aligned} &\lfloor\frac{w}{s}\rceil-\frac{w}{s},0\leq\lfloor\frac{w}{s}\rceil+z\leq 2^{N-1},\\
&-z,\lfloor\frac{w}{s}\rceil+z<0,\\
&2^{N-1}-z,\lfloor\frac{w}{s}\rceil+z>2^{N-1}.\end{aligned}\right.
$$

and the gradient with respect to zero point $z$ is:

$$
\frac{\partial\widehat{w}}{\partial z}=\left\{\begin{aligned} &0,0\leq\lfloor\frac{w}{s}\rceil+z\leq 2^{N-1},\\
&-1,otherwise,\\
\end{aligned}\right.
$$

and the full-precision weight $\mathbf{W}$ can also be updated through its gradient <sup>‡</sup>:

$$
\frac{\partial\widehat{w}}{\partial w}=\left\{\begin{aligned} &1,0\leq\lfloor\frac{w}{s}\rceil+z\leq 2^{N-1},\\
&0,otherwise,\\
\end{aligned}\right.
$$

## Appendix D Results Source of Other Method.

In this study, we present a thorough comparison of our method against existing PTQ techniques, including GPTQ [^22], AWQ [^37], OmniQ [^52], AutoRound [^11], QuIP# [^58], and AQLM [^20]. We also compare with existing QAT methods, including LLM-QAT [^43], BitDistiller [^19], PB-LLM [^51] and DB-LLM [^9]. Additionally, we also evaluate quantized parameter-efficient fine-tuning methods such as PEQA [^29], QLoRA [^16], QA-LoRA [^64], and IR-QLoRA [^47]. The results we discuss originate from their respective official publications, and other scholarly articles, or are derived from our reproduction. We meticulously document the source of the results for each method as follows:

- GPTQ, AWQ, OmniQ, AutoRound: The zero-shot accuracy results for Llama-2 models using these methods are derived from the AutoRound GitHub repository <sup>§</sup>. The perplexity results for the Llama-2 models using GPTQ, AWQ, and OmniQ are taken from the OmniQ paper [^52]. The results for Llama-3 models using AWQ <sup>¶</sup> and GPTQ <sup>∥</sup> were obtained through their open-source implementations.
- QuIP#, AQLM: We replicated the results using the official pre-trained models provided by QuIP# <sup>**</sup> and AQLM <sup>††</sup>.
- LLM-QAT, BitDistiller: These results are cited from BitDistiller [^19] paper.
- PB-LLM, DB-LLM: These results are cited from recent Llama-3 quantization empirical study [^28].
- PEQA: The per-channel quantization results (g=-1) are cited from their publication [^29], and the results for a group size of 64 were produced using our codebase.
- QA-LoRA, QLoRA, QLoRA w/ GPTQ: These results are cited from QA-LoRA [^64] paper.
- IR-QLoRA: These results are cited from IR-QLoRA [^47] paper.

Table 10: Model size of quantized models. Compression ratio indicates the compression ratio of quantized models compared with FP16 models.

| Model | \# Bit | Group size | bits/param | size (GiB) | Compression ratio (%) |
| --- | --- | --- | --- | --- | --- |
| LLaMA-2-7B | 16 | \- | 16 | 12.55 | \- |
| 2-6 | 4 | 32 | 4.63 | 3.98 | 68.33 |
|  | 4 | 64 | 4.31 | 3.74 | 70.20 |
|  | 4 | 128 | 4.16 | 3.62 | 71.14 |
| 2-6 | 3 | 32 | 3.59 | 3.35 | 73.28 |
|  | 3 | 64 | 3.30 | 3.13 | 75.08 |
|  | 3 | 128 | 3.15 | 3.01 | 75.98 |
| 2-6 | 2 | 32 | 2.56 | 2.42 | 80.71 |
|  | 2 | 64 | 2.28 | 2.21 | 82.40 |
|  | 2 | 128 | 2.14 | 2.10 | 83.25 |
| LLaMA-2-13B | 16 | \- | 16 | 24.24 | \- |
| 2-6 | 4 | 32 | 4.63 | 7.44 | 69.30 |
|  | 4 | 64 | 4.31 | 6.98 | 71.21 |
|  | 4 | 128 | 4.16 | 6.75 | 72.16 |
| 2-6 | 3 | 32 | 3.59 | 6.22 | 74.33 |
|  | 3 | 64 | 3.30 | 5.78 | 76.16 |
|  | 3 | 128 | 3.15 | 5.56 | 77.07 |
| 2-6 | 2 | 32 | 2.56 | 4.40 | 81.87 |
|  | 2 | 64 | 2.28 | 3.98 | 83.58 |
|  | 2 | 128 | 2.14 | 3.77 | 84.44 |
| LLaMA-2-70B | 16 | \- | 16 | 128.48 | \- |
| 2-6 | 4 | 32 | 4.63 | 37.83 | 70.55 |
|  | 4 | 64 | 4.31 | 35.34 | 72.49 |
|  | 4 | 128 | 4.16 | 34.10 | 73.46 |
| 2-6 | 3 | 32 | 3.59 | 31.26 | 75.67 |
|  | 3 | 64 | 3.30 | 28.87 | 77.53 |
|  | 3 | 128 | 3.15 | 27.67 | 78.46 |
| 2-6 | 2 | 32 | 2.56 | 21.40 | 83.34 |
|  | 2 | 64 | 2.28 | 19.16 | 85.09 |
|  | 2 | 128 | 2.14 | 18.04 | 85.96 |

## Appendix E Size of Quantized Models

This section illustrates model size reduction achieved through quantization. Models quantized to low-bit representations are more compact.

We implement N-bit quantization with a grouping size of $g$, where each group of $g$ weights shares the same FP16 step size and an N-bit zero point. Consequently, the average number of bits per parameter is calculated as $N+\frac{N+16}{g}$. It is important to note that only the linear layers within the transformer blocks are quantized; other layers, such as normalization layers, embeddings, and the classification head, remain in FP16 format. Table 10 provides detailed comparisons of quantized model sizes and their compression ratios.

Table 11: Lllma-2-7B 2-bit quantization performance with different group sizes for proposed EfficientQAT.

| Group | Avg. Bits | Avg. PPL | Avg. Accuracy |
| --- | --- | --- | --- |
| 32 | 2.56 | 7.59 | 60.28 |
| 64 | 2.28 | 7.68 | 60.14 |
| 128 | 2.10 | 7.99 | 59.50 |
| 256 | 2.07 | 8.18 | 58.67 |

## Appendix F Additional Ablation Analysis

Quantization Group Size. The group size is a crucial hyperparameter in weight-only quantization. A smaller group size offers more granular compression and reduces quantization loss but increases the number of quantization parameters required. As indicated in Table 11, a group size of 64 strikes an optimal balance for 2-bit quantization using EfficientQAT. It outperforms a group size of 128 by achieving a 0.31 lower perplexity and a 0.64% higher accuracy, yet it slightly underperforms compared to a group size of 32, with a marginal difference of 0.09 in perplexity and 0.14% in accuracy.

Table 12: Results about instruction tuning of large vision-language models. We following the overall training pipeling of LLaVA-1.5 [^39] and just change the fine-tuning methods. ‘QLoRA + Block-AP’ indicates that we leverage proposed Block-AP to quantized the QLoRA models into low-bits for fair comparisons. <sup>†</sup> MME’s perception scores are normalized to 100 percent.

<table><thead><tr><th rowspan="2">Model</th><th rowspan="2">Method</th><th colspan="2">#Bit</th><th rowspan="2">MMbench</th><th rowspan="2">MME <sup>†</sup></th><th rowspan="2">MM-Vet</th><th rowspan="2">ScienceQA</th><th rowspan="2">Avg.</th></tr><tr><th>Training</th><th>Inference</th></tr></thead><tbody><tr><td rowspan="2">LLaVA-1.5-7B</td><td>LoRA</td><td>16</td><td>16</td><td>66.1</td><td>73.8</td><td>30.2</td><td>68.4</td><td>59.6</td></tr><tr><td>QLoRA</td><td>4+16</td><td>16</td><td>64.1</td><td>72.8</td><td>30.3</td><td>68.0</td><td>58.8</td></tr><tr><td>2-9</td><td>QLoRA + Block-AP</td><td>4+16</td><td>4</td><td>63.6</td><td>72.0</td><td>29.8</td><td>67.7</td><td>58.3</td></tr><tr><td></td><td>EfficientQAT</td><td>4</td><td>4</td><td>64.4</td><td>73.2</td><td>30.3</td><td>68.1</td><td>58.8(+0.5)</td></tr><tr><td></td><td>QLoRA + Block-AP</td><td>4+16</td><td>3</td><td>62.9</td><td>71.8</td><td>29.7</td><td>66.4</td><td>57.7</td></tr><tr><td></td><td>EfficientQAT</td><td>3</td><td>3</td><td>63.2</td><td>71.4</td><td>30.9</td><td>67.3</td><td>58.2(+0.5)</td></tr><tr><td></td><td>QLoRA + Block-AP</td><td>4+16</td><td>2</td><td>53.7</td><td>64.3</td><td>28.9</td><td>60.7</td><td>51.9</td></tr><tr><td></td><td>EfficientQAT</td><td>2</td><td>2</td><td>62.3</td><td>68.0</td><td>27.8</td><td>63.4</td><td>55.4(+3.5)</td></tr><tr><td rowspan="2">LLaVA-1.5-13B</td><td>LoRA</td><td>16</td><td>16</td><td>68.5</td><td>77.1</td><td>38.3</td><td>71.2</td><td>63.8</td></tr><tr><td>QLoRA</td><td>4+16</td><td>16</td><td>67.6</td><td>76.9</td><td>36.0</td><td>69.9</td><td>62.7</td></tr><tr><td>2-9</td><td>QLoRA + Block-AP</td><td>4+16</td><td>4</td><td>67.4</td><td>76.6</td><td>35.6</td><td>69.3</td><td>62.4</td></tr><tr><td></td><td>EfficientQAT</td><td>4</td><td>4</td><td>67.5</td><td>74.8</td><td>35.6</td><td>70.2</td><td>62.0(-0.4)</td></tr><tr><td></td><td>QLoRA + Block-AP</td><td>4+16</td><td>3</td><td>66.8</td><td>75.5</td><td>34.5</td><td>68.4</td><td>61.3</td></tr><tr><td></td><td>EfficientQAT</td><td>3</td><td>3</td><td>67.4</td><td>74.8</td><td>35.3</td><td>69.3</td><td>61.7(+0.4)</td></tr><tr><td></td><td>QLoRA + Block-AP</td><td>4+16</td><td>2</td><td>62.5</td><td>72.1</td><td>32.5</td><td>65.0</td><td>58.0</td></tr><tr><td></td><td>EfficientQAT</td><td>2</td><td>2</td><td>63.9</td><td>73.1</td><td>33.9</td><td>68.6</td><td>59.9(+1.9)</td></tr></tbody></table>

## Appendix G Instruction Tuning for LVLMs.

Traditional Q-PEFT methods only do experiments on the language models. In this section, we further extend proposed EfficientQAT into Large vision-Language models (LVLMs) such as LLaVA [^40].

Training and Evaluation. For the fine-tuning of large vision-language models (LVLMs), we largely align with LLaVA1.5 [^39], which encompass the training model, datasets, and hyperparameters <sup>‡‡</sup>. Unlike LLaVA1.5, which begins fine-tuning with full-precision Vicuna models using either full fine-tuning or LoRA-based methods [^27], EfficientQAT starts with Vicuna models already quantized using our Block-AP method and continues with our E2E-QP fine-tuning approach. The training process involves two steps: initially freezing the LLM and pre-training a projector to align features with a Vision Transformer (ViT), followed by end-to-end fine-tuning of both the LLM and the projector. For EfficientQAT, we modify the learning rates in the second step to $2\times 10^{-5}$ for 4-bit and $3\times 10^{-5}$ for 2-bit and 3-bit.

Evaluation. Evaluation of the fine-tuned LVLMs are conducted across four benchmarks: MME [^23], MM-Vet [^67], MMBench [^42], and ScienceQA [^44].

Baseline. We compare our results with those of QLoRA [^16], applying our Block-AP method to quantize the QLoRA fine-tuned models to low bits for fair comparison.

Results. As shown in Table 12, EfficientQAT outperforms QLoRA [^16] in low-bit settings for both LLaVA-1.5-7B and LLaVA-1.5-13B models, consistent with previous results in LMMs. Remarkably, the 2-bit LLaVA-1.5-13B model trained with EfficientQAT achieves an average score of 59.9, surpassing the 59.6 of the FP16 LLaVA-1.5-7B model trained with LoRA. However, there is a slight performance decrease observed in the 4-bit EfficientQAT and 16-bit QLoRA compared to the 16-bit LoRA, indicating that further research is needed to optimize Q-PEFT within LVLMs.

## Appendix H Full Results

In Table 1, we present the average accuracy for five zero-shot tasks. This section offers a detailed breakdown of the task-specific accuracy numbers. Specifically, Tables 13, 14, and 15 detail the performance of 4-bit, 3-bit, and 2-bit quantization, respectively.

Table 13: 4-bit Llama 2 & 3 zero-shot accuracy by lm\_eval v0.4.2 ( acc is reported, not acc\_norm )

<table><tbody><tr><td>Model</td><td>Method</td><td>Bits</td><td>Group</td><td>WinoGrande</td><td>HellaSwag</td><td>ArcC</td><td>ArcE</td><td>PiQA</td><td>Average accuracy <math><semantics><mo>↑</mo> <ci>↑</ci> <annotation>\uparrow</annotation></semantics></math></td></tr><tr><td rowspan="8">2-7B</td><td>-</td><td>-</td><td>16</td><td>69.22</td><td>57.16</td><td>43.52</td><td>76.26</td><td>78.07</td><td>64.85</td></tr><tr><td>RTN</td><td>4</td><td>128</td><td>68.35</td><td>56.91</td><td>43.52</td><td>76.26</td><td>77.58</td><td>64.52</td></tr><tr><td>GPTQ</td><td>4</td><td>128</td><td>69.06</td><td>56.36</td><td>42.15</td><td>75.63</td><td>78.02</td><td>64.24</td></tr><tr><td>AWQ</td><td>4</td><td>128</td><td>68.98</td><td>56.40</td><td>43.86</td><td>76.14</td><td>77.31</td><td>64.54</td></tr><tr><td>OmniQ</td><td>4</td><td>128</td><td>68.98</td><td>56.59</td><td>43.34</td><td>75.76</td><td>77.91</td><td>64.52</td></tr><tr><td>AutoRound</td><td>4</td><td>128</td><td>68.67</td><td>56.79</td><td>42.58</td><td>75.76</td><td>78.13</td><td>64.39</td></tr><tr><td>QuIP#</td><td>4</td><td>-</td><td>69.22</td><td>56.65</td><td>43.00</td><td>75.51</td><td>78.02</td><td>64.48</td></tr><tr><td>EfficientQAT</td><td>4</td><td>128</td><td>69.22</td><td>57.00</td><td>42.32</td><td>75.13</td><td>77.69</td><td>64.27</td></tr><tr><td rowspan="8">2-13B</td><td>-</td><td>16</td><td>-</td><td>72.22</td><td>60.07</td><td>48.29</td><td>79.42</td><td>79.05</td><td>67.81</td></tr><tr><td>RTN</td><td>4</td><td>128</td><td>72.14</td><td>59.77</td><td>47.95</td><td>79.00</td><td>78.62</td><td>67.50</td></tr><tr><td>GPTQ</td><td>4</td><td>128</td><td>72.14</td><td>59.76</td><td>47.53</td><td>78.58</td><td>78.35</td><td>67.27</td></tr><tr><td>AWQ</td><td>4</td><td>128</td><td>73.32</td><td>59.80</td><td>46.50</td><td>79.38</td><td>79.05</td><td>67.61</td></tr><tr><td>OmniQ</td><td>4</td><td>128</td><td>72.06</td><td>59.53</td><td>47.18</td><td>78.37</td><td>78.35</td><td>67.10</td></tr><tr><td>AutoRound</td><td>4</td><td>128</td><td>71.67</td><td>59.87</td><td>47.01</td><td>79.25</td><td>79.00</td><td>67.36</td></tr><tr><td>QuIP#</td><td>4</td><td>-</td><td>72.69</td><td>59.49</td><td>46.59</td><td>78.62</td><td>79.00</td><td>67.28</td></tr><tr><td>EfficientQAT</td><td>4</td><td>128</td><td>71.98</td><td>59.87</td><td>47.53</td><td>79.34</td><td>78.89</td><td>67.52</td></tr><tr><td rowspan="8">2-70B</td><td>-</td><td>16</td><td>-</td><td>77.98</td><td>64.77</td><td>54.44</td><td>82.70</td><td>82.15</td><td>72.41</td></tr><tr><td>RTN</td><td>4</td><td>128</td><td>78.14</td><td>63.93</td><td>54.78</td><td>82.79</td><td>81.66</td><td>72.26</td></tr><tr><td>GPTQ</td><td>4</td><td>128</td><td>78.22</td><td>64.45</td><td>54.61</td><td>82.79</td><td>81.88</td><td>72.39</td></tr><tr><td>AWQ</td><td>4</td><td>128</td><td>77.58</td><td>64.48</td><td>55.12</td><td>82.70</td><td>82.32</td><td>72.44</td></tr><tr><td>OmniQ</td><td>4</td><td>128</td><td>77.51</td><td>64.52</td><td>55.12</td><td>82.91</td><td>81.88</td><td>72.39</td></tr><tr><td>AutoRound</td><td>4</td><td>128</td><td>78.30</td><td>64.60</td><td>54.52</td><td>82.87</td><td>82.05</td><td>72.47</td></tr><tr><td>QuIP#</td><td>4</td><td>-</td><td>77.82</td><td>64.51</td><td>54.44</td><td>82.37</td><td>81.72</td><td>72.17</td></tr><tr><td>EfficientQAT</td><td>4</td><td>128</td><td>78.45</td><td>64.57</td><td>55.12</td><td>83.00</td><td>81.94</td><td>72.62</td></tr><tr><td rowspan="5">3-8B</td><td>-</td><td>-</td><td>16</td><td>72.61</td><td>60.17</td><td>50.43</td><td>80.09</td><td>79.60</td><td>68.58</td></tr><tr><td>RTN</td><td>4</td><td>128</td><td>73.16</td><td>59.04</td><td>48.38</td><td>79.25</td><td>79.11</td><td>67.79</td></tr><tr><td>GPTQ</td><td>4</td><td>128</td><td>73.72</td><td>59.17</td><td>47.78</td><td>79.38</td><td>78.94</td><td>67.80</td></tr><tr><td>AWQ</td><td>4</td><td>128</td><td>73.01</td><td>59.43</td><td>50.00</td><td>79.55</td><td>79.22</td><td>68.24</td></tr><tr><td>EfficientQAT</td><td>4</td><td>128</td><td>72.53</td><td>59.43</td><td>50.94</td><td>79.84</td><td>79.43</td><td>68.43</td></tr><tr><td rowspan="5">3-70B</td><td>-</td><td>16</td><td></td><td>80.51</td><td>66.36</td><td>60.41</td><td>86.99</td><td>82.37</td><td>75.33</td></tr><tr><td>RTN</td><td>4</td><td>128</td><td>78.77</td><td>65.83</td><td>57.34</td><td>85.69</td><td>82.26</td><td>73.98</td></tr><tr><td>GPTQ</td><td>4</td><td>128</td><td>80.51</td><td>66.12</td><td>59.04</td><td>85.77</td><td>82.26</td><td>74.74</td></tr><tr><td>AWQ</td><td>4</td><td>128</td><td>80.35</td><td>65.82</td><td>59.13</td><td>86.41</td><td>82.15</td><td>74.77</td></tr><tr><td>EfficientQAT</td><td>4</td><td>128</td><td>79.24</td><td>66.27</td><td>59.13</td><td>85.86</td><td>82.37</td><td>74.57</td></tr></tbody></table>

Table 14: 3-bit Llama 2 & 3 zero-shot accuracy by lm\_eval v0.4.2 ( acc is reported, not acc\_norm )

<table><tbody><tr><td>Model</td><td>Method</td><td>Bits</td><td>Group</td><td>WinoGrande</td><td>HellaSwag</td><td>ArcC</td><td>ArcE</td><td>PiQA</td><td>Average accuracy <math><semantics><mo>↑</mo> <ci>↑</ci> <annotation>\uparrow</annotation></semantics></math></td></tr><tr><td rowspan="8">2-7B</td><td>-</td><td>-</td><td>16</td><td>69.22</td><td>57.16</td><td>43.52</td><td>76.26</td><td>78.07</td><td>64.85</td></tr><tr><td>RTN</td><td>3</td><td>128</td><td>67.56</td><td>54.90</td><td>38.57</td><td>72.98</td><td>76.28</td><td>62.06</td></tr><tr><td>GPTQ</td><td>3</td><td>128</td><td>68.59</td><td>53.66</td><td>40.19</td><td>73.74</td><td>76.01</td><td>62.44</td></tr><tr><td>AWQ</td><td>3</td><td>128</td><td>67.40</td><td>54.98</td><td>41.64</td><td>74.07</td><td>76.01</td><td>62.82</td></tr><tr><td>OmniQ</td><td>3</td><td>128</td><td>66.69</td><td>54.42</td><td>39.85</td><td>74.37</td><td>76.77</td><td>62.42</td></tr><tr><td>AutoRound</td><td>3</td><td>128</td><td>68.27</td><td>55.33</td><td>42.92</td><td>75.25</td><td>76.82</td><td>63.72</td></tr><tr><td>QuIP#</td><td>3</td><td>-</td><td>68.19</td><td>55.85</td><td>41.89</td><td>74.62</td><td>77.04</td><td>63.52</td></tr><tr><td>EfficientQAT</td><td>3</td><td>128</td><td>69.14</td><td>55.90</td><td>42.83</td><td>74.66</td><td>77.58</td><td>64.02</td></tr><tr><td rowspan="8">2-13B</td><td>-</td><td>16</td><td>-</td><td>72.22</td><td>60.07</td><td>48.29</td><td>79.42</td><td>79.05</td><td>67.81</td></tr><tr><td>RTN</td><td>3</td><td>128</td><td>70.72</td><td>57.74</td><td>44.62</td><td>77.69</td><td>78.07</td><td>65.77</td></tr><tr><td>GPTQ</td><td>3</td><td>128</td><td>70.88</td><td>57.83</td><td>45.65</td><td>77.99</td><td>78.56</td><td>66.18</td></tr><tr><td>AWQ</td><td>3</td><td>128</td><td>71.82</td><td>58.58</td><td>44.62</td><td>77.95</td><td>77.75</td><td>66.14</td></tr><tr><td>OmniQ</td><td>3</td><td>128</td><td>70.01</td><td>58.46</td><td>46.16</td><td>77.86</td><td>78.40</td><td>66.18</td></tr><tr><td>AutoRound</td><td>3</td><td>128</td><td>71.59</td><td>59.11</td><td>45.82</td><td>78.58</td><td>78.29</td><td>66.68</td></tr><tr><td>QuIP#</td><td>-</td><td>3</td><td>72.45</td><td>58.26</td><td>44.62</td><td>77.90</td><td>78.07</td><td>66.26</td></tr><tr><td>EfficientQAT</td><td>3</td><td>128</td><td>72.06</td><td>59.01</td><td>47.95</td><td>79.00</td><td>78.40</td><td>67.28</td></tr><tr><td rowspan="8">2-70B</td><td>-</td><td>16</td><td>-</td><td>77.98</td><td>64.77</td><td>54.44</td><td>82.70</td><td>82.15</td><td>72.41</td></tr><tr><td>RTN</td><td>3</td><td>128</td><td>77.90</td><td>61.98</td><td>52.39</td><td>81.10</td><td>80.79</td><td>70.83</td></tr><tr><td>GPTQ</td><td>3</td><td>128</td><td>77.66</td><td>62.94</td><td>53.67</td><td>81.65</td><td>81.45</td><td>71.47</td></tr><tr><td>AWQ</td><td>3</td><td>128</td><td>76.48</td><td>63.75</td><td>53.67</td><td>81.40</td><td>81.77</td><td>71.41</td></tr><tr><td>OmniQ</td><td>3</td><td>128</td><td>76.48</td><td>63.54</td><td>52.82</td><td>81.02</td><td>81.50</td><td>71.07</td></tr><tr><td>AutoRound</td><td>3</td><td>128</td><td>76.56</td><td>63.83</td><td>52.56</td><td>81.73</td><td>81.50</td><td>71.24</td></tr><tr><td>QuIP#</td><td>3</td><td>-</td><td>76.24</td><td>64.22</td><td>55.89</td><td>82.11</td><td>82.21</td><td>72.13</td></tr><tr><td>EfficientQAT</td><td>3</td><td>128</td><td>77.27</td><td>64.20</td><td>53.75</td><td>81.73</td><td>81.83</td><td>71.76</td></tr><tr><td rowspan="5">3-8B</td><td>-</td><td>-</td><td>16</td><td>72.61</td><td>60.17</td><td>50.43</td><td>80.09</td><td>79.60</td><td>68.58</td></tr><tr><td>RTN</td><td>3</td><td>128</td><td>66.54</td><td>50.87</td><td>36.69</td><td>65.36</td><td>74.16</td><td>58.72</td></tr><tr><td>GPTQ</td><td>3</td><td>128</td><td>70.88</td><td>55.13</td><td>37.80</td><td>65.24</td><td>73.83</td><td>60.58</td></tr><tr><td>AWQ</td><td>3</td><td>128</td><td>70.96</td><td>55.43</td><td>44.20</td><td>75.84</td><td>77.69</td><td>64.82</td></tr><tr><td>EfficientQAT</td><td>3</td><td>128</td><td>71.51</td><td>57.81</td><td>48.81</td><td>80.01</td><td>78.63</td><td>67.35</td></tr><tr><td rowspan="5">3-70B</td><td>-</td><td>16</td><td></td><td>80.51</td><td>66.36</td><td>60.41</td><td>86.99</td><td>82.37</td><td>75.33</td></tr><tr><td>RTN</td><td>3</td><td>128</td><td>65.90</td><td>54.22</td><td>48.46</td><td>78.83</td><td>79.05</td><td>65.29</td></tr><tr><td>GPTQ</td><td>3</td><td>128</td><td>78.14</td><td>62.58</td><td>52.99</td><td>82.07</td><td>80.63</td><td>71.28</td></tr><tr><td>AWQ</td><td>3</td><td>128</td><td>78.85</td><td>64.26</td><td>58.36</td><td>84.51</td><td>82.26</td><td>73.65</td></tr><tr><td>EfficientQAT</td><td>3</td><td>128</td><td>77.82</td><td>65.53</td><td>55.12</td><td>83.12</td><td>80.52</td><td>72.42</td></tr></tbody></table>

Table 15: 2-bit Llama 2 & 3 zero-shot accuracy by lm\_eval v0.4.2 ( acc is reported, not acc\_norm )

<table><tbody><tr><td>Model</td><td>Method</td><td>Bits</td><td>Group</td><td>WinoGrande</td><td>HellaSwag</td><td>ArcC</td><td>ArcE</td><td>PiQA</td><td>Average accuracy <math><semantics><mo>↑</mo> <ci>↑</ci> <annotation>\uparrow</annotation></semantics></math></td></tr><tr><td rowspan="9">2-7B</td><td>-</td><td>-</td><td>16</td><td>69.22</td><td>57.16</td><td>43.52</td><td>76.26</td><td>78.07</td><td>64.85</td></tr><tr><td>GPTQ</td><td>2</td><td>128</td><td>55.17</td><td>32.59</td><td>21.25</td><td>40.45</td><td>58.32</td><td>41.56</td></tr><tr><td>OmniQ</td><td>2</td><td>128</td><td>55.88</td><td>40.28</td><td>23.46</td><td>50.13</td><td>65.13</td><td>46.98</td></tr><tr><td>AutoRound</td><td>2</td><td>128</td><td>61.01</td><td>40.28</td><td>32.25</td><td>65.99</td><td>72.96</td><td>54.50</td></tr><tr><td>AQLM</td><td>2</td><td>2x8</td><td>65.27</td><td>49.96</td><td>32.85</td><td>66.92</td><td>73.07</td><td>57.61</td></tr><tr><td>AQLM</td><td>2</td><td>1x16</td><td>65.19</td><td>53.42</td><td>39.68</td><td>74.07</td><td>76.88</td><td>61.85</td></tr><tr><td>QuIP#</td><td>2</td><td>-</td><td>65.67</td><td>52.19</td><td>37.88</td><td>71.84</td><td>75.46</td><td>60.61</td></tr><tr><td>EfficientQAT</td><td>2</td><td>128</td><td>66.22</td><td>50.84</td><td>36.52</td><td>69.78</td><td>74.16</td><td>59.50</td></tr><tr><td>EfficientQAT</td><td>2</td><td>64</td><td>65.98</td><td>51.58</td><td>36.86</td><td>70.96</td><td>75.30</td><td>60.14</td></tr><tr><td rowspan="9">2-13B</td><td>-</td><td>16</td><td>-</td><td>72.22</td><td>60.07</td><td>48.29</td><td>79.42</td><td>79.05</td><td>67.81</td></tr><tr><td>GPTQ</td><td>2</td><td>128</td><td>55.80</td><td>41.06</td><td>21.93</td><td>55.60</td><td>67.08</td><td>48.29</td></tr><tr><td>OmniQ</td><td>2</td><td>128</td><td>57.93</td><td>46.23</td><td>30.29</td><td>63.22</td><td>70.13</td><td>53.56</td></tr><tr><td>AutoRound</td><td>2</td><td>128</td><td>64.33</td><td>53.35</td><td>38.57</td><td>71.17</td><td>76.17</td><td>60.72</td></tr><tr><td>AQLM</td><td>2</td><td>2x8</td><td>66.22</td><td>54.62</td><td>40.10</td><td>73.06</td><td>77.09</td><td>62.22</td></tr><tr><td>AQLM</td><td>2</td><td>1x16</td><td>70.09</td><td>57.62</td><td>43.52</td><td>75.25</td><td>78.29</td><td>64.95</td></tr><tr><td>QuIP#</td><td>2</td><td>-</td><td>69.06</td><td>56.53</td><td>42.92</td><td>75.72</td><td>77.97</td><td>64.44</td></tr><tr><td>EfficientQAT</td><td>2</td><td>128</td><td>68.90</td><td>55.66</td><td>42.83</td><td>75.04</td><td>76.99</td><td>63.88</td></tr><tr><td>EfficientQAT</td><td>2</td><td>64</td><td>68.36</td><td>55.27</td><td>41.89</td><td>74.83</td><td>77.04</td><td>63.48</td></tr><tr><td rowspan="9">2-70B</td><td>-</td><td>16</td><td>-</td><td>77.98</td><td>64.77</td><td>54.44</td><td>82.70</td><td>82.15</td><td>72.41</td></tr><tr><td>GPTQ</td><td>2</td><td>128</td><td>49.57</td><td>25.04</td><td>22.70</td><td>25.08</td><td>49.51</td><td>34.38</td></tr><tr><td>OmniQ</td><td>2</td><td>128</td><td>64.33</td><td>35.45</td><td>33.28</td><td>67.21</td><td>74.10</td><td>54.87</td></tr><tr><td>AutoRound</td><td>2</td><td>128</td><td>74.90</td><td>59.65</td><td>46.59</td><td>78.37</td><td>79.00</td><td>67.70</td></tr><tr><td>AQLM</td><td>2</td><td>2x8</td><td>75.61</td><td>61.94</td><td>51.45</td><td>79.76</td><td>80.47</td><td>69.85</td></tr><tr><td>AQLM</td><td>2</td><td>1x16</td><td>76.01</td><td>62.78</td><td>52.99</td><td>81.36</td><td>81.07</td><td>70.84</td></tr><tr><td>QuIP#</td><td>2</td><td>-</td><td>75.77</td><td>62.86</td><td>52.65</td><td>81.90</td><td>81.39</td><td>70.91</td></tr><tr><td>EfficientQAT</td><td>2</td><td>128</td><td>73.64</td><td>61.58</td><td>49.23</td><td>80.01</td><td>80.20</td><td>68.93</td></tr><tr><td>EfficientQAT</td><td>2</td><td>64</td><td>74.59</td><td>61.78</td><td>50.77</td><td>80.13</td><td>80.14</td><td>69.48</td></tr><tr><td rowspan="3">3-8B</td><td>-</td><td>-</td><td>16</td><td>72.61</td><td>60.17</td><td>50.43</td><td>80.09</td><td>79.60</td><td>68.58</td></tr><tr><td>AQLM</td><td>2</td><td>1x16</td><td>71.82</td><td>55.44</td><td>41.21</td><td>74.24</td><td>77.80</td><td>64.10</td></tr><tr><td>EfficientQAT</td><td>2</td><td>128</td><td>65.67</td><td>50.74</td><td>36.01</td><td>69.15</td><td>75.30</td><td>59.37</td></tr><tr><td></td><td>EfficientQAT</td><td>2</td><td>64</td><td>67.72</td><td>51.86</td><td>37.03</td><td>71.17</td><td>76.03</td><td>60.76</td></tr><tr><td rowspan="3">3-70B</td><td>-</td><td>16</td><td></td><td>80.51</td><td>66.36</td><td>60.41</td><td>86.99</td><td>82.37</td><td>75.33</td></tr><tr><td>AQLM</td><td>2</td><td>1x16</td><td>78.22</td><td>63.47</td><td>50.34</td><td>78.83</td><td>79.65</td><td>70.10</td></tr><tr><td>EfficientQAT</td><td>2</td><td>128</td><td>69.46</td><td>60.75</td><td>48.81</td><td>79.25</td><td>79.60</td><td>67.57</td></tr><tr><td></td><td>EfficientQAT</td><td>2</td><td>64</td><td>74.03</td><td>61.60</td><td>49.06</td><td>77.40</td><td>77.37</td><td>67.89</td></tr></tbody></table>

[^1]: Saleh Ashkboos, Ilia Markov, Elias Frantar, Tingxuan Zhong, Xincheng Wang, Jie Ren, Torsten Hoefler, and Dan Alistarh. Towards end-to-end 4-bit inference on generative large language models. arXiv preprint arXiv:2310.09259, 2023.

[^2]: Saleh Ashkboos, Amirkeivan Mohtashami, Maximilian L Croci, Bo Li, Martin Jaggi, Dan Alistarh, Torsten Hoefler, and James Hensman. Quarot: Outlier-free 4-bit inference in rotated llms. arXiv preprint arXiv:2404.00456, 2024.

[^3]: Yoshua Bengio, Nicholas Léonard, and Aaron C. Courville. Estimating or propagating gradients through stochastic neurons for conditional computation. ArXiv, abs/1308.3432, 2013.

[^4]: Yash Bhalgat, Jinwon Lee, Markus Nagel, Tijmen Blankevoort, and Nojun Kwak. Lsq+: Improving low-bit quantization through learnable offsets and better initialization. 2020 IEEE/CVF Conference on Computer Vision and Pattern Recognition Workshops (CVPRW), pages 2978–2985, 2020.

[^5]: Yonatan Bisk, Rowan Zellers, Jianfeng Gao, Yejin Choi, et al. Piqa: Reasoning about physical commonsense in natural language. In Proceedings of the AAAI conference on artificial intelligence, pages 7432–7439, 2020.

[^6]: Sébastien Bubeck, Varun Chandrasekaran, Ronen Eldan, Johannes Gehrke, Eric Horvitz, Ece Kamar, Peter Lee, Yin Tat Lee, Yuanzhi Li, Scott Lundberg, et al. Sparks of artificial general intelligence: Early experiments with gpt-4. arXiv preprint arXiv:2303.12712, 2023.

[^7]: Yuji Chai, John Gkountouras, Glenn G Ko, David Brooks, and Gu-Yeon Wei. Int2. 1: Towards fine-tunable quantized large language models with error correction through low-rank adaptation. arXiv preprint arXiv:2306.08162, 2023.

[^8]: Jerry Chee, Yaohui Cai, Volodymyr Kuleshov, and Christopher De Sa. Quip: 2-bit quantization of large language models with guarantees. arXiv preprint arXiv:2307.13304, 2023.

[^9]: Hong Chen, Chengtao Lv, Liang Ding, Haotong Qin, Xiabin Zhou, Yifu Ding, Xuebo Liu, Min Zhang, Jinyang Guo, Xianglong Liu, et al. Db-llm: Accurate dual-binarization for efficient llms. arXiv preprint arXiv:2402.11960, 2024.

[^10]: Wenhua Cheng, Yiyang Cai, Kaokao Lv, and Haihao Shen. Teq: Trainable equivalent transformation for quantization of llms. arXiv preprint arXiv:2310.10944, 2023.

[^11]: Wenhua Cheng, Weiwei Zhang, Haihao Shen, Yiyang Cai, Xin He, and Kaokao Lv. Optimize weight rounding via signed gradient descent for the quantization of llms. arXiv preprint arXiv:2309.05516, 2023.

[^12]: Wei-Lin Chiang, Zhuohan Li, Zi Lin, Ying Sheng, Zhanghao Wu, Hao Zhang, Lianmin Zheng, Siyuan Zhuang, Yonghao Zhuang, Joseph E. Gonzalez, Ion Stoica, and Eric P. Xing. Vicuna: An open-source chatbot impressing gpt-4 with 90%\* chatgpt quality, March 2023.

[^13]: Christopher Clark, Kenton Lee, Ming-Wei Chang, Tom Kwiatkowski, Michael Collins, and Kristina Toutanova. Boolq: Exploring the surprising difficulty of natural yes/no questions. arXiv preprint arXiv:1905.10044, 2019.

[^14]: Peter Clark, Isaac Cowhey, Oren Etzioni, Tushar Khot, Ashish Sabharwal, Carissa Schoenick, and Oyvind Tafjord. Think you have solved question answering? try arc, the ai2 reasoning challenge. arXiv preprint arXiv:1803.05457, 2018.

[^15]: Together Computer. Redpajama: an open dataset for training large language models, 2023.

[^16]: Tim Dettmers, Artidoro Pagnoni, Ari Holtzman, and Luke Zettlemoyer. Qlora: Efficient finetuning of quantized llms. arXiv preprint arXiv:2305.14314, 2023.

[^17]: Tim Dettmers, Ruslan Svirschevski, Vage Egiazarian, Denis Kuznedelev, Elias Frantar, Saleh Ashkboos, Alexander Borzunov, Torsten Hoefler, and Dan Alistarh. Spqr: A sparse-quantized representation for near-lossless llm weight compression. arXiv preprint arXiv:2306.03078, 2023.

[^18]: Xin Ding, Xiaoyu Liu, Yun Zhang, Zhijun Tu, Wei Li, Jie Hu, Hanting Chen, Yehui Tang, Zhiwei Xiong, Baoqun Yin, et al. Cbq: Cross-block quantization for large language models. arXiv preprint arXiv:2312.07950, 2023.

[^19]: Dayou Du, Yijia Zhang, Shijie Cao, Jiaqi Guo, Ting Cao, Xiaowen Chu, and Ningyi Xu. Bitdistiller: Unleashing the potential of sub-4-bit llms via self-distillation. arXiv preprint arXiv:2402.10631, 2024.

[^20]: Vage Egiazarian, Andrei Panferov, Denis Kuznedelev, Elias Frantar, Artem Babenko, and Dan Alistarh. Extreme compression of large language models via additive quantization. arXiv preprint arXiv:2401.06118, 2024.

[^21]: Steven K Esser, Jeffrey L McKinstry, Deepika Bablani, Rathinakumar Appuswamy, and Dharmendra S Modha. Learned step size quantization. arXiv preprint arXiv:1902.08153, 2019.

[^22]: Elias Frantar, Saleh Ashkboos, Torsten Hoefler, and Dan Alistarh. Gptq: Accurate post-training quantization for generative pre-trained transformers. arXiv preprint arXiv:2210.17323, 2022.

[^23]: Chaoyou Fu, Peixian Chen, Yunhang Shen, Yulei Qin, Mengdan Zhang, Xu Lin, Zhenyu Qiu, Wei Lin, Jinrui Yang, Xiawu Zheng, Ke Li, Xing Sun, and Rongrong Ji. Mme: A comprehensive evaluation benchmark for multimodal large language models. ArXiv, abs/2306.13394, 2023.

[^24]: Ruihao Gong, Yang Yong, Shiqiao Gu, Yushi Huang, Yunchen Zhang, Xianglong Liu, and Dacheng Tao. Llm-qbench: A benchmark towards the best practice for post-training quantization of large language models. arXiv preprint arXiv:2405.06001, 2024.

[^25]: Han Guo, Philip Greengard, Eric P Xing, and Yoon Kim. Lq-lora: Low-rank plus quantized matrix decomposition for efficient language model finetuning. arXiv preprint arXiv:2311.12023, 2023.

[^26]: Dan Hendrycks, Collin Burns, Steven Basart, Andy Zou, Mantas Mazeika, Dawn Song, and Jacob Steinhardt. Measuring massive multitask language understanding. arXiv preprint arXiv:2009.03300, 2020.

[^27]: J. Edward Hu, Yelong Shen, Phillip Wallis, Zeyuan Allen-Zhu, Yuanzhi Li, Shean Wang, and Weizhu Chen. Lora: Low-rank adaptation of large language models. ArXiv, abs/2106.09685, 2021.

[^28]: Wei Huang, Xudong Ma, Haotong Qin, Xingyu Zheng, Chengtao Lv, Hong Chen, Jie Luo, Xiaojuan Qi, Xianglong Liu, and Michele Magno. How good are low-bit quantized llama3 models? an empirical study. arXiv preprint arXiv:2404.14047, 2024.

[^29]: Jeonghoon Kim, Jung Hyun Lee, Sungdong Kim, Joonsuk Park, Kang Min Yoo, Se Jung Kwon, and Dongsoo Lee. Memory-efficient fine-tuning of compressed large language models via sub-4-bit integer quantization. arXiv preprint arXiv:2305.14152, 2023.

[^30]: Sehoon Kim, Coleman Hooper, Amir Gholami, Zhen Dong, Xiuyu Li, Sheng Shen, Michael W Mahoney, and Kurt Keutzer. Squeezellm: Dense-and-sparse quantization. arXiv preprint arXiv:2306.07629, 2023.

[^31]: Changhun Lee, Jungyu Jin, Taesu Kim, Hyungjun Kim, and Eunhyeok Park. Owq: Lessons learned from activation outliers for weight quantization in large language models. arXiv preprint arXiv:2306.02272, 2023.

[^32]: Jung Hyun Lee, Jeonghoon Kim, Se Jung Kwon, and Dongsoo Lee. Flexround: Learnable rounding based on element-wise division for post-training quantization. In International Conference on Machine Learning, pages 18913–18939. PMLR, 2023.

[^33]: Liang Li, Qingyuan Li, Bo Zhang, and Xiangxiang Chu. Norm tweaking: High-performance low-bit quantization of large language models. arXiv preprint arXiv:2309.02784, 2023.

[^34]: Qingyuan Li, Ran Meng, Yiduo Li, Bo Zhang, Liang Li, Yifan Lu, Xiangxiang Chu, Yerui Sun, and Yuchen Xie. A speed odyssey for deployable quantization of llms. arXiv preprint arXiv:2311.09550, 2023.

[^35]: Yixiao Li, Yifan Yu, Chen Liang, Pengcheng He, Nikos Karampatziakis, Weizhu Chen, and Tuo Zhao. Loftq: Lora-fine-tuning-aware quantization for large language models. arXiv preprint arXiv:2310.08659, 2023.

[^36]: Yuhang Li, Ruihao Gong, Xu Tan, Yang Yang, Peng Hu, Qi Zhang, Fengwei Yu, Wei Wang, and Shi Gu. Brecq: Pushing the limit of post-training quantization by block reconstruction. arXiv preprint arXiv:2102.05426, 2021.

[^37]: Ji Lin, Jiaming Tang, Haotian Tang, Shang Yang, Xingyu Dang, and Song Han. Awq: Activation-aware weight quantization for llm compression and acceleration. arXiv preprint arXiv:2306.00978, 2023.

[^38]: Yujun Lin, Haotian Tang, Shang Yang, Zhekai Zhang, Guangxuan Xiao, Chuang Gan, and Song Han. Qserve: W4a8kv4 quantization and system co-design for efficient llm serving. arXiv preprint arXiv:2405.04532, 2024.

[^39]: Haotian Liu, Chunyuan Li, Yuheng Li, and Yong Jae Lee. Improved baselines with visual instruction tuning. arXiv preprint arXiv:2310.03744, 2023.

[^40]: Haotian Liu, Chunyuan Li, Qingyang Wu, and Yong Jae Lee. Visual instruction tuning. arXiv preprint arXiv:2304.08485, 2023.

[^41]: Jing Liu, Ruihao Gong, Xiuying Wei, Zhiwei Dong, Jianfei Cai, and Bohan Zhuang. Qllm: Accurate and efficient low-bitwidth quantization for large language models. arXiv preprint arXiv:2310.08041, 2023.

[^42]: Yuanzhan Liu, Haodong Duan, Yuanhan Zhang, Bo Li, Songyang Zhang, Wangbo Zhao, Yike Yuan, Jiaqi Wang, Conghui He, Ziwei Liu, Kai Chen, and Dahua Lin. Mmbench: Is your multi-modal model an all-around player? ArXiv, abs/2307.06281, 2023.

[^43]: Zechun Liu, Barlas Oguz, Changsheng Zhao, Ernie Chang, Pierre Stock, Yashar Mehdad, Yangyang Shi, Raghuraman Krishnamoorthi, and Vikas Chandra. Llm-qat: Data-free quantization aware training for large language models. arXiv preprint arXiv:2305.17888, 2023.

[^44]: Pan Lu, Swaroop Mishra, Tony Xia, Liang Qiu, Kai-Wei Chang, Song-Chun Zhu, Oyvind Tafjord, Peter Clark, and A. Kalyan. Learn to explain: Multimodal reasoning via thought chains for science question answering. ArXiv, abs/2209.09513, 2022.

[^45]: Shuming Ma, Hongyu Wang, Lingxiao Ma, Lei Wang, Wenhui Wang, Shaohan Huang, Li Dong, Ruiping Wang, Jilong Xue, and Furu Wei. The era of 1-bit llms: All large language models are in 1.58 bits. arXiv preprint arXiv:2402.17764, 2024.

[^46]: Markus Nagel, Rana Ali Amjad, Mart Van Baalen, Christos Louizos, and Tijmen Blankevoort. Up or down? adaptive rounding for post-training quantization. In International Conference on Machine Learning, pages 7197–7206. PMLR, 2020.

[^47]: Haotong Qin, Xudong Ma, Xingyu Zheng, Xiaoyang Li, Yang Zhang, Shouda Liu, Jie Luo, Xianglong Liu, and Michele Magno. Accurate lora-finetuning quantization of llms via information retention. arXiv preprint arXiv:2402.05445, 2024.

[^48]: Yujia Qin, Shengding Hu, Yankai Lin, Weize Chen, Ning Ding, Ganqu Cui, Zheni Zeng, Yufei Huang, Chaojun Xiao, Chi Han, Yi Ren Fung, Yusheng Su, Huadong Wang, Cheng Qian, Runchu Tian, Kunlun Zhu, Shi Liang, Xingyu Shen, Bokai Xu, Zhen Zhang, Yining Ye, Bo Li, Ziwei Tang, Jing Yi, Yu Zhu, Zhenning Dai, Lan Yan, Xin Cong, Ya-Ting Lu, Weilin Zhao, Yuxiang Huang, Jun-Han Yan, Xu Han, Xian Sun, Dahai Li, Jason Phang, Cheng Yang, Tongshuang Wu, Heng Ji, Zhiyuan Liu, and Maosong Sun. Tool learning with foundation models. ArXiv, abs/2304.08354, 2023.

[^49]: Yujia Qin, Shi Liang, Yining Ye, Kunlun Zhu, Lan Yan, Ya-Ting Lu, Yankai Lin, Xin Cong, Xiangru Tang, Bill Qian, Sihan Zhao, Runchu Tian, Ruobing Xie, Jie Zhou, Marc H. Gerstein, Dahai Li, Zhiyuan Liu, and Maosong Sun. Toolllm: Facilitating large language models to master 16000+ real-world apis. ArXiv, abs/2307.16789, 2023.

[^50]: Keisuke Sakaguchi, Ronan Le Bras, Chandra Bhagavatula, and Yejin Choi. Winogrande: An adversarial winograd schema challenge at scale. Communications of the ACM, 64(9):99–106, 2021.

[^51]: Yuzhang Shang, Zhihang Yuan, Qiang Wu, and Zhen Dong. Pb-llm: Partially binarized large language models. arXiv preprint arXiv:2310.00034, 2023.

[^52]: Wenqi Shao, Mengzhao Chen, Zhaoyang Zhang, Peng Xu, Lirui Zhao, Zhiqian Li, Kaipeng Zhang, Peng Gao, Yu Qiao, and Ping Luo. Omniquant: Omnidirectionally calibrated quantization for large language models. arXiv preprint arXiv:2308.13137, 2023.

[^53]: Rohan Taori, Ishaan Gulrajani, Tianyi Zhang, Yann Dubois, Xuechen Li, Carlos Guestrin, Percy Liang, and Tatsunori B. Hashimoto. Stanford alpaca: An instruction-following llama model. [https://github.com/tatsu-lab/stanford\_alpaca](https://github.com/tatsu-lab/stanford_alpaca), 2023.

[^54]: BitBLAS Team. Bitblas.

[^55]: MLC team. MLC-LLM, 2023.

[^56]: Hugo Touvron, Thibaut Lavril, Gautier Izacard, Xavier Martinet, Marie-Anne Lachaux, Timothée Lacroix, Baptiste Rozière, Naman Goyal, Eric Hambro, Faisal Azhar, et al. Llama: Open and efficient foundation language models. arXiv preprint arXiv:2302.13971, 2023.

[^57]: Hugo Touvron, Louis Martin, Kevin Stone, Peter Albert, Amjad Almahairi, Yasmine Babaei, Nikolay Bashlykov, Soumya Batra, Prajjwal Bhargava, Shruti Bhosale, et al. Llama 2: Open foundation and fine-tuned chat models. arXiv preprint arXiv:2307.09288, 2023.

[^58]: Albert Tseng, Jerry Chee, Qingyao Sun, Volodymyr Kuleshov, and Christopher De Sa. Quip#: Even better llm quantization with hadamard incoherence and lattice codebooks. arXiv preprint arXiv:2402.04396, 2024.

[^59]: Xiuying Wei, Yunchen Zhang, Yuhang Li, Xiangguo Zhang, Ruihao Gong, Jinyang Guo, and Xianglong Liu. Outlier suppression+: Accurate quantization of large language models by equivalent and optimal shifting and scaling. arXiv preprint arXiv:2304.09145, 2023.

[^60]: Xiuying Wei, Yunchen Zhang, Xiangguo Zhang, Ruihao Gong, Shanghang Zhang, Qi Zhang, Fengwei Yu, and Xianglong Liu. Outlier suppression: Pushing the limit of low-bit transformer language models. Advances in Neural Information Processing Systems, 35:17402–17414, 2022.

[^61]: Guangxuan Xiao, Ji Lin, Mickael Seznec, Hao Wu, Julien Demouth, and Song Han. Smoothquant: Accurate and efficient post-training quantization for large language models. In International Conference on Machine Learning, pages 38087–38099. PMLR, 2023.

[^62]: Peng Xu, Wenqi Shao, Mengzhao Chen, Shitao Tang, Kaipeng Zhang, Peng Gao, Fengwei An, Yu Qiao, and Ping Luo. Besa: Pruning large language models with blockwise parameter-efficient sparsity allocation. arXiv preprint arXiv:2402.16880, 2024.

[^63]: Peng Xu, Wenqi Shao, Kaipeng Zhang, Peng Gao, Shuo Liu, Meng Lei, Fanqing Meng, Siyuan Huang, Yu Qiao, and Ping Luo. Lvlm-ehub: A comprehensive evaluation benchmark for large vision-language models. arXiv preprint arXiv:2306.09265, 2023.

[^64]: Yuhui Xu, Lingxi Xie, Xiaotao Gu, Xin Chen, Heng Chang, Hengheng Zhang, Zhensu Chen, Xiaopeng Zhang, and Qi Tian. Qa-lora: Quantization-aware low-rank adaptation of large language models. arXiv preprint arXiv:2309.14717, 2023.

[^65]: Yuzhuang Xu, Xu Han, Zonghan Yang, Shuo Wang, Qingfu Zhu, Zhiyuan Liu, Weidong Liu, and Wanxiang Che. Onebit: Towards extremely low-bit large language models. arXiv preprint arXiv:2402.11295, 2024.

[^66]: Kaining Ying, Fanqing Meng, Jin Wang, Zhiqian Li, Han Lin, Yue Yang, Hao Zhang, Wenbo Zhang, Yuqi Lin, Shuo Liu, et al. Mmt-bench: A comprehensive multimodal benchmark for evaluating large vision-language models towards multitask agi. arXiv preprint arXiv:2404.16006, 2024.

[^67]: Weihao Yu, Zhengyuan Yang, Linjie Li, Jianfeng Wang, Kevin Lin, Zicheng Liu, Xinchao Wang, and Lijuan Wang. Mm-vet: Evaluating large multimodal models for integrated capabilities. ArXiv, abs/2308.02490, 2023.

[^68]: Zhihang Yuan, Lin Niu, Jiawei Liu, Wenyu Liu, Xinggang Wang, Yuzhang Shang, Guangyu Sun, Qiang Wu, Jiaxiang Wu, and Bingzhe Wu. Rptq: Reorder-based post-training quantization for large language models. arXiv preprint arXiv:2304.01089, 2023.

[^69]: Zhihang Yuan, Yuzhang Shang, Yang Zhou, Zhen Dong, Chenhao Xue, Bingzhe Wu, Zhikai Li, Qingyi Gu, Yong Jae Lee, Yan Yan, et al. Llm inference unveiled: Survey and roofline model insights. arXiv preprint arXiv:2402.16363, 2024.

[^70]: Rowan Zellers, Ari Holtzman, Yonatan Bisk, Ali Farhadi, and Yejin Choi. Hellaswag: Can a machine really finish your sentence? arXiv preprint arXiv:1905.07830, 2019.

[^71]: Yilong Zhao, Chien-Yu Lin, Kan Zhu, Zihao Ye, Lequn Chen, Size Zheng, Luis Ceze, Arvind Krishnamurthy, Tianqi Chen, and Baris Kasikci. Atom: Low-bit quantization for efficient and accurate llm serving. arXiv preprint arXiv:2310.19102, 2023.