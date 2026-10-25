---
title: "The Complexity Trap: Simple Observation Masking Is as Efficient as LLM Summarization for Agent Context Management"
source: "https://ar5iv.labs.arxiv.org/html/2508.21433"
author:
published:
created: 2026-10-26
description: "llm (llm)-based agents solve complex tasks through iterative reasoning, exploration, and tool-use, a process that can result in long, expensive context histories. While state-of-the-art  se (se) agents like OpenHands o…"
tags:
  - "clippings"
---
Tobias Lindenbauer Affiliation: JetBrains Research Affiliation: School of Computation, Information and Technology, Technical University of Munichtobias.lindenbauer@jetbrains.com    Igor Slinko Affiliation: JetBrains Research    Ludwig Felder Affiliation: School of Computation, Information and Technology, Technical University of Munichtobias.lindenbauer@jetbrains.com    Egor Bogomolov Affiliation: JetBrains Research    Yaroslav Zharov Affiliation: JetBrains Research

###### Abstract

[llm](#id1) ([llm](#id1))-based agents solve complex tasks through iterative reasoning, exploration, and tool-use, a process that can result in long, expensive context histories. While state-of-the-art [se](#id5) ([se](#id5)) agents like OpenHands or Cursor use [llm](#id1) -based summarization to tackle this issue, it is unclear whether the increased complexity offers tangible performance benefits compared to simply omitting older observations. We present a systematic comparison of these strategies within SWE-agent on SWE-bench Verified across five diverse model configurations. We find that a simple observation-masking strategy halves cost relative to a raw agent while matching, and sometimes slightly exceeding, the solve rate of LLM summarization. For example, with Qwen3-Coder 480B, masking improves solve rate from 53.8% (raw agent) to 54.8%, while remaining competitive with summarization at a lower cost. These results suggest that, at least within SWE-agent on SWE-bench Verified, the most effective and efficient context management can be the simplest. We release code and data for reproducibility.<sup>1</sup> <sup>2</sup>

## 1 Introduction

The ambition to create autonomous agents that can independently handle complex [se](#id5) tasks is rapidly becoming a reality. These agents, powered by [llm](#id1), typically operate in an iterative loop [^36] [^27], at each turn they reason about the current state, devise a plan, and execute a tool (e.g., read a file, run tests). The output, or observation, from this tool is then added to the agent’s context for the next turn, extending its problem-solving trajectory (Figure 3). This agentic loop acts as a powerful test-time scaling mechanism [^24] [^17] [^30], utilizing the reasoning capabilities [^29] of [llm](#id1) at each turn while grounding them through environment responses.

However, this iterative context expansion presents a fundamental tradeoff between cost and capability, or effectiveness and efficiency. As the agent’s trajectory grows, calls to the [llm](#id1) become prohibitively expensive due to token-based pricing, and inefficient due to the quadratic attention complexity in the wide-spread Transformer architecture [^26]. More critically, even with context windows exceeding 1M tokens, [llm](#id1) suffer from the "lost in the middle" problem [^16]. While [llm](#id1) can process large context windows, they cannot properly make use of relevant information buried within their vast context [^18] [^7]. This challenge is acutely amplified in the [se](#id5) domain, where tool observations are notoriously verbose and noisy [^15]. A single action can yield thousands of tokens, whether from reading an entire source file, running a recursive directory listing, or a lengthy test suite log. Concretely, observation tokens make up around $84\%$ of an average SWE-agent turn [^32] (Figure 1) in our preliminary experiments (Section D.4) on SWE-bench Lite-50 [^11] [^3]. Due to this, targeting environment observations explicitly provides a strong baseline for [llm](#id1) -agent context management.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2508.21433/assets/data/token_type_distr.png)

Figure 1: Environment observation tokens dominate the context window of an se agent’s trajectory.

Without any context management strategy targeting cost-efficiency, we find that agent costs can more than double (Table 1), making context management not just an optimization but an economic necessity. In fact, our results show that any of the discussed management strategies are preferable to none, as they consistently reduce costs and often improve performance. This raises a critical question: which strategy offers the best trade-off?

A natural baseline approach directly targets verbose environment observations through observation masking strategies that explicitly omit tool outputs. State-of-the-art systems like SWE-agent [^32] and SWE-Search [^2] have adopted such relatively simple approaches. In parallel, an increasingly popular and more sophisticated solution is to prompt an [llm](#id1) to perform trajectory summarization, replacing parts of the agent’s history with condensed summaries (Figure 3). This approach is adopted by prominent open-source and proprietary [se](#id5) agents like OpenHands [^28] and Cursor [^9]. Open-source implementations of both the observation masking and [llm](#id1) summarization approaches rely on heuristic triggers such as fixed context window sizes or turn thresholds. Thus, the key difference is whether they discard or condense old context. Despite the critical impact of this choice on agent cost and performance, the relative trade-offs between these approaches remain largely unexplored.

In this work, we present a systematic comparison focused on the efficiency of context management strategies within SWE-agent [^32]. For this, we analyze the performance of [llm](#id1) -based trajectory summarization and a simple observation masking baseline for context management with respect to efficiency (cost in USD) and effectiveness (solve rate on the challenging and industry-standard SWE-bench Verified [^5] benchmark). We implement an [llm](#id1) -based summarization in SWE-agent [^32] and utilize the OpenHands-style prompt (Figure 11) to enable a controlled, within-scaffold comparison. We evaluate these strategies within the SWE-agent [^32] scaffold across a diverse spectrum of experimental conditions, including model families, model sizes, and reasoning regimes.

We find that the simple observation-masking approach more than halves the cost compared to the raw agent while matching and sometimes exceeding the performance of the [llm](#id1) -based summarization and the raw agent within SWE-agent. A finding that challenges current trends in the field and demonstrates that in the quest for more capable [llm](#id1) agents, complexity is not always the answer, sometimes the simplest solution is also the most effective.

Figure 2: The effectiveness versus efficiency tradeoff for context management strategies within SWE-agent [^32] on SWE-bench Verified [^5]. The plot compares solve rate (y-axis, $\uparrow$) against average cost per trajectory (x-axis, $\downarrow$) for different model configurations. We test each configuration with three strategies: raw agent (baseline, ⚫), [llm](#id1) -Summary (◼), and observation masking (▲). Across all models, the observation masking strategy consistently occupies the most efficient frontier, achieving solve rates competitive with, and sometimes superior to, the [llm](#id1) -Summary strategy. With Qwen3-Coder 480B [^25] [^8], the best-performing model in our experiments, Observation Masking is not only 52% cheaper than the Raw Agent baseline but also improves on the solve rate by 2.6 %. Moreover, it even reduces the cost per instance compared to [llm](#id1) -Summary by $0.03 ($15 across 500 instances).

## 2 Related Work

Current [se](#id5) agent research mostly focuses on improving the effectiveness of [se](#id5) agents by scaling training data [^10] [^21] [^33], selecting the most promising of multiple attempts [^21] [^10] [^37] [^2] [^1] [^31], providing execution-free or execution-based feedback to the agent through critics [^2] [^21] [^10] [^37] [^23], or enhancing the agent’s planning capabilities through explicit search strategies [^2] [^1]. While these methods improve solve rate, they come at the cost of increased inference costs and thus reduced efficiency. Efficient context management for [se](#id5) agents, on the other hand, has thus far received little attention. In this work, we investigate whether complex summarization strategies are necessary for efficient context management in [se](#id5) agents. For this we experiment with SWE-agent [^32]. Moreover, we show that without any such context management the instance cost routinely more than double, positioning research on efficient agents as pivotal for an economically feasible, large-scale deployment of [se](#id5) agents.

Recently, several works have conducted in-depth analyses on the effect of the context size on the [llm](#id1) performance [^16] [^7] [^18]. These consistently show that [llm](#id1) increasingly struggle to effectively utilize the context provided with increased context size. Despite the critical importance of context management for agent performance and deployment costs, existing work treats it as an implementation detail rather than a first-class research question. A recent exception, MEM1 [^38], explores dynamic state management for multi-hop QA [^34] [^13] [^12] and web navigation [^35] tasks. However, the authors do not compare to omission-based approaches. Furthermore, the benchmarks used result in relatively short trajectories (hundreds of tokens) [^38] compared to [se](#id5) agent trajectories that routinely are orders of magnitude larger [^15].

## 3 Experimental Configuration

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2508.21433/assets/data/management-strategies.png)

Figure 3: Overview of the context management strategies evaluated in our work. Box heights indicate the number of tokens in that portion of a typical trajectory. (a) The baseline ReAct-style 36 trajectory, where the context grows with each action-observation pair. (b) LLM-based summarization condenses older turns into a running summary and preserving a few recent turns in full (e.g., OpenHands 28 ). (c) Observation masking replaces observations older than a fixed window of N turns (here, M=1) with a placeholder (e.g., SWE-agent 32

In our experiments we cover one popular, representative approach for observation omission and [llm](#id1) -based summarization: (1) environment observation masking through a rolling window [^32] [^2], and (2) prompt-based [llm](#id1) summarization [^28] [^9]. In the following, we will define our chosen approaches in detail.

### 3.1 Context Management Strategies

#### Raw baseline agent.

The agent scaffolds we investigate use either ReAct [^36] or CodeAct [^27] as an agent framework. In these frameworks, the agent’s history, or trajectory, is a sequence of interactions with an environment. We formalize the trajectory at the end of turn $t-1$, denoted $\tau_{t-1}$, as a sequence of tokens:

$$
\tau_{t-1}=(o_{sys},o_{user},(r_{1},a_{1},o_{1}),\dots,(r_{t-1},a_{t-1},o_{t-1}))
$$

where $o_{sys}$ and $o_{user}$ are the immutable system and user prompts that initialize the task (Figure 3a). We define a turn $T_{i}=(r_{i},a_{i},o_{i})$ consisting of reasoning $r_{i}$, action $a_{i}$ and observation $o_{i}$ as the atomic unit of agent-environment interaction at turn $i$. This allows us to compactly express $\tau_{t-1}$ as:

$$
\tau_{t-1}=(o_{sys},o_{user},T_{1},\dots,T_{t-1})
$$

At the beginning of turn $t$, the agent policy $\pi$, typically an [llm](#id1), conditions on the history $\tau_{t-1}$ to generate the next reasoning and action pair: $(r_{t},a_{t})\sim\pi(\cdot|\tau_{t-1})$. Without any context management strategy, $\tau$ grows with each turn, leading to excessive computational costs and eventual context length limitations.

#### Environment observation masking.

This strategy manages the context size by selectively condensing past environment observations while preserving the full history of agent reasoning and actions. We define an observation masking function, $f_{mask}(\tau_{t-1},M)$, that takes the full trajectory $\tau_{t-1}$ and an integer window size $M$ as input. The function produces a condensed trajectory, $\tau^{\prime}_{t-1}$, by replacing environment observations older than the window with a placeholder (Figure 3b).

Formally, given the trajectory $\tau_{t-1}=(o_{sys},o_{user},T_{1},\dots,T_{t-1})$, the transformed trajectory is:

$$
\tau^{\prime}_{t-1}=(o_{sys},o_{user},(r_{1},a_{1},o^{\prime}_{1}),\dots,(r_{t-1},a_{t-1},o^{\prime}_{t-1}))
$$

where the observation at each turn $i$, denoted $o^{\prime}_{i}$, is conditionally defined as:

$$
o^{\prime}_{i}=\begin{cases}p_{i}&\text{if }i<t-M\\
o_{i}&\text{if }i\geq t-M\end{cases}
$$

Here, $p_{i}$ is a placeholder text representing the masked observation, such as “Previous 8 lines omitted for brevity.”. In the following turns, the agent [llm](#id1) then conditions on this condensed history $\tau^{\prime}_{t-1}$ to produce $(r_{t},a_{t})$. This approach, following SWE-agent [^32], retains the complete reasoning chain while reducing distant observation fidelity. While this strategy reduces the speed at which the tokens in $\tau$ grow, it does not solve the issue of indefinite growth.

#### Summaries.

This strategy uses a "summarizer [llm](#id1) " to condense the trajectory, which we denote as $\pi^{\prime}$. In contrast to $f_{mask}$, the goal of this strategy is to maintain salient information of the processed turns. It is controlled by two parameters $N$ and $M$. $N$ regulates how many turns the agent will accumulate at once, and $M$ regulates how many trailing turns should be left unaltered. We trigger summarization when the accumulated turns since the last summary reach $N+M$.

To help us define this approach, we will introduce two variables, $t_{last}$ and $s_{last}$. Let $t_{last}$ be the index of the final turn included in the most recent summary ($t_{last}=0$ at step 0). Let $s_{last}$ be the summary performed at index $t_{last}$ ($s_{last}=o_{user}$ at step 0). Then, we define the summarization as follows. First, we slice the trajectory to obtain the turns eligible for the summarization, containing the last summary $s_{last}$ and all turns between this summary and the $M$ to the last turn $\mathcal{T}_{sum}=(s_{last},T_{t_{last}+1},\dots,T_{t-1-M})$. Then, we generate a new summary $s_{t}$ by prompting $\pi^{\prime}$ with a summary instruction $o_{si}$ and the relevant trajectory slice $\mathcal{T}_{sum}$.

$$
\begin{split}s_{t}&\sim\pi^{\prime}(\cdot|o_{si},\mathcal{T}_{sum})\\
t_{last}&=t-1-M\end{split}
$$

Finally, the history provided to the main policy $\pi$ is reconstructed into a new condensed trajectory, $\tau^{\prime}_{t-1}$:

$$
\tau^{\prime}_{t-1}=(o_{sys},o_{user},s_{t},T_{t-M},\dots,T_{t-1})
$$

On the next turn, the agent [llm](#id1) conditions on this new, compact history: $(r_{t},a_{t})\sim\pi(\cdot|\tau^{\prime}_{t-1})$. This method ensures the trajectory’s growth is not only slowed, but bounded, as older interactions are recursively folded into an evolving summary.

### 3.2 Experimental Configuration

We conduct a rigorous within-scaffold study using SWE-agent [^32], covering distinct (1) model families, (2) model sizes, (3) model licenses, (4) reasoning regimes (thinking and non-thinking). Concretely, we use Qwen3-32B <sup>3</sup> [^25] in thinking and non-thinking mode with a context window of $122$ K tokens using YaRN [^22], Qwen3-Coder-480B-A35B-Instruct-FP8 <sup>4</sup> [^25] [^8], and Gemini 2.5 Flash <sup>5</sup> [^6] in thinking and non-thinking mode. We conduct all experiments with on SWE-agent [^32] on SWE-bench Verified [^5] unless otherwise specified. We run all our experiments on a shared cluster of eight NVIDIA H200 GPUs, each equipped with 141 GB of HBM, and a total of 8 TB local disk storage. For the Qwen-32B [^25] models we use two H200 GPUs, and 15 SWE-agent [^32] inference workers. We choose a conservative number of workers, because we may encounter long trajectories with context sizes $>100$ K tokens and must account for this in our KV-cache estimates. For Qwen3-Coder 480B [^25] [^8], we use all eight GPUs and 35 inference workers. We use vLLM [^14] to serve the Qwen models on our cluster. In our experiments with Gemini we use eight inference workers on SWE-agent [^32] due to quota restrictions. For further details see Appendix A.

Our preliminary experiments (Section D.4) indicate that the number of turns in the trajectory may influence the behavior of the context management strategies we investigate. Due to this, we experiment with long trajectories and thus set the turn limit to 250 in our main experiments. For the Observation Masking strategy we use a rolling window size of $M=10$ in our main experiments, because it resulted in the best performance with SWE-agent [^32] in our experiments (Section D.1). The [llm](#id1) -Summary strategy we implement in SWE-agent [^32] uses a slightly modified version of the OpenHands-style prompt (Appendix B). In contrast to OpenHands [^28], we summarize 21 turns at once $N=21$ and retain only the last ten $M=10$ turns. Besides aligning the number of unaltered tail turns for the Observation Masking and [llm](#id1) -Summary strategies, we also found that $M=10$ offered the best performance for the [llm](#id1) -Summary strategy in our experiments (Section D.2). For the agent model we use a temperature of 0.8, and for the summary model a temperature of zero. In contrast to experiments with Qwen3-32B [^25] thinking, we restrict the thinking budget of Gemini 2.5 Flash to zero or 800 tokens (denoted as thinking) due to cost constraints.

## 4 Main Results

Our main experiments within SWE-agent [^32] evaluate three context management strategies, with results summarized in Table 1. The results reveal a central finding that holds robustly across the diverse conditions we tested, spanning model families (Qwen [^25] [^8], Gemini [^6]), sizes (32B, 480B), licenses (open-source, proprietary), and reasoning regimes (with and without thinking). The context management strategies evaluated significantly reduce costs, without significantly reducing solve rate performance. For further details see Appendix C.

### 4.1 The Universal Benefit of Context Management

Our first and most foundational finding, reinforcing the motivation of this study, is that context management is not merely an optimization but a necessity. As shown in Table 1, leaving the agent’s context to grow unchecked (the Raw Agent baseline) is consistently the most expensive strategy. In all five of our experimental configurations, both Observation Masking and [llm](#id1) -Summary drastically reduce the cost per instance, in most cases by more than 50%.

Furthermore, this efficiency does not necessarily come at the cost of performance. In three of our five setups, the most efficient strategy also achieved a higher solve rate than the Raw Agent baseline. This demonstrates that beyond a certain point, more context becomes a liability rather than an asset, aligning with the "lost in the middle" problem [^16]. Therefore it is critical to question which approach offers the best trade-off between effectiveness and efficiency.

Table 1: Comparison of context management strategies with 95% bootstrap confidence intervals. We report change and significance (†) compared to the Raw Agent. We report Solve Rate (effectiveness, $\uparrow$) and Instance Cost (efficiency, $\downarrow$). For each model, we boldface the best-performing context management strategy for each metric. All experiments use SWE-agent [^32] on SWE-bench Verified [^5]. Further details in Appendix C.

<table><thead><tr><th>Model</th><th>Strategy</th><th>Solve Rate (%,<math><semantics><mo>↑</mo> <annotation>\uparrow</annotation></semantics></math>)</th><th>Instance Cost ($,<math><semantics><mo>↓</mo> <annotation>\downarrow</annotation></semantics></math>)</th></tr></thead><tbody><tr><th rowspan="3">Qwen3-32B</th><th>Raw Agent</th><td>17.0  <math><semantics><mrow><mo>±</mo> <mn>3.3</mn></mrow> <annotation>\pm 3.3</annotation></semantics></math></td><td>1.12  <math><semantics><mrow><mo>±</mo> <mn>0.18</mn></mrow> <annotation>\pm 0.18</annotation></semantics></math></td></tr><tr><th>Observation Masking</th><td>15.0  <math><semantics><mrow><mo>±</mo> <mn>3.1</mn></mrow> <annotation>\pm 3.1</annotation></semantics></math> (-11.8%)</td><td>0.55  <math><semantics><mrow><mo>±</mo> <mn>0.09</mn></mrow> <annotation>\pm 0.09</annotation></semantics></math> (-50.9%) <sup>†</sup></td></tr><tr><th>LLM-Summary</th><td>16.0  <math><semantics><mrow><mo>±</mo> <mn>3.3</mn></mrow> <annotation>\pm 3.3</annotation></semantics></math> (-5.9%)</td><td>0.50  <math><semantics><mrow><mo>±</mo> <mn>0.07</mn></mrow> <annotation>\pm 0.07</annotation></semantics></math> (-55.4%) <sup>†</sup></td></tr><tr><th rowspan="3">Qwen3-32B (thinking)</th><th>Raw Agent</th><td>23.0  <math><semantics><mrow><mo>±</mo> <mn>3.7</mn></mrow> <annotation>\pm 3.7</annotation></semantics></math></td><td>0.51  <math><semantics><mrow><mo>±</mo> <mn>0.07</mn></mrow> <annotation>\pm 0.07</annotation></semantics></math></td></tr><tr><th>Observation Masking</th><td>24.6  <math><semantics><mrow><mo>±</mo> <mn>3.8</mn></mrow> <annotation>\pm 3.8</annotation></semantics></math> (+7.0%)</td><td>0.46  <math><semantics><mrow><mo>±</mo> <mn>0.05</mn></mrow> <annotation>\pm 0.05</annotation></semantics></math> (-9.8%)</td></tr><tr><th>LLM-Summary</th><td>24.8  <math><semantics><mrow><mo>±</mo> <mn>3.9</mn></mrow> <annotation>\pm 3.9</annotation></semantics></math> (+7.3%)</td><td>0.51  <math><semantics><mrow><mo>±</mo> <mn>0.06</mn></mrow> <annotation>\pm 0.06</annotation></semantics></math> (0.0%)</td></tr><tr><th rowspan="3">Qwen3-Coder 480B</th><th>Raw Agent</th><td>53.4  <math><semantics><mrow><mo>±</mo> <mn>4.3</mn></mrow> <annotation>\pm 4.3</annotation></semantics></math></td><td>1.29  <math><semantics><mrow><mo>±</mo> <mn>0.26</mn></mrow> <annotation>\pm 0.26</annotation></semantics></math></td></tr><tr><th>Observation Masking</th><td>54.8  <math><semantics><mrow><mo>±</mo> <mn>4.4</mn></mrow> <annotation>\pm 4.4</annotation></semantics></math> (+2.6%)</td><td>0.61  <math><semantics><mrow><mo>±</mo> <mn>0.06</mn></mrow> <annotation>\pm 0.06</annotation></semantics></math> (-52.7%) <sup>†</sup></td></tr><tr><th>LLM-Summary</th><td>53.8  <math><semantics><mrow><mo>±</mo> <mn>4.2</mn></mrow> <annotation>\pm 4.2</annotation></semantics></math> (+0.7%)</td><td>0.64  <math><semantics><mrow><mo>±</mo> <mn>0.06</mn></mrow> <annotation>\pm 0.06</annotation></semantics></math> (-50.4%) <sup>†</sup></td></tr><tr><th rowspan="3">Gemini 2.5 Flash</th><th>Raw Agent</th><td>32.8  <math><semantics><mrow><mo>±</mo> <mn>4.1</mn></mrow> <annotation>\pm 4.1</annotation></semantics></math></td><td>0.41  <math><semantics><mrow><mo>±</mo> <mn>0.08</mn></mrow> <annotation>\pm 0.08</annotation></semantics></math></td></tr><tr><th>Observation Masking</th><td>35.6  <math><semantics><mrow><mo>±</mo> <mn>4.2</mn></mrow> <annotation>\pm 4.2</annotation></semantics></math> (+8.5%)</td><td>0.18  <math><semantics><mrow><mo>±</mo> <mn>0.03</mn></mrow> <annotation>\pm 0.03</annotation></semantics></math> (-56.1%) <sup>†</sup></td></tr><tr><th>LLM-Summary</th><td>36.0  <math><semantics><mrow><mo>±</mo> <mn>4.1</mn></mrow> <annotation>\pm 4.1</annotation></semantics></math> (+9.8%)</td><td>0.24  <math><semantics><mrow><mo>±</mo> <mn>0.04</mn></mrow> <annotation>\pm 0.04</annotation></semantics></math> (-41.5%) <sup>†</sup></td></tr><tr><th rowspan="3">Gemini 2.5 Flash (thinking)</th><th>Raw Agent</th><td>40.4  <math><semantics><mrow><mo>±</mo> <mn>4.3</mn></mrow> <annotation>\pm 4.3</annotation></semantics></math></td><td>0.56  <math><semantics><mrow><mo>±</mo> <mn>0.10</mn></mrow> <annotation>\pm 0.10</annotation></semantics></math></td></tr><tr><th>Observation Masking</th><td>36.4  <math><semantics><mrow><mo>±</mo> <mn>4.2</mn></mrow> <annotation>\pm 4.2</annotation></semantics></math> (-9.9%) <sup>†</sup></td><td>0.24  <math><semantics><mrow><mo>±</mo> <mn>0.04</mn></mrow> <annotation>\pm 0.04</annotation></semantics></math> (-57.1%) <sup>†</sup></td></tr><tr><th>LLM-Summary</th><td>31.4  <math><semantics><mrow><mo>±</mo> <mn>4.0</mn></mrow> <annotation>\pm 4.0</annotation></semantics></math> (-22.3%) <sup>†</sup></td><td>0.25  <math><semantics><mrow><mo>±</mo> <mn>0.05</mn></mrow> <annotation>\pm 0.05</annotation></semantics></math> (-55.4%) <sup>†</sup></td></tr></tbody></table>

### 4.2 Observation Masking: Dominant Efficiency with Minimal Complexity

Having established the clear need for context management, we now turn to the central finding of our work: the surprising power of simplicity. As we can see in Table 1, in four out of five experimental setups Observation Masking yielded the lowest cost per instance. It achieves this by drastically reducing the number of environment observation tokens processed in each agent turn without incurring the computational overhead of a separate summarization call. This is highly effective, because the agent’s context is heavily skewed towards environment observations in [se](#id5) (Figure 1).

Let’s consider our best-performing model Qwen3-Coder 480B [^25] [^8]. For it, the Observation Masking strategy achieves a cost of $0.61 per instance, a 52.7% reduction compared to the Raw Agent baseline ($1.29) and a tangible improvement over [llm](#id1) -Summary ($0.64) that amounts to 15$ in savings across 500 instances in SWE-bench Verified [^5]. This demonstrates that even for state-of-the-art models, a simple omission strategy is a powerful baseline for efficient context management.

### 4.3 Challenging the Need for Complex Summaries

Beyond consistently being the cheapest option, Observation Masking proves to be remarkably effective at maintaining high solve rates. This directly challenges the assumption that complex, semantic summarization is necessary to preserve critical information from an agent’s trajectory.

In fact, Observation Masking not only competes with [llm](#id1) -Summary, but can outperform it. With the Qwen3-Coder 480B [^25] [^8] model, Observation Masking achieved a solve rate of 54.8%, a slight improvement over the [llm](#id1) -Summary’s 53.8%. Similarly, for Gemini 2.5 Flash [^6] (thinking), it outperformed [llm](#id1) -Summary by five percentage points. In the cases where LLM-Summary did perform better, such as with Gemini 2.5 Flash, the margin was minimal (36.0% vs. 35.6%). This indicates that Observation Masking consistently performs on-par with, or better than the [llm](#id1) -Summary strategy. Moreover, using one of the two tested context management strategies outperformed Raw Agent in three out of five configurations. Highlighting that increased context can eventually result in noise for [llm](#id1) [^15] [^7] [^16].

The implication is clear: the most recent context is often sufficient for code-generating agents. Retaining the entire history, or even a sophisticated summary of it, may not be the most effective use of the model’s limited context window and our research budget.

### 4.4 The Trajectory Elongation Effect

A key question arising from our main results in Table 1 is why the [llm](#id1) -Summary strategy is less cost-effective than the Observation Masking strategy for all experiments, except Qwen3-32B. Our analysis reveals that this partially stems from an unexpected "trajectory elongation" side-effect of the [llm](#id1) -Summary context management strategy.

For this, we analyze the distribution of turns per instance in Figure 4. We note that [llm](#id1) -Summary leads to longer mean trajectory lengths for both Qwen3-Coder 480B and Gemini 2.5 Flash. For Gemini 2.5 Flash the mean trajectory length using [llm](#id1) -Summary is 52 turns, which is a $15\%$ increase over the mean trajectory length of the Observation Masking (44 turns) and $4\%$ over that of the Raw Agent (50 turns). Likewise, for Qwen3-Coder 480B, we observe an increase of the mean trajectory length by $15\%$ compared to the Raw Agent and $13\%$ compared to the Observation Masking strategy.

Notably, this trend of turn elongation translates directly to the efficiency of a strategy observed in Figure 2 and Table 1. The only experiment for which the [llm](#id1) -Summary strategy proved more cost-efficient is Qwen3-32B. In this case, the Observation Masking strategy, rather than the [llm](#id1) -Summary strategy led to a $13\%$ increase in mean trajectory length compared to the Raw Agent. This indicates that context summaries act as a reinforcing signal, encouraging the agent to keep going. This in turn results in trajectory elongation that diminishes the efficiency gained through the bounded context achieved by [llm](#id1) -Summary (Section 3.1).

Figure 4: Impact of context management strategies on trajectory length. Box plots show the distribution of trajectory lengths (in turns) across different strategies within SWE-agent [^32]. [llm](#id1) -Summary consistently leads to longer trajectories, suggesting they mask failure signals that would otherwise prompt earlier termination. The star indicates the mean trajectory length.

## 5 Discussion

Our results consistently demonstrate that a simple Observation Masking strategy is not only more cost-effective but also surprisingly competitive in solve rate compared to more complex [llm](#id1) -Summary methods within SWE-agent [^32]. This challenges the prevailing assumption that sophisticated, semantic context compression is always superior. The discovery of the "trajectory elongation" effect (Section 4.4) provides a compelling explanation for this phenomenon: the smoothed nature of a summary may inadvertently mask signs of a failing trajectory, encouraging the agent to persist in unproductive loops. We hypothesize the mechanism generalizes across scaffolds that trigger summarization by turn count or context thresholds, but we leave verification to future work.

### 5.1 Critic-Enhanced Summarization

A natural follow-up question is whether the [llm](#id1) -Summary strategy could be improved by making the summarization process more intelligent. We explore this by enhancing our [llm](#id1) -Summary strategy with execution-free feedback, a technique that has shown promise in scaling test-time compute for [se](#id5) agents [^21] [^10] [^37] [^2]. In this approach, the [llm](#id1) simultaneously generates a summary and critical analysis of the trajectory, incorporating both into the compressed context. This is akin to providing reflections within a single rollout, instead of across multiple rollouts [^23].

Testing on 150 samples from SWE-bench Verified [^5] using SWE-agent [^32], this critic-enhanced approach using the prompt presented in Figures 12, 13 and 14 showed no improvement in solve rate over standard [llm](#id1) -Summary. More concerning, we observed exacerbated trajectory elongation patterns, with critic-enhanced runs producing even longer trajectories than standard summarization. This is perhaps unsurprising: the critic’s reflections naturally encourage the agent to explore alternative solution paths, try additional debugging strategies, or reconsider its approach, all of which translate to more turns, thus driving cost and reducing efficiency gains.

This finding reinforces our central insight about trajectory elongation. While execution-free feedback aims to improve agent decision-making, it paradoxically increases computational costs by extending exploration. The critic’s guidance, rather than helping the agent efficiently recognize dead ends, provides additional avenues to pursue, further delaying termination. Furthermore, this increased cost, does not lead to increased downstream performance. This suggests that effective memory systems for AI agents require fundamental rethinking: simply adding more sophisticated feedback to summaries may compound rather than solve the efficiency challenges we identify. See Section D.3.

### 5.2 The Costs of Summarization

A closer examination of the cost breakdown reveals that the efficiency gap between strategies stems from two complementary effects. First, the "trajectory elongation" effect discussed in Section 4.4, and second, the costs of generating the summary.

As shown in Table 2, the direct API cost of generating summaries accounts for up to 7.2% of the total instance cost. Importantly, these summarization calls are particularly expensive because each requires processing a unique sequence of turns, limiting cache reuse to the [llm](#id1) -Summary system prompt (Figure 11). This poses a critical limitation given that, several modern [llm](#id1) APIs (e.g., Gemini) offer substantially cheaper cache hits than cache misses (often on the order of 3–4× cheaper). Once we subtract these summarization costs from the total, the efficiency difference between [llm](#id1) -Summary and Observation Masking largely disappears for most experiments. This indicates that the summarization API calls themselves constitute a significant portion of the efficiency gap. Nonetheless, the more complex [llm](#id1) -Summary strategy is still unable to consistently outperform the remarkably strong Observation Masking strategy on cost-efficiency, highlighting the need for more sophisticated strategies, potentially combining the strengths of these two approaches.

Table 2: Mean Instance [llm](#id1) -Summary Cost per Model. [llm](#id1) summary generation API costs explain a portion of the cost-efficiency difference between the [llm](#id1) -Summary and Observation Masking strategy.

| Model | Instance [llm](#id1) -Summary Cost ($) | Proportional Cost (%) |
| --- | --- | --- |
| Qwen3-32B | $0.0143$ | $2.86$ |
| Qwen3-32B (thinking) | $0.0033$ | $0.65$ |
| Qwen3-Coder 480B | $0.0439$ | $7.20$ |
| Gemini 2.5 Flash | $0.0161$ | $6.71$ |
| Gemini 2.5 Flash (thinking) | $0.0131$ | $5.24$ |

## 6 Limitations

While our study provides a rigorous evaluation of context management strategies, its scope has three main limitations.

First, we conduct our experiments within a single scaffold (SWE-agent [^32]). We do not claim generality across agent scaffolds. Preliminary results on SWE-bench Verified-50 [^3] [^5] using OpenHands [^28] indicate scaffold-specific nuance to our findings. See Appendix E for details. We leave systematic evaluation of further agents, such as OpenHands [^28], to future work.

Second, our experiments are situated exclusively within the [se](#id5) domain, using the SWE-bench [^11] benchmark. This domain is characterized by long, verbose tool outputs, a condition that naturally favors the efficiency of Observation Masking. Consequently, our findings on the superiority of this strategy may not generalize to domains where agent-environment interactions are more succinct.

Finally, both the [llm](#id1) -Summary and Observation Masking strategy, are triggered by simple, non-adaptive heuristics. Observation Masking employs a fixed-size rolling window that is agnostic to the relevance or staleness of past observations (e.g., retaining a file’s content after it has been modified). Similarly, [llm](#id1) -Summary operates on a fixed turn-based schedule, ignoring semantic boundaries or agent sub-goals. Our study also does not explore hybrid strategies that could potentially combine the benefits of both approaches.

## 7 Conclusion

Our comprehensive study on context management strategies within SWE-agent [^32] establishes the critical need for context management for economically feasible, and environmentally sustainable deployment. Foregoing context management more than doubles execution costs. Yet, contrary to the trend toward increasingly complex summarization, a simple Observation Masking strategy consistently performs on par with a sophisticated [llm](#id1) -Summary approach in cost efficiency and solve rate within SWE-agent [^32].

Our analysis uncovers a fundamental insight: [llm](#id1) -based summarization can inadvertently encourage “trajectory elongation,” where agents persist on unproductive paths, an effect especially pronounced with powerful models like Gemini 2.5 Flash or Qwen3-Coder 480B. Moreover, summary-generation [llm](#id1) calls account for a nontrivial fraction of per-instance cost for the strongest models (5–7%), eroding the net efficiency of these approaches. These observations point to several concrete next steps: evaluate whether these trends generalize to other agent scaffolds (e.g., OpenHands [^28]); move beyond chronological compression toward data-driven selection of what to retain; explore hybrid schemes that primarily use Observation Masking but invoke selective [llm](#id1) summarization at truly critical junctures; learn triggers that decide when summarization is beneficial versus harmful (e.g., detecting looping or plateaus); evaluate the generality of these strategies in domains beyond [se](#id5); and reduce summarization overhead via lightweight specialist summarizers or distillation.

Overall, our work demonstrates that in the quest for more capable AI agents, complexity is not always the answer. Sometimes the simplest solution is also the most effective.

## References

## Appendix A Experimental Configuration

To compute the instance cost reported in our experiments, we distinguish between proprietary models we access through the API, and models we host locally on our infrastructure. For the Gemini [^6] experiments, which we access through the Vertex AI API, we report the cost returned by the API. For Qwen [^25] [^8] experiments, we self-host the models as described in Section 3.2. To compute the cost, we use the observed per turn token usage and post-hoc compute the cost based on the the official Alibaba API pricing <sup>6</sup>. Note that in contrast to the Vertex AI platform, the official Alibaba API pricing for the Qwen3-32B model, does not distinguish between cache hit and miss input tokens. This leads to an inflated cost of the Qwen3-32B experiments. We refer readers interested in the cost structure of our experiments under different pricing schemes to out HuggingFace dataset <sup>7</sup>, in which we release all experimental data for our main experiments and our repository <sup>8</sup>.

## Appendix B Summary Prompt

We share our prompt template for summary generation in Figure 11. Compared to OpenHands [^28], we remove the part of the prompt that aims to handle summary generation for tasks outside the [se](#id5) domain, since out work is purely focused on the [se](#id5) domain. In addition to the system prompt shown in Figure 11, we provide a joint critic-summarization prompt in Figures 12, 13 and 14. We discuss the effects of generating execution-free in Section D.3 and Section 5.

## Appendix C Detailed Main Results

In this section, we provide further data on which we base our confidence intervals and significance indicators in Table 1. Table 3 presents the full asymmetric confidence intervals underlying our main results. The symmetric intervals in Table 1 are derived by averaging the asymmetric bounds: e.g., $17.0_{-3.2}^{+3.4}$ yields $17.0\pm 3.3$.

Table 3: Comparison of context management strategies with 95% bootstrap confidence intervals, showing asymmetry. We use † to indicate significance compared to the Raw Agent. We report Solve Rate (effectiveness, $\uparrow$) and Instance Cost (efficiency, $\downarrow$). For each model, we boldface the best-performing context management strategy for each metric (relative to the Raw Agent baseline). Change is reported relative to the Raw Agent baseline. All experiments use SWE-agent [^32] on SWE-bench Verified [^5].

<table><thead><tr><th>Model</th><th>Strategy</th><th>Solve Rate (%,<math><semantics><mo>↑</mo> <annotation>\uparrow</annotation></semantics></math>)</th><th>Instance Cost ($,<math><semantics><mo>↓</mo> <annotation>\downarrow</annotation></semantics></math>)</th></tr></thead><tbody><tr><th rowspan="3">Qwen3-32B</th><th>Raw Agent</th><td>17.0  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>3.4</mn></mrow> <mrow><mo>−</mo> <mn>3.2</mn></mrow></mmultiscripts> <annotation>{}_{-3.2}^{+3.4}</annotation></semantics></math></td><td>1.12  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>0.18</mn></mrow> <mrow><mo>−</mo> <mn>0.17</mn></mrow></mmultiscripts> <annotation>{}_{-0.17}^{+0.18}</annotation></semantics></math></td></tr><tr><th>Observation Masking</th><td>15.0  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>3.2</mn></mrow> <mrow><mo>−</mo> <mn>3.0</mn></mrow></mmultiscripts> <annotation>{}_{-3.0}^{+3.2}</annotation></semantics></math> (-11.8%)</td><td>0.55  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>0.09</mn></mrow> <mrow><mo>−</mo> <mn>0.08</mn></mrow></mmultiscripts> <annotation>{}_{-0.08}^{+0.09}</annotation></semantics></math> (-50.9%) <sup>†</sup></td></tr><tr><th>LLM-Summary</th><td>16.0  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>3.4</mn></mrow> <mrow><mo>−</mo> <mn>3.2</mn></mrow></mmultiscripts> <annotation>{}_{-3.2}^{+3.4}</annotation></semantics></math> (-5.9%)</td><td>0.50  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>0.07</mn></mrow> <mrow><mo>−</mo> <mn>0.06</mn></mrow></mmultiscripts> <annotation>{}_{-0.06}^{+0.07}</annotation></semantics></math> (-55.4%) <sup>†</sup></td></tr><tr><th rowspan="3">Qwen3-32B (thinking)</th><th>Raw Agent</th><td>23.0  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>3.8</mn></mrow> <mrow><mo>−</mo> <mn>3.6</mn></mrow></mmultiscripts> <annotation>{}_{-3.6}^{+3.8}</annotation></semantics></math></td><td>0.51  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>0.07</mn></mrow> <mrow><mo>−</mo> <mn>0.06</mn></mrow></mmultiscripts> <annotation>{}_{-0.06}^{+0.07}</annotation></semantics></math></td></tr><tr><th>Observation Masking</th><td>24.6  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>3.8</mn></mrow> <mrow><mo>−</mo> <mn>3.8</mn></mrow></mmultiscripts> <annotation>{}_{-3.8}^{+3.8}</annotation></semantics></math> (+7.0%)</td><td>0.46  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>0.05</mn></mrow> <mrow><mo>−</mo> <mn>0.05</mn></mrow></mmultiscripts> <annotation>{}_{-0.05}^{+0.05}</annotation></semantics></math> (-9.8%)</td></tr><tr><th>LLM-Summary</th><td>24.8  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>4.0</mn></mrow> <mrow><mo>−</mo> <mn>3.8</mn></mrow></mmultiscripts> <annotation>{}_{-3.8}^{+4.0}</annotation></semantics></math> (+7.3%)</td><td>0.51  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>0.06</mn></mrow> <mrow><mo>−</mo> <mn>0.06</mn></mrow></mmultiscripts> <annotation>{}_{-0.06}^{+0.06}</annotation></semantics></math> (0.0%)</td></tr><tr><th rowspan="3">Qwen3-Coder 480B</th><th>Raw Agent</th><td>53.4  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>4.4</mn></mrow> <mrow><mo>−</mo> <mn>4.2</mn></mrow></mmultiscripts> <annotation>{}_{-4.2}^{+4.4}</annotation></semantics></math></td><td>1.29  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>0.28</mn></mrow> <mrow><mo>−</mo> <mn>0.24</mn></mrow></mmultiscripts> <annotation>{}_{-0.24}^{+0.28}</annotation></semantics></math></td></tr><tr><th>Observation Masking</th><td>54.8  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>4.4</mn></mrow> <mrow><mo>−</mo> <mn>4.4</mn></mrow></mmultiscripts> <annotation>{}_{-4.4}^{+4.4}</annotation></semantics></math> (+2.6%)</td><td>0.61  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>0.06</mn></mrow> <mrow><mo>−</mo> <mn>0.05</mn></mrow></mmultiscripts> <annotation>{}_{-0.05}^{+0.06}</annotation></semantics></math> (-52.7%) <sup>†</sup></td></tr><tr><th>LLM-Summary</th><td>53.8  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>4.2</mn></mrow> <mrow><mo>−</mo> <mn>4.2</mn></mrow></mmultiscripts> <annotation>{}_{-4.2}^{+4.2}</annotation></semantics></math> (+0.7%)</td><td>0.64  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>0.06</mn></mrow> <mrow><mo>−</mo> <mn>0.05</mn></mrow></mmultiscripts> <annotation>{}_{-0.05}^{+0.06}</annotation></semantics></math> (-50.4%) <sup>†</sup></td></tr><tr><th rowspan="3">Gemini 2.5 Flash</th><th>Raw Agent</th><td>32.8  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>4.2</mn></mrow> <mrow><mo>−</mo> <mn>4.0</mn></mrow></mmultiscripts> <annotation>{}_{-4.0}^{+4.2}</annotation></semantics></math></td><td>0.41  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>0.08</mn></mrow> <mrow><mo>−</mo> <mn>0.07</mn></mrow></mmultiscripts> <annotation>{}_{-0.07}^{+0.08}</annotation></semantics></math></td></tr><tr><th>Observation Masking</th><td>35.6  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>4.2</mn></mrow> <mrow><mo>−</mo> <mn>4.2</mn></mrow></mmultiscripts> <annotation>{}_{-4.2}^{+4.2}</annotation></semantics></math> (+8.5%)</td><td>0.18  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>0.03</mn></mrow> <mrow><mo>−</mo> <mn>0.02</mn></mrow></mmultiscripts> <annotation>{}_{-0.02}^{+0.03}</annotation></semantics></math> (-56.1%) <sup>†</sup></td></tr><tr><th>LLM-Summary</th><td>36.0  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>4.2</mn></mrow> <mrow><mo>−</mo> <mn>4.0</mn></mrow></mmultiscripts> <annotation>{}_{-4.0}^{+4.2}</annotation></semantics></math> (+9.8%)</td><td>0.24  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>0.04</mn></mrow> <mrow><mo>−</mo> <mn>0.04</mn></mrow></mmultiscripts> <annotation>{}_{-0.04}^{+0.04}</annotation></semantics></math> (-41.5%) <sup>†</sup></td></tr><tr><th rowspan="3">Gemini 2.5 Flash (thinking)</th><th>Raw Agent</th><td>40.4  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>4.2</mn></mrow> <mrow><mo>−</mo> <mn>4.4</mn></mrow></mmultiscripts> <annotation>{}_{-4.4}^{+4.2}</annotation></semantics></math></td><td>0.56  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>0.10</mn></mrow> <mrow><mo>−</mo> <mn>0.10</mn></mrow></mmultiscripts> <annotation>{}_{-0.10}^{+0.10}</annotation></semantics></math></td></tr><tr><th>Observation Masking</th><td>36.4  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>4.2</mn></mrow> <mrow><mo>−</mo> <mn>4.2</mn></mrow></mmultiscripts> <annotation>{}_{-4.2}^{+4.2}</annotation></semantics></math> (-9.9%) <sup>†</sup></td><td>0.24  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>0.04</mn></mrow> <mrow><mo>−</mo> <mn>0.03</mn></mrow></mmultiscripts> <annotation>{}_{-0.03}^{+0.04}</annotation></semantics></math> (-57.1%) <sup>†</sup></td></tr><tr><th>LLM-Summary</th><td>31.4  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>4.0</mn></mrow> <mrow><mo>−</mo> <mn>4.0</mn></mrow></mmultiscripts> <annotation>{}_{-4.0}^{+4.0}</annotation></semantics></math> (-22.3%) <sup>†</sup></td><td>0.25  <math><semantics><mmultiscripts><mrow><mo>+</mo> <mn>0.05</mn></mrow> <mrow><mo>−</mo> <mn>0.04</mn></mrow></mmultiscripts> <annotation>{}_{-0.04}^{+0.05}</annotation></semantics></math> (-55.4%) <sup>†</sup></td></tr></tbody></table>

### C.1 Statistical Analysis

We assess significance using paired nonparametric bootstrap with $B=10{,}000$ replicates and show detailed results in Table 4. For each model-strategy pair, we compute the paired difference $\Delta=\text{mean(strategy)}-\text{mean(raw)}$ on the same $n=500$ instances, preserving instance-level correlations. We report:

- 95% percentile confidence intervals
- Two-sided p-values: $p=2\times\min(\Pr(\Delta^{*}\geq 0),\Pr(\Delta^{*}\leq 0))$
- Significance markers (†) when $p<0.05$

Table 4 provides the complete bootstrap statistics. Note that p-values of 0.0000 indicate no sign-crossing across all bootstrap replicates (resolution $\leq 10^{-4}$).

Table 4: Paired bootstrap differences vs. Raw Agent with 95% percentile CIs and two-sided bootstrap p-values (B=10,000). $\Delta$ Solve Rate is reported in percentage points (pp), $\Delta$ Mean Cost in dollars per instance. Negative cost differences indicate cost savings. All rows use n=500 common instances per model. We use † to indicate significance compared to the Raw Agent.

<table><thead><tr><th>Model</th><th>Strategy</th><th><math><semantics><mi>Δ</mi> <annotation>\Delta</annotation></semantics></math> Solve Rate (pp) [lo, hi]</th><th>p</th><th><math><semantics><mi>Δ</mi> <annotation>\Delta</annotation></semantics></math> Mean Cost ($) [lo, hi]</th><th>p</th></tr></thead><tbody><tr><th rowspan="2">Gemini 2.5 Flash</th><th>Observation Masking</th><td>2.8 [-0.8, 6.4]</td><td>0.1504</td><td>-0.2377 [-0.3202, -0.1614]</td><td>0.0000 <sup>†</sup></td></tr><tr><th>LLM-Summary</th><td>3.2 [-0.4, 7.0]</td><td>0.0948</td><td>-0.1725 [-0.2579, -0.0936]</td><td>0.0000 <sup>†</sup></td></tr><tr><th rowspan="2">Gemini 2.5 Flash (thinking)</th><th>Observation Masking</th><td>-4.0 [-7.8, -0.2]</td><td>0.0406 <sup>†</sup></td><td>-0.3143 [-0.4096, -0.2245]</td><td>0.0000 <sup>†</sup></td></tr><tr><th>LLM-Summary</th><td>-9.0 [-13.0, -5.2]</td><td>0.0000 <sup>†</sup></td><td>-0.3046 [-0.4074, -0.2043]</td><td>0.0000 <sup>†</sup></td></tr><tr><th rowspan="2">Qwen3-Coder 480B</th><th>Observation Masking</th><td>1.4 [-1.6, 4.4]</td><td>0.3856</td><td>-0.6762 [-0.9320, -0.4518]</td><td>0.0000 <sup>†</sup></td></tr><tr><th>LLM-Summary</th><td>0.4 [-3.0, 3.8]</td><td>0.8736</td><td>-0.6491 [-0.9048, -0.4263]</td><td>0.0000 <sup>†</sup></td></tr><tr><th rowspan="2">Qwen3-32B</th><th>Observation Masking</th><td>-2.0 [-5.0, 1.0]</td><td>0.2086</td><td>-0.5632 [-0.7479, -0.3817]</td><td>0.0000 <sup>†</sup></td></tr><tr><th>LLM-Summary</th><td>-1.0 [-4.6, 2.6]</td><td>0.6192</td><td>-0.6174 [-0.7904, -0.4454]</td><td>0.0000 <sup>†</sup></td></tr><tr><th rowspan="2">Qwen3-32B (thinking)</th><th>Observation Masking</th><td>1.6 [-2.0, 5.2]</td><td>0.3980</td><td>-0.0510 [-0.1255, 0.0187]</td><td>0.1586</td></tr><tr><th>LLM-Summary</th><td>1.8 [-1.8, 5.4]</td><td>0.3420</td><td>-0.0021 [-0.0785, 0.0741]</td><td>0.9370</td></tr></tbody></table>

## Appendix D Additional Studies

For the critic-enhanced summarizer in Figure 6, we experiment on SWE-bench Lite-50 [^3]. For the sensitivity to the rolling window size $M$ of the Observation Masking strategy and the configurations of the [llm](#id1) -Summary strategy, we show our results on a randomly sampled 150 instance subset of SWE-bench Verified [^5] that we release with our code. We conduct these studies with GPT-4.1-mini [^20].

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2508.21433/assets/data/llm-config-solve-rate.png)

Figure 5: Downstream task performance of a single experiment on a randomly generated 150-sample subset of SWE-bench Verified 5 across various different configuration combinations with respect to the tail length M. We find that a larger summarization window compared to the tail length improves performance.

### D.1 Rolling Window Size M

We experiment with the rolling window size $M$ of the Observation Masking strategy. In Figure 9 we can see that the performance of the strategy peaks at $M=10$, before falling again when further increasing the window size to $M=20$. Thus, we use this configuration of the Observation Masking strategy for our main experiments.

### D.2 -Summary Configuration

In Figure 5 we show the solve rate of different experimental configurations for [llm](#id1) -Summary in addition to those of our baselines. We find that using tail turns $M>0$ improves downstream performance. Furthermore, in contrast to the 50-50 split between turns to summarize and tail turns that OpenHands [^28] uses, we find that summarizing more turns at once improves the solve rate. We thus proceed with $N=21,M=10$ for our main experiments.

### D.3 Critic-Enhanced Summarizer

Because the [llm](#id1) -Summary prompt presented in Figure 11 is unable to consistently outperform the Observation Masking strategy on either solve rate or cost, we investigate whether we can increase the utility of the [llm](#id1) call by integrating execution-free feedback [^2] [^21] [^10] [^37] [^23]. For this, we rewrite the summary prompt from scratch. We present the resulting prompts in Figures 12, 13 and 14 and the results of the experiment in Figure 6.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2508.21433/assets/data/joint_critic_and_summarization_vs_openhands_ablation_configuration_results.png)

Figure 6: Downstream task performance a randomly generated 150-sample subset of SWE-bench Verified 5 comparing the prompt in Appendix B with the joint critic-summarization prompt presented in this this section. We find that simply prompting the model to include feedback in its summaries does not improve the solve rate and further increases the cost.

In comparison to the modified OpenHands prompt in Figure 11, we frame the task as generating a checkpoint instead of a summary and prompt the model to reflect on the turns to summarize. In doing so, we aim to encourage the model to generate an output that help the agent adjust its solution path during an attempt and avoid overly grounding it in previous, potentially sub-optimal or even flawed, turns through a plain summary.

To elicit meaningful reflections, we prompt the [llm](#id1) with guiding questions that it could reflect and provide insights on. These questions assess whether the agent is stuck or looping, aligned with the initial problem statement, reflect on the agent’s high-level solution approach with respect to the turns to summarize. Additionally, we provide few-shot examples to further guide the agents toward generating meaningful and actionable reflections [^4]. Finally, as we did in the OpenHands-style prompt (Figure 11), we provide the previous summary, or problem statement if none is available, and the turns to summarize to the model.

To our surprise, this style of prompt is not able to meaningfully outperform the prompt we used in our main experiments (Figure 11), and also was $\approx 25\%$ more expensive per instance and resulted in $\approx 13\%$ longer trajectories. This means that a joint critic-summarization approach may actually worsen the trajectory elongation effect discussed in Section 4.4 and further increase costs. We suggest that further work investigate when to summarize, what to summarize, and in what manner to understand the effects of various [llm](#id1) -Summary approaches on agent trajectory lengths and downstream performance.

### D.4 Behavior of the Covered Context Management Strategies Across Turns

In Figure 7, we show preliminary experimental results using the trajectory management strategies introduced in Section 3.1 on SWE-bench Lite-50 [^3] with GPT-4.1-mini [^19]. Here, we use a rolling window size of $M=5$, following SWE-agent [^32] and $N=21,M=10$ for the [llm](#id1) -based approach paired with a slightly modified version of OpenHand’s prompt [^28] (see Appendix B). We observe that the Observation Masking strategy poses a strong baseline, consistently performing equally or better on the downstream task on SWE-bench Lite-50 [^3] than the [llm](#id1) -Summary approach despite using being much simpler.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2508.21433/assets/data/preliminary_experiments_solve_rate.png)

Figure 7: Downstream task performance on SWE-bench Lite-50 3 across the context management strategies we cover in our study. Bars represent the mean across three experiments, the errors bars show the standard deviations. Surprisingly, f R W f\_{RW} performs on par with the llm -based strategy using more compute.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2508.21433/assets/data/preliminary_experiments.png)

Figure 8: Preliminary experimental results motivating our study. Dashed lines show expected behavior based on mean token counts per turn type observed in the raw agent trajectories. We micro-average all results with standard deviation shown as shaded regions. (a),(b), and (c) The observed data closely match the simulated effects of applying either trajectory management strategy to the simulated raw agent trajectory. The effects of the llm and Observation Masking approach on cost and context window size overlap at lower turn numbers. Due to the bounding of the context-window we expect the approach to be especially effective on long trajectories. (d) With an increasing number of turns, our empiric data becomes increasingly sparse.

To investigate why this is the case, and uncover potential scenarios in which the [llm](#id1) -Summary strategy may be beneficial, we analyze the behavior of these strategies across turns in Figure 8. The solid colored lines are the empirically observed results using SWE-agent [^32]. For each trajectory management strategy, we run three experiments, yielding 150 trajectories total. To visualize these data, we use the micro-averaged mean for each turn. The dashed lines indicate the empirically grounded simulated behavior. To generate the data for these simulated trajectories, we compute the mean token consumption per token type across all experimental data available for the raw agent:

$$
\bar{x}=\frac{1}{T_{total}}\sum_{i=1}^{3}\sum_{j=1}^{50}\sum_{k=1}^{T_{local}}x_{ijk}\quad\text{where}\quad x\in\{r,a,o\}
$$

where $T_{total}$ are the total number of turns $T$ we observed across all instances and experiments and $T_{local}$ is the number of turns of a single trajectory. We then generate a turn $T_{sim}=(\bar{r},\bar{a},\bar{o})$ using placeholder tokens. By repeatedly appending $T_{sim}$ we generate a simulated agent trajectory $\tau_{sim}=(T_{sim},\ldots,T_{sim})$ of arbitrary length. To generate the data for the simulated [llm](#id1) -Summary and Observation Masking context management trajectories, we apply these strategies to $\tau_{sim}$. This allows us to study the expected behavior of these trajectory management strategies up to a large number of turns.

Figures 8 a, b and c show that our experimental data match the expected behaviour of simulated trends closely. Surprisingly, we find that the $f_{RW}$ is competitive with the the [llm](#id1) -based strategy on cost and even outperforms it on context compression.

$f_{RW}$ is a strong baseline, due to the distribution of tokens across the types $r,a,o$. In Figure 1 we plot the share of token types in $T_{sim}$. The environment observation tokens $o$ overwhelmingly dominate the composition of $T_{sim}$, contributing $\approx 84\%$. Thus targeting this token type is extremely effective.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2508.21433/assets/data/baseline_tail_length_ablation_configuration_results.png)

Figure 9: Downstream task performance of a single experiment on a randomly generated 150-sample subset of SWE-bench Verified 5 across different context window sizes. We find that M = 10 M=10 yields optimal performance.

We can see this in effect in Figures 8a, and b. While the cost of the two context management strategies is similar, due to the worse cache behavior of $f_{RW}$, $f_{RW}$ offers superior compression especially at lower turn numbers. Looking at our simulations on the other hand, we expect the [llm](#id1) -based approach to start outperforming $f_{RW}$ on longer trajectories because it bounds the maximum context size in a fuzzy manner, resulting in a saw-function for both the cost and context window size. This motivates us to set the turn limit in our main experiments to 250.

## Appendix E Preliminary Findings using OpenHands

To probe generality across scaffolds, we ran a preliminary study with OpenHands (v0.43.0) on a 50-instance slice of SWE-bench Verified [^5] [^3] using Gemini 2.5 Flash [^6] (no thinking), with turn limit of 250, [llm](#id1) -Summary N=21, M=10, and Observation Masking M=10. We present the results in Table 5 and Figure 10. These results suggest that while per-instance costs are similar for masking and summarization, solve rate favored summarization on this slice. We hypothesize that scaffold-specific trajectory construction and observation preprocessing modulate the utility of summaries. For example, SWE-agent [^32] typically elides raw error logs and long traces from the dialog history, which may already mask failure signals; OpenHands [^28] retains richer logs, making summaries more informative. This motivates a controlled multi-scaffold study.

Figure 10: Preliminary results on the effectiveness versus efficiency tradeoff for context management strategies within OpenHands [^28] on the SWE-bench Verified-50 [^3] subset. The plot compares solve rate (y-axis, ↑) against average cost per trajectory (x-axis, ↓) Gemini 2.5 Flash [^6] without thinking. These results indicate that agent specific design decisions may impact the efficiency and effectiveness of different context management strategies.

Table 5: Comparison of context management strategies for Gemini 2.5 Flash. We report Solve Rate (effectiveness, $\uparrow$) and Instance Cost (efficiency, $\downarrow$). We boldface the best-performing context management strategy for each metric and report change relative to the Raw Agent. All experiments use OpenHands [^28] on the SWE-bench Verified-50 subset [^3].

<table><tbody><tr><th>Model</th><th>Strategy</th><th>Solve Rate (%,<math><semantics><mo>↑</mo> <annotation>\uparrow</annotation></semantics></math>)</th><th>Instance Cost ($,<math><semantics><mo>↓</mo> <annotation>\downarrow</annotation></semantics></math>)</th></tr><tr><th rowspan="3">Gemini 2.5 Flash</th><th>Raw Agent</th><th>40.0</th><th>2.39</th></tr><tr><th>Observation Masking</th><td>30.0(-25.0%)</td><td>1.14(-52.3%)</td></tr><tr><th>LLM-Summary</th><td>42.0(+5.0%)</td><td>1.22 (-49.0%)</td></tr></tbody></table>

<svg id="A5.F11.pic1" height="456.85" overflow="visible" version="1.1" viewBox="0 0 477.38 456.85" width="477.38"><g style="--ltx-stroke-color:#000000;--ltx-fill-color:#000000;" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,456.85) matrix(1 0 0 -1 0 0)"><g style="--ltx-fill-color:#FF318C;" fill="#FF318C" fill-opacity="1.0"><path style="stroke:none" d="M 0 5.91 L 0 450.95 C 0 454.21 2.64 456.85 5.91 456.85 L 471.47 456.85 C 474.73 456.85 477.38 454.21 477.38 450.95 L 477.38 5.91 C 477.38 2.64 474.73 0 471.47 0 L 5.91 0 C 2.64 0 0 2.64 0 5.91 Z"></path></g><g style="--ltx-fill-color:#FFCEE4;" fill="#FFCEE4" fill-opacity="1.0"><path style="stroke:none" d="M 1.97 5.91 L 1.97 432.74 L 475.41 432.74 L 475.41 5.91 C 475.41 3.73 473.65 1.97 471.47 1.97 L 5.91 1.97 C 3.73 1.97 1.97 3.73 1.97 5.91 Z"></path></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 441.34)"><foreignObject style="--ltx-fo-width:31.37em;--ltx-fo-height:0.69em;--ltx-fo-depth:0.19em;font-size:10pt;" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 9.61)" width="434.07"><span id="A5.F11.pic1.1" style="width:31.37em;"><span id="A5.F11.pic1.1.1"><a href="#id1"><abbr href="#id1" title="The Complexity Trap: Simple Observation Masking Is as Efficient as LLM Summarization for Agent Context Management">llm</abbr></a> <span id="A5.F11.pic1.1.1.1" style="--ltx-fg-color:#FFFFFF;">-Summary Prompt</span></span></span></foreignObject></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 14.82)"><foreignObject overflow="visible" width="31.37em"><pre><code>You are maintaining a context-aware state summary for an interactive agent.
You will be given a list of events corresponding to actions taken by the
agent, and the most recent previous summary if one exists. Track:

USER_CONTEXT: (Preserve essential user requirements, goals, and
clarifications in concise form)

COMPLETED: (Tasks completed so far, with brief results)
PENDING: (Tasks that still need to be done)
CURRENT_STATE: (Current variables, data structures, or relevant state)

For code-specific tasks, also include:
CODE_STATE: (File paths, function signatures, data structures)
TESTS: (Failing cases, error messages, outputs)
CHANGES: (Code edits, variable updates)
DEPS: (Dependencies, imports, external calls)
VERSION_CONTROL_STATUS: (Repository state, current branch, PR status,
commit history)

PRIORITIZE:
1. Adapt tracking format to match the actual task type
2. Capture key user requirements and goals
3. Distinguish between completed and pending tasks
4. Keep all sections concise and relevant

SKIP: Tracking irrelevant details for the current task type

Example formats:

For code tasks:
USER_CONTEXT: Fix FITS card float representation issue
COMPLETED: Modified mod_float() in card.py, all tests passing
PENDING: Create PR, update documentation
CODE_STATE: mod_float() in card.py updated
TESTS: test_format() passed
CHANGES: str(val) replaces f"{val:.16G}"
DEPS: None modified
VERSION_CONTROL_STATUS: Branch: fix-float-precision, Latest commit: a1b2c3d

&lt;PREVIOUS_SUMMARY&gt;
...
&lt;/PREVIOUS_SUMMARY&gt;
&lt;TURN-0&gt;
...
&lt;/TURN-0&gt;
...
&lt;TURN-20&gt;
...
&lt;/TURN-20&gt;</code></pre></foreignObject></g></g></svg>

Figure 11: The [llm](#id1) -Summary prompt we use in SWE-agent [^32] is a slightly modified version of the OpenHands [llm](#id1) -Summary system prompt [^28]. Additionally we pass the previous summary and the turns to summarize. If no previous summary is available we instead pass the task problem statement as initial context for the summary generation.

<svg id="A5.F12.pic1" height="740.67" overflow="visible" version="1.1" viewBox="0 0 477.38 740.67" width="477.38"><g style="--ltx-stroke-color:#000000;--ltx-fill-color:#000000;" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,740.67) matrix(1 0 0 -1 0 0)"><g style="--ltx-fill-color:#FF318C;" fill="#FF318C" fill-opacity="1.0"><path style="stroke:none" d="M 0 5.91 L 0 734.76 C 0 738.02 2.64 740.67 5.91 740.67 L 471.47 740.67 C 474.73 740.67 477.38 738.02 477.38 734.76 L 477.38 5.91 C 477.38 2.64 474.73 0 471.47 0 L 5.91 0 C 2.64 0 0 2.64 0 5.91 Z"></path></g><g style="--ltx-fill-color:#FFCEE4;" fill="#FFCEE4" fill-opacity="1.0"><path style="stroke:none" d="M 1.97 5.91 L 1.97 715.02 L 475.41 715.02 L 475.41 5.91 C 475.41 3.73 473.65 1.97 471.47 1.97 L 5.91 1.97 C 3.73 1.97 1.97 3.73 1.97 5.91 Z"></path></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 724.38)"><foreignObject style="--ltx-fo-width:31.37em;--ltx-fo-height:0.75em;--ltx-fo-depth:0.25em;font-size:10pt;" height="13.84" overflow="visible" transform="matrix(1 0 0 -1 0 10.38)" width="434.07"><span id="A5.F12.pic1.1" style="width:31.37em;"><span id="A5.F12.pic1.1.1"><span id="A5.F12.pic1.1.1.1" style="--ltx-fg-color:#FFFFFF;">Joint Critic-Summary Prompt (Part 1)</span></span></span></foreignObject></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 14.82)"><foreignObject overflow="visible" width="31.37em"><pre><code>You are maintaining a context-aware state checkpoint for an interactive agent
working on software engineering tasks (specifically, bug fixes), assessing the
agents progress toward completing the task, and offering suggestions and
guidance if it is not on track.
You will be given a list of turns corresponding to actions taken by the agent
and their resulting observations, and the most recent previous checkpoint if
one exists. You must proceed in the following two phases:
1. &lt;CHECKPOINT&gt;
2. &lt;REFLECTIONS&gt;

A &lt;CHECKPOINT&gt; should capture the current repository state and the agent’s
progress towards completing the task. It consists of:
USER_CONTEXT: (Preserve essential user requirements, goals, and clarifications
based on findings, previous checkpoints, and the initial problem statement in
concise form)
CODE_STATE: (File paths, function signatures, data structures followed by
their current state)
TESTS: (Failing cases, error messages, outputs)
CHANGES: (Code edits, variable updates)
DEPS: (Dependencies, imports, external calls)

PRIORITIZE:
1. Capture key requirements from the initial issue description or previous
checkpoints and reflections
2. Keep all sections concise and relevant to fixing the issue
3. Focus on information that quantifies the agent’s progress towards a solution

Next you must reflect on the agent’s progres in the below to turns to
generate a set of &lt;REFLECTIONS&gt;. Here are some aspects to consider
when generating the &lt;REFLECTIONS&gt;:
- Are the agent’s actions still aligned with the initial user requirement?
- Is the agent making progress?
- Is it stuck in a loop or repeatedly carrying out the same actions?
- Can you identify any problematic patterns in the agent’s actions?
- Did the agent follow the issue description or your previous feedback?
    If so, why did or didn’t it make meaningful progress?
- Which critical piece of information might help the agent get back on
    track?
- What has the agent not tried so far?

Key requirements for the &lt;REFLECTIONS&gt;:
1. Avoid reporting nitpicks as they may confuse the agent.
2. Your reflections should be diverse with respect to any available previous
reflections.
3. Provide up to 2 reflections total. Reflections are mutually exclusive.
4. Each reflection should identify one distinct problem and may include one
or two fixes for that problem.
5. Limit yourself to the most critical issues that are blocking progress.

When generating the &lt;REFLECTIONS&gt;, follow the format below:
&lt;REFLECTIONS&gt;
Problem-A: (a detailed description of a problem the agent is facing)
Fix-A.1: (proposed solution, guidance or hint for overcoming the problem
including the rationale for it)
&lt;/REFLECTIONS&gt;</code></pre></foreignObject></g></g></svg>

Figure 12: Part 1 of our joint critic and summarization [llm](#id1) -Summary prompt. Compared to the [llm](#id1) -Summary prompt we use in our main experiments (Figure 11), we also prompt the [llm](#id1) to act as execution-free critic regarding the turns it is summarizing.

<svg id="A5.F13.pic1" height="1007.44" overflow="visible" version="1.1" viewBox="0 0 477.38 1007.44" width="477.38"><g style="--ltx-stroke-color:#000000;--ltx-fill-color:#000000;" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,1007.44) matrix(1 0 0 -1 0 0)"><g style="--ltx-fill-color:#FF318C;" fill="#FF318C" fill-opacity="1.0"><path style="stroke:none" d="M 0 5.91 L 0 1001.54 C 0 1004.8 2.64 1007.44 5.91 1007.44 L 471.47 1007.44 C 474.73 1007.44 477.38 1004.8 477.38 1001.54 L 477.38 5.91 C 477.38 2.64 474.73 0 471.47 0 L 5.91 0 C 2.64 0 0 2.64 0 5.91 Z"></path></g><g style="--ltx-fill-color:#FFCEE4;" fill="#FFCEE4" fill-opacity="1.0"><path style="stroke:none" d="M 1.97 5.91 L 1.97 981.8 L 475.41 981.8 L 475.41 5.91 C 475.41 3.73 473.65 1.97 471.47 1.97 L 5.91 1.97 C 3.73 1.97 1.97 3.73 1.97 5.91 Z"></path></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 991.16)"><foreignObject style="--ltx-fo-width:31.37em;--ltx-fo-height:0.75em;--ltx-fo-depth:0.25em;font-size:10pt;" height="13.84" overflow="visible" transform="matrix(1 0 0 -1 0 10.38)" width="434.07"><span id="A5.F13.pic1.1" style="width:31.37em;"><span id="A5.F13.pic1.1.1"><span id="A5.F13.pic1.1.1.1" style="--ltx-fg-color:#FFFFFF;">Joint Critic-Summary Prompt (Part 2)</span></span></span></foreignObject></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 14.82)"><foreignObject overflow="visible" width="31.37em"><pre><code>Example output and format:
&lt;CHECKPOINT&gt;
USER_CONTEXT: Fix failing authentication in REST API. Users report
"Invalid token" errors after ˜30 minutes of activity. The API should
maintain user sessions properly with JWT tokens that expire after 1 hour.
CODE_STATE:
1. api/auth_middleware.py: validate_token() MODIFIED with logging.
2. api/auth_utils.py: refresh_token() MODIFIED to auto-refresh at 45 min.
3. config.py: JWT_EXPIRATION unchanged at 3600.
TESTS:
1. tests/test_auth_integration.py::test_long_session: FAILING - "Token
expired at 32 minutes"
2. tests/test_auth_integration.py::test_token_refresh: PASSING
3. tests/test_auth_unit.py: ALL PASSING
CHANGES:
1. auth_utils.py: Added auto-refresh logic when token age &gt; 2700 seconds.
2. auth_middleware.py: Added debug logging for token validation steps.
DEPS: PyJWT==2.4.0, python-jose==3.3.0 (both imported)
&lt;/CHECKPOINT&gt;
&lt;REFLECTIONS&gt;
Problem-A: Agent has spent 6 turns modifying the token refresh logic
and adding complex auto-refresh mechanisms, but hasn’t investigated why
tokens are expiring at ˜30 minutes when they’re configured for 60 minutes.
The agent is treating the symptom (early expiration) by adding refresh
logic, rather than finding the root cause. The fact that tokens consistently
expire at 30-32 minutes suggests either: (1) a configuration mismatch
somewhere else overriding the 3600-second setting, (2) a timezone/clock
issue between server and client, or (3) the JWT library might be using a
different time unit or has a default max age.
Fix-A.1: Stop adding refresh logic and investigate the actual token
expiration time. Add logging to print the exact ’exp’ claim value when
tokens are created and when they’re validated. Check if there’s another
config file, environment variable, or hardcoded value setting token
expiration to 1800 seconds (30 min). Also verify the JWT library’s time unit
- some libraries use milliseconds while others use seconds. The issue is
likely a simple configuration problem, not a need for complex refresh
mechanisms.
Fix-A.2: The solution the agent is trying to implement is overly complex,
confusing, and not tackeling the root cause. The agent should take a step
back and think about what it is actually trying to do, discard its current
approach and come up with a simpler solution that is more likely to work.
It should recall SE best practices and clean code principles.

Problem-B: Agent has successfully modified token validation and refresh logic,
but the integration test continues to fail at exactly 32 minutes. Despite the
previous guidance to investigate configuration mismatches, the agent hasn’t
checked for environmental differences between unit tests (which pass) and
integration tests (which fail). The agent also hasn’t noticed that two
different JWT libraries are imported (PyJWT and python-jose), which could
mean tokens are created with one library but validated with another.
Additionally, the agent keeps focusing on server-side fixes without considering
that the test client might have its own timeout or token handling logic
that’s causing the consistent 32-minute failure.
Fix-B.1: Audit which JWT library is actually being used where. Search for
‘from jose import‘ and ‘from jwt import‘ patterns across the codebase. Create
a simple debug endpoint that generates a token and immediately decodes it
with both libraries to see if they interpret expiration differently. The
symptom of tokens expiring at ˜32 minutes (close to but not exactly 30)
could indicate timestamp precision or timezone handling differences between
the libraries. Consider standardizing on one JWT library throughout the codebase.
&lt;/REFLECTIONS&gt;</code></pre></foreignObject></g></g></svg>

Figure 13: Part 2 of our joint critic and summarization [llm](#id1) -Summary prompt. Compared to the [llm](#id1) -Summary prompt we use in our main experiments (Figure 11), we also prompt the [llm](#id1) to act as execution-free critic regarding the turns it is summarizing.

<svg id="A5.F14.pic1" height="77.6" overflow="visible" version="1.1" viewBox="0 0 477.38 77.6" width="477.38"><g style="--ltx-stroke-color:#000000;--ltx-fill-color:#000000;" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,77.6) matrix(1 0 0 -1 0 0)"><g style="--ltx-fill-color:#FF318C;" fill="#FF318C" fill-opacity="1.0"><path style="stroke:none" d="M 0 5.91 L 0 71.69 C 0 74.95 2.64 77.6 5.91 77.6 L 471.47 77.6 C 474.73 77.6 477.38 74.95 477.38 71.69 L 477.38 5.91 C 477.38 2.64 474.73 0 471.47 0 L 5.91 0 C 2.64 0 0 2.64 0 5.91 Z"></path></g><g style="--ltx-fill-color:#FFCEE4;" fill="#FFCEE4" fill-opacity="1.0"><path style="stroke:none" d="M 1.97 5.91 L 1.97 51.95 L 475.41 51.95 L 475.41 5.91 C 475.41 3.73 473.65 1.97 471.47 1.97 L 5.91 1.97 C 3.73 1.97 1.97 3.73 1.97 5.91 Z"></path></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 61.31)"><foreignObject style="--ltx-fo-width:31.37em;--ltx-fo-height:0.75em;--ltx-fo-depth:0.25em;font-size:10pt;" height="13.84" overflow="visible" transform="matrix(1 0 0 -1 0 10.38)" width="434.07"><span id="A5.F14.pic1.1" style="width:31.37em;"><span id="A5.F14.pic1.1.1"><span id="A5.F14.pic1.1.1.1" style="--ltx-fg-color:#FFFFFF;">Joint Critic-Summary Prompt (Part 3)</span></span></span></foreignObject></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 14.82)"><foreignObject overflow="visible" width="31.37em"><pre><code>&lt;PREVIOUS_SUMMARY&gt;
...
&lt;/PREVIOUS_SUMMARY&gt;
&lt;TURN-0&gt;
...
&lt;/TURN-0&gt;
...
&lt;TURN-20&gt;
...
&lt;/TURN-20&gt;</code></pre></foreignObject></g></g></svg>

Figure 14: Part 3 of our joint critic and summarization [llm](#id1) -Summary prompt. Compared to the [llm](#id1) -Summary prompt we use in our main experiments (Figure 11), we also prompt the [llm](#id1) to act as execution-free critic regarding the turns it is summarizing.

## NeurIPS Paper Checklist

1. Claims
2. Question: Do the main claims made in the abstract and introduction accurately reflect the paper’s contributions and scope?
3. Answer: \[Yes\]
4. Justification: The claims made in the abstract and introduction are discussed in detail in Section 4 and Section 5. Additionally, we extensively document our experimental setup in Section 3 and take steps to ensure our experiments cover diverse configurations and find that our experimental results support our claims across all configurations.
5. Guidelines:
	- The answer NA means that the abstract and introduction do not include the claims made in the paper.
	- The abstract and/or introduction should clearly state the claims made, including the contributions made in the paper and important assumptions and limitations. A No or NA answer to this question will not be perceived well by the reviewers.
	- The claims made should match theoretical and experimental results, and reflect how much the results can be expected to generalize to other settings.
	- It is fine to include aspirational goals as motivation as long as it is clear that these goals are not attained by the paper.
6. Limitations
7. Question: Does the paper discuss the limitations of the work performed by the authors?
8. Answer: \[Yes\]
9. Justification: We explicitly document the limitations of our work and the threats to the validity of our study in Section 6.
10. Guidelines:
	- The answer NA means that the paper has no limitation while the answer No means that the paper has limitations, but those are not discussed in the paper.
	- The authors are encouraged to create a separate "Limitations" section in their paper.
	- The paper should point out any strong assumptions and how robust the results are to violations of these assumptions (e.g., independence assumptions, noiseless settings, model well-specification, asymptotic approximations only holding locally). The authors should reflect on how these assumptions might be violated in practice and what the implications would be.
	- The authors should reflect on the scope of the claims made, e.g., if the approach was only tested on a few datasets or with a few runs. In general, empirical results often depend on implicit assumptions, which should be articulated.
	- The authors should reflect on the factors that influence the performance of the approach. For example, a facial recognition algorithm may perform poorly when image resolution is low or images are taken in low lighting. Or a speech-to-text system might not be used reliably to provide closed captions for online lectures because it fails to handle technical jargon.
	- The authors should discuss the computational efficiency of the proposed algorithms and how they scale with dataset size.
	- If applicable, the authors should discuss possible limitations of their approach to address problems of privacy and fairness.
	- While the authors might fear that complete honesty about limitations might be used by reviewers as grounds for rejection, a worse outcome might be that reviewers discover limitations that aren’t acknowledged in the paper. The authors should use their best judgment and recognize that individual actions in favor of transparency play an important role in developing norms that preserve the integrity of the community. Reviewers will be specifically instructed to not penalize honesty concerning limitations.
11. Theory assumptions and proofs
12. Question: For each theoretical result, does the paper provide the full set of assumptions and a complete (and correct) proof?
13. Answer: \[N/A\]
14. Justification: Our work presents an extensive, empiric study of the cost-performance tradeoff of [se](#id5) agent context management and thus does not contain theoretical results.
15. Guidelines:
	- The answer NA means that the paper does not include theoretical results.
	- All the theorems, formulas, and proofs in the paper should be numbered and cross-referenced.
	- All assumptions should be clearly stated or referenced in the statement of any theorems.
	- The proofs can either appear in the main paper or the supplemental material, but if they appear in the supplemental material, the authors are encouraged to provide a short proof sketch to provide intuition.
	- Inversely, any informal proof provided in the core of the paper should be complemented by formal proofs provided in appendix or supplemental material.
	- Theorems and Lemmas that the proof relies upon should be properly referenced.
16. Experimental result reproducibility
17. Question: Does the paper fully disclose all the information needed to reproduce the main experimental results of the paper to the extent that it affects the main claims and/or conclusions of the paper (regardless of whether the code and data are provided or not)?
18. Answer: \[Yes\]
19. Justification: In Section 3 we comprehensively document our experimental setup, including the models used, their hyperparameters, and the benchmark we evaluate on. Furthermore, we provide details on our chosen configurations in Section D.4 and Appendix D.
20. Guidelines:
	- The answer NA means that the paper does not include experiments.
	- If the paper includes experiments, a No answer to this question will not be perceived well by the reviewers: Making the paper reproducible is important, regardless of whether the code and data are provided or not.
	- If the contribution is a dataset and/or model, the authors should describe the steps taken to make their results reproducible or verifiable.
	- Depending on the contribution, reproducibility can be accomplished in various ways. For example, if the contribution is a novel architecture, describing the architecture fully might suffice, or if the contribution is a specific model and empirical evaluation, it may be necessary to either make it possible for others to replicate the model with the same dataset, or provide access to the model. In general. releasing code and data is often one good way to accomplish this, but reproducibility can also be provided via detailed instructions for how to replicate the results, access to a hosted model (e.g., in the case of a large language model), releasing of a model checkpoint, or other means that are appropriate to the research performed.
	- While NeurIPS does not require releasing code, the conference does require all submissions to provide some reasonable avenue for reproducibility, which may depend on the nature of the contribution. For example
		1. If the contribution is primarily a new algorithm, the paper should make it clear how to reproduce that algorithm.
		2. If the contribution is primarily a new model architecture, the paper should describe the architecture clearly and fully.
		3. If the contribution is a new model (e.g., a large language model), then there should either be a way to access this model for reproducing the results or a way to reproduce the model (e.g., with an open-source dataset or instructions for how to construct the dataset).
		4. We recognize that reproducibility may be tricky in some cases, in which case authors are welcome to describe the particular way they provide for reproducibility. In the case of closed-source models, it may be that access to the model is limited in some way (e.g., to registered users), but it should be possible for other researchers to have some path to reproducing or verifying the results.
21. Open access to data and code
22. Question: Does the paper provide open access to the data and code, with sufficient instructions to faithfully reproduce the main experimental results, as described in supplemental material?
23. Answer: \[TODO\]
24. Justification: We release our [llm](#id1) -summary implementation <sup>9</sup> for SWE-agent [^32]. The experimental results of our main experiments can be openly accessed via HuggingFace <sup>10</sup>.
25. Guidelines:
	- The answer NA means that paper does not include experiments requiring code.
	- Please see the NeurIPS code and data submission guidelines ([https://nips.cc/public/guides/CodeSubmissionPolicy](https://nips.cc/public/guides/CodeSubmissionPolicy)) for more details.
	- While we encourage the release of code and data, we understand that this might not be possible, so “No” is an acceptable answer. Papers cannot be rejected simply for not including code, unless this is central to the contribution (e.g., for a new open-source benchmark).
	- The instructions should contain the exact command and environment needed to run to reproduce the results. See the NeurIPS code and data submission guidelines ([https://nips.cc/public/guides/CodeSubmissionPolicy](https://nips.cc/public/guides/CodeSubmissionPolicy)) for more details.
	- The authors should provide instructions on data access and preparation, including how to access the raw data, preprocessed data, intermediate data, and generated data, etc.
	- The authors should provide scripts to reproduce all experimental results for the new proposed method and baselines. If only a subset of experiments are reproducible, they should state which ones are omitted from the script and why.
	- At submission time, to preserve anonymity, the authors should release anonymized versions (if applicable).
	- Providing as much information as possible in supplemental material (appended to the paper) is recommended, but including URLs to data and code is permitted.
26. Experimental setting/details
27. Question: Does the paper specify all the training and test details (e.g., data splits, hyperparameters, how they were chosen, type of optimizer, etc.) necessary to understand the results?
28. Answer: \[Yes\]
29. Justification: In Section 3 we comprehensively document our experimental setup, including the models used, their hyperparameters, and the benchmark we evaluate on. Furthermore, we provide details on our chosen configurations in Section D.4 and Appendix D.
30. Guidelines:
	- The answer NA means that the paper does not include experiments.
	- The experimental setting should be presented in the core of the paper to a level of detail that is necessary to appreciate the results and make sense of them.
	- The full details can be provided either with the code, in appendix, or as supplemental material.
31. Experiment statistical significance
32. Question: Does the paper report error bars suitably and correctly defined or other appropriate information about the statistical significance of the experiments?
33. Answer: \[Yes\]
34. Justification: In Section 3 we show the standard deviation of the solve rate, cost and context window size in our preliminary experiments.
35. Guidelines:
	- The answer NA means that the paper does not include experiments.
	- The authors should answer "Yes" if the results are accompanied by error bars, confidence intervals, or statistical significance tests, at least for the experiments that support the main claims of the paper.
	- The factors of variability that the error bars are capturing should be clearly stated (for example, train/test split, initialization, random drawing of some parameter, or overall run with given experimental conditions).
	- The method for calculating the error bars should be explained (closed form formula, call to a library function, bootstrap, etc.)
	- The assumptions made should be given (e.g., Normally distributed errors).
	- It should be clear whether the error bar is the standard deviation or the standard error of the mean.
	- It is OK to report 1-sigma error bars, but one should state it. The authors should preferably report a 2-sigma error bar than state that they have a 96% CI, if the hypothesis of Normality of errors is not verified.
	- For asymmetric distributions, the authors should be careful not to show in tables or figures symmetric error bars that would yield results that are out of range (e.g. negative error rates).
	- If error bars are reported in tables or plots, The authors should explain in the text how they were calculated and reference the corresponding figures or tables in the text.
36. Experiments compute resources TODO
37. Question: For each experiment, does the paper provide sufficient information on the computer resources (type of compute workers, memory, time of execution) needed to reproduce the experiments?
38. Answer: \[Yes\]
39. Justification: We present details on our infrastructure in Section 3.
40. Guidelines:
	- The answer NA means that the paper does not include experiments.
	- The paper should indicate the type of compute workers CPU or GPU, internal cluster, or cloud provider, including relevant memory and storage.
	- The paper should provide the amount of compute required for each of the individual experimental runs as well as estimate the total compute.
	- The paper should disclose whether the full research project required more compute than the experiments reported in the paper (e.g., preliminary or failed experiments that didn’t make it into the paper).
41. Code of ethics
42. Question: Does the research conducted in the paper conform, in every respect, with the NeurIPS Code of Ethics [https://neurips.cc/public/EthicsGuidelines](https://neurips.cc/public/EthicsGuidelines)?
43. Answer: \[Yes\]
44. Justification: Our findings pave the way toward effective and efficient [llm](#id1) agents, offering an immediate way of reducing the impact of AI on our climate and environment.
45. Guidelines:
	- The answer NA means that the authors have not reviewed the NeurIPS Code of Ethics.
	- If the authors answer No, they should explain the special circumstances that require a deviation from the Code of Ethics.
	- The authors should make sure to preserve anonymity (e.g., if there is a special consideration due to laws or regulations in their jurisdiction).
46. Broader impacts
47. Question: Does the paper discuss both potential positive societal impacts and negative societal impacts of the work performed?
48. Answer: \[Yes\]
49. Justification: We highlight the positive impact of reduced computational costs on our environment in Section 7.
50. Guidelines:
	- The answer NA means that there is no societal impact of the work performed.
	- If the authors answer NA or No, they should explain why their work has no societal impact or why the paper does not address societal impact.
	- Examples of negative societal impacts include potential malicious or unintended uses (e.g., disinformation, generating fake profiles, surveillance), fairness considerations (e.g., deployment of technologies that could make decisions that unfairly impact specific groups), privacy considerations, and security considerations.
	- The conference expects that many papers will be foundational research and not tied to particular applications, let alone deployments. However, if there is a direct path to any negative applications, the authors should point it out. For example, it is legitimate to point out that an improvement in the quality of generative models could be used to generate deepfakes for disinformation. On the other hand, it is not needed to point out that a generic algorithm for optimizing neural networks could enable people to train models that generate Deepfakes faster.
	- The authors should consider possible harms that could arise when the technology is being used as intended and functioning correctly, harms that could arise when the technology is being used as intended but gives incorrect results, and harms following from (intentional or unintentional) misuse of the technology.
	- If there are negative societal impacts, the authors could also discuss possible mitigation strategies (e.g., gated release of models, providing defenses in addition to attacks, mechanisms for monitoring misuse, mechanisms to monitor how a system learns from feedback over time, improving the efficiency and accessibility of ML).
51. Safeguards
52. Question: Does the paper describe safeguards that have been put in place for responsible release of data or models that have a high risk for misuse (e.g., pretrained language models, image generators, or scraped datasets)?
53. Answer: \[N/A\]
54. Justification: Our work presents an extensive evaluation regarding the efficiency of existing [se](#id5) agent systems, using existing models and an existing benchmark and thus does not pose any such risk.
55. Guidelines:
	- The answer NA means that the paper poses no such risks.
	- Released models that have a high risk for misuse or dual-use should be released with necessary safeguards to allow for controlled use of the model, for example by requiring that users adhere to usage guidelines or restrictions to access the model or implementing safety filters.
	- Datasets that have been scraped from the Internet could pose safety risks. The authors should describe how they avoided releasing unsafe images.
	- We recognize that providing effective safeguards is challenging, and many papers do not require this, but we encourage authors to take this into account and make a best faith effort.
56. Licenses for existing assets
57. Question: Are the creators or original owners of assets (e.g., code, data, models), used in the paper, properly credited and are the license and terms of use explicitly mentioned and properly respected?
58. Answer: \[Yes\]
59. Justification: We credit the authors of SWE-bench [^11], SWE-bench Lite-50 [^3], SWE-bench Verified [^5], SWE-agent [^32] and OpenHands [^28] at the appropriate locations in our work in Section 2, Section 3, and Section 4.
60. Guidelines:
	- The answer NA means that the paper does not use existing assets.
	- The authors should cite the original paper that produced the code package or dataset.
	- The authors should state which version of the asset is used and, if possible, include a URL.
	- The name of the license (e.g., CC-BY 4.0) should be included for each asset.
	- For scraped data from a particular source (e.g., website), the copyright and terms of service of that source should be provided.
	- If assets are released, the license, copyright information, and terms of use in the package should be provided. For popular datasets, [paperswithcode.com/datasets](https://ar5iv.labs.arxiv.org/html/paperswithcode.com/datasets) has curated licenses for some datasets. Their licensing guide can help determine the license of a dataset.
	- For existing datasets that are re-packaged, both the original license and the license of the derived asset (if it has changed) should be provided.
	- If this information is not available online, the authors are encouraged to reach out to the asset’s creators.
61. New assets
62. Question: Are new assets introduced in the paper well documented and is the documentation provided alongside the assets?
63. Answer: \[TODO\]
64. Justification: We release our implementation for generating [llm](#id1) -summaries in SWE-agent [^32] with this work in a well-documented manner.
65. Guidelines:
	- The answer NA means that the paper does not release new assets.
	- Researchers should communicate the details of the dataset/code/model as part of their submissions via structured templates. This includes details about training, license, limitations, etc.
	- The paper should discuss whether and how consent was obtained from people whose asset is used.
	- At submission time, remember to anonymize your assets (if applicable). You can either create an anonymized URL or include an anonymized zip file.
66. Crowdsourcing and research with human subjects
67. Question: For crowdsourcing experiments and research with human subjects, does the paper include the full text of instructions given to participants and screenshots, if applicable, as well as details about compensation (if any)?
68. Answer: \[N/A\]
69. Justification: All of our experiments are conducted on existing benchmarks. Our work does not involve any crowdsourced labor.
70. Guidelines:
	- The answer NA means that the paper does not involve crowdsourcing nor research with human subjects.
	- Including this information in the supplemental material is fine, but if the main contribution of the paper involves human subjects, then as much detail as possible should be included in the main paper.
	- According to the NeurIPS Code of Ethics, workers involved in data collection, curation, or other labor should be paid at least the minimum wage in the country of the data collector.
71. Institutional review board (IRB) approvals or equivalent for research with human subjects
72. Question: Does the paper describe potential risks incurred by study participants, whether such risks were disclosed to the subjects, and whether Institutional Review Board (IRB) approvals (or an equivalent approval/review based on the requirements of your country or institution) were obtained?
73. Answer: \[N/A\]
74. Justification: All of our experiments are conducted on existing benchmarks. Our work does not involve any crowdsourced labor.
75. Guidelines:
	- The answer NA means that the paper does not involve crowdsourcing nor research with human subjects.
	- Depending on the country in which research is conducted, IRB approval (or equivalent) may be required for any human subjects research. If you obtained IRB approval, you should clearly state this in the paper.
	- We recognize that the procedures for this may vary significantly between institutions and locations, and we expect authors to adhere to the NeurIPS Code of Ethics and the guidelines for their institution.
	- For initial submissions, do not include any information that would break anonymity (if applicable), such as the institution conducting the review.
76. Declaration of LLM usage
77. Question: Does the paper describe the usage of LLMs if it is an important, original, or non-standard component of the core methods in this research? Note that if the LLM is used only for writing, editing, or formatting purposes and does not impact the core methodology, scientific rigorousness, or originality of the research, declaration is not required.
78. Answer: \[Yes\]
79. Justification: Our work focuses on the efficiency on AI agents in [se](#id5), thus we explicitly discuss the use of specific [llm](#id1) throughout our work, for example in Section 3 and Section 4.
80. Guidelines:
	- The answer NA means that the core method development in this research does not involve LLMs as any important, original, or non-standard components.
	- Please refer to our LLM policy ([https://neurips.cc/Conferences/2025/LLM](https://neurips.cc/Conferences/2025/LLM)) for what should or should not be described.

[^1]: Vaibhav Aggarwal, Ojasv Kamal, Abhinav Japesh, Zhijing Jin, and Bernhard Schölkopf. DARS: Dynamic action re-sampling to enhance coding agent performance by adaptive tree traversal. In *Proceedings of the 63rd Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers)*, pages 19808–19855, Vienna, Austria, July 2025. Association for Computational Linguistics. doi: 10.18653/v1/2025.acl-long.973.

[^2]: Antonis Antoniades, Albert Örwall, Kexun Zhang, Yuxi Xie, Anirudh Goyal, and William Yang Wang. SWE-search: Enhancing software agents with monte carlo tree search and iterative refinement. In *The Thirteenth International Conference on Learning Representations*, 2025. URL [https://openreview.net/forum?id=G7sIFXugTX](https://openreview.net/forum?id=G7sIFXugTX).

[^3]: Ibragim Badertdinov, Maria Trofimova, Yuri Anapolskiy, Sergey Abramov, Karina Zainullina, Alexander Golubev, Sergey Polezhaev, Daria Litvintseva, Simon Karasik, Filipp Fisin, Sergey Skvortsov, Maxim Nekrashevich, Anton Shevtsov, and Boris Yangel. Scaling data collection for training software engineering agents. *Nebius blog*, 2024.

[^4]: Tom Brown, Benjamin Mann, Nick Ryder, Melanie Subbiah, Jared D Kaplan, Prafulla Dhariwal, Arvind Neelakantan, Pranav Shyam, Girish Sastry, Amanda Askell, Sandhini Agarwal, Ariel Herbert-Voss, Gretchen Krueger, Tom Henighan, Rewon Child, Aditya Ramesh, Daniel Ziegler, Jeffrey Wu, Clemens Winter, Chris Hesse, Mark Chen, Eric Sigler, Mateusz Litwin, Scott Gray, Benjamin Chess, Jack Clark, Christopher Berner, Sam McCandlish, Alec Radford, Ilya Sutskever, and Dario Amodei. Language Models are Few-Shot Learners. In H. Larochelle, M. Ranzato, R. Hadsell, M. F. Balcan, and H. Lin, editors, *Advances in Neural Information Processing Systems*, volume 33, pages 1877–1901. Curran Associates, Inc., 2020. URL [https://proceedings.neurips.cc/paper\_files/paper/2020/file/1457c0d6bfcb4967418bfb8ac142f64a-Paper.pdf](https://proceedings.neurips.cc/paper_files/paper/2020/file/1457c0d6bfcb4967418bfb8ac142f64a-Paper.pdf).

[^5]: Neil Chowdhury, James Aung, Chan Jun Shern, Oliver Jaffe, Dane Sherburn, Giulio Starace, Evan Mays, Rachel Dias, Marwan Aljubeh, Mia Glaese, Carlos E. Jimenez, John Yang, Leyton Ho, Tejal Patwardhan, Kevin Liu, and Aleksander Madry. Introducing swe-bench verified, 2024. URL [https://openai.com/index/introducing-swe-bench-verified/](https://openai.com/index/introducing-swe-bench-verified/). Accessed on March 12, 2025.

[^6]: Gheorghe Comanici, Eric Bieber, Mike Schaekermann, et al. Technical report, Google DeepMind, 2025.

[^7]: Kelly Hong, Anton Troynikov, and Jeff Huber. Context rot: How increasing input tokens impacts llm performance. Technical report, Chroma, July 2025. URL [https://research.trychroma.com/context-rot](https://research.trychroma.com/context-rot).

[^8]: Binyuan Hui, Jian Yang, Zeyu Cui, Jiaxi Yang, Dayiheng Liu, Lei Zhang, Tianyu Liu, Jiajun Zhang, Bowen Yu, Kai Dang, et al. Qwen2. 5-coder technical report. *arXiv preprint arXiv:2409.12186*, 2024.

[^9]: Anysphere Inc. *Summarization: Context management for long conversations in chat*. URL [https://docs.cursor.com/en/agent/chat/summarization](https://docs.cursor.com/en/agent/chat/summarization). Accessed Aug. 7, 2025.

[^10]: Naman Jain, Jaskirat Singh, Manish Shetty, Liang Zheng, Koushik Sen, and Ion Stoica. R2E-Gym: Procedural Environments and Hybrid Verifiers for Scaling Open-Weights SWE Agents, 2025.

[^11]: Carlos E Jimenez, John Yang, Alexander Wettig, Shunyu Yao, Kexin Pei, Ofir Press, and Karthik R Narasimhan. SWE-bench: Can language models resolve real-world github issues? In *The Twelfth International Conference on Learning Representations*, 2024. URL [https://openreview.net/forum?id=VTF8yNQM66](https://openreview.net/forum?id=VTF8yNQM66).

[^12]: Bowen Jin, Hansi Zeng, Zhenrui Yue, Jinsung Yoon, Sercan Arik, Dong Wang, Hamed Zamani, and Jiawei Han. Search-r1: Training llms to reason and leverage search engines with reinforcement learning. *arXiv preprint arXiv:2503.09516*, 2025.

[^13]: Tom Kwiatkowski, Jennimaria Palomaki, Olivia Redfield, Michael Collins, Ankur Parikh, Chris Alberti, Danielle Epstein, Illia Polosukhin, Matthew Kelcey, Jacob Devlin, Kenton Lee, Kristina N. Toutanova, Llion Jones, Ming-Wei Chang, Andrew Dai, Jakob Uszkoreit, Quoc Le, and Slav Petrov. Natural questions: a benchmark for question answering research. *Transactions of the Association of Computational Linguistics*, 2019.

[^14]: Woosuk Kwon, Zhuohan Li, Siyuan Zhuang, Ying Sheng, Lianmin Zheng, Cody Hao Yu, Joseph E. Gonzalez, Hao Zhang, and Ion Stoica. Efficient memory management for large language model serving with pagedattention. In *Proceedings of the ACM SIGOPS 29th Symposium on Operating Systems Principles*, 2023.

[^15]: Tobias Lindenbauer, Georg Groh, and Hinrich Schuetze. From knowledge to noise: CTIM-rover and the pitfalls of episodic memory in software engineering agents. In *Proceedings of the 1st Workshop for Research on Agent Language Models (REALM 2025)*, pages 411–427, Vienna, Austria, July 2025. Association for Computational Linguistics. doi: 10.18653/v1/2025.realm-1.30.

[^16]: Nelson F. Liu, Kevin Lin, John Hewitt, Ashwin Paranjape, Michele Bevilacqua, Fabio Petroni, and Percy Liang. Lost in the middle: How language models use long contexts. 12:157–173, 2024. doi: 10.1162/tacl\_a\_00638.

[^17]: Runze Liu, Junqi Gao, Jian Zhao, Kaiyan Zhang, Xiu Li, Biqing Qi, Wanli Ouyang, and Bowen Zhou. Can 1b LLM surpass 405b LLM? rethinking compute-optimal test-time scaling.

[^18]: Ali Modarressi, Hanieh Deilamsalehy, Franck Dernoncourt, Trung Bui, Ryan A. Rossi, Seunghyun Yoon, and Hinrich Schütze. NoLiMa: Long-context evaluation beyond literal matching.

[^19]: OpenAI, Ananya Kumar, Jiahui Yu, John Hallman, et al. Introducing gpt-4.1 in the api. Technical report, OpenAI, April 2025. URL [https://openai.com/index/gpt-4-1/](https://openai.com/index/gpt-4-1/).

[^20]: OpenAI et al. GPT-4 Technical Report, March 2024. URL [http://arxiv.org/abs/2303.08774](http://arxiv.org/abs/2303.08774). arXiv:2303.08774 \[cs.CL\].

[^21]: Jiayi Pan, Xingyao Wang, Graham Neubig, Navdeep Jaitly, Heng Ji, Alane Suhr, and Yizhe Zhang. Training Software Engineering Agents and Verifiers with SWE-Gym, December 2024. URL [http://arxiv.org/abs/2412.21139](http://arxiv.org/abs/2412.21139). arXiv:2412.21139 \[cs\].

[^22]: Bowen Peng, Jeffrey Quesnelle, Honglu Fan, and Enrico Shippole. YaRN: Efficient context window extension of large language models.

[^23]: Noah Shinn, Federico Cassano, Ashwin Gopinath, Karthik Narasimhan, and Shunyu Yao. Reflexion: language agents with verbal reinforcement learning. In A. Oh, T. Naumann, A. Globerson, K. Saenko, M. Hardt, and S. Levine, editors, *Advances in Neural Information Processing Systems*, volume 36, pages 8634–8652, 2023. URL [https://proceedings.neurips.cc/paper\_files/paper/2023/file/1b44b878bb782e6954cd888628510e90-Paper-Conference.pdf](https://proceedings.neurips.cc/paper_files/paper/2023/file/1b44b878bb782e6954cd888628510e90-Paper-Conference.pdf).

[^24]: Charlie Victor Snell, Jaehoon Lee, Kelvin Xu, and Aviral Kumar. Scaling LLM test-time compute optimally can be more effective than scaling parameters for reasoning. In *The Thirteenth International Conference on Learning Representations*, 2025. doi: 10.48550/arXiv.2408.03314.

[^25]: Qwen Team. Qwen3 technical report, 2025. URL [https://arxiv.org/abs/2505.09388](https://arxiv.org/abs/2505.09388).

[^26]: Ashish Vaswani, Noam Shazeer, Niki Parmar, Jakob Uszkoreit, Llion Jones, Aidan N Gomez, Ł ukasz Kaiser, and Illia Polosukhin. Attention is all you need. In *Advances in Neural Information Processing Systems*, volume 30. Curran Associates, Inc., 2017. URL [https://proceedings.neurips.cc/paper\_files/paper/2017/file/3f5ee243547dee91fbd053c1c4a845aa-Paper.pdf](https://proceedings.neurips.cc/paper_files/paper/2017/file/3f5ee243547dee91fbd053c1c4a845aa-Paper.pdf).

[^27]: Xingyao Wang, Yangyi Chen, Lifan Yuan, Yizhe Zhang, Yunzhu Li, Hao Peng, and Heng Ji. Executable Code Actions Elicit Better LLM Agents. In *Proceedings of the 41st International Conference on Machine Learning*, pages 50208–50232, July 2024. URL [https://proceedings.mlr.press/v235/wang24h.html](https://proceedings.mlr.press/v235/wang24h.html).

[^28]: Xingyao Wang, Boxuan Li, Yufan Song, Frank F. Xu, Xiangru Tang, Mingchen Zhuge, Jiayi Pan, Yueqi Song, Bowen Li, Jaskirat Singh, Hoang H. Tran, Fuqiang Li, Ren Ma, Mingzhang Zheng, Bill Qian, Yanjun Shao, Niklas Muennighoff, Yizhe Zhang, Binyuan Hui, Junyang Lin, Robert Brennan, Hao Peng, Heng Ji, and Graham Neubig. Openhands: An open platform for AI software developers as generalist agents. In *The Thirteenth International Conference on Learning Representations*, 2025. URL [https://openreview.net/forum?id=OJd3ayDDoF](https://openreview.net/forum?id=OJd3ayDDoF).

[^29]: Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Brian Ichter, Fei Xia, Ed Chi, Quoc Le, and Denny Zhou. Chain-of-Thought Prompting Elicits Reasoning in Large Language Models. In *Proceedings of the 36th International Conference on Machine Learning.*, 2022. doi: 10.48550/arXiv.2201.11903.

[^30]: Yangzhen Wu, Zhiqing Sun, Shanda Li, Sean Welleck, and Yiming Yang. Scaling inference computation: Compute-optimal inference for problem-solving with language models. In *The 4th Workshop on Mathematical Reasoning and AI at NeurIPS’24*, 2024. URL [https://openreview.net/forum?id=j7DZWSc8qu](https://openreview.net/forum?id=j7DZWSc8qu).

[^31]: Chunqiu Steven Xia, Yinlin Deng, Soren Dunn, and Lingming Zhang. Agentless: Demystifying LLM-based software engineering agents.

[^32]: John Yang, Carlos E Jimenez, Alexander Wettig, Kilian Lieret, Shunyu Yao, Karthik R Narasimhan, and Ofir Press. SWE-agent: Agent-computer interfaces enable automated software engineering. In *The Thirty-eighth Annual Conference on Neural Information Processing Systems*, 2024. URL [https://doi.org/10.48550/arXiv.2405.1579](https://doi.org/10.48550/arXiv.2405.1579).

[^33]: John Yang, Kilian Leret, Carlos E. Jimenez, Alexander Wettig, Kabir Khandpur, Yanzhe Zhang, Binyuan Hui, Ofir Press, Ludwig Schmidt, and Diyi Yang. SWE-smith: Scaling Data for Software Engineering Agents, April 2025. arXiv:2504.21798 \[cs\].

[^34]: Zhilin Yang, Peng Qi, Saizheng Zhang, Yoshua Bengio, William W. Cohen, Ruslan Salakhutdinov, and Christopher D. Manning. HotpotQA: A dataset for diverse, explainable multi-hop question answering.

[^35]: Shunyu Yao, Howard Chen, John Yang, and Karthik Narasimhan. WebShop: Towards scalable real-world web interaction with grounded language agents.

[^36]: Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik R Narasimhan, and Yuan Cao. React: Synergizing reasoning and acting in language models. In *The Eleventh International Conference on Learning Representations*, 2023. URL [https://openreview.net/forum?id=WE\_vluYUL-X](https://openreview.net/forum?id=WE_vluYUL-X).

[^37]: Kexun Zhang, Weiran Yao, Zuxin Liu, Yihao Feng, Zhiwei Liu, Rithesh R N, Tian Lan, Lei Li, Renze Lou, Jiacheng Xu, Bo Pang, Yingbo Zhou, Shelby Heinecke, Silvio Savarese, Huan Wang, and Caiming Xiong. Diversity empowers intelligence: Integrating expertise of software engineering agents. In *The Thirteenth International Conference on Learning Representations*, 2025. URL [https://openreview.net/forum?id=cKlzKs3Nnb](https://openreview.net/forum?id=cKlzKs3Nnb).

[^38]: Zijian Zhou, Ao Qu, Zhaoxuan Wu, Sunghwan Kim, Alok Prakash, Daniela Rus, Jinhua Zhao, Bryan Kian Hsiang Low, and Paul Pu Liang. MEM1: Learning to synergize memory and reasoning for efficient long-horizon agents.