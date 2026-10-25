---
title: "Training Long-Context, Multi-Turn Software Engineering Agents with Reinforcement Learning"
source: "https://ar5iv.labs.arxiv.org/html/2508.03501"
author:
published:
created: 2026-10-25
description: "Research on applications of Reinforcement Learning (RL) to Large Language Models (LLMs) has mostly been focused on single-turn problems, such as mathematical reasoning or single-shot code generation. While these proble…"
tags:
  - "clippings"
---
Alexander Golubev    Maria Trofimova    Sergei Polezhaev    Ibragim Badertdinov    Maksim Nekrashevich    Anton Shevtsov    Simon Karasik    Sergey Abramov    Andrei Andriushchenko    Filipp Fisin    Sergei Skvortsov    Boris Yangel

###### Abstract

Research on applications of Reinforcement Learning (RL) to Large Language Models (LLMs) has mostly been focused on single-turn problems, such as mathematical reasoning or single-shot code generation. While these problems can be viewed as token-level multi-turn MDPs, this view corresponds to a degenerate case of multi-turn interaction where the environment provides no feedback. This contrasts with many real-world domains, such as software engineering (SWE), which require rich multi-turn interactions with a stateful environment that responds to each action with a non-trivial observation.

To bridge this gap, we demonstrate the successful application of RL to this general regime. Using a modified Decoupled Advantage Policy Optimization (DAPO) algorithm, we train an agent based on Qwen2.5-72B-Instruct to solve real-world software engineering tasks. Our approach increases the agent’s success rate on the SWE-bench Verified benchmark from a 20% rejection fine-tuned baseline to 39%, without relying on any teacher models. On SWE-rebench, our agent matches or outperforms leading open-weight models such as DeepSeek-V3-0324 and Qwen3-235B-A22B using an identical scaffolding, offering a viable path toward building more capable autonomous agents for complex real-world problems based on open models.

## 1 Introduction

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2508.03501/assets/figures/rl_loop.png)

Figure 1: Performance of the RL-trained agent over training iterations. To accelerate evaluations, Pass@1/10 are computed on a random subset of 50 tasks from SWE-bench Verified and averaged over 10 independent runs to reduce variance. Pass@1 is also reported together with the standard error of the mean.

Large Language Models are increasingly deployed within autonomous agents in complex, real-world domains. Software engineering is an especially compelling application area, promising substantial economic impact through automation of debugging, code generation, and software maintenance tasks. However, current approaches to developing effective SWE agents rely predominantly on one of three strategies: (i) combining sophisticated scaffolding with proprietary LLMs [^2] [^13] [^18], (ii) leveraging extensive inference-time scaling techniques [^1], or (iii) supervised fine-tuning (SFT) of open-weight models using demonstrations from stronger teacher models [^23] [^27] [^12]. While these methods have yielded strong initial results, they are often resource-intensive and depend on powerful, proprietary models. This reality underscores the need for methods that can build comparably effective systems from smaller, open-weight models. Reinforcement learning offers a promising alternative by directly optimizing an agent’s policy through interaction with a responsive environment, potentially achieving stronger performance without reliance on teacher models.

The interactive, structured nature of SWE, where actions produce observable transitions and verifiable outcomes, makes it an ideal domain for RL. Yet, to date, most RL applications for LLMs have been limited to single-turn tasks, such as math reasoning or single-shot code generation, which can be trivially modeled as multi-armed bandits or degenerate MDPs with no intermediate environmental feedback (Figure 2).

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2508.03501/assets/figures/problem_illustration.png)

Figure 2: Illustration of task structure differences between bandit-style problems ( top, e.g., math) and POMDPs ( bottom, e.g., software engineering). In bandit settings, the agent takes a single action to produce a final solution based on an initial observation. In contrast, POMDPs require a multi-step interaction loop where the agent repeatedly takes actions and interprets new environmental feedback to guide its subsequent decisions.

In contrast, SWE scenarios require agents to manage stateful, multi-turn interactions. Successfully applying RL in this context involves several key challenges:

- Long-horizon, multi-turn interaction: Agents must maintain coherence across dozens of steps with context windows spanning hundreds of thousands of tokens.
- Complex, informative feedback: Actions elicit rich outputs (e.g., compiler traces, test logs) that must be interpreted to guide subsequent decisions effectively.
- Data scalability and fidelity: Generating high-quality trajectories requires the reproduction of specific repository states in controlled environments, which limits dataset scale. Large-scale datasets such as SWE-smith [^23] and SWE-rebench [^3] begin to address this gap; we primarily build on the latter.
- Sparse, delayed rewards: Success signals typically emerge only at the end of long action sequences, complicating credit assignment.
- Expensive and noisy evaluation: Unrolling trajectories and subsequent evaluation are costly, and flakiness in tests introduces noise in the reward signal.

In this paper, we address these challenges by developing a complete RL pipeline tailored explicitly to interactive SWE tasks. Our core contributions include:

- A scalable RL framework, based on a modified DAPO algorithm [^25], specifically adapted to handle the demands of long-horizon, multi-turn SWE scenarios.
- Empirical demonstration of RL effectiveness by training a Qwen2.5-72B-Instruct agent that achieves approximately 39% success on the SWE-bench Verified benchmark, doubling the baseline performance of a rejection-fine-tuned agent. Moreover, our agent matches or surpasses top open-weight models, such as DeepSeek-V3-0324 and Qwen3-235B-A22B, on the SWE-rebench benchmark (Table 1).
- Detailed analysis of our RL training methodology, including algorithmic modifications, hyperparameter settings, key findings and discussion of promising future directions for applying RL to LLM-based agents in interactive, stateful environments.

Our results show that RL can be successfully applied to general, interactive environments, advancing the capabilities of open-weight LLM agents beyond single-turn benchmarks and toward real-world scenarios, where future autonomous agents are expected to operate.

## 2 Preliminaries

### 2.1 Task Formulation

We formalize the task of autonomous SWE agent as a Partially Observable Markov Decision Process (POMDP) [^11], defined by the tuple $\langle\mathcal{Z},\mathcal{A},\Omega,T,\mathcal{O},R,\gamma\rangle$:

- $\mathcal{Z}$: Set of true, latent environment states.
- $\mathcal{A}$: Set of actions available to the agent.
- $\Omega$: Set of observations the agent can receive.
- $T(z^{\prime}|z,a)$: Transition probability to state $z^{\prime}$ from state $z$ after action $a$.
- $\mathcal{O}(o|z^{\prime})$: Probability of receiving observation $o$ from state $z^{\prime}$.
- $R(z,a)$: Reward provided from taking action $a$ from state $z$.
- $\gamma\in[0,1]$: Discount factor.

At each step $t$, the environment’s latent state $z_{t}$ is unobservable. Instead, the agent maintains a history $h_{t}$ of all previous actions and observations: $h_{t}=(o_{0},a_{0},o_{1},a_{1},\dots,a_{t-1},o_{t})$. A complete history corresponding to a finished episode is called a trajectory $\tau$. The agent’s policy, parameterized by $\theta$, selects the next action based on this history: $\pi_{\theta}(a_{t}|h_{t})$. Due to the LLM’s autoregressive nature, the policy probability factorizes over tokens within an action:

$$
\pi_{\theta}(a_{t}|h_{t})=\prod_{k=1}^{|a_{t}|}\pi_{\theta}(a_{t,k}|h_{t},a_{t,<k}).
$$

Here, $a_{t,k}$ is the $k$ -th token of the action $a_{t}$, and $|a_{t}|$ is the length of the action in tokens.

In our SWE setting, these components are instantiated as follows:

- Environment State ($z_{t}$): Complete, hidden software environment state, including file system, source code, and running processes.
- Action ($a_{t}$): A command string generated autoregressively by the LLM (potentially with reasoning and tool calls).
- Observation ($o_{t}$): The execution output from a command (typically stdout, stderr, and exit codes). The initial observation $o_{0}$ contains the natural-language description of the task from the GitHub issue.
- History ($h_{t}$): Complete observable history of past actions and observations, conditioning the policy’s next decision.

Our goal is to find a policy $\pi_{\theta}$ that maximizes the expected cumulative reward. The cumulative reward for a trajectory $\tau$ is the sum of rewards over all its steps, $G(\tau)=\sum_{t=0}^{|\tau|-1}R(z_{t},a_{t})$, where $|\tau|$ denotes the number of actions in the trajectory. Given our sparse reward structure where immediate rewards $R(z_{t},a_{t})$ are zero for all non-terminal steps, this simplifies to the terminal reward, which we denote as $R(\tau)$. $R(\tau)$ is $1$ if the final proposed patch passes the validation test suite, and $0$ otherwise. We use this terminal reward as the sole signal for policy optimization.

### 2.2 From PPO to DAPO

Reinforcement learning has traditionally been dominated by *Proximal Policy Optimization* (PPO) [^14], which employs a learned critic to estimate the advantage of each generated action. While powerful, PPO introduces extra overhead and sensitivity to hyperparameter tuning. In our setting, the PPO objective can be the following:

$$
\mathcal{J}_{\text{PPO}}(\theta)=\mathbb{E}_{\begin{subarray}{c}o_{0}\sim\mathcal{D},\\
\tau\sim\pi_{old}(\cdot|o_{0})\end{subarray}}\Biggl[\frac{1}{|\tau|}\sum_{t=0}^{|\tau|-1}\frac{1}{|a_{t}|}\sum_{k=1}^{|a_{t}|}\min\Bigl(\\
\rho_{t,k}(\theta)A_{t,k},\text{clip}(\rho_{t,k}(\theta),1-\varepsilon,1+\varepsilon)A_{t,k}\Bigr)\Biggr],
$$

where $o_{0}\sim\mathcal{D}$ indicates that initial observations are sampled from the distribution of tasks in our dataset. $\rho_{t,k}(\theta)\triangleq\frac{\pi_{\theta}(a_{t,k}|h_{t},a_{t,<k})}{\pi_{\theta_{\text{old}}}(a_{t,k}|h_{t},a_{t,<k})}$ is the probability ratio for the $k$ -th token in action $a_{t}$, and $A_{t,k}$ is its corresponding advantage estimate.

*Group-Relative Policy Optimization* (GRPO) [^16] eliminates the need for a learned advantage estimator. Instead, it computes a Monte Carlo estimate of the advantage for each trajectory by normalizing its terminal reward against the mean and standard deviation of rewards from a group of trajectories sampled from the same policy.

For a given initial observation, GRPO samples a group of $G$ complete trajectories $\{\tau^{(1)},\dots,\tau^{(G)}\}$ from the policy and collects their terminal rewards $\{R(\tau^{(1)}),\dots,R(\tau^{(G)})\}$. The advantage of a trajectory $\tau^{(i)}$ is calculated relative to the average reward of the group, $\bar{R}=\frac{1}{G}\sum_{i=1}^{G}R(\tau^{(i)})$:

$$
\hat{A}^{(i)}=\frac{R(\tau^{(i)})-\bar{R}}{\sigma_{R}+\delta},
$$

where $\sigma_{R}$ is the standard deviation of rewards in the group and $\delta$ is a small constant for numerical stability. This single advantage value $\hat{A}^{(i)}$ is then applied to all tokens within the trajectory $\tau^{(i)}$.

The GRPO objective function uses the same clipped surrogate objective as PPO but substitutes the critic-based advantage with this group-wise advantage estimate:

$$
\mathcal{J}_{\text{GRPO}}(\theta)=\mathbb{E}_{\begin{subarray}{c}o_{0}\sim\mathcal{D},\\
\{\tau^{(i)}\}_{i=1}^{G}\sim\pi_{old}(\cdot|o_{0})\end{subarray}}\Biggl[\\
\frac{1}{G}\sum_{i=1}^{G}\frac{1}{|\tau^{(i)}|}\sum_{t=0}^{|\tau^{(i)}|-1}\frac{1}{|a_{t}^{(i)}|}\sum_{k=1}^{|a_{t}^{(i)}|}\min\Bigl(\\
\quad\rho_{t,k}^{(i)}(\theta)\hat{A}^{(i)},\text{clip}(\rho_{t,k}^{(i)}(\theta),1-\varepsilon,1+\varepsilon)\hat{A}^{(i)}\Bigr)\Biggr].
$$

DAPO extends GRPO by introducing several practical improvements for enhanced stability and efficiency. Key modifications include:

- Clip higher: Instead of a symmetric clip range $(1-\varepsilon,1+\varepsilon)$, DAPO uses asymmetric bounds $(1-\varepsilon_{low},1+\varepsilon_{high})$. Typically, $\varepsilon_{high}$ is set higher than $\varepsilon_{low}$ to better prevent policy’s entropy collapse.
- Dynamic sampling: Sampled commands that carry no learning signal (i.e., $\hat{A}^{(i)}=0$) are filtered out, focusing computation on effective updates.
- Soft overlong punishment: Specifically, when the response exceeds a predefined length threshold, a gradual penalty within a specified interval is applied. Within this interval, the longer the response, the greater the punishment it receives. This penalty is added to the original test-based reward, thereby signaling to the model to avoid excessively long responses.
- Token-level loss: The original GRPO algorithm uses sample-level averaging, where each trajectory has equal weight. DAPO suggests averaging the loss over all tokens in the batch, as shown in Equation 5. This method ensures every token across the batch contributes equally to the gradient, giving greater influence to longer trajectories.

$$
\mathcal{J}_{\text{DAPO}}(\theta)=\mathbb{E}_{\begin{subarray}{c}o_{0}\sim\mathcal{D},\\
\{\tau^{(i)}\}_{i=1}^{G}\sim\pi_{old}(\cdot|o_{0})\end{subarray}}\Biggl[\\
\frac{1}{\sum_{i=1}^{G}\sum_{t=0}^{|\tau^{(i)}|-1}|a_{t}^{(i)}|}\sum_{i=1}^{G}\sum_{t=0}^{|\tau^{(i)}|-1}\sum_{k=1}^{|a_{t}^{(i)}|}\min\Bigl(\\
\quad\rho_{t,k}^{(i)}(\theta)\hat{A}^{(i)},\text{clip}(\rho_{t,k}^{(i)}(\theta),1-\varepsilon_{low},1+\varepsilon_{high})\hat{A}^{(i)}\Bigr)\Biggr].
$$

Our implementation builds on DAPO, but introduces some modifications for the multi-turn SWE setting, further discussed in detail.

### 2.3 Agent Scaffolding

The SWE agent in our work employs a ReAct-style loop [^24], interacting with the environment through a predefined set of tools. The entire action–observation history conditions every decision. We provide an evaluation infrastructure supporting over 7,000 curated SWE tasks, each with a natural-language issue description and a reproducible environment.

Our SWE agent interacts with an environment using the following tools:

- Arbitrary shell commands (ls, cat, grep, etc.).
- An edit command that replaces a specified range of lines in a file with new text. The command requires the agent to provide the replacement text with precise indentation and can operate on either the currently open file or one specified by a file path.
- A submit command that takes no arguments, signals that the agent has finished its work. This action terminates an episode.

Each SWE task includes a GitHub-style issue with a natural language description, a failing test suite that evaluates final patch correctness, and a sandboxed environment initialized from a repository snapshot.

## 3 Reinforcement Learning for Multi-Turn Agents

This section outlines our training methodology, consisting of carefully curated data and a two-phase training pipeline optimized explicitly for multi-turn SWE tasks.

### 3.1 Data

We start from the publicly available SWE-rebench dataset, containing 21,336 tasks sourced from approximately 3,400 Python GitHub repositories. Each task includes a GitHub issue, a ground-truth solution patch, a validation test suite, and a reproducible environment.

To ensure high-quality and stable training, we apply rigorous filtering criteria, resulting in 7,249 tasks selected according to:

- Task correctness: Remove tasks causing test failures due to invalid references or imports (e.g., AttributeError, ImportError), as indicated in task metadata. These cases would require agent to guess particular identifier names.
- Controlled complexity: Include only tasks modifying up to seven files and fewer than 500 lines of code to maintain manageable complexity.
- LLM-assessed quality: Exclude tasks with unclear issue descriptions, overly complex tasks, or flawed test patches according to LLM-generated scores from the original dataset. This is done by removing all problems with the assigned LLM score of $3.0$ in the metadata field.
- Deterministic tests: Remove tasks exhibiting flakiness upon repeated executions of tests (50 trials), ensuring stable training signals. Non-deterministic test behavior occurs mostly due to external service calls or floating-point inaccuracies.

For evaluation, we use the standard SWE-bench Verified benchmark, a 50-problem random subset (referred to as Verified-50) for faster intermediate evaluations, and the latest monthly splits of SWE-rebench (May-June) which are not included in the training set, ensuring a fair and decontamination-free comparison. The full list of Verified-50 problems can be found in Appendix B.

### 3.2 Phase 1: Rejection Fine-Tuning (RFT)

We start from the open-weight Qwen2.5-72B-Instruct model. Out of the box it reaches only $\sim$ 11% on SWE-bench Verified with the dominant failure being incorrect instruction following, resulting in commands that are not in the appropriate format.

To address this problem and warm up the model for the RL stage, we perform rejection fine-tuning (RFT). First, we run the initial checkpoint 10 times on the 7,249 SWE-rebench tasks and keep only those trajectories whose patches pass the test suite. This yields a set of 6,548 successful trajectories, on which we run a single epoch of supervised fine-tuning. During this epoch, we mask every agent turn that triggered an environment-formatting error, thereby focusing the loss on valid actions and improving adherence to the tool structure (see Figure 3). After RFT, the model’s accuracy rises to $\sim$ 20% (Table 1). All hyperparameters for this and subsequent stages are listed in the training details section.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2508.03501/assets/figures/rft_trajectory_example.png)

Figure 3: An example trajectory from the agent’s interaction used in RFT. Only green (successful) assistant turns contribute to training loss.

### 3.3 Phase 2: Multi-Turn RL

The core of our work involves applying RL over thousands of problems in an iterative loop. Each RL iteration includes:

- Problem sampling: A subset of problems is selected from the training pool.
- Rollouts generation: We sample $G=10$ complete trajectories per problem using the current policy.
- Reward computation: The final reward signal, $R_{\text{final}}(\tau)$, combines the binary success reward from testing with a penalty for trajectory length. We add a linear penalty for exceeding a certain number of steps. The reward is computed as follows:
	$$
	\displaystyle R_{\text{final}}(\tau)
	$$
	 
	$$
	\displaystyle=R(\tau)+R_{\text{length}}(\tau)
	$$
	 
	$$
	\displaystyle R_{\text{length}}(\tau)
	$$
	 
	$$
	\displaystyle=\begin{cases}0,&\text{if }|\tau|<L_{\text{thr}}\\
	\frac{L_{\text{thr}}-|\tau|}{T_{\text{max}}-L_{\text{thr}}},&\text{if }|\tau|\geq L_{\text{thr}}\end{cases}
	$$
	Here, $R(\tau)\in\{0,1\}$ is the terminal reward defined in the previous section, $L_{\text{thr}}$ and $T_{\text{max}}$ are hyperparameters representing the penalty threshold and maximum episode length, respectively.
- Advantage estimation: Rewards are averaged and normalized within each 10-sample group; samples with zero advantage are dropped.
- Optimization: We update model’s all parameters using DAPO’s clipped token-level objective.

We begin RL training at a 65k context length, already beyond the default 32k of most open-weight LLMs, but still insufficient for repositories with long stack traces or diff histories. Once performance plateaus at approximately 32%, we switch to a longer context setting where we increase the context window to 131k and double the maximum number of agent turns $T_{\text{max}}$ from 40 to 80. For stable updates in this second stage, we also decrease the high clip bound $\epsilon_{\text{high}}$, increase the batch size, and lower the number of instances sampled on each iteration, i.e., shifting training closer to on-policy mode (see Table 3).

As part of the transition between stages, we perform a one-time curriculum adjustment by refreshing the training pool. We define a task’s cumulative resolution rate as its success rate over all prior training iterations. Tasks are removed from the pool if their cumulative resolution rate exceeds $\frac{2}{3}$ (indicating the task can be solved reliably) or remains at zero (indicating it is likely unsolvable). This filtering reduces the training pool from 7,249 to 2,028 tasks. This strategy focuses computational resources in Stage 2 on problems that are most beneficial for learning.

This second phase boosts performance to 39.0% on SWE-bench Verified. On the held-out SWE-rebench evaluation sets, it achieves 35.0% on the May split and 31.7% on the June split. The significant gap between our final Pass@1 score (39.0%) and Pass@10 score (58.4%) suggests that while the agent’s single best guess may be incorrect, a valid solution frequently exists within its top proposals. This indicates strong potential for application of re-ranking or best-of- $n$ selection mechanisms to further improve performance.

<table><tbody><tr><td>Model</td><td colspan="2">SWE-bench Verified</td><td colspan="2">SWE-rebench May</td><td colspan="2">SWE-rebench June</td></tr><tr><td></td><td>Pass@1</td><td>Pass@10</td><td>Pass@1</td><td>Pass@10</td><td>Pass@1</td><td>Pass@10</td></tr><tr><td>Qwen2.5-72B-Instruct</td><td>11.42 <math><semantics><mo>±</mo> <annotation>\pm</annotation></semantics></math> 0.24</td><td>31.0</td><td>14.50 <math><semantics><mo>±</mo> <annotation>\pm</annotation></semantics></math> 1.33</td><td>40.0</td><td>14.63 <math><semantics><mo>±</mo> <annotation>\pm</annotation></semantics></math> 1.03</td><td>36.6</td></tr><tr><td>+ RFT @ 65k (ours)</td><td>20.46 <math><semantics><mo>±</mo> <annotation>\pm</annotation></semantics></math> 0.42</td><td>43.0</td><td>22.50 <math><semantics><mo>±</mo> <annotation>\pm</annotation></semantics></math> 1.18</td><td>45.0</td><td>20.98 <math><semantics><mo>±</mo> <annotation>\pm</annotation></semantics></math> 1.51</td><td>43.9</td></tr><tr><td>+ Stage 1 RL @ 65k (ours)</td><td>35.74 <math><semantics><mo>±</mo> <annotation>\pm</annotation></semantics></math> 0.28</td><td>54.6</td><td>36.50 <math><semantics><mo>±</mo> <annotation>\pm</annotation></semantics></math> 1.59</td><td>55.0</td><td>31.22 <math><semantics><mo>±</mo> <annotation>\pm</annotation></semantics></math> 0.80</td><td>53.7</td></tr><tr><td>+ Stage 2 RL @ 131k (ours)</td><td>39.04 <math><semantics><mo>±</mo> <annotation>\pm</annotation></semantics></math> 0.50</td><td>58.4</td><td>35.00 <math><semantics><mo>±</mo> <annotation>\pm</annotation></semantics></math> 1.54</td><td>52.5</td><td>31.71 <math><semantics><mo>±</mo> <annotation>\pm</annotation></semantics></math> 1.31</td><td>53.7</td></tr><tr><td>DeepSeek-V3-0324</td><td>39.56 <math><semantics><mo>±</mo> <annotation>\pm</annotation></semantics></math> 0.47</td><td>62.2</td><td>36.75 <math><semantics><mo>±</mo> <annotation>\pm</annotation></semantics></math> 0.92</td><td>60.0</td><td>31.46 <math><semantics><mo>±</mo> <annotation>\pm</annotation></semantics></math> 1.38</td><td>58.5</td></tr><tr><td>Qwen3-235B no-thinking</td><td>25.84 <math><semantics><mo>±</mo> <annotation>\pm</annotation></semantics></math> 0.37</td><td>54.4</td><td>27.25 <math><semantics><mo>±</mo> <annotation>\pm</annotation></semantics></math> 1.15</td><td>57.5</td><td>22.93 <math><semantics><mo>±</mo> <annotation>\pm</annotation></semantics></math> 2.16</td><td>48.8</td></tr><tr><td>Qwen3-32B no-thinking</td><td>20.40 <math><semantics><mo>±</mo> <annotation>\pm</annotation></semantics></math> 0.34</td><td>44.0</td><td>21.75 <math><semantics><mo>±</mo> <annotation>\pm</annotation></semantics></math> 1.54</td><td>50.0</td><td>17.56 <math><semantics><mo>±</mo> <annotation>\pm</annotation></semantics></math> 1.30</td><td>36.6</td></tr><tr><td>Llama4 Maverick</td><td>15.84 <math><semantics><mo>±</mo> <annotation>\pm</annotation></semantics></math> 0.54</td><td>47.2</td><td>19.00 <math><semantics><mo>±</mo> <annotation>\pm</annotation></semantics></math> 1.72</td><td>50.0</td><td>13.66 <math><semantics><mo>±</mo> <annotation>\pm</annotation></semantics></math> 1.79</td><td>39.0</td></tr></tbody></table>

Table 1: Comparison of our models against open-weight baselines on SWE-bench Verified and SWE-rebench. Pass@1 metrics are averaged over 10 runs and reported with the standard error of the mean. Our final model and the baseline models are evaluated with a 131k context length; our intermediate models are evaluated at the context length used during their respective training stages (65k). All models use their default decoding parameters as specified in their Hugging Face configuration.

## 4 Training Details and Results

### 4.1 Main Results

Our two-phase procedure yields substantial improvements. Rejection fine-tuning provides an initial performance boost by enhancing the model’s ability to interact correctly with the environment. This is followed by over 100 RL iterations that progressively refine the agent’s policy. The final model achieves $39.0\%$ on the full SWE-bench Verified. Performance trends are shown in Figure 4.

For head-to-head comparisons, we evaluate DeepSeek-V3-0324, Llama-4 Maverick, Qwen3-235B no-thinking, and Qwen3-32B no-thinking within the same environment and tool setup (see Table 1) on both SWE-bench Verified and SWE-rebench. To benchmark our final model against specialized SWE agents, we summarize recent results in Table 2.

| Agent | Base Model | SWE-bench Ver. before | SWE-bench Ver. after | Teacher Distillation |
| --- | --- | --- | --- | --- |
| Ours | Qwen2.5-72B-Instruct | 11.4% | 39.0% | No |
| DeepSWE-32B, [^9] | Qwen3-32B | 23% | 42.2% | No |
| SkyRL-Agent-14B-v0, [^4] | Qwen3-14B | 18.0% | 21.6% | No |
| SWE-Fixer-72B, [^21] | Qwen2.5-72B-Base | – | 30.2% | Yes |
| SWE-agent-LM-32B, [^23] | Qwen2.5-Coder-32B-Instruct | 14.3% | 40.2% | Yes |
| Skywork-SWE-32B, [^27] | Qwen2.5-Coder-32B-Instruct | 6.4% | 38.0% | Yes |
| SWE-Gym-32B, [^12] | Qwen2.5-Coder-32B-Instruct | 7.0% | 20.6% | Yes |
| SWESynInfer-72B, [^10] | Qwen2.5-72B-Instruct | 25.4% | 30.2% | Yes |
| R2EGym-Agent-32B, [^6] | Qwen2.5-Coder | 7.0% | 34.4% | Yes |

Table 2: Comparison of specialized SWE agents on SWE-bench Verified. The “before” column shows the Pass@1 score of the base model prior to training, while the “after” column shows the final performance of the fully trained agent.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2508.03501/assets/figures/rl_iterations.png)

Figure 4: A detailed performance trend of RL-trained agent over all iterations. Statistics include Pass@1, Pass@10, the number of submit commands and the average number of steps per trajectory. All metrics are computed on Verified-50.

### 4.2 Infrastructure

The described process is based on a fully synchronous RL training process, meaning that inference and training stages are interleaved: once rollout generation is completed, trajectories are used for training. This setup enables fully on-policy training with no policy lag between sampling and updates. However, as described earlier, we sample 10 trajectories for each problem. This results in 2-8 optimization steps per iteration depending on batch size and the number of trajectories with $\hat{A}^{(i)}\neq 0$.

We believe that asynchronous frameworks can offer greater scalability, but to eliminate complexities like trajectory lag, a synchronous framework is a reasonable choice for these experiments. A key drawback of the synchronous approach is the “straggler” problem: the time for each generation iteration is determined by the single slowest trajectory to complete, which can reduce overall throughput.

To enable full-parameter training on sequences up to 131k tokens, we leverage context parallelism, which partitions long sequences across GPUs. All training and inference are conducted on 16 H200 nodes.

We describe more details regarding infrastructure in Appendix C.

### 4.3 Hyperparameters

#### Inference.

For rollout generation, we run the model with a temperature of 1.0, explicitly disabling all other decoding parameters such as top\_p, top\_k, min\_p, repetition\_penalty and others. This ensures unbiased sampling, which is critical for the validity of importance sampling ratios used during training. We demonstrate the dangers of biased sampling procedures later in our findings.

#### Training.

For RFT, we perform one epoch of training at a 65k context length, with a learning rate of $5\times 10^{-6}$, AdamW optimizer with weight decay of 0.1, 10 warmup steps, and a cosine decay scheduler with $\textit{end\_lr}=0.0$. We use a batch size of 64, resulting in 50 gradient update steps.

For the RL pipeline, we list all hyperparameters changed across stages in Table 3. Both setups share common settings: $\textit{gradient\_clipping}=1.0$; AdamW with $\beta_{1}=0.9$, $\beta_{2}=0.999$, $\varepsilon=1\times 10^{-8}$, weight decay of 0.1; learning rate of $10^{-6}$ and $\textit{num\_epochs}=1$.

| Training Stage | Problems / Iteration | Total Problems | Batch Size | Clip Range |
| --- | --- | --- | --- | --- |
| Stage 1 | 300 | 7249 | 128 | \[0.2, 0.3\] |
| Stage 2 | 100 | 2028 | 256 | \[0.2, 0.26\] |

Table 3: Key hyperparameters across the two RL training stages.

For Qwen2.5-72B-instruct models we use YaRN positional encoding with $\textit{factor}=4.0$ to enable 131k context length training and inference.

### 4.4 Findings

A commonly adopted practice in dataset preparation, also mentioned in DeepSWE [^9], is to filter or mask trajectories that exceed the model’s maximum context length. This is often motivated by the desire to reduce reward noise. However, we find this must be applied with caution. Manually crafted heuristics can introduce biases, breaking the assumption that the training data is sampled from the same distribution as the policy being optimized. In our setting, these long trajectories often occur when the agent is stuck in a repetitive loop. By discarding these trajectories, one also discards specific negative examples of this failure mode. As a result, the agent is not penalized for this looping behavior and fails to learn how to break out of such cycles, which can lead to it occurring more frequently during training.

We also observe a more subtle instability related to discrepancies between sampling and training. Midway through training, we upgraded the vLLM [^8] runtime, which introduced internal changes to decoding parameters, enabling top\_k and min\_p by default. While this initially improved evaluation metrics, it caused performance to degrade after 5–10 training iterations. We attribute this to a mismatch between the rollout and training distributions. The DAPO objective relies on the importance sampling ratio $\rho_{t,k}(\theta)$ to weight the advantage term. This is valid only if rollouts are sampled from $\pi_{old}$. Using different decoding parameters amounts to sampling from a modified distribution, $\pi_{\text{rollout}}\neq\pi_{old}$, which makes the ratio an incorrect estimator and leads to biased gradient updates. Once the sampling configuration was restored, stability recovered.

## 5 Discussion and Future Work

Our work successfully demonstrates that modern reinforcement learning algorithms, specifically a modified DAPO, can train capable agents for complex, interactive software engineering tasks. This process, however, highlights fundamental challenges in agent-based learning and reveals several key directions for future research:

- Sparse Rewards and Credit Assignment. A fundamental challenge is that the agent receives only a single binary success signal at the end of a long trajectory. This sparsity makes it difficult to perform credit assignment, that is, to identify which specific actions in a long sequence were crucial for the outcome. Broadcasting a single advantage estimate across thousands of preceding tokens can result in noisy and inefficient policy updates. Several research directions could address this: (i) reward shaping, which involves designing intermediate rewards based on signals like passing a subset of tests or reducing compiler errors; (ii) training an auxiliary critic or value head to provide step-level advantage estimates, enabling more granular updates; and (iii) prefix sampling, where rollouts are initiated from a shared non-empty trajectory prefix to better isolate the impact of later decisions.
- Uncertainty and Risk-Awareness. The binary, success-based reward objective encourages the agent to submit a patch “at any cost”, which leads it to act confidently even when a solution is unlikely. For real-world deployment, agents must recognize when to abstain. This requires better uncertainty estimation; for instance, by training the model to explicitly output a confidence score or by using the policy’s output entropy as a proxy for uncertainty. Such estimates would enable a precision-recall trade-off, allowing the agent to decide when to halt or to apply more compute for best-of- $n$ selection without an external outcome-supervision model.

## References

## Appendix A Related Work

#### Software engineering agents.

Early systems, notably SWE-agent [^22], demonstrated that large language models (LLMs) could effectively operate within sandboxed software environments using predefined toolkits (e.g., shell commands, code editors) and be evaluated via automated unit tests [^7]. Subsequent frameworks introduced alternative scaffolding and prompting strategies to enhance model interactions, including Agentless [^20], OpenHands [^19], and Moatless [^1]. Our work closely aligns with the original SWE-agent design, leveraging similar prompting structures and tool configurations, enabling a direct evaluation of reinforcement learning improvements within this established context.

#### Strategies to improve SWE agents.

Prior efforts to enhance SWE agent performance beyond improving scaffoldings broadly focus on either *test-time exploration* or *model-level improvements* through supervised training. Test-time exploration strategies, such as Monte Carlo Tree Search (MCTS) [^1] and guided 1-step lookahead [^26], significantly boost task success by exploring multiple solution trajectories. However, these methods are computationally expensive and sometimes introduce infrastructure complexity due to operations such as environment rollbacks and parallel execution paths.

Alternatively, many efforts have targeted model-level improvements via supervised fine-tuning (SFT) using expert-curated demonstrations. Prominent examples include SWE-smith [^23], SWE-Fixer [^21], and Skywork-SWE [^27], all of which achieved success by training open-weight models on extensive demonstration data. In contrast, our method relies exclusively on self-generated interaction data obtained through direct RL training. This approach simplifies data collection, eliminates need for strong teacher models and opens way for iterative self-improvement.

#### Reinforcement learning for coding.

Reinforcement learning (RL) has shown notable success in structured reasoning domains like mathematics, exemplified by DeepSeekMath [^16] and Seed-Thinking-v1.5 [^15]. In the coding domain specifically, RL applications traditionally focused on single-turn code generation tasks [^5], neglecting the complexities introduced by multi-turn interactions and extended contexts.

Recent advancements, however, have started addressing these complexities. Concurrent to our work, DeepSWE [^9] successfully scaled critic-free RL training to a 32B-parameter model (Qwen3-32B), while Sky-RL [^4] introduced an asynchronous RL pipeline for long-context tasks. Our work contributes to this emerging area by demonstrating a complete, multi-stage methodology to successfully apply RL to a larger 72B-parameter model with a 131k token context.

## Appendix B Verified-50 Problems

The Verified-50 dataset, which we curate from the SWE-bench Verified by randomly selecting problems, contains the following problem instances:

⬇

sympy\_\_sympy-22080

django\_\_django-15315

django\_\_django-11333

matplotlib\_\_matplotlib-20826

django\_\_django-11532

django\_\_django-16642

django\_\_django-14855

sphinx-doc\_\_sphinx-8721

pylint-dev\_\_pylint-4604

sympy\_\_sympy-13615

django\_\_django-13089

django\_\_django-15987

django\_\_django-14725

sympy\_\_sympy-14248

pytest-dev\_\_pytest-7982

django\_\_django-15280

scikit-learn\_\_scikit-learn-13142

pytest-dev\_\_pytest-5809

matplotlib\_\_matplotlib-23299

django\_\_django-16560

django\_\_django-15103

sympy\_\_sympy-16792

django\_\_django-14007

psf\_\_requests-2317

django\_\_django-11880

django\_\_django-16136

django\_\_django-16661

sympy\_\_sympy-17139

sphinx-doc\_\_sphinx-8595

sympy\_\_sympy-14531

django\_\_django-10880

sympy\_\_sympy-19346

sphinx-doc\_\_sphinx-9229

django\_\_django-11265

matplotlib\_\_matplotlib-25332

scikit-learn\_\_scikit-learn-13135

pydata\_\_xarray-6744

pydata\_\_xarray-6461

sympy\_\_sympy-15017

django\_\_django-13417

matplotlib\_\_matplotlib-24870

django\_\_django-15368

django\_\_django-11095

django\_\_django-15554

pydata\_\_xarray-6992

django\_\_django-15863

django\_\_django-13363

sympy\_\_sympy-13852

django\_\_django-14017

pylint-dev\_\_pylint-4661

## Appendix C Infrastructure Details

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2508.03501/assets/figures/pipeline_overview.png)

Figure 5: One synchronous iteration of the RL pipeline (green: GPU heavy; yellow: CPU heavy).

The entire pipeline of distributed agent execution and evaluation is orchestrated at scale using Kubernetes and Tracto AI [^17]. The primary compute units are pods, each provisioned with eight H200 GPUs, 32 CPUs, and 960 GiB of CPU RAM.

Within this environment, model training is conducted using an internal framework built on JAX, while inference is accelerated using the vLLM framework [^8], version 0.7.4.

Figure 5 demonstrates the workflow of a single synchronous iteration. As illustrated, the training process for a new policy begins only after the verification for all trajectories in the batch is complete.

[^1]: Antoniades, A.; Örwall, A.; Zhang, K.; Xie, Y.; Goyal, A.; and Wang, W. 2025. SWE-Search: Enhancing Software Agents with Monte Carlo Tree Search and Iterative Refinement. arXiv:2410.20285.

[^2]: AugmentCode. 2025. AugmentCode: AI Software Developer Platform.

[^3]: Badertdinov, I.; Golubev, A.; Nekrashevich, M.; Shevtsov, A.; Karasik, S.; Andriushchenko, A.; Trofimova, M.; Litvintseva, D.; and Yangel, B. 2025. SWE-rebench: An Automated Pipeline for Task Collection and Decontaminated Evaluation of Software Engineering Agents. arXiv:2505.20411.

[^4]: Cao, S.; Hegde, S.; Li, D.; Griggs, T.; Liu, S.; Tang, E.; Pan, J.; Wang, X.; Malik, A.; Neubig, G.; Hakhamaneshi, K.; Liaw, R.; Moritz, P.; Zaharia, M.; Gonzalez, J. E.; and Stoica, I. 2025. SkyRL-v0: Train Real-World Long-Horizon Agents via Reinforcement Learning.

[^5]: Dou, S.; Liu, Y.; Jia, H.; Xiong, L.; Zhou, E.; Shen, W.; Shan, J.; Huang, C.; Wang, X.; Fan, X.; Xi, Z.; Zhou, Y.; Ji, T.; Zheng, R.; Zhang, Q.; Huang, X.; and Gui, T. 2024. StepCoder: Improve Code Generation with Reinforcement Learning from Compiler Feedback. arXiv:2402.01391.

[^6]: Jain, N.; Singh, J.; Shetty, M.; Zheng, L.; Sen, K.; and Stoica, I. 2025. R2E-Gym: Procedural Environments and Hybrid Verifiers for Scaling Open-Weights SWE Agents. arXiv:2504.07164.

[^7]: Jimenez, C. E.; Yang, J.; Wettig, A.; Yao, S.; Pei, K.; Press, O.; and Narasimhan, K. 2024. SWE-bench: Can Language Models Resolve Real-World GitHub Issues? arXiv:2310.06770.

[^8]: Kwon, W.; Li, Z.; Zhuang, S.; Sheng, Y.; Zheng, L.; Yu, C. H.; Gonzalez, J. E.; Zhang, H.; and Stoica, I. 2023. Efficient Memory Management for Large Language Model Serving with PagedAttention. In *Proceedings of the ACM SIGOPS 29th Symposium on Operating Systems Principles*.

[^9]: Luo, M.; Jain, N.; Singh, J.; Tan, S.; Patel, A.; Wu, Q.; Ariyak, A.; Cai, C.; Venkat, T.; Zhu, S.; Athiwaratkun, B.; Roongta, M.; Zhang, C.; Li, L. E.; Popa, R. A.; Sen, K.; and Stoica, I. 2025. DeepSWE: Training a State-of-the-Art Coding Agent from Scratch by Scaling RL. https://pretty-radio-b75.notion.site/DeepSWE-Training-a-Fully-Open-sourced-State-of-the-Art-Coding-Agent-by-Scaling-RL-22281902c1468193aabbe9a8c59bbe33. Notion Blog.

[^10]: Ma, Y.; Cao, R.; Cao, Y.; Zhang, Y.; Chen, J.; Liu, Y.; Liu, Y.; Li, B.; Huang, F.; and Li, Y. 2024. Lingma SWE-GPT: An Open Development-Process-Centric Language Model for Automated Software Improvement. arXiv:2411.00622.

[^11]: Murphy, K. 2025. Reinforcement Learning: An Overview. arXiv:2412.05265.

[^12]: Pan, J.; Wang, X.; Neubig, G.; Jaitly, N.; Ji, H.; Suhr, A.; and Zhang, Y. 2025. Training Software Engineering Agents and Verifiers with SWE-Gym. arXiv:2412.21139.

[^13]: Refact AI. 2025. Refact.ai: AI Software Engineering Agent.

[^14]: Schulman, J.; Wolski, F.; Dhariwal, P.; Radford, A.; and Klimov, O. 2017. Proximal Policy Optimization Algorithms. arXiv:1707.06347.

[^15]: Seed, B.;:; Chen, J.; Fan, T.; Liu, X.; Liu, L.; Lin, Z.; Wang, M.; Wang, C.; Wei, X.; Xu, W.; Yuan, Y.; Yue, Y.; Yan, L.; Yu, Q.; Zuo, X.; Zhang, C.; Zhu, R.; An, Z.; Bai, Z.; Bao, Y.; Bin, X.; Chen, J.; Chen, F.; Chen, H.; Chen, R.; Chen, L.; Chen, Z.; Chen, J.; Chen, S.; Chen, K.; Chen, Z.; Chen, J.; Chen, J.; Chi, J.; Dai, W.; Dai, N.; Dai, J.; Dou, S.; Du, Y.; Du, Z.; Duan, J.; Dun, C.; Fan, T.-H.; Feng, J.; Feng, J.; Feng, Z.; Fu, Y.; Fu, W.; Fu, H.; Ge, H.; Guo, H.; Han, M.; Han, L.; Hao, W.; Hao, X.; He, Q.; He, J.; He, F.; Heng, W.; Hong, Z.; Hou, Q.; Hu, L.; Hu, S.; Hu, N.; Hua, K.; Huang, Q.; Huang, Z.; Huang, H.; Huang, Z.; Huang, T.; Huang, W.; Jia, W.; Jia, B.; Jia, X.; Jiang, Y.; Jiang, H.; Jiang, Z.; Jiang, K.; Jiang, C.; Jiao, J.; Jin, X.; Jin, X.; Lai, X.; Li, Z.; Li, X.; Li, L.; Li, H.; Li, Z.; Wan, S.; Wang, Y.; Li, Y.; Li, C.; Li, N.; Li, S.; Li, X.; Li, X.; Li, A.; Li, Y.; Liang, N.; Liang, X.; Lin, H.; Lin, W.; Lin, Y.; Liu, Z.; Liu, G.; Liu, G.; Liu, C.; Liu, Y.; Liu, G.; Liu, J.; Liu, C.; Liu, D.; Liu, K.; Liu, S.; Liu, Q.; Liu, Y.; Liu, K.; Liu, G.; Liu, B.; Long, R.; Lou, W.; Lou, C.; Luo, X.; Luo, Y.; Lv, C.; Lv, H.; Ma, B.; Ma, Q.; Ma, H.; Ma, Y.; Ma, J.; Ma, W.; Ma, T.; Mao, C.; Min, Q.; Nan, Z.; Ning, G.; Ou, J.; Pan, H.; Pang, R.; Peng, Y.; Peng, T.; Qian, L.; Qian, L.; Qiao, M.; Qu, M.; Ren, C.; Ren, H.; Shan, Y.; Shen, W.; Shen, K.; Shen, K.; Sheng, G.; Shi, J.; Shi, W.; Shi, G.; Cao, S. S.; Song, Y.; Song, Z.; Su, J.; Sun, Y.; Sun, T.; Sun, Z.; Wan, B.; Wang, Z.; Wang, X.; Wang, X.; Wang, S.; Wang, J.; Wang, Q.; Wang, C.; Wang, S.; Wang, Z.; Wang, C.; Wang, J.; Wang, S.; Wang, X.; Wang, Z.; Wang, Y.; Wang, W.; Wang, T.; Wei, C.; Wei, H.; Wei, Z.; Wei, S.; Wu, Z.; Wu, Y.; Wu, Y.; Wu, B.; Wu, S.; Wu, J.; Wu, N.; Wu, S.; Wu, J.; Xi, C.; Xia, F.; Xian, Y.; Xiang, L.; Xiang, B.; Xiao, B.; Xiao, Z.; Xiao, X.; Xiao, Y.; Xin, C.; Xin, S.; Xiong, Y.; Xu, J.; Xu, Z.; Xu, C.; Xu, J.; Xu, Y.; Xu, W.; Xu, Y.; Xu, S.; Yan, S.; Yan, S.; Yang, Q.; Yang, X.; Yang, T.; Yang, Y.; Yang, Y.; Yang, X.; Yang, Z.; Yang, G.; Yang, Y.; Yao, X.; Yi, B.; Yin, F.; Yin, J.; Ying, Z.; Yu, X.; Yu, H.; Yu, S.; Yu, M.; Yu, H.; Yuan, S.; Yuan, J.; Zeng, Y.; Zhan, T.; Zhang, Z.; Zhang, Y.; Zhang, M.; Zhang, W.; Zhang, R.; Zhang, Z.; Zhang, T.; Zhang, X.; Zhang, Z.; Zhang, S.; Zhang, W.; Zhang, X.; Zhang, Y.; Zhang, Y.; Zhang, G.; Zhang, H.; Zhang, Y.; Zheng, R.; Zheng, N.; Zheng, Z.; Zheng, Y.; Zheng, C.; Zhi, X.; Zhong, W.; Zhong, C.; Zhong, Z.; Zhong, B.; Zhou, X.; Zhou, N.; Zhou, H.; Zhu, H.; Zhu, D.; Zhu, W.; and Zuo, L. 2025. Seed1.5-Thinking: Advancing Superb Reasoning Models with Reinforcement Learning. arXiv:2504.13914.

[^16]: Shao, Z.; Wang, P.; Zhu, Q.; Xu, R.; Song, J.; Bi, X.; Zhang, H.; Zhang, M.; Li, Y. K.; Wu, Y.; and Guo, D. 2024. DeepSeekMath: Pushing the Limits of Mathematical Reasoning in Open Language Models. arXiv:2402.03300.

[^17]: TractoAI. 2025. TractoAI: Cloud platform for deploying, scaling, and monitoring AI and Big Data workloads.

[^18]: Trae. 2025. Trae: AI Software Developer Platform.

[^19]: Wang, X.; Li, B.; Song, Y.; Xu, F. F.; Tang, X.; Zhuge, M.; Pan, J.; Song, Y.; Li, B.; Singh, J.; Tran, H. H.; Li, F.; Ma, R.; Zheng, M.; Qian, B.; Shao, Y.; Muennighoff, N.; Zhang, Y.; Hui, B.; Lin, J.; Brennan, R.; Peng, H.; Ji, H.; and Neubig, G. 2025. OpenHands: An Open Platform for AI Software Developers as Generalist Agents. arXiv:2407.16741.

[^20]: Xia, C. S.; Deng, Y.; Dunn, S.; and Zhang, L. 2024. Agentless: Demystifying LLM-based Software Engineering Agents. arXiv:2407.01489.

[^21]: Xie, C.; Li, B.; Gao, C.; Du, H.; Lam, W.; Zou, D.; and Chen, K. 2025. SWE-Fixer: Training Open-Source LLMs for Effective and Efficient GitHub Issue Resolution. arXiv:2501.05040.

[^22]: Yang, J.; Jimenez, C. E.; Wettig, A.; Lieret, K.; Yao, S.; Narasimhan, K.; and Press, O. 2024. SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering. arXiv:2405.15793.

[^23]: Yang, J.; Leret, K.; Jimenez, C. E.; Wettig, A.; Khandpur, K.; Zhang, Y.; Hui, B.; Press, O.; Schmidt, L.; and Yang, D. 2025. SWE-smith: Scaling Data for Software Engineering Agents. arXiv:2504.21798.

[^24]: Yao, S.; Zhao, J.; Yu, D.; Du, N.; Shafran, I.; Narasimhan, K.; and Cao, Y. 2023. ReAct: Synergizing Reasoning and Acting in Language Models. arXiv:2210.03629.

[^25]: Yu, Q.; Zhang, Z.; Zhu, R.; Yuan, Y.; Zuo, X.; Yue, Y.; Dai, W.; Fan, T.; Liu, G.; Liu, L.; Liu, X.; Lin, H.; Lin, Z.; Ma, B.; Sheng, G.; Tong, Y.; Zhang, C.; Zhang, M.; Zhang, W.; Zhu, H.; Zhu, J.; Chen, J.; Chen, J.; Wang, C.; Yu, H.; Song, Y.; Wei, X.; Zhou, H.; Liu, J.; Ma, W.-Y.; Zhang, Y.-Q.; Yan, L.; Qiao, M.; Wu, Y.; and Wang, M. 2025. DAPO: An Open-Source LLM Reinforcement Learning System at Scale. arXiv:2503.14476.

[^26]: Zainullina, K.; Golubev, A.; Trofimova, M.; Polezhaev, S.; Badertdinov, I.; Litvintseva, D.; Karasik, S.; Fisin, F.; Skvortsov, S.; Nekrashevich, M.; Shevtsov, A.; and Yangel, B. 2025. Guided Search Strategies in Non-Serializable Environments with Applications to Software Engineering Agents. arXiv:2505.13652.

[^27]: Zeng, L.; Li, Y.; Xiao, Y.; Li, C.; Liu, C. Y.; Yan, R.; Wei, T.; He, J.; Song, X.; Liu, Y.; and Zhou, Y. 2025. Skywork-SWE: Unveiling Data Scaling Laws for Software Engineering in LLMs. arXiv:2506.19290.