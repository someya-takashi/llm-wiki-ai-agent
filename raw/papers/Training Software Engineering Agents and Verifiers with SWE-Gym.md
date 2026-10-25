---
title: "Training Software Engineering Agents and Verifiers with SWE-Gym"
source: "https://ar5iv.labs.arxiv.org/html/2412.21139"
author:
published:
created: 2026-10-25
description: "We present SWE-Gym, the first environment for training real-world software engineering (SWE) agents.SWE-Gym contains 2,438 real-world Python task instances, each comprising a codebase with an executable runtime enviro…"
tags:
  - "clippings"
---
Jiayi Pan Affiliation: UC Berkeley Correspondence to: [jiayipan@berkeley.edu](mailto:jiayipan@berkeley.edu)    Xingyao Wang Affiliation: UIUC Correspondence to: [xingyao6@illinois.edu](mailto:xingyao6@illinois.edu)    Graham Neubig Affiliation: CMU    Navdeep Jaitly Affiliation: Apple    Heng Ji Affiliation: UIUC    Alane Suhr Affiliation: UC Berkeley Correspondence to: [suhr@berkeley.edu](mailto:suhr@berkeley.edu)    Yizhe Zhang Affiliation: Apple Correspondence to: [yizzhang@apple.com](mailto:yizzhang@apple.com)

###### Abstract

We present SWE-Gym, the first environment for training real-world software engineering (SWE) agents. SWE-Gym contains 2,438 real-world Python task instances, each comprising a codebase with an executable runtime environment, unit tests, and a task specified in natural language. We use SWE-Gym to train language model based SWE agents, achieving up to 19% absolute gains in resolve rate on the popular SWE-Bench Verified and Lite test sets. We also experiment with inference-time scaling through verifiers trained on agent trajectories sampled from SWE-Gym. When combined with our fine-tuned SWE agents, we achieve 32.0% and 26.0% on SWE-Bench Verified and Lite, respectively, reflecting a new state-of-the-art for open-weight SWE agents. To facilitate further research, we publicly release SWE-Gym, models, and agent trajectories.<sup>1</sup>

###### Keywords:

Machine Learning, ICML <sup>†</sup>

## 1 Introduction

Figure 1: SWE-Gym enables scalable improvements for software engineering agents. Top: Training time scaling shows consistent performance improvements as we obtain more training trajectories, with no signs of saturation at 491 trajectories. We use temperature $t=0$. Bottom: For inference time scaling, we generate a number of candidate trajectories per task and select the best using a verifier trained on SWE-Gym. This approach demonstrates roughly logarithmic gains with the number of sampled solutions. $t=0$ (excluded from regression) is used as the first hypothesis to be consistent with the top figure; later rollouts use $t=0.5$.

| Dataset (split) | Repository-Level | Executable Environment | Real task | \# Instances (total) | \# Instances (train) |
| --- | --- | --- | --- | --- | --- |
| CodeFeedback [^48] | ✗ | ✗ | ✓ | 66,383 | 66,383 |
| APPS [^10] | ✗ | ✓ | ✓ | 10,000 | 5,000 |
| HumanEval [^7] | ✗ | ✓ | ✓ | 164 | 0 |
| MBPP [^29] | ✗ | ✓ | ✓ | 974 | 374 |
| R2E [^15] | ✓ | ✓ | ✗ | 246 | 0 |
| SWE-Bench (train) [^16] | ✓ | ✗ | ✓ | 19,008 | 19,008 |
| SWE-Gym Raw | ✓ | ✗ | ✓ | 66,894 | 66,894 |
| SWE-Bench (test) [^16] | ✓ | ✓ | ✓ | 2,294 | 0 |
| SWE-Gym | ✓ | ✓ | ✓ | 2,438 | 2,438 |

Table 1: SWE-Gym is the first publicly-available training environment combining real-world software engineering tasks from GitHub issues with pre-installed dependencies and executable test verification. Repository-level: whether each task is situated in a sophisticated repository; Executable Environment: whether each instance in the resource comes with an executable environment with all relevant dependencies pre-installed; Real task: whether the instruction for each instance is collected from human developers.

Language models (LMs) have remarkable promise in automating software engineering (SWE) tasks, as most clearly measured by recent progress on recent benchmarks like SWE-Bench [^16] and Commit0 [^46]. While LM-based SWE agents have shown significant performance gains through improving agent-computer interfaces [^39] and prompting strategies [^35], advances in SWE agents have been limited by a reliance on proprietary models, rather than improving underlying LM itself.

Unlike other domains where supervised fine-tuning and reinforcement learning have significantly improved LM capabilities, for example in general chat [^22], mathematical reasoning [^26] [^41], and web navigation [^23], software engineering agents currently lack suitable training environments. Creating such an environment for SWE agents is uniquely challenging. Real-world software engineering requires interaction with an executable runtime that has been prepared with the appropriate software dependencies and reproducible test suites, among other requirements. These challenges are reflected in the existing resources (Tab. 1). For example, the SWE-Bench [^16] training split contains only the solutions (git patches that solve the task), and lacks executable environments and reward signals. R2E [^15] uses synthetic instructions that are very far from real-world problems, while datasets such as CodeContests [^17] and APPS [^10] focus only on isolated tasks rather than realistic repository-level coding problems.

To bridge this gap, we present SWE-Gym, the first training environment combining real-world software engineering tasks from GitHub issues with pre-installed dependencies and executable test verification.<sup>2</sup> SWE-Gym contains 2,438 Python tasks sourced from 11 popular open-source repositories (Tab. ), providing useful environments for training LMs as agents and verifiers.

SWE-Gym supports training state-of-the-art open-weight SWE agents. Based on the OpenHands [^35] agent scaffold for general-purpose software development (§5.1), we fine-tune a 32B Qwen-2.5 coder model [^14] using only 491 agent-environment interaction trajectories sampled using SWE-Gym, and achieve substantial absolute improvements of +12.3% (to 15.3%) and +13.6% (to 20.6%) on SWE-Bench Lite and SWE-Bench Verified respectively (§3.2).

SWE-Gym is effective across agent scaffolds. In another agent scaffold based on a SWE-Bench specialized workflow (Moatless, [^50], §5.1), we observe an improvement to 19.7% (32B model) and 10.0% (7B model) on SWE-Bench Lite through self-improvement, where the LM interacts with SWE-Gym, receives reward from it, and learns to improve itself through rejection sampling fine-tuning.

SWE-Gym supports training verifier models that enable inference-time scaling. We use the test suite executions provided by SWE-Gym to determine whether sampled agent trajectories are successful or not. Given these samples, we train a verifier model (i.e., an outcome-supervised reward model [^8]) that estimates a trajectory’s probability of success, and thus enables inference-time scaling. Specifically, at inference time we sample multiple agent trajectories, and select the one with the highest estimated reward according to the verifier. This approach further improves the resolve rate to 32.0% (+11.4% absolute improvement) on SWE-Bench Verified (§4.1.1, Fig. 1 bottom) and 26.0% on SWE-Bench Lite (§4.1.2), establishing a new state-of-the-art among systems with publicly accessible weights (Tab. 9).

Our baseline training and inference-time scaling methods on SWE-Gym yield continuously improved results with increasing compute (Fig. 1). In the training phase, performance scales with the number of sampled trajectories up to our current limit of 491 trajectories, suggesting that performance is currently limited by sampling compute rather than SWE-Gym’s size. Similarly, using the agent and verifier trained by SWE-Gym, the bottom panel shows that adding more compute during the inference time steadily improves the results.

Category Metric SWE-Gym SWE-Gym Lite Size # Instances 2,438 (2,294) 230 (300) # Repos 11 (12) 11 (12) Issue Text Length by Words 239.8 (195.1) 186.2 (175.9) Codebase # Non-test Files 971.2 (2944.2) 818.8 (2988.5) # Non-test Lines 340675.0 (363728.4) 340626.2 (377562.4) Gold Patch # Lines edited 69.8 (32.8) 10.6 (10.1) # Files edited 2.5 (1.7) 1.0 (1.0) # Func. edited 4.1 (3.0) 1.4 (1.34) Tests # Fail to Pass 10.0 (9.0) 2.04 (3.5) # Total 760.8 (132.5) 99.9 (85.2)

Table 2: Statistics comparing SWE-Gym with the SWE-Bench test split (in parenthesis). Except for size metrics, we report the average value across instances.

Figure 2: Repository distribution of SWE-Gym instances.

## 2 SWE-Gym Environment

SWE-Gym comprises 2,438 real-world software engineering tasks sourced from pull requests in 11 popular Python repositories, with pre-configured executable environments and expert-validated test cases, constructed in close alignment with SWE-Bench [^16]. These repositories are separate from those used in SWE-Bench to avoid contamination. These tasks require SWE agents to develop test-passing solutions for real-world GitHub issues using provided codebases and executable environments. Such agents must map from natural language descriptions of the issue, as well as the initial state of the repository, to a pull request represented as a git patch. To solve the task, agents must edit or create files, and can execute code in the environment for debugging and testing.

The standard SWE-Gym includes 2,438 tasks. We also identify a subset of 230 tasks, SWE-Gym Lite, which contains generally easier and more self-contained tasks that are suitable for rapid prototyping, in alignment with SWE-Bench Lite [^16].

To support future research in SWE agent development and automatic dataset synthesis, we also release a large set of Python GitHub issues without executable environments, which we refer to as SWE-Gym Raw. SWE-Gym Raw includes 66,894 instances spanning 358 Python repositories.

### 2.1 Dataset Construction

Identify Repositories To identify repositories as the basis of SWE-Gym, we first use SEART GitHub search <sup>3</sup> to filter a list of initial repositories. Unlike SWE-Bench, which focuses on the top 5k most downloaded PyPI libraries [^16], we select Python repositories that were created before 7/1/2022 and have more than 500 stars, with at least 300 lines of code, more than 500 pull requests (PRs) and 100 contributors. This results in 358 repositories.

Extract Training Instances from Repositories. We then use SWE-Bench’s instance extraction script to convert these repositories into SWE-Bench-style instances, each corresponding to a Github issue including the natural language description of the issue, a snapshot of the repository in which the issue was created, and a set of unit tests. Over the 358 repositories, we extract 64,689 task instances. We refer to this dataset as SWE-Gym Raw, which is over three times larger than the 19K instances gathered in previous work [^16] and includes nearly ten times as many repositories.

While SWE-Gym Raw instances contain code, issue descriptions and the solution, they do not contain executable environments. And it’s unclear if the unit tests are effective in evaluating the correctness of a solution. We focus on a subset of repositories for which we semi-manually create such executable environments. Of the 358 total repositories collected, we select a subset of 11 repositories with high number of instances.

Version Training Instances. Associating instances with their respective version numbers (e.g. 1.2.3) and setting up environments version-by-version makes the environment collection process more practical by avoiding redundant setup work. We generalize SWE-Bench’s versioning script to support versioning via script execution, and semi-automatically collect versions for each instance based on information available in the repository (e.g., pyproject.toml, git tag, etc).

Setup Executable Environments and Verify Instances An executable environment with pre-installed dependencies is essential for developing software engineering agents, as it most closely reflects an agent’s setup at deployment time, and enables incremental unit test feedback during development. However, configuring dependencies for a specific version of the codebase can be particularly challenging due to the absence of a universal installation method for Python packages. This issue is further compounded by backward compatibility problems where an older version of codebase requires an older version of a dependency, making environment setup even more difficult in old GitHub issues. Despite these challenges, ignoring these difficult environments might cause a distribution bias that decreases the utility of SWE-Gym, so we take on this challenge for the instances from these 11 selected repositories. We manually configure the dependencies for each task instance given on the relevant configuration files (e.g., requirements.txt), continuous integration scripts, or documentation provided in the repository snapshot at the time of the instance’s issue creation. We then run the execution-based validation script from SWE-Bench to verify that the gold patch (i.e., the code diff submitted by a human developer) passes more unit tests than the original code. We estimate that the entire process takes around 200 human annotation hours and 10k CPU core hours. After validation and filtering out failed instances, we obtain 2,438 unit-test-validated instances from 11 different repositories. We publicly release the pre-built Docker images for each instance to allow full reproducibility. Each image takes up an average of 2.6 GB, totaling 6 TB.

### 2.2 SWE-Gym Lite

Solving software engineering tasks is hard and computationally intensive, costing usually $1 or more per task with frontier models [^35]. To improve research efficiency via faster agent evaluation, [^16] introduce SWE-Bench Lite, a canonical subset of 300 instances from SWE-Bench. Following the SWE-Bench Lite filtering pipeline,<sup>4</sup> we delineate the SWE-Gym Lite split, comprising 230 instances. Similar to SWE-Bench Lite, this subset excludes tasks that require editing more than one file, tasks with poorly described problem statements, those with excessively complex ground-truth code diffs, and tests focused on error message validation.

### 2.3 Dataset Statistics

Fig. 2 illustrates that the task distribution across repositories exhibits a long-tail pattern. Notably, tasks associated with pandas comprise nearly one-third of the total, whereas tasks related to bokeh represent a mere one percent.

Our analysis suggests that tasks and repositories included in SWE-Gym are more challenging than those included in SWE-Bench. Tab. shows that overall, SWE-Gym has statistics similar to SWE-Bench, with several key differences. Codebases in SWE-Gym, on average, have relatively fewer files than SWE-Bench, but a similar number of lines of code. However, SWE-Gym has significantly more lines and files edited per gold patch when compared to SWE-Bench. Longer issue descriptions and a larger number of edits in the gold patch may loosely correlate with higher task complexity. Additionally, our experiments find consistently lower model performance on SWE-Gym compared to SWE-Bench.<sup>5</sup> Beyond models and scaffolds over-fitting to SWE-Bench, one explanation for this decreased performance on SWE-Gym could be our deliberate inclusion of sophisticated repositories, such as pandas and machine-learning libraries like MONAI.

## 3 Training LMs as SWE Agents with SWE-Gym

With SWE-Gym, we train language model agents. Our primary objective is to establish reasonable baselines for SWE-Gym using two agent scaffolds (OpenHands, [^35], §3.2; Moatless Tools, [^50], §3.3) and validate the effectiveness of our dataset.

### 3.1 Setting

Agent Scaffolds. We experiment with two types of agents scaffolds –general-purpose prompting and specialized workflows, demonstrated through OpenHands CodeAct [^35] and MoatlessTools [^50].

Policy Improvement Algorithm. As a baseline, we employ a simple policy improvement algorithm: rejection sampling fine-tuning (a.k.a. filtered behavior cloning). This method is widely used as a baseline in language model and reinforcement learning literature [^49] [^28]. At each iteration, a policy is sampled to interact with the environment. This policy may either be a stronger teacher policy or the student policy itself. Only trajectories that achieve rewards exceeding a predefined threshold are retained. The student model is then fine-tuned to mimic these high-reward trajectories using the standard negative log-likelihood loss. In our setting, the reward threshold is set to include only successful trajectories.

Evaluation Metrics. We perform evaluation on SWE-Bench Lite and SWE-Bench Verified [^16]. We measure: (1) Resolve Rate, RR, (%), the percentage of resolved problems; (2) Empty Patch, EP, (%), the percentage of trajectories where none of the code in the repository is edited. We use OpenHands’s remote runtime [^21] feature to parallelize evaluation (e.g., execute unit tests).

Training Setup. We use Qwen-2.5-Coder-Instruct [^13] 7B, 14B, and 32B variants as our base models. For hyper-parameters and details of the training runs, please refer to §B.2.

### 3.2 Training General-Purpose Prompting Agents

<table><tbody><tr><th>Model</th><td colspan="3">Empty Patch (<math><semantics><mrow><mo>%</mo><mo>,</mo><mo>↓</mo></mrow> <annotation>\%,\downarrow</annotation></semantics></math>)</td><td colspan="3">Stuck in Loop (<math><semantics><mrow><mo>%</mo><mo>,</mo><mo>↓</mo></mrow> <annotation>\%,\downarrow</annotation></semantics></math>)</td><td colspan="3">Avg. Turn(s)</td><td colspan="3">Resolve Rate (<math><semantics><mrow><mo>%</mo><mo>,</mo><mo>↑</mo></mrow> <annotation>\%,\uparrow</annotation></semantics></math>)</td></tr><tr><th>Size</th><td>zero-shot</td><td>fine-tuned</td><td><math><semantics><mi>Δ</mi> <annotation>\Delta</annotation></semantics></math></td><td>zero-shot</td><td>fine-tuned</td><td><math><semantics><mi>Δ</mi> <annotation>\Delta</annotation></semantics></math></td><td>zero-shot</td><td>fine-tuned</td><td><math><semantics><mi>Δ</mi> <annotation>\Delta</annotation></semantics></math></td><td>zero-shot</td><td>fine-tuned</td><td><math><semantics><mi>Δ</mi> <annotation>\Delta</annotation></semantics></math></td></tr><tr><th colspan="13">SWE-Bench Lite (300 instances)</th></tr><tr><th>7B</th><td><math><semantics><mn>40.3</mn> <annotation>40.3</annotation></semantics></math></td><td><math><semantics><mn>29.7</mn> <annotation>29.7</annotation></semantics></math></td><td><math><semantics><mrow><mo>−</mo> <mn>10.7</mn></mrow> <annotation>{\color[rgb]{0.1328,0.5469,0.1328}-10.7}</annotation></semantics></math></td><td><math><semantics><mn>47.0</mn> <annotation>47.0</annotation></semantics></math></td><td><math><semantics><mn>31.0</mn> <annotation>31.0</annotation></semantics></math></td><td><math><semantics><mrow><mo>−</mo> <mn>16.0</mn></mrow> <annotation>{\color[rgb]{0.1328,0.5469,0.1328}-16.0}</annotation></semantics></math></td><td><math><semantics><mn>20.3</mn> <annotation>20.3</annotation></semantics></math></td><td><math><semantics><mn>22.2</mn> <annotation>22.2</annotation></semantics></math></td><td><math><semantics><mrow><mo>+</mo> <mn>1.9</mn></mrow> <annotation>+1.9</annotation></semantics></math></td><td><math><semantics><mrow><mn>1.0</mn> <mo></mo><mrow><mo>(</mo><mrow><mo>±</mo> <mn>1.0</mn></mrow><mo>)</mo></mrow></mrow> <annotation>1.0(\pm 1.0)</annotation></semantics></math></td><td><math><semantics><mrow><mn>10.0</mn> <mo></mo><mrow><mo>(</mo><mrow><mo>±</mo> <mn>2.4</mn></mrow><mo>)</mo></mrow></mrow> <annotation>10.0(\pm 2.4)</annotation></semantics></math></td><td><math><semantics><mrow><mo>+</mo> <mn>9.0</mn></mrow> <annotation>{\color[rgb]{0.1328,0.5469,0.1328}+9.0}</annotation></semantics></math></td></tr><tr><th>14B</th><td><math><semantics><mn>49.7</mn> <annotation>49.7</annotation></semantics></math></td><td><math><semantics><mn>18.1</mn> <annotation>\mathbf{18.1}</annotation></semantics></math></td><td><math><semantics><mrow><mo>−</mo> <mn>31.6</mn></mrow> <annotation>{\color[rgb]{0.1328,0.5469,0.1328}-31.6}</annotation></semantics></math></td><td><math><semantics><mn>31.7</mn> <annotation>31.7</annotation></semantics></math></td><td><math><semantics><mn>27.1</mn> <annotation>27.1</annotation></semantics></math></td><td><math><semantics><mrow><mo>−</mo> <mn>4.6</mn></mrow> <annotation>{\color[rgb]{0.1328,0.5469,0.1328}-4.6}</annotation></semantics></math></td><td><math><semantics><mn>23.2</mn> <annotation>23.2</annotation></semantics></math></td><td><math><semantics><mn>21.4</mn> <annotation>21.4</annotation></semantics></math></td><td><math><semantics><mrow><mo>−</mo> <mn>1.8</mn></mrow> <annotation>-1.8</annotation></semantics></math></td><td><math><semantics><mrow><mn>2.7</mn> <mo></mo><mrow><mo>(</mo><mrow><mo>±</mo> <mn>1.9</mn></mrow><mo>)</mo></mrow></mrow> <annotation>2.7(\pm 1.9)</annotation></semantics></math></td><td><math><semantics><mrow><mn>12.7</mn> <mo></mo><mrow><mo>(</mo><mrow><mo>±</mo> <mn>2.3</mn></mrow><mo>)</mo></mrow></mrow> <annotation>12.7(\pm 2.3)</annotation></semantics></math></td><td><math><semantics><mrow><mo>+</mo> <mn>10.0</mn></mrow> <annotation>{\color[rgb]{0.1328,0.5469,0.1328}+10.0}</annotation></semantics></math></td></tr><tr><th>32B</th><td><math><semantics><mn>27.0</mn> <annotation>\mathbf{27.0}</annotation></semantics></math></td><td><math><semantics><mn>18.1</mn> <annotation>\mathbf{18.1}</annotation></semantics></math></td><td><math><semantics><mrow><mo>−</mo> <mn>8.9</mn></mrow> <annotation>{\color[rgb]{0.1328,0.5469,0.1328}-8.9}</annotation></semantics></math></td><td><math><semantics><mn>16.7</mn> <annotation>\mathbf{16.7}</annotation></semantics></math></td><td><math><semantics><mn>18.1</mn> <annotation>\mathbf{18.1}</annotation></semantics></math></td><td><math><semantics><mrow><mo>+</mo> <mn>1.5</mn></mrow> <annotation>{\color[rgb]{0.8633,0.0781,0.2344}+1.5}</annotation></semantics></math></td><td><math><semantics><mn>15.5</mn> <annotation>\mathbf{15.5}</annotation></semantics></math></td><td><math><semantics><mn>29.3</mn> <annotation>29.3</annotation></semantics></math></td><td><math><semantics><mrow><mo>+</mo> <mn>13.9</mn></mrow> <annotation>+13.9</annotation></semantics></math></td><td><math><semantics><mrow><mn>3.0</mn> <mo></mo><mrow><mo>(</mo><mrow><mo>±</mo> <mn>1.4</mn></mrow><mo>)</mo></mrow></mrow> <annotation>\mathbf{3.0}(\pm 1.4)</annotation></semantics></math></td><td><math><semantics><mrow><mn>15.3</mn> <mo></mo><mrow><mo>(</mo><mrow><mo>±</mo> <mn>2.5</mn></mrow><mo>)</mo></mrow></mrow> <annotation>\mathbf{15.3}(\pm 2.5)</annotation></semantics></math></td><td><math><semantics><mrow><mo>+</mo> <mn>12.3</mn></mrow> <annotation>{\color[rgb]{0.1328,0.5469,0.1328}\mathbf{+12.3}}</annotation></semantics></math></td></tr><tr><th colspan="13">SWE-Bench Verified (500 instances)</th></tr><tr><th>7B</th><td><math><semantics><mn>45.8</mn> <annotation>45.8</annotation></semantics></math></td><td><math><semantics><mn>33.8</mn> <annotation>33.8</annotation></semantics></math></td><td><math><semantics><mrow><mo>−</mo> <mn>12.0</mn></mrow> <annotation>{\color[rgb]{0.1328,0.5469,0.1328}-12.0}</annotation></semantics></math></td><td><math><semantics><mn>39.6</mn> <annotation>39.6</annotation></semantics></math></td><td><math><semantics><mn>21.0</mn> <annotation>\mathbf{21.0}</annotation></semantics></math></td><td><math><semantics><mrow><mo>−</mo> <mn>18.6</mn></mrow> <annotation>{\color[rgb]{0.1328,0.5469,0.1328}-18.6}</annotation></semantics></math></td><td><math><semantics><mn>21.9</mn> <annotation>21.9</annotation></semantics></math></td><td><math><semantics><mn>35.3</mn> <annotation>35.3</annotation></semantics></math></td><td><math><semantics><mrow><mo>+</mo> <mn>13.4</mn></mrow> <annotation>+13.4</annotation></semantics></math></td><td><math><semantics><mrow><mn>1.8</mn> <mo></mo><mrow><mo>(</mo><mrow><mo>±</mo> <mn>1.1</mn></mrow><mo>)</mo></mrow></mrow> <annotation>1.8(\pm 1.1)</annotation></semantics></math></td><td><math><semantics><mrow><mn>10.6</mn> <mo></mo><mrow><mo>(</mo><mrow><mo>±</mo> <mn>2.1</mn></mrow><mo>)</mo></mrow></mrow> <annotation>10.6(\pm 2.1)</annotation></semantics></math></td><td><math><semantics><mrow><mo>+</mo> <mn>8.8</mn></mrow> <annotation>{\color[rgb]{0.1328,0.5469,0.1328}+8.8}</annotation></semantics></math></td></tr><tr><th>14B</th><td><math><semantics><mn>44.9</mn> <annotation>44.9</annotation></semantics></math></td><td><math><semantics><mn>14.5</mn> <annotation>14.5</annotation></semantics></math></td><td><math><semantics><mrow><mo>−</mo> <mn>30.4</mn></mrow> <annotation>{\color[rgb]{0.1328,0.5469,0.1328}-30.4}</annotation></semantics></math></td><td><math><semantics><mn>32.1</mn> <annotation>32.1</annotation></semantics></math></td><td><math><semantics><mn>21.3</mn> <annotation>21.3</annotation></semantics></math></td><td><math><semantics><mrow><mo>−</mo> <mn>10.7</mn></mrow> <annotation>{\color[rgb]{0.1328,0.5469,0.1328}-10.7}</annotation></semantics></math></td><td><math><semantics><mn>25.5</mn> <annotation>25.5</annotation></semantics></math></td><td><math><semantics><mn>30.1</mn> <annotation>30.1</annotation></semantics></math></td><td><math><semantics><mrow><mo>+</mo> <mn>4.6</mn></mrow> <annotation>+4.6</annotation></semantics></math></td><td><math><semantics><mrow><mn>4.0</mn> <mo></mo><mrow><mo>(</mo><mrow><mo>±</mo> <mn>1.6</mn></mrow><mo>)</mo></mrow></mrow> <annotation>4.0(\pm 1.6)</annotation></semantics></math></td><td><math><semantics><mrow><mn>16.4</mn> <mo></mo><mrow><mo>(</mo><mrow><mo>±</mo> <mn>2.0</mn></mrow><mo>)</mo></mrow></mrow> <annotation>16.4(\pm 2.0)</annotation></semantics></math></td><td><math><semantics><mrow><mo>+</mo> <mn>12.4</mn></mrow> <annotation>{\color[rgb]{0.1328,0.5469,0.1328}+12.4}</annotation></semantics></math></td></tr><tr><th>32B</th><td><math><semantics><mn>9.5</mn> <annotation>\mathbf{9.5}</annotation></semantics></math></td><td><math><semantics><mn>13.8</mn> <annotation>\mathbf{13.8}</annotation></semantics></math></td><td><math><semantics><mrow><mo>+</mo> <mn>4.3</mn></mrow> <annotation>{\color[rgb]{0.8633,0.0781,0.2344}+4.3}</annotation></semantics></math></td><td><math><semantics><mn>29.4</mn> <annotation>\mathbf{29.4}</annotation></semantics></math></td><td><math><semantics><mn>23.8</mn> <annotation>23.8</annotation></semantics></math></td><td><math><semantics><mrow><mo>−</mo> <mn>5.6</mn></mrow> <annotation>{\color[rgb]{0.1328,0.5469,0.1328}-5.6}</annotation></semantics></math></td><td><math><semantics><mn>24.6</mn> <annotation>24.6</annotation></semantics></math></td><td><math><semantics><mn>31.6</mn> <annotation>31.6</annotation></semantics></math></td><td><math><semantics><mrow><mo>+</mo> <mn>7.0</mn></mrow> <annotation>+7.0</annotation></semantics></math></td><td><math><semantics><mrow><mn>7.0</mn> <mo></mo><mrow><mo>(</mo><mrow><mo>±</mo> <mn>1.3</mn></mrow><mo>)</mo></mrow></mrow> <annotation>\mathbf{7.0}(\pm 1.3)</annotation></semantics></math></td><td><math><semantics><mrow><mn>20.6</mn> <mo></mo><mrow><mo>(</mo><mrow><mo>±</mo> <mn>2.1</mn></mrow><mo>)</mo></mrow></mrow> <annotation>\mathbf{20.6}(\pm 2.1)</annotation></semantics></math></td><td><math><semantics><mrow><mo>+</mo> <mn>13.6</mn></mrow> <annotation>{\color[rgb]{0.1328,0.5469,0.1328}\mathbf{+13.6}}</annotation></semantics></math></td></tr></tbody></table>

Table 3: Model performance (fine-tuned on 491 SWE-Gym-sampled trajectories) on SWE-Bench [^16] using OpenHands [^35] as agent scaffold. We use Qwen-2.5-Coder-Instruct as the base model. We set temperature $t=0$ for evaluation.

In this subsection, we use OpenHands (version CodeActAgent 2.1, [^35] [^34]) as our agent scaffold. OpenHands CodeActAgent uses a ReAct-style [^40] general-purpose prompting without specialized workflow (§5.1) and relies on the underlying LM to perform action and interpret observation. It equips an LM with a bash terminal and a file editor. We disable the browser feature of OpenHands in this work. We use OpenHands’s remote runtime [^21] feature to roll out agents in parallel (i.e., execute agent’s actions) on SWE-Gym.

Trajectory Collection. We roll out 491 successful trajectories where the agent successfully solves the given task from SWE-Gym. These trajectories are sampled from two models (gpt-4o-2024-08-06 and claude-3-5-sonnet-20241022) with different temperature settings. Each success trajectory, on average, has 39.9 LM completion messages (roughly 19 turns) and $18,578$ tokens. Please refer to Tab. 8 for more details. While SWE-Gym contains many more tasks for trajectory collection and allows repeated sampling, our current set of 491 trajectories was primarily limited by computational budget constraints rather than the availability of tasks. Please refer to Tab. 10 and §B.3 for details on the temperature setting and the resolve rate in the training set.

Additional Evaluation Metrics & Settings. In addition to the metrics presented in §3.1, we include the following for further analysis: (3) Stuck in Loop (%), the percentage of trajectories where the agent is stuck in a loop by repeating the exact same action for the last three turns; (4) Avg. Turn(s), the average number of action-observation turns for these trajectories. Our evaluation of the trained LM is bounded by either 100 interaction turns or the 32k context window length, whichever is reached first. We use sampling temperature of 0 (i.e., greedy) unless otherwise specified.

Training on SWE-Gym trajectories turns LM into effective agents to fix issues. As shown in Tab. 3, despite the base model Qwen-2.5-Coder-Instruct-32B only performs $3.0\%$ and $7.0\%$ on SWE-Bench Lite and Verified, the model fine-tuned on SWE-Gym-sampled trajectories is able to achieve consistent improvements, up to $12.3\%$ ($3.0\%\rightarrow 15.3\%$) and $13.6\%$ ($7.0\%\rightarrow 20.6\%)$ absolute performance (i.e., more Github issue resolved) with the largest 32B model.

Training help reduce agent’s stuck-in-loop behavior. Open-weight LMs, often suffer from stuck-in-loop, especially when prompted with general-purpose prompts (§5.1): the LM would repeat exactly the same action for multiple turns and never get out of this loop. This is evident in the zero-shot stuck in loop statistics in Tab. 3, where even the largest 32B model gets stuck in loop $30\%$ of the time. The fine-tuned model consistently reduces the stuck in loop rate ranging from $-4.6\%$ to $-18.6\%$ for problems in both SWE-Bench Lite and Verified, with the exception of 32B model on SWE-Bench Lite ($+1.5\%$). The reduced stuck-in-loop rate likely encourages the agent to execute more code edits, as evident by the decreased empty patch rate across different scales (except for 32B on SWE-Bench Verified).

Performance scales with model size. Rather unsurprisingly, we see the resolve rate, the empty patch, and the stuck-in-loop rate improves consistently in both SWE-Bench Lite and Verified as we switch to bigger base models (Tab. 3).

Self-improvement is not yet working. We use the fine-tuned 32B model to rollout trajectories from SWE-Gym for 6 times and obtain 868 successful trajectories (i.e. on-policy trajectories) using temperature $t=0.5$. We further fine-tune the base 32B model on a mixture of 868 on-policy trajectories and the previously collected 491 off-policy trajectories. When evaluating the model on SWE-Bench Lite, we observe a performance drop from 15.3% to 8.7% compared to the model that was fine-tuned using only off-policy trajectories from GPT-4 and Claude.

We hypothesize improved results could be achieved with more advanced optimization methods, such as Proximal Policy Optimization (PPO) [^25], or by employing a stronger base model. These directions remain promising avenues for future investigation.

### 3.3 Self-improvement with Specialized Workflow Agents

We then explore the opposite end of the scaffold spectrum—agents with specialized workflows—using the MoatlessTools Agent (version 0.0.2; [^50]) for our experiments. Unlike OpenHands CodeAct, which offers extensive freedom in long-horizon planning, this scaffold constrains the model’s action space with pre-defined workflows, effectively reducing task horizons. As demonstrated in Tab. 3 and Tab. 5, we observe that open-weight models consistently deliver better zero-shot performance when paired with the MoatlessTools scaffold.

Given the improved zero-shot performance and shorter task horizon with the MoatlessTools scaffold, we explore if SWE-Gym allows agents to self-improve without reliance on a strong teacher. With a limited compute budget, we conduct this experiment with only 7B and 34B models, using LoRA [^12] for the 34B models for improved efficiency.

We start our experiments with ablations on 7B model and scale it to 32B. We sample the model for 30 rounds on SWE-Gym-Lite with a high temperature of 1.0, adding successful trajectories to the fine-tuning dataset. This process is repeated twice, after which the improvements are marginal.

Easy Data Bias Degrades Model Performance. During repeated sampling, similar to the observation in [^4], we find that the success probability for each instance follows a long-tail distribution as shown in Fig. 3, with more instances being solved simply by increasing the number of samples. While having a broader coverage of tasks should be beneficial for training, as first observed in math reasoning [^31], repeated sampling introduces a distribution bias toward easier tasks, making it suboptimal to naively train on all successful trajectories.

Figure 3: Success distribution over 30 rounds on SWE-Gym Lite with 7B model in zero-shot. The distribution is naturally biased toward easy tasks. Per instance capping reduces this bias but lowers the total trajectory count for training. We set temperature $t=1$ during sampling.

Mitigate Easy Data Bias with Per Instance Capping. To mitigate such bias, [^31] propose to keep the training set size same but obtain equal or more number of positive instances on difficult tasks. However, obtaining positive instances from harder tasks is inherently difficult and on average requires more rounds of sampling and therefore more compute. We consider a compute-bounded scenario with a fixed sampling budget to optimize model performance through data selection.

We propose per-instance capping - a simple method limiting the maximum instances per task. As shown in Fig. 3, this approach balances dataset bias and size. Too low a cap limits dataset size and hurts performance (see §4.2), while too high a cap skews distribution toward easier tasks.

As shown in Tab. 4, empirically, we find a threshold of 2 achieves a good balance between dataset size and difficulty bias, performing marginally better than the full dataset while improving training speed. We rank trajectories by the rounds of model responses they need, fewer the better.

| Cap | \# Traj | Empty Patch ($\%,\downarrow$) | Resolve Rate ($\%,\uparrow$) |
| --- | --- | --- | --- |
| 0 (Zero-shot) | 0 | 56.3 | 7.0 |
| 1 | 36 | 37.3 | 9.0 |
| 2 | 62 | 29 | 9.7 |
| 3 | 82 | 43.7 | 7.7 |
| No Cap (All) | 172 | 30.7 | 9.3 |

Table 4: Resolve rate and empty patch rate on SWE-Bench Lite after 7B model trained with with data from different instance capping strategies (Cap) and therefore different number of trajectories (Traj).

<table><tbody><tr><th rowspan="2">Setting</th><td colspan="2">7B Model</td><td colspan="2">32B Model</td></tr><tr><td>EP(<math><semantics><mrow><mo>%</mo><mo>,</mo><mo>↓</mo></mrow> <annotation>\%,\downarrow</annotation></semantics></math>)</td><td>RR(<math><semantics><mrow><mo>%</mo><mo>,</mo><mo>↑</mo></mrow> <annotation>\%,\uparrow</annotation></semantics></math>)</td><td>EP(<math><semantics><mrow><mo>%</mo><mo>,</mo><mo>↓</mo></mrow> <annotation>\%,\downarrow</annotation></semantics></math>)</td><td>RR(<math><semantics><mrow><mo>%</mo><mo>,</mo><mo>↑</mo></mrow> <annotation>\%,\uparrow</annotation></semantics></math>)</td></tr><tr><th>Zero-Shot</th><td>56.3%</td><td>7.0%</td><td>24.3%</td><td>19.0%</td></tr><tr><th>Iteration 1</th><td>29.0%</td><td>9.0%</td><td>18.3%</td><td>19.7%</td></tr><tr><th>Iteration 2</th><td>23.3%</td><td>10.0%</td><td>9.7%</td><td>19.7%</td></tr></tbody></table>

Table 5: Resolve rate (RR) and Empty patch rate (EP) on SWE-Bench Lite with MoatlessTools Scaffold after online rejection sampling fine-tuning, evaluated at temperature $t=0$. RR shown in highlighted columns.

Results. With a capping of 2, we perform two rounds of policy improvements for both 7B and 32B models and present results in Tab. 5. For the 7B model, the resolve rate improves significantly, rising from 7.0% in the zero-shot setting to 9.0% after the first iteration and reaching 10.0% after the second iteration, indicating notable benefits from training on SWE-Gym. In contrast, the 32B model demonstrates strong initial performance with a zero-shot resolve rate of 19.0%, but shows only marginal improvement to 19.7% after the first iteration and no additional gains in subsequent iterations. We hypothesize that this plateau may stem from limitations inherent to the agent scaffold and the rejection sampling finetuning algorithm we employed to improve the policy.

## 4 Scaling Agent Improvements with SWE-Gym

### 4.1 Inference-Time Scaling with SWE-Gym-powered Verifier

Beyond training the agent, trajectories sampled from SWE-Gym also allow us to train a verifier (a.k.a. reward model). As a baseline, we train an outcome-supervised reward model (ORM) [^8] that, given the relevant context of the task execution (e.g., problem statement, trajectories, git diff), generates a score representing the probability if the agent solves the problem.

We show that such learned verifiers enable effective inference-time scaling for further performance improvement. Using the same underlying LM, one can sample multiple solutions for the same problem and use the verifier to pick the best solution.

#### 4.1.1 Verifier for Agent Scaffold with General-Purpose Prompting

For OpenHands CodeActAgent [^35] [^34] that uses general-purpose prompting (§5.1), we consider a verifier (ORM) setting where the input is an interleaved trajectory $\tau$ and output is a scalar reward $r$:

$$
\tau=[o_{1},a_{1},o_{2},a_{2},\dots,o_{n},a_{n}],r\in[0,1]
$$

where observation $o_{k}$ can be the problem-statement, command execution output, error messages, etc; action $a_{k}$ can be bash command or file operations (e.g., edit, view) from the agent. We include verifier prompt template in §B.5.

Training Setup. Following the same training hyper-parameters as described in §3, we train a 32B Qwen2.5-Coder-Instruct as verifier. We train the LM to predict ‘YES’ or ‘NO’ regarding if the agent has successfully solved the request based on the agent trajectory (see the verifier prompt in §B.5). At inference time, conditioned on the prompt and the agent trajectory $\tau$, we use SGLang [^47] to obtain the log probability of the next token being ‘YES’ ($l_{y}$) or ‘NO’ ($l_{n}$). We then calculate the probability of success as ORM by normalizing the log probability: $p_{\text{yes}}=\exp(l_{y})/(\exp(l_{y})+\exp(l_{n}))$.

Verifier Training Data. We re-use two sets of trajectories we sampled for agent training in §3.2: (1) off-policy trajectories which contain 443 successful trajectories that are sampled from gpt-4o and claude-3-5-sonnet; (2) on-policy trajectories which contain 875 successful trajectories sampled from the agent model (fine-tuned Qwen2.5-Coder-Instruct-32B). Note that we only keep trajectories that fit in the context window (32k tokens) for training. We combine both on-policy and off-policy trajectories, randomly sample the same amount of unsuccessful trajectories from each subset (1318 in total), and combine them as our dataset for verifier training.

Metrics. We measure (1) Pass@K, which represents the percentage of tasks where at least one successful solution is found among K sampled trajectories, and (2) Best@K, which selects the trajectory with the highest verification score ($p_{\text{yes}}$) among K samples and reports the percentage of these selected trajectories that are successful. Pass@K evaluates the model’s ability to find any working solution (i.e., the upper bound for Best@K), while Best@K assesses our verifier’s capability to identify the most promising solution. We calculate the mean and variance for each data point following [^18], please see §B.1 for details on the calculation.

Figure 4: Scaling inference-time compute improves performance on SWE-Bench Verified using a fine-tuned verifier. Both the agent and the verifier are Qwen2.5-Coder-Instruct-32B model fine-tuned on the corresponding dataset (§4.1.1). OpenHands [^35] is used as the agent scaffold. The first rollout was performed with temperature $t=0$, and $t=0.5$ was used for the rest.

Result. Fig. 4 shows how Pass@K and Best@K metrics scale with the number of sampled agent trajectories using the fine-tuned 32B model as the agent model. Pass@K demonstrates strong improvement, rising from $20.6\%$ to $37.8\%$ as K increases from 1 to 8, and to $42.8\%$ at k=16. The Best@K metric, which relies on our verifier’s ability to select the best trajectory, demonstrates more modest but steady progress, improving from $20.6\%$ at k=1 to $29.8\%$ at k=8, and to $32.0\%$ at k=16. The gap between Pass@K and Best@K reveals room for more advanced reward modeling techniques for coding agents. Surprisingly, we found that fine-tuning the model using LoRA [^12] (29.8% $@8$) via Unsloth library [^32] performs better than full-parameter fine-tuning for verifier training (27.2% $@8$), potentially due to the effect of regularization. Furthermore, as shown in Fig. 1 (bottom), the Best@K curve exhibits strong linearity on a logarithmic scale, indicating a promising scaling behavior.

Figure 5: Abaltion study for verifier training (§4.1.1). Performances are evaluated on SWE-Bench Verified. Both the agent and the verifier are Qwen2.5-Coder-Instruct-32B model fine-tuned on the corresponding dataset (§4.1.1). OpenHands [^35] is used as the agent scaffold.

Ablation Study: Training Data matters for Verifier. As shown in Fig. 5, our ablation study demonstrates that the choice of training data can significantly impact verifier performance. Different from the LoRA-trained model in the previous paragraph, all ablation studies perform full-parameter fine-tuning on a 32B Qwen2.5-Coder-Instruct for the verifier. Training with a mixture of off-policy and on-policy data yields the best results (our default setting), with Best@k scaling from $20\%$ to approximately $27\%$ at k=8. In contrast, using only on-policy data from the fine-tuned model shows moderate but limited improvement, while training exclusively on off-policy data from Claude and GPT leads to early performance plateaus around $22\%$. Notably, using off-policy data with twice the number of negative examples performs slightly worse with a lower number of $k$, suggesting that overweighting negative examples during training could make the verifier overly conservative. These findings indicate that verifier training benefits most from a diverse dataset combining both off-policy and on-policy examples, as well as balanced positive and negative data, enabling better generalization and more accurate identification of successful solutions during inference time scaling.

#### 4.1.2 Verifier for Agent Scaffold with Specialized Workflow

Figure 6: Scaling inference-time compute for MoatlessTools Agents with learned verifiers. We set temperature $t=0.5$ during sampling.

For MoatlessTools Agents with specialized workflows, given that it doesn’t have a turn-taking action-observation trajectory like OpenHands CodeActAgent, we prepare verifier inputs through a parsing process. This process combines task descriptions, relevant agent context, and generated patches, training the model to output a single confidence token indicating task success.. We adapt the context extractor from [^44] and provide the prompt template in §B.4.

We train 7B and 32B verifiers using on-policy trajectories from the last round of sampling, applying LoRA [^12] for regularization. To address data bias, we cap positive data points per instance at 2 and balance the dataset by subsampling failure cases to match successful ones.

Result. We evaluate the verifiers by sampling the policy 8 times at temperature 0.5. As shown in Fig. 6, these verifiers enable effective scaling across model sizes - the 7B agent-verifier system improves from 10% to 13.3% success rate on SWE-Bench Lite, while the 32B system improves from 19.7% to 26.3%. The 7B verifier plateaus after N=4 samples when ranking trajectories from both 7B and 32B agents. In contrast, the 32B verifier continues improving even at N=8, suggesting model size affects scaling behavior.

### 4.2 Training-Time Scaling with Data

In this subsection, we study the impact of training data scaling on agent performance. We identify three types of data scaling for sampled trajectories: (1) Random Scaling (No Dedup.), which simply increases the total number of training trajectories regardless of uniqueness (Fig. 7); (2) Unique Instance Scaling (Dedup.), which scales the number of distinct instances by randomly selecting one success trajectory per instance (Fig. 8); and (3) Repository Scaling (Dedup.), which samples trajectories by sequentially including all instances from one repository at a time, sorted by repository name, to study the impact of repository-level diversity.

Setup. Using OpenHands [^35], we evaluate these scaling approaches on SWE-Bench Verified: random scaling on the full trajectory dataset from §3.2 (491 trajectories), unique instance scaling on these trajectories deduplicated by instance ID (294 trajectories), and repository-based scaling where we sort repositories alphabetically and include all trajectories from each repository in order (e.g., first 25% contains complete trajectories from the first N repositories). We compare models trained on 25%, 50%, and 100% of the corresponding dataset for each approach. Please refer to Tab. 7 for detailed statistics of these datasets.

Model performance scales well with more training examples. As illustrated in Fig. 7, we observe favorable scaling behavior: there is consistent improvements in model resolve rate as training data increases, particularly for the 32B model. These results suggest that SWE-Gym’s current size and repository diversity are likely not performance bottlenecks - further improvements could likely be achieved by allocating more computing resources to sampling more training trajectories.

Figure 7: Model performance scaling with training data size. The x-axis shows the percentage of training data used in log base 2 scale.

Similar scaling trends suggest instance and repository diversity is not yet a bottleneck. Fig. 7 reveals comparable overall performance between different scaling approaches up to where deduplication takes effect. While Random Scaling (No Dedup.) achieves higher final performance, this is likely due to having more trajectories (491 vs 294) rather than better scaling efficiency. Among deduplicated approaches, Repository Scaling shows stronger initial performance at 25% data, suggesting that complete repository coverage may provide more coherent learning signals early in training. These results suggest that the repository and instance diversity of SWE-Gym is not yet a bottleneck - further improvements could likely be achieved by simply sampling more agent trajectory data for traning, regardless of duplication or repository distribution.

Figure 8: Comparison of three data sampling approaches: without deduplication, repository-based sampling, and random sampling (§4.2). All variants use the 32B model evaluated on SWE-Bench Verified.

## 5 Related Work

### 5.1 Agents that Solve Github Issues

In this paper, we focus on software engineering agents designed to automatically resolve GitHub issues, specifically in the context of SWE-Bench [^16]. These agents operate in a well-defined setting: given a GitHub issue and its associated code repository, they generate a valid code modification (i.e., git diff patch) to address the issue. The correctness of these modifications is verified through the execution of a test suite written by human developers.

Existing agent designs can be categorized by the number of human priors injected into the agent workflow: (1) specialized workflow: [^38] [^50] [^45] [^6] build specialized workflow with human-defined stages (e.g., localization, code edit, patch re-ranking selection), where an LM is iteratively prompted to produce the final results. While restricting the LM to work on pre-defined subtasks, this approach effectively reduces the task horizon and alleviates the need for long-term planning. However, these specialized workflows face limitations: they require significant human engineering to define appropriate stages and transitions, may not generalize well to novel issue types outside their designed workflow, and can be brittle when intermediate steps fail. (2) general-purpose prompting: Systems such as [^39] [^35] rely on general prompting approaches, including methods like ReAct [^40] and CodeAct [^34]. These systems primarily depend on the LM itself to plan through long horizons and produce the next action based on an evolving history of actions and observations, without enforcing a fixed workflow or relying heavily on human pre-defined stages. While more flexible, these general approaches place higher demands on the underlying LM’s capabilities and can be computationally expensive due to multiple rounds of interaction with a growing interaction history.

Most existing successful methods are based on prompting proprietary models and crafting specialized workflow to workaround existing limitations of those models. This starkly contrasts the success of learning-based approaches in other domains [^27] [^1], where a system learns from prior interactions and rewards to develop task competency progressively. We argue that proper training environments and baselines can help accelerate research in this direction. We verify that we can use SWE-Gym to build strong agents through learning.

### 5.2 Environments for Training Software Agents

There is no existing dataset suitable for training software engineering agents. Existing datasets often exhibit distinct constraints. SWE-Bench [^16] is widely used for evaluating software engineering performance. The training split lacks executable environments and success signals present in the evaluation split, making its usefulness limited for model training. HumanEval [^7] is designed for standalone code generation tasks, akin to coding competitions. Therefore, it falls short of addressing the complex challenges inherent in real-world, repository-level software engineering tasks, which involve thousands of files, millions of lines of code, and tasks such as bug fixing, feature development, and system optimization. Similarly, R2E [^15] is a small evaluation dataset with 246 instances and, due to its synthetic nature, lacks the realism and complexity in real world software engineering scenario. There is also limited empirical evidence if this environment is useful for training. Our proposed SWE-Gym instead uses real-world GitHub issue bodies as task instructions, and uses the associated unit tests for validation. This approach introduces a realistic and complex task formulations, aligning closely with real-world software engineering challenges.

### 5.3 Post-training: From Chatbots, Reasoners to Agents

Post-training, which finetunes a pre-trained language model, usually through supervised or reinforcement learning, has proven highly effective in improving model performance across various domains. RLHF [^22] has become a standard method for adapting language models into chatbots, improving both performance and alignment of the model [^30]. In mathematical reasoning, two standard datasets, MATH [^11] and GSM-8K [^8], have question and answer pairs for both training and evaluation, which enables researchers to explore methods to train both policy, and verifiers (reward models) [^8] [^33].

Earlier works [^34] [^5] [^42] [^36] shows distilling agent trajectories from stronger models consistently improves weaker models. Recent works shift towards self-improving methods. [^37] [^43] [^3] show how RL or rejection sampling fine-tuning, guided by a reward signal, can enable models to self-improve without reliance on a more capable teacher model.

Despite the wide success, post-training generally rely on either expert demonstration data or a training environment with reliable reward signals, which is largely missing in the software engineering domain. This limitation has contributed to a reliance on prompting-based methods with proprietary language models, as reflected in most top-performing approaches on SWE-bench [^16]. Our work addresses this gap with SWE-Gym, a training environment grounded in real-world software engineering tasks, using associated expert written tests as the reward signal. Our experiments verify that we can use SWE-Gym to build strong SWE agents without any prompt engineering. Concurrent with our work, [^19] and [^9] both study the training of software engineering agents, with the latter also exploring verifiers training and dataset construction. We discuss the key differences between these concurrent works in §A.

## 6 Conclusions

In this paper, we introduce SWE-Gym, the first open training environment that bridges critical gaps in enabling scalable learning for software engineering agents. By combining real-world Python tasks with repository-level context, pre-configured execution environments, and test verifications, SWE-Gym will be a foundation for advancing LM agent training research. Through extensive experiments, we demonstrate that SWE-Gym enables both agent and verifier models to achieve significant improvements in resolving complex software tasks. Our findings highlight the scalability of these approaches, revealing potential for continuous performance gains with increased compute.

## 7 Acknowledgments

We thank John Yang and Ofir Press for helpful discussions, and John Yang for assistance in reproducing data analysis results from SWE-Bench. We thank Modal Labs <sup>6</sup> for the GPU compute support through its Academic Credits Program.

## References

## Appendix A Comparison with Concurrent Works

[^19] trains an LM agent, Lingma SWE-GPT, using a method similar to our rejection sampling fine-tuning baseline, with a dataset comparable to our SWE-Gym Raw splits. Without executable unit test feedback, they rely on manually defined heuristics to filter out low-quality trajectories, such as comparing similarity between submitted patches and edit locations with gold patches. The model weights are publicly accessible but not the training pipeline or the dataset.

Most relevant to our work are two consecutive blog posts by [^9] and [^2], who also construct an executable training environment with real-world tasks from GitHub. Instead of manual configuration, they employ a general environment setup script and simply discard instances that fail the setup process. This approach leads to key differences in dataset size and distribution: while it biases the environment away from tasks with complex dependencies, they successfully collect 6,415 instances, about 1.5 times larger than our dataset. In [^9], they also study training agents and verifiers with the environment. Additionally, they explore a lookahead setting where a trained verifier ranks and selects the best next action. With a substantially large collection of agent trajectories (80,036 compared to thousands in our experiments) and model size (72B compared to 32B), Their best system achieves 40% accuracy on SWE-Bench Verified. While their dataset and agent trajectories are publicly accessible, the model is not.

In comparison, with a comparable dataset size, our SWE-Gym has executable feedback, avoids potential dataset bias through manual configuration of environments, while providing comprehensive analysis of agent and verifier training, their scaling behaviors, and positive results on agent self-improvement. Our system achieves competitive results with significantly lower compute and a smaller model size (32B vs 72B). Lastly, we open source all artifacts of the project, including dataset, model weights, agent trajectory data and the training pipeline.

<table><tbody><tr><th>Model</th><td colspan="2">SWE-Bench</td><td colspan="2">Openness</td></tr><tr><th>Name, Model Size</th><td>Lite</td><td>Verified</td><td>Model</td><td>Environment</td></tr><tr><th><sup><a href="#fn:19">19</a></sup>, 72B</th><td>22.0</td><td>30.2</td><td>✓</td><td>✗</td></tr><tr><th><sup><a href="#fn:9">9</a></sup> Agent and Verifier, 72B</th><td>-</td><td>40.6</td><td>✗</td><td>✓</td></tr><tr><th>Our SWE-Gym Agent and Verifier, 32B</th><td>26.0</td><td>32.0</td><td>✓</td><td>✓</td></tr></tbody></table>

Table 6: Comparison of model performance on SWE-Bench benchmark and if the model weights and environments are publically accessible (openness).

## Appendix B Experiment Details

<table><thead><tr><th></th><th>Original</th><th>Dedup.</th><th colspan="2">Sorted by Random (Dedup.)</th><th colspan="2">Sorted by Repo (Dedup.)</th></tr><tr><th></th><th></th><th></th><th>First 25%</th><th>First 50%</th><th>First 25%</th><th>First 50%</th></tr></thead><tbody><tr><th>getmoto/moto</th><td>155</td><td>72</td><td>12</td><td>33</td><td>0</td><td>46</td></tr><tr><th>Project-MONAI/MONAI</th><td>95</td><td>53</td><td>17</td><td>25</td><td>53</td><td>53</td></tr><tr><th>pandas-dev/pandas</th><td>70</td><td>61</td><td>14</td><td>30</td><td>0</td><td>0</td></tr><tr><th>python/mypy</th><td>46</td><td>27</td><td>7</td><td>12</td><td>0</td><td>0</td></tr><tr><th>dask/dask</th><td>45</td><td>29</td><td>8</td><td>17</td><td>6</td><td>29</td></tr><tr><th>iterative/dvc</th><td>36</td><td>24</td><td>8</td><td>12</td><td>0</td><td>0</td></tr><tr><th>conan-io/conan</th><td>20</td><td>12</td><td>1</td><td>7</td><td>12</td><td>12</td></tr><tr><th>pydantic/pydantic</th><td>11</td><td>7</td><td>2</td><td>4</td><td>0</td><td>0</td></tr><tr><th>facebookresearch/hydra</th><td>7</td><td>5</td><td>2</td><td>5</td><td>0</td><td>5</td></tr><tr><th>bokeh/bokeh</th><td>3</td><td>2</td><td>1</td><td>1</td><td>2</td><td>2</td></tr><tr><th>modin-project/modin</th><td>3</td><td>2</td><td>1</td><td>1</td><td>0</td><td>0</td></tr><tr><th>Total</th><td>491</td><td>294</td><td>73</td><td>147</td><td>73</td><td>147</td></tr></tbody></table>

Table 7: Distribution of success trajectories used in training-time scaling experiments (§4.2). Dedup. denotes that the trajectories are deduplicated by randomly select ONE success trajectory per instance ID; Sorted by random (repo) X% (Dedup.) denotes a subset of trajectories taken from the first X% from dedup. instances that are sorted randomly (by repository name).

<table><thead><tr><th></th><th></th><th></th><th></th><th></th><th></th><th colspan="7">Percentiles</th><th></th></tr><tr><th></th><th>Resolved</th><th>Count</th><th>Mean</th><th>Std</th><th>Min</th><th>Max</th><th>5%</th><th>10%</th><th>25%</th><th>50%</th><th>75%</th><th>90%</th><th>95%</th></tr></thead><tbody><tr><td rowspan="2">Num. of Messages</td><td>✗</td><td><math><semantics><mn>5,557.0</mn> <annotation>5,557.0</annotation></semantics></math></td><td><math><semantics><mn>39.2</mn> <annotation>39.2</annotation></semantics></math></td><td><math><semantics><mn>31.9</mn> <annotation>31.9</annotation></semantics></math></td><td><math><semantics><mn>7.0</mn> <annotation>7.0</annotation></semantics></math></td><td><math><semantics><mn>101.0</mn> <annotation>101.0</annotation></semantics></math></td><td><math><semantics><mn>9.0</mn> <annotation>9.0</annotation></semantics></math></td><td><math><semantics><mn>9.0</mn> <annotation>9.0</annotation></semantics></math></td><td><math><semantics><mn>9.0</mn> <annotation>9.0</annotation></semantics></math></td><td><math><semantics><mn>29.0</mn> <annotation>29.0</annotation></semantics></math></td><td><math><semantics><mn>61.0</mn> <annotation>61.0</annotation></semantics></math></td><td><math><semantics><mn>100.0</mn> <annotation>100.0</annotation></semantics></math></td><td><math><semantics><mn>101.0</mn> <annotation>101.0</annotation></semantics></math></td></tr><tr><td>✓</td><td><math><semantics><mn>491.0</mn> <annotation>491.0</annotation></semantics></math></td><td><math><semantics><mn>39.9</mn> <annotation>39.9</annotation></semantics></math></td><td><math><semantics><mn>19.9</mn> <annotation>19.9</annotation></semantics></math></td><td><math><semantics><mn>13.0</mn> <annotation>13.0</annotation></semantics></math></td><td><math><semantics><mn>101.0</mn> <annotation>101.0</annotation></semantics></math></td><td><math><semantics><mn>19.0</mn> <annotation>19.0</annotation></semantics></math></td><td><math><semantics><mn>21.0</mn> <annotation>21.0</annotation></semantics></math></td><td><math><semantics><mn>25.0</mn> <annotation>25.0</annotation></semantics></math></td><td><math><semantics><mn>33.0</mn> <annotation>33.0</annotation></semantics></math></td><td><math><semantics><mn>47.5</mn> <annotation>47.5</annotation></semantics></math></td><td><math><semantics><mn>65.0</mn> <annotation>65.0</annotation></semantics></math></td><td><math><semantics><mn>87.0</mn> <annotation>87.0</annotation></semantics></math></td></tr><tr><td rowspan="2">Num. of Tokens</td><td>✗</td><td><math><semantics><mn>5,557.0</mn> <annotation>5,557.0</annotation></semantics></math></td><td><math><semantics><mn>17,218.3</mn> <annotation>17,218.3</annotation></semantics></math></td><td><math><semantics><mn>17,761.6</mn> <annotation>17,761.6</annotation></semantics></math></td><td><math><semantics><mn>1,615.0</mn> <annotation>1,615.0</annotation></semantics></math></td><td><math><semantics><mn>167,834.0</mn> <annotation>167,834.0</annotation></semantics></math></td><td><math><semantics><mn>1,833.0</mn> <annotation>1,833.0</annotation></semantics></math></td><td><math><semantics><mn>1,907.0</mn> <annotation>1,907.0</annotation></semantics></math></td><td><math><semantics><mn>2,268.0</mn> <annotation>2,268.0</annotation></semantics></math></td><td><math><semantics><mn>12,305.0</mn> <annotation>12,305.0</annotation></semantics></math></td><td><math><semantics><mn>26,434.0</mn> <annotation>26,434.0</annotation></semantics></math></td><td><math><semantics><mn>41,182.2</mn> <annotation>41,182.2</annotation></semantics></math></td><td><math><semantics><mn>51,780.6</mn> <annotation>51,780.6</annotation></semantics></math></td></tr><tr><td>✓</td><td><math><semantics><mn>491.0</mn> <annotation>491.0</annotation></semantics></math></td><td><math><semantics><mn>18,578.5</mn> <annotation>18,578.5</annotation></semantics></math></td><td><math><semantics><mn>11,361.4</mn> <annotation>11,361.4</annotation></semantics></math></td><td><math><semantics><mn>2,560.0</mn> <annotation>2,560.0</annotation></semantics></math></td><td><math><semantics><mn>81,245.0</mn> <annotation>81,245.0</annotation></semantics></math></td><td><math><semantics><mn>5,813.0</mn> <annotation>5,813.0</annotation></semantics></math></td><td><math><semantics><mn>8,357.0</mn> <annotation>8,357.0</annotation></semantics></math></td><td><math><semantics><mn>11,559.5</mn> <annotation>11,559.5</annotation></semantics></math></td><td><math><semantics><mn>15,999.0</mn> <annotation>15,999.0</annotation></semantics></math></td><td><math><semantics><mn>22,040.5</mn> <annotation>22,040.5</annotation></semantics></math></td><td><math><semantics><mn>31,632.0</mn> <annotation>31,632.0</annotation></semantics></math></td><td><math><semantics><mn>39,512.5</mn> <annotation>39,512.5</annotation></semantics></math></td></tr></tbody></table>

Table 8: Statistics of SWE-Gym-sampled trajectories. We use the tokenizer from Qwen-2.5-Coder-Instruct-7B to estimate the number of tokens.

<table><thead><tr><th>Agent</th><th>Model</th><th>Model Size</th><th>Training Data</th><th>Resolved <math><semantics><mrow><mo>(</mo><mo>%</mo><mo>)</mo></mrow> <annotation>(\%)</annotation></semantics></math></th></tr><tr><th colspan="5">SWE-Bench Verified (500 instances)</th></tr></thead><tbody><tr><th>RAG</th><td>SWE-Llama <sup><a href="#fn:16">16</a></sup></td><td>7B</td><td>10K instances</td><td>1.4</td></tr><tr><th>RAG</th><td>SWE-Llama <sup><a href="#fn:16">16</a></sup></td><td>13B</td><td>10K instances</td><td>1.2</td></tr><tr><th>Lingma Agent <sup><a href="#fn:19">19</a></sup></th><td>Lingma SWE-GPT (v0925)</td><td>7B</td><td>90K PRs from 4K repos</td><td>18.2</td></tr><tr><th>Lingma Agent <sup><a href="#fn:19">19</a></sup></th><td>Lingma SWE-GPT (v0925)</td><td>72B</td><td>90K PRs from 4K repos</td><td>28.8</td></tr><tr><th>OpenHands <sup><a href="#fn:35">35</a></sup> (Ours)</th><td>fine-tuned Qwen2.5-Coder-Instruct</td><td>32B</td><td>491 agent trajectories from 11 repos</td><td>20.6</td></tr><tr><th>OpenHands w/ Verifier <sup><a href="#fn:35">35</a></sup> (Ours)</th><td>fine-tuned Qwen2.5-Coder-Instruct</td><td>32B (Agent & Verifier)</td><td><math><semantics><mn>491</mn> <annotation>491</annotation></semantics></math> agent trajectories from 11 repos for agent + <math><semantics><mrow><mn>1318</mn> <mo>×</mo> <mn>2</mn></mrow> <annotation>1318\times 2</annotation></semantics></math> success/failure agent trajectories for verifier</td><td>32.0</td></tr></tbody></table>

Table 9: Performance comparison with SWE-Bench [^16] baselines with publicly accessible weights. Data source: [https://www.swebench.com/](https://www.swebench.com/), Accessed on Dec 21, 2024.

### B.1 Mean and Variance for Pass@N and Best@N.

We mostly follow [^18] for obtaining the mean and variance for the Pass@N and Best@N curve. Given a total of M rounds of rollouts, for $N<M$, we calculate the mean and variance across 100 randomly selected sub-samples of size $N$ from the $M$ rollouts. For the OpenHands CodeActAgent inference-time scaling curve at §4, we exclude this calculation for N=1, as we use a temperature of 0 for the first attempt.

### B.2 Training Details.

OpenHands Agent Experiment. We use torchtune [^24] for full parameter fine-tuning with a learning rate of 1e-4, maximum 5 epochs, global batch size of 8, max context length of 32768. We fine-tuned both 7B, 14B, and 32B variant of the model, and experiments were performed with 2-8x NVIDIA H100 80G GPU on modal [^20]. The only exception is in the main experiment of §4.1.1, where we use LoRA [^12] (29.8% $@8$) via Unsloth library [^32] to train the verifier for max 2 epochs, while other hyper-parameter stays the same.

MoatlessTools Agent Experiment. All MoatlessTools models are trained with a context window of 10240. For experiments with the 7B model, we use torchtune to train the policy model with full-finetuning using 4 H100 GPUs. We set batch size to 8, learning rate to $2\times 10^{-5}$, and train for 5 epochs.

For the 32B model, we use Unsloth [^32] with a single H100 GPU for LoRA fine-tuning. We set the number of epochs to 5, batch size to 8, LoRA rank to 64, and learning rate to $5\times 10^{-4}$. We use the same configuration for verifier training.

For MoatlessAgent experiments, we serve the agent with FP8 quantization for improved throughput, which we found to have minimal effects on model performance.

### B.3 Details of OpenHands Trajectory Sampling

Trajectory Set Sampled from Model Sampled on Dataset Temperature Max Turns Success trajectories $D_{0}$ gpt-4o-2024-08-06 SWE-Gym Lite 0 30 19 ($8.26\%$) (Cumulative) Total $D_{0}$ $\mathbf{19}$ $D_{1}\setminus D_{0}$ gpt-4o-2024-08-06 SWE-Gym Lite 0.2 30 11 ($4.78\%$) gpt-4o-2024-08-06 SWE-Gym Lite 0.3 30 17 ($7.39\%$) gpt-4o-2024-08-06 SWE-Gym Lite 0.4 30 21 ($9.13\%$) gpt-4o-2024-08-06 SWE-Gym Lite 0.5 30 18 ($7.83\%$) gpt-4o-2024-08-06 SWE-Gym Lite 0.8 30 20 ($8.70\%$) (Cumulative) Total $D_{1}$ $\mathbf{106}$ $D_{2}\setminus D_{1}$ gpt-4o-2024-08-06 SWE-Gym Lite 0 50 19 ($8.26\%$) claude-3-5-sonnet-20241022 SWE-Gym Lite 0 50 67 ($29.1\%$) gpt-4o-2024-08-06 SWE-Gym Full 0 50 <sup>∗</sup> 111 ($4.55\%$) gpt-4o-2024-08-06 SWE-Gym Full 1 50 188 ($7.71\%$) (Cumulative) Total $D_{2}$ $\mathbf{491}$

- Run into infrastructure-related error where some instances failed to complete, this number might be under estimate of actual number of success trajectories.

Table 10: Summary of trajectories sampled from SWE-Gym.

As detailed in Tab. 10, we collect a few sets of trajectories for fine-tuning experiments. We collect dataset $D_{0}$ by sample gpt-4o-2024-08-06 on SWE-Gym Lite with temperature 0 and collected 19 trajectories that eventually solve the task (evaluated by unit test in SWE-Gym). We then varied the temperatures (setting t={0.2, 0.3, 0.4, 0.5, 0.8}) and sample on SWE-Gym Lite. Combining these instances with $D_{0}$, we get 106 trajectories that solve the given problem ($D_{1}$). We set the maximum number of turns to be 30 for both $D_{0}$ and $D_{1}$. To experiment on the effect of max turn, we set max number of turns to 50 and sample gpt-4o-2024-08-06 (19 resolved out of 230) and claude-3-5-sonnet-20241022 (67 resolved out of 230) with temperature 0 on SWE-Gym Lite, and sample gpt-4o-2024-08-06 (temperature t={0, 1}) on SWE-Gym full set (in total 299 resolved out of 4876 instances). This gives us in in total 106 + 19 + 67 + 299 = 491 success trajectories, which forms our final training trajectories $D_{2}$.

### B.4 MoatlessTools ORM Prompt

The following is a pseudo-code that generates a prompt for MoatlessTools Verifier (ORM), which is modified from [^44]. Unlike [^44], which relies on proprietary models like Claude-3.5-Sonnet for context extraction, we obtain context directly from the agent’s trajectory being evaluated.

⬇

SYSTEM\_MESSAGE = ”””You are an expert in python for software engineering and code review. Your responsibility is to review the patches generated by language models to fix some issues and provide feedback on the quality of their code.”””

USER\_MESSAGE=”””I want you to evaluate an LLM-generated candidate patch that tries to resolve an issue in a codebase.

To assist you in this task, you are provided with the following information:

\- You are given an issue text on a github repository (wrapped with <issue\_description></issue\_description>).

\- You are also given some identified code spans that are relevant to the issue.

Each code span is wrapped with <code\_span file\_path=FILE\_PATH span\_id=SPAN\_ID></code\_span> tags, where FILE\_PATH is the path to the file containing the code span, and SPAN\_ID is the unique identifier for the code span.

Each code span also comes with the line numbers for you to better understand the context. It’s possible that the code span are not sufficient to fix the issue, adjust your score accordingly.

\- You are given the candidate patch that tries to resolve the target issue.

For your convenience, you are given the hunks of original code and the code after applying the patch.

The code before the patch is wrapped with <before\_patch></before\_patch> and the code after the patch is wrapped with <after\_patch></after\_patch>.

Note that the file names in before\_patch starts with ’a/’ and the file names in after\_patch starts with ’b/’.

<issue\_description>

{issue\_text}

</issue\_description>

<before\_patch>

{before\_patch}

</before\_patch>

<after\_patch>

{after\_patch}

</after\_patch>

{code\_spans}

Response in ”True” or ”False” for whether the patch has resolved the issue.”””

### B.5 OpenHands ORM Prompt

The following is a pseudo-code that generates a prompt for OpenHands Verifier (ORM).

⬇

SYSTEM\_MESSAGE = ”’You are an expert judge evaluating AI assistant interactions. Your task is to determine if the assistant successfully resolved the user’s request.

Key evaluation criteria:

1\. Did the assistant complete the main task requested by the user?

2\. Did the assistant handle all edge cases and requirements specified?

3\. Were there any errors or issues in the final solution?

4\. Did the assistant verify the solution works as intended?

Respond only with ”\<judgement>YES\</judgement>” or ”\<judgement>NO\</judgement>”.”’

USER\_MESSAGE = ”’Please evaluate the following interaction between an AI assistant and a user:

\=== INTERACTION LOG ===

”’ + traj\_str + ”’

\=== END INTERACTION ===

Based on the above interaction, did the assistant successfully resolve the user’s initial request? Respond with YES or NO.”’

messages = \[

{’role’: ’system’, ’content’: SYSTEM\_MESSAGE},

{’role’: ’user’, ’content’: USER\_MESSAGE},

{’role’: ’assistant’, ’content’: ’\<judgement>’ + (”YES” if resolved else ”NO”) + ’\</judgement>’}

\]

The last assistant messages that contains judgement is only provided during training time. At inference time, the trained verifier is responsible predicting the probability of ‘Yes’ and ‘No’.

[^1]: Akkaya, I., Andrychowicz, M., Chociej, M., Litwin, M., McGrew, B., Petron, A., Paino, A., Plappert, M., Powell, G., Ribas, R., et al. Solving rubik’s cube with a robot hand. *arXiv preprint arXiv:1910.07113*, 2019.

[^2]: Badertdinov, I., Trofimova, M., Anapolskiy, Y., Abramov, S., Zainullina, K., Golubev, A., Polezhaev, S., Litvintseva, D., Karasik, S., Fisin, F., Skvortsov, S., Nekrashevich, M., Shevtsov, A., and Yangel, B. Scaling data collection for training software engineering agents. *Nebius blog*, 2024.

[^3]: Bai, H., Zhou, Y., Cemri, M., Pan, J., Suhr, A., Levine, S., and Kumar, A. Digirl: Training in-the-wild device-control agents with autonomous reinforcement learning. *ArXiv*, abs/2406.11896, 2024. URL [https://api.semanticscholar.org/CorpusID:270562229](https://api.semanticscholar.org/CorpusID:270562229).

[^4]: Brown, B., Juravsky, J., Ehrlich, R., Clark, R., Le, Q. V., R’e, C., and Mirhoseini, A. Large language monkeys: Scaling inference compute with repeated sampling. *ArXiv*, abs/2407.21787, 2024. URL [https://api.semanticscholar.org/CorpusID:271571035](https://api.semanticscholar.org/CorpusID:271571035).

[^5]: Chen, B., Shu, C., Shareghi, E., Collier, N., Narasimhan, K., and Yao, S. Fireact: Toward language agent fine-tuning. *ArXiv*, abs/2310.05915, 2023. URL [https://api.semanticscholar.org/CorpusID:263829338](https://api.semanticscholar.org/CorpusID:263829338).

[^6]: Chen, D., Lin, S., Zeng, M., Zan, D., Wang, J.-G., Cheshkov, A., Sun, J., Yu, H., Dong, G., Aliev, A., Wang, J., Cheng, X., Liang, G., Ma, Y., Bian, P., Xie, T., and Wang, Q. Coder: Issue resolving with multi-agent and task graphs. *CoRR in ArXiv*, abs/2406.01304, 2024.

[^7]: Chen, M., Tworek, J., Jun, H., Yuan, Q., Pondé, H., Kaplan, J., Edwards, H., Burda, Y., Joseph, N., Brockman, G., Ray, A., Puri, R., Krueger, G., Petrov, M., Khlaaf, H., Sastry, G., Mishkin, P., Chan, B., Gray, S., Ryder, N., Pavlov, M., Power, A., Kaiser, L., Bavarian, M., Winter, C., Tillet, P., Such, F. P., Cummings, D. W., Plappert, M., Chantzis, F., Barnes, E., Herbert-Voss, A., Guss, W. H., Nichol, A., Babuschkin, I., Balaji, S., Jain, S., Carr, A., Leike, J., Achiam, J., Misra, V., Morikawa, E., Radford, A., Knight, M. M., Brundage, M., Murati, M., Mayer, K., Welinder, P., McGrew, B., Amodei, D., McCandlish, S., Sutskever, I., and Zaremba, W. Evaluating large language models trained on code. *ArXiv*, abs/2107.03374, 2021. URL [https://api.semanticscholar.org/CorpusID:235755472](https://api.semanticscholar.org/CorpusID:235755472).

[^8]: Cobbe, K., Kosaraju, V., Bavarian, M., Chen, M., Jun, H., Kaiser, L., Plappert, M., Tworek, J., Hilton, J., Nakano, R., Hesse, C., and Schulman, J. Training verifiers to solve math word problems. *ArXiv*, abs/2110.14168, 2021. URL [https://api.semanticscholar.org/CorpusID:239998651](https://api.semanticscholar.org/CorpusID:239998651).

[^9]: Golubev, A., Polezhaev, S., Zainullina, K., Trofimova, M., Badertdinov, I., Anapolskiy, Y., Litvintseva, D., Karasik, S., Fisin, F., Skvortsov, S., Nekrashevich, M., Shevtsov, A., Abramov, S., and Yangel, B. Leveraging training and search for better software engineering agents. *Nebius blog*, 2024. https://nebius.com/blog/posts/training-and-search-for-software-engineering-agents.

[^10]: Hendrycks, D., Basart, S., Kadavath, S., Mazeika, M., Arora, A., Guo, E., Burns, C., Puranik, S., He, H., Song, D., and Steinhardt, J. Measuring coding challenge competence with APPS. In Vanschoren, J. and Yeung, S. (eds.), *Proceedings of the Neural Information Processing Systems Track on Datasets and Benchmarks 1, NeurIPS Datasets and Benchmarks 2021, December 2021, virtual*, 2021a. URL [https://datasets-benchmarks-proceedings.neurips.cc/paper/2021/hash/c24cd76e1ce41366a4bbe8a49b02a028-Abstract-round2.html](https://datasets-benchmarks-proceedings.neurips.cc/paper/2021/hash/c24cd76e1ce41366a4bbe8a49b02a028-Abstract-round2.html).

[^11]: Hendrycks, D., Burns, C., Kadavath, S., Arora, A., Basart, S., Tang, E., Song, D. X., and Steinhardt, J. Measuring mathematical problem solving with the math dataset. *ArXiv*, abs/2103.03874, 2021b. URL [https://api.semanticscholar.org/CorpusID:232134851](https://api.semanticscholar.org/CorpusID:232134851).

[^12]: Hu, E. J., Shen, Y., Wallis, P., Allen-Zhu, Z., Li, Y., Wang, S., Wang, L., and Chen, W. Lora: Low-rank adaptation of large language models. In *The Tenth International Conference on Learning Representations, ICLR 2022, Virtual Event, April 25-29, 2022*. OpenReview.net, 2022. URL [https://openreview.net/forum?id=nZeVKeeFYf9](https://openreview.net/forum?id=nZeVKeeFYf9).

[^13]: Hui, B., Yang, J., Cui, Z., Yang, J., Liu, D., Zhang, L., Liu, T., Zhang, J., Yu, B., Dang, K., et al. Qwen2. 5-coder technical report. *arXiv preprint arXiv:2409.12186*, 2024a.

[^14]: Hui, B., Yang, J., Cui, Z., Yang, J., Liu, D., Zhang, L., Liu, T., Zhang, J., Yu, B., Dang, K., et al. Qwen2. 5-coder technical report. *arXiv preprint arXiv:2409.12186*, 2024b.

[^15]: Jain, N., Shetty, M., Zhang, T., Han, K., Sen, K., and Stoica, I. R2E: turning any github repository into a programming agent environment. In *Forty-first International Conference on Machine Learning, ICML 2024, Vienna, Austria, July 21-27, 2024*. OpenReview.net, 2024. URL [https://openreview.net/forum?id=kXHgEYFyf3](https://openreview.net/forum?id=kXHgEYFyf3).

[^16]: Jimenez, C. E., Yang, J., Wettig, A., Yao, S., Pei, K., Press, O., and Narasimhan, K. R. Swe-bench: Can language models resolve real-world github issues? In *The Twelfth International Conference on Learning Representations, ICLR 2024, Vienna, Austria, May 7-11, 2024*. OpenReview.net, 2024. URL [https://openreview.net/forum?id=VTF8yNQM66](https://openreview.net/forum?id=VTF8yNQM66).

[^17]: Li, Y., Choi, D., Chung, J., Kushman, N., Schrittwieser, J., Leblond, R., Eccles, T., Keeling, J., Gimeno, F., Dal Lago, A., et al. Competition-level code generation with alphacode. *Science*, 378(6624):1092–1097, 2022.

[^18]: Lightman, H., Kosaraju, V., Burda, Y., Edwards, H., Baker, B., Lee, T., Leike, J., Schulman, J., Sutskever, I., and Cobbe, K. Let’s verify step by step. *ArXiv*, abs/2305.20050, 2023. URL [https://api.semanticscholar.org/CorpusID:258987659](https://api.semanticscholar.org/CorpusID:258987659).

[^19]: Ma, Y., Cao, R., Cao, Y., Zhang, Y., Chen, J., Liu, Y., Liu, Y., Li, B., Huang, F., and Li, Y. Lingma swe-gpt: An open development-process-centric language model for automated software improvement. *arXiv preprint arXiv:2411.00622*, 2024.

[^20]: Modal. Modal: High-performance AI infrastructure. [https://modal.com/](https://modal.com/), 2024. Accessed: 2024-12-18.

[^21]: Neubig, G. and Wang, X. Evaluation of LLMs as Coding Agents on SWE-Bench (at 30x Speed!). *All Hands AI blog*, 2024. URL [https://www.all-hands.dev/blog/evaluation-of-llms-as-coding-agents-on-swe-bench-at-30x-speed](https://www.all-hands.dev/blog/evaluation-of-llms-as-coding-agents-on-swe-bench-at-30x-speed).

[^22]: Ouyang, L., Wu, J., Jiang, X., Almeida, D., Wainwright, C., Mishkin, P., Zhang, C., Agarwal, S., Slama, K., Ray, A., et al. Training language models to follow instructions with human feedback. *Advances in neural information processing systems*, 35:27730–27744, 2022.

[^23]: Pan, J., Zhang, Y., Tomlin, N., Zhou, Y., Levine, S., and Suhr, A. Autonomous evaluation and refinement of digital agents. *ArXiv*, abs/2404.06474, 2024. URL [https://api.semanticscholar.org/CorpusID:269009430](https://api.semanticscholar.org/CorpusID:269009430).

[^24]: PyTorch Team. torchtune: PyTorch native post-training library. [https://github.com/pytorch/torchtune](https://github.com/pytorch/torchtune), 2024.

[^25]: Schulman, J., Wolski, F., Dhariwal, P., Radford, A., and Klimov, O. Proximal policy optimization algorithms. *ArXiv*, abs/1707.06347, 2017. URL [https://api.semanticscholar.org/CorpusID:28695052](https://api.semanticscholar.org/CorpusID:28695052).

[^26]: Shao, Z., Wang, P., Zhu, Q., Xu, R., Song, J., Bi, X., Zhang, H., Zhang, M., Li, Y., Wu, Y., et al. Deepseekmath: Pushing the limits of mathematical reasoning in open language models. *arXiv preprint arXiv:2402.03300*, 2024.

[^27]: Silver, D., Hubert, T., Schrittwieser, J., Antonoglou, I., Lai, M., Guez, A., Lanctot, M., Sifre, L., Kumaran, D., Graepel, T., Lillicrap, T. P., Simonyan, K., and Hassabis, D. Mastering chess and shogi by self-play with a general reinforcement learning algorithm. *ArXiv*, abs/1712.01815, 2017. URL [https://api.semanticscholar.org/CorpusID:33081038](https://api.semanticscholar.org/CorpusID:33081038).

[^28]: Snell, C. B., Kostrikov, I., Su, Y., Yang, M., and Levine, S. Offline rl for natural language generation with implicit language q learning. *ArXiv*, abs/2206.11871, 2022. URL [https://api.semanticscholar.org/CorpusID:249954054](https://api.semanticscholar.org/CorpusID:249954054).

[^29]: Tao, N., Ventresque, A., Nallur, V., and Saber, T. Enhancing program synthesis with large language models using many-objective grammar-guided genetic programming. *Algorithms*, 17(7):287, 2024. doi: 10.3390/A17070287. URL [https://doi.org/10.3390/a17070287](https://doi.org/10.3390/a17070287).

[^30]: Team, Q. Qwen2.5: A party of foundation models, September 2024. URL [https://qwenlm.github.io/blog/qwen2.5/](https://qwenlm.github.io/blog/qwen2.5/).

[^31]: Tong, Y., Zhang, X., Wang, R., Wu, R. M., and He, J. Dart-math: Difficulty-aware rejection tuning for mathematical problem-solving. *ArXiv*, abs/2407.13690, 2024. URL [https://api.semanticscholar.org/CorpusID:271270574](https://api.semanticscholar.org/CorpusID:271270574).

[^32]: Unsloth Team. Easily finetune and train LLMs. Get faster with unsloth. [https://unsloth.ai/](https://unsloth.ai/), 2024.

[^33]: Wang, P., Li, L., Shao, Z., Xu, R., Dai, D., Li, Y., Chen, D., Wu, Y., and Sui, Z. Math-shepherd: Verify and reinforce LLMs step-by-step without human annotations. In Ku, L.-W., Martins, A., and Srikumar, V. (eds.), *Proceedings of the 62nd Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers)*, pp. 9426–9439, Bangkok, Thailand, August 2024a. Association for Computational Linguistics. doi: 10.18653/v1/2024.acl-long.510. URL [https://aclanthology.org/2024.acl-long.510](https://aclanthology.org/2024.acl-long.510).

[^34]: Wang, X., Chen, Y., Yuan, L., Zhang, Y., Li, Y., Peng, H., and Ji, H. Executable code actions elicit better LLM agents. In *Forty-first International Conference on Machine Learning, ICML 2024, Vienna, Austria, July 21-27, 2024*. OpenReview.net, 2024b. URL [https://openreview.net/forum?id=jJ9BoXAfFa](https://openreview.net/forum?id=jJ9BoXAfFa).

[^35]: Wang, X., Li, B., Song, Y., Xu, F. F., Tang, X., Zhuge, M., Pan, J., Song, Y., Li, B., Singh, J., Tran, H. H., Li, F., Ma, R., Zheng, M., Qian, B., Shao, Y., Muennighoff, N., Zhang, Y., Hui, B., Lin, J., Brennan, R., Peng, H., Ji, H., and Neubig, G. OpenHands: An Open Platform for AI Software Developers as Generalist Agents. *CoRR in ArXiv*, abs/2407.16741, 2024c.

[^36]: Wu, Z., Bai, H., Zhang, A., Gu, J., Vinod Vydiswaran, V., Jaitly, N., and Zhang, Y. Divide-or-conquer? which part should you distill your llm? *ArXiv*, 2024.

[^37]: Xi, Z., Ding, Y., Chen, W., Hong, B., Guo, H., Wang, J., Yang, D., Liao, C., Guo, X., He, W., Gao, S., Chen, L., Zheng, R., Zou, Y., Gui, T., Zhang, Q., Qiu, X., Huang, X., Wu, Z., and Jiang, Y.-G. Agentgym: Evolving large language model-based agents across diverse environments. *ArXiv*, abs/2406.04151, 2024. URL [https://api.semanticscholar.org/CorpusID:270285866](https://api.semanticscholar.org/CorpusID:270285866).

[^38]: Xia, C. S., Deng, Y., Dunn, S., and Zhang, L. Agentless: Demystifying llm-based software engineering agents. *CoRR*, abs/2407.01489, 2024. doi: 10.48550/ARXIV.2407.01489. URL [https://doi.org/10.48550/arXiv.2407.01489](https://doi.org/10.48550/arXiv.2407.01489).

[^39]: Yang, J., Jimenez, C. E., Wettig, A., Lieret, K., Yao, S., Narasimhan, K., and Press, O. Swe-agent: Agent-computer interfaces enable automated software engineering. *CoRR*, abs/2405.15793, 2024. doi: 10.48550/ARXIV.2405.15793. URL [https://doi.org/10.48550/arXiv.2405.15793](https://doi.org/10.48550/arXiv.2405.15793).

[^40]: Yao, S., Zhao, J., Yu, D., Du, N., Shafran, I., Narasimhan, K. R., and Cao, Y. React: Synergizing reasoning and acting in language models. In *The Eleventh International Conference on Learning Representations, ICLR 2023, Kigali, Rwanda, May 1-5, 2023*. OpenReview.net, 2023. URL [https://openreview.net/forum?id=WE\_vluYUL-X](https://openreview.net/forum?id=WE_vluYUL-X).

[^41]: Yuan, L., Cui, G., Wang, H., Ding, N., Wang, X., Deng, J., Shan, B., Chen, H., Xie, R., Lin, Y., Liu, Z., Zhou, B., Peng, H., Liu, Z., and Sun, M. Advancing LLM reasoning generalists with preference trees. *CoRR*, abs/2404.02078, 2024. doi: 10.48550/ARXIV.2404.02078. URL [https://doi.org/10.48550/arXiv.2404.02078](https://doi.org/10.48550/arXiv.2404.02078).

[^42]: Zeng, A., Liu, M., Lu, R., Wang, B., Liu, X., Dong, Y., and Tang, J. Agenttuning: Enabling generalized agent abilities for llms. In *Annual Meeting of the Association for Computational Linguistics*, 2023. URL [https://api.semanticscholar.org/CorpusID:264306101](https://api.semanticscholar.org/CorpusID:264306101).

[^43]: Zhai, Y., Bai, H., Lin, Z., Pan, J., Tong, S., Zhou, Y., Suhr, A., Xie, S., LeCun, Y., Ma, Y., and Levine, S. Fine-tuning large vision-language models as decision-making agents via reinforcement learning. *ArXiv*, abs/2405.10292, 2024. URL [https://api.semanticscholar.org/CorpusID:269790773](https://api.semanticscholar.org/CorpusID:269790773).

[^44]: Zhang, K., Yao, W., Liu, Z., Feng, Y., Liu, Z., Murthy, R., Lan, T., Li, L., Lou, R., Xu, J., Pang, B., Zhou, Y., Heinecke, S., Savarese, S., Wang, H., and Xiong, C. Diversity empowers intelligence: Integrating expertise of software engineering agents. *ArXiv*, abs/2408.07060, 2024a. URL [https://api.semanticscholar.org/CorpusID:271860093](https://api.semanticscholar.org/CorpusID:271860093).

[^45]: Zhang, Y., Ruan, H., Fan, Z., and Roychoudhury, A. Autocoderover: Autonomous program improvement. In *ISSTA*, 2024b.

[^46]: Zhao, W., Jiang, N., Lee, C., Chiu, J. T., Cardie, C., Gallé, M., and Rush, A. M. Commit0: Library generation from scratch, 2024. URL [https://arxiv.org/abs/2412.01769](https://arxiv.org/abs/2412.01769).

[^47]: Zheng, L., Yin, L., Xie, Z., Sun, C., Huang, J., Yu, C. H., Cao, S., Kozyrakis, C., Stoica, I., Gonzalez, J. E., Barrett, C., and Sheng, Y. Sglang: Efficient execution of structured language model programs, 2024a. URL [https://arxiv.org/abs/2312.07104](https://arxiv.org/abs/2312.07104).

[^48]: Zheng, T., Zhang, G., Shen, T., Liu, X., Lin, B. Y., Fu, J., Chen, W., and Yue, X. Opencodeinterpreter: Integrating code generation with execution and refinement. *ArXiv*, abs/2402.14658, 2024b. URL [https://api.semanticscholar.org/CorpusID:267782452](https://api.semanticscholar.org/CorpusID:267782452).

[^49]: Zhou, Y., Zanette, A., Pan, J., Levine, S., and Kumar, A. Archer: Training language model agents via hierarchical multi-turn rl. *ArXiv*, abs/2402.19446, 2024. URL [https://api.semanticscholar.org/CorpusID:268091206](https://api.semanticscholar.org/CorpusID:268091206).

[^50]: Örwall, A. Moatless Tool. [https://github.com/aorwall/moatless-tools](https://github.com/aorwall/moatless-tools), 2024. Accessed: 2024-10-22.