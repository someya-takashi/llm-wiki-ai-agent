---
title: "SWE-Lego: Pushing the Limits of Supervised Fine-tuning for Software Issue Resolving"
source: "https://ar5iv.labs.arxiv.org/html/2601.01426"
author:
published:
created: 2026-10-26
description: "We present SWE-Lego, a supervised fine-tuning (SFT) recipe designed to achieve state-of-the-art performance in software engineering (SWE) issue resolving. In contrast to prevalent methods that rely on complex training …"
tags:
  - "clippings"
---
Chaofan Tao Jierun Chen Yuxin Jiang Kaiqi Kou Shaowei Wang Ruoyu Wang Xiaohui Li    Sidi Yang Yiming Du Jianbo Dai Zhiming Mao Xinyu Wang Lifeng Shang Haoli Bai Affiliation: Huawei Technologies, NTU, HKU, CUHK [https://github.com/SWE-Lego/SWE-Lego](https://github.com/SWE-Lego/SWE-Lego) [https://huggingface.co/SWE-Lego](https://huggingface.co/SWE-Lego) Affiliation: Huawei Technologies, NTU, HKU, CUHK [https://github.com/SWE-Lego/SWE-Lego](https://github.com/SWE-Lego/SWE-Lego) [https://huggingface.co/SWE-Lego](https://huggingface.co/SWE-Lego)

###### Abstract

We present SWE-Lego, a supervised fine-tuning (SFT) recipe designed to achieve state-of-the-art performance in software engineering (SWE) issue resolving. In contrast to prevalent methods that rely on complex training paradigms (e.g., mid-training, SFT, reinforcement learning, and their combinations), we explore how to push the limits of a lightweight SFT-only approach for SWE tasks. SWE-Lego comprises three core building blocks, with key findings summarized as follows: 1) the SWE-Lego dataset, a collection of 32k high-quality task instances and 18k validated trajectories, combining real and synthetic data to complement each other in both quality and quantity; 2) a refined SFT procedure with error masking and a difficulty-based curriculum, which demonstrably improves action quality and overall performance. Empirical results show that with these two building bricks alone, the SFT can push SWE-Lego models to state-of-the-art performance among open-source models of comparable size on SWE-bench Verified: SWE-Lego-Qwen3-8B reaches 42.2%, and SWE-Lego-Qwen3-32B attains 52.6%. 3) We further evaluate and improve test-time scaling (TTS) built upon the SFT foundation. Based on a well-trained verifier, SWE-Lego models can be significantly boosted—for example, 42.2% $\rightarrow$ 49.6% and 52.6% $\rightarrow$ 58.8% under TTS@16 for the 8B and 32B models, respectively.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2601.01426/assets/model_performance_scatter.png)

(a) Performance comparisons.

## 1 Introduction

Recent advances of large language models (LLMs) have led to remarkable progress in autonomous software engineering (SWE) agents, capable of repository-scale issue resolving [^55] [^60] [^48]. Unlike short, single-file tasks [^12] [^3] [^8] [^23], SWE agents are able to navigate code repositories, faithfully reproduce failing tests, identify buggy files and lines, implement fixes, and validate within executable sandboxes [^52] [^24] [^38], which demands long-horizon reasoning, robust tool use, and multi-turn interaction abilities.

To equip LLMs with these capabilities, a common approach is supervised fine-tuning (SFT), which distills knowledge from stronger SWE agents. However, prior efforts often lack datasets that include execution environments and realistic bug instances needed to produce high-quality, diverse trajectories [^44] [^54] [^17]. Beyond SFT, performance on SWE tasks can be further improved via mid-training, but this typically requires substantially more computation and data [^10] [^56]. Alternatively, recent work applies reinforcement learning (RL) without a fixed teacher model or predefined SWE trajectories [^26] [^39] [^46], yet limited executable instances remain a bottleneck. In addition, RL is more computationally intensive, demands heavier training infrastructure, and is prone to training collapse due to hyperparameter sensitivity. These practical constraints motivate a critical question: *how far can we push a lightweight, SFT-only training recipe for SWE tasks?*

To investigate, we propose SWE-Lego, a simple yet powerful SFT framework for training SWE agents. We discover that with careful data design, training techniques, and test-time scaling for inference, SFT alone can achieve state-of-the-art performance, matching or surpassing more complex training paradigms under similar LLM scales, as shown in Figure 1. SWE-Lego comprises three core building blocks:

Empirically, SWE-Lego establishes new state-of-the-art results among open-source models on SWE-bench Verified [^18]: SWE-Lego-Qwen3-8B reaches 42.2%, and SWE-Lego-Qwen3-32B attains 52.6% using only SFT (Figure 1(a)). With test-time scaling, the performance rises to 49.6% and 58.8% under TTS@16, respectively. These results demonstrate that a meticulously crafted SFT pipeline can rival or exceed the performance of more complex and computationally expensive training regimes. Notably, our results are obtained without Git hacking, whereas prior reports are often inflated by such solution leakage. Ablations in Figure 1(b) show that our SWE-Lego dataset yields a substantial boost of +25.6%, refined SFT contributes an additional +3.8%, and TTS ultimately adds +6.2%, which together lift Qwen3-32B from 23.2% to 58.8%, confirming the effectiveness of each building blocks. We hope SWE-Lego can provide a reproducible and lightweight post-training paradigm for building effective SWE agents, and the associated data and models of this work will be open-sourced for future research in the community.

## 2 SWE-Lego Dataset: Combining Real-world and Synthetic Data

### 2.1 Overview

LLMs are increasingly integrated into autonomous agents for real-world SWE tasks. We adopt the SWE-Agent paradigm [^55], in which an LLM operates through a specialized agent-computer interface that allows it to autonomously navigate repositories, search and edit code, and execute tests to diagnose and resolve issues. In this context, a *SWE task instance* is usually defined as the concrete SWE issue within a codebase, comprising a natural-language problem description, test cases, a reference golden patch, and related artifacts. Besides, a *trajectory* is the complete sequence of actions taken by an agent and the interleaved observations from environments. To train capable SWE agents, *our primary goal is to curate a large-scale, diverse, and executable collection of task instances, together with high-quality, validated trajectories*.

| Dataset | Type | Executable | Repositories | Task Instances | Valid Trajectories |
| --- | --- | --- | --- | --- | --- |
| R2E-Gym-Train | Synthetic | Yes | 10 | 4.6k | 3.3k |
| SWE-Swiss | Synthetic | Yes | $<$ 600 | 10k | NA |
| SWE-Fixer | Real | No | 856 | 115k | NA |
| SWE-Gym | Real | Yes | 11 | 2.4k | 491 |
| SWE-bench-Train | Real | No | 37 | 19k | NA |
| SWE-rebench | Real | Yes | 3.5k | 21k | NA |
| SWE-smith | Real & Synthetic | Yes | 128 | 50k | 5k |
| SWE-Lego | Real & Synthetic | Yes | 3251 | 32.1k | 14.1k resolved + 4.0k semi-resolved |

Table 1: Comparison of public SWE issue-resolving datasets and our SWE-Lego dataset. The proposed dataset combines real and synthetic instances at scale, with executable environments and a large pool of validated trajectories, providing a stronger foundation for training SWE agents. Note that SWE-Swiss and SWE-Fixer release only sub-trajectories, and therefore, counts of full trajectories are not reported. “Semi-resolved” means the trajectories do not pass all test cases but have partial correctness, e.g., correct file localization.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2601.01426/assets/data-pipeline-v2.png)

Figure 2: Our SWE‑Lego pipeline comprises three stages: environment construction from over 3,000 repositories; hybrid task creation by combining real pull requests with synthetic bugs; and expert‑trajectory generation and curation for SFT.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2601.01426/assets/figures/trajectory_scaling.png)

(a) Scaling of Valid Trajectories

We adopts a hybrid data construction strategy that combines real-world and synthetic SWE task instances, as shown in Figure 2. This is motivated by the empirical observation that neither data source alone is sufficient: real-world instances, while authentic, are inherently limited in quantity given strict filtering criteria. On the other hand, synthetic instances, while scalable, lack the complexity of natural software repositories (Table 3). We therefore mix the real with synthetic instances, and apply rigorous generation and validation procedures to produce high-quality trainable trajectories.

The resulting SWE-Lego dataset comprises 32k executable task instances and 18k validated expert trajectories to date, as summarized in Table 1. Compared with existing open-source SWE datasets, SWE-Lego spans more repositories, offers more executable task instances, and contains substantially more validated trajectories. Moreover, scaling this hybrid corpus consistently yields better downstream performance on SWE-bench Verified, as evidenced in Figure 3. In the following subsections, we detail the pipeline used to construct the SWE-Lego dataset.

### 2.2 SWE Task Instance Creation and Validation

#### Repository Collection and Sandbox Construction.

The proposed dataset is built upon SWE-rebench [^4], a large collection of real-world executable repositories. We select over 3,000 Python-centric repositories with permissive licenses and active maintenance histories. To ensure reproducibility, we deploy a fully automated pipeline that parses configuration files (e.g., setup.py) to build Docker containers. Only repositories that successfully build and pass sanity tests are retained, serving as the verifiable “base plates” for our subsequent task construction.

#### Real-world and Synthetic Task Construction.

*Real-world tasks* are derived from resolved GitHub pull requests (PRs) paired with corresponding issue descriptions. They offer high authenticity with production-level bug complexity, but are often labor-intensive and limited in quantity due to the strict filtering criteria [^4] [^35] [^17] [^58]. Moreover, each real-world task requires a unique sandbox aligned with the pre-fix snapshot. While SWE-smith [^54] proposes a “PR mirror” method to generate real-world instances (prompting LLMs to revert PR edits via.diff plaintext instead of checking out the base commits), we find this approach yields lower-quality data compared to directly collecting PRs that satisfy SWE-rebench [^4] ’s filtering rules. Thus, we adopt the latter strategy for real-world task construction.

*Synthetic tasks*, by contrast, are generated via active bug injection following SWE-smith [^54], leveraging two core techniques: (i) *LLM Rewrite*: prompting models to rewrite code using only function headers and docstrings; (ii) *AST Reformulation*: extracting abstract syntax trees (ASTs) for classes/functions and applying random transformations, e.g., removing conditionals/loops, modifying operators or dependencies. Synthetic tasks enable high scalability and efficiency, as they are not constrained by the availability of filtered real PRs. Even repositories with no qualified PRs can be used to create large-scale tasks at low cost. Additionally, synthetic tasks from the same repository share a single sandbox environment, since all bugs are injected into the same codebase version.

These two data sources are complementary for scaling SWE tasks: real-world data provides *depth* (complexity and realism), while synthetic data provides *breadth* (quantity and coverage). Table 2 summarizes the core-field differences between the two, and additional implementation details are provided in Appendix A.1. Table 3 presents a quantitative comparison of complexity. We observe a stark contrast: real-world instances touch more files and change more lines than synthetic ones. This confirms that real-world data captures the sprawl of complex bugs, whereas synthetic data offers focused, surgical training signals and yields a higher rate of valid trajectories for training. Further analysis of issue categories is provided in Appendix A.2. Figure 3 demonstrates the effectiveness of this hybrid approach: for a fixed set of repositories, scaling synthetic data, e.g., from “real only” to “real + 5 synthetic insts./repo”, consistently improves both the number of valid expert trajectories and the resolve rate of the trained models. Together with Table 1, this shows that scaling executable hybrid data, rather than real instance alone, is key to obtaining richer supervision signals and higher downstream performance.

| Core Field | Real-World Tasks | Synthetic Tasks |
| --- | --- | --- |
| golden\_patch | Merged PR diff as the ground-truth fix. | Inverse the patch of the programmatically injected bug. |
| FAIL-TO-PASS & PASS-TO-PASS | Label of test sets that induced by running the original project tests before and after the merged patch. | Label of test sets that induced by running the same tests on buggy vs. restored (pre-injection) code. |
| problem\_statement | Original GitHub issue linked to the PR. | LLM-generated description conditioned on the golden patch, test files, and logs. |
| image\_name (Sandbox) | Per-PR sandbox tied to the exact pre-merge commit. | Shared sandbox for many injected bugs on the same base commit given the repository. |

Table 2: High-level comparison of how core fields are instantiated for real-world and synthetic SWE tasks in SWE-Lego. Both sources populate the same unified schema, but differ in data origin and sandbox granularity, leading to realistic yet costly real instances and scalable, controllable synthetic ones.

<table><tbody><tr><td>Category</td><td>Metric</td><td>Real-World Tasks</td><td>Synthetic Tasks</td><td>Real-World + Synthetic Tasks</td></tr><tr><td>Instances</td><td>–</td><td>18409</td><td>13710</td><td>32119</td></tr><tr><td>Issue Length</td><td>Words</td><td>135</td><td>168</td><td>149</td></tr><tr><td rowspan="3">Golden Patch</td><td>Files</td><td>3.7</td><td>1.0</td><td>2.6</td></tr><tr><td>Hunks</td><td>9.5</td><td>1.3</td><td>6.0</td></tr><tr><td>Lines</td><td>138.0</td><td>18.8</td><td>87.3</td></tr><tr><td rowspan="3">Test Cases</td><td>Fail-to-Pass</td><td>12.4</td><td>10.9</td><td>11.7</td></tr><tr><td>Pass-to-Pass</td><td>74.6</td><td>41.2</td><td>60.4</td></tr><tr><td>Total</td><td>87.0</td><td>52.1</td><td>72.1</td></tr><tr><td>Valid Trajectories</td><td>–</td><td>5010</td><td>9100</td><td>14110</td></tr></tbody></table>

Table 3: Characteristic comparisons of task instances. Real-world tasks are more complex, touching more files and modifying more lines, while synthetic tasks are more focused and yield more valid, fully resolving trajectories. The combined dataset covers a broader spectrum of difficulty.

### 2.3 Trajectory Rollout and Validation

With the hybrid task instances, we now roll out expert trajectories that are later used for SFT. We adopt the OpenHands scaffold (v0.53) [^45] with Qwen3-Coder-480B-A35B-Instruct [^51] as the teacher agent, and set the maximum number of interaction turns to 100. The model offers competitive coding and reasoning capabilities on challenging SWE tasks and, being open-weight, can be deployed locally at scale for trajectory distillation. During distillation, we found that the agent sometimes peeked at Git history or future commits, thereby cheating rather than faithfully resolving the issue. As showcased in Figure 4, we also observed avoidable tool-calling errors and tools that were ineffective in practice. To enhance trajectory quality, we propose and implement the following three practices.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2601.01426/assets/traj_generation_practices_cropped_v3.png)

Figure 4: Examples of problematic commands or tool interactions: (a) high‑risk commands that can cause Git hacking; (b) view\_range parameter mis-specification; (c) ineffective task\_tracker.

Preventing Git Hacking. Recently it is known by the SWE community that LLM agents can unexpectedly ‘‘hack’’ git metadata to locate the golden patch by directly inspecting commit logs <sup>1</sup>. A demonstrative example is shown Figure 4(a), and its associated full trajectory is listed in Appendix D. To prevent such Git hacking, we sanitize the repository history exposed to the agent. For real instances, we remove all commits and log messages dated after the issue creation date so that future fixes are invisible. For synthetic instances, since the buggy version of repository is ahead from the non-buggy version due to intentional bug injection, we remove the entire git history and all logs, exposing only a single snapshot of the buggy codebase. This forces the agent to genuinely reason about code and tests rather than reading answers from version control. As expected, we see in Table 5 that the valid-trajectory rate drops slightly from an inflated 31% to 30%, and the average number of turns increases a bit from 61.8 to 62.5. In addition, as mentioned in Section 5, we also report the results of SWE-bench Verified without Git hacking throughout this paper, unless specified otherwise.

Handling Malformed Tool Errors. With Qwen3-Coder-480B-A35B-Instruct, we observe frequent malformed calls to the str\_replace\_editor tool, e.g., passing a string to view\_range or specifying out-of-bounds line ranges (see Figure 4(b)), causing tool failures and a waste of interaction budget. To mitigate these errors, we apply lightweight post-hoc correction, i.e., parsing strings to integers and clipping ranges to valid spans, which enables the agent to inspect code more reliably. Without robust tool handling, the valid-trajectory rate decreases slightly from 30% to 29%, and the average number of turns increases from 62.5 to 66.6 (see Table 5).

Pruning Ineffective Tools. Recent work [^31] [^2] has begun integrating task-management tools such as task\_tracker for long-horizon planning and progress tracking. However, we find that Qwen3-Coder-480B-A35B-Instruct struggles to use them effectively, which often cause execution errors (see Figure 4(c)). We therefore discard this tool and restrict the tool set to four essential operations: execute\_bash, str\_replace\_editor, think, and finish, to keep trajectories streamlined, as evidenced by fewer interaction turns (from 65.5 to 62.5) and a higher valid-trajectory rate (from 27% to 30%) in Table 5.

#### Validation and Filtering.

Quality control is enforced through rigorous post-hoc validation. We classify trajectories as *resolved* if they pass all tests without regression. However, not all resolved trajectories are equal, nor are all unresolved ones useless. As shown in Table 5, our filtering strategy is two-fold: (i) we filter low-quality resolved trajectories (e.g., those that “cheat” by modifying tests), which yields a modest performance gain; (ii) we recycle semi-resolved trajectories, i.e., those that correctly locate the buggy file (with the recall rate of 100%) but fail to fix it. It results 4k additional trajectories for training. Adding these trajectories provides a clear boost (+1.2%), demonstrating that even partial successes contain valuable supervision for fault localization.

| Practice | Avg Turns | Valid Traj. Rate (%) |
| --- | --- | --- |
| All | 62.5 | 30 |
| w/o preventing Git hacking | 61.8 | 31 |
| w/o handling malformed tool errors | 66.6 | 29 |
| w/o pruning ineffective tools | 65.5 | 27 |

Table 4: Ablation of rollout practices. The valid trajectory rate is evaluated across 100 SWE-Lego instances.

| Data Strategy | \# Traj | Resolve Rate (%) |
| --- | --- | --- |
| All resolved | 14.6k | 40.4 |
| \- low-quality resolved | 14.1k | 41.0 |
| \+ semi-resolved | 18.1k | 42.2 |

Table 5: Ablation of filtering strategies on SWE-bench Verified.

<svg id="S2.SS3.SSS0.Px1.p2.pic1" height="276.5" overflow="visible" version="1.1" viewBox="0 0 600 276.5" width="600"><g style="--ltx-stroke-color:#000000;--ltx-fill-color:#000000;" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,276.5) matrix(1 0 0 -1 0 0)"><g style="--ltx-fill-color:#000000;" fill="#000000" fill-opacity="1.0"><path style="stroke:none" d="M 0 5.91 L 0 253.96 C 0 257.22 2.64 259.87 5.91 259.87 L 594.09 259.87 C 597.36 259.87 600 257.22 600 253.96 L 600 5.91 C 600 2.64 597.36 0 594.09 0 L 5.91 0 C 2.64 0 0 2.64 0 5.91 Z"></path></g><g style="--ltx-fill-color:#F2F2F2;" fill="#F2F2F2" fill-opacity="1.0"><path style="stroke:none" d="M 1.97 5.91 L 1.97 253.96 C 1.97 256.14 3.73 257.9 5.91 257.9 L 594.09 257.9 C 596.27 257.9 598.03 256.14 598.03 253.96 L 598.03 5.91 C 598.03 3.73 596.27 1.97 594.09 1.97 L 5.91 1.97 C 3.73 1.97 1.97 3.73 1.97 5.91 Z"></path></g><g transform="matrix(1.0 0.0 0.0 1.0 7.87 251.21)"><g style="--ltx-stroke-color:#000000;--ltx-fill-color:#000000;" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g style="--ltx-fill-color:#000000;" fill="#000000" fill-opacity="1.0"><path style="stroke:none" d="M 0 4.53 L 0 20.76 C 0 23.26 2.03 25.29 4.53 25.29 L 95.28 25.29 C 97.78 25.29 99.81 23.26 99.81 20.76 L 99.81 4.53 C 99.81 2.03 97.78 0 95.28 0 L 4.53 0 C 2.03 0 0 2.03 0 4.53 Z"></path></g><g style="--ltx-fill-color:#000000;" fill="#000000" fill-opacity="1.0"><path style="stroke:none" d="M 1.57 4.53 L 1.57 20.76 C 1.57 22.39 2.9 23.72 4.53 23.72 L 95.28 23.72 C 96.91 23.72 98.24 22.39 98.24 20.76 L 98.24 4.53 C 98.24 2.9 96.91 1.57 95.28 1.57 L 4.53 1.57 C 2.9 1.57 1.57 2.9 1.57 4.53 Z"></path></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.19)"><foreignObject style="--ltx-fo-width:4.78em;--ltx-fo-height:0.6em;--ltx-fo-depth:0.17em;font-size:11.5pt;" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 9.61)" width="76.13"><span id="S2.SS3.SSS0.Px1.p2.pic1.1" style="--ltx-fg-color:#FFFFFF;">Takeaways</span></foreignObject></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 221.95)"><foreignObject style="--ltx-fo-width:40.23em;--ltx-fo-height:1.74em;--ltx-fo-depth:15.04em;font-size:10pt;" height="232.31" overflow="visible" transform="matrix(1 0 0 -1 0 24.14)" width="556.67"><span id="S2.SS3.SSS0.Px1.p2.pic1.2" style="width:40.23em;"><span id="S2.I1"><span id="S2.I1.i1" style="list-style-type:none;">• <span id="S2.I1.i1.p1"><span id="S2.I1.i1.p1.1"><span id="S2.I1.i1.p1.1.1" style="--ltx-fg-color:#000000;">We introduce the</span> <span id="S2.I1.i1.p1.1.2" style="--ltx-fg-color:#000000;">SWE-Lego</span> <span id="S2.I1.i1.p1.1.3" style="--ltx-fg-color:#000000;">dataset, comprising</span> <span id="S2.I1.i1.p1.1.4" style="--ltx-fg-color:#000000;">32k</span> <span id="S2.I1.i1.p1.1.5" style="--ltx-fg-color:#000000;">high-quality software engineering task instances and</span> <span id="S2.I1.i1.p1.1.6" style="--ltx-fg-color:#000000;">18k</span> <span id="S2.I1.i1.p1.1.7" style="--ltx-fg-color:#000000;">expert trajectories, the core building brick for training effective SWE agents.</span><br></span></span></span><span id="S2.I1.i2" style="list-style-type:none;">• <span id="S2.I1.i2.p1"><span id="S2.I1.i2.p1.1"><span id="S2.I1.i2.p1.1.1" style="--ltx-fg-color:#000000;">We demonstrate that combining real-world and synthetic data yields complementary benefits in both</span> <span id="S2.I1.i2.p1.1.2" style="--ltx-fg-color:#000000;">quantity and quality</span><span id="S2.I1.i2.p1.1.3" style="--ltx-fg-color:#000000;">. Moverover, trajectory curation and filtering further enhance data utility, thereby improving the issue resolving rate.</span></span></span></span></span></span></foreignObject></g></g></svg>

## 3 Refined Supervised Fine-tuning

Conventional supervised fine-tuning on expert trajectories has proven effective for SWE tasks, however, the intrinsic characteristics of these trajectories remains underexplored. A trajectory may ultimately resolve an issue, yet it often contains erroneous intermediate steps, e.g., failed attempts and malformed tool calls. Rather than blindly learning from the entire trajectory, we propose *step-level error masking*, which enables the model to learn from correct actions while excluding incorrect ones. Furthermore, we investigate *curriculum learning* in the SWE context, wherein progressively increasing trajectory difficulty better guides the training process.

### 3.1 Step-level Error Masking

While the high-quality trajectories identified in Section 2.3 offer valuable learning signals at a macro level, we propose a more granular refinement: step-level error masking. This approach is motivated by the observation that expert demonstrations rarely proceed monotonically to success; they often contain intermediate missteps, such as invalid function calls or incorrect arguments, that are later identified and corrected. Our method maintains the full trajectory context but selectively masks the loss calculation on erroneous agent responses. As illustrated in Figure 5, we use regular expression to identify error messages provided by the terminal environment, and apply error masking to the corresponding agent response. Crucially, errors stemming from reproducing bugs or executing test files are excluded from this masking process. This technique applies gradient updates solely to valid actions, thereby providing a robust learning signal that enhances both action generation and error recovery capabilities. By emphasizing learning from correct actions, it directly reduces core reasoning failures, such as “Incorrect Implementation” and “Localization Error”, that are prevalent in later training stages (Section 3.4). Empirical results in Table 7 demonstrate that step-level error masking improves model performance by over 2 points.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2601.01426/assets/error_mask.png)

Figure 5: An example of step-level error masking, which maintains the complete trajectory context while selectively masking the loss calculation on incorrect agent responses.

### 3.2 Difficulty-Based Curriculum Learning

To maximize sample efficiency and training performance, we implement a curriculum learning strategy that progressively exposes the model to tasks of increasing complexity. This approach encourages the model to acquire fundamental skills from simpler examples before advancing to more challenging problems, thereby promoting more robust generalization. We explore two principled methods for difficulty categorization:

- Model-Based Scoring: We fine-tune a Qwen2.5-Coder-32B-Instruct [^16] model as a difficulty scorer on a dataset of 1.5k human-annotated (task, difficulty) pairs. This scorer achieves 70% accuracy on a held-out test set and classifies tasks into three tiers: Easy (estimated solving time $<$ 15 minutes), Medium (15 minutes to 1 hour), and Hard ($>$ 1 hour).
- Trajectory-Length-Based Heuristic: We discover a *strong negative correlation* ($r=-0.95$) between instance resolve rate and the number of turns in the trajectory, as depicted in Figure 7. Leveraging this finding, we partition the data into three difficulty bins based on turn count: Easy (0-50 turns), Medium (50-70 turns), and Hard (70-100 turns).

For curriculum learning, we select the trajectory-length-based heuristic because (i) it outperformed the model-based approach by 0.5 points, and (ii) it is more computationally efficient and less prone to mislabeling.

We adopt a three-stage SFT curriculum based on difficulty rankings. In each stage, the model is trained exclusively on data from the corresponding difficulty tier. To mitigate catastrophic forgetting of previously acquired skills, the training data for each subsequent stage is augmented with all data from the preceding stages. This curriculum aligns with the training dynamics observed in Section 3.4: it first grounds the model on “Easy” tasks to overcome basic “Failed to Reproduce” errors, then introduces “Hard” tasks to develop the strategic planning needed to avoid “Ran Out of Max Turns” failures. Table 7 shows that curriculum learning consistently enhances the performance of both the 8B and 32B models, with the highest results achieved when combined with error masking.

Figure 6: Correlation between number of turns and average resolve rate.

<table><tbody><tr><td rowspan="2">Error Masking</td><td rowspan="2">Curriculum Learning</td><td colspan="2">Resolve Rate (%)</td></tr><tr><td>SWE-Lego-8B</td><td>SWE-Lego-32B</td></tr><tr><td>✓</td><td>✓</td><td>42.2</td><td>52.6</td></tr><tr><td>✓</td><td>✗</td><td>41.8 (-0.4)</td><td>51.8 (-0.8)</td></tr><tr><td>✗</td><td>✓</td><td>40.2 (-2.0)</td><td>50.4 (-2.2)</td></tr><tr><td>✗</td><td>✗</td><td>39.4 (-2.8)</td><td>48.8 (-3.8)</td></tr></tbody></table>

Figure 7: Ablation of training strategies.

### 3.3 Training Details

For training, we use the Qwen3-8B/32B <sup>2</sup> [^51] series as the base models. We perform full-parameter SFT on the collected SWE-Lego trajectories using LLaMA-Factory framework [^61]. Models are trained for 4 epochs with a global batch size of 64. We use the AdamW optimizer [^25] with a weight decay of 0.01 and a cosine learning rate schedule with a warmup ratio of 0.1. The learning rate is set to 1e-4 for the 7B/8B models and 5e-5 for 32B models. To enable training with long-horizon trajectories, we use a maximum context length of 128k tokens. For the Qwen3 series, RoPE scaling techniques such as YaRN [^36] are adopted to support context lengths of up to 128k tokens.

### 3.4 Error Analysis during Training

To validate our methodology, we analyze the agent’s failure modes during training by categorizing all failed instances (including unresolved, error, and empty patch cases) into five error types based on the agent’s trajectory:

- Failed to Reproduce: The agent either did not attempt reproduction or attempted but failed to reproduce the issue.
- Read Localization Error: The agent reproduces the issue but fails to open or inspect all files modified in the ground-truth (golden) patch.
- Write Localization Error: The agent opens all required files but fails to identify the correct lines to edit, either no edits are made to ground-truth files or the edited lines have no overlap with the ground-truth patch.
- Ran Out of Max Turns: The agent successfully reproduced the issue and correctly localized the read and write locations, but reached the maximum iteration limit (100 turns) before completing the task.
- Incorrect Implementation: The case does not fall into any of the above categories; the agent appears to find the right location but implements an incorrect fix, resulting in failing tests.

Figure 8 shows a clear progression. Early on, “Failed to Reproduce” errors (38.97%) dominate, indicating basic task misalignment. After epoch 1, this error vanishes, but “Ran Out of Max Turns” errors spike (35.14%), revealing a strategic planning deficit. In later epochs, “Incorrect Implementation” and “Localization Error” become the bottlenecks. This shift from process failures to reasoning failures motivates our combined approach: curriculum learning sequentially addresses the early and mid-stage errors by structuring task complexity, while step-level error masking specifically hones the fine-grained reasoning needed to overcome the persistent late-stage errors.

(a) Resolve Rate Improvement Trend

(b) Error Type Distribution Across Epochs

Figure 8: Evolution of model performance and failure modes over 4 epochs. (a) Resolve rate increases steadily, reaching 42.2% by epoch 4. (b) Error modes shift from early “Failed to Reproduce” to mid “Ran Out of Max Turns” to late “Incorrect Implementation” and “Localization Error” bottlenecks.

<svg id="S3.SS4.p4.pic1" height="291.95" overflow="visible" version="1.1" viewBox="0 0 600 291.95" width="600"><g style="--ltx-stroke-color:#000000;--ltx-fill-color:#000000;" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,291.95) matrix(1 0 0 -1 0 0)"><g style="--ltx-fill-color:#000000;" fill="#000000" fill-opacity="1.0"><path style="stroke:none" d="M 0 5.91 L 0 269.41 C 0 272.67 2.64 275.32 5.91 275.32 L 594.09 275.32 C 597.36 275.32 600 272.67 600 269.41 L 600 5.91 C 600 2.64 597.36 0 594.09 0 L 5.91 0 C 2.64 0 0 2.64 0 5.91 Z"></path></g><g style="--ltx-fill-color:#F2F2F2;" fill="#F2F2F2" fill-opacity="1.0"><path style="stroke:none" d="M 1.97 5.91 L 1.97 269.41 C 1.97 271.59 3.73 273.35 5.91 273.35 L 594.09 273.35 C 596.27 273.35 598.03 271.59 598.03 269.41 L 598.03 5.91 C 598.03 3.73 596.27 1.97 594.09 1.97 L 5.91 1.97 C 3.73 1.97 1.97 3.73 1.97 5.91 Z"></path></g><g transform="matrix(1.0 0.0 0.0 1.0 7.87 266.66)"><g style="--ltx-stroke-color:#000000;--ltx-fill-color:#000000;" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g style="--ltx-fill-color:#000000;" fill="#000000" fill-opacity="1.0"><path style="stroke:none" d="M 0 4.53 L 0 20.76 C 0 23.26 2.03 25.29 4.53 25.29 L 95.28 25.29 C 97.78 25.29 99.81 23.26 99.81 20.76 L 99.81 4.53 C 99.81 2.03 97.78 0 95.28 0 L 4.53 0 C 2.03 0 0 2.03 0 4.53 Z"></path></g><g style="--ltx-fill-color:#000000;" fill="#000000" fill-opacity="1.0"><path style="stroke:none" d="M 1.57 4.53 L 1.57 20.76 C 1.57 22.39 2.9 23.72 4.53 23.72 L 95.28 23.72 C 96.91 23.72 98.24 22.39 98.24 20.76 L 98.24 4.53 C 98.24 2.9 96.91 1.57 95.28 1.57 L 4.53 1.57 C 2.9 1.57 1.57 2.9 1.57 4.53 Z"></path></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.19)"><foreignObject style="--ltx-fo-width:4.78em;--ltx-fo-height:0.6em;--ltx-fo-depth:0.17em;font-size:11.5pt;" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 9.61)" width="76.13"><span id="S3.SS4.p4.pic1.1" style="--ltx-fg-color:#FFFFFF;">Takeaways</span></foreignObject></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 236.63)"><foreignObject style="--ltx-fo-width:40.23em;--ltx-fo-height:1.8em;--ltx-fo-depth:16.11em;font-size:10pt;" height="247.76" overflow="visible" transform="matrix(1 0 0 -1 0 24.91)" width="556.67"><span id="S3.SS4.p4.pic1.2" style="width:40.23em;"><span id="S3.I3"><span id="S3.I3.i1" style="list-style-type:none;">• <span id="S3.I3.i1.p1"><span id="S3.I3.i1.p1.1"><span id="S3.I3.i1.p1.1.1" style="--ltx-fg-color:#000000;">We refine conventional SFT for SWE tasks with two innovations: (1)</span> <span id="S3.I3.i1.p1.1.2" style="--ltx-fg-color:#000000;">step-level error masking</span><span id="S3.I3.i1.p1.1.3" style="--ltx-fg-color:#000000;">, which enables the model to learn from effective intermediate actions, and (2)</span> <span id="S3.I3.i1.p1.1.4" style="--ltx-fg-color:#000000;">curriculum learning</span><span id="S3.I3.i1.p1.1.5" style="--ltx-fg-color:#000000;">, which progressively increases task difficulty, approximated by the number of interaction turns.</span><br></span></span></span><span id="S3.I3.i2" style="list-style-type:none;">• <span id="S3.I3.i2.p1"><span id="S3.I3.i2.p1.1"><span id="S3.I3.i2.p1.1.1" style="--ltx-fg-color:#000000;">Our refined SFT outperforms conventional SFT by</span> <span id="S3.I3.i2.p1.1.2" style="--ltx-fg-color:#000000;">3.8%</span><span id="S3.I3.i2.p1.1.3" style="--ltx-fg-color:#000000;">, establishing new state-of-the-art performance among open-source models of comparable size on SWE-Bench Verified: SWE-Lego-Qwen3-8B hits</span> <span id="S3.I3.i2.p1.1.4" style="--ltx-fg-color:#000000;">42.2%</span><span id="S3.I3.i2.p1.1.5" style="--ltx-fg-color:#000000;">, and SWE-Lego-Qwen3-32B reaches</span> <span id="S3.I3.i2.p1.1.6" style="--ltx-fg-color:#000000;">52.6%</span><span id="S3.I3.i2.p1.1.7" style="--ltx-fg-color:#000000;">.</span></span></span></span></span></span></foreignObject></g></g></svg>

## 4 Test-time Scaling

Test-time scaling (TTS) can effectively improve the SWE agent performance by allocating additional compute during inference. Typically there are two complementary dimensions: *sequential scaling* with more interaction turns, and *parallel scaling* that generates multiple rollouts and selects the best via a verifier. However, it still remains unclear their optimal balance, given limited constraints on compute budget or latency. In Section 4.1, we first systematically investigate this trade-off and identify when to prioritize each dimension. We further investigate the design of verifier for better parallel scaling in Section 4.2.

### 4.1 Balancing Sequential and Parallel Scaling

We investigate resource allocation between sequential and parallel scaling by varying both the maximum interaction turns and the number of candidate rollouts. We use *latency* as the cost metric for budget control. For sequential turns, as the trajectory lengthens, the growing context requires more computation per turn, resulting in latency that scales super-linearly (between linear and quadratic) with respect to the number of interaction turns; In contrast, candidate rollouts are executed in batch, so latency exhibits sub-linear scaling with respect to the number of rollouts.

As shown in Figure 9, sequential scaling is highly efficient in the low-turn regime: additional turns yields more environment feedback, enables error correction, and allows iterative refinement. This makes sequential scaling the preferred strategy under tight latency budgets. However, performance starts to saturate around 100–140 turns. The agent either succeeds within the allotted turns or encounters obstacles that additional turns alone cannot overcome (more analysis in Appendix B.1). Beyond this saturation point, parallel scaling coupled with verifier-based selection becomes more effective, as independent trajectories explore diverse paths through the solution space. The iso-latency contours in Figure 9 highlight this balance: under equivalent latency, the optimal allocation shifts from sequential-dominated to parallel-dominated as the total latency budget increases.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2601.01426/assets/tts_2d.png)

Figure 9: Resolve rate on SWE-bench Verified as a function of maximum interaction turns (x-axis) and number of parallel rollouts (y-axis). Curves indicate iso-latency contours where different sequential-parallel combinations yield equivalent wall-clock time. Sequential scaling is most effective at low turn counts; after saturation, improvements are driven primarily by parallel scaling.

### 4.2 Improved Parallel Scaling

During parallel scaling, a verifier takes a trajectory and its proposed patch and returns a score between 0 and 1 that reflects the likelihood of successful resolution. We compare two paradigms: the *regressive verifier* [^34], which appends a scoring head to the backbone and is trained with binary cross-entropy loss, and the *generative verifier* [^35] [^17], which formulates verification as text generation by predicting “yes” or “no” and computing the score from normalized token probabilities. The generative formulation aligns with the pre-trained next-token prediction objective, potentially better leveraging the model’s inherent knowledge [^59]. For each test instance, we generate $K$ candidate rollouts and let the verifier select the top-scoring one. TTS@ $K$ is the fraction of instances resolved under this top-1 selection from $K$ candidates.

(a) Generative vs. Regressive Verifier

(b) Comparison with Existing Verifiers

Figure 10: Parallel TTS performance on SWE-bench Verified. (a) Ablation on the verifier paradigm: generative verifiers consistently outperform regressive counterparts across model sizes (8B and 32B). (b) Comparison with publicly available verifiers: SWE-Lego-Verifier-8B outperforms OpenHands-Critic-32B and R2E-Gym-Verifier-14B, with the performance gap widening as rollouts increase.

#### Regressive vs. Generative Verifier.

We compare the two paradigms using the same training data (5k resolved, 13k unresolved trajectories from real-world tasks) and the same backbone (e.g., Qwen3-8B). Emperically, we find that larger verifiers benefit stronger rollout models, so for $32$ B rollout models we use Qwen3-Coder-30B-A3B as the verifier (see Appendix B.2 for more details). For consistency with prior work, we set the maximum interaction turns to 100 in all verifier experiments. As shown in Figure 10(a), the generative verifier consistently outperforms the regressive one. At small $K$, both achieve similar TTS@ $K$; as $K$ increases, the regressive verifier plateaus while the generative verifier continues to improve. On SWE-Lego-Qwen3-8B, the gap reaches 2.8% at $K{=}16$ (49.6% vs. 46.8%). We hypothesize that it is because the generative formulation aligns with next-token prediction, better leveraging pre-trained semantics. We therefore adopt the generative paradigm.

#### Comparison with Existing Verifiers.

We compare SWE-Lego-Verifier-8B against publicly available verifiers on SWE-Lego-Qwen3-8B rollouts (Figure 10(b)). Our verifier achieves TTS@16 of 49.6%, outperforming OpenHands-Critic-32B (44.0%) [^34] and R2E-Gym-Verifier-14B (47.0%) [^17]. Beyond absolute performance, we observe qualitatively different scaling behaviors across verifier paradigms. OpenHands-Critic-32B, which employs the regressive paradigm, exhibits performance degradation at higher $K$, a counterintuitive result indicating that larger candidate pools overwhelm its discriminative capacity. In contrast, both generative verifiers (ours and R2E-Gym) maintain monotonic improvement toward the Pass@ $K$ upper bound, further confirming that the generative formulation provides more robust scaling properties. The performance advantage over R2E-Gym-Verifier-14B likely stems from two factors: a larger volume of high-quality training trajectories and better distribution alignment between training data and SWE-Lego agent behaviors.

<svg id="S4.SS2.SSS0.Px2.p2.pic1" height="176.87" overflow="visible" version="1.1" viewBox="0 0 600 176.87" width="600"><g style="--ltx-stroke-color:#000000;--ltx-fill-color:#000000;" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,176.87) matrix(1 0 0 -1 0 0)"><g style="--ltx-fill-color:#000000;" fill="#000000" fill-opacity="1.0"><path style="stroke:none" d="M 0 5.91 L 0 154.33 C 0 157.6 2.64 160.24 5.91 160.24 L 594.09 160.24 C 597.36 160.24 600 157.6 600 154.33 L 600 5.91 C 600 2.64 597.36 0 594.09 0 L 5.91 0 C 2.64 0 0 2.64 0 5.91 Z"></path></g><g style="--ltx-fill-color:#F2F2F2;" fill="#F2F2F2" fill-opacity="1.0"><path style="stroke:none" d="M 1.97 5.91 L 1.97 154.33 C 1.97 156.51 3.73 158.27 5.91 158.27 L 594.09 158.27 C 596.27 158.27 598.03 156.51 598.03 154.33 L 598.03 5.91 C 598.03 3.73 596.27 1.97 594.09 1.97 L 5.91 1.97 C 3.73 1.97 1.97 3.73 1.97 5.91 Z"></path></g><g transform="matrix(1.0 0.0 0.0 1.0 7.87 151.58)"><g style="--ltx-stroke-color:#000000;--ltx-fill-color:#000000;" fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="matrix(1 0 0 1 0 0)"><g style="--ltx-fill-color:#000000;" fill="#000000" fill-opacity="1.0"><path style="stroke:none" d="M 0 4.53 L 0 20.76 C 0 23.26 2.03 25.29 4.53 25.29 L 95.28 25.29 C 97.78 25.29 99.81 23.26 99.81 20.76 L 99.81 4.53 C 99.81 2.03 97.78 0 95.28 0 L 4.53 0 C 2.03 0 0 2.03 0 4.53 Z"></path></g><g style="--ltx-fill-color:#000000;" fill="#000000" fill-opacity="1.0"><path style="stroke:none" d="M 1.57 4.53 L 1.57 20.76 C 1.57 22.39 2.9 23.72 4.53 23.72 L 95.28 23.72 C 96.91 23.72 98.24 22.39 98.24 20.76 L 98.24 4.53 C 98.24 2.9 96.91 1.57 95.28 1.57 L 4.53 1.57 C 2.9 1.57 1.57 2.9 1.57 4.53 Z"></path></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 13.39 9.19)"><foreignObject style="--ltx-fo-width:4.78em;--ltx-fo-height:0.6em;--ltx-fo-depth:0.17em;font-size:11.5pt;" height="12.3" overflow="visible" transform="matrix(1 0 0 -1 0 9.61)" width="76.13"><span id="S4.SS2.SSS0.Px2.p2.pic1.1" style="--ltx-fg-color:#FFFFFF;">Takeaways</span></foreignObject></g></g></g> <g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 122.32)"><foreignObject style="--ltx-fo-width:40.23em;--ltx-fo-height:1.74em;--ltx-fo-depth:7.84em;font-size:10pt;" height="132.68" overflow="visible" transform="matrix(1 0 0 -1 0 24.14)" width="556.67"><span id="S4.SS2.SSS0.Px2.p2.pic1.2" style="width:40.23em;"><span id="S4.I1"><span id="S4.I1.i1" style="list-style-type:none;">• <span id="S4.I1.i1.p1"><span id="S4.I1.i1.p1.1"><span id="S4.I1.i1.p1.1.1" style="--ltx-fg-color:#000000;">Sequential-then-Parallel</span><span id="S4.I1.i1.p1.1.2" style="--ltx-fg-color:#000000;">: We suggest to prioritize sequential scaling before a certain saturation point, and then allocate remaining compute to parallel scaling.</span><br></span></span></span><span id="S4.I1.i2" style="list-style-type:none;">• <span id="S4.I1.i2.p1"><span id="S4.I1.i2.p1.1"><span id="S4.I1.i2.p1.1.1" style="--ltx-fg-color:#000000;">Generative <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="&gt;"><semantics><mo style="--ltx-fg-color:#000000;" mathcolor="#000000">&gt;</mo> <annotation encoding="application/x-tex">&gt;</annotation></semantics></math> Regressive</span><span id="S4.I1.i2.p1.1.2" style="--ltx-fg-color:#000000;">: The generative verifier consistently outperforms the regressive variant for parallel scaling across different model sizes and rollout budgets.</span></span></span></span></span></span></foreignObject></g></g></svg>

## 5 Comparisons with Existing Methods

We compare the performance of SWE-Lego models with both proprietary and open-source baselines on SWE-bench Verified. As mentioned in Section 2.2, Docker images of SWE-bench Verified released prior to Sep 3, 2025 may contain the ground truth commits, which can lead to Git hacking, e.g., via git log --all or by grepping commit messages. For fair comparisons with prior methods, we report our results both with and without *git hacking*. The results without hacking are obtained using the latest SWE-bench Verified Docker images. From Table 6, we see that our SWE-Lego-Qwen3-8B achieves a $42.2\%$ resolve rate with refined SFT and reaches 49.6% with further TTS@16, while SWE-Lego-Qwen3-32B attains $52.6\%$ with SFT and 58.8% with TTS@16. These *hack-free* results surpass most open-source models and several larger proprietary models, even when their results are generally inflated by git hacking.

<table><tbody><tr><td>Model</td><td>Scaffold</td><td>Training</td><td>Resolve Rate (%)</td></tr><tr><td colspan="4">Proprietary Models  </td></tr><tr><td>   OpenAI-GPT-4o <sup><a href="#fn:30">30</a></sup></td><td>Internal pipeline</td><td>-</td><td>33.2</td></tr><tr><td>   OpenAI-o3 <sup><a href="#fn:33">33</a></sup></td><td>Mini-SWE-agent</td><td>-</td><td>58.4</td></tr><tr><td>   Claude-4-Sonnet <sup><a href="#fn:1">1</a></sup></td><td>SWE-agent</td><td>-</td><td>66.6</td></tr><tr><td>   Claude-4.5-Sonnet <sup><a href="#fn:2">2</a></sup></td><td>Mini-SWE-agent</td><td>-</td><td>70.6</td></tr><tr><td>   OpenAI-GPT-5 <sup><a href="#fn:32">32</a></sup></td><td>OpenHands</td><td>-</td><td>71.8</td></tr><tr><td>   Gemini 3 Pro Preview <sup><a href="#fn:14">14</a></sup></td><td>Mini-SWE-agent</td><td>-</td><td>74.2</td></tr><tr><td colspan="4">Open-Source Models  </td></tr><tr><td colspan="4">Parameters <math><semantics><mo>≈</mo> <annotation>\approx</annotation></semantics></math> 7B  </td></tr><tr><td>   Qwen2.5-Coder-Instruct-7B <sup><a href="#fn:16">16</a></sup></td><td>MOpenHands</td><td>-</td><td>1.0</td></tr><tr><td>   Qwen3-8B <sup><a href="#fn:51">51</a></sup></td><td>OpenHands</td><td>-</td><td>7.6</td></tr><tr><td>   SWE-Gym-7B <sup><a href="#fn:35">35</a></sup></td><td>OpenHands</td><td>SFT</td><td>10.6</td></tr><tr><td>   SWE-agent-LM-7B <sup><a href="#fn:54">54</a></sup></td><td>SWE-agent</td><td>SFT</td><td>15.2</td></tr><tr><td>   Lingma-SWE-GPT-7B <sup><a href="#fn:28">28</a></sup></td><td>SWESynInfer</td><td>SFT</td><td>18.2</td></tr><tr><td>   SWE-Mirror-LM-7B <sup><a href="#fn:44">44</a></sup></td><td>MOpenHands</td><td>SFT</td><td>22.8</td></tr><tr><td>   SWE-Dev-7B <sup><a href="#fn:43">43</a></sup></td><td>OpenHands</td><td>RL</td><td>23.4</td></tr><tr><td>   Klear-Agent-8B-SFT <sup><a href="#fn:21">21</a></sup></td><td>Mini-SWE-agent-plus</td><td>SFT</td><td>39.0</td></tr><tr><td>   SWE-Lego-Qwen3-8B</td><td>OpenHands</td><td>SFT</td><td>44.4 / 42.2</td></tr><tr><td>    + TTS@16</td><td>OpenHands</td><td>SFT</td><td>53.4 / 49.6</td></tr><tr><td colspan="4">Parameters <math><semantics><mo>=</mo> <annotation>=</annotation></semantics></math> 32B  </td></tr><tr><td>   Qwen2.5-Coder-Instruct-32B <sup><a href="#fn:16">16</a></sup></td><td>MOpenHands</td><td>-</td><td>6.2</td></tr><tr><td>   Qwen3-32B <sup><a href="#fn:51">51</a></sup></td><td>OpenHands</td><td>-</td><td>23.2</td></tr><tr><td>   SWE-Gym-32B <sup><a href="#fn:35">35</a></sup></td><td>OpenHands</td><td>SFT</td><td>20.6</td></tr><tr><td>   R2E-Gym-32B <sup><a href="#fn:17">17</a></sup></td><td>R2E-Gym</td><td>SFT</td><td>34.4</td></tr><tr><td>    + TTS@16</td><td>R2E-Gym</td><td>SFT</td><td>49.4</td></tr><tr><td>    + TTS@26</td><td>R2E-Gym</td><td>SFT</td><td>51.0</td></tr><tr><td>   SWE-Dev-32B <sup><a href="#fn:43">43</a></sup></td><td>OpenHands</td><td>RL</td><td>36.6</td></tr><tr><td>   Skywork-SWE-32B <sup><a href="#fn:58">58</a></sup></td><td>OpenHands</td><td>SFT</td><td>38.0</td></tr><tr><td>    + TTS@8</td><td>OpenHands</td><td>SFT</td><td>47.0</td></tr><tr><td>   SWE-agent-LM-32B <sup><a href="#fn:54">54</a></sup></td><td>SWE-agent</td><td>SFT</td><td>40.2</td></tr><tr><td>   DeepSWE-32B-Preview <sup><a href="#fn:26">26</a></sup></td><td>OpenHands</td><td>RL</td><td>42.2</td></tr><tr><td>    + TTS@16</td><td>OpenHands</td><td>RL</td><td>59.0</td></tr><tr><td>   SWE-Mirror-LM-32B <sup><a href="#fn:44">44</a></sup></td><td>MOpenHands</td><td>SFT</td><td>52.2</td></tr><tr><td>   CWM-32B <sup><a href="#fn:10">10</a></sup></td><td>Agentless</td><td>CPT + SFT + RL</td><td>53.9</td></tr><tr><td>   SWE-Lego-Qwen3-32B</td><td>OpenHands</td><td>SFT</td><td>57.6 / 52.6</td></tr><tr><td>    + TTS@16</td><td>OpenHands</td><td>SFT</td><td>60.4 / 58.8</td></tr></tbody></table>

Table 6: Performance comparison on the SWE-bench Verified. Our results are reported in the “A/B” format, representing the results with and without *Git hacking* respectively.

## 6 Related Work

### 6.1 Training Dataset for Coding

High-quality data is foundational for code intelligence. Early works focused on large-scale pretraining corpora like The Stack [^20], often employing rigorous filtering pipelines [^22] to ensure quality. The field has since shifted towards instruction tuning, utilizing self-instruct methods [^7], Git commit structures [^29], and synthetic generation techniques like Evol-Instruct [^27] and OSS-Instruct [^47]. Broader instruction tuning efforts have also contributed significantly, with datasets like OpenHermes [^42] and Glaive [^13] providing extensive general and code-specific training data. Recently, attention has turned to repository-level SWE datasets. Works such as R2E-Gym [^17], SWE-smith [^54], and SWE-rebench [^4] provide scalable pipelines for task collection and environment construction. However, there remains a scarcity of large-scale, verifiable expert trajectories for long-horizon issue fixing, a gap our hybrid dataset addresses.

### 6.2 Coding LLMs and Agents

Recent progress spans both powerful foundation models and agentic frameworks that enable autonomous software engineering. Foundation models such as Code Llama [^37] and DeepSeek Coder [^15] continue to serve as critical backbones, achieving state-of-the-art performance through massive code pretraining and extended context windows. To handle repository contexts, some approaches integrate RAG [^38] or AST-based retrieval [^60]. Agentic frameworks such as Agentless [^48], SWE-agent [^55], Mini-SWE-Agent [^41], Mini-SWE-Agent-Plus [^21] OpenHands [^40] and MOpenHands [^57] serve as scaffolds to streamline interactions with development environments. Others emphasize planning [^5] that views the repository-level coding as a planning problem or self-evolving the SWE agents on the fly [^49].

### 6.3 Benchmarks for Software Engineering

Evaluation has progressed from function-level synthesis [^9] to repository-level tasks. Benchmarks like ClassEval [^12], CrossCodeEval [^11], and RepoBench [^24] assess broader context utilization. For autonomous agents, the focus is on realistic issue resolving, utilizing datasets like Defects4J [^19] and environments like InterCode [^52]. SWE-bench [^18] has become the standard for GitHub issue resolution, inspiring variants for multilingual [^57], multimodal [^53], and comprehensive agentic evaluation [^50]. Safety benchmarks like CyberSecEval [^6] further ensure code security. Our work aligns with this trend, focusing on scaling capabilities for practical software development.

## 7 Conclusions

We present SWE-Lego, a framework consisting of hybrid executable data construction, refined SFT, and verifier-guided TTS, which achieves strong hack-free performance on SWE-bench Verified. The core driver of performance gains lies in scalable, verifiable hybrid data instances and expert trajectories. Meanwhile, error masking and curriculum learning stabilize the training process, and our proposed generative verifier, built on parallel test-time scaling, efficiently allocates inference computing resources to boost the performance further. Collectively, these strategies provide a reproducible paradigm for building robust agentic systems for software engineering issue resolving.

## References

## Appendix

## Appendix A Additional Information on Data Construction

### A.1 Real and Synthetic Instance Construction

Here we detail how we collect or generate the core fields, golden\_patch, FAIL-to-PASS, PASS\_TO\_PASS, problem\_statement and image\_name of SWE-Lego data instances.

- golden\_patch: For real-world instances, the golden\_patch corresponds to the merged PR diff, representing the human-authored fix. For synthetic instances, we inject a bug and then derive the golden patch by reverting this injection, i.e., the diff between the buggy branch and the original clean commit. Specifically, real tasks use a buggy pre-commit version of the repository in the sandbox, with the patch serving as the golden solution. For synthetic tasks, following SWE-smith [^54]), we place the non-buggy repository version inside the sandbox, while the buggy version resides in a remote Git branch; here, the patch field records the bug injection logic. In practice, before solving a synthetic task, one must check out the buggy branch in the sandbox. The golden solution is to revert the patch specified in the patch field.
- FAIL-to-PASS and PASS\_TO\_PASS: In real data, we trace tests that are introduced or modified in the PR: tests that fail on the pre-merge commit but pass after applying the patch form the FAIL\_TO\_PASS set, while tests that pass in both states are labeled as PASS\_TO\_PASS. For synthetic data, we run the repository’s test suite before and after the injected bug. Newly failing tests define FAIL\_TO\_PASS, and unaffected tests form PASS\_TO\_PASS. This shared test-based view ensures that every instance, regardless of origin, is grounded in executable signals about both correctness and regression.
- problem\_statement: For real-world instances, the problem\_statement is extracted from the GitHub issue linked to the PR, preserving the natural language description written by developers and users. For synthetic instances, we prompt an LLM with the golden patch, failing tests, and execution logs to synthesize a realistic issue description that mimics how a human would report the bug. The result is a unified textual interface where real issues contribute authentic language, and synthetic issues expand coverage in a controlled manner.
- image\_name: Each real-world instance is associated with a unique Docker image (a.k.a., sandbox) built from the exact pre-merge commit, since different PRs can depend on distinct dependency states or build configurations. In contrast, synthetic instances from the same repository typically share a sandbox image based on a stable base commit, because bug injections operate at the code level without introducing new system-level dependencies.

### A.2 Analysis of Data Categories

<table><tbody><tr><td>Category</td><td>Description</td><td>Real Data (%)</td><td>Synthetic Data (%)</td></tr><tr><td colspan="4">Top-5 Categories</td></tr><tr><td>A</td><td>API / signature mismatch</td><td>26.6</td><td>12.8</td></tr><tr><td>B</td><td>Logic / conditional bug</td><td>29.9</td><td>49.4</td></tr><tr><td>C</td><td>Input validation / boundary / sentinel handling error</td><td>16.1</td><td>5.9</td></tr><tr><td>D</td><td>Constructor / inheritance / inheritance contract break</td><td>2.5</td><td>7.5</td></tr><tr><td>E</td><td>Missing import / symbol / attribute error</td><td>8.3</td><td>19.3</td></tr><tr><td colspan="4">The Remaining Categories</td></tr><tr><td>F</td><td>State consistency / bookkeeping / caching bug</td><td>6.4</td><td>2.6</td></tr><tr><td>G</td><td>Copy semantics / mutability / in-place mutation of inputs</td><td>0.7</td><td>0.3</td></tr><tr><td>H</td><td>Protocol / spec conformance bug</td><td>7.2</td><td>1.4</td></tr><tr><td>I</td><td>IO / file system / resource handling bug</td><td>1.9</td><td>0.7</td></tr><tr><td>J</td><td>Security / sensitive data leakage</td><td>0.4</td><td>0.1</td></tr></tbody></table>

Table 7: Distribution of categories in real-world and synthetic training dataset.

Following the categorization method of SWE instances proposed in BugPilot [^39], we prompt the Claude-4-Sonnet to categorize the training instances SWE-Lego for analyses.

The comparison on Table 7 reveals a significant complementarity between the real and synthetic data sources in SWE-Lego. Real-world data tends to focus on “external interaction and boundary defense” (e.g., API changes, input validation, protocol conformance), whereas synthetic data emphasizes “internal implementation and structural integrity” (e.g., logic flow, dependency management). Beyond the inherent scalability of synthetic data generation, incorporating it into the training set offers distributional benefits by supplementing rare structural defects and rebalancing the focus between interface adaptation and internal logic.

## Appendix B Additional Analysis on Test-Time Scaling

We present additional experiments on test-time scaling, analyzing the saturation in sequential scaling and the effects of training-data scale and verifier-model scale on parallel scaling.

### B.1 Sequential Scaling Analysis

#### Truncation Rate and Average Turns.

As the turn limit increases for SWE-Lego-Qwen3-8B rollouts, the truncation rate declines quickly but becomes marginal beyond roughly 140 turns (Figure 11(a)). The average number of turns per trajectory similarly rises and then plateaus. Most trajectories either terminate (success or failure) or fall into unproductive states well before exhausting large turn budgets, so additional turns beyond this threshold yield little marginal utility.

#### Resolve Rate by Turn Count.

Resolve rates are low for very small turn counts (insufficient exploration) and also for very large counts (the agent is typically stuck, retrying ineffective approaches), with the highest rates appearing at intermediate ranges (Figure 11(b)).

These observations provide empirical support for the sequential scaling saturation phenomenon: beyond a certain turn limit, additional sequential compute yields limited marginal benefit. This motivates the shift to parallel scaling with verifier selection once sequential scaling saturates.

(a) Truncation Rate and Average Turns

(b) Resolve Rate by Turn Count

Figure 11: Analysis of sequential scaling behavior. (a) Truncation rate and average turns under different maximum turn limits: both metrics show diminishing changes beyond 140 turns. (b) Resolve rate across turn count intervals: trajectories with very few or very many turns tend to have lower resolve rates.

### B.2 Impact of Training Data Scale and Model Scale

#### Training Data Scale.

We ablate verifier training data volume on SWE-Lego-Qwen3-8B rollouts by comparing the full set (18K trajectories: 5K resolved + 13K unresolved) to a 6K subset matching the scale used by R2E-Gym-Verifier. As shown in Figure 12(a), the 18K-trained verifier consistently outperforms the 6K verifier across all $K$. At TTS@16, it achieves 49.6% vs. 47.6%, with the gap widening as $K$ increases, indicating that larger training sets improve discrimination in larger candidate pools. Notably, with the same 6K scale, our verifier slightly outperforms R2E-Gym-Verifier-14B (47.6% vs. 47.0%), suggesting that data quality and alignment with the rollout agent also matter, beyond sheer quantity.

#### Verifier Model Scale.

We compare 8B and 30B verifiers (Qwen3-8B and Qwen3-Coder-30B-A3B backbones) when scoring rollouts from SWE-Lego-Qwen3-8B and SWE-Lego-Qwen3-32B agents (Figure 12(b)). For 8B rollouts, 8B and 30B verifiers perform similarly across $K$. For 32B rollouts, the 30B verifier is consistently better, especially at larger $K$; at TTS@16, it reaches 58.8% vs. 56.6%. One possible e explanation is that 32B outputs contain subtler errors that benefit from larger verifier capacity; as $K$ grows, more near-correct candidates increase the premium on fine-grained ranking.

(a) Impact of Training Data Scale

(b) Impact of Verifier Model Scale

Figure 12: Ablation studies on verifier training. (a) Training data scale: verifiers trained on 18K trajectories outperform those trained on 6K trajectories. (b) Model scale: for 8B rollouts, verifier size has minimal impact; for 32B rollouts, the 30B verifier provides gains over the 8B verifier, especially at higher $K$.

## Appendix C Extensions and Future Work

#### Beyond Issue Resolving.

Extending the scope beyond defect repair to other tasks, like feature implementation, refactoring or tests generation, reveal limitations in current training data and benchmarks. Constructing datasets that adequately encode ambiguous requirements, nuanced acceptance criteria, and non-regression constraints remains challenging. Moreover, validating agent performance across richer dimensions, such as functional correctness, performance bounds, and interface stability, requires more complex evaluation frameworks than those used for bug fixing. Supervising higher-order behaviors like design decisions, system integration, and code refactoring also lacks well-defined signals for effective learning.

#### Multilingual SWE.

Expanding beyond Python to languages such as JavaScript/TypeScript, Java, Go, and Rust is impeded by language-specific toolchains, build systems, and testing frameworks. The absence of cross-language sandboxes and multilingual datasets limits agents’ ability to generalize across ecosystems. Further challenges include navigating heterogeneous dependency management practices, adapting to diverse testing paradigms, and accounting for language-specific code organization idioms, all of which complicate the transfer of core agentic skills (navigation, localization, editing, validation) across programming languages.

#### Cross-task Capability Balance.

Heterogeneous code tasks, including repository-level issue resolution, file-specific code reasoning, and competitive coding, exhibit divergent demands on agent capabilities, leading to risks of uneven performance. Balancing proficiency across these tasks is hindered by varying difficulty gradients and interaction requirements, which may cause agents to overfit to dominant task archetypes. The lack of standardized metrics for comparing capability across such distinct tasks further complicates efforts to assess and mitigate this imbalance.

## Appendix D Example of Git Hacking

In Figure 13, we present a concrete example of Git hacking: an SWE agent peeks at the Git commit history and unintentionally retrieves the pre-existing solution. During evaluation, the agent may run common Git commands (e.g., git log, git diff) to bypass independent problem-solving and directly replicate the golden patch, falsely inflating performance and invalidating the evaluation of genuine software engineering capabilities.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2601.01426/assets/figures/git_hack_example.png)

Figure 13: Demonstration of Git hacking by SWE agents.

[^1]: Anthropic Claude Sonnet 4. Note: [https://www.anthropic.com/claude/sonnet](https://www.anthropic.com/claude/sonnet) \[Accessed 31-08-2025\] Cited by: Table 6.

[^2]: Anthropic Introducing claude sonnet 4.5. Note: [https://www.anthropic.com/news/claude-sonnet-4-5](https://www.anthropic.com/news/claude-sonnet-4-5) 2025-09-30 Cited by: §2.3, Table 6.

[^3]: J. Austin, A. Odena, M. Nye, M. Bosma, H. Michalewski, D. Dohan, E. Jiang, C. Cai, M. Terry, Q. Le, et al. Program synthesis with large language models. arXiv preprint arXiv:2108.07732. Cited by: §1.

[^4]: I. Badertdinov, A. Golubev, M. Nekrashevich, A. Shevtsov, S. Karasik, A. Andriushchenko, M. Trofimova, D. Litvintseva, and B. Yangel SWE-rebench: an automated pipeline for task collection and decontaminated evaluation of software engineering agents. External Links: 2505.20411 Cited by: 1st item, §2.2, §2.2, §6.1.

[^5]: R. Bairi, A. Sonwane, A. Kanade, V. Arun, et al. CodePlan: repository-level coding using llms and planning. arXiv preprint arXiv:2309.12499. Cited by: §6.2.

[^6]: M. Bhatt, S. Chennabasappa, Y. Li, C. Nikolaidis, D. Song, S. Wan, F. Ahmad, C. Aschermann, Y. Chen, D. Kapil, et al. Cyberseceval 2: a wide-ranging cybersecurity evaluation suite for large language models. arXiv preprint arXiv:2404.13161. Cited by: §6.3.

[^7]: S. Chaudhary Code alpaca: an instruction-following llama model for code generation. GitHub repository. External Links: [Link](https://github.com/sahil280114/codealpaca) Cited by: §6.1.

[^8]: M. Chen, J. Tworek, H. Jun, Q. Yuan, H. P. de Oliveira Pinto, J. Kaplan, H. Edwards, Y. Burda, N. Joseph, G. Brockman, A. Ray, R. Puri, G. Krueger, M. Petrov, H. Khlaaf, G. Sastry, P. Mishkin, B. Chan, S. Gray, N. Ryder, M. Pavlov, A. Power, L. Kaiser, M. Bavarian, C. Winter, P. Tillet, F. P. Such, D. Cummings, M. Plappert, F. Chantzis, E. Barnes, A. Herbert-Voss, W. H. Guss, A. Nichol, A. Paino, N. Tezak, J. Tang, I. Babuschkin, S. Balaji, S. Jain, W. Saunders, C. Hesse, A. N. Carr, J. Leike, J. Achiam, V. Misra, E. Morikawa, A. Radford, M. Knight, M. Brundage, M. Murati, K. Mayer, P. Welinder, B. McGrew, D. Amodei, S. McCandlish, I. Sutskever, and W. Zaremba Evaluating large language models trained on code. External Links: 2107.03374 Cited by: §1.

[^9]: M. Chen, J. Tworek, H. Jun, Q. Yuan, H. P. d. O. Pinto, J. Kaplan, H. Edwards, Y. Burda, N. Joseph, G. Brockman, et al. Evaluating large language models trained on code. arXiv preprint arXiv:2107.03374. Cited by: §6.3.

[^10]: J. Copet, Q. Carbonneaux, G. Cohen, J. Gehring, J. Kahn, J. Kossen, F. Kreuk, E. McMilin, M. Meyer, Y. Wei, et al. CWM: an open-weights llm for research on code generation with world models. arXiv preprint arXiv:2510.02387. Cited by: §1, Table 6.

[^11]: Y. Ding, B. Wang, S. Joty, and S. C. Hoi CrossCodeEval: a diverse and multilingual benchmark for cross-file code completion. In Thirty-seventh Conference on Neural Information Processing Systems Datasets and Benchmarks Track, Cited by: §6.3.

[^12]: X. Du, M. Liu, K. Wang, H. Wang, J. Liu, Y. Chen, J. Feng, S. Shang, S. Chen, Y. Zhao, et al. ClassEval: a manually-crafted benchmark for evaluating llms on class-level code generation. arXiv preprint arXiv:2308.01861. Cited by: §1, §6.3.

[^13]: GlaiveAI Glaive code assistant dataset. Hugging Face. External Links: [Link](https://huggingface.co/datasets/glaiveai/glaive-code-assistant-v3) Cited by: §6.1.

[^14]: Google A new era of intelligence with gemini 3. Note: [https://blog.google/products/gemini/gemini-3/](https://blog.google/products/gemini/gemini-3/) Published: 2025-11-18; Accessed: 2025-12-22 Cited by: Table 6.

[^15]: D. Guo, Q. Zhu, D. Yang, Z. Xie, K. Dong, W. Zhang, G. Chen, X. Bi, Y. Wu, Y. Li, et al. DeepSeek coder: when the large language model meets programming–the rise of code intelligence. arXiv preprint arXiv:2401.14196. Cited by: §6.2.

[^16]: B. Hui, J. Yang, Z. Cui, J. Yang, D. Liu, L. Zhang, T. Liu, J. Zhang, B. Yu, K. Dang, et al. Qwen2. 5-coder technical report. arXiv preprint arXiv:2409.12186. Cited by: 1st item, Table 6, Table 6.

[^17]: N. Jain, J. Singh, M. Shetty, L. Zheng, K. Sen, and I. Stoica R2E-gym: procedural environments and hybrid verifiers for scaling open-weights swe agents. External Links: 2504.07164 Cited by: 3rd item, §1, §2.2, §4.2, §4.2, Table 6, §6.1.

[^18]: C. E. Jimenez, J. Yang, A. Wettig, S. Yao, K. Pei, O. Press, and K. Narasimhan SWE-bench: can language models resolve real-world github issues?. In The Twelfth International Conference on Learning Representations, Cited by: §1, §6.3.

[^19]: R. Just, D. Jalali, L. Inozemtseva, M. D. Ernst, R. Holmes, and G. Fraser Defects4J: a database of existing faults to enable controlled testing studies for java programs. In Proceedings of the 2014 International Symposium on Software Testing and Analysis, pp. 437–440. Cited by: §6.3.

[^20]: D. Kocetkov, R. Li, L. B. Allal, J. Li, C. Mou, C. M. Ferrandis, Y. Jernite, M. Mitchell, S. Hughes, T. Wolf, et al. The stack: 3 tb of permissively licensed source code. Transactions on Machine Learning Research. Cited by: §6.1.

[^21]: Kwai-Klear Mini-swe-agent-plus: the 100-line ai agent that solves github issues with text-edit tool. GitHub. External Links: [Link](https://github.com/Kwai-Klear/mini-swe-agent-plus) Cited by: Table 6, §6.2.

[^22]: R. Li, L. B. Allal, Y. Zi, N. Muennighoff, D. Kocetkov, C. Mou, M. Marone, C. Akiki, J. Li, J. Chim, et al. StarCoder: may the source be with you!. arXiv preprint arXiv:2305.06161. Cited by: §6.1.

[^23]: Y. Li, D. Choi, J. Chung, N. Kushman, J. Schrittwieser, R. Leblond, T. Eccles, J. Keeling, F. Gimeno, A. Dal Lago, et al. Competition-level code generation with alphacode. Science 378 (6624), pp. 1092–1097. Cited by: §1.

[^24]: T. Liu, C. Xu, and J. McAuley RepoBench: benchmarking repository-level code auto-completion systems. In The Twelfth International Conference on Learning Representations, Cited by: §1, §6.3.

[^25]: I. Loshchilov and F. Hutter Decoupled weight decay regularization. External Links: 1711.05101, [Link](https://arxiv.org/abs/1711.05101) Cited by: §3.3.

[^26]: M. Luo, N. Jain, J. Singh, S. Tan, A. Patel, Q. Wu, A. Ariyak, C. Cai, T. Venkat, S. Zhu, B. Athiwaratkun, M. Roongta, C. Zhang, L. E. Li, R. A. Popa, K. Sen, and I. Stoica DeepSWE: training a fully open-sourced, state-of-the-art coding agent by scaling rl. Note: [https://www.together.ai/blog/deepswe](https://www.together.ai/blog/deepswe) Blog post External Links: [Link](https://www.together.ai/blog/deepswe) Cited by: 3rd item, §1, Table 6.

[^27]: Z. Luo, C. Xu, P. Zhao, Q. Sun, X. Geng, W. Hu, C. Tao, J. Ma, Q. Lin, and D. Jiang WizardCoder: empowering code large language models with evol-instruct. In The Twelfth International Conference on Learning Representations, Cited by: §6.1.

[^28]: Y. Ma, R. Cao, Y. Cao, Y. Zhang, J. Chen, Y. Liu, Y. Liu, B. Li, F. Huang, and Y. Li SWE-gpt: a process-centric language model for automated software improvement. Proceedings of the ACM on Software Engineering 2 (ISSTA), pp. 2362–2383. Cited by: Table 6.

[^29]: N. Muennighoff, Q. Liu, A. Zebaze, Q. Zheng, B. Hui, Z. Terry, K. Richardson, P. Lewis, and S. Longpre OctoPack: instruction tuning code large language models. arXiv preprint arXiv:2308.07124. Cited by: §6.1.

[^30]: OpenAI Hello gpt-4o. Note: [https://openai.com/zh-Hans-CN/index/hello-gpt-4o/](https://openai.com/zh-Hans-CN/index/hello-gpt-4o/) Published: 2024-05-13; Accessed: 2025-12-22 Cited by: Table 6.

[^31]: OpenAI Building more with gpt-5.1-codex-max. Note: [https://openai.com/index/gpt-5-1-codex-max/](https://openai.com/index/gpt-5-1-codex-max/) 2025-11-19 Cited by: §2.3.

[^32]: OpenAI Introducing gpt-5. Note: [https://openai.com/zh-Hans-CN/index/introducing-gpt-5/](https://openai.com/zh-Hans-CN/index/introducing-gpt-5/) Published: 2025-08-07; Accessed: 2025-12-22 Cited by: Table 6.

[^33]: OpenAI Introducing openai o3 and o4-mini. Note: [https://openai.com/zh-Hans-CN/index/introducing-o3-and-o4-mini/](https://openai.com/zh-Hans-CN/index/introducing-o3-and-o4-mini/) Published: 2025-04-16; Accessed: 2025-12-22 Cited by: Table 6.

[^34]: OpenHands Team OpenHands critic 32b exp 20250417. Hugging Face. Note: [https://huggingface.co/OpenHands/openhands-critic-32b-exp-20250417](https://huggingface.co/OpenHands/openhands-critic-32b-exp-20250417) Accessed: 2025-12-08 Cited by: 3rd item, §4.2, §4.2.

[^35]: J. Pan, X. Wang, G. Neubig, N. Jaitly, H. Ji, A. Suhr, and Y. Zhang Training software engineering agents and verifiers with swe-gym. In Forty-second International Conference on Machine Learning, ICML 2025, Vancouver, BC, Canada, July 13-19, 2025, External Links: [Link](https://openreview.net/forum?id=Cq1BNvHx74) Cited by: 3rd item, §2.2, §4.2, Table 6, Table 6.

[^36]: B. Peng, J. Quesnelle, H. Fan, and E. Shippole YaRN: efficient context window extension of large language models. External Links: 2309.00071, [Link](https://arxiv.org/abs/2309.00071) Cited by: §3.3.

[^37]: B. Rozière, J. Gehring, F. Gloeckle, S. Sootla, I. Gat, X. E. Tan, Y. Adi, J. Liu, T. Remez, J. Rapin, et al. Code llama: open foundation models for code. arXiv preprint arXiv:2308.12950. Cited by: §6.2.

[^38]: D. Shrivastava, H. Larochelle, and D. Tarlow RepoFusion: training code models to understand your repository. arXiv preprint arXiv:2306.10998. Cited by: §1, §6.2.

[^39]: A. Sonwane, I. White, H. Lee, M. Pereira, L. Caccia, M. Kim, Z. Shi, C. Singh, A. Sordoni, M. Côté, et al. BugPilot: complex bug generation for efficient learning of swe skills. arXiv preprint arXiv:2510.19898. Cited by: §A.2, §1.

[^40]: O. Team OpenDevin: an open platform for ai software developers. GitHub. Note: [https://github.com/OpenDevin/OpenDevin](https://github.com/OpenDevin/OpenDevin) Cited by: §6.2.

[^41]: S. Team Mini-swe-agent. GitHub. Note: [https://github.com/SWE-agent/Mini-SWE-Agent](https://github.com/SWE-agent/Mini-SWE-Agent) Cited by: §6.2.

[^42]: Teknium OpenHermes 2.5: an open source dataset of synthetic data for generalist llm assistants. Hugging Face. External Links: [Link](https://huggingface.co/datasets/teknium/OpenHermes-2.5) Cited by: §6.1.

[^43]: H. Wang, Z. Hou, Y. Wei, J. Tang, and Y. Dong SWE-dev: building software engineering agents with training and inference scaling. arXiv preprint arXiv:2506.07636. Cited by: Table 6, Table 6.

[^44]: J. Wang, D. Zan, S. Xin, S. Liu, Y. Wu, and K. Shen Swe-mirror: scaling issue-resolving datasets by mirroring issues across repositories. arXiv preprint arXiv:2509.08724. Cited by: 2nd item, §1, Table 6, Table 6.

[^45]: X. Wang, B. Li, Y. Song, F. F. Xu, X. Tang, M. Zhuge, J. Pan, Y. Song, B. Li, J. Singh, et al. Openhands: an open platform for ai software developers as generalist agents. arXiv preprint arXiv:2407.16741. Cited by: §2.3.

[^46]: Y. Wei, Z. Sun, E. McMilin, J. Gehring, D. Zhang, G. Synnaeve, D. Fried1, L. Zhang, and S. Wang Toward training superintelligent software agents through self-play swe-rl. arXiv preprint arXiv:2512.18552. Cited by: §1.

[^47]: Y. Wei, Z. Wang, J. Liu, Y. Ding, and L. Zhang Magicoder: empowering code generation with oss-instruct. arXiv preprint arXiv:2312.02120. Cited by: §6.1.

[^48]: C. S. Xia, Y. Deng, S. Dunn, and L. Zhang Agentless: demystifying llm-based software engineering agents. External Links: 2407.01489 Cited by: §1, §6.2.

[^49]: C. S. Xia, Z. Wang, Y. Yang, Y. Wei, and L. Zhang Live-swe-agent: can software engineering agents self-evolve on the fly?. arXiv preprint arXiv:2511.13646. Cited by: §6.2.

[^50]: J. Xu, K. Deng, W. Li, S. Yu, H. Tang, H. Huang, Z. Lai, Z. Zhan, Y. Wu, C. Zhang, K. Lei, Y. Yao, X. Lei, W. Zhu, Z. Feng, H. Li, J. Xiong, D. Li, Z. Gao, K. Wu, W. Xiang, Z. Zhan, Y. Zhang, W. Gong, and Z. Gao SWE-compass: towards unified evaluation of agentic coding abilities for large language models. External Links: 2511.05459 Cited by: §6.3.

[^51]: A. Yang, A. Li, B. Yang, B. Zhang, B. Hui, B. Zheng, B. Yu, C. Gao, C. Huang, C. Lv, C. Zheng, D. Liu, F. Zhou, F. Huang, F. Hu, H. Ge, H. Wei, H. Lin, J. Tang, J. Yang, J. Tu, J. Zhang, J. Yang, J. Yang, J. Zhou, J. Zhou, J. Lin, K. Dang, K. Bao, K. Yang, L. Yu, L. Deng, M. Li, M. Xue, M. Li, P. Zhang, P. Wang, Q. Zhu, R. Men, R. Gao, S. Liu, S. Luo, T. Li, T. Tang, W. Yin, X. Ren, X. Wang, X. Zhang, X. Ren, Y. Fan, Y. Su, Y. Zhang, Y. Zhang, Y. Wan, Y. Liu, Z. Wang, Z. Cui, Z. Zhang, Z. Zhou, and Z. Qiu Qwen3 technical report. arXiv preprint arXiv:2505.09388. Cited by: §2.3, §3.3, Table 6, Table 6.

[^52]: J. a. Yang, S. Yao, K. Narasimhan, and D. Tao InterCode: standardizing and benchmarking interactive coding with execution feedback. In Thirty-seventh Conference on Neural Information Processing Systems Datasets and Benchmarks Track, Cited by: §1, §6.3.

[^53]: J. Yang, C. E. Jimenez, A. L. Zhang, K. Lieret, J. Yang, X. Wu, O. Press, N. Muennighoff, G. Synnaeve, K. R. Narasimhan, D. Yang, S. I. Wang, and O. Press SWE-bench multimodal: do ai systems generalize to visual software domains?. In The Thirteenth International Conference on Learning Representations, External Links: [Link](https://openreview.net/forum?id=riTiq3i21b) Cited by: §6.3.

[^54]: J. Yang, K. Lieret, C. E. Jimenez, A. Wettig, K. Khandpur, Y. Zhang, B. Hui, O. Press, L. Schmidt, and D. Yang SWE-smith: scaling data for software engineering agents. External Links: 2504.21798 Cited by: 1st item, 1st item, 2nd item, §1, §2.2, §2.2, Table 6, Table 6, §6.1.

[^55]: J. Yang, C. E. Yalcin, K. Narasimhan, and S. Yao SWE-agent: agent-computer interfaces enable software engineering language models. arXiv preprint arXiv:2405.15793. Cited by: §1, §2.1, §6.2.

[^56]: Z. Yang, S. Wang, K. Fu, W. He, W. Xiong, Y. Liu, Y. Miao, B. Gao, Y. Wang, Y. Ma, et al. Kimi-dev: agentless training as skill prior for swe-agents. arXiv preprint arXiv:2509.23045. Cited by: §1.

[^57]: D. Zan, Z. Huang, W. Liu, H. Chen, L. Zhang, S. Xin, L. Chen, Q. Liu, X. Zhong, A. Li, S. Liu, Y. Xiao, L. Chen, Y. Zhang, J. Su, T. Liu, R. Long, K. Shen, and L. Xiang Multi-swe-bench: a multilingual benchmark for issue resolving. External Links: 2504.02605 Cited by: §6.2, §6.3.

[^58]: L. Zeng, Y. Li, Y. Xiao, C. Li, C. Y. Liu, R. Yan, T. Wei, J. He, X. Song, Y. Liu, et al. Skywork-swe: unveiling data scaling laws for software engineering in llms. arXiv preprint arXiv:2506.19290. Cited by: §2.2, Table 6.

[^59]: L. Zhang, A. Hosseini, H. Bansal, M. Kazemi, A. Kumar, and R. Agarwal Generative verifiers: reward modeling as next-token prediction. In The Thirteenth International Conference on Learning Representations, ICLR 2025, Singapore, April 24-28, 2025, External Links: [Link](https://openreview.net/forum?id=Ccwp4tFEtE) Cited by: §4.2.

[^60]: Y. Zhang, H. Ruan, Z. Fan, and A. Roychoudhury AutoCodeRover: autonomous program improvement. In Proceedings of the 33rd ACM SIGSOFT International Symposium on Software Testing and Analysis, Cited by: §1, §6.2.

[^61]: Y. Zheng, R. Zhang, J. Zhang, Y. Ye, Z. Luo, Z. Feng, and Y. Ma LlamaFactory: unified efficient fine-tuning of 100+ language models. In Proceedings of the 62nd Annual Meeting of the Association for Computational Linguistics (Volume 3: System Demonstrations), Bangkok, Thailand. External Links: [Link](http://arxiv.org/abs/2403.13372) Cited by: §3.3.