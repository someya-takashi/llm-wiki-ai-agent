---
title: "FP8-LM: Training FP8 Large Language Models"
source: "https://ar5iv.labs.arxiv.org/html/2310.18313"
author:
published:
created: 2026-08-02
description: "In this paper, we explore FP8 low-bit data formats for efficient training of large language models (LLMs). Our key insight is that most variables, such as gradients and optimizer states, in LLM training can employ low-…"
tags:
  - "clippings"
---
Houwen Peng <sup>∗</sup> Kan Wu <sup>∗</sup> Yixuan Wei <sup>∗</sup>    Guoshuai Zhao   Yuxiang Yang   Ze Liu   Yifan Xiong   Ziyue Yang    Bolin Ni   Jingcheng Hu   Ruihang Li   Miaosen Zhang   Chen Li   Jia Ning   Ruizhe Wang   Zheng Zhang    Shuguang Liu   Joe Chau   Han Hu <sup>†</sup>   Peng Cheng <sup>†</sup>    
Microsoft Azure and Microsoft Research

###### Abstract

In this paper, we explore FP8 low-bit data formats for efficient training of large language models (LLMs). Our key insight is that most variables, such as gradients and optimizer states, in LLM training can employ low-precision data formats without compromising model accuracy and requiring no changes to hyper-parameters. Specifically, we propose a new FP8 automatic mixed-precision framework for training LLMs. This framework offers three levels of FP8 utilization to streamline mixed-precision and distributed parallel training for LLMs. It gradually incorporates 8-bit gradients, optimizer states, and distributed learning in an incremental manner. Experiment results show that, during the training of GPT-175B model on H100 GPU platform, our FP8 mixed-precision training framework not only achieved a remarkable 39% reduction in real memory usage but also ran 75% faster than the widely adopted BF16 framework (i.e., Megatron-LM), surpassing the speed of Nvidia Transformer Engine by 37%. This largely reduces the training costs for large foundation models. Furthermore, our FP8 mixed-precision training methodology is generic. It can be seamlessly applied to other tasks such as LLM instruction tuning and reinforcement learning with human feedback, offering savings in fine-tuning expenses. Our FP8 low-precision training framework is open-sourced at [aka.ms/MS.AMP](https://github.com/Azure/MS-AMP).

<sup>†</sup> <sup>†</sup>

## Introduction

Large language models (LLMs) [^6] [^61] [^7] [^76] have demonstrated unprecedented capabilities in language comprehension and generation, leading to breakthroughs in reasoning, math, science, and many other tasks [^43] [^1]. However, training LLMs is extremely costly. For example, PaLM takes 6,144 TPUv4 chips to train a 540B model, while GPT-3 175B consumes several thousand petaflop/s-days of compute for pre-training [^7] [^6]. This motivates the needs of reducing the training costs of LLMs, especially for the scaling of next-generation super-intelligent models.

Low-precision training is one of the most promising directions to reduce the costs, as it can provide high speed, small memory footprint, and low communication overhead. Most existing training systems, *e.g.*, Megatron-LM [^60], MetaSeq [^76], and Colossal-AI [^27], train LLMs with either FP32 full-precision or FP16/BF16 mixed-precision by default. This is not essential, however, to achieve full accuracy for large models. With the release of Nvidia H100 GPU, FP8 is becoming the next-generation datatype for low-precision representation [^39] [^34]. Theoretically, FP8 can achieve $2\times$ speed-up, 50% - 75% memory cost savings, and 50% - 75% communication savings compared with current 16-bit and 32-bit floating point mixed-precision training, which is very promising for scaling-up next-generation foundation models.

Unfortunately, the current support for FP8 training is rare and limited. The only usable framework is the Nvidia Transformer Engine (TE) [^40], but it applies FP8 solely for GEMM computation and still retains master weights and gradients using high precision, *e.g.*, FP16 or FP32. As a result, the end-to-end speed-up, memory and communication cost savings are very limited, which does not fully unveil the power of FP8. To address this issue, we propose an extremely optimized FP8 mixed-precision framework for LLM training. The core idea is to infiltrate FP8 compute, storage, and communication into the whole progress of large model training, making the forward and backward pass all used the low-precision FP8, thus largely reducing system workloads compared to previous frameworks [^33] [^40] [^34]. Specifically, we design three optimization levels that utilize FP8 to streamline mixed-precision and distributed training. The three levels gradually incorporate 8-bit collective communication, optimizer, and distributed parallel training in an incremental manner. The higher optimization level indicates using more FP8 during LLM training. Moreover, for large-scale training, such as GPT-175B trained on thousand of GPUs, our framework provides FP8 low-bit parallelism, including tensor, pipeline, and sequence parallelism, paving the way to next-generation low-precision parallel training.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2310.18313/assets/x1.png)

Figure 1: An analysis of comparing the maximum model sizes attainable through the utilization of either the prevalent BF16 or our FP8 mixed-precision training approach on a cluster of Nvidia H100 GPUs with 80GB memory.

Training LLMs with FP8 is non-trivial. The challenges stem from issues such as data underflow or overflow, coupled with quantization errors arising from the narrower dynamic range and reduced precision inherent in FP8 data formats. These challenges cause numerical instabilities and irreversible divergences throughout the training process. To tackle them, we propose two techniques: *precision decoupling* and *automatic scaling* for preventing the loss of critical information. The former one involves decoupling the influence of data precision on parameters such as weights, gradients, optimizer states, and assigning reduced precision to components that are not precision sensitive. The latter one is to preserve gradient values within the representation range of FP8 data formats through the dynamic adjustment of tensor scaling factors, thereby alleviating underflow and overflow occurrences during all-reduce communication.

To validate the proposed FP8 low-precision framework, we apply it to GPT-style model training, encompassing both pre-training and supervised fine-tuning (SFT). The experimental results demonstrate the effectiveness of our FP8 methodology, yielding substantial benefits including a 29% to 39% reduction in real memory usage (*e.g.*, 29% reduction for GPT-7B while 39% for GPT-175B ) and a notable 63% to 65% decrease in weight-related communication overhead compared to the prevalent BF16 mixed-precision training approach. Without changes to any hyper-parameters, such as learning rate and weight decay, the models trained using FP8 exhibit performance equivalency to those employing BF16 high precision, both in pre-training and downstream tasks. It is noteworthy that during the training of GPT-175B model, our FP8 mix-precision framework reduces training time by 37% compared to TE [^40], while consuming 42% less memory on H100 GPU platform. More importantly, the reduction in costs achieved through the utilization of low-precision FP8 can be further increased, as the scale of models continues to expand, which is presented in Fig. 1.

For fine-tuning, we employ FP8 mixed-precision for instruction tuning and reinforcement learning with human feedback (RLHF) to better align pre-trained LLMs with end tasks and user preferences. Specifically, we fine-tune pre-trained models on publicly user-shared instruction-following data [^59]. The models tuned with our FP8 mixed-precision demonstrate comparable performance to those utilizing the half-precision BF16 [^77] on the AlpacaEval [^28] and MT-Bench [^77] benchmarks, while achieving 27% improvements in training speed. Moreover, FP8 mixed-precision exhibits considerable potentials in RLHF, a process that necessitates loading multiple models during training. Through the utilization of FP8 in training, the prevalent RLHF framework AlpacaFarm [^14] can yield a 32% reduction in model weights and a 62% reduction in optimizer states’ memory consumption. This further demonstrates the versatility and adaptability of our FP8 low-precision training framework.

We are making the following contributions to drive the design of next-generation FP8 low-precision training for LLMs.

- A new FP8 mixed-precision training framework. It unlocks 8-bit weights, gradients, optimizer, and distributed training gradually in an add-on fashion, which is convenient in use. This 8-bit framework can be used as a simple drop-in replacement for existing 16/32-bit mixed-precision counterparts, without requiring any changes to the hyper-parameters and training receipts. Additionally, we provide a Pytorch implementation that enables 8-bit low-precision training in a few lines of code.
- A new family of GPT-style models trained with FP8. We apply the proposed FP8 scheme to GPT pre-training and fine-tuning (*i.e.*, SFT and RLHF), and demonstrate its potentials on a variety of model scales ranging from 7B to 175B parameters. We equip prevalent parallel computation paradigms with FP8 supports, including tensor, pipeline, and sequence parallelisms, enabling the utilization of FP8 to train large foundation models. We open-source the first FP8 GPT training codebase based upon Megatron-LM [^60] implementation.

We expect the release of our FP8 framework will establish a new paradigm for next-generation low-precision training system dedicated to large foundation models.

## FP8 LLMs

Mixed-precision [^33] has been widely used in LLM training to improve compute and memory efficiency. The most popular mixed-precision schemes are FP16-FP32 and BF16-FP32. Because of the restricted numerical range of FP16, FP16-FP32 scheme has been known instabilities for training large models [^48] [^75]. Consequently, the community now commonly adopts BF16-FP32 for training LLMs, such as Megatron-Turing NLG-530B [^61], Bloom-175B [^57] and Gopher [^48]. The underlying reason is that BF16 has a wide dynamic range to maintain numerical stability while matching the performance of the full-precision FP32. Moreover, BF16 employs half the number of bits as compared to FP32, thus reducing considerable memory footprints while improving compute efficiency.

FP8 is a natural evolution from 16-bit data formats to further reducing computing costs. However, training LLMs with reduced-precision FP8 poses new challenges. The dynamic range and representation precision of FP8 <sup>1</sup> are much lower than BF16 and FP16, which inevitably induces more training collapses, such as loss spikes or even NaNs. To address the issues, tensor scaling techniques are proposed [^63] [^34]. The core idea is multiplying higher precision values with a scaling factor prior to their casting to FP8 in order to move them into a range that better overlaps with the representable range of a corresponding FP8 format <sup>2</sup> [^34]. Such a per-tensor scaling technique reduces data quantization errors while improving numerical stability and accuracy, thus enabling the utilization of the lower-precision FP8 for training large models.

Unfortunately, the current support for FP8 low-precision training is restricted. Nvidia TE [^40] only supports FP8 compute for linear layers in Transformer [^69], while leaving all other operations, such as weight update and gradient synchronization, still using higher precision. In this work, we present an extremely optimized FP8 mixed-precision strategy for LLM training. The new FP8 optimization includes three key perspectives: FP8 communication, FP8 optimizer, and FP8 distributed training. By integrating these aspects, the training of LLMs such as the 175B GPT-3 model can fully harness the advantages of FP8 low-precision and improve training efficiency.

### 2.1 FP8 Gradient and All-Reduce Communication

Existing mixed-precision training methodologies [^33] [^40] typically employ 16-bit or 32-bit datatype for the computation and storage of gradients, resulting in a high bandwidth requirement for collective communication throughout the training process. We found that directly applying FP8 to gradients leads to a decrease in accuracy. The fundamental issue lies in the underflow and overflow problems arising from the low-bit all-reduce operation. Specifically, there are two standard methods aggregating gradients across GPUs during all-reduce: *pre-scaling* and *post-scaling*. Pre-scaling divides the gradient $g_{i}$ calculated on the $i$ -th GPU by the total number of GPUs (*i.e.*, $N$) before being summed, which is formulated as:

$$
g=g_{1}/N+g_{2}/N+\cdots+g_{N}/N.
$$

When $N$ is large, this division can cause data underflow, especially for FP8 low-precision representation of gradients. To mitigate this issue, post-scaling performs the gradient summation first, followed by the division scaling during the gradient collection process:

$$
g=(g_{1}+g_{2}+\cdots+g_{N})/N.
$$

This post-scaling approach keeps the gradients close to the maximum value of the FP8 datatype, effectively alleviating the underflow issue. However, this approach encounters overflow issues when aggregating gradients.

In contrast, we propose an *automatic scaling* technique to resolve both the underflow and overflow issues in the pre-scaling and post-scaling approaches. To be specific, we introduce an auto-scaling factor $\mu$, that changes on the fly during the training, to reduce the occurrences of overflow and underflow in gradients:

$$
g^{\prime}_{i}=\mu\cdot g_{i}.
$$

A statistical analysis is conducted on the gradient values of $g^{\prime}_{i}$, with the objective of quantifying the proportion of values that attains the maximum feasible value within the FP8 representation range. If the ratio of the maximum value exceeds a specified threshold, *i.e.*, 0.001%, $\mu$ is set to $1/2$ in the subsequent training step, thereby mitigating the risk of overflow. Conversely, when the ratio consistently remains the threshold, we opt to exponentially increase $\mu$ to $2$ over the span of 1,000 training steps, thereby effectively mitigating the risk of underflow occurrences.

Another key obstacle of FP8 collective communication lies in devising an effective strategy to manage the tensor-wise scaling factors that are associated with each gradient tensor. The current NCCL implementation [^38] lacks the capability of performing all-reduce operation considering the additional tensor-wise scaling factors. Meanwhile, efficient implementation is also very challenging, especially considering that the NCCL gradient summation operates at sub-tensor level. This complexity increases significantly when incorporating updates for tensor-wise scaling factors. To overcome this issue, we propose a new mechanism that scales FP8 gradients across GPUs using a single shared scalar. To be specific, let $(g^{\prime}_{i},s^{\prime}_{i})$ denote a scaling tensor which stores the weight gradient in the $i$ -th GPU, where $g^{\prime}_{i}$ is a FP8 tensor and $s^{\prime}_{i}$ is the corresponding scaling factor. The actual weight gradient is ${g^{\prime}_{i}}/{s^{\prime}_{i}}$. Prior to the all-reduce operation over gradient tensors, we first gather the scaling factors $s^{\prime}_{i}$ of each gradient tensor on all GPUs and calculate the global minimum scaling factor $s^{\prime}_{g}$ as:

$$
s^{\prime}_{g}=\textrm{min}\left(s^{\prime}_{1},~{}s^{\prime}_{2},~{}\ldots,~{}s^{\prime}_{N}\right),
$$

where the global minimum scaling factor $s^{\prime}_{g}$ is shared across GPUs. We use this shared scaling factor $s^{\prime}_{g}$ to unify the rescaling of the gradient tensors across GPUs. In this way, all gradient tensors associated with the same weight use the same shared scaling factor to quantize the tensors into FP8 format on all GPUs:

$$
g^{\prime\prime}_{i}=\textrm{FP8}\left(s^{\prime}_{g}\cdot\left(g^{\prime}_{i}/s^{\prime}_{i}\right)\right).
$$

This approach reduces communication overhead by transmitting only a single scalar $s^{\prime}_{g}$, making the additional synchronization step highly efficient. As the input tensors share the same scaling factor, it eliminates the need of considering all-reduce the scaling factors in parallel and allows for standard NCCL all-reduce operation to be performed. The final collected gradient is obtained as follows:

$$
g=g^{\prime\prime}_{1}+g^{\prime\prime}_{2}+\cdots+g^{\prime\prime}_{N},\qquad s=N\cdot s^{\prime}_{g},
$$

where $g$ is the final aggregated gradient and $s$ is the corresponding scaling factor. Rescaling the scaling factor for the summed gradient $g$ is equivalent to dividing $g$ by $N$ in theory. By implementing the aforementioned dual strategies of distributed and automated scaling, we can successfully realize FP8 low-bit gradient communication while preserving model accuracy. Furthermore, this approach entails storing gradients in FP8 and conducting communication in FP8 as well, thereby yielding reductions in GPU memory usage and communication bandwidth consumption.

### 2.2 FP8 Optimizer

In the training of LLMs, Adam and its variants [^23] [^32] are the most frequently-used optimization methods, that maintain copies of model weights, gradients, first-order and second-order gradient moments for model updates. Mixed-precision training [^33] with Adam optimizer typically stores master weights, gradients and gradient moments in 32-bit float format for numerical stability [^60] [^50] [^76] [^57]. Consequently, the Adam optimizer consumes 16 bytes of memory per parameter during training:

$$
\underbrace{4}_{\text{master weights}}+\underbrace{4}_{\text{gradients}}+~{}~{}\underbrace{~{}~{}4~{}~{}+~{}~{}4~{}~{}}_{\text{Adam states}}~{}~{}=~{}~{}16\ \text{bytes}.
$$

When model size is large, the memory consumption of the variables in Adam will become a bottleneck. Previous work [^48] [^75] [^31] has revealed that reducing precision of the variables in optimizer to 16-bit leads to accuracy degradation when training billion-scale models <sup>3</sup>. This prompts an evaluation of which variables in the optimizer should be allocated high precision and which can be accommodated with low-precision.

To clarify, we decouple the influence of data precision on the variables in the optimizer and investigate which one can be assigned lower precision, *i.e.*, *precision decoupling*. We find a guiding principle: the gradient statistics can use lower precision, while the master weights necessitate high precision. More concretely, the first-order gradient moment can tolerate a high quantization error and can be assigned with low-precision FP8, while the second-order moment requires a higher precision, as analyzed in Sec. 3.3. This stems from the fact that, during model updates in Adam, the direction of the gradient holds greater significance than its magnitude. FP8 with tensor scaling can effectively preserve the distribution of the first-order moment as the high-precision tensor, though it introduces precision degradation to some extend. Calculating the square of gradients for the second-order gradient moment might lead to data underflow due to the typically small gradient values. Therefore, allocating a 16-bit higher precision is necessary to preserve numerical accuracy.

On the other hand, we find that it is crucial to keep the master weights using high precision. The underlying reason is that weight updates can sometimes become extremely small or large during training, higher precision for the master weights helps prevent loss of information when updating weights, ensuring more stable and accurate training. In the implementation, the master weights have two viable options: utilizing either FP32 full-precision or FP16 with tensor scaling. FP16 with tensor scaling offers the advantage of conserving memory without compromising accuracy. Consequently, our default choice is to employ FP16 with tensor scaling for storing master weights in the optimizer. Our FP8 mixed-precision optimizer consumes 6 bytes of memory per parameter during training:

$$
\underbrace{2}_{\text{master weights}}+\underbrace{1}_{\text{gradients}}+~{}~{}\underbrace{~{}~{}1~{}~{}+~{}~{}2~{}~{}}_{\text{Adam states}}~{}~{}=~{}~{}6\ \text{bytes}.
$$

This new low-bit optimizer reduces memory footprints by 2.6x in comparison to the previous solution, as exemplified in Eq. (7). Noteworthily, this is the first FP8 optimizer for LLM training. The experiments in Sec. 3.2 show that FP8 optimizer can preserve model accuracy at various scales, ranging from 125M to 175B parameters.

### 2.3 FP8 Distributed Parallel Training

Training LLMs like GPT-3 requires distributed learning strategies for parallelizing across GPUs. The frequently-used strategies include data parallelism, tensor parallelism, pipeline parallelism, and sequence parallelism. Each parallelism has its own merits and has been used in a complementary fashion in existing systems [^61] [^60] [^76] [^57] [^27]. For FP8 supports of these strategies, neither data parallelism nor pipeline parallelism necessitates any specific modifications, because they do not involve additional FP8 compute and communication when splitting data batches or model layers into segments across devices.

Tensor parallelism partitions individual layers of a model across multiple devices, such that the shards of weight, gradient and activation tensors are placed on separate GPUs, instead of a single one. To equip tensor parallelism with FP8, we convert the sharded weight and activation tensors to FP8 format for linear layer computation, enabling the forward compute and backward gradient collective communication all using FP8. On the other hand, sequence parallelism splits input sequences into multiple chunks and the sub-sequences are fed to different devices to save activation memory. As shown in Fig. 2, sequence and tensor parallelism are performed in parallel to different parts of a Transformer model to make the best use of the available memory and improve training efficiency. There is a converter $g$ between sequence and tensor parallel regions to all-gather sequence partitions in the forward pass (or reduce-scatter tensor segments in the backward pass). We add an FP8 datatype conversion prior to $g$, such that the all-gather (or reduce-scatter) operation uses FP8 low-bit activation to save communication cost across GPUs.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2310.18313/assets/x2.png)

Figure 2: Transformer layer with FP8 tensor and sequence parallelism. The FP8 low-bit operation is highlighted with orange. g 𝑔 is all-gather in forward pass and reduce-scatter in backward pass, while ¯ \\bar{g} is reduce-scatter in forward pass and all-gather in backward pass. The gather-reduce operation between sequence parallel and tensor parallel is executed utilizing FP8 low-precision activation, thus saving half communication costs.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2310.18313/assets/x3.png)

Figure 3: ZeRO tensor partitioning with and without scaling factors. Left: the original high-precision ZeRO method, which splits a single tensor into multiple partitions and distributes them to different devices. Right: the proposed FP8 ZeRO, which distributes each tensor in its entirety across devices while taking tensor scaling into account.

In addition, Zero Redundancy Optimizer (ZeRO) [^50] is another frequently-used distributed learning technique in large model training. The core idea of ZeRO is to shade model states over devices, such that each device only hold a fraction of data (*e.g.*, master weights, gradients, and optimizer states) required for a training step. To reduce memory consumption, ZeRO method generally splits a single tensor into multiple partitions and distributes them to different devices. Directly applying FP8 to ZeRO is infeasible, because it is difficult to handle the scaling factors associated with the FP8 partitions. The per-tensor scaling factors should be distributed along with FP8 partitions. To address this issue, we implement a new FP8 distribution scheme that distributes each tensor as a whole across devices, rather than partitioning it into multiple sub-tensors as in ZeRO. The distribution of FP8 tensors is processed in a greedy manner, as outlined in Alg. 1. Specifically, our method first sorts the tensors of model states according to their sizes, and then distributes the tensors to different GPUs based upon the remaining memory size of each GPU. The distribution follows the principle that the GPUs with larger remaining memory get a higher priority in receiving new distributed tensors. In this way, the tensor scaling factors can be distributed along with the tensors smoothly, while reducing communication and compute complexity. Figure 3 presents a visual illustration of the difference in ZeRO tensor partitioning between scenarios with and without scaling factors.

Algorithm 1 Greedy Distribution Algorithm for ZeRO

FP8 tensors with their corresponding scaling factors: $T=\{(s_{1},t_{1}),(s_{2},t_{2}),\dots,(s_{n},t_{n})\}$, where $s$ denotes scaling factors while $t$ represents 8-bit tensors. The size of each tensor: $C=\{c_{1},c_{2},\dots,c_{n}\}$.

Partitions representing scaling tensors assigned to each GPU.

Sort $T$ in descending order of their sizes to get $T^{\prime}=\{(s^{\prime}_{1},t^{\prime}_{1}),(s^{\prime}_{2},t^{\prime}_{2}),\dots,(s^{\prime}_{n},t^{\prime}_{n})\}$ and $C^{\prime}=\{c^{\prime}_{1},c^{\prime}_{2},\dots,c^{\prime}_{n}\}$, where $c^{\prime}_{1}\geqslant c^{\prime}_{2}\geqslant\dots\geqslant c^{\prime}_{n}$.

Initialize memory usage $u_{j}=0$ and partition $p_{j}=\emptyset$ for each GPU $G_{j}$.

for $i=1$ to $n$ do

$j\leftarrow\arg\min_{j}u_{j}$ $\triangleright$ Find the GPU $j\in[1,m]$ with the least memory usage.

$p_{j}\leftarrow p_{j}\cup\{(s^{\prime}_{i},t^{\prime}_{i})\}$ $\triangleright$ Assign $(s^{\prime}_{i},t^{\prime}_{i})$ to $G_{j}$.

$u_{j}\leftarrow u_{j}+c^{\prime}_{i}$ $\triangleright$ Update the memory usage of $G_{j}$.

end for

return Partitions $P=\{p_{1},p_{2},\dots,p_{m}\}$

## Experiment

In this section, we assess the effectiveness of the proposed FP8 mixed-precision training approach on GPT-style LLMs, including a wide range of model scales, from 125 million to 175 billion parameters. For performance ablation, we compare GPT models trained with FP8 against those trained with half-precision BF16 and full-precision FP32. For generality evaluation, we conduct experiments encompassing both FP8 low-bit pre-training and fine-tuning, considering instruction tuning and human preference alignment.

### 3.1 Experimental Setup

#### 3.1.1 Training Dataset

Our pre-training data is constructed using open-sourced language collections from several sources, including CommonCrawl <sup>4</sup>, The Pile [^16], C4 [^49], OpenWebText [^46] [^17], CC-NEWS [^30], CC-Stories [^68], Redpajama [^53], and Wikipedia <sup>5</sup>. We apply fuzzy deduplication [^26] across CommonCrawl snapshots to enhance data quality. Tab. 10 in Appendix A.3 provides details of our pre-training data, including information such as the number of tokens from each source and associated sampling weights. For a more comprehensive understanding of the data and its cleaning pipeline, readers are encouraged to refer to Appendix A.3.

Moreover, for instruction tuning, we follow the same settings as Vicuna-v1.1 [^70], which uses a publicly user-shared instruction following data [^59]. For reinforcement learning with human feedback, the training data we used is a combination of the Anthropic’s Helpful and Harmless dataset [^2] and Open-Assistant dataset [^25]. The training framework and associated configurations align with the publicly available AlpacaFarm [^14].

#### 3.1.2 Model Configuration

| params | dimension | $n$ heads | $n$ layers | TP | PP | SP | learning rate | batch size | $n$ tokens |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 125M | 768 | 12 | 12 | 1 | 1 | ✓ | $6.0e^{-4}$ | 1M | 100B |
| 7B | 4096 | 32 | 32 | 1 | 1 | ✓ | $3.0e^{-4}$ | 4M | 100B |
| 13B | 5120 | 40 | 40 | 2 | 1 | ✓ | $3.0e^{-4}$ | 4M | 100B |
| 175B | 12288 | 96 | 96 | 8 | 4 | ✓ | $3.0e^{-5}$ | 1M | 40B |

Table 1: Model sizes, architectures, and training hyper-parameters. TP, PP, and SP indicate tensor, pipeline, and sequence parallelism, respectively. To mitigate carbon emissions and save cost, we restrict the training of the 175B model to a dataset comprising only 40B tokens, which has proven to be sufficient for evaluating system performance.

The model architecture we used is a decoder-only Transformer [^6], which has been widely-used in recent generative LLMs like PaLM [^7], OPT [^76], and LLaMA [^67]. In addition to the base architecture, we integrate several modifications proposed recently to improve model efficiency and effectiveness. 1) *Rotary Positional Embedding*: Drawing inspiration from recent successful experiments [^5] [^67], we incorporate rotary positional embeddings (RoPE) [^62] into our approach. This addition enables us to capture both absolute and relative positions information, enhancing performance especially when extrapolating to larger context windows. 2) *Flash Attention*: The standard attention implementation is bottlenecked by memory access [^21]. Flash Attention [^10] proposed an IO-aware exact attention algorithm which uses tiling to reduce the amount of HBM accesses, achieving substantial acceleration.

We train the models using the proposed FP8 optimizer, which is built upon Adam [^23] with decoupled weight decay [^32], following the common practise with the decay rates $\beta_{1}$ = 0.9, $\beta_{2}$ = 0.95, and weight decay = 0.1. The learning rate schedule is cosine-like, and the final learning rate is 10% of the maximal learning rate. We train the models for 100B tokens in total with a batch size of 4M tokens, and the input sequence length is set to 2048. The model warm-up is conducted for 1,000 iterations. Tab. 1 presents the details of model configurations and the corresponding training settings. The training is conducted on Azure NDv5 H100 GPU platform [^35].

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2310.18313/assets/figures/GPT-7b.png)

(a) GPT-7B

<table><tbody><tr><td></td><td>HS</td><td>Lambada</td><td>BoolQ</td><td>PIQA</td><td>COPA</td><td>Winogrande</td><td>Arc-C</td><td>Arc-E</td><td>ObQA</td><td>Avg</td></tr><tr><td colspan="8">GPT-7B model zero-shot performance</td><td></td><td></td><td></td></tr><tr><td>BF16</td><td>61.3</td><td>61.4</td><td>61.2</td><td>75.0</td><td>79.0</td><td>58.5</td><td>32.9</td><td>59.7</td><td>36.4</td><td>58.4</td></tr><tr><td>FP8</td><td>60.0</td><td>61.8</td><td>62.0</td><td>74.2</td><td>78.0</td><td>59.8</td><td>32.9</td><td>58.7</td><td>34.6</td><td>58.0</td></tr><tr><td colspan="8">GPT-13B model zero-shot performance</td><td></td><td></td><td></td></tr><tr><td>BF16</td><td>64.8</td><td>64.9</td><td>63.4</td><td>75.9</td><td>82.0</td><td>61.0</td><td>35.2</td><td>61.5</td><td>40.6</td><td>61.0</td></tr><tr><td>FP8</td><td>64.1</td><td>63.4</td><td>63.9</td><td>76.2</td><td>81.0</td><td>61.6</td><td>34.9</td><td>61.3</td><td>36.8</td><td>60.4</td></tr></tbody></table>

Table 2: Zero-shot performance on downstream tasks. The models are trained with either the standard BF16 mixed-precision scheme [^60] or the proposed FP8 low-precision scheme.

### 3.2 Main Results

#### 3.2.1 Model Performance

We first compare the performance of models trained using FP8 mixed-precision with those trained using BF16. In Fig. 4, the pre-training loss over tokens is displayed for GPT models of 7B, 13B, and 175B parameters. The training configurations and hyper-parameters remain consistent across models trained with FP8 and BF16. The only difference lies in the mixed-precision schemes utilized. As shown in Fig. 4, the loss curves almost overlap with each other. The results unequivocally demonstrate that the proposed FP8 mixed-precision scheme can achieve equivalent performance to the prevalent higher-precision BF16 scheme [^60] [^48] [^18] across a diverse array of model scales. Also, we evaluate the pre-trained models on a wide range of downstream tasks, including HellaSwag (HS) [^74], Lambada [^44] BoolQ [^8], PIQA [^4], COPA [^54], Winogrande [^55], Arc [^9], and OpenbookQA (ObQA) [^36]. As reported in Tab. 2, the FP8 pre-trained models exhibit comparable zero-shot performance in comparison to their BF16 counterparts. This result provides further validation that models pre-trained with FP8 low-precision maintain both accuracy and intrinsic in-context learning capabilities at a level comparable to their high-precision counterparts.

Furthermore, we leverage the proposed FP8 mixed-precision approach for fine-tuning LLMs in instruction following. For a fair comparison, we follow the same instruction tuning settings as Vicuna-v1.1 [^70], which adopts the open-sourced LLaMA-7B [^67] as the base model for fine-tuning. Fig. 5 presents the fine-tuning loss, where the curves corresponding to BF16 and FP8 display a notable degree of overlap. Meanwhile, the win-rate of our FP8 fine-tuned models against Davinci-003 [^42] is also comparable to that of Vicuna-v1.1, which is fine-tuned using BF16 half-precision, as reported in Tab. 3. This indicates that our FP8 low-bit training scheme is versatile, as it is applicable not only to pre-training phase but also to downstream fine-tuning tasks.

In addition, we further apply the proposed FP8 mixed-precision scheme to reinforcement learning from human feedback (RLHF), a more complex process to align LLMs with user preferences. Following the same training setting as AlpacaFarm [^14], a recent RL framework for LLM alignment, we optimize policy models with PPO algorithm [^58]. The solely difference lies in the choice of mixed-precision training schemes, *i.e.*, BF16 *v.s.* FP8. From the results reported in Fig. 6 and Tab. 4, we observe a notable reduction in memory utilization, for instance, a 32% memory reduction concerning model weights and a 62% reduction concerning optimizer states. Consequently, it can be inferred that FP8 is capable of replicating the BF16 mixed-precision for RLHF training. This underscores the broader applicability and versatility of our FP8 low-bit training solution.

![[Uncaptioned image]](https://ar5iv.labs.arxiv.org/html/2310.18313/assets/figures/sft.png)

Figure 5: SFT training loss.

![[Uncaptioned image]](https://ar5iv.labs.arxiv.org/html/2310.18313/assets/figures/rlhf.png)

Figure 6: RLHF training loss.

#### 3.2.2 System Performance

In this section, we evaluate system-level performance of FP8 mixed-precision, considering communication efficiency, memory utilization, and the overall speed, with an emphasis on cost savings. Our method employs 8-bit gradients for all-reduce collective communication among GPUs. Theoretically, this results in a 75% reduction in communication costs when compared to the mainstream 32-bit scheme (Despite BF16 mixed-precision computing gradients using 16-bit precision, it still employs 32-bit precision for all-reduce communication [^60]). Due to the impact of system transmission loss, the observed practical reduction during GPT model training falls within the range of 63% to 65%, as indicated in Table 5. Furthermore, it is worth noting that the recent Nvidia Transformer Engine (TE) [^40] still relies on full-precision FP32 for collective communication, resulting in the same level of reduction for our FP8 solution.

When training GPT models with identical batch sizes, FP8 mixed-precision can lead to a reduction in memory footprint ranging from 28% to 39% when compared to BF16, as reported in Tab. 5. These reductions in memory consumption are attributed to the FP8 gradient and FP8 optimizer techniques we have introduced. Moreover, compared with TE [^40], our solution is also very competitive, obtaining 36.1%, 36.0%, and 42.1% additional memory reductions for different model sizes, *i.e.*, GPT-7B, 13B, and 175B. Although TE employs FP8 for compute, it still uses high-precision optimizer and gradients, which consumes much more memory than our solution. In addition, the saved memory in our method can be used to train larger batch size or longer sequence. For example, when employing 32 H100 GPUs with a memory capacity of 80GB, our approach enables the training of models with a context of 4,096 tokens, accommodating up to 175 billion parameters. In contrast, TE can only accommodate models with a context of 2,048 tokens. This showcases the potential of integrating our FP8 mixed-precision training into existing LLMs, empowering them to train longer sequences with the same GPU resources.

Moreover, our FP8 mixed-precision scheme shows a superior training throughput compared to the prevalent BF16 scheme, achieving a notable speed-up of 75% when applied to GPT-175B model. The model FLOPS utilization (MFU) of FP8 mixed-precision training is 34.2% on H100 GPUs, being 37.3% superior to TE. These findings provide substantial evidence that our FP8 scheme effectively conserves memory, reduces communication costs during the training of large models, and ultimately enhances system utilization efficiency on the latest H100 GPU platform.

<table><tbody><tr><td rowspan="2">Model</td><td rowspan="2">TP</td><td rowspan="2">PP</td><td rowspan="2">DP</td><td>Micro</td><td>Mixed</td><td>GPU</td><td>Throughput</td><td rowspan="2">TFLOPS</td><td>MFU</td><td colspan="2">Weight-related Comm.</td></tr><tr><td>BS</td><td>Precision</td><td>Mem. (GB)</td><td>(#samples/s)</td><td>(%)</td><td>Rate (%)</td><td>Volume (GB)</td></tr><tr><td rowspan="4">GPT-7B</td><td rowspan="4">1</td><td rowspan="4">1</td><td rowspan="4">32</td><td>2</td><td>BF16</td><td>69.6</td><td>159.2</td><td>445</td><td>45.0</td><td>10.1</td><td>37.2</td></tr><tr><td>2</td><td>FP8 (TE)</td><td>77.3</td><td>224.5</td><td>627</td><td>31.7</td><td>9.7</td><td>37.2</td></tr><tr><td>2</td><td>FP8 (Ours)</td><td>49.4 (-29%)</td><td>219.8 (+38%)</td><td>615</td><td>31.1</td><td>7.9</td><td>13.9 (-63%)</td></tr><tr><td>4</td><td>FP8 (Ours)</td><td>69.3</td><td>230.5 (+45%)</td><td>645</td><td>32.6</td><td>10.4</td><td>13.9 (-63%)</td></tr><tr><td rowspan="4">GPT-13B</td><td rowspan="4">2</td><td rowspan="4">1</td><td rowspan="4">16</td><td>2</td><td>BF16</td><td>68.2</td><td>79.3</td><td>420</td><td>42.5</td><td>11.1</td><td>34.3</td></tr><tr><td>2</td><td>FP8 (TE)</td><td>76.4</td><td>111.7</td><td>592</td><td>29.9</td><td>7.1</td><td>34.3</td></tr><tr><td>2</td><td>FP8 (Ours)</td><td>48.9 (-28%)</td><td>109.5 (+38%)</td><td>575</td><td>29.1</td><td>3.9</td><td>12.4 (-64%)</td></tr><tr><td>4</td><td>FP8 (Ours)</td><td>67.8</td><td>121.5 (+53%)</td><td>644</td><td>32.5</td><td>9.3</td><td>12.4 (-64%)</td></tr><tr><td rowspan="4">GPT-175B</td><td rowspan="4">8</td><td rowspan="4">4</td><td rowspan="4">4</td><td>1</td><td>BF16</td><td>66.1</td><td>22.4</td><td>386</td><td>39.0</td><td>8.8</td><td>23.4</td></tr><tr><td>1</td><td>FP8 (TE)</td><td>69.6</td><td>28.7</td><td>493</td><td>24.9</td><td>3.9</td><td>23.4</td></tr><tr><td>1</td><td>FP8 (Ours)</td><td>40.3 (-39%)</td><td>27.1 (+21%)</td><td>473</td><td>23.9</td><td>2.5</td><td>8.2 (-65%)</td></tr><tr><td>4</td><td>FP8 (Ours)</td><td>57.7</td><td>39.3 (+75%)</td><td>677</td><td>34.2</td><td>10.9</td><td>8.2 (-65%)</td></tr></tbody></table>

Table 5: System-level performance on Nvidia H100 GPUs 80GB. Here, TP, PP, and DP represent tensor, pipeline, and data parallelism respectively. BS indicates batch size, while MFU denotes model FLOPs utilization. Weight-related communication contains the all-gather operator on weights and the reduce-scatter operator on weight gradients.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2310.18313/assets/x4.png)

(a)

### 3.3 Ablation Study

We ablate various design choices of FP8 mixed-precision training strategy for LLMs and report the performance in Tab. 6 – 8 and Fig. 7 – 8. The ablation experiments are conducted on GPT models, whose architectures and training settings are elaborated in Tab. 1. Importantly, our ablation study yields several guidelines for the effective utilization of 8-bit datatype in LLM training, which can facilitate future research on low-bit model training.

![[Uncaptioned image]](https://ar5iv.labs.arxiv.org/html/2310.18313/assets/x7.png)

Table 6: Precision decoupling for the variables within the optimizer. Here, our focus is on ablating the master weight and optimizer states, as these components are precision sensitive. The optimizer states include both first-order and second-order gradient moments. Note that the FP16 master weight uses tensor scaling.

Communication. We first analyze the limitations of the conventional pre-scaling and post-scaling methods when aggregating low-bit gradients during the all-reduce communication process. As shown in Fig. 7, we conduct a statistical analysis on SNR, underflow rate, and overflow rate of weight gradients across different Transformer blocks. It is observed that the pre-scaling method has relative larger underflow rate when quantifying gradients from 32-bit to 8-bit, while the post-scaling method has higher overflow rate. In contrast, the proposed auto-scaling technique can diminish both the underflow ratio and the overflow ratio, while getting much better SNR, as shown in Fig. 7 (a). This demonstrates the effectiveness of auto-scaling method in reducing quantization errors when utilizing 8-bit datatype for gradient all-reduce.

<table><tbody><tr><td rowspan="2">Model</td><td rowspan="2">TP</td><td rowspan="2">PP</td><td rowspan="2">DP</td><td>Micro</td><td>Mixed</td><td colspan="2">Act-related Comm.</td></tr><tr><td>BS</td><td>Precision</td><td>Rate (%)</td><td>Volume (GB)</td></tr><tr><td rowspan="2">GPT-13B</td><td rowspan="2">2</td><td rowspan="2">1</td><td rowspan="2">16</td><td rowspan="2">2</td><td>BF16</td><td>12.9</td><td>4.7</td></tr><tr><td>FP8 (Ours)</td><td>5.3</td><td>3.1</td></tr><tr><td rowspan="2">GPT-175B</td><td rowspan="2">8</td><td rowspan="2">4</td><td rowspan="2">4</td><td rowspan="2">1</td><td>BF16</td><td>14.9</td><td>5.9</td></tr><tr><td>FP8 (Ours)</td><td>5.2</td><td>3.9</td></tr></tbody></table>

Table 7: Activation-related communication volume reduction in sequence and tensor parallelism, including the all-gather operator on activation and the reduce-scatter on activation gradients.

<table><tbody><tr><td rowspan="2">Model</td><td rowspan="2">TP</td><td rowspan="2">PP</td><td rowspan="2">DP</td><td>Micro</td><td>Mixed</td><td colspan="2">GPU Memory</td></tr><tr><td>BS</td><td>Precision</td><td>Min</td><td>Max</td></tr><tr><td rowspan="3">GPT-7B</td><td rowspan="3">1</td><td rowspan="3">1</td><td rowspan="3">32</td><td rowspan="3">2</td><td>BF16</td><td>69.07</td><td>69.63</td></tr><tr><td>FP8 (TE)</td><td>76.97</td><td>77.28</td></tr><tr><td>FP8 (Ours)</td><td>49.06</td><td>49.36</td></tr><tr><td rowspan="3">GPT-13B</td><td rowspan="3">2</td><td rowspan="3">1</td><td rowspan="3">16</td><td rowspan="3">2</td><td>BF16</td><td>67.98</td><td>68.18</td></tr><tr><td>FP8 (TE)</td><td>73.68</td><td>76.36</td></tr><tr><td>FP8 (Ours)</td><td>48.45</td><td>48.85</td></tr><tr><td rowspan="3">GPT-175B</td><td rowspan="3">8</td><td rowspan="3">4</td><td rowspan="3">4</td><td rowspan="3">1</td><td>BF16</td><td>65.60</td><td>66.12</td></tr><tr><td>FP8 (TE)</td><td>69.04</td><td>69.57</td></tr><tr><td>FP8 (Ours)</td><td>38.64</td><td>40.28</td></tr></tbody></table>

Table 8: Comparing ZeRO distribution methods in terms of memory load across GPUs. Here “Min” and “Max” denote the minimum and maximum memory utilization observed across GPUs. Our FP8 ZeRO method uses less memory while achieving memory-aware load balancing.

Optimizer. We further ablate the impact of reduced precision for the variables in the AdamW optimizer. We set the BF16 mixed-precision optimizer as the baseline, since it has been widely used in existing LLM training frameworks [^33] [^60] [^40]. Tab. 6 presents the settings of reduced precision for the variables, while Fig. 8 plots the corresponding training losses. We observe that: 1) FP8 master weight induces performance degradation (see the #2a *vs.* #3 lines in Fig. 8), while FP16 can maintain accuracy as FP32 (see #2a *vs.* #0 and #1) but requiring using tensor scaling. It reveals that the master weight is precision-sensitive. This can be attributed to the master weight’s role in updating weights, which tend to exhibit small magnitudes, necessitating high precision to maintain accuracy. 2) The training loss of BF16 master weight is slightly higher than that of FP16 with a scaling factor because BF16 has fewer mantissa bits, resulting in lower precision (see #2a *vs.* #2b). 3) The second-order gradient moment is more precision-sensitive than the first-order one, because the square calculation is easy to cause underflow and leads to accuracy degradation. Utilizing FP8 for the second-order gradient moment can lead to divergent training loss (see the #4 dot in Fig. 8).

Parallelism. In our FP8 LLM training framework, we introduce FP8 low-bit convertors into sequence parallelism and tensor parallelism to reduce activation communication costs across GPUs. Here we conduct an analysis experiment to count the activation-related communication volume during GPT model training, and report the numbers in Tab. 8. It is observed that our FP8 parallel scheme results in a substantial reduction of 34% in activation-related communication costs compared to the original method utilizing BF16. Furthermore, in ZeRO distributed training, our method distributes each FP8 tensor along with its associated scaling factor as a whole, rather than partitioning the tensor into splits across GPUs. This strategy not only results in more GPU memory savings but also maintains a balanced memory load across GPUs, as demonstrated in Tab. 8.

## Related Work

Mixed-precision Training. Efficient training through reduced mixed-precision has been widely used in modern deep learning to save computing costs. While some works have taken bit-reduction to the extreme, *i.e.* 1-bit binary networks [^19] [^52], they have not been successful in maintaining model accuracy [^34]. The most practical scheme now is the FP16 half-precision method [^33], which can maintain accuracy while improving training efficiency. The computations during forward pass and back propagation use FP16 while the master weights use FP32. Since FP16 has a narrower dynamic range, FP16 mixed-precision entails loss scaling [^33] to prevent loss of accuracy. Fortunately, the need for loss scaling can be avoided by using BF16 datatype, because BF16 maintains the same dynamic range as the full-precision FP32. This results in that large model training now prefers to use BF16 mixed-precision scheme, which is more stable during training [^61] [^57] [^75].

FP8 is a natural progression from 16-bit data formats to further reducing computing cost. Early pioneering efforts in FP8 low-bit model training [^71] [^63] [^11] have largely remained at the simulation stage. Consequently, there exists a notable gap between the projected capabilities of these approaches and their actual performance on hardware [^34]. With the advent of Nvidia Hopper GPU architecture [^39], FP8 is emerging as a viable and practical data type for the next-generation low-precision training, as discussed in [^34]. At present, the Nvidia Transformer Engine (TE) [^40] serves as the primary framework for FP8 mixed-precision training. However, its support for FP8 usage remains somewhat constrained. TE’s current implementation restricts FP8 usage solely to weight computation, retaining the storage of model weights and gradient calculations with 16-bit data types. Consequently, the end-to-end speed-up, memory and communication cost savings are limited. In contrast, our work infiltrates FP8 gradient, optimizer, and distributed training into the whole progress of model training, fully unveiling the capabilities of FP8.

Large Language Models. Recent years have witnessed a substantial evolution in the field of LLMs. Autoregressive language modeling – predicting the future of a text sequence from its past – provides a simple yet powerful objective that admits formulation of numerous tasks. While there exist alternative methodologies, such as masked language modeling [^12] and permutation language modeling [^73], the autoregressive method now is more promising because of its strong performance. Following the scaling laws [^6] and the refined laws [^18], various LLMs are have been proposed, including dense models: GPT-3 [^6], Jurassic-1 [^29], Gopher [^48], Chinchilla [^18], Bloom [^57], OPT [^76] Megatron-Turing NLG [^61], PaLM [^7], LaMDA [^65], LLaMA [^67], and sparse models: GLaM [^13], and Switch transformers [^15]. Each of them has demonstrated remarkably competitive few-shot performance across a wide range of tasks at the time of their respective releases. Nonetheless, these models still encounter challenges, such as overwhelming computational requirements and the need for acquiring more high-quality training data. In this work, we delve into the utilization of low-precision techniques to mitigate the training costs, which is a crucial step for the continued expansion of language models.

Low-precision training has been widely used in LLM training to reduce compute cost. OPT [^76] and GLM [^75] utilize FP16 for forwards and backwards and FP32 for optimizer states and master weights, to reduce the GPU memory usage and improve training efficiency. Bloom [^57] find that FP16 can cause numerical instabilities and irreversible divergences, especially when training models larger than 100B parameters, because FP16’s dynamic range is limited. Consequently, Bloom and other LLMs, such as Gopher [^48] and Chinchilla [^18], adopt BF16 mixed-precision, because BF16 has a wide dynamic range that is the same as FP32. LLM training and tuning with 8-bit low-precision were not well-explored in previous works, because the hardware support for FP8 is not available before the release of Nvidia Hopper infrastructure. This work presents the first exploration of FP8 pre-training and fine-tuning for LLMs, while proposing an extremely-optimized FP8 mixed-precision scheme. We hope this work could facilitate future research in FP8 and, potentially, extend to exploring even lower precision training, such as 4-bit and 1-bit.

## Conclusion

In this work, we explore 8-bit training for LLMs. We introduce a new FP8 mixed-precision training framework, which incorporates 8-bit collective communication, optimizer, and distributed parallel training in an incremental manner. To our best knowledge, this is the first work infiltrating FP8 compute, storage and communication into the whole progress of large language model training. Extensive experiments demonstrate the proposed method effectively diminishes communication overhead and curtails memory utilization in the context of GPT model training at various scales. In future work, we plan to scale up the size and training steps of the FP8 GPT models and further train them with our 8-bit mixed-precision scheme. Moreover, we will also use the proposed FP8 scheme to train multi-modal large models, and explore low-bit deployment of LLMs on various edge devices, such as smart phones.

## Contribution and Acknowledgement

This project was initially proposed by Han Hu and Peng Cheng, who are the directional lead. Shuguang Liu served as the product lead throughout the project.

The contributions for all the co-authors are detailed as follows:

FP8 Framework: Kan Wu, Houwen Peng, Ze Liu, Peng Cheng, Han Hu

System: Yifan Xiong, Ziyue Yang, Yuxiang Yang, Guoshuai Zhao, Peng Cheng

Hardware Infrastructure: Guoshuai Zhao, Yuxiang Yang, Yifan Xiong, Peng Cheng, Shuguang Liu, Joe Chau

Data: Ruihang Li, Miaosen Zhang, Jia Ning, Chen Li, Ruizhe Wang, Houwen Peng, Han Hu

Pre-training: Yixuan Wei, Kan Wu, Ze Liu, Miaosen Zhang, Zheng Zhang, Houwen Peng, Han Hu

Alignment (SFT, RS, and RLHF): Bolin Ni, Jingcheng Hu, Yixuan Wei, Houwen Peng, Han Hu

Evaluation: Yixuan Wei, Bolin Ni, Jingcheng Hu

Product Engineering: Yuxiang Yang, Kan Wu, Yifan Xiong, Ziyue Yang, Guoshuai Zhao, Peng Cheng

We thank Eric Chung, Bita Darvish Rouhani, Yu Pei, Hyunseung Harry Yoo, Zhenghong Zhou, Gongrui Zhang, and Zhirong Wu for helpful discussions.

We thank Baining Guo and Lidong Zhou for their guidance and support for this project.

## References

## Appendix A Appendix

### A.1 FP8 Data Formats

In September 2022, NVIDIA, ARM, and Intel published FP8 specification for standardization as an interchange format for AI [^34]. The industry has moved from 32-bit precision to 16-bit, and now even 8-bit precision for AI model training. This development reflects a broader industry trend that has transitioned from high-precision to low-precision training. Notably, the proposed FP8 specification introduces two distinct data types, $E$ 5 $M$ 2 and $E$ 4 $M$ 3, which offer a trade-off between a larger range and higher precision of stored values [^41].

- $E$ 4 $M$ 3 consists of 1 sign bit, 4 exponent bits and 3 bits of mantissa. It can store values up to +/-448 and NaN.
- $E$ 5 $M$ 2 consists of 1 sign bit, 5 exponent bits and 2 bits of mantissa. It can store values up to +/-57344, +/- inf and NaN.

The FP8 format [^34] roughly follows the IEEE 754 standard. Compared to higher precision data formats such as FP16 and FP32, FP8 suffers from two kinds of representation degradation:

- *Lower representation range.* The representation range in a data format specifies the range between the maximum and minimum values that the format can accurately represent. There are two modes, a normal mode, which defines a regular range with relatively constant precision, and a subnormal mode, which extends the range to represent smaller values with lower precision. The normal range primarily depends on the number of exponent ($E$) bits, with more $E$ bits resulting in a larger normal range. On the other hand, the subnormal range is primarily influenced by the number of mantissa ($M$) bits, where an increase in $M$ bits leads to a larger subnormal range. As illustrated in Tab. 9, the representation range of FP8 is notably narrower compared to that of FP16 and FP32, especially in the case of the $S$ 1 $E$ 4 $M$ 3 sub-format ($S$ denotes the sign bit). This discrepancy represents the primary challenge when employing FP8 for training large models.
- *Lower representation precision.* The limited number of mantissa ($M$ bits) leads to quantization representation errors. Due to the considerably fewer $M$ bits in FP8, the representation precision of FP8 is substantially lower than that of FP16, as depicted in Tab. 9. This challenge stands as another significant hurdle when considering the use of FP8 for training large models.

FP8 consists of two sub-formats: $S1E4M3$ and $S1E5M2$. The former offers a narrower representation range but higher precision, while the latter provides a larger range but lower precision. These two sub-formats give users the flexibility to strike a balance between their requirements for range and precision in model training.

Table 9: Representation range and error for different data formats

<table><tbody><tr><td>Data format</td><td colspan="3">Representation Range</td><td colspan="2">Maximum Relative Error</td></tr><tr><td></td><td>Max normal</td><td>Min normal</td><td>Min subnormal</td><td>Min - Max (normal)</td><td>Min <math><semantics><mo>∼</mo> <csymbol>similar-to</csymbol> <annotation>\sim</annotation></semantics></math> Max (subnormal)</td></tr><tr><td>FP32 (S1E8M23)</td><td><math><semantics><mrow><mn>3.40</mn> <mo>×</mo> <msup><mn>10</mn> <mn>38</mn></msup></mrow> <apply><cn>3.40</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <cn>38</cn></apply></apply> <annotation>3.40\times 10^{38}</annotation></semantics></math></td><td><math><semantics><mrow><mn>1.18</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>38</mn></mrow></msup></mrow> <apply><cn>1.18</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>38</cn></apply></apply></apply> <annotation>1.18\times 10^{-38}</annotation></semantics></math></td><td><math><semantics><mrow><mn>1.40</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>45</mn></mrow></msup></mrow> <apply><cn>1.40</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>45</cn></apply></apply></apply> <annotation>1.40\times 10^{-45}</annotation></semantics></math></td><td><math><semantics><mrow><mrow><mn>1.19</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>7</mn></mrow></msup></mrow> <mo>∼</mo> <mrow><mn>5.96</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>8</mn></mrow></msup></mrow></mrow> <apply><csymbol>similar-to</csymbol> <apply><cn>1.19</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>7</cn></apply></apply></apply> <apply><cn>5.96</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>8</cn></apply></apply></apply></apply> <annotation>1.19\times 10^{-7}\sim 5.96\times 10^{-8}</annotation></semantics></math></td><td><math><semantics><mrow><mrow><mn>5.00</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>1</mn></mrow></msup></mrow> <mo>∼</mo> <mrow><mn>1.19</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>7</mn></mrow></msup></mrow></mrow> <apply><csymbol>similar-to</csymbol> <apply><cn>5.00</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>1</cn></apply></apply></apply> <apply><cn>1.19</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>7</cn></apply></apply></apply></apply> <annotation>5.00\times 10^{-1}\sim 1.19\times 10^{-7}</annotation></semantics></math></td></tr><tr><td>FP16 (S1E5M10)</td><td><math><semantics><mrow><mn>65</mn><mo>,</mo><mn>504</mn></mrow> <list><cn>65</cn> <cn>504</cn></list> <annotation>65,504</annotation></semantics></math></td><td><math><semantics><mrow><mn>6.10</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>5</mn></mrow></msup></mrow> <apply><cn>6.10</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>5</cn></apply></apply></apply> <annotation>6.10\times 10^{-5}</annotation></semantics></math></td><td><math><semantics><mrow><mn>5.96</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>8</mn></mrow></msup></mrow> <apply><cn>5.96</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>8</cn></apply></apply></apply> <annotation>5.96\times 10^{-8}</annotation></semantics></math></td><td><math><semantics><mrow><mrow><mn>9.76</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>4</mn></mrow></msup></mrow> <mo>∼</mo> <mrow><mn>4.89</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>4</mn></mrow></msup></mrow></mrow> <apply><csymbol>similar-to</csymbol> <apply><cn>9.76</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>4</cn></apply></apply></apply> <apply><cn>4.89</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>4</cn></apply></apply></apply></apply> <annotation>9.76\times 10^{-4}\sim 4.89\times 10^{-4}</annotation></semantics></math></td><td><math><semantics><mrow><mrow><mn>5.00</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>1</mn></mrow></msup></mrow> <mo>∼</mo> <mrow><mn>9.78</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>4</mn></mrow></msup></mrow></mrow> <apply><csymbol>similar-to</csymbol> <apply><cn>5.00</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>1</cn></apply></apply></apply> <apply><cn>9.78</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>4</cn></apply></apply></apply></apply> <annotation>5.00\times 10^{-1}\sim 9.78\times 10^{-4}</annotation></semantics></math></td></tr><tr><td>BF16 (S1E8M7)</td><td><math><semantics><mrow><mn>3.39</mn> <mo>×</mo> <msup><mn>10</mn> <mn>38</mn></msup></mrow> <apply><cn>3.39</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <cn>38</cn></apply></apply> <annotation>3.39\times 10^{38}</annotation></semantics></math></td><td><math><semantics><mrow><mn>1.18</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>38</mn></mrow></msup></mrow> <apply><cn>1.18</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>38</cn></apply></apply></apply> <annotation>1.18\times 10^{-38}</annotation></semantics></math></td><td><math><semantics><mrow><mn>9.18</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>41</mn></mrow></msup></mrow> <apply><cn>9.18</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>41</cn></apply></apply></apply> <annotation>9.18\times 10^{-41}</annotation></semantics></math></td><td><math><semantics><mrow><mrow><mn>7.75</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>3</mn></mrow></msup></mrow> <mo>∼</mo> <mrow><mn>3.94</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>3</mn></mrow></msup></mrow></mrow> <apply><csymbol>similar-to</csymbol> <apply><cn>7.75</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>3</cn></apply></apply></apply> <apply><cn>3.94</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>3</cn></apply></apply></apply></apply> <annotation>7.75\times 10^{-3}\sim 3.94\times 10^{-3}</annotation></semantics></math></td><td><math><semantics><mrow><mrow><mn>5.00</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>1</mn></mrow></msup></mrow> <mo>∼</mo> <mrow><mn>7.94</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>3</mn></mrow></msup></mrow></mrow> <apply><csymbol>similar-to</csymbol> <apply><cn>5.00</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>1</cn></apply></apply></apply> <apply><cn>7.94</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>3</cn></apply></apply></apply></apply> <annotation>5.00\times 10^{-1}\sim 7.94\times 10^{-3}</annotation></semantics></math></td></tr><tr><td>FP8 (S1E4M3)</td><td><math><semantics><mn>448</mn> <cn>448</cn> <annotation>448</annotation></semantics></math></td><td><math><semantics><mrow><mn>1.56</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>2</mn></mrow></msup></mrow> <apply><cn>1.56</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>2</cn></apply></apply></apply> <annotation>1.56\times 10^{-2}</annotation></semantics></math></td><td><math><semantics><mrow><mn>1.95</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>3</mn></mrow></msup></mrow> <apply><cn>1.95</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>3</cn></apply></apply></apply> <annotation>1.95\times 10^{-3}</annotation></semantics></math></td><td><math><semantics><mrow><mrow><mn>1.11</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>1</mn></mrow></msup></mrow> <mo>∼</mo> <mrow><mn>7.69</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>2</mn></mrow></msup></mrow></mrow> <apply><csymbol>similar-to</csymbol> <apply><cn>1.11</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>1</cn></apply></apply></apply> <apply><cn>7.69</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>2</cn></apply></apply></apply></apply> <annotation>1.11\times 10^{-1}\sim 7.69\times 10^{-2}</annotation></semantics></math></td><td><math><semantics><mrow><mrow><mn>5.00</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>1</mn></mrow></msup></mrow> <mo>∼</mo> <mrow><mn>1.67</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>1</mn></mrow></msup></mrow></mrow> <apply><csymbol>similar-to</csymbol> <apply><cn>5.00</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>1</cn></apply></apply></apply> <apply><cn>1.67</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>1</cn></apply></apply></apply></apply> <annotation>5.00\times 10^{-1}\sim 1.67\times 10^{-1}</annotation></semantics></math></td></tr><tr><td>FP8 (S1E5M2)</td><td><math><semantics><mrow><mn>57</mn><mo>,</mo><mn>344</mn></mrow> <list><cn>57</cn> <cn>344</cn></list> <annotation>57,344</annotation></semantics></math></td><td><math><semantics><mrow><mn>6.10</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>5</mn></mrow></msup></mrow> <apply><cn>6.10</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>5</cn></apply></apply></apply> <annotation>6.10\times 10^{-5}</annotation></semantics></math></td><td><math><semantics><mrow><mn>1.53</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>5</mn></mrow></msup></mrow> <apply><cn>1.53</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>5</cn></apply></apply></apply> <annotation>1.53\times 10^{-5}</annotation></semantics></math></td><td><math><semantics><mrow><mrow><mn>2.00</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>1</mn></mrow></msup></mrow> <mo>∼</mo> <mrow><mn>1.67</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>1</mn></mrow></msup></mrow></mrow> <apply><csymbol>similar-to</csymbol> <apply><cn>2.00</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>1</cn></apply></apply></apply> <apply><cn>1.67</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>1</cn></apply></apply></apply></apply> <annotation>2.00\times 10^{-1}\sim 1.67\times 10^{-1}</annotation></semantics></math></td><td><math><semantics><mrow><mrow><mn>5.00</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>1</mn></mrow></msup></mrow> <mo>∼</mo> <mrow><mn>5.00</mn> <mo>×</mo> <msup><mn>10</mn> <mrow><mo>−</mo> <mn>1</mn></mrow></msup></mrow></mrow> <apply><csymbol>similar-to</csymbol> <apply><cn>5.00</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>1</cn></apply></apply></apply> <apply><cn>5.00</cn> <apply><csymbol>superscript</csymbol> <cn>10</cn> <apply><cn>1</cn></apply></apply></apply></apply> <annotation>5.00\times 10^{-1}\sim 5.00\times 10^{-1}</annotation></semantics></math></td></tr></tbody></table>

### A.2 FP8 Tensor Scaling

We now discuss the underlying mechanisms for how large model training with FP8 overcomes the challenges associated with representation range and precision degradation. The key technique behind is tensor scaling, which scales the tensor values that originally locate out the representation range of a data format to its comfort zone, as visualized in Fig. 9. The pioneer scaling techniques [^33] [^37] apply a global scaling factor to the loss, such that gradients of all layers are scaled by a single adaptive factor. The utilization of the global loss scaling technique, in conjunction with various other training strategies, has facilitated the widespread adoption of FP16 mixed-precision training on V100 and A100 GPUs. Remarkably, this approach has resulted in minimal to no degradation in accuracy, particularly for small to medium-sized models [^33]. Nonetheless, when dealing with super-large models or complex tasks, such as in the training of models like DALL-E [^51], the global loss scaling technique still encounters significant underflow issues. As a consequence, block-wise [^51] and layer-wise [^64] gradient scaling are proposed.

While the global scaling technique enables almost no accuracy drop for FP16 training (with a range of \[5.96E-8, 6.55E+4\]), the fine-grained per-tensor scaling will enable stable model training using even shallower range by FP8 (with a range of \[1.95E-3, 448\] for $E$ 4 $M$ 3 and a range of \[1.53E-5, 5.73E+4\] for $E$ 5 $M$ 2). Fig. 9 shows that the representation range of FP8 has been large enough to deal with general model training. In the per-tensor scaling technique, various strategies are available for choosing the suitable scaling factor for a given FP8 tensor. Two common approaches are “just-in-time scaling" and “delayed scaling" [^41].

- *Just-in-time scaling*. This strategy involves determining the scaling factor based on the maximum absolute value (amax) of the tensor being generated. However, in practical applications, this approach is often infeasible because it necessitates multiple passes through the data. Specifically, the operator first produces and writes out the output in higher precision, then calculates the maximum absolute value of the output, and finally applies this scaling factor to all values to obtain the final FP8 output. This process introduces a significant amount of overhead, which can substantially reduce the benefits of using FP8.
- *Delayed scaling*. This strategy involves selecting the scaling factor based on the maximum absolute values observed in a certain number of preceding iterations. This approach allows for the full performance benefits of FP8 computation but necessitates the storage of a history of maximum values as additional parameters of the FP8 operators.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2310.18313/assets/figures/tensorscaling.png)

Refer to caption

### A.3 Pre-training Data

Tab. 10 presents an overview of our collected data sources along with the corresponding sampling weights employed in pre-training. The arXiv and StackExchange subsets are collected from Redpajama [^53], while BookCorpus2 [^78], Books3 [^45], DM-Math [^56], Gutenberg [^47], HackerNews <sup>6</sup>, NIH ExPorter <sup>7</sup>, OpenSubtitles [^66], and USPTO <sup>8</sup> subsets are extracted from The Pile [^16]. The Wikipedia data is downloaded from HuggingFace [^20]. We use the 20220301 dump, including 24 languages: bg, ca, cs, da, de, en, es, fr, hi, hr, hu, it, jp, ko, nl, pl, pt, ro, ru, sl, sr, sv, uk, zh.

We pre-process 11 CommonCrawl snapshots, ranging from 2018 to 2023, with the CCNet pipeline [^72]. This process involves data deduplication at the line level, followed by language identification utilizing a fastText linear classifier [^22] to eliminate non-English pages. A filtering mechanism based on an n-gram language model is employed to exclude low-quality content. In addition, we train a linear classifier [^53] to distinguish documents similar to Wikipedia pages from randomly sampled CommonCrawl documents. Documents not classified as resembling Wikipedia are excluded. Finally, we perform fuzzy deduplication [^26] across all the processed snapshots from CommonCrawl.

We collect Python code data from Github using a repository list provided by Bing indexing [^3]. The cleaning of the code data includes three steps. First, we remove control characters, except for $\textbackslash t$ and $\textbackslash n$. Next, we remove copyright comments in the code. An alphanumeric rate filter is then applied, removing lines with a rate below 0.5 if they are comments, and discarding the entire file if its overall alphanumeric rate is less than 0.98. Files with less than 5 lines or a maximum line length exceeding 1,000 characters are also discarded. Also, files with an average line length of more than 100 characters are discarded. Lastly, a pattern search is conducted to identify key Python keywords (*e.g.*, import, from, def, class, if, for, try, etc.) within the code. Files containing less than 3 instances of these keywords are eliminated. This comprehensive process ensures that the remaining Python code data is of high quality and suitable for use in academic research. We additionally add Python code from Stack [^24], and perform fuzzy deduplication within all the collected Python code.

<table><tbody><tr><td>Dataset</td><td>Sampling prop.</td><td>Epochs</td><td>Training Tokens (Billion)</td></tr><tr><td colspan="4">Web Crawls</td></tr><tr><td>CommonCrawl</td><td>51.71%</td><td>0.16</td><td>51.71</td></tr><tr><td>C4</td><td>25.56%</td><td>0.16</td><td>25.56</td></tr><tr><td>OpenWebText</td><td>2.73%</td><td>0.16</td><td>2.73</td></tr><tr><td colspan="4">Technical & Science content</td></tr><tr><td>arXiv</td><td>1.54%</td><td>0.05</td><td>1.54</td></tr><tr><td>StackExchange</td><td>1.42%</td><td>0.08</td><td>1.42</td></tr><tr><td>DM-Math</td><td>0.39%</td><td>0.05</td><td>0.39</td></tr><tr><td>USPTO</td><td>0.52%</td><td>0.05</td><td>0.52</td></tr><tr><td>NIH ExPorter</td><td>0.04%</td><td>0.05</td><td>0.04</td></tr><tr><td colspan="4">Programming Languages</td></tr><tr><td>Python</td><td>4.50%</td><td>0.11</td><td>4.50</td></tr><tr><td colspan="4">Other Curated Sources</td></tr><tr><td>Wikipedia</td><td>4.50%</td><td>0.16</td><td>4.50</td></tr><tr><td>Books</td><td>4.50%</td><td>0.09</td><td>4.50</td></tr><tr><td>News</td><td>2.00%</td><td>0.11</td><td>2.00</td></tr><tr><td>Dialogue</td><td>2.00%</td><td>0.27</td><td>2.00</td></tr><tr><td colspan="3">Total</td><td>100.00</td></tr></tbody></table>

Table 10: Pre-training data. For each subset we list the sampling weight, number of epochs, and training tokens. Books data includes BookCorpus2 [^78], Books3 [^45], and Gutenberg [^47]. Dialogue data includes HackerNews and OpenSubtitles [^66]. For experiments with a training token count of less than 100 billion, we employ the same sampling proportion.

[^1]: Rohan Anil, Andrew M Dai, Orhan Firat, Melvin Johnson, Dmitry Lepikhin, Alexandre Passos, Siamak Shakeri, Emanuel Taropa, Paige Bailey, Zhifeng Chen, et al. Palm 2 technical report. *arXiv preprint arXiv:2305.10403*, 2023.

[^2]: Yuntao Bai, Andy Jones, Kamal Ndousse, Amanda Askell, Anna Chen, Nova DasSarma, Dawn Drain, Stanislav Fort, Deep Ganguli, Tom Henighan, et al. Training a helpful and harmless assistant with reinforcement learning from human feedback. *arXiv preprint arXiv:2204.05862*, 2022.

[^3]: Microsoft Bing. Bing webmaster tools. 2022. URL [https://www.bing.com/webmasters/](https://www.bing.com/webmasters/).

[^4]: Yonatan Bisk, Rowan Zellers, Jianfeng Gao, Yejin Choi, et al. Piqa: Reasoning about physical commonsense in natural language. In *Proceedings of the AAAI conference on artificial intelligence*, volume 34, pages 7432–7439, 2020.

[^5]: Sidney Black, Stella Biderman, Eric Hallahan, Quentin Anthony, Leo Gao, Laurence Golding, Horace He, Connor Leahy, Kyle McDonell, Jason Phang, et al. Gpt-neox-20b: An open-source autoregressive language model. In *Proceedings of BigScience Episode# 5–Workshop on Challenges & Perspectives in Creating Large Language Models*, pages 95–136, 2022.

[^6]: Tom Brown, Benjamin Mann, Nick Ryder, Melanie Subbiah, Jared D Kaplan, Prafulla Dhariwal, Arvind Neelakantan, Pranav Shyam, Girish Sastry, Amanda Askell, Sandhini Agarwal, Ariel Herbert-Voss, Gretchen Krueger, Tom Henighan, Rewon Child, Aditya Ramesh, Daniel Ziegler, Jeffrey Wu, Clemens Winter, Chris Hesse, Mark Chen, Eric Sigler, Mateusz Litwin, Scott Gray, Benjamin Chess, Jack Clark, Christopher Berner, Sam McCandlish, Alec Radford, Ilya Sutskever, and Dario Amodei. Language models are few-shot learners. In *Advances in Neural Information Processing Systems*, volume 33, pages 1877–1901. Curran Associates, Inc., 2020.

[^7]: Aakanksha Chowdhery, Sharan Narang, Jacob Devlin, Maarten Bosma, Gaurav Mishra, Adam Roberts, Paul Barham, Hyung Won Chung, Charles Sutton, Sebastian Gehrmann, Parker Schuh, Kensen Shi, Sasha Tsvyashchenko, Joshua Maynez, Abhishek B Rao, Parker Barnes, Yi Tay, Noam M. Shazeer, Vinodkumar Prabhakaran, Emily Reif, Nan Du, Benton C. Hutchinson, Reiner Pope, James Bradbury, Jacob Austin, Michael Isard, Guy Gur-Ari, Pengcheng Yin, Toju Duke, Anselm Levskaya, Sanjay Ghemawat, Sunipa Dev, Henryk Michalewski, Xavier García, Vedant Misra, Kevin Robinson, Liam Fedus, Denny Zhou, Daphne Ippolito, David Luan, Hyeontaek Lim, Barret Zoph, Alexander Spiridonov, Ryan Sepassi, David Dohan, Shivani Agrawal, Mark Omernick, Andrew M. Dai, Thanumalayan Sankaranarayana Pillai, Marie Pellat, Aitor Lewkowycz, Erica Oliveira Moreira, Rewon Child, Oleksandr Polozov, Katherine Lee, Zongwei Zhou, Xuezhi Wang, Brennan Saeta, Mark Díaz, Orhan Firat, Michele Catasta, Jason Wei, Kathleen S. Meier-Hellstern, Douglas Eck, Jeff Dean, Slav Petrov, and Noah Fiedel. PaLM: Scaling language modeling with pathways. *ArXiv*, abs/2204.02311, 2022.

[^8]: Christopher Clark, Kenton Lee, Ming-Wei Chang, Tom Kwiatkowski, Michael Collins, and Kristina Toutanova. BoolQ: Exploring the surprising difficulty of natural yes/no questions. In *Proceedings of the 2019 Conference of the North American Chapter of the Association for Computational Linguistics: Human Language Technologies, Volume 1 (Long and Short Papers)*, pages 2924–2936, Minneapolis, Minnesota, June 2019. Association for Computational Linguistics. doi: 10.18653/v1/N19-1300. URL [https://aclanthology.org/N19-1300](https://aclanthology.org/N19-1300).

[^9]: Peter Clark, Isaac Cowhey, Oren Etzioni, Tushar Khot, Ashish Sabharwal, Carissa Schoenick, and Oyvind Tafjord. Think you have solved question answering? try arc, the ai2 reasoning challenge. *arXiv:1803.05457v1*, 2018.

[^10]: Tri Dao, Dan Fu, Stefano Ermon, Atri Rudra, and Christopher Ré. Flashattention: Fast and memory-efficient exact attention with io-awareness. *Advances in Neural Information Processing Systems*, 35:16344–16359, 2022.

[^11]: Tim Dettmers, Mike Lewis, Sam Shleifer, and Luke Zettlemoyer. 8-bit optimizers via block-wise quantization. In *International Conference on Learning Representations*, 2021.

[^12]: Jacob Devlin, Ming-Wei Chang, Kenton Lee, and Kristina Toutanova. BERT: Pre-training of deep bidirectional transformers for language understanding. In *Proceedings of the 2019 Conference of the North American Chapter of the Association for Computational Linguistics: Human Language Technologies, Volume 1 (Long and Short Papers)*, pages 4171–4186, Minneapolis, Minnesota, June 2019. Association for Computational Linguistics. doi: 10.18653/v1/N19-1423. URL [https://www.aclweb.org/anthology/N19-1423](https://www.aclweb.org/anthology/N19-1423).

[^13]: Nan Du, Yanping Huang, Andrew M Dai, Simon Tong, Dmitry Lepikhin, Yuanzhong Xu, Maxim Krikun, Yanqi Zhou, Adams Wei Yu, Orhan Firat, et al. Glam: Efficient scaling of language models with mixture-of-experts. In *International Conference on Machine Learning*, pages 5547–5569. PMLR, 2022.

[^14]: Yann Dubois, Xuechen Li, Rohan Taori, Tianyi Zhang, Ishaan Gulrajani, Jimmy Ba, Carlos Guestrin, Percy Liang, and Tatsunori B Hashimoto. Alpacafarm: A simulation framework for methods that learn from human feedback. *arXiv preprint arXiv:2305.14387*, 2023.

[^15]: William Fedus, Barret Zoph, and Noam Shazeer. Switch transformers: Scaling to trillion parameter models with simple and efficient sparsity. *The Journal of Machine Learning Research*, 23(1):5232–5270, 2022.

[^16]: Leo Gao, Stella Biderman, Sid Black, Laurence Golding, Travis Hoppe, Charles Foster, Jason Phang, Horace He, Anish Thite, Noa Nabeshima, et al. The pile: An 800gb dataset of diverse text for language modeling. *arXiv preprint arXiv:2101.00027*, 2020.

[^17]: Aaron Gokaslan and Vanya Cohen. Openwebtext corpus. [http://Skylion007.github.io/OpenWebTextCorpus](http://skylion007.github.io/OpenWebTextCorpus), 2019.

[^18]: Jordan Hoffmann, Sebastian Borgeaud, Arthur Mensch, Elena Buchatskaya, Trevor Cai, Eliza Rutherford, Diego de Las Casas, Lisa Anne Hendricks, Johannes Welbl, Aidan Clark, et al. Training compute-optimal large language models. *arXiv:2203.15556*, 2022.

[^19]: Itay Hubara, Matthieu Courbariaux, Daniel Soudry, Ran El-Yaniv, and Yoshua Bengio. Binarized neural networks. *Advances in neural information processing systems*, 29, 2016.

[^20]: HuggingFace. wikipedia - datasets at hugging face. 2022. URL [https://huggingface.co/datasets/wikipedia](https://huggingface.co/datasets/wikipedia).

[^21]: Andrei Ivanov, Nikoli Dryden, Tal Ben-Nun, Shigang Li, and Torsten Hoefler. Data movement is all you need: A case study on optimizing transformers. *Proceedings of Machine Learning and Systems*, 3:711–732, 2021.

[^22]: Armand Joulin, Édouard Grave, Piotr Bojanowski, and Tomáš Mikolov. Bag of tricks for efficient text classification. In *Proceedings of the 15th Conference of the European Chapter of the Association for Computational Linguistics: Volume 2, Short Papers*, pages 427–431, 2017.

[^23]: Diederik P. Kingma and Jimmy Ba. Adam: A method for stochastic optimization. In *3rd International Conference on Learning Representations*, San Diego, CA, 2015. URL [http://arxiv.org/abs/1412.6980](http://arxiv.org/abs/1412.6980).

[^24]: Denis Kocetkov, Raymond Li, LI Jia, Chenghao Mou, Yacine Jernite, Margaret Mitchell, Carlos Muñoz Ferrandis, Sean Hughes, Thomas Wolf, Dzmitry Bahdanau, et al. The stack: 3 tb of permissively licensed source code. *Transactions on Machine Learning Research*, 2022.

[^25]: Andreas Köpf, Yannic Kilcher, Dimitri von Rütte, Sotiris Anagnostidis, Zhi-Rui Tam, Keith Stevens, Abdullah Barhoum, Nguyen Minh Duc, Oliver Stanley, Richárd Nagyfi, et al. Openassistant conversations–democratizing large language model alignment. *arXiv preprint arXiv:2304.07327*, 2023.

[^26]: Katherine Lee, Daphne Ippolito, Andrew Nystrom, Chiyuan Zhang, Douglas Eck, Chris Callison-Burch, and Nicholas Carlini. Deduplicating training data makes language models better. In *Proceedings of the 60th Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers)*, pages 8424–8445, 2022.

[^27]: Shenggui Li, Hongxin Liu, Zhengda Bian, Jiarui Fang, Haichen Huang, Yuliang Liu, Boxiang Wang, and Yang You. Colossal-ai: A unified deep learning system for large-scale parallel training. In *Proceedings of the 52nd International Conference on Parallel Processing*, pages 766–775, 2023a.

[^28]: Xuechen Li, Tianyi Zhang, Yann Dubois, Rohan Taori, Ishaan Gulrajani, Carlos Guestrin, Percy Liang, and Tatsunori B. Hashimoto. Alpacaeval: An automatic evaluator of instruction-following models. [https://github.com/tatsu-lab/alpaca\_eval](https://github.com/tatsu-lab/alpaca_eval), 2023b.

[^29]: Opher Lieber, Or Sharir, Barak Lenz, and Yoav Shoham. Jurassic-1: Technical details and evaluation. *White Paper. AI21 Labs*, 1, 2021.

[^30]: Yinhan Liu, Myle Ott, Naman Goyal, Jingfei Du, Mandar Joshi, Danqi Chen, Omer Levy, Mike Lewis, Luke Zettlemoyer, and Veselin Stoyanov. Roberta: A robustly optimized bert pretraining approach. *arXiv preprint arXiv:1907.11692*, 2019.

[^31]: Ze Liu, Han Hu, Yutong Lin, Zhuliang Yao, Zhenda Xie, Yixuan Wei, Jia Ning, Yue Cao, Zheng Zhang, Li Dong, et al. Swin transformer v2: Scaling up capacity and resolution. In *Proceedings of the IEEE/CVF conference on computer vision and pattern recognition*, pages 12009–12019, 2022.

[^32]: Ilya Loshchilov and Frank Hutter. Decoupled weight decay regularization. In *International Conference on Learning Representations*, 2018.

[^33]: Paulius Micikevicius, Sharan Narang, Jonah Alben, Gregory Diamos, Erich Elsen, David Garcia, Boris Ginsburg, Michael Houston, Oleksii Kuchaiev, Ganesh Venkatesh, et al. Mixed precision training. *arXiv preprint arXiv:1710.03740*, 2017.

[^34]: Paulius Micikevicius, Dusan Stosic, Neil Burgess, Marius Cornea, Pradeep Dubey, Richard Grisenthwaite, Sangwon Ha, Alexander Heinecke, Patrick Judd, John Kamalu, et al. Fp8 formats for deep learning. *arXiv preprint arXiv:2209.05433*, 2022.

[^35]: Microsoft. Azure high-performance computing. 2023. URL [https://azure.microsoft.com/en-us/solutions/high-performance-computing](https://azure.microsoft.com/en-us/solutions/high-performance-computing).

[^36]: Todor Mihaylov, Peter Clark, Tushar Khot, and Ashish Sabharwal. Can a suit of armor conduct electricity? a new dataset for open book question answering. In *Proceedings of the 2018 Conference on Empirical Methods in Natural Language Processing*, pages 2381–2391, 2018.

[^37]: Nvidia. Apex. 2018. URL [https://nvidia.github.io/apex](https://nvidia.github.io/apex).

[^38]: Nvidia. The nvidia collective communications library. 2020. URL [https://developer.nvidia.com/nccl](https://developer.nvidia.com/nccl).

[^39]: Nvidia. Nvidia h100 tensor core gpu architecture. 2022a. URL [https://resources.nvidia.com/en-us-tensor-core](https://resources.nvidia.com/en-us-tensor-core).

[^40]: Nvidia. Nvidia transformer engine. 2022b. URL [https://docs.nvidia.com/deeplearning/transformer-engine/index.html](https://docs.nvidia.com/deeplearning/transformer-engine/index.html).

[^41]: Nvidia. Using fp8 with transformer engine. 2022c. URL [https://docs.nvidia.com/deeplearning/transformer-engine/user-guide/examples/fp8\_primer.html](https://docs.nvidia.com/deeplearning/transformer-engine/user-guide/examples/fp8_primer.html).

[^42]: OpenAI. Model index for researchers. 2022. URL [https://platform.openai.com/docs/model-index-for-researchers](https://platform.openai.com/docs/model-index-for-researchers).

[^43]: OpenAI. GPT-4 technical report. *arXiv preprint arXiv:2303.08774*, 2023.

[^44]: Denis Paperno, Germán Kruszewski, Angeliki Lazaridou, Ngoc-Quan Pham, Raffaella Bernardi, Sandro Pezzelle, Marco Baroni, Gemma Boleda, and Raquel Fernández. The lambada dataset: Word prediction requiring a broad discourse context. In *Proceedings of the 54th Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers)*, pages 1525–1534, 2016.

[^45]: Shawn Presser. Books3. [https://twitter.com/theshawwn/status/1320282149329784833](https://twitter.com/theshawwn/status/1320282149329784833), 2020.

[^46]: Alec Radford, Jeff Wu, Rewon Child, David Luan, Dario Amodei, and Ilya Sutskever. Language models are unsupervised multitask learners. 2019.

[^47]: Jack W Rae, Anna Potapenko, Siddhant M Jayakumar, Chloe Hillier, and Timothy P Lillicrap. Compressive transformers for long-range sequence modelling. In *International Conference on Learning Representations*, 2019.

[^48]: Jack W Rae, Sebastian Borgeaud, Trevor Cai, Katie Millican, Jordan Hoffmann, Francis Song, John Aslanides, Sarah Henderson, Roman Ring, Susannah Young, et al. Scaling language models: Methods, analysis & insights from training gopher. *arXiv preprint arXiv:2112.11446*, 2021.

[^49]: Colin Raffel, Noam Shazeer, Adam Roberts, Katherine Lee, Sharan Narang, Michael Matena, Yanqi Zhou, Wei Li, and Peter J Liu. Exploring the limits of transfer learning with a unified text-to-text transformer. *The Journal of Machine Learning Research*, 21(1):5485–5551, 2020.

[^50]: Samyam Rajbhandari, Jeff Rasley, Olatunji Ruwase, and Yuxiong He. Zero: Memory optimizations toward training trillion parameter models. In *SC20: International Conference for High Performance Computing, Networking, Storage and Analysis*, pages 1–16. IEEE, 2020.

[^51]: Aditya Ramesh, Mikhail Pavlov, Gabriel Goh, Scott Gray, Chelsea Voss, Alec Radford, Mark Chen, and Ilya Sutskever. Zero-shot text-to-image generation. In *International Conference on Machine Learning*, pages 8821–8831. PMLR, 2021.

[^52]: Mohammad Rastegari, Vicente Ordonez, Joseph Redmon, and Ali Farhadi. Xnor-net: Imagenet classification using binary convolutional neural networks. In *European conference on computer vision*, pages 525–542. Springer, 2016.

[^53]: Redpajama. Redpajama-data: an open source recipe to reproduce llama training dataset. 2023. URL [https://github.com/togethercomputer/RedPajama-Data](https://github.com/togethercomputer/RedPajama-Data).

[^54]: Melissa Roemmele, Cosmin Adrian Bejan, and Andrew S. Gordon. Choice of plausible alternatives: An evaluation of commonsense causal reasoning. In *AAAI Spring Symposium*, 2011.

[^55]: Keisuke Sakaguchi, Ronan Le Bras, Chandra Bhagavatula, and Yejin Choi. Winogrande: An adversarial winograd schema challenge at scale. *Communications of the ACM*, 64(9):99–106, 2021.

[^56]: David Saxton, Edward Grefenstette, Felix Hill, and Pushmeet Kohli. Analysing mathematical reasoning abilities of neural models. In *International Conference on Learning Representations*, 2018.

[^57]: Teven Le Scao, 388 Authors, and Thomas Wolf. BLOOM: A 176B-parameter open-access multilingual language model. *ArXiv*, abs/2211.05100, 2022.

[^58]: John Schulman, Filip Wolski, Prafulla Dhariwal, Alec Radford, and Oleg Klimov. Proximal policy optimization algorithms. *arXiv preprint arXiv:1707.06347*, 2017.

[^59]: ShareGPT. Openchat: Advancing open-source language models with imperfect data. 2023. URL [https://sharegpt.com/](https://sharegpt.com/).

[^60]: Mohammad Shoeybi, Mostofa Patwary, Raul Puri, Patrick LeGresley, Jared Casper, and Bryan Catanzaro. Megatron-lm: Training multi-billion parameter language models using model parallelism. *arXiv preprint arXiv:1909.08053*, 2019.

[^61]: Shaden Smith, Mostofa Patwary, Brandon Norick, Patrick LeGresley, Samyam Rajbhandari, Jared Casper, Zhun Liu, Shrimai Prabhumoye, George Zerveas, Vijay Korthikanti, et al. Using deepspeed and megatron to train megatron-turing nlg 530b, a large-scale generative language model. *arXiv preprint arXiv:2201.11990*, 2022.

[^62]: Jianlin Su, Yu Lu, Shengfeng Pan, Ahmed Murtadha, Bo Wen, and Yunfeng Liu. Roformer: Enhanced transformer with rotary position embedding. *arXiv preprint arXiv:2104.09864*, 2021.

[^63]: Xiao Sun, Jungwook Choi, Chia-Yu Chen, Naigang Wang, Swagath Venkataramani, Vijayalakshmi Viji Srinivasan, Xiaodong Cui, Wei Zhang, and Kailash Gopalakrishnan. Hybrid 8-bit floating point (hfp8) training and inference for deep neural networks. *Advances in neural information processing systems*, 32, 2019.

[^64]: Xiao Sun, Naigang Wang, Chia-Yu Chen, Jiamin Ni, Ankur Agrawal, Xiaodong Cui, Swagath Venkataramani, Kaoutar El Maghraoui, Vijayalakshmi Viji Srinivasan, and Kailash Gopalakrishnan. Ultra-low precision 4-bit training of deep neural networks. *Advances in Neural Information Processing Systems*, 33:1796–1807, 2020.

[^65]: Romal Thoppilan, Daniel De Freitas, Jamie Hall, Noam Shazeer, Apoorv Kulshreshtha, Heng-Tze Cheng, Alicia Jin, Taylor Bos, Leslie Baker, Yu Du, et al. Lamda: Language models for dialog applications. *arXiv preprint arXiv:2201.08239*, 2022.

[^66]: Jörg Tiedemann. Finding alternative translations in a large corpus of movie subtitle. In *Proceedings of the Tenth International Conference on Language Resources and Evaluation (LREC’16)*, pages 3518–3522, 2016.

[^67]: Hugo Touvron, Thibaut Lavril, Gautier Izacard, Xavier Martinet, Marie-Anne Lachaux, Timothée Lacroix, Baptiste Rozière, Naman Goyal, Eric Hambro, Faisal Azhar, et al. Llama: Open and efficient foundation language models. *arXiv preprint arXiv:2302.13971*, 2023.

[^68]: Trieu H Trinh and Quoc V Le. A simple method for commonsense reasoning. *arXiv preprint arXiv:1806.02847*, 2018.

[^69]: Ashish Vaswani, Noam Shazeer, Niki Parmar, Jakob Uszkoreit, Llion Jones, Aidan N Gomez, Łukasz Kaiser, and Illia Polosukhin. Attention is all you need. In *Advances in Neural Information Processing Systems*, pages 5998–6008. Curran Associates, Inc., 2017. URL [http://papers.nips.cc/paper/7181-attention-is-all-you-need.pdf](http://papers.nips.cc/paper/7181-attention-is-all-you-need.pdf).

[^70]: VicunaTeam. Vicuna: An open-source chatbot impressing gpt-4 with 90quality. 2023. URL [https://lmsys.org/blog/2023-03-30-vicuna/](https://lmsys.org/blog/2023-03-30-vicuna/).

[^71]: Naigang Wang, Jungwook Choi, Daniel Brand, Chia-Yu Chen, and Kailash Gopalakrishnan. Training deep neural networks with 8-bit floating point numbers. *Advances in neural information processing systems*, 31, 2018.

[^72]: Guillaume Wenzek, Marie-Anne Lachaux, Alexis Conneau, Vishrav Chaudhary, Francisco Guzmán, Armand Joulin, and Edouard Grave. Ccnet: Extracting high quality monolingual datasets from web crawl data. *arXiv preprint arXiv:1911.00359*, 2019.

[^73]: Zhilin Yang, Zihang Dai, Yiming Yang, Jaime Carbonell, Russ R Salakhutdinov, and Quoc V Le. XLNet: Generalized autoregressive pretraining for language understanding. In H. Wallach, H. Larochelle, A. Beygelzimer, F. d'Alché-Buc, E. Fox, and R. Garnett, editors, *Advances in Neural Information Processing Systems*, volume 32. Curran Associates, Inc., 2019. URL [https://proceedings.neurips.cc/paper/2019/file/dc6a7e655d7e5840e66733e9ee67cc69-Paper.pdf](https://proceedings.neurips.cc/paper/2019/file/dc6a7e655d7e5840e66733e9ee67cc69-Paper.pdf).

[^74]: Rowan Zellers, Ari Holtzman, Yonatan Bisk, Ali Farhadi, and Yejin Choi. HellaSwag: Can a machine really finish your sentence? In *Proceedings of the 57th Annual Meeting of the Association for Computational Linguistics*, pages 4791–4800, Florence, Italy, July 2019. Association for Computational Linguistics. doi: 10.18653/v1/P19-1472. URL [https://aclanthology.org/P19-1472](https://aclanthology.org/P19-1472).

[^75]: Aohan Zeng, Xiao Liu, Zhengxiao Du, Zihan Wang, Hanyu Lai, Ming Ding, Zhuoyi Yang, Yifan Xu, Wendi Zheng, Xiao Xia, et al. Glm-130b: An open bilingual pre-trained model. In *The Eleventh International Conference on Learning Representations*, 2022.

[^76]: Susan Zhang, Stephen Roller, Naman Goyal, Mikel Artetxe, Moya Chen, Shuohui Chen, Christopher Dewan, Mona Diab, Xian Li, Xi Victoria Lin, et al. Opt: Open pre-trained transformer language models. *arXiv preprint arXiv:2205.01068*, 2022.

[^77]: Lianmin Zheng, Wei-Lin Chiang, Ying Sheng, Siyuan Zhuang, Zhanghao Wu, Yonghao Zhuang, Zi Lin, Zhuohan Li, Dacheng Li, Eric Xing, et al. Judging llm-as-a-judge with mt-bench and chatbot arena. *arXiv preprint arXiv:2306.05685*, 2023.

[^78]: Yukun Zhu, Ryan Kiros, Rich Zemel, Ruslan Salakhutdinov, Raquel Urtasun, Antonio Torralba, and Sanja Fidler. Aligning books and movies: Towards story-like visual explanations by watching movies and reading books. In *Proceedings of the IEEE international conference on computer vision*, pages 19–27, 2015.