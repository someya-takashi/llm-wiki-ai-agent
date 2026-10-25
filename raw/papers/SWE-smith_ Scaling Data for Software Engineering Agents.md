---
title: "SWE-smith: Scaling Data for Software Engineering Agents"
source: "https://ar5iv.labs.arxiv.org/html/2504.21798"
author:
published:
created: 2026-10-26
description: "Despite recent progress in Language Models (LMs) for software engineering, collecting training data remains a significant pain point.Existing datasets are small, with at most s of training instances from  or fewer Git…"
tags:
  - "clippings"
---
John Yang <sup>1</sup>    Kilian Lieret <sup>2</sup>    Carlos E. Jimenez <sup>2</sup>    Alexander Wettig <sup>2</sup>    Kabir Khandpur <sup>3</sup>    Yanzhe Zhang <sup>1</sup>    Binyuan Hui <sup>4</sup>    Ofir Press <sup>2</sup>    Ludwig Schmidt <sup>1</sup>    Diyi Yang <sup>1</sup>    <sup>1</sup> Stanford University    <sup>2</sup> Princeton University    <sup>3</sup> Indepedent    <sup>4</sup> Alibaba Qwen

###### Abstract

Despite recent progress in Language Models (LMs) for software engineering, collecting training data remains a significant pain point. Existing datasets are small, with at most $1{,}000$ s of training instances from $11$ or fewer GitHub repositories. The procedures to curate such datasets are often complex, necessitating hundreds of hours of human labor; companion execution environments also take up several terabytes of storage, severely limiting their scalability and usability. To address this pain point, we introduce SWE-smith, a novel pipeline for generating software engineering training data at scale. Given any Python codebase, SWE-smith constructs a corresponding execution environment, then automatically synthesizes $100$ s to $1{,}000$ s of task instances that break existing test(s) in the codebase. Using SWE-smith, we create a dataset of $50$ k instances sourced from $128$ GitHub repositories, an order of magnitude larger than all previous works. We train SWE-agent-LM-32B, achieving $40.2$ % Pass@1 resolve rate on the SWE-bench Verified benchmark, state of the art among open source models. We open source SWE-smith (collection procedure, task instances, trajectories, models) to lower the barrier of entry for research in LM systems for automated software engineering. All assets available at [https://swesmith.com](https://swesmith.com/).

## 1 Introduction

Figure 1: Scaling task instances (left) and performance (right) for SWE-agent’s with SWE-smith. Using SWE-smith, we can create $100$ s to $1000$ s of instances for any Python codebase, enabling us to train SWE-agent-LM-32B which achieves $40.2$ % on SWE-bench Verified.

Language Model (LM) agents, such as SWE-agent [^49] or OpenHands [^38], have made remarkable progress towards automating software engineering (SE) tasks, as tracked by benchmarks such as SWE-bench [^18]. However, the most effective agents still rely on proprietary LMs, as building open source LMs for SE remains bottlenecked by the lack of large-scale, high-quality training data. To ensure that open research remains relevant in this field, it is critical to develop infrastructure for collecting software engineering training data at scale.

The current open-source ecosystem offers two kinds of data sources to train LMs on SE tasks. One simple approach is to crawl pull requests (PRs) and issues from GitHub repositories. However, without execution environments or tests, these instances offer no reliable way of validating generated solutions, and LMs are limited to learning from the surface form of code [^44] or via rewards based on superficial string similarity [^41].

In contrast, SWE-bench provides reliable validation by running unit tests against proposed solutions. Another line of work has simply extended the SWE-bench collection strategy to a new set of repositories for training purposes [^31]. This produces flexible environments for training and distilling LM agents, since we can generate agent trajectories and filter them based on the unit test results. However, the scalability of this approach is severely limited by the challenges associated with SWE-bench’s collection strategy. SWE-bench’s filtering process leaves only a small number of PRs that not only resolve a Github issue, but also make meaningful changes to unit tests. Also, setting up execution environments for each instance requires a substantial amount of human intervention.

In this paper, we introduce the SWE-smith toolkit, which marries the flexible execution environments of SWE-bench with scalable instance collection (Figure 1). SWE-smith features several techniques to automatically synthesize bugs in existing GitHub repositories, such as (1) generating errant rewrites of functions with an LM, (2) procedurally modifying the abstract syntax tree (AST) of functions, (3) undoing PRs, and (4) combining bugs. Our key insight is that execution-based validation can not only validate proposed solutions, but also identify bug candidates which cause substantial software regression (i.e., break tests).

Figure 2: SWE-smith creates training data for software engineering agents by crafting bugs into real codebases. Given a codebase, we employ several strategies to create task instances that break existing tests. Using SWE-smith, we create $50$ k+ task instances with execution environments from $128$ real world repositories.

In a nutshell, SWE-smith puts forth the following task creation workflow, as shown in Figure 2. Given a codebase, we automatically set up a corresponding environment using SWE-agent [^49]. Within this environment, we then use the aforementioned techniques to synthesize $100$ s to $1,000$ s of task instances. Finally, we craft realistic issue descriptions automatically with LMs. SWE-smith’s design significantly reduces the amount of human labor and storage required for constructing execution environments. Using SWE-smith, we create a dataset of $50$ k task instances across $128$ real-world GitHub repositories.

Using the SWE-smith dataset, we achieve a new open-weight state of the art result on SWE-bench verified. Using the SWE-smith task instances, we generate $5{,}016$ expert trajectories with Claude 3.7 Sonnet and fine-tune Qwen 2.5 Coder Instruct $32$ B. The resulting LM, SWE-agent-LM-32B, achieves $40.2\%$ (+ $33.4$ %) on SWE-bench Verified in a single attempt, without inference-time scaling. This sets a new state of the art for open-weight models.

The scale and diversity of the SWE-smith dataset enables us to begin establishing truths and investigate interesting phenomena about developing SWE-agents. Training on more instances, bug types, and repositories helps. LM generated issue text approximates real ones effectively. Using SWE-smith, we find that it’s possible to optimize LMs to perform well for specific repositories while only suffering minor generalization loss.

We release SWE-smith as an open-source toolkit — including instances, environments, and trajectories — to catalyze the development of stronger open-source LM agents.

## 2 SWE-smith: Software Task Generation at Scale

The core principle of SWE-smith’s collection strategy is to define an execution environment first, and then synthesize task instances within the environment. Conceptually, this is a simple inversion of SWE-bench’s approach, which instead prioritizes identifying task instances, and then attempts to build an environment for each. In this section, we describe the procedure in detail and show how, in practice, SWE-smith scales significantly better in terms of repositories, task instances, and storage.

### 2.1 Collection

Building execution environments for repositories with passing tests. Given a repository, we run SWE-agent [^49] on the latest commit for at most $100$ steps, instructing it to install the codebase and run the test suite. We then manually verify the installation and testing instructions, check if more than $80$ % of existing tests pass, and finally create a Docker image for the repository. We target repositories for the $5,000$ most downloaded packages listed in the Python Package Index (PyPI) as of November 18, 2024, sort the PyPI packages by GitHub stars, and then remove any PyPI package with less than $1,000$ stars, as well as all $12$ SWE-bench test repositories from consideration. More in §A.2.

Creating task instance candidates. Per repository, we employ four different strategies to create candidates. As shown in Figure 2, each strategy takes in a repository as input, then produces task instance candidates represented as.diff files. Extensive details in §B.

- LM Generation: Per repository, we identify all programmatic entities (functions, classes), then take two approaches: (1) provide an LM with the function and prompt it to introduce errant modifications (henceforth referred to as “LM Modify”), and (2) given only the function header and docstring, ask the LM to rewrite it (“LM Rewrite”). More in §B.1.
- Procedural Modification: Per function, we acquire an abstract syntax tree (AST) representation of the code, then randomly perform one or more transformations (e.g., remove a conditional/loop, change an operator, + $11$ more. See Table 8). More in §B.2.
- Combine Bugs: LM generation and Procedural Modification task instances exclusively edit one function or class. To create more complex tasks that require editing multiple portions of the codebase, we devise a “Patch Combination” strategy that creates a task instance by aggregating candidates from the same file(s) or module(s). More in §B.3.
- Invert PRs (or “PR Mirror”): Per repository, we collect all PRs that modify Python files. Per PR, we attempt to *undo* its revisions in the current version of the repository. To achieve this, we provide an LM with the PR’s code changes (a.diff plaintext) and prompt it to rewrite each affected file such that the PR edits are reverted. Unlike SWE-bench, we do not check out the PR’s base commit, as the install specifications determined in the previous step may not be compatible with older versions of the repo. More in §B.4.

Execution-based validation of candidates. We apply each candidate patch to the corresponding repository, run the test suite, and only keep patches that break one or more existing, passing tests (referred to as Fail-to-Pass or F2P test(s)). For efficiency purposes, we also limit testing runtime to two minutes; bug candidates that cause test runtimes in excess of this time limit are discarded. Minor additional details in §A.3.

Generating problem statements. The issue text associated with a bug can significantly alter the difficulty and feasibility of the task instance. Detailed descriptions of “expected” vs. “observed” behavior or bug-reproduction code in issue text heavily affect an agent’s capacity to localize bugs or iterate on proposed solutions. We explore several techniques covered fully in §D, and ultimately settle on a simple strategy. Per task instance, we provide an LM with the.diff patch, source code of a random F2P test, and execution output from running the repository’s test suite with the bug patch applied. We prompt the LM for GitHub issue-style text that includes reproduction code based on the F2P test.

What human labor remains? The steps requiring manual effort are (1) parsing the correct installation setup procedures from the agent trajectory ($\sim 7$ min per repository), and (2) implementing the parser for test outputs ($\sim 1$ min per repository). Step two requires very little time because parsers can be reused for repositories with the same testing infrastructure (e.g., pytest). SWE-smith removes the need for manual efforts to determine installation specifications for multiple versions of a codebase across time, the most costly step of SWE-bench collection. Creating SWE-smith took one author $\sim{}20$ h of human labor.

### 2.2 Features

We apply SWE-smith to $128$ Python repositories, generating a total of $50$ k instances. Table 1 captures the key statistics. On average, we generate $381$ task instances per repository, with as many as $2277$ for pandas-dev/pandas. We summarize the distribution of task instances per repository in Figure 1, where repositories are grouped into one of six general categories. SWE-smith took $ $1360$ to create ($ $1000$ to generate bugs, $ $160$ for automatic repository installation with SWE-agent, $ $200$ to generate issues for $10$ K bugs). Generating an issue costs $2.54$ ¢ on average. More dataset analyses in §C.

Figure 3: Distribution of instances per repo for $128$ repo’s grouped into $6$ categories.

| Bug Type | Yield % | \# Insts | Cost | F2P | Lines |
| --- | --- | --- | --- | --- | --- |
| Combine | $96.9$ % | $10,092$ | $0.00$ ¢ | $15$ | $11$ |
| LM Modify | $56.0$ % | $17,887$ | $0.38$ ¢ | $4$ | $3$ |
| LM Rewrite | $35.0$ % | $4,173$ | $3.93$ ¢ | $4$ | $24$ |
| PR Mirror | $33.8$ % | $2,344$ | $5.53$ ¢ | $3$ | $14$ |
| Procedural | $40.2$ % | $15,641$ | $0.00$ ¢ | $7$ | $5$ |
| Total | 50.1 | 50,137 | 2.32¢ | 6 | 5 |

Table 1: Summary of SWE-smith statistics. “Yield %” is the % of candidates generated by a strategy that break $1+$ tests. “Cost” is the average cost to generate one candidate. “F2P” (Fail to Pass tests), “Lines \[Edited\]” are median values.

Bug generation strategies vary in cost and yield rate. Of methods relying on LMs, PR Mirrors are more expensive because the task entails rewriting entire files, as opposed to individual functions for LM Modify and LM Rewrite. Yield rates are limited by either lack of test coverage for the change or because the bug candidate did not actually introduce relevant issues. For example, for LM Rewrite, the LM is asked to re-implement the function; it is not explicitly asked for bugs. When requested outright (LM Modify), the yield is higher.

| Dataset | \# Tasks | \# Repos | Exec? | Source | Env. Size |
| --- | --- | --- | --- | --- | --- |
| R2E [^15] | $0.25$ k | $137$ |  | Synth | $270$ GBs |
| R2E-gym (Subset) [^16] | $4.6$ k | $10$ |  | Synth | $4$ TBs |
| SWE-bench-extra [^5] | $6.38$ k | $2$ k |  | Real | \- |
| SWE-bench-train [^18] | $19$ k | $37$ |  | Real | \- |
| SWE-fixer [^44] | $115$ k | $856$ |  | Real | \- |
| SWE-gym [^31] | $2.4$ k | $11$ |  | Real | $6$ TBs |
| SWE-smith | $50$ k | $128$ |  | Both | $295$ GBs |

Table 2: Comparison of open source training datasets for software engineering tasks. Relative to existing datasets, SWE-smith has multiple times the number of task instances, repositories, and environments at a fraction of prior storage costs. SWE-fixer and SWE-bench-train task instances do not have execution environments, so “Env. Size” is blank.

How difficult are SWE-smith task instances? To determine whether task instances produced by SWE-smith are realistic and challenging, we train a Qwen $2.5$ $32$ B model on $1{,}699$ human-annotated (task, label) pairs from [^9] to rate tasks as (easy, medium, hard) by training. To quantify difficulty, each difficulty label corresponds to values of $1$ / $5$ / $9$. The model achieves $75.3$ % test accuracy. We then rate difficulty of task instances from both SWE-smith and prior SWE-bench style datasets [^9] [^18] [^31] [^50]. SWE-smith task instances span a broad range of difficulties, similar to SWE-bench and SWE-gym. The average difficulty score for SWE-smith ($5.27$ – $5.72$ across bug generation strategies) is comparable to SWE-bench ($5.01$) and SWE-gym ($5.62$). This suggests SWE-smith enables realistic and appropriately challenging evaluation. We discuss why bug strategies yield different levels of difficulty and visualize difficulty per dataset in §E.

Scaling execution environments. Unlike SWE-bench which creates a Docker image per task instance, SWE-smith leverages a simpler design where tasks from the same repository share the same environment, reducing storage overhead significantly, as shown in Table 2. This approach not only makes scaling task instances more affordable, but also renders SWE-smith more accessible and maintainable than existing datasets. We estimate that creating a similar quantity of task instances ($50$ k) using SWE-bench would require $50$ to $150$ TBs of storage for environments, a $500$ x difference. Extended discussion in §C.1.

## 3 Experiments

To explore the utility of SWE-smith for training software engineering agents, we use rejection sampling fine-tuning [^54] as the primary procedure for improving a base LM with SWE-smith. Our experiment workflow is as follows. First, we curate a subset of SWE-smith task instances. Next, we run an agent system with an expert model on this subset. At this step, the trajectory corresponding to each run is recorded. Then, we fine-tune the base (or “student”) model on the trajectories corresponding to resolved instances. Finally, we evaluate the agent system run with the student model on a separate, test split.

Models. For expert models, we use claude-3-7-sonnet-20250219 [^2]. For fair comparisons with prior works [^31], we also use claude-3-5-sonnet-20240620 and gpt-4o-2024-08-06. We use the Qwen-2.5-Coder-Instruct [^14] $7$ B and $32$ B series as the base models. Training and hyperparameter details are in §F.1.

Agent system. We use SWE-agent [^49], an agent system for solving GitHub issues. SWE-agent provides a base LM with an Agent Computer Interface (ACI) that enables more effective interactions with a codebase. At each turn, SWE-agent prompts an LM to generate a ReAct [^52] style (thought, action) pair, where the action either edits a file or executes a shell command. We choose SWE-agent because, at the time of writing, SWE-agent with Claude 3.7 Sonnet is the top open source solution on SWE-bench. When generating trajectories with expert models, we run SWE-agent for at most $75$ steps and $ $2.00$ cost limit. For inference of student models, we impose the same $75$ step maximum and fix temperature at $0.0$. Full configuration details are in §F.1.

Evaluation metrics. We evaluate on the SWE-bench Lite and Verified [^9] subsets. SWE-bench evaluates AI systems on their ability to solve software issues from $12$ real world GitHub repositories. The Lite split is a subset of $300$ instances, curated to be an easier evaluation set that’s less costly to run. The Verified split is a human-curated subset of $500$ instances, selected for clearer problem statements and more reliable evaluation. To assess generalization beyond Python, we also evaluate on SWE-bench Multilingual, a new dataset introduced in this paper. SWE-Bench Multilingual consists of $300$ task instances that cover $9$ additional programming languages. See §F.2 for more details. We report the % resolved metric, the proportion of successfully resolved instances.

## 4 Results

Table 3 compares the performance of Qwen 2.5 Coder Instruct models (7B and 32B), fine-tuned on $5{,}016$ SWE-smith trajectories. We refer to them as SWE-agent-LM-7B and SWE-agent-LM-32B; the latter achieves state-of-the-art performance.

<table><tbody><tr><th>Model</th><th>System</th><td>Train Size</td><td>Lite</td><td>Verified</td></tr><tr><th colspan="5">Closed Weight Models</th></tr><tr><th>GPT-4o <sup><a href="#fn:27">27</a></sup></th><th>Agentless</th><td>-</td><td><math><semantics><mn>32.0</mn> <annotation>32.0</annotation></semantics></math></td><td><math><semantics><mn>38.8</mn> <annotation>38.8</annotation></semantics></math></td></tr><tr><th></th><th>OpenHands</th><td>-</td><td><math><semantics><mn>22.0</mn> <annotation>22.0</annotation></semantics></math></td><td>-</td></tr><tr><th></th><th>SWE-agent</th><td>-</td><td><math><semantics><mn>18.3</mn> <annotation>18.3</annotation></semantics></math></td><td><math><semantics><mn>23.0</mn> <annotation>23.0</annotation></semantics></math></td></tr><tr><th>Claude 3.5 Sonnet <sup><a href="#fn:1">1</a></sup></th><th>Agentless</th><td>-</td><td><math><semantics><mn>40.7</mn> <annotation>40.7</annotation></semantics></math></td><td><math><semantics><mn>50.8</mn> <annotation>50.8</annotation></semantics></math></td></tr><tr><th></th><th>AutoCodeRover</th><td>-</td><td>-</td><td><math><semantics><mn>46.2</mn> <annotation>46.2</annotation></semantics></math></td></tr><tr><th></th><th>OpenHands</th><td>-</td><td><math><semantics><mn>41.7</mn> <annotation>41.7</annotation></semantics></math></td><td><math><semantics><mn>53.0</mn> <annotation>53.0</annotation></semantics></math></td></tr><tr><th></th><th>SWE-agent</th><td>-</td><td><math><semantics><mn>23.0</mn> <annotation>23.0</annotation></semantics></math></td><td><math><semantics><mn>33.6</mn> <annotation>33.6</annotation></semantics></math></td></tr><tr><th>Claude 3.7 Sonnet <sup><a href="#fn:2">2</a></sup></th><th>SWE-agent</th><td>-</td><td>48.0</td><td>58.2</td></tr><tr><th>Llama3-SWE-RL-70B <sup><a href="#fn:41">41</a></sup></th><th>Agentless</th><td>11M</td><td>-</td><td><math><semantics><mn>41.0</mn> <annotation>41.0</annotation></semantics></math></td></tr><tr><th colspan="5">Open Weight Models</th></tr><tr><th>Lingma-SWE-GPT-72B <sup><a href="#fn:22">22</a></sup></th><th>SWE-SynInfer</th><td>-</td><td>-</td><td><math><semantics><mn>28.8</mn> <annotation>28.8</annotation></semantics></math></td></tr><tr><th>Qwen3-235B-A22B <sup><a href="#fn:33">33</a></sup></th><th>OpenHands</th><td>-</td><td>-</td><td><math><semantics><mn>34.4</mn> <annotation>34.4</annotation></semantics></math></td></tr><tr><th>R2E-Gym-32B <sup><a href="#fn:16">16</a></sup></th><th>OpenHands</th><td><math><semantics><mn>3.3</mn> <annotation>3.3</annotation></semantics></math> k</td><td>-</td><td><math><semantics><mn>34.4</mn> <annotation>34.4</annotation></semantics></math></td></tr><tr><th>SWE-fixer-72B <sup><a href="#fn:44">44</a></sup></th><th>SWE-Fixer</th><td><math><semantics><mn>110</mn> <annotation>110</annotation></semantics></math> k</td><td><math><semantics><mn>24.7</mn> <annotation>24.7</annotation></semantics></math></td><td><math><semantics><mn>32.8</mn> <annotation>32.8</annotation></semantics></math></td></tr><tr><th>SWE-gym-32B <sup><a href="#fn:31">31</a></sup></th><th>OpenHands</th><td><math><semantics><mn>491</mn> <annotation>491</annotation></semantics></math></td><td><math><semantics><mn>15.3</mn> <annotation>15.3</annotation></semantics></math></td><td><math><semantics><mn>20.6</mn> <annotation>20.6</annotation></semantics></math></td></tr><tr><th>SWE-agent-LM-7B</th><th>SWE-agent</th><td><math><semantics><mn>2</mn> <annotation>2</annotation></semantics></math> k</td><td><math><semantics><mn>11.7</mn> <annotation>11.7</annotation></semantics></math></td><td><math><semantics><mn>15.2</mn> <annotation>15.2</annotation></semantics></math></td></tr><tr><th>SWE-agent-LM-32B</th><th>SWE-agent</th><td><math><semantics><mn>5</mn> <annotation>5</annotation></semantics></math> k</td><td>30.7</td><td>40.2</td></tr></tbody></table>

Table 3: Resolve rates for existing solutions on SWE-bench Lite and Verified, collected from [^17], compared to models fine-tuned on SWE-smith. All performance numbers are pass@ $1$. We do not compare against systems that use verifiers or multiple attempts at test time.

The final dataset of $5{,}016$ training points was curated as follows. We start by collecting a large pool of expert trajectories. First, we carried out each of the ablations in Section 4.1, giving us an initial set of $5{,}105$ trajectories. Next, based on our observation that PR Mirror and LM Rewrite task instances yield the most effective expert trajectories (discussed below), we run the expert model on all task instances of these types, bumping up the total number to $6{,}457$ task instances. Ultimately, we attempt to generate expert trajectories for $8{,}686$ unique task instances, or $17.3$ % of the SWE-smith dataset. Reinforcing the difficulty rating findings from Section 2.2, we observe that SWE-smith task instances are non-trivial for the top agent systems today. The final pool of $6{,}457$ represents a $36$ % resolve rate of all $17{,}906$ attempts to solve one of the $8{,}686$ task instances.

Next, we perform minor filtering of this collection. As reported in [^31], we also observe that “easier” trajectories – task instances that are repeatedly solved across multiple runs — degrade model performance. Therefore, we limit the number of times any SWE-smith task instance is represented in the training set to $3$ trajectories. This leads to the final $5{,}016$ training set. More details in §F.3.

Performance improves with more data points. Extending similar graphs from [^16] [^31], Figure 1 shows increasing performance with more trajectories.

Comparison at the same training set size. To compare with prior works [^16] [^31], we run expert trajectory generation on $1000$ random SWE-smith task instances with SWE-agent + Claude 3.5 Sonnet ($800$) or GPT-4o ($200$). We then fine-tune the $32$ B model on $500$ successful trajectories, a training set size both works report on. Our model achieves a $28.2$ % resolve rate on SWE-bench Verified, a relative difference of $+8.2$ % with [^31] and $+0.7$ % with [^16].

### 4.1 Ablations of SWE-smith

We perform several ablations of how SWE-smith’s bug and problem statement generation strategies impact the quality of training data. We use Claude 3.7 Sonnet as the expert for fine-tuning Qwen $2.5$ $7$ B Coder Instruct, and report the performance on SWE-bench Verified.

LM Rewrite and Procedural bugs are comparable to PR mirrors. We randomly sample $1000$ instances per bug generation strategy (LM Modify, LM Rewrite, Procedural Modifications, PR Mirrors). Per instance, we generate issue text with an LM and run expert trajectory generation. We then fine-tune a student model per strategy, capping training points to the minimum number of successful trajectories from any strategy ($507$) for fair comparison.

Table 5 summarizes the results. Trajectories generated from PR mirrors are empirically the most effective training data — this is expected, since they are most reflective of SWE-bench. What’s noteworthy is that trajectories from Procedural Modification and LM Rewrite instances lead to competitive models. There is a steep drop-off with LM Modify bugs.

LM generated issues are comparable to real issues. We randomly sample $600$ PR Mirror task instances. We compare LM generated issues with three alternatives — fixed issue templates, the source code + test logs of a random Fail-to-Pass test, and the original issue text associated with the PR. We again cap training points to the minimum number of successful trajectories ($259$) for fairness.

As shown in Table 5, training on task instances with LM generated issues is empirically comparable to using the original issue text. Using fixed issue templates not only leads to the fewest successful trajectories, but also results in relatively homogeneous problem solving sequences. The expert trajectories from fixed issue templates have $31$ % fewer unique actions compared to LM generated text ($379$ vs. $550$). While providing a Fail-to-Pass test case leads to more successful expert trajectories, leaking the evaluation criteria causes the model to skip over writing a reproduction script, which accounts for the performance drop. Of $500$ SWE-bench Verified instances, the student model trained on LM-generated issues attempts to reproduce the bug for $379$ of the runs. The model trained on test-based issues only does so for $127$ cases, a $66$ % decrease.

| Strategy | \# Trajs. | % Resolved |
| --- | --- | --- |
| LM Modify | $802$ | $5.7$ ($\pm 1.5$) |
| LM Rewrite | $507$ | $8.8$ ($\pm 1.7$) |
| Procedural | $745$ | $8.6$ ($\pm 1.8$) |
| PR Mirror | $557$ | $9.2$ ($\pm 1.7)$ |

Table 4: Comparison of training on $1000$ SWE-smith instances created with different strategies.

| Issue | \# Trajs. | % Resolved |
| --- | --- | --- |
| Fixed | 259 | $6.4$ ($\pm 1.5$) |
| F2P Test | 390 | $7.3$ ($\pm 1.9$) |
| LM | 328 | $7.7$ ($\pm 1.5$) |
| Original | 319 | $7.8$ ($\pm 1.8$) |

Table 5: Comparing training on $600$ PR Mirror instances with varied issue text.

Task difficulty correlates with solvability but not with effectiveness as training data. First, we run our difficulty rating model on $10$ k randomly selected SWE-smith task instances. From this pool, we curate subsets of $1000$ instances corresponding to the three difficulty levels, then run expert trajectory generation per subset $3$ times. For the easy/medium/hard subsets, the resolve rate by the expert model are $58.6$ %, $41.0$ %, and $17.0$ % respectively.

Next, from all successful trajectories, we create four fine-tuning datasets of $500$ trajectories each corresponding to difficulty scores of $2$, $4$, $6$, and $8$. As mentioned in Section 2.2, the corresponding scores for easy/medium/hard are $1$ / $5$ / $9$. Therefore, the SFT dataset for score $2$ is made up of trajectories corresponding to $375$ easy and $125$ medium instances, and so on. Somewhat surprisingly, we do not observe strong correlation between increased difficulty and downstream performance. For the student models trained on the $2$ / $4$ / $6$ / $8$ difficulty SFT datasets, we get pass@1 scores of $12.4$ %, $10.8$ %, $13.6$ %, and $12.2$ % on SWE-bench Verified.

Figure 4: We fine-tune a $7$ B base and our $32$ B models on $700$ trajectories for SymPy. Specialization boosts performance with minor generalization loss.

Figure 5: At $700$ training samples, we observe performance increases logarithmically with repositories.

Training on more repositories improves general performance. We train models in four settings by sampling $700$ expert trajectories on Procedural Modification tasks from pools of $4$, $25$, $50$, and $100$ repositories. Echoing similar findings for code generation tasks [^46], we find that increasing repositories represented in the training set improves performance, as shown in Figure 5, with an approximately logarithmic relation between model performance and number of repositories.

Repository-specialized models excel on the target repository with minor generalization loss. We experiment with training models to be specialists on one particular repository. To assess performance, we evaluate models on a subset of SWE-bench Verified tasks that are (1) from SymPy, and (2) created after January 1st, 2022, a total of $22$ instances. To create SymPy specific training data, we first select a base commit of SymPy just before the cutoff date. Next, we create $1276$ Procedural Modification task instances, then generate $700$ expert trajectories. We evaluate specialization in two settings: (1) single-repository fine-tuning, and (2) specialist stage fine-tuning, both shown in Figure 5. For single-repository tuning, we compare a model initialized with Qwen-2.5-Coder-Instruct 7B and trained on $700$ instances sampled from $100$ repositories, to the same Qwen base model but fine-tuned on the $700$ SymPy instances only. For specialist stage fine-tuning, we simply compare SWE-agent-LM-32B to the same model further fine-tuned on the $700$ SymPy instances.

Specialization significantly boosts performance for the target repository with only slight drops in general performance in both the single-repository fine-tuning ($21.2$ % vs. $13.6$ %) and specialist stage fine-tuning ($42.4$ % vs. $33.3$ %) settings.

### 4.2 Analysis of Agent Behavior

Figure 6: SWE-agent-LM-32B takes fewer steps to submit compared to Claude 3.7 Sonnet for instances resolved by both models.

Figure 7: For unsuccessfully resolved tasks, a frequent failure mode is that SWE-agent-LM-32B will repeat actions.

This section analyzes the behavior, failure modes, and efficiency of SWE-agent when run with SWE-agent-LM-32B or Claude 3.7 Sonnet on SWE-bench verified.

SWE-agent-LM-32B can solve tasks efficiently. SWE-agent-LM-32B resolves tasks in fewer steps on average (24.9) than Claude 3.7 Sonnet (29.1), though the difference becomes marginal when accounting for different average difficulties of the resolved tasks: On the overlap of tasks that are resolved by both LMs, SWE-agent-LM-32B uses 24.8 steps compared to 25.6 used by Claude 3.7 Sonnet (see Fig. 7). While shorter trajectories are not always preferred (additional actions can be used for additional validation purposes, for example), this shows that SWE-agent-LM-32B solves tasks very efficiently. At the same time SWE-agent-LM-32B also demonstrates that it can remain focused throughout long trajectories, with 31 instances being resolved after 40 steps or more. We further highlight that the accuracy of naturally terminating <sup>1</sup> agent submissions with SWE-agent-LM-32B achieve an accuracy nearly matching that of Claude 3.7 Sonnet (60% vs 63%), showing that SWE-agent-LM-32B is adept at determining whether an instance has been resolved. As the overall cost and turn count averages scale strongly with the cost and turn limits, we reserve a more thorough analysis for §F.5.1.

Repetitive actions are a key problem. We observe a tendency for SWE-agent-LM-32B to get stuck in long sequences of repetitive actions, in particular long sequences of calls that display different portions of a file instead of using search commands. <sup>2</sup> More than 25% of SWE-agent-LM-32B trajectories have a repetitive sequence of at least length 10, compared to less than 4% for Claude 3.7 Sonnet (see Figure 7). The occurrence of long repetitive sequences correlates strongly with the agent’s ability to solve the corresponding task instance, largely because the LM continues issuing similar commands until either the agent cost or turn limit is reached, at which point the run is terminated. For example, repetitive sequences of length 10 correspond to an 89% failure probability. Simple interventions from the agent scaffold can mitigate repetitive actions, but do not seem to improve resolve rates (see §F.5).

Figure 8: More than half of the unresolved instances of SWE-agent-LM-32B correspond to runs terminated by cost/step limits, and these limits are frequently reached before source code has been modified. See §F.5 for more.

Localization is the dominant failure mode. Guided by a short plan in the system prompt, SWE-agent typically starts by *localizing* (search and read actions), *reproducing* (test file creation and execution), before modifying source files and validating the fixes. If the agent gets stuck at any of these stages or keeps on iterating, the agent loop is eventually interrupted by runtime limits (cost, number of LM calls, runtime). While this rarely happens with Claude 3.7 Sonnet, 53% of SWE-agent-LM-32b’s failures are associated with such limits (Figure 8). The agent often already gets stuck during localization or initial efforts to reproduce a bug, with endlessly repeated actions being a persistent issue. More on failure modes in §F.5.

## 5 Related Work

LMs for Software Engineering. As contemporary LMs have saturated traditional code generation tasks [^4] [^8], software engineering benchmarks [^15] [^18] [^50] [^58] [^55], notably SWE-bench, have become a new de facto evaluation setting due to their diverse, complex, real-world programming challenges. The most significant source of open source progress on SWE-bench has been the development of LM-based workflows [^29] [^42] [^57] and agents [^3] [^38] [^49] [^56]. Workflow-based systems are typically human-engineered decompositions of a task into a sequence of sub-goals. [^50] suggests such pipelines may not generalize effectively to non-Python repositories, requiring additional human intervention to re-adapt. We therefore elect to focus on generating trajectories with and for LM agent systems [^36] [^48] [^52]. Because no workflow is imposed, agent systems inherently rely more on the LM to plan and refine its actions, putting more focus on an LM’s capabilities, not inference scaffolds.

Training Datasets for Coding. Prior work around training data has focused on instruction following [^21] [^24] [^35] [^39] [^40] [^53] and preference learning [^19] [^20] for code completion tasks. Several recent works introduce training sets for retrieval augmented generation [^18] [^44], workflows [^41], and agent [^5] [^22] [^31] [^16] approaches to SWE-bench. Our work applies [^12] at a repository level: by having an LM break a codebase, we drastically reduce the human effort needed to define a task and build its environment. Concurrent to our work, [^46] (RePOST) also constructs execution environments for repository functions, but differs significantly in methodology and evaluation. RePOST sandboxes a function and its dependencies to a separate script, then generates tests with an LM, removing the original codebase as context. The tasks’ source is repository-level; the environments and tasks are not. RePOST evaluates solely on code generation (e.g., HumanEval [^8]). [^16] (R2E-Gym) improves open source LMs’ performance on SWE-bench with inference time scaling and verifiers. R2E-gym’s $51$ % resolve rate is not comparable to Table 3 results, as each instance is attempted $26$ times. R2E-gym’s $4.6$ k training instances are collected using SWE-bench’s pipeline, with some augmentations around using LMs to synthesize issue text and tests. To our knowledge, we are the first to address the limited scalability of previous approaches.

## 6 Discussion

Limitations and future directions. First, SWE-smith’s collection pipeline is Python-centric. The mechanisms to identify programmatic objects (e.g. functions, classes) and perform transformations rely heavily on the Python specific ast library. That said, SWE-smith’s collection strategy is transferable to other languages. Second, due to both compute/budget constraints and our work’s primary stance as a dataset contribution, we only include fine-tuning as a demonstration of SWE-smith’s effectiveness. We do not explore other training techniques such as reasoning capabilities elicited via reinforcement learning.

Conclusion. We introduce SWE-smith, a dataset of $50$ k software engineering task instances from across $128$ real world GitHub repositories. SWE-smith collection pipeline allows us to scale up task instances, environments, and trajectories at a fraction of prior costs without sacrificing faithfulness to open source software development practices. Using SWE-smith, we train SWE-agent-LM-32B, achieving a state-of-the-art $40.2$ % on SWE-bench Verified. Our experiments show how SWE-smith can be used to identify fundamental trends about developing SWE-agents. We believe SWE-smith provides the foundational data and infrastructure needed to train software engineering agents in a truly scalable manner.

## Acknowledgments

We thank Princeton Language & Intelligence (PLI) for providing credits for running closed-source API models. Thanks to Samuel Ainsworth for his constant support of bitbop.io ([https://bitbop.io/](https://bitbop.io/)), the compute service for which the majority of the project was carried out with. We’d also like to thank Akshat Bubna, Howard Halim, Andrew Liu, Peyton Walters, and the great team at Modal ([https://modal.com/](https://modal.com/)) for providing credits that made fine-tuning and model serving efforts extremely easy for this project. This work is partially supported by ONR grant N000142412532 and NSF grant IIS-2247357. We also thank Open Philanthropy and Andreessen Horowitz for providing funding for this work. Finally, thanks to Tianyu Gao, William Held, Niklas Muennighoff, Rafael Rafailov, Yijia Shao, Chenglei Si, Anikait Singh, Tianyi Zhang, Kexin Pei, and Karthik Narasimhan for constructive discussions and support throughout this project.

## References

## Appendix

The appendix is generally structured as follows. In Sections A to D, we review details about SWE-smith’s infrastructure and collection strategies for curating the SWE-smith task instances and execution environments, providing comparisons to existing datasets such as SWE-bench and SWE-gym along the way. In Sections E and onward, we discuss more about how we created the trajectories dataset, then provide additional ablations and results showcasing the effectiveness of SWE-smith as a dataset.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2504.21798/assets/overview.png)

Figure 9: An overview of pipelines in SWE-smith. Scripts/functions and manual steps are highlighted in blue. Artifacts that are also the inputs and outputs of these scripts are in orange. SWE-smith fits in seamlessly with the SWE-bench and SWE-agent ecosystem. Use SWE-smith to construct execution environments and generate task instances. Use SWE-agent to generate expert trajectories on SWE-smith task instances and run inference with models trained on these trajectories. Use SWE-bench to evaluate how good your models are at resolving GitHub issues and performing software engineering tasks.

## Appendix A Infrastructure

We cover additional details about how SWE-smith works, specifically

- The form factor of a SWE-smith task instance.
- How we identify repositories and the SWE-agent configuration we use to automatically install them.
- How the task validation and evaluation harnesses work.

### A.1 SWE-smith Task Instance

We briefly review the format of a SWE-smith task instance, highlight how it is different from a SWE-bench task instance, and discuss why SWE-smith’s relatively simple infrastructure compared to SWE-bench allows us scale task collection much more efficiently.

A SWE-smith task instance is very similar to the form factor of a SWE-bench task instance, with several minor differences. A SWE-smith task instance includes the following fields:

- repo: The repository the task instance is from.
- instance\_id: A unique identifier (usually (repo).(bug\_type).(hash))
- base\_commit: Hash of the GitHub branch that points to the repository with the bug patch applied.
- patch: The diff that causes the bug. It is applied to the original codebase to create the bug. Reverting this patch is effectively the solution.
- problem\_statement: The generated issue text that conveys the bug. It is provided to a model or system before it begins attempting a fix.
- created\_at: A timestamp matching when the bug was successfully validated and pushed to the mirror repository as a branch.
- FAIL\_TO\_PASS: The unit tests that break when the test suite is run with the bug patch applied.
- PASS\_TO\_PASS: The unit tests that do not break. These correspond to the set of all tests minus the FAIL\_TO\_PASS tests.

We summarize the key distinctions between a SWE-smith and SWE-bench task instance:

- SWE-smith task instances do not include the version or environment\_setup\_commit fields, which SWE-bench requires as additional identifiers for specifying repository-specific installation instructions across time. In SWE-smith, unique installation instructions are specified for each (repository, commit).
- The hints\_text field is not included. In SWE-bench, this refers to the issue and PR thread comments written after the first commit of the corresponding PR.
- The created\_at field is assigned the timestamp reflecting when the bug was successfully validated. Originally, created\_at refers to when a PR was created.
- There is no test\_patch field, as the SWE-smith collection pipeline does not create or synthesize any hidden tests. All FAIL\_TO\_PASS bugs are visible and runnable in the repository at inference time.

### A.2 Repository Selection

In addition to the criteria discussed in Section 2.1, we also ensure that a repository has a license that allows non-proprietary use. The majority of software licenses are permissive (BSD, MIT, Apache), while the remainder are largely protective licenses (GPL) that still allow for non-commercial use. We inspected the repositories with custom licenses and confirmed they allowed for the use cases exercised in our work. The licenses for each repository are fully listed in Table 6.

| Apache License 2.0 | Project-MONAI/MONAI; alanjds/drf-nested-routers; arrow-py/arrow; buriy/python-readability; facebookresearch/fvcore; getmoto/moto; google/textfsm; iterative/dvc; jax-ml/jax; jd/tenacity; kayak/pypika; modin-project/modin; pyca/pyopenssl; spulec/freezegun; tkrajina/gpxpy; tornadoweb/tornado; weaveworks/grafanalib |
| --- | --- |
| BSD 2-Clause ”Simplified” License | madzak/python-json-logger; pyasn1/pyasn1; pygments/pygments; sunpy/sunpy |
| BSD 3-Clause ”New” or ”Revised” License | Suor/funcy; alecthomas/voluptuous; andialbrecht/sqlparse; cookiecutter/cookiecutter; dask/dask; django/channels; django/daphne; encode/starlette; gawel/pyquery; gweis/isodate; john-kurkowski/tldextract; lepture/mistune; oauthlib/oauthlib; pallets/click; pallets/flask; pallets/jinja; pallets/markupsafe; pandas-dev/pandas; scrapy/scrapy; theskumar/python-dotenv |
| GNU General Public License v3.0 | Cog-Creators/Red-DiscordBot; adrienverge/yamllint |
| GNU Lesser General Public License v2.1 | chardet/chardet; paramiko/paramiko; pylint-dev/astroid |
| GNU Lesser General Public License v3.0 | Knio/dominate |
| ISC License | kennethreitz/records |
| MIT License | amueller/word\_cloud; borntyping/python-colorlog; bottlepy/bottle; cantools/cantools; cdgriffith/Box; cknd/stackprinter; conan-io/conan; cool-RR/PySnooper; datamade/usaddress; dbader/schedule; erikrose/parsimonious; facebookresearch/hydra; facelessuser/soupsieve; getnikola/nikola; graphql-python/graphene; hukkin/tomli; jaraco/inflect; jawah/charset\_normalizer; joke2k/faker; keleshev/schema; life4/textdistance; luozhouyang/python-string-similarity; marshmallow-code/apispec; marshmallow-code/marshmallow; marshmallow-code/webargs; martinblech/xmltodict; matthewwithanm/python-markdownify; mewwts/addict; mido/mido; mozillazg/python-pinyin; msiemens/tinydb; pdfminer/pdfminer; pndurette/gTTS; pudo/dataset; pydantic/pydantic; pyparsing/pyparsing; pytest-dev/iniconfig; python-hyper/h11; python-jsonschema/jsonschema; python-openxml/python-docx; pyupio/safety; pyvista/pyvista; r1chardj0n3s/parse; rsalmei/alive-progress; rubik/radon; rustedpy/result; scanny/python-pptx; seatgeek/thefuzz; sloria/environs; sqlfluff/sqlfluff; termcolor/termcolor; tobymao/sqlglot; tox-dev/pipdeptree; tweepy/tweepy; un33k/python-slugify; vi3k6i5/flashtext |
| Other | Mimino666/langdetect; PyCQA/flake8; agronholm/exceptiongroup; agronholm/typeguard; aio-libs/async-timeout; benoitc/gunicorn; cloudpipe/cloudpickle; davidhalter/parso; django-money/django-money; gruns/furl; kurtmckee/feedparser; lincolnloop/python-qrcode; mahmoud/boltons; mahmoud/glom; mozilla/bleach; pexpect/ptyprocess; prettytable/prettytable; pwaller/pyfiglet; pydata/patsy; pydicom/pydicom; python-trio/trio; python/mypy; pyutils/line\_profiler; seperman/deepdiff |

Table 6: License associated with each repository as of April 8, 2025. All licenses are permissive and allow for public, nonprofit use.

We deliberately limit the search scope for repositories to those predominantly written in Python. Following precedents, focusing on Python repositories allowed us to form assumptions about installation and testing procedures (e.g. repository is organized as a PyPI package, pytest is the testing framework) that made scaling up automatic repository setup with SWE-agent more tractable. A worthwhile direction to consider for future work is expanding the coverage of repositories to be more comprehensive of codebases written in different programming languages, as [^50] does, extending SWE-bench style evaluation to JavaScript repositories with multimodal inputs.

Automated repository installation. The goal of this step is to first, get the installation and testing instructions for a repository, and second, create a Docker image containing the repository with the development environment set up.

We provide the system prompt given to SWE-agent that asks it to install a repository in Figure A.2. Each repository installation task is initialized with a clone of the original repository. No additional steps (e.g. pypi package downloads, conda environment setup) are performed.

We run SWE-agent with claude-3-5-sonnet-20241022 with a maximum cost limit of $ $2$ and a maximum call limit of $150$. The installation run terminates whenever one of these conditions is met. For every run, we record the interactions. We then manually review the trajectory, identifying the appropriate installation and testing specifications.

Each run incurs an average cost of $ $0.72$ and an average of $17$ steps before SWE-agent issues the submit command. The runs typically finish within two minutes. The majority of Python repositories require fewer steps — typically, SWE-agent will view the CONTRIBUTING.md, run the installation command provided verbatim in the text, and then runs pytest, showing all tests passing. A minority of repositories will require several steps because additional dependencies must be installed with apt-get. The manual review process following this requires $3$ to $20$ minutes. One author carried out this effort for $128$ repositories, taking an estimated $18$ human hours to accomplish. In the process of reaching $128$ repositories, the author gave up on $17$ repositories at the manual review stage.

<svg id="A1.SS2.p7.pic1" height="841.25" overflow="visible" version="1.1" viewBox="0 0 477.38 841.25" width="477.38"><g style="--ltx-stroke-color:#000000;--ltx-fill-color:#000000;" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,841.25) matrix(1 0 0 -1 0 0)"><g style="--ltx-fill-color:#1F77B4;" fill="#1F77B4" fill-opacity="1.0"><path style="stroke:none" d="M 0 5.91 L 0 835.34 C 0 838.6 2.64 841.25 5.91 841.25 L 471.47 841.25 C 474.73 841.25 477.38 838.6 477.38 835.34 L 477.38 5.91 C 477.38 2.64 474.73 0 471.47 0 L 5.91 0 C 2.64 0 0 2.64 0 5.91 Z"></path></g><g style="--ltx-fill-color:#F4F8FB;" fill="#F4F8FB" fill-opacity="1.0"><path style="stroke:none" d="M 1.97 5.91 L 1.97 817.14 L 475.41 817.14 L 475.41 5.91 C 475.41 3.73 473.65 1.97 471.47 1.97 L 5.91 1.97 C 3.73 1.97 1.97 3.73 1.97 5.91 Z"></path></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 825.73)"><foreignObject style="--ltx-fo-width:31.37em;--ltx-fo-height:0.69em;--ltx-fo-depth:0.19em;font-size:10pt;" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 9.61)" width="434.07"><span id="A1.SS2.p7.pic1.1" style="width:31.37em;"><span id="A1.SS2.p7.pic1.1.1"><span id="A1.SS2.p7.pic1.1.1.1" style="--ltx-fg-color:#FFFFFF;">System prompt for generating bugs with an LM</span></span> </span></foreignObject></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 16.2)"><foreignObject style="--ltx-fo-width:33.91em;--ltx-fo-height:57.03em;--ltx-fo-depth:0.18em;font-size:10pt;" height="791.56" overflow="visible" transform="matrix(1 0 0 -1 0 789.13)" width="469.22"><span id="A1.SS2.p7.pic1.2" style="width:33.91em;"><span id="A1.SS2.p7.pic1.2.1"><math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&lt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000" mathsize="0.900em">&lt;</mo> <annotation encoding="application/x-tex">&lt;</annotation></semantics></math> <span id="A1.SS2.p7.pic1.2.1.1" style="font-size:90%;--ltx-fg-color:#000000;">uploaded_files <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&gt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&gt;</mo> <annotation encoding="application/x-tex">&gt;</annotation></semantics></math> {{working_dir}}<span id="A1.SS2.p7.pic1.2.1.1.2"><br><math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&lt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&lt;</mo> <annotation encoding="application/x-tex">&lt;</annotation></semantics></math></span> /uploaded_files <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&gt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&gt;</mo> <annotation encoding="application/x-tex">&gt;</annotation></semantics></math><span id="A1.SS2.p7.pic1.2.1.1.3"><br>I’ve uploaded a python code repository in the directory</span> {{working_dir}}<span id="A1.SS2.p7.pic1.2.1.1.4">.<br></span></span></span><span id="A1.SS2.p7.pic1.2.2"><span id="A1.SS2.p7.pic1.2.2.1" style="font-size:90%;--ltx-fg-color:#000000;">Can you please install this repository? Your goal should be to configure the repository’s development environment such that existing tests pass. You are currently in the root directory of the repository, and nothing has been installed yet. You in an Ubuntu 22.04 environment.<br></span></span><span id="A1.SS2.p7.pic1.2.3"><span id="A1.SS2.p7.pic1.2.3.1" style="font-size:90%;--ltx-fg-color:#000000;">The repository is predominantly written in Python. Here are several tips for installing it:<br></span></span><span id="A1.SS2.p7.pic1.2.4"><span id="A1.SS2.p7.pic1.2.4.1" style="font-size:90%;--ltx-fg-color:#000000;">1. A good place to start is to look for a <span id="A1.SS2.p7.pic1.2.4.1.1">CONTRIBUTING.[md <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="|"><semantics><mo style="--ltx-fg-color:#000000;" fence="false" mathcolor="#000000" stretchy="false">|</mo> <annotation encoding="application/x-tex">|</annotation></semantics></math> rst]</span> file, which will often contain instructions on how to install the repository and any dependencies it may have. Occasionally, the <span id="A1.SS2.p7.pic1.2.4.1.2">README.md</span> file may also contain installation instructions.<br></span></span><span id="A1.SS2.p7.pic1.2.5"><span id="A1.SS2.p7.pic1.2.5.1" style="font-size:90%;--ltx-fg-color:#000000;">2. Usually, a repository may have <span id="A1.SS2.p7.pic1.2.5.1.1">setup.py</span> or <span id="A1.SS2.p7.pic1.2.5.1.2">pyproject.toml</span> files which can be used to install the package. <span id="A1.SS2.p7.pic1.2.5.1.3">pip install -e.</span> is commonly used, although many packages will also require an additional specifier that installs development packages as well (e.g. <span id="A1.SS2.p7.pic1.2.5.1.4">pip install -e.[dev]</span>).<br></span></span><span id="A1.SS2.p7.pic1.2.6"><span id="A1.SS2.p7.pic1.2.6.1" style="font-size:90%;--ltx-fg-color:#000000;">3. To check whether the repository was installed successfully, run tests and see if they pass. You can usually find tests in a <span id="A1.SS2.p7.pic1.2.6.1.1">tests/</span> or <span id="A1.SS2.p7.pic1.2.6.1.2">test/</span> directory. You can run tests using <span id="A1.SS2.p7.pic1.2.6.1.3">pytest</span> or <span id="A1.SS2.p7.pic1.2.6.1.4">unittest</span>, depending on the framework used by the repository.<br></span></span><span id="A1.SS2.p7.pic1.2.7"><span id="A1.SS2.p7.pic1.2.7.1" style="font-size:90%;--ltx-fg-color:#000000;">4. Sometimes, you will need to install additional packages, often listed in a <span id="A1.SS2.p7.pic1.2.7.1.1">requirements.txt</span> or <span id="A1.SS2.p7.pic1.2.7.1.2">environment.yml</span> file. Also, be mindful of Ubuntu system dependencies that may need to be installed via <span id="A1.SS2.p7.pic1.2.7.1.3">apt-get</span> (e.g. <span id="A1.SS2.p7.pic1.2.7.1.4">sudo apt-get install <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&lt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&lt;</mo> <annotation encoding="application/x-tex">&lt;</annotation></semantics></math> package <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&gt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&gt;</mo> <annotation encoding="application/x-tex">&gt;</annotation></semantics></math></span>).<br></span></span><span id="A1.SS2.p7.pic1.2.8"><span id="A1.SS2.p7.pic1.2.8.1" style="font-size:90%;--ltx-fg-color:#000000;">Once you are finished with installing the repository, run the <span id="A1.SS2.p7.pic1.2.8.1.1">submit</span> command to submit your changes for review.</span></span></span></foreignObject></g></g></svg>

Figure 10: Prompt provided to SWE-agent + an LM asking it to install a repository.

### A.3 Validation, Evaluation Harnesses

We adapt SWE-bench’s validation script to convert each bug patch into a SWE-bench style task instance. This step ensures SWE-smith can be run by existing SWE-bench solutions. The conversion involves two steps. First, the bug patch is applied and pushed as a branch to a mirror clone of the repository. Second, we create a SWE-bench style task instance from the bug patch, populating important fields such as Fail-to-Pass and Pass-to-Pass tests with information from the validation logs.

## Appendix B Bug Generation Strategies

In this section, we review each of the bug generation strategies we employ in depth. While we experimented with several bug generation strategies, the ones we elect to include are those we found to satisfy several desirable properties.

1. The approach works in a codebase-agnostic manner.
2. The approach reliably yields usable task instances (meaning $1+$ passing tests break).
3. The approach is controllable; via each strategy’s parameters, we can affect the quantity and quality of the generated bugs.

<svg id="A2.p3.pic1" height="768.88" overflow="visible" version="1.1" viewBox="0 0 477.38 768.88" width="477.38"><g style="--ltx-stroke-color:#000000;--ltx-fill-color:#000000;" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,768.88) matrix(1 0 0 -1 0 0)"><g style="--ltx-fill-color:#1F77B4;" fill="#1F77B4" fill-opacity="1.0"><path style="stroke:none" d="M 0 5.91 L 0 762.97 C 0 766.24 2.64 768.88 5.91 768.88 L 471.47 768.88 C 474.73 768.88 477.38 766.24 477.38 762.97 L 477.38 5.91 C 477.38 2.64 474.73 0 471.47 0 L 5.91 0 C 2.64 0 0 2.64 0 5.91 Z"></path></g><g style="--ltx-fill-color:#F4F8FB;" fill="#F4F8FB" fill-opacity="1.0"><path style="stroke:none" d="M 1.97 5.91 L 1.97 744.77 L 475.41 744.77 L 475.41 5.91 C 475.41 3.73 473.65 1.97 471.47 1.97 L 5.91 1.97 C 3.73 1.97 1.97 3.73 1.97 5.91 Z"></path></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 753.36)"><foreignObject style="--ltx-fo-width:31.37em;--ltx-fo-height:0.69em;--ltx-fo-depth:0.19em;font-size:10pt;" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 9.61)" width="434.07"><span id="A2.p3.pic1.1" style="width:31.37em;"><span id="A2.p3.pic1.1.1"><span id="A2.p3.pic1.1.1.1" style="--ltx-fg-color:#FFFFFF;">System prompt for generating bugs with an LM</span></span> </span></foreignObject></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignObject style="--ltx-fo-width:33.91em;--ltx-fo-height:51.98em;--ltx-fo-depth:0em;font-size:10pt;" height="719.19" overflow="visible" transform="matrix(1 0 0 -1 0 719.19)" width="469.22"><span id="A2.p3.pic1.2" style="width:33.91em;"><span id="A2.p3.pic1.2.1"><span id="A2.p3.pic1.2.1.1" style="font-size:90%;--ltx-fg-color:#000000;">You are a software developer doing chaos monkey testing. Your job is to rewrite a function such that it introduces a logical bug that will break existing unit test(s) in a codebase.</span></span> <span id="A2.p3.pic1.2.2"><span id="A2.p3.pic1.2.2.1" style="font-size:90%;--ltx-fg-color:#000000;">To this end, some kinds of bugs you might introduce include:<br></span></span><span id="A2.p3.pic1.2.3"><span id="A2.p3.pic1.2.3.1" style="font-size:90%;--ltx-fg-color:#FF0000;">(Per inference call, only 3 of the following tips are randomly selected and shown)<span id="A2.p3.pic1.2.3.1.1" style="--ltx-fg-color:#000000;"><br>- Alter calculation order for incorrect results: Rearrange the sequence of operations in a calculation to subtly change the output (e.g., change (a + b) * c to a + (b * c)).<br>- Introduce subtle data transformation errors: Modify data processing logic, such as flipping a sign, truncating a value, or applying the wrong transformation function.<br>- Change variable assignments to alter computation state: Assign a wrong or outdated value to a variable that affects subsequent logic.<br>- Mishandle edge cases for specific inputs: Change handling logic to ignore or improperly handle boundary cases, like an empty array or a null input.<br>- Modify logic in conditionals or loops: Adjust conditions or loop boundaries (e.g., replace <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&lt;="><semantics><mrow><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&lt;</mo> <mo lspace="0em">⁣</mo> <mo style="--ltx-fg-color:#000000;" mathcolor="#000000">=</mo></mrow> <annotation encoding="application/x-tex">&lt;=</annotation></semantics></math> with <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&lt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&lt;</mo> <annotation encoding="application/x-tex">&lt;</annotation></semantics></math>) to change the control flow.<br>- Introduce off-by-one errors in indices or loop boundaries: Shift an index or iteration boundary by one, such as starting a loop at 1 instead of 0.<br>- Adjust default values or constants to affect behavior: Change a hardcoded value or default parameter that alters how the function behaves under normal use.<br>- Reorder operations while maintaining syntax: Rearrange steps in a process so the function produces incorrect intermediate results without breaking the code.<br>- Swallow exceptions or return defaults silently: Introduce logic that catches an error but doesn’t log or handle it properly, leading to silent failures.<br></span></span></span><span id="A2.p3.pic1.2.4"><span id="A2.p3.pic1.2.4.1" style="font-size:90%;--ltx-fg-color:#000000;">Tips about the bug-introducing task:<br><span id="A2.p3.pic1.2.4.1.1" style="--ltx-fg-color:#FF0000;">(At inference time, tips are randomly shuffled)</span><br>- It should not cause compilation errors.<br>- It should not be a syntax error.<br>- It should be subtle and challenging to detect.<br>- It should not modify the function signature.<br>- It should not modify the documentation significantly.<br>- For longer functions, if there is an opportunity to introduce multiple bugs, please do!” - Please DO NOT INCLUDE COMMENTS IN THE CODE indicating the bug location or the bug itself.<br></span></span><span id="A2.p3.pic1.2.5"><span id="A2.p3.pic1.2.5.1" style="font-size:90%;--ltx-fg-color:#000000;">Your answer should be formatted as follows:<br></span></span><span id="A2.p3.pic1.2.6"><span id="A2.p3.pic1.2.6.1" style="font-size:90%;--ltx-fg-color:#000000;">Explanation: <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&lt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&lt;</mo> <annotation encoding="application/x-tex">&lt;</annotation></semantics></math> explanation <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&gt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&gt;</mo> <annotation encoding="application/x-tex">&gt;</annotation></semantics></math><br></span></span><span id="A2.p3.pic1.2.7"><span id="A2.p3.pic1.2.7.1" style="font-size:90%;--ltx-fg-color:#000000;">Bugged Code:<br><span id="A2.p3.pic1.2.7.1.1">‘‘‘</span><br><math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&lt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&lt;</mo> <annotation encoding="application/x-tex">&lt;</annotation></semantics></math> bugged_code <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&gt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&gt;</mo> <annotation encoding="application/x-tex">&gt;</annotation></semantics></math><br><span id="A2.p3.pic1.2.7.1.2">‘‘‘</span><br></span></span></span></foreignObject></g></g></svg>

Figure 11: System prompt provided to an LM to generate bugs by modifying an existing, working function. Text in red are not included at the actual prompt.

### B.1 Generating with an LM

We describe our workflows for generating bugs with an LM. For each function or class in a codebase, we prompt an LM to generate either a rewrite that introduces bugs or a complete re-implementation from scratch. This strategy is illustrated in Figure 12.

Figure 12: Workflow to generate bugs for a function or class with an LM. We first extract all functions or classes from a codebase, then enumerate across all candidates and prompt the LM to generate either a bug-laced rewrite or a re-implementation.

Modify existing functions. Given a Python codebase, we use the ast library to identify all unique functions, excluding any functions found under a testing related directory (e.g. tests, testing). Next, given a function, the LM is asked to write a new version that introduces logical, runtime bugs. Within the prompt, shown in Figure B, several suggestions of types of bugs along with a demonstration of a rewrite are provided.

<svg id="A2.SS1.p3.pic1" height="343.63" overflow="visible" version="1.1" viewBox="0 0 477.38 343.63" width="477.38"><g style="--ltx-stroke-color:#000000;--ltx-fill-color:#000000;" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,343.63) matrix(1 0 0 -1 0 0)"><g style="--ltx-fill-color:#1F77B4;" fill="#1F77B4" fill-opacity="1.0"><path style="stroke:none" d="M 0 5.91 L 0 337.73 C 0 340.99 2.64 343.63 5.91 343.63 L 471.47 343.63 C 474.73 343.63 477.38 340.99 477.38 337.73 L 477.38 5.91 C 477.38 2.64 474.73 0 471.47 0 L 5.91 0 C 2.64 0 0 2.64 0 5.91 Z"></path></g><g style="--ltx-fill-color:#F4F8FB;" fill="#F4F8FB" fill-opacity="1.0"><path style="stroke:none" d="M 1.97 5.91 L 1.97 319.52 L 475.41 319.52 L 475.41 5.91 C 475.41 3.73 473.65 1.97 471.47 1.97 L 5.91 1.97 C 3.73 1.97 1.97 3.73 1.97 5.91 Z"></path></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 328.12)"><foreignObject style="--ltx-fo-width:31.37em;--ltx-fo-height:0.69em;--ltx-fo-depth:0.19em;font-size:10pt;" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 9.61)" width="434.07"><span id="A2.SS1.p3.pic1.1" style="width:31.37em;"><span id="A2.SS1.p3.pic1.1.1"><span id="A2.SS1.p3.pic1.1.1.1" style="--ltx-fg-color:#FFFFFF;">Prompts for reimplementing bugs with an LM</span></span> </span></foreignObject></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 16.89)"><foreignObject style="--ltx-fo-width:33.91em;--ltx-fo-height:21.02em;--ltx-fo-depth:0.23em;font-size:10pt;" height="293.94" overflow="visible" transform="matrix(1 0 0 -1 0 290.82)" width="469.22"><span id="A2.SS1.p3.pic1.2" style="width:33.91em;"><span id="A2.SS1.p3.pic1.2.1"><span id="A2.SS1.p3.pic1.2.1.1" style="font-size:90%;--ltx-fg-color:#000000;">System Prompt<span id="A2.SS1.p3.pic1.2.1.1.1"><br>You are a software developer and you have been asked to implement a function.<br></span></span></span><span id="A2.SS1.p3.pic1.2.2"><span id="A2.SS1.p3.pic1.2.2.1" style="font-size:90%;--ltx-fg-color:#000000;">You will be given the contents of an entire file, with one or more functions defined in it. Please implement the function(s) that are missing. Do NOT modify the function signature, including the function name, parameters, return types, or docstring if provided. Do NOT change any other code in the file. You should not use any external libraries.<br></span></span><span id="A2.SS1.p3.pic1.2.3"><span id="A2.SS1.p3.pic1.2.3.1" style="font-size:90%;--ltx-fg-color:#000000;">Task Instance Prompt<span id="A2.SS1.p3.pic1.2.3.1.1"><br>Please implement the function func_signature in the following code:<br></span></span></span><span id="A2.SS1.p3.pic1.2.4"><span id="A2.SS1.p3.pic1.2.4.1" style="font-size:90%;--ltx-fg-color:#000000;">{file_src_code}<br></span></span><span id="A2.SS1.p3.pic1.2.5"><span id="A2.SS1.p3.pic1.2.5.1" style="font-size:90%;--ltx-fg-color:#000000;">Remember, you should not modify the function signature, including the function name, parameters, return types, or docstring if provided. Do NOT change any other code in the file. Format your output as:<br></span></span><span id="A2.SS1.p3.pic1.2.6"><span id="A2.SS1.p3.pic1.2.6.1" style="font-size:90%;--ltx-fg-color:#000000;">[explanation]<br></span></span><span id="A2.SS1.p3.pic1.2.7"><span id="A2.SS1.p3.pic1.2.7.1" style="font-size:90%;--ltx-fg-color:#000000;">{func_to_write}</span></span></span></foreignObject></g></g></svg>

Figure 13: System prompt provided to an LM to generate bugs by re-implementing an existing target function. file\_src\_code refers to the original source file minus the target function’s original implementation. func\_to\_write refers to the signature and docstring of the target function.

In our experiments, we use OpenAI’s o3 mini model [^28] (o3-mini-2025-01-31) as the main base model for bug generation. Based on our empirical observations of an LM’s tendencies, we include several explicit guidelines in the prompt about what the rewrite should not do. Notably, it is important to ask the LM to not generate any inline comments denoting the location of a bug; we observe that without explicitly specifying this, model generation outputs tend to have inline comments pointing out the bug. We also want to avoid the complexities of identifying and removing such comments from a file diff representation. Second, we state that rewrites causing compilation or syntax errors (e.g. undeclared variables, function definition modifications) should be avoided because such bugs are relatively trivial to solve. We do not experiment extensively with different prompts or generating multiple buggy rewrites per function.

Modify existing classes. This method involves a simple amendment to the function rewriting approach. Instead of identifying unique functions (ast.FunctionDef), the codebase traversal logic instead looks for classes (ast.ClassDef). Otherwise, all other aspects of the implementation are near identical to function rewriting, with minor changes to the prompt to make bug suggestions and the demonstration more class oriented.

Rewrite existing functions. Instead of providing an LM with the original function, we explore an alternative strategy of asking an LM to re-implement a function from scratch. Similar to above, we again use the ast library to identify all unique functions. However, instead of directly asking for a bug, we remove the function’s implementation, then prompt the LM with the entire file containing the function (minus the original implementation). In the task description, we then explicitly ask for the LM to implement the function without changing the function signature.

### B.2 Procedural Modification

We explore a zero-cost approach to create bugs by performing random modifications to the ast representation of a function or class. A “procedural modification” refers to a function that takes in an ast and applies a fixed transformation to it, such as removing a loop or swapping the blocks of an if/else clause. This strategy is illustrated in Figure 14.

Figure 14: Workflow to generate bugs via procedural modifications. Per function/class, the source code is first convert into an ast. The modification then mutates the ast (e.g. removes an assignment statement). The ast is then converted back into source code with the specific modification introduced.

Similar to the workflow for generating bugs with an LM, we first identify all functions or classes in a repository. Per procedural modification, we first impose a set of criteria that filters out any candidates for which the modification would be impossible. For instance, if the procedural modification removes a random conditional from a function, the modification’s criteria will filter out any candidates that are not functions or do not have a conditional. For the remaining candidates, the procedural modification is applied with controlled likelihood, where likelihood is a fraction indicating how often the procedural modification is applied within a candidate. For example, if the procedural modification removes a random function with a likelihood of $0.5$, then for every conditional declared within the function, there is a $50$ % chance it gets removed. We introduce likelihood so procedural modifications do not lead to changes that are too difficult. Finally, the modified ast is converted back into source code.

Table 7 is a complete list of filtering criteria that is used for any procedural modification. For the filter\_min\_complexity and filter\_max\_complexity criteria, we define a simple definition of “complexity” as a sum of the number of conditional blocks, loops, boolean operators, exception handling blocks, and comparison operators in a function. The purpose of filter\_min\_complexity is to remove both simple, uninteresting functions (e.g. getter, setter methods) from consideration. filter\_max\_complexity is occasionally used to avoid changing long, monolithic functions.

| Index | Criteria | Description |
| --- | --- | --- |
| 1 | filter\_functions | Is the ast a function definition |
| 2 | filter\_classes | Is the ast a class definition |
| 3 | filter\_classes\_has\_base | Is the ast a class definition with parents |
| 4 | filter\_loops | Does the ast contain a For or While loop? |
| 5 | filter\_conditionals | Does the ast contain a conditional block? |
| 6 | filter\_assignments | Is the ast a function def. with assignments? |
| 7 | filter\_wrappers | Does the ast contain try or with blocks? |
| 8 | filter\_if\_else | Does the ast contain an if-else block? |
| 9 | filter\_operators | Does the ast contain binary, boolean operators? |
| 10 | filter\_min\_complexity | Is the ast $\geq$ a complexity score? |
| 11 | filter\_max\_complexity | Is the ast $\leq$ a complexity score? |

Table 7: Pool of criteria used to filter for functions or classes with specific properties. Per procedural modification, a subset of these criteria is first used to filter functions and/or classes from a codebase. The modification is then run on the remainder.

Table 8 is an exhaustive list of all procedural modifications used to create bugs in a codebase.

<table><tbody><tr><th colspan="2">Procedural Modification</th><th>Criteria</th><td>Description</td></tr><tr><th>Class</th><td>Remove Functions</td><th>2, 10</th><td>Removes method(s) + reference(s).</td></tr><tr><th></th><td>Remove Parent</td><th>3, 10</th><td>Removes base class from class header.</td></tr><tr><th></th><td>Shuffle Methods</td><th>2, 10</th><td>Shuffles method definitions in a class.</td></tr><tr><th>Control</th><td>Invert If/Else</td><th>8</th><td>Inverts the if-else bodies of a condition.</td></tr><tr><th>Flow</th><td>Shuffle Lines</td><th>11, 12</th><td>Shuffles the lines of a function.</td></tr><tr><th>Expressions</th><td>Change Constants</td><th>1, 9, 10</th><td><math><semantics><mrow><mo>±</mo> <mn>1</mn></mrow> <annotation>\pm 1</annotation></semantics></math> to a constant numeric value.</td></tr><tr><th></th><td>Break Chains</td><th>1, 9, 10</th><td>Removes operator(s), operator(s).</td></tr><tr><th></th><td>Swap Operands</td><th>1, 9, 10</th><td>Mixes order of operands.</td></tr><tr><th></th><td>Change Operator</td><th>1, 9, 10</th><td>Changes operator(s) (e.g. <math><semantics><mo>+</mo> <annotation>+</annotation></semantics></math> to <math><semantics><mo>−</mo> <annotation>-</annotation></semantics></math>).</td></tr><tr><th>Removal</th><td>Loops</td><th>1, 4, 10</th><td>Remove loops (e.g. for, while).</td></tr><tr><th></th><td>Conditionals</td><th>1, 5, 10</th><td>Remove conditionals (if).</td></tr><tr><th></th><td>Assignments</td><th>1, 6, 10</th><td>Remove assignment statements.</td></tr><tr><th></th><td>Wrappers</td><th>1, 7, 10</th><td>Remove exception (try), context (with).</td></tr></tbody></table>

Table 8: The $13$ procedural modification techniques we use to create bugs in a codebase. The “Criteria” column contains indices referencing the corresponding filter defined in Table 7. There are four informal categories — Class, Control Flow, Expressions, Removal — which indicates the general type of modification being made.

### B.3 Combine Bug Patches

We discuss the two strategies we use to combine bug patches from the same file or the same module. In practice, we combine LM and procedurally generated bugs that have been validated successfully as usable task instances.

Figure 15: Workflow to generate bugs by combining bug patches. We take $n$ patches (generated using an LM or procedural modification), then sequentially apply each bug patch to the codebase. If all individual patches apply successfully, we save the resulting single patch which now represents all $n$ bugs combined.

From the same file. If two or more functions are defined within a single file, this strategy merges the function-level bug patches together. Given $n$ function-level bugs and $k$ as the number of bugs to combine, there are ${n\choose k}$ unique file-level candidate bug patches, which can be a large search space to cover. To make the search space tractable, ensure no single function-level bug is repeatedly used, and generate instances that reliably have $1$ + Fail to Pass tests, we implement the following approach described in Algorithm 1.

$codebase$, $bugs$; $num\_bugs$, $limit\_per\_file$; $max\_combos$

$min\_bugs$ $\geq$ 2;

$max\_bugs$ $\geq$ $min\_bugs$;

procedure CombineFileBugs

  for each $file$ in $codebase$ do

    $file\_bugs$ $\leftarrow$ bugs that apply to $file$

    $combinations$ $\leftarrow$ get\_combos($file\_bugs$, $num\_bugs$, $max\_combos$)

   for each $combo$ in combinations do

     Apply $combo$ to $codebase$

     if success then

      Save $combo$ to disk

      if $limit\_per\_file$ reached then

        break

      end if

       $combinations$ $\leftarrow$ \[c for c in $combinations$ if c $\cap$ $combo$ $=\emptyset$\]

     end if

   end for

  end for

end procedure

Algorithm 1 Combine multiple patches from the same file.

For each file in a codebase, we first identify the function-level bugs (or bug patches) that edit that file. The pool of bugs we draw from have been validated, meaning we have already ensured there is $1$ + Fail to Pass test(s) associated with the bug. From these pool of file\_bugs, the get\_combos function then generates up to max\_combos sets of bugs, where the size of each set is num\_bugs. For each combo, or set of bugs, the bugs are applied to the codebase one by one. If all patches are successfully combined, this means they were successfully merged, and the merged patch, which consists of multiple function-level bugs, is saved and re-validated as a single bug. Merging patches occasionally fails if there is an overlapping conflict between two files, akin to a merge conflict with git; this usually happens when a function is declared within another. To ensure a function-level bug is only used once, any remaining bug sets in combinations using any patch in combo are removed.

The limit\_per\_file and max\_combos parameters prevent any one file from being over-represented and constrains an otherwise combinatorial large search space. We run this algorithm across all codebase files, typically setting num\_bugs $=[2,4]$, limit\_per\_file $=3$, max\_combos $=40$. Decreasing num\_bugs or increasing the other three parameters improves the yield.

From the same module. There are several ways one could imagine composing function-level bugs from multiple bugs, such as combining those that break the same test or have a programmatic relationship (e.g. function a calls function b). We found a relatively straightforward and effective approach to be combining files that edit the same “module”. By “module” we are referring to a subdirectory within the source code (e.g. sklearn/feature\_extraction, astropy/convolution). Out of all SWE-bench instances that edit $2$ + files, $75$ % modify files within the same submodule, suggesting a high degree of intra-module code changes. The implementation for our approach is described in Algorithm 2

$bugs$; $num\_bugs$; $limit\_per\_module$; $max\_combos$; $depth$

$num\_bugs$ $\geq$ 2;

procedure CombineModuleBugs

   $map\_path\_to\_bugs\leftarrow\{\}$

  for each $bug$ in $bugs$ do

    $path\leftarrow$ get\_path\_from(bug)

    $map\_path\_to\_patches[path]\leftarrow[bug]$

  end for

  Collapse nested paths based on $depth$

  for all $(path,patches)$ in $map\_path\_to\_patches$ do

    $combinations$ $\leftarrow$ get\_combos(patches, $num\_bugs$, $max\_combos$)

   for each $combo$ in $combinations$ do

     Apply $combo$ to $codebase$

     if success and num\_files\_changed(combo) $\geq 2$ then

      Save $combo$ to disk

      if $limit\_per\_module$ reached then

        break

      end if

       $combinations$ $\leftarrow$ \[c for c in $combinations$ if c $\cap$ $combo$ $=\emptyset$\]

     end if

   end for

  end for

end procedure

Algorithm 2 Combine multiple patches from the same module.

The implementation for this approach is similar to Algorithm 1 with two key changes. First, we do not do file-by-file or folder-by-folder traversal. Instead, using the diff patches, we create a dictionary map\_path\_to\_bugs that mimics the file structure of a codebase. For example, if bug modifies path a/b/c/d.py, it is represented as map\_path\_to\_bugs\[a\]\[b\]\[c\]\[d.py\] $=$ \[bug\]. Additional bugs that modify the same path are appended to the list. Since every bug is a function-level bug, there will never be a bug registered in multiple lists. We then “collapse” up to depth indices. So for instance, at depth $=3$, the above data structure is collapsed into map\_path\_to\_bugs\[a/b/c\]\[d.py\] $=$ \[bug\]. Finally, any nested dictionaries are collapsed into a single list of patches (e.g. map\_path\_to\_bugs\[a/b/c\] $=$ \[bug\]). Mirroring the procedure in Algorithm 1, we then iterate across this dictionary’s values (lists of bugs). Second, we only save patches that modify $2+$ files; aggregate bugs (represented by combo) modifying a single file are not considered.

Again, we run this strategy across all $100$ repositories, with parameters num\_bugs $=[2,5]$, limit\_per\_module $=10$, max\_combos $=100$, and depth $=2$. Reducing num\_bugs, depth and increasing the other parameters yields more bugs. We choose a depth of $2$ because empirically, we find that meaningful modules are usually declared as immediate sub-folders of the main source code folder (e.g. in sklearn/feature\_extraction, sklearn is the source code folder while feature\_extraction is the module). A shallower depth leads to less meaningful groupings, while yield decreases significantly for every increased level of depth, particularly for smaller repositories.

### B.4 Pull Request Mirroring

We finally discuss the fourth and last strategy for generating bugs - mirroring real world pull requests (PR). We visualize this process in Figure 16.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2504.21798/assets/diagram-pr-mirror.png)

Figure 16: Workflow to generate bugs by reverting changes made in the diff patch corresponding to a real GitHub pull request (PR). Given the patch and the files modified by the patch, we prompt the LM to generate a complete rewrite of each file that reverses the changes made in the PR. The changes are applied to the codebase, and we extract the patch, which now captures the reversal of the PR changes.

Why use an LM? When we initially implemented this approach, we attempted to directly perform a git apply --reverse \[patch\] on the codebase. However, for the large majority of patches, this fails. We performed troubleshooting by inspecting $100$ PR patches on the sqlfluff/sqlfluff repository, leading us to two observations.

1. The majority of these PRs reflect changes that remain present in the codebase today (making the bug creation promising).
2. However, many patches can not be reversed because the exact location (e.g. lines, file) of the relevant code changed because of other changes.

Therefore, we employ LMs to perform patch reversal, and find that reasoning models (e.g. o3-mini [^28]) are particularly effective.

Description of method. We follow SWE-bench’s methodology for crawling PRs created January 1st, 2023 and onwards, with minor and arbitrary exceptions for some repositories where we crawl older PRs as well. Per PR, we iterate across the file(s) changed by the patch. Per file, we prompt an LM with the file-specific changes from the patch along with the file’s source code in the current state of the repository (not the repository’s state corresponding to when the PR was applied, referred to as the base\_commit in SWE-bench). The LM is asked to generate a rewrite of the file that reverts the changes reflected in the PR. We aggregate the changes across all file(s) into a single patch.

Because we are interested in problems that our expert trajectory generation method (SWE-agent + Claude 3.7 Sonnet) has a chance of solving, we do not attempt to reproduce PRs that change more than $8$ files. This constraint is imposed because no SWE-bench instance that edits more than $6$ files has ever been solved [^17].

How well does PR mirroring work? We scrape the PRs corresponding to $100$ randomly selected SWE-bench task instances from the django/django GitHub repository and attempt to recreate these task instances with SWE-smith’s collection process. We successfully recovered $92$ of $100$ task instances. Of these, $84$ break identical F2P test(s), with the remaining $8$ breaking a subset because some tests were removed over time. This sanity check gives us confidence that the PR mirroring strategy lives up to its name.

Comparison to SWE-bench. This approach has several benefits and drawbacks compared to SWE-bench’s collection pipeline. First, it removes the need to create instance-specific Docker images — all PRs are mirrored against the same version of a repository. This also implies that there is no need to write installation specifications for past versions of a repository, which is typically the most laborious step in task construction with SWE-bench. Finally, this strategy also allows us to loosen the requirements on what PRs we attempt to convert into a task instance. In SWE-bench, the core requirements for what PRs to attempt to convert into a task instance include:

1. It must edit $1+$ code files (e.g. not just.md,.rst files).
2. It must reference $1+$ GitHub issues, which serves as the problem statement.
3. It must edit $1+$ testing related files ($1+$ files with a test-adjacent keyword in it).

With this collection strategy and SWE-smith’s focus on training data, the second and third requirements are no longer necessary. If there is no associated issue, issue text can simply be generated. If the patch does not contain any testing related changes, this is tolerable, as the validation stage will determine whether the PR breaks any tests. With these considerations, we purport that SWE-smith’s PR mirroring strategy can re-purpose a higher percentage of real world code changes for training purposes.

The main downside is that the rest of the repository is out of sync with the state of the codebase when the PR was applied. As a result, it’s possible that changes in the behavior of the rest of the codebase may affect the issue’s reproducibility or the accuracy of the issue description (e.g. line numbers referenced in the issue text are likely somewhat off with respect to the codebase). However, a simple mitigation for this is to create a Docker image for a repository at an earlier commit that’s closer to the original creation date of the issue. While we do not carry out a targeted experiment, we hypothesize that using SWE-smith, we would be able to reproduce SWE-bench entirely with $10$ x less human hours with an estimated $2294$ x $ $0.055$ = $ $126.17$ in costs.

## Appendix C Dataset Statistics

We present additional breakdowns and analyses of the SWE-smith dataset, focusing on the kinds of repositories and bugs that are represented.

Repository categorization. We present an exhaustive list of repositories used in SWE-smith in Table 9. We categorize the repositories into seven general buckets: Data Parsing and Transformation ($39$), Web & API Development ($11$), Code Quality & Testing ($12$), Visualization & Presentation ($8$), System Tools & Protocols ($17$), Natural Language Processing ($7$), and Miscellaneous ($6$). The categorizations were performed by first, determining an appropriate set of categories based on manual inspection supported by the descriptions and GitHub topics associated with each repository. After settling upon the buckets, we asked GPT-4o to provide a label based on the repository’s metadata and README dump. SWE-smith represents a wider and more variegated coverage of software tools and applications compared to any prior works.

<table><thead><tr><th>Repository</th><th>Description</th></tr></thead><tbody><tr><td colspan="2">Code Quality and Testing</td></tr><tr><td>PyCQA/flake8</td><td>flake8 is a python tool that glues together pycodestyle, pyflakes, mccabe, and third-party plugins to check the style and quality of some python code.</td></tr><tr><td>Suor/funcy</td><td>A fancy and practical functional tools</td></tr><tr><td>adrienverge/yamllint</td><td>A linter for YAML files.</td></tr><tr><td>agronholm/typeguard</td><td>Run-time type checker for Python</td></tr><tr><td>cknd/stackprinter</td><td>Debugging-friendly exceptions for Python</td></tr><tr><td>cool-RR/PySnooper</td><td>Never use print for debugging again</td></tr><tr><td>getmoto/moto</td><td>A library that allows you to easily mock out tests based on AWS infrastructure.</td></tr><tr><td>pylint-dev/astroid</td><td>A common base representation of python source code for pylint and other projects</td></tr><tr><td>pytest-dev/iniconfig None pytest-dev/iniconfig</td><td>None</td></tr><tr><td>python/mypy</td><td>Optional static typing for Python</td></tr><tr><td>pyupio/safety</td><td>Safety checks Python dependencies for known security vulnerabilities and suggests the proper remediations for vulnerabilities detected.</td></tr><tr><td>pyutils/line_profiler</td><td>Line-by-line profiling for Python</td></tr><tr><td>rubik/radon</td><td>Various code metrics for Python code</td></tr><tr><td>spulec/freezegun</td><td>Let your Python tests travel through time</td></tr><tr><td>sqlfluff/sqlfluff</td><td>A modular SQL linter and auto-formatter with support for multiple dialects and templated code.</td></tr><tr><td colspan="2">Data Parsing and Transformation</td></tr><tr><td>alecthomas/voluptuous</td><td>CONTRIBUTIONS ONLY: Voluptuous, despite the name, is a Python data validation library.</td></tr><tr><td>andialbrecht/sqlparse</td><td>A non-validating SQL parser module for Python</td></tr><tr><td>buriy/python-readability</td><td>fast python port of arc90’s readability tool, updated to match latest readability.js!</td></tr><tr><td>burnash/gspread</td><td>Google Sheets Python API</td></tr><tr><td>chardet/chardet</td><td>Python character encoding detector</td></tr><tr><td>cloudpipe/cloudpickle</td><td>Extended pickling support for Python objects</td></tr><tr><td>dask/dask</td><td>Parallel computing with task scheduling</td></tr><tr><td>datamade/usaddress</td><td>:us: a python library for parsing unstructured United States address strings into address components</td></tr><tr><td>davidhalter/parso</td><td>A Python Parser</td></tr><tr><td>erikrose/parsimonious</td><td>The fastest pure-Python PEG parser I can muster</td></tr><tr><td>facelessuser/soupsieve</td><td>A modern CSS selector implementation for BeautifulSoup</td></tr><tr><td>gawel/pyquery</td><td>A jquery-like library for python</td></tr><tr><td>google/textfsm</td><td>Python module for parsing semi-structured text into python tables.</td></tr><tr><td>gruns/furl</td><td>�� URL parsing and manipulation made easy.</td></tr><tr><td>gweis/isodate</td><td>ISO 8601 date/time parser</td></tr><tr><td>hukkin/tomli</td><td>A lil’ TOML parser</td></tr><tr><td>jawah/charset_normalizer</td><td>Truly universal encoding detector in pure Python</td></tr><tr><td>john-kurkowski/tldextract</td><td>Accurately separates a URL’s subdomain, domain, and public suffix, using the Public Suffix List (PSL).</td></tr><tr><td>joke2k/faker</td><td>Faker is a Python package that generates fake data for you.</td></tr><tr><td>jsvine/pdfplumber</td><td>Plumb a PDF for detailed information about each char, rectangle, line, et cetera — and easily extract text and tables.</td></tr><tr><td>kayak/pypika</td><td>PyPika is a python SQL query builder that exposes the full richness of the SQL language using a syntax that reflects the resulting query. PyPika excels at all sorts of SQL queries but is especially useful for data analysis.</td></tr><tr><td>keleshev/schema</td><td>Schema validation just got Pythonic</td></tr><tr><td>kennethreitz/records</td><td>SQL for Humans™</td></tr><tr><td>kurtmckee/feedparser</td><td>Parse feeds in Python</td></tr><tr><td>lepture/mistune</td><td>A fast yet powerful Python Markdown parser with renderers and plugins.</td></tr><tr><td>madzak/python-json-logger</td><td>Json Formatter for the standard python logger</td></tr><tr><td>mahmoud/glom</td><td>☄️ Python’s nested data operator (and CLI), for all your declarative restructuring needs. Got data? Glom it! ☄️</td></tr><tr><td>marshmallow-code/marshmallow</td><td>A lightweight library for converting complex objects to and from simple Python datatypes.</td></tr><tr><td>martinblech/xmltodict</td><td>Python module that makes working with XML feel like you are working with JSON</td></tr><tr><td>matthewwithanm/python-markdownify</td><td>Convert HTML to Markdown</td></tr><tr><td>mewwts/addict</td><td>The Python Dict that’s better than heroin.</td></tr><tr><td>mido/mido</td><td>MIDI Objects for Python</td></tr><tr><td>modin-project/modin</td><td>Modin: Scale your Pandas workflows by changing a single line of code</td></tr><tr><td>mozilla/bleach</td><td>Bleach is an allowed-list-based HTML sanitizing library that escapes or strips markup and attributes</td></tr><tr><td>msiemens/tinydb</td><td>TinyDB is a lightweight document oriented database optimized for your happiness:)</td></tr><tr><td>pandas-dev/pandas</td><td>Flexible and powerful data analysis / manipulation library for Python, providing labeled data structures similar to R data.frame objects, statistical functions, and much more</td></tr><tr><td>pdfminer/pdfminer.six</td><td>Community maintained fork of pdfminer - we fathom PDF</td></tr><tr><td>pudo/dataset</td><td>Easy-to-use data handling for SQL data stores with support for implicit table creation, bulk loading, and transactions.</td></tr><tr><td>pydantic/pydantic</td><td>Data validation using Python type hints</td></tr><tr><td>pydata/patsy</td><td>Describing statistical models in Python using symbolic formulas</td></tr><tr><td>pydicom/pydicom</td><td>Read, modify and write DICOM files with python code</td></tr><tr><td>pygments/pygments</td><td>Pygments is a generic syntax highlighter written in Python</td></tr><tr><td>pyparsing/pyparsing</td><td>Python library for creating PEG parsers</td></tr><tr><td>python-jsonschema/jsonschema</td><td>An implementation of the JSON Schema specification for Python</td></tr><tr><td>python-openxml/python-docx</td><td>Create and modify Word documents with Python</td></tr><tr><td>r1chardj0n3s/parse</td><td>Parse strings using a specification based on the Python format() syntax.</td></tr><tr><td>scanny/python-pptx</td><td>Create Open XML PowerPoint documents in Python</td></tr><tr><td>scrapy/scrapy</td><td>Scrapy, a fast high-level web crawling & scraping framework for Python.</td></tr><tr><td>seperman/deepdiff</td><td>DeepDiff: Deep Difference and search of any Python object/data. DeepHash: Hash of any object based on its contents. Delta: Use deltas to reconstruct objects by adding deltas together.</td></tr><tr><td>sloria/environs</td><td>simplified environment variable parsing</td></tr><tr><td>sunpy/sunpy</td><td>SunPy - Python for Solar Physics</td></tr><tr><td>tkrajina/gpxpy</td><td>gpx-py is a python GPX parser. GPX (GPS eXchange Format) is an XML based file format for GPS tracks.</td></tr><tr><td>tobymao/sqlglot</td><td>Python SQL Parser and Transpiler</td></tr><tr><td>un33k/python-slugify</td><td>Returns unicode slugs</td></tr><tr><td colspan="2">Machine Learning and AI</td></tr><tr><td>facebookresearch/fvcore</td><td>Collection of common code that’s shared among different research projects in FAIR computer vision team.</td></tr><tr><td>facebookresearch/hydra</td><td>Hydra is a framework for elegantly configuring complex applications</td></tr><tr><td>HIPS/autograd</td><td>Efficiently computes derivatives of NumPy code.</td></tr><tr><td>iterative/dvc</td><td>�� Data Versioning and ML Experiments</td></tr><tr><td>jaraco/inflect</td><td>Correctly generate plurals, ordinals, indefinite articles; convert numbers to words</td></tr><tr><td>life4/textdistance</td><td>�� Compute distance between sequences. 30+ algorithms, pure python implementation, common interface, optional external libs usage.</td></tr><tr><td>luozhouyang/python-string-similarity</td><td>A library implementing different string similarity and distance measures using Python.</td></tr><tr><td>Mimino666/langdetect</td><td>Port of Google’s language-detection library to Python.</td></tr><tr><td>mozillazg/python-pinyin</td><td>汉字转拼音(pypinyin)</td></tr><tr><td>pndurette/gTTS</td><td>Python library and CLI tool to interface with Google Translate’s text-to-speech API</td></tr><tr><td>Project-MONAI/MONAI</td><td>AI Toolkit for Healthcare Imaging</td></tr><tr><td>seatgeek/thefuzz</td><td>Fuzzy String Matching in Python</td></tr><tr><td>vi3k6i5/flashtext</td><td>Extract Keywords from sentence or Replace keywords in sentences.</td></tr><tr><td colspan="2">System Tools and Protocols</td></tr><tr><td>agronholm/exceptiongroup</td><td>Backport of PEP 654 (exception groups)</td></tr><tr><td>aio-libs/async-timeout</td><td>asyncio-compatible timeout class</td></tr><tr><td>arrow-py/arrow</td><td>�� Better dates & times for Python</td></tr><tr><td>borntyping/python-colorlog</td><td>A colored formatter for the python logging module</td></tr><tr><td>cantools/cantools</td><td>CAN bus tools.</td></tr><tr><td>conan-io/conan</td><td>Conan - The open-source C and C++ package manager</td></tr><tr><td>cookiecutter/cookiecutter</td><td>A cross-platform command-line utility that creates projects from cookiecutters (project templates), e.g. Python package projects, C projects.</td></tr><tr><td>dbader/schedule</td><td>Python job scheduling for humans.</td></tr><tr><td>gruns/icecream</td><td>�� Never use print() to debug again.</td></tr><tr><td>jd/tenacity</td><td>Retrying library for Python</td></tr><tr><td>mahmoud/boltons</td><td>�� Like builtins, but boltons. 250+ constructs, recipes, and snippets which extend (and rely on nothing but) the Python standard library. Nothing like Michael Bolton.</td></tr><tr><td>oauthlib/oauthlib</td><td>A generic, spec-compliant, thorough implementation of the OAuth request-signing logic</td></tr><tr><td>pallets/click</td><td>Python composable command line interface toolkit</td></tr><tr><td>paramiko/paramiko</td><td>The leading native Python SSHv2 protocol library.</td></tr><tr><td>pexpect/ptyprocess</td><td>Run a subprocess in a pseudo terminal</td></tr><tr><td>pyasn1/pyasn1</td><td>Generic ASN.1 library for Python</td></tr><tr><td>pyca/pyopenssl</td><td>A Python wrapper around the OpenSSL library</td></tr><tr><td>python-hyper/h11</td><td>A pure-Python, bring-your-own-I/O implementation of HTTP/1.1</td></tr><tr><td>python-trio/trio</td><td>Trio – a friendly Python library for async concurrency and I/O</td></tr><tr><td>rustedpy/result</td><td>NOT MAINTAINED - A simple Rust like Result type for Python 3. Fully type annotated.</td></tr><tr><td>termcolor/termcolor</td><td>ANSI color formatting for output in terminal</td></tr><tr><td>theskumar/python-dotenv</td><td>Reads key-value pairs from a.env file and can set them as environment variables. It helps in developing applications following the 12-factor principles.</td></tr><tr><td>tox-dev/pipdeptree</td><td>A command line utility to display dependency tree of the installed Python packages</td></tr><tr><td colspan="2">Visualization and Presentation</td></tr><tr><td>amueller/word_cloud</td><td>A little word cloud generator in Python</td></tr><tr><td>lincolnloop/python-qrcode</td><td>Python QR Code image generator</td></tr><tr><td>prettytable/prettytable</td><td>Display tabular data in a visually appealing ASCII table format</td></tr><tr><td>pwaller/pyfiglet</td><td>An implementation of figlet written in Python</td></tr><tr><td>rsalmei/alive-progress</td><td>A new kind of Progress Bar, with real-time throughput, ETA, and very cool animations!</td></tr><tr><td>weaveworks/grafanalib</td><td>Python library for building Grafana dashboards</td></tr><tr><td colspan="2">Web and API Development</td></tr><tr><td>Cog-Creators/Red-DiscordBot</td><td>A multi-function Discord bot</td></tr><tr><td>Knio/dominate</td><td>Dominate is a Python library for creating and manipulating HTML documents using an elegant DOM API. It allows you to write HTML pages in pure Python very concisely, which eliminate the need to learn another template language, and to take advantage of the more powerful features of Python.</td></tr><tr><td>alanjds/drf-nested-routers</td><td>Nested Routers for Django Rest Framework</td></tr><tr><td>benoitc/gunicorn</td><td>gunicorn ’Green Unicorn’ is a WSGI HTTP Server for UNIX, fast clients and sleepy applications.</td></tr><tr><td>bottlepy/bottle</td><td>bottle.py is a fast and simple micro-framework for python web-applications.</td></tr><tr><td>django-money/django-money</td><td>Money fields for Django forms and models.</td></tr><tr><td>django/channels</td><td>Developer-friendly asynchrony for Django</td></tr><tr><td>django/daphne</td><td>Django Channels HTTP/WebSocket server</td></tr><tr><td>encode/starlette</td><td>The little ASGI framework that shines. ��</td></tr><tr><td>getnikola/nikola</td><td>A static website and blog generator</td></tr><tr><td>graphql-python/graphene</td><td>GraphQL framework for Python</td></tr><tr><td>marshmallow-code/apispec</td><td>A pluggable API specification generator. Currently supports the OpenAPI Specification (f.k.a. the Swagger specification)..</td></tr><tr><td>marshmallow-code/webargs</td><td>A friendly library for parsing HTTP request arguments, with built-in support for popular web frameworks, including Flask, Django, Bottle, Tornado, Pyramid, webapp2, Falcon, and aiohttp.</td></tr><tr><td>pallets/jinja</td><td>A very fast and expressive template engine.</td></tr><tr><td>pallets/markupsafe</td><td>Safely add untrusted strings to HTML/XML markup.</td></tr><tr><td>tornadoweb/tornado</td><td>Tornado is a Python web framework and asynchronous networking library, originally developed at FriendFeed.</td></tr><tr><td>tweepy/tweepy</td><td>Twitter for Python!</td></tr></tbody></table>

### C.1 Bug Generation Statistics

We provide extensive details about different aspects of each of the bug generation strategies, including the yield rates, labor/monetary costs, and dataset characterizations.

Yield rates. In Table 10, we provide the yield rates for each bug generation method across all repositories in SWE-smith. In general, we find that the PR Mirroring has the lowest yield rate at $13.18$ % (although this rate is somewhat higher than SWE-bench’s yield rate of $2294/93139=2.46$ %). For using LMs to generate bugs, modifying functions to introduce bugs intentionally has a higher yield than asking LMs to perform a best-effort rewrite. The efficacy of Procedural Modifications varies by strategy. For instance, shuffling the functions declared in a class only breaks existing test(s) $1.93$ % of the time, but inverting a conditional will lead to a task instance for $47.04$ % of modifications. Finally, combining bug patches has an extremely high yield rate - this is to be expected because we only attempt to combine bug patches that have been validated as usable task instances breaking $1+$ tests.

| Strategy | \# Repos | \# Candidates | \# Instances | Yield Rate |
| --- | --- | --- | --- | --- |
| Combine (file) | 124 | 6020 | 5865 | 97.43% |
| Combine (module) | 65 | 4396 | 4227 | 96.16% |
| LM (Modify) | 108 | 31950 | 17887 | 55.98% |
| LM (Rewrite) | 128 | 11908 | 4173 | 35.04% |
| PR Mirroring | 108 | 6934 | 2344 | 33.8% |
| Procedural (Class Rm Base) | 103 | 1401 | 463 | 33.05% |
| Procedural (Class Rm Funcs) | 103 | 2506 | 1180 | 47.09% |
| Procedural (Class Shuffle Funcs) | 103 | 2504 | 47 | 1.88% |
| Procedural (Ctrl Invert If) | 105 | 4695 | 2321 | 49.44% |
| Procedural (Ctrl Shuffle) | 104 | 9055 | 4015 | 44.34% |
| Procedural (Op Break Chains) | 71 | 747 | 225 | 30.12% |
| Procedural (Op Change Const) | 77 | 723 | 257 | 35.55% |
| Procedural (Op Change) | 81 | 1507 | 450 | 29.86% |
| Procedural (Op Swap) | 87 | 2141 | 483 | 22.56% |
| Procedural (Remove Assign) | 121 | 5470 | 2661 | 48.65% |
| Procedural (Remove Cond) | 120 | 5288 | 2311 | 43.7% |
| Procedural (Remove Loop) | 110 | 1945 | 860 | 44.22% |
| Procedural (Remove Wrapper) | 80 | 884 | 368 | 41.63% |
| All | 129 | 100074 | 50137 | 50.1% |

Table 10: Yield rates for different bug generation strategies covered in Section B. We show the number of repositories that each strategy was run on, the number of bug candidates generated by each strategy, and the number of instances, or the number of candidates that were validated to have $1+$ Fail to Pass test. The yield rate for

The number of repositories captured by each bug generation technique varies due to each strategy’s specific preconditions, which at times may not be effective for some repositories. For instance, the Procedural (Class \*) set of methods only mutates Python classes. This strategy is fruitless for the minority of SWE-smith repositories that do not define any classes. The Procedural (Op Break Chains) method randomly removes operations and operands from expressions with two or more operations (e.g. $a+b+c\rightarrow a+b$) — such expressions are not always present in SWE-smith repositories.

The collective yield rate across SWE-smith’s bug generation strategies is significantly higher than SWE-bench’s collection strategy.

| Yield Rate | \# of Repositories |
| --- | --- |
| $0$ - $25$ % | $10$ |
| $25$ - $50$ % | $31$ |
| $50$ - $75$ % | $60$ |
| $75$ - $100$ % | $27$ |

Table 11: Yield rates for different repositories represented in SWE-smith.

The yield rate also varies with respect to the repository it is being applied to. We provide a summary of yield rates by repository in Table 11. We generally observe that lower test coverage correlates with a lower yield rate.

Dataset characterizations. In Table 12, we provide statistics about the validated task instances produced by different bug generation strategies. Our work’s LM-based strategies rewrite one function in one file. Procedural modifications will also only change one file, but depending on the strategy, $1+$ functions or classes may be changed. Combining multiple patches from the same file always produces a patch with $2+$ functions edited. Combining across modules produces a patch with $2+$ files edited. The targeted nature of each of the bug creation strategies is reflected in the typical number of functions and files that the bugs produced by each strategy edits.

| Strategy | \# Instances | \# F2P | $\Delta$ Lines | $\Delta$ Functions | $\Delta$ Files |
| --- | --- | --- | --- | --- | --- |
| Combine | 10092 | 15 (5-48) | 19 (12-36) | 2 (2-3) | 1 (1-2) |
| LM | 22060 | 4 (1-17) | 6 (3-15) | 1 (1-1) | 1 (1-1) |
| PR Mirroring | 2344 | 3 (1-14) | 20 (8-55) | 2 (2-4) | 1 (1-2) |
| Procedural | 15641 | 7 (2-32) | 7 (5-15) | 1 (1-1) | 1 (1-1) |

Table 12: Statistics for attributes of a SWE-smith task instance across different bug generation strategies, reported as median (IQR), where IQR is the inter-quartile range (25th–75th percentile).

In Figure 17, we show the distributions for different attributes of SWE-smith compared to other SWE-bench style datasets. Compared to prior works, there is a much higher proportion of task instances with more than one Fail-to-Pass test. For any one repository, we find that SWE-smith task instances collectively cause failures for a much higher percentage of the testing suit than other datasets; a potential benefit of this is that training on SWE-smith based trajectories may expose models to a much broader set of functionalities in a codebase. The number of lines and files edited by SWE-smith task instances is highly similar to the trend lines for SWE-bench Verified.

Figure 17: Comparison of cumulative distributions for Fail-to-Pass tests along with the lines and files edited by the gold patch across SWE-smith and four SWE-bench style datasets.

We note that unlike other datasets, the trend line of SWE-smith task instances is “adjustable”. In other words, the Figure 17 distributions are a capture of the task instances provided in this release of SWE-smith. However, because of SWE-smith’s flexible bug creation techniques, the distribution can be “shaped” if needed. For instance, generating more task instances using the bug patch combination method would shift all three curves in Figure 17. We make this point to highlight the fact that the attributes of SWE-bench task instances are, in a sense, constrained by real world software development behavior. On the other hand, SWE-smith can be used to break tests and code that may not be reflected at all in any existing pull request. In this sense, we argue that LMs trained on SWE-smith have better “exposure” to a codebase compared to exclusively training on pull requests.

Continuation of scaling execution environments. The validation and evaluation procedures for SWE-smith deviate slightly from SWE-bench’s harnesses. The main reasons for these differences can largely be attributed to the granularity of installation specifications. In SWE-bench, each task instance corresponds to a unique base commit, with additional version and environment\_setup\_commit keys needed as indirection for mapping an instance to the correct set of installation and testing instructions. Across time, the continuous evolution of a repository and its dependencies make for an incredibly high degree of variability in how a repository should be installed correctly. To solve this variability, the community has resorted to creating an image per task instance, as done in [^9]. Therefore, for $2294$ SWE-bench task instances, there are $2294$ unique Docker images, each at a size of at least several gigabytes ($\sim 5$ - $6$ GBs).

On the other hand, the simplicity and scalability of SWE-smith’s design allows one to support many task instances with comparatively much fewer Docker images. As mentioned above, installation and testing procedures are (repository, commit) specific. Therefore, when bugs are generated from each (repository, commit), all bugs can be reproduced and tested successfully from the same Docker image. In other words, if I generate $100$ bugs for a repository at some commit, instead of $100$ Docker images, only a single Docker image is required to run inference on any of the $100$ task instances.

This design is what enables SWE-smith to be significantly more space-efficient than SWE-bench. Based on the publicly released images, for SWE-bench’s $2294$ task instances, $1.2$ TBs of storage are required to download all Docker images locally. for SWE-bench Multimodal’s $517$ task instances, $1.2$ TBs are required. The higher per-instance Docker image size for SWE-bench Multimodal is due to how JavaScript dependency management tools (e.g. npm) require more storage compared to equivalent Python infrastructure (e.g. pypi). [^31] states that each image for the $2438$ instances an average of $2.6$ GB, totaling 6 TB of storage total. Such a storage requirement can be a significant barrier for academic practitioners.

On the other hand, with more than $20$ x the number of bugs, SWE-smith requires only $125$ Docker images total, corresponding to the number of unique (repository, commit) pairs (in this work, for each repository, we only determine installation and test specifications for one commit). The $125$ images require a total of $290.54$ GBs. In summary, compared to SWE-bench’s task collection strategy, SWE-smith’s design makes it easier to not only create task instances, but also train on them as well.

### C.2 Case Study: SWE-bench & SWE-smith

To better understand the differences between the SWE-bench and SWE-smith collection strategies, we perform SWE-smith collection on the pallets/flask GitHub repository, one of the $12$ test split repositories from the original SWE-bench benchmark. We review the steps covered in Section 2.1 applied to pallets/flask in detail. First, we defined the installation and testing specifications for the pallets/flask repository at commit bc09840. Next, we apply the LM modification bug generation strategy to this version of the repository, generating $267$ unique bugs.

We observe several differences. First, the SWE-smith collection strategy yields a much higher number of bugs outright. From SWE-bench, $11$ task instances are from the pallets/flask repository. The task instances were originally filtered from $2434$ pull requests (PRs), with $107$ satisfying SWE-bench’s filtering criteria of (1) being linked to one or more issues and (2) featuring 1+ new tests. Out of these $107$, the $11$ ($0.45$ % of $2434$) task instances represent the proportion of PRs that execution environments could be successfully constructed for. On the other hand, running the function-level rewriting strategy for bug generation originally yielded $402$ candidates, of which $267$ were determined to be valid task instances.

Second, SWE-smith requires significantly less human effort while only incurring minor costs. Collecting the $11$ pallets/flask task instances (steps include scraping PRs, determining repository versions across time, defining version-specific installation/test specifications, running execution-based validation multiple times) took an estimated $38$ hours worth of human labor. On the contrary, defining installation and testing specifications for the latest commit of pallets/flasks took $10$ minutes. The subsequent function-level rewriting strategy for bugs took $23$ minutes to run, incurring a total cost of just $ $2.47$ ($\sim$ $ $0.00613$ per instance). The final execution-based validation step that filters out $402-267=135$ unqualified bug candidates ran in $14$ minutes. Since both the bug and problem statement generation strategies are repository agnostic, no additional human intervention is necessary for these steps. Head to head, per instance for the pallets/flask repository, SWE-bench style collection requires $38\times 60/11=207.27$ minutes compared to $0.176$ minutes ($\sim 10.6$ seconds) and $ $0.00613$ in API costs using SWE-smith.

Third, collectively, SWE-smith task instances break a significantly larger proportion of existing tests in a codebase. We define “bug coverage” as the proportion of tests broken by $1$ + instance across all task instances. For the SWE-bench split of pallets/flask, there are $207$ unique tests across all $11$ instances. Of these $207$ tests, $15$ are broken by $1$ + instance, corresponding to a bug coverage rate of $7.25$ %. For the SWE-smith split of pallets/flask, there are $474$ unique tests across $267$ instances. The larger amount of tests is due to increased test coverage in the pallets/flask repository as of Nov. 28, 2024 (when SWE-smith was collected) compared to June 2023 (when SWE-bench was collected). Of these $474$ tests, $422$ are broken by $1$ + instance, a bug coverage rate of $89.03$ %. We attribute the significant difference to a consistent tendency in real world open source software development workflows, that is, the minority of tests are introduced to capture existing, errant behavior in the repository. The significant majority of tests are committed alongside working code, ensuring that already correct behavior is upheld. Well-maintained repositories will typically not merge commits that cause such tests to fail. This results in a large number of tests where few to no commits correspond to those tests’ failures.

Finally, SWE-smith does not yield instances appropriate for evaluation. The SWE-smith pipeline as presented does not produce hidden tests, a crucial difference that makes SWE-bench more suitable for evaluation. Consequently, when expert trajectories are generated, the Fail-to-Pass tests are present in the repository at inference time. Furthermore, our issue generation strategy does not include checks for known problems such as underspecified text descriptions or solution leakage [^9]. Simple amendments could make SWE-smith task instances suitable for evaluation, such as deleting Fail-to-Pass test functions or files along with a validation procedure around the ambiguity and leakage of the issue text. Finally, thorough analyses of how faithful SWE-smith task instances are to real world issues and PRs would be necessary to justify synthetic bugs for evaluation.

## Appendix D Issue Generation

We cover the four issue generation strategies we experiment with to determine issue text’s effect on how solvable a SWE-smith instance is along with the trajectory’s value as a training data point.

Generated with LM. We prompt an LM with a randomly selected SWE-bench Verified problem statement, the bug patch, list of Fail-to-Pass tests, source code for one Fail-to-Pass test, and the execution logs of running all the Fail-to-Pass tests. We ask the LM to generate an issue that describes the bug conveyed in the patch in the style of the SWE-bench Verified demonstration. Figure D shows the system prompt for this strategy.

Fixed issue templates. We create a set of $7$ pre-defined issue templates, listed in Table 13. Each template uses information from the bug patch or Fail-to-Pass tests associated with every task instance. Given a dataset of task instances, we randomly select one of the templates to use as the problem statement according to the probabilities listed in Table 13. The reason we assign the highest likelihood for the prompt that provides all four categories of information (bug type, files changed, functions changed, Fail-to-Pass tests) is to ensure that a higher proportion of task instances are well-specified.

| Template | Prob. | Information Provided |
| --- | --- | --- |
| Basic | $0.05$ | None |
| Files | $0.1$ | States which file(s) have bug(s). |
| Funcs | $0.15$ | States which file(s) and func(s) have bug(s). |
| Tests | $0.1$ | States that some tests are failing. |
| F2P Tests | $0.1$ | States which tests are failing. |
| Bug Type | $0.05$ | States failure type. |
| Bug Type + Files | $0.15$ | States failure type and which file(s) have bug(s) |
| Bug Type + Files | $0.15$ | States failure type, which file(s) have bug(s), |
| + Test |  | and a random F2P test. |
| Bug Type + Files | $0.15$ | States failure type, which file(s) and func(s) |
| + Funcs + Test |  | have bug(s), and a random F2P test. |

Table 13: List of issue text templates we use to generate problem statements. Across all templates, four types of information are included — the files with bugs, functions with bugs, Fail-to-Pass test(s), and the type of bug. Templates that offer less information are generally assigned a lower probability.

Fail-to-Pass test code and execution logs. Another approach is showing the source code and test execution logs for a randomly selected Fail-to-Pass test. This approach is motivated by the lack of reproduction code or expected/actual behavior of code communicated with fixed issue templates. We show code and execution logs only for a single Fail-to-Pass test; if a task instance has more than one Fail-to-Pass test, we do not disclose remaining tests.

Original issue text. This strategy works exclusively for some task instances generated using PR Mirroring. If a PR is successfully mirrored, we use the text from the associated issues as the problem statement, exactly as done in SWE-bench. Of the $2345$ task instances represented in SWE-smith mirrored from real-world PRs, $708$ or $30.19$ % of these have one or more associated GitHub issue(s) to create a SWE-bench style problem statement.

<svg id="A4.p6.pic1" height="656.11" overflow="visible" version="1.1" viewBox="0 0 477.38 656.11" width="477.38"><g style="--ltx-stroke-color:#000000;--ltx-fill-color:#000000;" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,656.11) matrix(1 0 0 -1 0 0)"><g style="--ltx-fill-color:#1F77B4;" fill="#1F77B4" fill-opacity="1.0"><path style="stroke:none" d="M 0 5.91 L 0 650.2 C 0 653.46 2.64 656.11 5.91 656.11 L 471.47 656.11 C 474.73 656.11 477.38 653.46 477.38 650.2 L 477.38 5.91 C 477.38 2.64 474.73 0 471.47 0 L 5.91 0 C 2.64 0 0 2.64 0 5.91 Z"></path></g><g style="--ltx-fill-color:#F4F8FB;" fill="#F4F8FB" fill-opacity="1.0"><path style="stroke:none" d="M 1.97 5.91 L 1.97 632 L 475.41 632 L 475.41 5.91 C 475.41 3.73 473.65 1.97 471.47 1.97 L 5.91 1.97 C 3.73 1.97 1.97 3.73 1.97 5.91 Z"></path></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 640.59)"><foreignObject style="--ltx-fo-width:31.37em;--ltx-fo-height:0.69em;--ltx-fo-depth:0.19em;font-size:10pt;" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 9.61)" width="434.07"><span id="A4.p6.pic1.1" style="width:31.37em;"><span id="A4.p6.pic1.1.1"><span id="A4.p6.pic1.1.1.1" style="--ltx-fg-color:#FFFFFF;">System prompt for generating issues with an LM</span></span> </span></foreignObject></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignObject style="--ltx-fo-width:33.91em;--ltx-fo-height:43.83em;--ltx-fo-depth:0em;font-size:10pt;" height="606.41" overflow="visible" transform="matrix(1 0 0 -1 0 606.41)" width="469.22"><span id="A4.p6.pic1.2" style="width:33.91em;"><span id="A4.p6.pic1.2.1"><span id="A4.p6.pic1.2.1.1" style="font-size:90%;--ltx-fg-color:#000000;">You are a software engineer helping to create a realistic dataset of synthetic GitHub issues.<br></span></span><span id="A4.p6.pic1.2.2"><span id="A4.p6.pic1.2.2.1" style="font-size:90%;--ltx-fg-color:#000000;">You will be given the following input:<br></span></span><span id="A4.p6.pic1.2.3"><span id="A4.p6.pic1.2.3.1" style="font-size:90%;--ltx-fg-color:#000000;">1. Demonstration: A realistic GitHub issue to mimic (included in the <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&lt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&lt;</mo> <annotation encoding="application/x-tex">&lt;</annotation></semantics></math> <span id="A4.p6.pic1.2.3.1.1">demonstration <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&gt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&gt;</mo> <annotation encoding="application/x-tex">&gt;</annotation></semantics></math></span> tag).<br>2. Patch: A git diff output/PR changes that introduces a bug (included in the <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&lt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&lt;</mo> <annotation encoding="application/x-tex">&lt;</annotation></semantics></math> <span id="A4.p6.pic1.2.3.1.2">patch <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&gt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&gt;</mo> <annotation encoding="application/x-tex">&gt;</annotation></semantics></math></span> tag).<br>3. Test output: The output of running the tests after the patch is applied (included in the <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&lt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&lt;</mo> <annotation encoding="application/x-tex">&lt;</annotation></semantics></math> <span id="A4.p6.pic1.2.3.1.3">test_output <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&gt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&gt;</mo> <annotation encoding="application/x-tex">&gt;</annotation></semantics></math></span> tag).<br>4. Test source code: Source code for one or more tests that failed (included in the <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&lt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&lt;</mo> <annotation encoding="application/x-tex">&lt;</annotation></semantics></math> <span id="A4.p6.pic1.2.3.1.4">test_source_code <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&gt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&gt;</mo> <annotation encoding="application/x-tex">&gt;</annotation></semantics></math></span> tag).<br></span></span><span id="A4.p6.pic1.2.4"><span id="A4.p6.pic1.2.4.1" style="font-size:90%;--ltx-fg-color:#000000;">Output: A realistic GitHub issue for the patch.<br></span></span><span id="A4.p6.pic1.2.5"><span id="A4.p6.pic1.2.5.1" style="font-size:90%;--ltx-fg-color:#000000;">Guidelines:<br>- Mimic the style and structure of the demonstration issues. If the demonstration issues are not well structured, your output should also be not well structured. If the demonstrations use improper or no markdown, your output should also use improper or no markdown. If the demonstrations are short/long, your output should also be short/long (if possible). If the demonstrations include human ”flavor text” or ”fluff”, your output should also include human ”flavor text” or ”fluff”. Do this even if it conflicts with your default behavior of trying to be extremely concise and helpful.<br>- DO NOT explain the fix/what caused the bug itself, focus on how to reproduce the issue it introduces<br>- Do not mention pytest or what exact test failed. Instead, generate a realistic issue.<br>- If possible, include information about how to reproduce the issue. An ideal reproduction script should raise an error<br>or print an unexpected output together with the expected output.<br>However, still include this information in a style very similar to the demonstration issues.<br></span></span></span></foreignObject></g></g></svg>

Figure 18: System prompt provided to an LM to generate an issue based off the bug patch and testing information of a task instance along with a demonstration problem statement randomly selected from SWE-bench Verified.

## Appendix E Difficulty Rating

We train a model that labels a task with one of three difficulty labels: $<15$ minutes (easy), $15$ minutes - $1$ hour (medium), and $1$ + hour (hard). This model allows us to quantify the difficulty of individual task instances and, in aggregate, the difficulty of entire datasets.

To train this model, we use $1699$ annotations from [^9]. In their work towards curating SWE-bench Verified, a subset of $1699$ SWE-bench task instances were labeled with four difficulty levels: $<15$ min, $15$ min - $1$ hr, $1$ - $4$ hrs, and $4$ + hrs. Generally, three annotators were assigned to each instance, and the difficulty annotations were ensembled by taking the majority choice for a sample, or the median if there is no majority. The distribution of annotated difficulties, from easiest to hardest, is $24.5$ %, $53.5$ %, $19.4$ %, and $2.8$ %.

Because there are very few samples in the $4$ + hr category, we reclassify the $1$ - $4$ hr and $4$ + hr instances into a single $1$ + hr category. Next, we create corresponding train and test datasets at a $80$ / $20$ % split, randomly shuffling the instances while ensuring the train and test distributions do not deviate significantly from the original. An instance’s problem statement and solution patch are provided as input, and one of the three difficulty labels serves as the target output. We perform LoRA fine-tuning [^13] on a Qwen 2.5 32B Instruct model using the Unsloth [^10] library. The model achieves an accuracy of $75.3$ % on the test set. All errant predictions are off by one; in other words, the model never predicted $<15$ min when the label was $1$ + hr, and vise versa.

Using this model, we can grade the difficulty of a SWE-smith instance once the bug patch and corresponding issue text have been created. To provide a succinct summary of difficulty for a dataset of SWE-bench style task instances, we propose a “difficulty score” metric. Each label corresponds to a numeric difficulty score of $1$, $5$, and $9$, from easiest to hardest. The difficulty score is therefore the average difficulty score across all task instances.

Figure 19: Distribution of task instance difficulty (easy/medium/hard) for existing SWE-bench style datasets (left $5$ bars) and SWE-smith (right $5$ bars), assessed by our difficulty rating model. The average difficulty score for each dataset is listed above each bar. For SWE-smith, per bug strategy, we sample $1000$ task instances with LM generated issue text.

| Dataset | \# Instances | Score | easy | med | hard |
| --- | --- | --- | --- | --- | --- |
| SWE-bench | $2294$ | $5.014$ | $438$ | $1408$ | $446$ |
| Lite | $300$ | $3.893$ | $93$ | $197$ | $10$ |
| Verified | $500$ | $3.960$ | $173$ | $284$ | $43$ |
| SWE-bench Multimodal | $510$ | $6.036$ | $55$ | $265$ | $186$ |
| SWE-gym | $2438$ | $5.625$ | $288$ | $1456$ | $664$ |
| Lite | $230$ | $3.890$ | $67$ | $156$ | $4$ |
| SWE-smith (LM Modify) | $1000$ | $3.304$ | $441$ | $542$ | $17$ |
| SWE-smith (LM Rewrite) | $1000$ | $5.272$ | $68$ | $796$ | $136$ |
| SWE-smith (Procedural) | $1000$ | $3.596$ | $374$ | $603$ | $23$ |
| SWE-smith (PR Mirror) | $1000$ | $4.876$ | $206$ | $619$ | $175$ |
| SWE-smith (Combine) | $1000$ | $5.720$ | $52$ | $716$ | $232$ |

Table 14: The score is averaged over all task instances, where easy/med/hard corresponds to $1$ / $5$ / $9$. For SWE-smith, we sample $1000$ task instances per bug strategy.

Figure 19 summarizes our findings for difficulties across different SWE-bench style datasets. We provide a more thorough rundown of task instances per difficulty level in Table 14. We find that different SWE-smith bug generation methods yield different levels of difficulty. LM Modify are consistently rated to be easy - from several manual spot checks, we notice that while the prompt for LM Modify provides several examples of types of bugs and does not name specific issues to create, the large majority of bugs created by this strategy are simple variable assignment mistakes (e.g. a=a; b=b is changed to a=b; b=a). An open-ended prompt like ours does not actually yield high diversity in terms of mistakes created. Procedural modifications are, as expected, the next easiest, as the types of bugs created by this strategy are finite. PR Mirrors and LM Rewrites yield much harder tasks, confirmed not only by our bug rating model, but also the lower average resolve rate on these tasks by our expert model (SWE-agent + Claude 3.7 Sonnet). Finally, aggregating smaller functions together is a simple but effective strategy for creating bugs that are rated as more complex. This effect aligns with our original expectations; generally, bugs that require editing more functions and files tend to be rated as more difficult. SWE-smith can be used to create task instances with a range of difficulties.

## Appendix F Experiments

In this section, we provide additional details about the configurations and parameters used to generate trajectories with an expert model and run inference on a fine-tuned model. We then provide additional ablations and analyses about the SWE-smith dataset and the agents trained on SWE-smith.

### F.1 Training Details

Rejection sampling fine-tuning. Our fine-tuning setup heavily inherits from [^31] ’s work. We perform full parameter fine tuning using the torchtune [^32] library, with learning rate 5e-5, maximum $3$ epochs, and max context length of $32768$. Training was carried on Modal [^23] on $2$ - $8$ NVIDIA H100 80G GPUs. As discussed in Section 3, the procedure for rejection sampling fine-tuning (RFT) is as follows. We first generate expert demonstrations/trajectories using SWE-agent and a “strong” model (e.g. Claude 3.7 Sonnet, GPT 4o) on SWE-smith task instances. Of these, we then only train a student model on the trajectories corresponding to resolved instances.

SWE-agent configuration. We use two different configurations, one for generating trajectories with an expert model, and a separate one for running inference on the fine-tuned Qwen, student models. The configurations are generally quite similar, with minor differences around how LMs’ responses are elicited, the parsing mechanism for an LM response, constraints around message sizes, and the system prompt.

We will first review the information common to both configurations. The prompt template informing an agent of the task’s nature and problem statement is included in Figure F.1. This prompt is very similar to the original SWE-agent prompt used in [^49]. The prompt templates for showing environment feedback are identical as well. If there is execution output, the text is simply preceded by OBSERVATION: \[output\]. If there is no output (e.g rm -r succeeds silently), then the agent is informed “Your command ran successfully and did not produce any output”. The agent computer interface (ACI) provided is also identical; SWE-agent provides LM with access to three general tools:

- bash: Execute a bash command in terminal.
- str\_replace\_editor: A tool for viewing, creating, and editing files.
- submit: A special keyword for the LM to indicate the task is completed or if it is unable to proceed further with the task.

<svg id="A6.SS1.p4.pic1" height="443.98" overflow="visible" version="1.1" viewBox="0 0 477.38 443.98" width="477.38"><g style="--ltx-stroke-color:#000000;--ltx-fill-color:#000000;" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,443.98) matrix(1 0 0 -1 0 0)"><g style="--ltx-fill-color:#1F77B4;" fill="#1F77B4" fill-opacity="1.0"><path style="stroke:none" d="M 0 5.91 L 0 438.07 C 0 441.33 2.64 443.98 5.91 443.98 L 471.47 443.98 C 474.73 443.98 477.38 441.33 477.38 438.07 L 477.38 5.91 C 477.38 2.64 474.73 0 471.47 0 L 5.91 0 C 2.64 0 0 2.64 0 5.91 Z"></path></g><g style="--ltx-fill-color:#F4F8FB;" fill="#F4F8FB" fill-opacity="1.0"><path style="stroke:none" d="M 1.97 5.91 L 1.97 419.87 L 475.41 419.87 L 475.41 5.91 C 475.41 3.73 473.65 1.97 471.47 1.97 L 5.91 1.97 C 3.73 1.97 1.97 3.73 1.97 5.91 Z"></path></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 428.46)"><foreignObject style="--ltx-fo-width:31.37em;--ltx-fo-height:0.69em;--ltx-fo-depth:0.19em;font-size:10pt;" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 9.61)" width="434.07"><span id="A6.SS1.p4.pic1.1" style="width:31.37em;"><span id="A6.SS1.p4.pic1.1.1"><span id="A6.SS1.p4.pic1.1.1.1" style="--ltx-fg-color:#FFFFFF;">Task Instance Prompt provided to SWE-agent</span></span> </span></foreignObject></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 16.47)"><foreignObject style="--ltx-fo-width:31.37em;--ltx-fo-height:28.3em;--ltx-fo-depth:0.19em;font-size:10pt;" height="394.28" overflow="visible" transform="matrix(1 0 0 -1 0 391.59)" width="434.07"><span id="A6.SS1.p4.pic1.2" style="width:31.37em;"><span id="A6.SS1.p4.pic1.2.1"><math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&lt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&lt;</mo> <annotation encoding="application/x-tex">&lt;</annotation></semantics></math> <span id="A6.SS1.p4.pic1.2.1.1" style="--ltx-fg-color:#000000;">uploaded_files <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&gt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&gt;</mo> <annotation encoding="application/x-tex">&gt;</annotation></semantics></math><br>{{working_dir}}<br><math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&lt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&lt;</mo> <annotation encoding="application/x-tex">&lt;</annotation></semantics></math> /uploaded_files <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&gt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&gt;</mo> <annotation encoding="application/x-tex">&gt;</annotation></semantics></math><br>I’ve uploaded a python code repository in the directory {{working_dir}}. Consider the following PR description:<br></span></span><span id="A6.SS1.p4.pic1.2.2"><math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&lt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&lt;</mo> <annotation encoding="application/x-tex">&lt;</annotation></semantics></math> <span id="A6.SS1.p4.pic1.2.2.1" style="--ltx-fg-color:#000000;">pr_description <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&gt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&gt;</mo> <annotation encoding="application/x-tex">&gt;</annotation></semantics></math><br>{{problem_statement}}<br><math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&lt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&lt;</mo> <annotation encoding="application/x-tex">&lt;</annotation></semantics></math> /pr_description <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&gt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&gt;</mo> <annotation encoding="application/x-tex">&gt;</annotation></semantics></math><br></span></span><span id="A6.SS1.p4.pic1.2.3"><span id="A6.SS1.p4.pic1.2.3.1" style="--ltx-fg-color:#000000;">Can you help me implement the necessary changes to the repository so that the requirements specified in the <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&lt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&lt;</mo> <annotation encoding="application/x-tex">&lt;</annotation></semantics></math> pr_description <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&gt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&gt;</mo> <annotation encoding="application/x-tex">&gt;</annotation></semantics></math> are met? I’ve already taken care of all changes to any of the test files described in the <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&lt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&lt;</mo> <annotation encoding="application/x-tex">&lt;</annotation></semantics></math> pr_description <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&gt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&gt;</mo> <annotation encoding="application/x-tex">&gt;</annotation></semantics></math>. This means you DON’T have to modify the testing logic or any of the tests in any way! Your task is to make the minimal changes to non-tests files in the {{working_dir}} directory to ensure the <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&lt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&lt;</mo> <annotation encoding="application/x-tex">&lt;</annotation></semantics></math> pr_description <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&gt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&gt;</mo> <annotation encoding="application/x-tex">&gt;</annotation></semantics></math> is satisfied. Follow these steps to resolve the issue:<br>1. As a first step, it might be a good idea to find and read code relevant to the <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&lt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&lt;</mo> <annotation encoding="application/x-tex">&lt;</annotation></semantics></math> pr_description <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&gt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&gt;</mo> <annotation encoding="application/x-tex">&gt;</annotation></semantics></math><br>2. Create a script to reproduce the error and execute it with ‘python <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&lt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&lt;</mo> <annotation encoding="application/x-tex">&lt;</annotation></semantics></math> filename.py <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&gt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&gt;</mo> <annotation encoding="application/x-tex">&gt;</annotation></semantics></math> ‘ using the bash tool, to confirm the error<br>3. Edit the source code of the repo to resolve the issue<br>4. Rerun your reproduce script and confirm that the error is fixed!<br>5. Think about edgecases and make sure your fix handles them as well Your thinking should be thorough and so it’s fine if it’s very long.</span></span></span></foreignObject></g></g></svg>

Figure 20: A copy of the prompt provided to an LM via SWE-agent informing the LM of the nature of the task, the task description itself, and several tips on how to proceed.

We briefly review the distinctions. First, tool invocation works differently for expert versus student models. For the Claude and GPT series models that are used as experts, we use function calling for models to invoke the aforementioned tools. On the other hand, the student model is asked to generate a response with XML tags to delineate the thought and action. Therefore, when fine-tuning on expert trajectories, a key processing step is to convert the expert trajectories’ function calling format into the XML style response — fine-tuning directly on the expert trajectories does not work.

We note that we use these particular settings because as of the publication of this paper, this tool setting reflects the absolute state-of-the-art performance achieved with an open source agent system (SWE-agent) and any existing LM (Claude 3.7 Sonnet). It is certainly possible to explore more tool designs and experiment with different formatting calls, as many existing prior works, notably [^49], have performed. However, given the focus of our work, we do not bother with repeating such a ”hyperparameter sweep” across configurations for the agent system, as this effort is expensive and has already been performed to suggest that the configuration we are using is ideal for expert level performance.

For generating trajectories with expert models, we run with a maximum of $75$ steps and a cost limit of $ $2.00$. A run terminates automatically when either of these limits are reached or the context window of the expert model is exceeded. The overwhelming majority of automatic terminations are due to the $75$ maximum steps limit.

For running inference with student models, we run with a maximum of $75$ steps or a cost limit <sup>3</sup> of $ $2.00$, where the run similarly terminates when either the steps, cost or context window limit is reached. For the student model, per LM inference call, we truncate the message history to only keep the $5$ most recent tool outputs. While we occasionally sample trajectories with the expert model set at various temperatures, for the student model, the temperature is fixed at $0.0$.

### F.2 Evaluation Datasets

SWE-bench. SWE-bench is a widely used benchmark that evaluates AI systems on their ability to resolve GitHub issues [^18]. Given a codebase along with a description of a bug or feature, the AI system is asked to modify the codebase in such a way that the issue presented in the description is resolved. SWE-bench consists of $2294$ such task instances, collected from real world pull requests (PRs) and issues in $12$ GitHub repositories that are predominantly Python. As discussed in Section 3, the Lite and Verified subsets are curated from the main SWE-bench repository with the goal of making evaluation either more efficent or more reliable. Since evaluation on the entirety of SWE-bench is fairly costly and does not have as many comparable references, we do not evaluate SWE-agent-LM-32B on the entire SWE-bench test set.

SWE-bench Multimodal. SWE-bench Multimodal applies SWE-bench collection strategy to $12$ additional predominantly JavaScript and TypeScript GitHub repositories, where task instances are associated with issues that have visual asset(s) in them [^50]. The evaluation dataset consists of $510$ task instances. While the original work evaluates vision language models (VLMs) specifically, we do not evaluate SWE-agent-LM-32B which, as it is based on Qwen $2.5$ Coder Instruct, does not have the ability to process images as inputs.

| jqlang/jq | 9 |
| --- | --- |
| redis/redis | 12 |
| micropython/micropython | 5 |
| valkey-io/valkey | 4 |
| nlohmann/json | 1 |
| fmtlib/fmt | 11 |
| C/C++ | 42 |
| prometheus/prometheus | 8 |
| caddyserver/caddy | 14 |
| gin-gonic/gin | 8 |
| hashicorp/terraform | 5 |
| gohugoio/hugo | 7 |
| Go | 42 |
| briannesbitt/carbon | 10 |
| laravel/framework | 13 |
| phpoffice/phpspreadsheet | 10 |
| php-cs-fixer/php-cs-fixer | 10 |
| PHP | 43 |

| apache/druid | 5 |
| --- | --- |
| reactivex/rxjava | 1 |
| apache/lucene | 9 |
| projectlombok/lombok | 17 |
| google/gson | 9 |
| javaparser/javaparser | 2 |
| Java | 43 |
| babel/babel | 5 |
| mrdoob/three.js | 3 |
| vuejs/core | 5 |
| preactjs/preact | 17 |
| axios/axios | 6 |
| immutable-js/immutable-js | 2 |
| facebook/docusaurus | 5 |
| JS/TS | 43 |

| rubocop/rubocop | 16 |
| --- | --- |
| jekyll/jekyll | 5 |
| faker-ruby/faker | 2 |
| fastlane/fastlane | 7 |
| fluent/fluentd | 12 |
| jordansissel/fpm | 2 |
| Ruby | 44 |
| tokio-rs/axum | 7 |
| nushell/nushell | 5 |
| sharkdp/bat | 8 |
| burntsushi/ripgrep | 2 |
| uutils/coreutils | 5 |
| tokio-rs/tokio | 9 |
| astral-sh/ruff | 7 |
| Rust | 43 |

Table 15: Number of task instances per repository and language in the SWE-bench Multilingual evaluation set. The entire dataset includes $300$ task instances covering $9$ languages.

SWE-bench Multilingual. SWE-bench Multilingual is an evaluation dataset consisting of 300 task instances that we introduce with this work. A single author carried out SWE-bench’s collection strategy for $42$ additional GitHub repositories, covering the following $9$ programming languages: JavaScript, TypeScript, C, C++, Go, Java, PHP, Ruby, and Rust. These repositories span a wide range of application domains, including web frameworks, data storage and processing tools, core utilities, and widely used libraries. A brief summary of the dataset is presented in Table 15.

Like SWE-bench Verified, we curate the dataset by excluding task instances deemed by a team of three authors to have ambiguous or underspecified issue text. Each task instance edits (meaning additions and removals) on average $48$ lines of code. Similar to SWE-bench and SWE-smith, the median number of Fail-to-Pass tests is one.

We introduce SWE-bench Multilingual to:

1. Provide a benchmark to evaluate model and agent performance across a variety of programming languages and application domains. Existing agent systems often rely on Python-specific tooling, effectively overfitting to the original SWE-bench [^50]. Although SWE-bench Multimodal addresses this to some degree, its focus on visual inputs is a confounding factor for text-only evaluation of software engineering capabilities.
2. Remain fully compatible with SWE-bench, so current users can adopt it without changing infrastructure.
3. Keep the dataset small enough to run quickly. While concurrent work like [^55] provides more task instances in multiple languages, we purposely constrain the number of task instances so that the dataset is easy to run quickly.

In §F.4, we briefly discuss how performance by existing state of the art methods for SWE-bench is markedly worse on SWE-bench Multilingual, then offer some clear directions for potential next steps to build better agentic coding models that would involve extending SWE-smith.

### F.3 Trajectory Dataset Breakdown

<table><tbody><tr><th>Purpose</th><th>Bug Gen.</th><th>Issue Gen.</th><td># Instances</td><td>Temp.</td><td># Traj.</td></tr><tr><th colspan="6">claude-3-7-sonnet-20250219</th></tr><tr><th>Ablation</th><th>LM (Modify)</th><th>LM</th><td>1000</td><td>0</td><td>605</td></tr><tr><th>(Bug Type)</th><th>LM (Rewrite)</th><th>LM</th><td>1000</td><td>0</td><td>507</td></tr><tr><th></th><th>Procedural</th><th>LM</th><td>1000</td><td>0</td><td>745</td></tr><tr><th></th><th>PR Mirrors</th><th>LM</th><td>1000</td><td>0</td><td>557</td></tr><tr><th>Ablation</th><th>PR Mirrors</th><th>Fixed</th><td>600</td><td>0</td><td>259</td></tr><tr><th>(Issue Type)</th><th>PR Mirrors</th><th>F2P Test</th><td>600</td><td>0</td><td>390</td></tr><tr><th></th><th>PR Mirrors</th><th>Original</th><td>600</td><td>0</td><td>328</td></tr><tr><th></th><th>PR Mirrors</th><th>LM</th><td>600</td><td>0</td><td>319</td></tr><tr><th>Ablation</th><th>Procedural</th><th>LM</th><td>1000</td><td>0</td><td>721</td></tr><tr><th>(Repositories)</th><th>Procedural</th><th>LM</th><td>1000</td><td>0</td><td>709</td></tr><tr><th></th><th>Procedural</th><th>LM</th><td>1000</td><td>0</td><td>723</td></tr><tr><th></th><th>Procedural</th><th>LM</th><td>1000</td><td>0</td><td>707</td></tr><tr><th>Final Dataset</th><th>LM (Rewrite)</th><th>LM</th><td>3574</td><td>0</td><td>1003</td></tr><tr><th>Curation</th><th>PR Mirrors</th><th>LM</th><td>1049</td><td>0</td><td>349</td></tr><tr><th colspan="6">claude-3-5-sonnet-20250219</th></tr><tr><th>Compare with prior work</th><th>All</th><th>LM</th><td>800</td><td>0</td><td>535</td></tr><tr><th colspan="6">gpt-4o-2024-08-06</th></tr><tr><th>Compare with prior work</th><th>All</th><th>LM</th><td>200</td><td>0</td><td>89</td></tr></tbody></table>

Table 16: Breakdown of trajectories sampled from SWE-smith. Trajectories were generated from subsets of SWE-smith that were either for the purpose of ablations or performance. All trajectories were generated with a maximum of $75$ steps and a $ $2$ cost limit.

| Bug Type | Count |
| --- | --- |
| Combine (File) | 123 |
| Combine (Module) | 7 |
| LM (Modify) | 11 |
| LM (Rewrite) | 1532 |
| Procedural | 1495 |
| PR Mirror | 1848 |

Table 17: Bug types represented in final training dataset.

| Repository | Count | Repository | Count |
| --- | --- | --- | --- |
| getmoto/moto | $378$ | sqlfluff/sqlfluff | $122$ |
| pandas-dev/pandas | $320$ | pylint-dev/astroid | $110$ |
| conan-io/conan | $243$ | pydicom/pydicom | $103$ |
| pydantic/pydantic | $209$ | tobymao/sqlglot | $101$ |
| iterative/dvc | $181$ | pygments/pygments | $99$ |
| dask/dask | $139$ | scanny/python-pptx | $98$ |

Table 18: Top ten repositories by number of trajectories represented in final dataset for main result.

We provide a thorough review of the dataset of SWE-agent trajectories released with this work in Table 16. The majority are generated with claude-3-7-sonnet-20250219. To compare with prior work, a minority were generated with claude-3-5-sonnet-20240620 and gpt-4o-2024-08-06. As mentioned in Section 4, to guard against the easy data bias phenomenon, we impose a per-instance cap of $3$, meaning for any task instance, we include at most $3$ trajectories successfully resolving that task instance in our fine-tuning dataset. From the pool of trajectories reflected in Table 16, we curate a set of $5000$ trajectories that we then use to train SWE-agent-LM-32B.

Tables 18 and 18 show what repositories and bug types are represented in the final training dataset. In total, $123$ repositories are represented, with at least $10$ trajectories from $91$ repositories. Trajectories are on average $58$ turns long, meaning an LM typically takes $29$ actions for a given demonstration trajectory. We visualize this distribution in Figure 21.

Figure 21: Distribution of number of turns for trajectories represented in the final dataset.

### F.4 Training Analyses

We provide additional experiments and discussions around training SWE-agent-LM-32B.

Pass@k trend line. To calculate the Pass@1 score discussed in our main result, we ran SWE-agent with SWE-agent-LM-32B six times. In Figure 23, we observe increasing performance at higher values of k, a phenomenon that reflects observations in prior works across LMs for software engineering, code generation, web navigation, and theorem proving. While we do not explore work around inference time scaling and training a separate verifier model to select the best solution candidate generated by multiple roll-outs, as done in [^31] and [^16], SWE-agent-LM-32B is fully compatible with the generate-then-select pipelines explored by such works. Given its strong Pass@1 performance, SWE-agent-LM-32B would likely be quite competitive for Best@k results as well. As mentioned before, all trajectories generated in the course of SWE-smith have been released publicly, which the community might find useful for training better verifiers.

Rejection sampling fine-tuning ablation. To confirm that rejection sampling fine-tuning leads to better performance on the downstream task, we compare against a setting where we randomly sample n training points with no filtering criteria, at n = \[100, 200, 400, 800, 1600\] and fine-tune the same student model (Qwen $2.5$ Coder Instruct $32$ B. We then run SWE-agent with each student model on the SWE-bench Verified dataset three times, with the “% Resolved” corresponding to the Pass@1 score. We show results in Figure 23, which confirms that fine-tuning only on trajectories corresponding to successfully resolved tasks is better than randomly sampling trajectories.

SWE-bench Multilingual performance. To assess how well SWE-agent-LM-32B and existing models generalize to non-Python coding domains, we evaluate the performance of our model, Qwen $2.5$ Coder Instruct 32B, and Claude $3.7$ Sonnet with SWE-agent on our new dataset, which we introduced in Section F.2. Out of $300$ task instances, we found that Claude $3.7$ Sonnet achieved a $43$ % Pass@1 resolve rate, which is significantly better than SWE-agent-LM-32B ($8.4$ %) and Qwen $2.5$ Coder Instruct ($6.5$ %). SWE-agent-LM-32B does not demonstrate a significant improvement over the baseline model. Through several spot checks of different trajectories, we came to a working hypothesis that while the rejection sampling fine-tuning process had improved its ability to carry out multi-turn interactions in this task setting, there were instances where code edits reflected syntax closer to Python despite code and files viewed in previous steps clearly not being written in Python.

While the result for SWE-agent-LM-32B SWE-bench Multilingual is clearly subpar, we are excited by such a finding, as it motivates future work on top of SWE-smith. To elaborate, we expect that the path to open agent coding models capable of generalizing to many repositories and languages will be paved by more data and better training techniques, both of which SWE-smith is very capable of facilitating. First, regarding data, although we wrote SWE-smith to be Python centric, the collection methodology and bug generation techniques (especially LM based methods) should be readily transferable to other repositories. Second, the negative result on SWE-bench Multilingual provides a clear impetus for exploring whether better training techniques could lead to models that are trained on one code domain (e.g., Python), but can generalize to many languages and repositories.

Figure 22: SWE-agent-LM-32B Pass@k curve on SWE-bench Verified. We observe higher % resolved when considering more runs.

Figure 23: Rejection sampling fine-tuning leads to better performance than random sampling of trajectories for training.

### F.5 Agent Behavioral Studies

#### F.5.1 Turn counts and cost

While agents are frequently quoted with a singular cost-per-instance number, this can be very misleading in the case of SWE-agent-LM-32B. Because most of the failed instances fail due to termination by the cost or turn count limit, the average cost and turn counts depend strongly on these limits (see Fig. 25).

We can also chart the number of resolved instances vs step limits. To avoid reevaluating the agent with multiple step limits, we use one run with step limit 75 and then assume that a successful agent run that terminates after step $n$ would have failed when restricted by a limit smaller than $n$. This chart corroborates the point made in section 3: SWE-agent-LM-32B has a higher resolution rate for very low step limits.

Figure 24: The average step count depends strongly on the prescribed step limit.

Figure 25: Number of successful instances submitted before a given step limit.

#### F.5.2 Analysis of agent action space

Reduction to *base commands.* In addition to the dedicated tools provided to the agent as part of the agent computer interface (Section F.1), the agent can execute arbitrary bash commands. This makes quantitative analyses of the agent action space challenging. For example, the agent might issue commands like PYTHONPATH=/testbed/repo cd /testbed/repo && python3 reproduce.py. We have found the following procedure to determine a *base command* effective to meaningfully describe the action:

1. Strip any environment variable manipulation from the beginning of the command.
2. When multiple commands are chained with && or semicolons, only consider the last command.
3. Remove all arguments. Because some commands have subcommands (e.g., git checkout), we apply several basic heuristics to determine whether to keep the first or the first two words.

Repetitive actions. We determine the longest repetitive sequence of actions by determining the longest sequence of identical base commands within the agent actions. Note that this means that e.g., str\_replace\_editor view actions that target different files are considered to be repetitive actions as far as this analysis is concerned.

#### F.5.3 Failure mode analysis

Categorizing the failure mode proceeds as shown in Figure 26:

1. Error conditions: If the agent terminates due to an error (environment errors, inability of the LM to correctly format its messages, etc.) or because it exceeded its maximum context window, we return the error or context category.
2. Early termination: If the agent was terminated because of a step or cost limit, we return one of the stuck … subcategories. Note that the SWE-agent still attempts to extract a submission (list of changes/patch). We determine the subcategory based on which part of the workflow agentic loop was terminated:
	1. If no source (i.e., non-test) file was modified <sup>4</sup> and no attempt at testing was made, we return stuck at localization. If test commands were run (i.e., python, pytest, …, or similar commands), we return stuck at reproduction.
	2. If source files *were* modified, we check whether the changes include changes to all source files that are modified in the gold patch. If not, we return incorrect localization (stuck), else incorrect edit (stuck).
3. Successful submission: If the agent terminated and submitted a solution naturally, we return incorrect localization or incorrect edit, depending on whether the changes from the submitted patch included changes to all files from the SWE-bench gold patch.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2504.21798/assets/failure_analysis_flowchart_vert.png)

Figure 26: Categorizing failure modes

#### F.5.4 Mitigating repetitive actions

As described in section 4, SWE-agent-LM-32B frequently shows highly repetitive actions for unresolved instances. In light of this, it seems promising to investigate whether agent scaffolding interventions can be used to mitigate the problem and increase the success rates.

We make the following modification to the agent scaffold:

- We add warning messages to the observation (command output) if a base command is repeated four (str\_replace\_editor view) or six (any other base command) times. The warning message advises to try different commands, and in particular suggest to locate relevant context using find or grep.
- If the warning messages do not break the string of repetitive base commands and the repetition length reaches 6 (str\_replace\_editor view) or 8 (any other base command), every following action is resampled up to 10 times, stopping at the first base command that is distinct from the previous ones. To further increase the likelihood of breaking the cycle, we inject assistant messages or raise the temperature if the repetition length reaches 7 or 9.

This effectively reduces the number of repetitive actions (see Fig. 27). However, the overall number of resolved instances drops slightly to 192 ($38.4\%$). Variations of the above strategies yield similar outcomes: while repetition is suppressed, success rates do not improve substantially. This may suggest that repetitive actions are better understood as *symptoms* of the model’s difficulty in solving an instance (such as when the instance is out-of-distribution or particularly challenging) rather than constituting intrinsic failure modes.

Figure 27: Scaffold interventions can drastically reduce the number of repetitive actions.

## Appendix G Miscellaneous

Teaser figure description. We briefly describe how the left hand graph of Figure 1, which depicts scaling of task instance collection for the SWE-smith vs. SWE-bench, was created. For SWE-smith, we simply collected the number of task instances for each repository. For SWE-bench, we ran the SWE-bench task instance candidate collection script on all $128$ repositories, which first crawls all PRs from a given repository. Then, each PR that edits at least one or more Python files and changes at least one or more testing related files is converted into a candidate task instance. Finally, based on the average task instance yield rate reported in [^18], we estimate the number of viable task instances to be $20$ % of the candidates. We then determine the number of task instances for n repositories at intervals of $5$ repositories ranging from $5$ to $250$, where the repositories are sorted by number of stars. In other words, the first five repositories we account for in the figure are the five with the fewest number of stars out of the $128$ repositories used.

Extended related works. We discuss additional related works briefly, primarily about similar work towards synthesizing trajectories for training LM agents, but for the domain of web tasks. To improve the interactive capabilities of open source LMs [^7], prior works have also explored trajectory generation techniques for web benchmarks and settings [^45] [^51] [^59]. For web navigation, existing strategies rely on (1) performing random walks which are then labeled retroactively with instructions [^43] [^25], (2) using online web tutorials as a source of indirect supervision for generating synthetic trajectories [^30], or (3) collecting human demonstrations [^34] [^47]. These procedures do not translate well to the software engineering setting; random sequences of command line interactions usually do not achieve meaningful effects on a codebase. Our cursory efforts around replaying trajectories synthesized from online code edit sequences (e.g. GitHub commit histories) were unsuccessful due to the limited information available, which primarily capture file-level changes without reflecting the underlying skills, decision-making, or the broader context of a software development process.

Our exploration of using SWE-agent to automatically determine installation and testing specifications for a repository is heavily influenced by two research directions - automatic execution environment construction using LMs [^6] [^11] [^37], and generating unit tests using LMs [^26]. Although relatively much less than SWE-bench style collection, SWE-smith still requires minimal amounts of human labor (around $8$ minutes total per repository). As we expand SWE-smith to more repositories and languages, we are continuing to consider how to completely automate the environment construction process end to end.

[^1]: Anthropic. Introducing claude 3.5 sonnet, 2024. URL [https://www.anthropic.com/news/claude-3-5-sonnet](https://www.anthropic.com/news/claude-3-5-sonnet).

[^2]: Anthropic. Introducing claude 3.7 sonnet, 2025. URL [https://www.anthropic.com/news/claude-3-7-sonnet](https://www.anthropic.com/news/claude-3-7-sonnet).

[^3]: Antonis Antoniades, Albert Örwall, Kexun Zhang, Yuxi Xie, Anirudh Goyal, and William Wang. SWE-Search: Enhancing Software Agents with Monte Carlo Tree Search and Iterative Refinement, December 2024. URL [http://arxiv.org/abs/2410.20285](http://arxiv.org/abs/2410.20285). arXiv:2410.20285 \[cs\].

[^4]: Jacob Austin, Augustus Odena, Maxwell Nye, Maarten Bosma, Henryk Michalewski, David Dohan, Ellen Jiang, Carrie Cai, Michael Terry, Quoc Le, and Charles Sutton. Program Synthesis with Large Language Models, August 2021. URL [http://arxiv.org/abs/2108.07732](http://arxiv.org/abs/2108.07732). arXiv:2108.07732 \[cs\].

[^5]: Ibragim Badertdinov, Maria Trofimova, Yuri Anapolskiy, Sergey Abramov, Karina Zainullina, Alexander Golubev, Sergey Polezhaev, Daria Litvintseva, Simon Karasik, Filipp Fisin, Sergey Skvortsov, Maxim Nekrashevich, Anton Shevtsov, and Boris Yangel. Scaling data collection for training software engineering agents. *Nebius blog*, 2024.

[^6]: Ben Bogin, Kejuan Yang, Shashank Gupta, Kyle Richardson, Erin Bransom, Peter Clark, Ashish Sabharwal, and Tushar Khot. Super: Evaluating agents on setting up and executing tasks from research repositories, 2024. URL [https://arxiv.org/abs/2409.07440](https://arxiv.org/abs/2409.07440).

[^7]: Baian Chen, Chang Shu, Ehsan Shareghi, Nigel Collier, Karthik Narasimhan, and Shunyu Yao. FireAct: Toward Language Agent Fine-tuning, October 2023. URL [http://arxiv.org/abs/2310.05915](http://arxiv.org/abs/2310.05915). arXiv:2310.05915 \[cs\].

[^8]: Mark Chen, Jerry Tworek, Heewoo Jun, Qiming Yuan, Henrique Ponde de Oliveira Pinto, Jared Kaplan, Harri Edwards, Yuri Burda, Nicholas Joseph, Greg Brockman, Alex Ray, Raul Puri, Gretchen Krueger, Michael Petrov, Heidy Khlaaf, Girish Sastry, Pamela Mishkin, Brooke Chan, Scott Gray, Nick Ryder, Mikhail Pavlov, Alethea Power, Lukasz Kaiser, Mohammad Bavarian, Clemens Winter, Philippe Tillet, Felipe Petroski Such, Dave Cummings, Matthias Plappert, Fotios Chantzis, Elizabeth Barnes, Ariel Herbert-Voss, William Hebgen Guss, Alex Nichol, Alex Paino, Nikolas Tezak, Jie Tang, Igor Babuschkin, Suchir Balaji, Shantanu Jain, William Saunders, Christopher Hesse, Andrew N. Carr, Jan Leike, Josh Achiam, Vedant Misra, Evan Morikawa, Alec Radford, Matthew Knight, Miles Brundage, Mira Murati, Katie Mayer, Peter Welinder, Bob McGrew, Dario Amodei, Sam McCandlish, Ilya Sutskever, and Wojciech Zaremba. Evaluating Large Language Models Trained on Code, July 2021. URL [http://arxiv.org/abs/2107.03374](http://arxiv.org/abs/2107.03374). arXiv:2107.03374 \[cs\].

[^9]: Neil Chowdhury, James Aung, Chan Jun Shern, Oliver Jaffe, Dane Sherburn, Giulio Starace, Evan Mays, Rachel Dias, Marwan Aljubeh, Mia Glaese, et al. Introducing swe-bench verified, 2024. *URL https://openai.com/index/introducing-swe-bench-verified*, 2024.

[^10]: Michael Han Daniel Han and Unsloth team. Unsloth, 2023. URL [http://github.com/unslothai/unsloth](http://github.com/unslothai/unsloth).

[^11]: Aleksandra Eliseeva, Alexander Kovrigin, Ilia Kholkin, Egor Bogomolov, and Yaroslav Zharov. Envbench: A benchmark for automated environment setup, 2025. URL [https://arxiv.org/abs/2503.14443](https://arxiv.org/abs/2503.14443).

[^12]: Patrick Haluptzok, Matthew Bowers, and Adam Tauman Kalai. Language models can teach themselves to program better, 2023. URL [https://arxiv.org/abs/2207.14502](https://arxiv.org/abs/2207.14502).

[^13]: Edward J. Hu, Yelong Shen, Phillip Wallis, Zeyuan Allen-Zhu, Yuanzhi Li, Shean Wang, Lu Wang, and Weizhu Chen. Lora: Low-rank adaptation of large language models, 2021. URL [https://arxiv.org/abs/2106.09685](https://arxiv.org/abs/2106.09685).

[^14]: Binyuan Hui, Jian Yang, Zeyu Cui, Jiaxi Yang, Dayiheng Liu, Lei Zhang, Tianyu Liu, Jiajun Zhang, Bowen Yu, Keming Lu, Kai Dang, Yang Fan, Yichang Zhang, An Yang, Rui Men, Fei Huang, Bo Zheng, Yibo Miao, Shanghaoran Quan, Yunlong Feng, Xingzhang Ren, Xuancheng Ren, Jingren Zhou, and Junyang Lin. Qwen2.5-coder technical report, 2024. URL [https://arxiv.org/abs/2409.12186](https://arxiv.org/abs/2409.12186).

[^15]: Naman Jain, Manish Shetty, Tianjun Zhang, King Han, Koushik Sen, and Ion Stoica. R2e: Turning any github repository into a programming agent environment. In *ICML 2024*, 2024.

[^16]: Naman Jain, Jaskirat Singh, Manish Shetty, Liang Zheng, Koushik Sen, and Ion Stoica. R2e-gym: Procedural environments and hybrid verifiers for scaling open-weights swe agents, 2025. URL [https://arxiv.org/abs/2504.07164](https://arxiv.org/abs/2504.07164).

[^17]: Carlos E. Jimenez, John Yang, Alexander Wettig, Shunyu Yao, Kexin Pei, Ofir Press, and Karthik Narasimhan. Swe-bench leaderboard, 2024a. URL [https://swe-bench.github.io/](https://swe-bench.github.io/).

[^18]: Carlos E. Jimenez, John Yang, Alexander Wettig, Shunyu Yao, Kexin Pei, Ofir Press, and Karthik Narasimhan. SWE-bench: Can Language Models Resolve Real-World GitHub Issues?, November 2024b. URL [http://arxiv.org/abs/2310.06770](http://arxiv.org/abs/2310.06770). arXiv:2310.06770 \[cs\].

[^19]: Jiawei Liu, Thanh Nguyen, Mingyue Shang, Hantian Ding, Xiaopeng Li, Yu Yu, Varun Kumar, and Zijian Wang. Learning code preference via synthetic evolution, 2024a. URL [https://arxiv.org/abs/2410.03837](https://arxiv.org/abs/2410.03837).

[^20]: Zhihan Liu, Shenao Zhang, Yongfei Liu, Boyi Liu, Yingxiang Yang, and Zhaoran Wang. DSTC: Direct Preference Learning with Only Self-Generated Tests and Code to Improve Code LMs, December 2024b. URL [http://arxiv.org/abs/2411.13611](http://arxiv.org/abs/2411.13611). arXiv:2411.13611 \[cs\].

[^21]: Ziyang Luo, Can Xu, Pu Zhao, Qingfeng Sun, Xiubo Geng, Wenxiang Hu, Chongyang Tao, Jing Ma, Qingwei Lin, and Daxin Jiang. Wizardcoder: Empowering code large language models with evol-instruct, 2023. URL [https://arxiv.org/abs/2306.08568](https://arxiv.org/abs/2306.08568).

[^22]: Yingwei Ma, Rongyu Cao, Yongchang Cao, Yue Zhang, Jue Chen, Yibo Liu, Yuchen Liu, Binhua Li, Fei Huang, and Yongbin Li. Lingma swe-gpt: An open development-process-centric language model for automated software improvement, 2024. URL [https://arxiv.org/abs/2411.00622](https://arxiv.org/abs/2411.00622).

[^23]: Modal. Modal: High-performance ai infrastructure, 2025. URL [https://modal.com/](https://modal.com/).

[^24]: Niklas Muennighoff, Qian Liu, Armel Zebaze, Qinkai Zheng, Binyuan Hui, Terry Yue Zhuo, Swayam Singh, Xiangru Tang, Leandro von Werra, and Shayne Longpre. OctoPack: Instruction Tuning Code Large Language Models, February 2024. URL [http://arxiv.org/abs/2308.07124](http://arxiv.org/abs/2308.07124). arXiv:2308.07124 \[cs\].

[^25]: Shikhar Murty, Dzmitry Bahdanau, and Christopher D. Manning. NNetscape Navigator: Complex Demonstrations for Web Agents Without a Demonstrator, October 2024. URL [http://arxiv.org/abs/2410.02907](http://arxiv.org/abs/2410.02907). arXiv:2410.02907 \[cs\].

[^26]: Niels Mündler, Mark Niklas Müller, Jingxuan He, and Martin Vechev. Swt-bench: Testing and validating real-world bug-fixes with code agents, 2025. URL [https://arxiv.org/abs/2406.12952](https://arxiv.org/abs/2406.12952).

[^27]: OpenAI. Gpt-4o system card, 2024a. URL [https://arxiv.org/abs/2410.21276](https://arxiv.org/abs/2410.21276).

[^28]: OpenAI. Openai o3-mini system card, 2024b. URL [https://cdn.openai.com/o3-mini-system-card-feb10.pdf](https://cdn.openai.com/o3-mini-system-card-feb10.pdf).

[^29]: Albert Orwall. Moatless tools, 2024. URL [https://github.com/aorwall/moatless-tools](https://github.com/aorwall/moatless-tools).

[^30]: Tianyue Ou, Frank F. Xu, Aman Madaan, Jiarui Liu, Robert Lo, Abishek Sridhar, Sudipta Sengupta, Dan Roth, Graham Neubig, and Shuyan Zhou. Synatra: Turning Indirect Knowledge into Direct Demonstrations for Digital Agents at Scale, September 2024. URL [http://arxiv.org/abs/2409.15637](http://arxiv.org/abs/2409.15637). arXiv:2409.15637 \[cs\].

[^31]: Jiayi Pan, Xingyao Wang, Graham Neubig, Navdeep Jaitly, Heng Ji, Alane Suhr, and Yizhe Zhang. Training Software Engineering Agents and Verifiers with SWE-Gym, December 2024. URL [http://arxiv.org/abs/2412.21139](http://arxiv.org/abs/2412.21139). arXiv:2412.21139 \[cs\].

[^32]: PyTorch. torchtune: Pytorch’s finetuning library, April 2024. URL [https//github.com/pytorch/torchtune](https://ar5iv.labs.arxiv.org/html/https//github.com/pytorch/torchtune).

[^33]: Qwen, An Yang, Baosong Yang, Beichen Zhang, Binyuan Hui, Bo Zheng, Bowen Yu, Chengyuan Li, Dayiheng Liu, Fei Huang, Haoran Wei, Huan Lin, Jian Yang, Jianhong Tu, Jianwei Zhang, Jianxin Yang, Jiaxi Yang, Jingren Zhou, Junyang Lin, Kai Dang, Keming Lu, Keqin Bao, Kexin Yang, Le Yu, Mei Li, Mingfeng Xue, Pei Zhang, Qin Zhu, Rui Men, Runji Lin, Tianhao Li, Tianyi Tang, Tingyu Xia, Xingzhang Ren, Xuancheng Ren, Yang Fan, Yang Su, Yichang Zhang, Yu Wan, Yuqiong Liu, Zeyu Cui, Zhenru Zhang, and Zihan Qiu. Qwen2.5 Technical Report, January 2025. URL [http://arxiv.org/abs/2412.15115](http://arxiv.org/abs/2412.15115). arXiv:2412.15115 \[cs\].

[^34]: Junhong Shen, Atishay Jain, Zedian Xiao, Ishan Amlekar, Mouad Hadji, Aaron Podolny, and Ameet Talwalkar. Scribeagent: Towards specialized web agents using production-scale workflow data, 2024. URL [https://arxiv.org/abs/2411.15004](https://arxiv.org/abs/2411.15004).

[^35]: Alexander Shypula, Aman Madaan, Yimeng Zeng, Uri Alon, Jacob Gardner, Milad Hashemi, Graham Neubig, Parthasarathy Ranganathan, Osbert Bastani, and Amir Yazdanbakhsh. Learning performance-improving code edits, 2024. URL [https://arxiv.org/abs/2302.07867](https://arxiv.org/abs/2302.07867).

[^36]: Theodore R. Sumers, Shunyu Yao, Karthik Narasimhan, and Thomas L. Griffiths. Cognitive architectures for language agents, 2024. URL [https://arxiv.org/abs/2309.02427](https://arxiv.org/abs/2309.02427).

[^37]: Konstantinos Vergopoulos, Mark Niklas Müller, and Martin Vechev. Automated benchmark generation for repository-level coding tasks, 2025. URL [https://arxiv.org/abs/2503.07701](https://arxiv.org/abs/2503.07701).

[^38]: Xingyao Wang, Boxuan Li, Yufan Song, Frank F. Xu, Xiangru Tang, Mingchen Zhuge, Jiayi Pan, Yueqi Song, Bowen Li, Jaskirat Singh, Hoang H. Tran, Fuqiang Li, Ren Ma, Mingzhang Zheng, Bill Qian, Yanjun Shao, Niklas Muennighoff, Yizhe Zhang, Binyuan Hui, Junyang Lin, Robert Brennan, Hao Peng, Heng Ji, and Graham Neubig. OpenHands: An Open Platform for AI Software Developers as Generalist Agents, October 2024. URL [http://arxiv.org/abs/2407.16741](http://arxiv.org/abs/2407.16741). arXiv:2407.16741 \[cs\].

[^39]: Yuxiang Wei, Federico Cassano, Jiawei Liu, Yifeng Ding, Naman Jain, Zachary Mueller, Harm de Vries, Leandro von Werra, Arjun Guha, and Lingming Zhang. SelfCodeAlign: Self-Alignment for Code Generation, November 2024a. URL [http://arxiv.org/abs/2410.24198](http://arxiv.org/abs/2410.24198). arXiv:2410.24198 \[cs\].

[^40]: Yuxiang Wei, Zhe Wang, Jiawei Liu, Yifeng Ding, and Lingming Zhang. Magicoder: Empowering code generation with oss-instruct, 2024b. URL [https://arxiv.org/abs/2312.02120](https://arxiv.org/abs/2312.02120).

[^41]: Yuxiang Wei, Olivier Duchenne, Jade Copet, Quentin Carbonneaux, Lingming Zhang, Daniel Fried, Gabriel Synnaeve, Rishabh Singh, and Sida I. Wang. SWE-RL: Advancing llm reasoning via reinforcement learning on open software evolution, 2025. URL [https://arxiv.org/abs/2502.18449](https://arxiv.org/abs/2502.18449).

[^42]: Chunqiu Steven Xia, Yinlin Deng, Soren Dunn, and Lingming Zhang. Agentless: Demystifying LLM-based Software Engineering Agents, October 2024. URL [http://arxiv.org/abs/2407.01489](http://arxiv.org/abs/2407.01489). arXiv:2407.01489 \[cs\].

[^43]: Jiannan Xiang, Tianhua Tao, Yi Gu, Tianmin Shu, Zirui Wang, Zichao Yang, and Zhiting Hu. Language Models Meet World Models: Embodied Experiences Enhance Language Models, October 2023. URL [http://arxiv.org/abs/2305.10626](http://arxiv.org/abs/2305.10626). arXiv:2305.10626 \[cs\].

[^44]: Chengxing Xie, Bowen Li, Chang Gao, He Du, Wai Lam, Difan Zou, and Kai Chen. Swe-fixer: Training open-source llms for effective and efficient github issue resolution, 2025a. URL [https://arxiv.org/abs/2501.05040](https://arxiv.org/abs/2501.05040).

[^45]: Tianbao Xie, Danyang Zhang, Jixuan Chen, Xiaochuan Li, Siheng Zhao, Ruisheng Cao, Toh Jing Hua, Zhoujun Cheng, Dongchan Shin, Fangyu Lei, Yitao Liu, Yiheng Xu, Shuyan Zhou, Silvio Savarese, Caiming Xiong, Victor Zhong, and Tao Yu. OSWorld: Benchmarking Multimodal Agents for Open-Ended Tasks in Real Computer Environments, May 2024. URL [http://arxiv.org/abs/2404.07972](http://arxiv.org/abs/2404.07972). arXiv:2404.07972 \[cs\].

[^46]: Yiqing Xie, Alex Xie, Divyanshu Sheth, Pengfei Liu, Daniel Fried, and Carolyn Rose. Repost: Scalable repository-level coding environment construction with sandbox testing, 2025b. URL [https://arxiv.org/abs/2503.07358](https://arxiv.org/abs/2503.07358).

[^47]: Yiheng Xu, Dunjie Lu, Zhennan Shen, Junli Wang, Zekun Wang, Yuchen Mao, Caiming Xiong, and Tao Yu. AgentTrek: Agent Trajectory Synthesis via Guiding Replay with Web Tutorials, December 2024. URL [http://arxiv.org/abs/2412.09605](http://arxiv.org/abs/2412.09605). arXiv:2412.09605 \[cs\].

[^48]: John Yang, Akshara Prabhakar, Karthik Narasimhan, and Shunyu Yao. InterCode: Standardizing and Benchmarking Interactive Coding with Execution Feedback, October 2023. URL [http://arxiv.org/abs/2306.14898](http://arxiv.org/abs/2306.14898). arXiv:2306.14898 \[cs\].

[^49]: John Yang, Carlos E. Jimenez, Alexander Wettig, Kilian Lieret, Shunyu Yao, Karthik Narasimhan, and Ofir Press. SWE-agent: Agent-Computer Interfaces Enable Automated Software Engineering, November 2024a. URL [http://arxiv.org/abs/2405.15793](http://arxiv.org/abs/2405.15793). arXiv:2405.15793 \[cs\].

[^50]: John Yang, Carlos E. Jimenez, Alex L. Zhang, Kilian Lieret, Joyce Yang, Xindi Wu, Ori Press, Niklas Muennighoff, Gabriel Synnaeve, Karthik R. Narasimhan, Diyi Yang, Sida I. Wang, and Ofir Press. SWE-bench Multimodal: Do AI Systems Generalize to Visual Software Domains?, October 2024b. URL [http://arxiv.org/abs/2410.03859](http://arxiv.org/abs/2410.03859). arXiv:2410.03859 \[cs\].

[^51]: Shunyu Yao, Howard Chen, John Yang, and Karthik Narasimhan. WebShop: Towards Scalable Real-World Web Interaction with Grounded Language Agents, February 2023a. URL [http://arxiv.org/abs/2207.01206](http://arxiv.org/abs/2207.01206). arXiv:2207.01206 \[cs\].

[^52]: Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, and Yuan Cao. ReAct: Synergizing Reasoning and Acting in Language Models, March 2023b. URL [http://arxiv.org/abs/2210.03629](http://arxiv.org/abs/2210.03629). arXiv:2210.03629 \[cs\].

[^53]: Zhaojian Yu, Xin Zhang, Ning Shang, Yangyu Huang, Can Xu, Yishujie Zhao, Wenxiang Hu, and Qiufeng Yin. Wavecoder: Widespread and versatile enhancement for code large language models by instruction tuning, 2024. URL [https://arxiv.org/abs/2312.14187](https://arxiv.org/abs/2312.14187).

[^54]: Zheng Yuan, Hongyi Yuan, Chengpeng Li, Guanting Dong, Keming Lu, Chuanqi Tan, Chang Zhou, and Jingren Zhou. Scaling relationship on learning mathematical reasoning with large language models, 2023. URL [https://arxiv.org/abs/2308.01825](https://arxiv.org/abs/2308.01825).

[^55]: Daoguang Zan, Zhirong Huang, Wei Liu, Hanwu Chen, Linhao Zhang, Shulin Xin, Lu Chen, Qi Liu, Xiaojian Zhong, Aoyan Li, Siyao Liu, Yongsheng Xiao, Liangqiang Chen, Yuyu Zhang, Jing Su, Tianyu Liu, Rui Long, Kai Shen, and Liang Xiang. Multi-swe-bench: A multilingual benchmark for issue resolving, 2025. URL [https://arxiv.org/abs/2504.02605](https://arxiv.org/abs/2504.02605).

[^56]: Kexun Zhang, Weiran Yao, Zuxin Liu, Yihao Feng, Zhiwei Liu, Rithesh Murthy, Tian Lan, Lei Li, Renze Lou, Jiacheng Xu, Bo Pang, Yingbo Zhou, Shelby Heinecke, Silvio Savarese, Huan Wang, and Caiming Xiong. Diversity Empowers Intelligence: Integrating Expertise of Software Engineering Agents, August 2024a. URL [http://arxiv.org/abs/2408.07060](http://arxiv.org/abs/2408.07060). arXiv:2408.07060 \[cs\].

[^57]: Yuntong Zhang, Haifeng Ruan, Zhiyu Fan, and Abhik Roychoudhury. AutoCodeRover: Autonomous Program Improvement, July 2024b. URL [http://arxiv.org/abs/2404.05427](http://arxiv.org/abs/2404.05427). arXiv:2404.05427 \[cs\].

[^58]: Wenting Zhao, Nan Jiang, Celine Lee, Justin T. Chiu, Claire Cardie, Matthias Gallé, and Alexander M. Rush. Commit0: Library Generation from Scratch, December 2024. URL [http://arxiv.org/abs/2412.01769](http://arxiv.org/abs/2412.01769). arXiv:2412.01769 \[cs\].

[^59]: Shuyan Zhou, Frank F. Xu, Hao Zhu, Xuhui Zhou, Robert Lo, Abishek Sridhar, Xianyi Cheng, Tianyue Ou, Yonatan Bisk, Daniel Fried, Uri Alon, and Graham Neubig. WebArena: A Realistic Web Environment for Building Autonomous Agents, April 2024. URL [http://arxiv.org/abs/2307.13854](http://arxiv.org/abs/2307.13854). arXiv:2307.13854 \[cs\].