---
title: "SWE-rebench: An Automated Pipeline for Task Collection and Decontaminated Evaluation of Software Engineering Agents"
source: "https://ar5iv.labs.arxiv.org/html/2505.20411"
author:
published:
created: 2026-10-26
description: "LLM-based agents have shown promising capabilities in a growing range of software engineering (SWE) tasks. However, advancing this field faces two critical challenges. First, high-quality training data is scarce, espec…"
tags:
  - "clippings"
---
Ibragim Badertdinov Thanks: Equal contribution. Correspondence to ibragim-bad@nebius.com Affiliation: Nebius    Alexander Golubev <sup>1</sup> Affiliation: Nebius    Maksim Nekrashevich Affiliation: Nebius    Anton Shevtsov Affiliation: Nebius    Simon Karasik Affiliation: Nebius    Andrei Andriushchenko Affiliation: Nebius    Maria Trofimova Affiliation: Nebius    Daria Litvintseva Affiliation: Nebius    Boris Yangel Affiliation: Nebius

###### Abstract

LLM-based agents have shown promising capabilities in a growing range of software engineering (SWE) tasks. However, advancing this field faces two critical challenges. First, high-quality training data is scarce, especially data that reflects real-world SWE scenarios, where agents must interact with development environments, execute code and adapt behavior based on the outcomes of their actions. Existing datasets are either limited to one-shot code generation or comprise small, manually curated collections of interactive tasks, lacking both scale and diversity. Second, the lack of fresh interactive SWE tasks affects evaluation of rapidly improving models, as static benchmarks quickly become outdated due to contamination issues. To address these limitations, we introduce a novel, automated, and scalable pipeline to continuously extract real-world interactive SWE tasks from diverse GitHub repositories. Using this pipeline, we construct SWE-rebench, a public dataset comprising over 21,000 interactive Python-based SWE tasks, suitable for reinforcement learning of SWE agents at scale. Additionally, we use continuous supply of fresh tasks collected using SWE-rebench methodology to build a contamination-free benchmark for agentic software engineering. We compare results of various LLMs on this benchmark to results on SWE-bench Verified and show that performance of some language models might be inflated due to contamination issues.

## 1 Introduction

Large Language Models (LLMs) have demonstrated impressive capabilities in SWE tasks, including code generation, debugging, and automated development workflows. Building on these capabilities, researchers have begun creating LLM-driven agents that interact with real codebases and development environments, performing actions and receiving feedback [^1]. While frontier proprietary models drive the performance of the most competitive agents (e.g., OpenHands [^2], Moatless Tools [^3], Agentless [^4]) on key benchmarks like SWE-bench [^5], there exists a significant opportunity to enhance open-source models [^6] [^7] [^8] [^9] [^10] [^11] [^12]. Progress in this direction, particularly toward complex agentic behaviors, may be accelerated with access to large-scale, high-quality training data that mirrors the interactivity inherent in real-world software development. Existing powerful open-source models like DeepSeek-V3 [^13], LLaMa 4 [^14] and Qwen3 [^15] could potentially be fine-tuned to achieve comparable performance in specific SWE domains, but this hinges on the availability of suitable interactive task data.

Current approaches to training LLMs for programming often rely on code data from open-source repositories [^16] or synthetic instruction datasets [^17] that are used for instruction tuning. However, training robust software engineering agents for real-world scenarios necessitates datasets that extend beyond simple code generation. To truly enable learning through methods like Reinforcement Learning (RL), which thrives on trial-and-error, agents require interactive tasks coupled with automatic verification mechanisms. Such data must allow agents to perform diverse actions, observe environment responses after each step, and receive eventual verification outcomes that determine task success. Unlike domains such as mathematics [^18] or web navigation [^19], software engineering has historically lacked such large-scale interactive datasets due to the complexities of configuring diverse, executable environments at scale. While recent efforts like SWE-Gym [^20] and SWE-PolyBench [^21] represent promising steps, their manual curation processes and reliance on a limited number of repositories constrain their scope, diversity, and scalability.

Furthermore, the evaluation of rapidly advancing LLM-based agents also faces significant challenges. Static benchmarks, while initially valuable, can become compromised by data contamination as newer models become exposed to test instances during their extensive pre/post-training. Moreover, the lack of standardized evaluation protocols, variability in agent scaffolds and inconsistent reporting practices make direct comparisons between models difficult and can obscure their true capabilities.

To address these challenges in both training data availability and evaluation reliability, we present a scalable, fully automated pipeline for continuous collection of software engineering tasks from real-world GitHub repositories. Building upon our prior work such as SWE-bench Extra [^22], which has been well-received by the community and is already used to train open-source software engineering agents [^8], our approach eliminates manual intervention and significantly expands task diversity and scale. To the best of our knowledge, this is the first system enabling fully automated, scalable collection of executable tasks from a wide set of real-world repositories, specifically designed to support interactive agent training and robust benchmarking.

Our main contributions are as follows:

- A scalable and fully automated pipeline for mining real-world software engineering tasks from GitHub, covering environment configuration, build setup, and test validation.
- SWE-rebench <sup>1</sup>, a public dataset of more than 21,000 interactive Python-based SWE tasks, designed to train and benchmark agents in diverse executable environments, particularly suitable for reinforcement learning-based approaches.
- A public SWE-rebench leaderboard <sup>2</sup> that offers continuously updated, decontaminated, and standardized evaluations for LLM-based agents, promoting transparency and fair comparisons across both open- and closed-source models.

By focusing on scale and automation, SWE-rebench aims to fill a critical gap in the LLM agent ecosystem. We believe it will serve as a foundational resource for accelerating open-source research and improving the reliability and performance of LLM-based software engineering agents.

## 2 An automated pipeline for collecting software engineering tasks

In this section we describe our automated pipeline for mining verifiable software engineering tasks at scale, that we used to build SWE-rebench, a dataset of 21,336 verifiable SWE tasks from 3468 distinct GitHub repositiories. Our pipeline comprises four stages: preliminary task collection, automated installation instruction configuration, execution-based installation verification, and quality assessment, which are fully described in this section. While our methodology incorporates several techniques from SWE-bench, it also introduces innovations to enhance automation and scalability. We detail the distinctions and novel aspects of our approach compared to the original SWE-bench methodology in Appendix G. The computationally intensive nature of our pipeline is managed through a distributed storage and computing platform TractoAI [^23], which provides capabilities for efficient parallel processing and data management, helping us optimize throughput of each stage to enable rapid reprocessing whenever we change the pipeline.

### 2.1 Preliminary task collection

In the first stage, we download raw input data from multiple origins, merge them, and perform preliminary filtering. The primary sources for our data are GitHub Archive [^24] and GitHub.

- GitHub Archive. The GitHub Archive is a major source of public events on GitHub. Each day, it publishes a JSON archive listing all GitHub events from that day. We use this archive to collect detailed data about issues: issue description, discussion, linked pull requests, and metadata such as creation date and labels. We also extract information about pull requests, including their merge status, last commit, and discussions.
- GitHub. We clone relevant GitHub repositories with their full commit histories to our local storage. A local copy enables efficient access to repository data and helps avoid GitHub API rate limits. We use preserved commit history to identify changes associated with pull requests and perform version analysis for automated dependency setup in later stages.

To initiate the dataset building process, we download approximately 450,000 pull requests linked to issues created before May 1, 2025. These originate from over 30,000 repositories that feature permissive licenses granting broad usage rights (see Appendix D for the list of included license types) and where Python constitutes over 75% of the codebase lines of code.

We then link issues with pull requests that mention resolving them in their title or description, applying filters to select instances where:

- The issue is from a Python repository with a permissive license.
- The issue is marked as resolved.
- The PR is merged into the main branch.
- The PR is not linked to multiple issues.
- The issue description is longer than 10 characters.
- The PR must introduces or modifies tests and includes code changes beyond test files.
- Changes affect 1 to 15 files.

This filtering aims to eliminate unsuitable candidates, particularly those lacking tests. We require pull requests that introduce or modify tests, as these are crucial for automatically evaluating whether a proposed code patch resolves the described issue. For each selected pull request, the overall patch is divided into two components: a solution patch, containing changes to non-test files intended to address the issue, and a test patch, comprising only changes to test files. After applying all filtering criteria, approximately 153,400 potential task instances remain.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2505.20411/assets/data_collection_v2.png)

Figure 1: Overview of the automated pipeline for collecting software engineering data.

### 2.2 Automated installation instructions configuration

Datasets like SWE-bench [^5] or SWE-Gym [^20] rely on manual curation to configure executable environments for each repository. This approach inherently limits scalability, often confining such datasets to a small selection of well-known repositories. Key steps typically include project versioning (mapping multiple task instances to a single valid environment) and defining setup instructions (to install dependencies and run tests). Manually conducting these steps on a large-scale, diverse task collection is infeasible; therefore, we employ a fully automated approach.

After preliminary filtering described in Section 2.1, remaining issues are treated as task instances. We group these task instances by project versions inferred from `git tag` outputs, normalizing versions to `major.minor` format (e.g., `1.2.3` is normalized to `1.2`). For each version group, we select the `base_commit` of the pull request linked to the task instance with the most recent `base_commit` date. We prioritize this most recent `base_commit` because developers typically maintain dependency compatibility within minor versions, and later commits often include important environment fixes. This approach generally provides a stable dependency set, often sufficient for executing test patches from all tasks in that group within a shared environment. The `git tag` command provides a version for approximately 95% of task instances. We assign a unique version to the rest of the tasks, so that each one of them uses its own environment. We employ an agentless approach, inspired by [^4], to generate candidate environment setup instructions. This process involves several LLM-driven steps:

- Identifying relevant files: An LLM scans repository files (e.g., `README.md`, `Dockerfile`, `setup.py`) to find potential sources of installation information.
- Extracting installation recipe: The LLM processes the concatenated content of files identified in the previous stage to produce a structured JSON object detailing the installation recipe. Files are provided to the LLM in the format: `<filename>F.ext</filename>\n<content>CONTENT</content>`. An example of the LLM’s reasoning and the resulting JSON recipe is provided in Appendix B.3.

We use the Qwen2.5-72B-Instruct model [^25] (prompt in Appendix B.2) to generate up to three candidate JSON recipes per task. If an error occurs during the subsequent scripted installation or test execution (derived from a recipe), the LLM attempts to refine that recipe by analyzing error logs and the original instructions (see correction prompt in Appendix B.4). This iterative refinement enables successful environment configuration for tasks with issues like missing libraries or incorrect setups, allowing their inclusion in the final dataset. Our approach successfully produces a working installation recipe for at least one task in 31% of all repositories.

We also explored dependency installation using an interactive agent that directly interacts with a Docker environment to install projects and run tests. While this interactive agent occasionally configured environments more effectively, it proved to be significantly more resource-demanding. The chosen agentless method is more computationally efficient for large-scale processing, and generating multiple candidate recipes can further improve its effectiveness, making it our primary approach. A comparative evaluation of these approaches on a curated subset of SWE-bench tasks is detailed in Appendix C.

### 2.3 Execution-based installation verification

To confirm task solvability and the integrity of the provided tests, we perform execution-based installation verification. This stage involves installing the environment for each task within a container and executing the tests from the pull request’s test patch. We then parse the test run outputs to ensure that: (1) at least one test from the test patch fails before applying the solution patch (i.e., changes to non-test files from the original pull request), (2) all tests from the test patch that initially failed subsequently pass after the solution patch is applied, and (3) any tests from the test patch that initially passed continue to pass after the solution patch is applied. Tasks are considered valid only if they meet these conditions.

Processing numerous task instances, each potentially with multiple candidate recipes requiring installation, testing, and logging, necessitates distributed execution to manage the workload efficiently. We use TractoAI for this purpose, as it enables distributed container building and parallel execution of verification tasks in built containers across a cluster.

For installation verification, we use a default base container with pre-installed basic dependencies (e.g., conda, gcc). Our own image registry and internal PyPI/APT mirrors help cache popular dependencies, accelerating container launches and reducing reliance on external sources. During task verification we perform the following steps:

- Install project dependencies in an isolated container using buildah. We utilize tmpfs for file system operations to minimize disk I/O and accelerate builds.
- Execute tests and parse logs to identify tests validating the solution.
- Build final container images upon successful verification.
- Record exact dependency versions (via `pip freeze`, `conda env export`) after a successful setup to mitigate reproducibility issues from unpinned dependencies in Python projects, ensuring consistent environment recreation in the future.

### 2.4 Automated instance quality assessment

For our collected tasks to be effectively used for reinforcement learning, they should possess certain properties; otherwise, RL agents might generate trajectories that appear as failures but are actually due to task imperfections (e.g., an underspecified issue making the task unsolvable, or flawed tests that a correct solution cannot pass), leading to incorrectly penalizing the agent. While SWE-bench Verified ensures these properties through manual verification, the scale of our collection necessitates an automated approximation of these checks. To assess these properties automatically, we fine-tune an instruction-following model using human annotations from SWE-bench Verified to predict:

- Issue Clarity: Whether the GitHub issue description is sufficiently detailed for a developer to understand and solve the problem.
- Task Complexity: The estimated effort to resolve the issue, considering reasoning, code modification, and codebase familiarity.
- Test Patch Correctness: Whether tests in the pull request accurately verify the intended fix without over-reliance on specific implementation details.

We fine-tune Qwen 2.5-72B-Instruct using annotations from SWE-bench Verified. For each of the over 3,800 examples, the model receives the issue description, the canonical solution patch, and the test patch as input. It is then prompted to predict one of three binary quality labels: Issue Clarity, Task Complexity, or Test Patch Correctness. We train the model to predict each label independently; each task instance is assessed for each quality characteristic separately using a 75/25 training/validation split (total 413 validation examples).

For Task Complexity (where ’high-score’ implies >1 hour to solve; 100 high-score vs. 313 low-score examples in validation), our fine-tuned model achieved 81% accuracy and a weighted F1-score of 0.82. This is an improvement over the baseline Qwen-72B-Instruct, which achieved 68% accuracy. For Test Patch Correctness (180 high-score vs. 233 low-score examples), the model achieved 67% accuracy (weighted F1: 0.65). For Issue Clarity (84 high-score vs. 329 low-score examples), it achieved 79% accuracy (weighted F1: 0.76). A more detailed prediction quality analysis, including precision and recall per class, can be found in Appendix F, Table 4.

The LLM-generated labels for Issue Clarity, Task Complexity, and Test Patch Correctness are provided as metadata with each task instance. While this automated assessment is not perfect, these labels offer users a means to filter the dataset and select task instances according to their specific criteria. For example, these labels facilitate task difficulty control more precise than heuristics like the number of modified files (used, for example, for SWE-bench Lite subset), as the number of changed files can be misleading: a multi-file change might be simple (e.g., a repeated parameter update), while a single-file change might lack clear issue descriptions or adequate tests for full validation. Thus, these LLM-based scores for difficulty and clarity of description and tests empower users to perform more nuanced task selection, helping them identify challenging yet solvable and clearly specified tasks beneficial for their specific model training or evaluation needs, and potentially aiding in mitigating benchmark saturation.

This four-stage pipeline automates the collection and processing of interactive software engineering tasks. The process yields the SWE-rebench dataset of 21,336 annotated task instances, which is publicly available on [Hugging Face Datasets](https://huggingface.co/datasets/nebius/SWE-rebench). Accompanying code for utilizing the dataset, including scripts for tasks evaluation, is available on [GitHub](https://github.com/SWE-rebench/SWE-bench-fork). An example of a task instance with its full annotation is provided in Appendix E.

## 3 SWE-rebench benchmark

In this section we discuss key limitations of existing evaluation setups for LLM-based software engineering agents and how our automated data pipeline described in Section 2 helps to address them. We leverage this pipeline to construct [SWE-rebench](http://swe-rebench.com/), a benchmark built from hundreds of real-world, executable SWE tasks. It comprises 294 executable tasks from 169 diverse repositories, selected using filtering criteria detailed in Appendix H, and is part of the broader SWE-rebench dataset release. To ensure reliable and standardized evaluation, we maintain a private leaderboard based on this benchmark.

### 3.1 Challenges in SWE agent benchmarking

We identified the following key areas for improvement:

- Potential data contamination: SWE-bench, the de facto evaluation standard for SWE agents, has been public since late 2023. Models released afterward may have been exposed to its data during training, risking inflated scores and confounding generalization with memorization.
- Incomparable results due to scaffolding variability: Current evaluation practices allow for a wide range of setups. Performance on SWE-bench is often heavily influenced by highly engineered prompts, complex multi-agent frameworks, retry mechanisms, best-of-N sampling strategies and validation loops. While these techniques demonstrate the potential of systems built around LLMs, they make it difficult to isolate and compare raw capabilities of different LLMs. Furthermore, the scaffoldings are often developed and tuned on subsets from SWE-bench, inadvertently leading to a potential for implicit overfitting to the benchmark’s specific characteristics.
- Lack of standardized and verifiable evaluation: SWE-bench results are typically performed and reported by individual teams. This decentralized approach lacks a mechanism for independent verification and can potentially lead to inconsistencies or misleading reporting practices such as reporting pass@N as pass@1 or implicitly using information derived from final tests. The reliance on closed-source frameworks for many submissions further reduces the transparency and reproducibility of the evaluation process.
- High variance in agent performance across runs: Due to the stochastic nature of agent trajectories, the outcome of a single run can vary significantly. This includes cases where a model may successfully generate correct actions or recover from mistakes in some runs, but fail to do so in others. Without averaging or reporting performance across multiple runs, the results can be unrepresentative. In particular, evaluating an agent multiple times and reporting only the best-performing run risks overstating the model’s actual capabilities and resolved rate.

### 3.2 Principles of SWE-rebench

SWE-rebench is designed to address the above challenges and support rigorous, model-centric evaluation through several core principles:

This standardized approach allows SWE-rebench to focus on measuring two fundamental aspects of model performance:

- The ability to comprehend a real-world software issue (presented as a GitHub issue), devise a plan, implement a correct code patch, and potentially validate the solution.
- The ability to follow instructions and operate within a structured agentic framework, which is represented by our ReAct scaffolding.

### 3.3 Result analysis

Table 1: Comparison of model performance on SWE-rebench Jan 2025 and SWE-rebench (Mar–Apr 2025). All metrics are reported in percentages. Models released after 1st of March 2025 are denoted with an asterisk (\*).

<table><thead><tr><th>Model</th><th colspan="3">SWE-rebench (Jan)</th><th colspan="3">SWE-rebench (Mar–Apr)</th></tr><tr><th></th><th>Resolved</th><th>SEM</th><th>Pass@5</th><th>Resolved</th><th>SEM</th><th>Pass@5</th></tr></thead><tbody><tr><th>gpt-4.1-2025-04-14*</th><td>31.1</td><td>0.79</td><td>44.4</td><td>26.7</td><td>1.09</td><td>39.0</td></tr><tr><th>DeepSeek-V3-0324*</th><td>21.7</td><td>1.64</td><td>35.0</td><td>21.3</td><td>0.98</td><td>32.4</td></tr><tr><th>DeepSeek-V3-1226</th><td>19.1</td><td>0.58</td><td>33.3</td><td>21.9</td><td>1.44</td><td>31.4</td></tr><tr><th>Qwen3-235B-A22B no-think*</th><td>15.2</td><td>1.76</td><td>29.9</td><td>16.6</td><td>0.93</td><td>25.7</td></tr><tr><th>Qwen3-235B-A22B think*</th><td>13.7</td><td>1.81</td><td>29.9</td><td>12.2</td><td>1.33</td><td>25.7</td></tr><tr><th>Qwen3-32B no-think*</th><td>13.2</td><td>1.17</td><td>23.9</td><td>13.7</td><td>1.03</td><td>26.7</td></tr><tr><th>Qwen3-32B think*</th><td>11.8</td><td>0.99</td><td>22.2</td><td>11.2</td><td>0.56</td><td>21.0</td></tr><tr><th>Llama-4-Maverick-Instruct*</th><td>8.5</td><td>0.90</td><td>20.5</td><td>12.2</td><td>1.69</td><td>27.6</td></tr><tr><th>Qwen2.5-72B-Instruct</th><td>8.2</td><td>0.79</td><td>18.8</td><td>9.3</td><td>1.26</td><td>19.0</td></tr><tr><th>Llama-3.3-70B-Instruct</th><td>8.2</td><td>0.44</td><td>15.4</td><td>11.2</td><td>0.47</td><td>22.9</td></tr><tr><th>Llama-4-Scout-Instruct*</th><td>5.0</td><td>0.63</td><td>12.8</td><td>5.3</td><td>0.38</td><td>14.3</td></tr><tr><th>gemma-3-27b-it*</th><td>4.3</td><td>0.81</td><td>9.4</td><td>4.8</td><td>0.30</td><td>10.5</td></tr><tr><th>Qwen2.5-Coder-32B-Instruct</th><td>2.7</td><td>0.57</td><td>7.7</td><td>3.2</td><td>0.77</td><td>9.5</td></tr></tbody></table>

We leverage the decontaminated nature of SWE-rebench to analyze performance trends over time and identify potential signs of contamination effects in prior benchmarks. Specifically, we evaluate models on two distinct temporal subsets of tasks: those created in January 2025 and those from March–April 2025. Table 1 presents model performance across these time windows.

To investigate potential overfitting to the SWE-bench Verified dataset, we compare model performance on SWE-rebench tasks to the same models’ performance on SWE-bench Verified. This comparison focuses on open-source models released in 2024 or early 2025, for which the risk of data leakage from the Verified subset is higher. Table 2 summarizes the comparative results on SWE-bench Verified and the March-April 2025 slice of SWE-rebench.

The results from this evaluation showcase several notable observations:

- GPT-4.1 is the only model, which performance noticeably declined on the March–April subset compared to the January subset.
- LLaMa-4-Maverick exhibits a high pass@5 score relative to models with similar mean resolution rates, yet has a relatively modest resolution rate. This indicates that while the model can produce correct solutions to more complex problems, it lacks reliability across runs, demonstrating high potential but inconsistent execution.
- Qwen2.5-Coder-32B-Instruct underperforms expectations, especially considering its strong code generation capabilities. Analysis of its trajectories reveals problems with instruction following; the model frequently hallucinated environment responses or enters loops of formatting errors, ultimately failing without producing a meaningful solution attempt.
- Qwen3 models perform similarly with or without think mode enabled – in some cases, the no-think variant even slightly surpasses the think version. This suggests the base model’s capabilities are sufficiently strong for deliberate planning to provide no measurable advantage. The nearly identical pass@5 scores further indicate that the model’s problem-solving efficiency remains consistent even without explicit reasoning mechanisms
- DeepSeek models demonstrate the strongest performance among open-source models across both SWE-rebench subsets and the SWE-bench Verified benchmark. Notably, both the December and March releases of DeepSeek-V3 consistently outperform other open models in resolution rate and pass@5, highlighting their robustness to changes in task distribution.

Table 2: Comparison of model performance on SWE-bench Verified and SWE-rebench (Mar–Apr 2025). All metrics are reported in percentages.

<table><thead><tr><th>Model</th><th colspan="3">SWE-bench Verified</th><th colspan="3">SWE-rebench (Mar–Apr)</th></tr><tr><th></th><th>Resolved</th><th>SEM</th><th>Pass@5</th><th>Resolved</th><th>SEM</th><th>Pass@5</th></tr></thead><tbody><tr><th>DeepSeek-V3-0324</th><td>39.7</td><td>0.35</td><td>57.4</td><td>21.3</td><td>0.98</td><td>32.4</td></tr><tr><th>DeepSeek-V3-1226</th><td>35.2</td><td>0.52</td><td>51.0</td><td>21.9</td><td>1.44</td><td>31.4</td></tr><tr><th>LLaMA-3.3-70B-Instruct</th><td>18.1</td><td>0.66</td><td>31.6</td><td>11.2</td><td>0.47</td><td>22.9</td></tr><tr><th>LLaMA-4-Maverick-Instruct</th><td>16.0</td><td>0.79</td><td>39.2</td><td>12.2</td><td>1.69</td><td>27.6</td></tr><tr><th>Qwen2.5-72B-Instruct</th><td>11.3</td><td>0.35</td><td>27.8</td><td>9.3</td><td>1.26</td><td>19.0</td></tr><tr><th>LLaMA-4-Scout-Instruct</th><td>8.8</td><td>0.48</td><td>22.2</td><td>5.3</td><td>0.38</td><td>14.3</td></tr><tr><th>Qwen2.5-Coder-32B-Instruct</th><td>4.9</td><td>0.46</td><td>16.0</td><td>3.2</td><td>0.77</td><td>9.5</td></tr></tbody></table>

For evaluation details and experimental setup, see Appendix J.

## 4 Discussion and limitations

Our automated pipeline and the resulting SWE-rebench dataset are designed to address the lack of large-scale, real-world tasks for agent-based training, and the need for up-to-date benchmarks that remain free from data contamination. By automating the extraction and validation of executable tasks, we enable broad coverage and continual supply of fresh data. However, the emphasis on scalability introduces trade-offs, particularly a reduced ability to manually curate and verify the quality and clarity of each individual task.

Extracting consistently high-quality, verifiable SWE tasks from diverse real-world GitHub repositories (Section 2) is an inherently imperfect process. While our multi-stage filtering, refinements to existing methodologies and automated dependency installation are designed for robustness at scale, they rely on heuristics and LLM-driven interpretations. For instance, our LLM-based approach to generating installation instructions from repository files (Qwen2.5-72B-Instruct, Section 2.2), while far more scalable than manual methods, was validated on a limited set of 18 repositories for prompt engineering and may not capture every project’s subtleties. Similarly, the automated task quality assessment (Section 2.4), where an LLM is fine-tuned on SWE-bench Verified task labels to predict complexity and relevance, serves as a valuable scalable proxy but cannot fully replicate nuanced human judgment, thus, containing errors and decreasing quality of the datasets.

Finally, while our benchmark is intended to support transparency and standardization in evaluating SWE agents, it may also accelerate the development of increasingly autonomous AI systems in software engineering. This progress brings potential risks, such as overreliance on AI-generated code or misuse of automated agents for introducing vulnerabilities. We believe that fostering openness, decontaminated evaluations, and rigorous benchmarking practices helps mitigate these concerns and contributes to responsible advancement of the field.

We outline following main limitations of our work:

- Automated task quality assessment: While we employ automated quality assessment, the fully automated pipeline may result in some tasks being imperfectly described or unsolvable solely from the issue. This can lead to lower absolute success rates compared to manually curated benchmarks.
- Limited language diversity: The initial version of SWE-rebench and its underlying dataset are focused exclusively on Python-based tasks. Fundamentally, our pipeline is language-agnostic and can be extended to incorporate tasks from projects utilizing other programming languages.

## 5 Conclusion and future work

We have introduced a novel, fully automated pipeline for the continuous collection of software engineering tasks from open-source repositories. This pipeline provides a scalable and reliable source of fresh, decontaminated data, particularly suitable for training and evaluating LLM-based agents. It serves as the foundation for SWE-rebench public datasets and a continuously updated benchmark designed for robust and transparent evaluation of agent performance in realistic SWE scenarios. By shifting the paradigm towards automated data collection and dynamic benchmarking, we address critical limitations of existing static benchmarks and thereby facilitate more rapid and open progress in the field of AI for software engineering. Our future work will concentrate on several key areas:

- Expanding data coverage and volume: We aim to significantly increase the dataset volume by extending our collection methodology from tasks strictly tied to GitHub issues to a broader set of code changes represented by arbitrary pull requests.
- Improving task filtering pipeline: To improve the overall quality of extracted tasks, we aim to refine our filtering heuristics used in the pipeline.
- Support for new programming languages: Applying the same core methodology, we plan to collect datasets for projects in other popular languages (e.g., JavaScript, Java, C++), thereby expanding the linguistic and technological diversity of SWE-rebench.
- Keeping SWE-rebench benchmark up-to-date: Maintaining evaluation process on fresh tasks for the existing models, evaluating a wider range of LLMs and sharing detailed performance analyses with the community.

We believe that our automated data collection pipeline and the SWE-rebench benchmark provide a vital foundation for developing and assessing the next generation of LLM-based agents for real-world software engineering challenges.

## References

## Appendix A Related work

#### Benchmarks for software engineering

With the progress of LLM capabilities, there has been a growing need for benchmarks to evaluating their coding performance. While HumanEval [^27], MBPP [^28], and APPS [^29] have proven valuable for standalone function-level evaluation, their utility has largely saturated for state-of-the-art models. Consequently, LiveCodeBench [^30] has emerged, which is continuously updated to reflect the evolving capabilities of LLMs. However, these benchmarks often differ significantly from how LLMs are applied in real-world development workflows, which typically involve repository-level context and interaction. SWE-bench [^5] introduced execution-based, repository-level tasks sourced from real-world repositories, offering a more realistic evaluation paradigm. A challenge with SWE-bench is that many new LLMs were released after its construction, raising concerns about its continued relevance and potential contamination. Several follow-up works, such as SWE-bench+ [^31] and Multi-SWE-bench [^32], address specific shortcomings like post-cutoff filtering and multilingual annotation. Nevertheless, all these extensions fundamentally rely on a manually curated construction process. This manual element complicates timely updates and makes it difficult to keep the benchmark consistently aligned with the rapidly advancing capabilities of LLMs. In contrast, SWE-rebench automates the entire process of task creation from real-world repositories at scale. This automation enables a new class of dataset and benchmark: one that is task-rich, diverse, regularly updated, and suitable for both training and evaluation without the bottlenecks of manual curation.

#### Training datasets for code generation

Most prior work on collecting data to improve LLMs for code has focused on supervised fine-tuning [^33] [^34] [^35], often by leveraging samples from more powerful proprietary models. On the other hand, the remarkable success of large-scale reinforcement learning (RL) in math reasoning tasks [^36] [^37] [^38] implies that it should also excel in software engineering, as both domains involve rich, multi-step workflows with potentially verifiable outcomes. However, leveraging RL effectively in this setting demands gathering a large collection of real-world, interactive SWE tasks which is challenging, as each task requires the construction of a suitable and reliable execution environment. Some works attempt to gather such tasks by artificially injecting bugs into code [^7] or generating synthetic tests [^39]. While these approaches can generate data at scale, they may not fully capture the nuances and complexities of real-world software issues. The original SWE-bench dataset and its extensions for training [^20] provide valuable real-world tasks, but still suffer from limited diversity and scale due to their reliance on manual construction. In contrast, SWE-rebench reflects real-world development processes and preserves high task quality through its grounding in real repositories, while achieving significantly greater scale and diversity via its automated construction pipeline. This makes it particularly well-suited for training agents through an automated construction pipeline.

## Appendix B Automated dependency installation

As a reminder, the goal of automated dependency installation is to generate correct environment setup instructions for each task instance without using manual curation. The process is entirely LLM-driven and consists of the following sub-stages:

1. Identifying relevant files: An LLM is asked to analyze the list of repository files and select those most likely to contain installation instructions, such as README.md, setup.py, or requirements.txt.
2. Extracting installation recipe: Based on the content of files found in the previous stage, the model generates a structured JSON recipe specifying how to install dependencies and run tests for the repository.
3. Updating the recipe (if needed): If the recipe fails, another LLM call is made to analyze the failure and refine the recipe.

In the following subsections, we show the exact prompts used at each of these stages, along with examples of LLM-generated outputs. These materials are intended to provide transparency and facilitate reproducibility of our environment setup pipeline.

### B.1 Identifying installation-related files

The automated generation of installation recipes, a key part of our agentless approach described in Section 2.2, begins with identifying repository files that are likely to contain setup instructions. To achieve this, an LLM is provided with a list of all files present in the repository. The LLM’s task is to analyze this list and return a curated subset of file paths deemed most relevant for extracting installation, dependency, and testing information. The prompt below details the instructions given to the LLM for this file identification step. The output of this step (a JSON array of file paths) subsequently serves as input for the next LLM call, which extracts the actual installation commands.

Listing 1: LLM prompt for identifying installation-related files

⬇

You are tasked with identifying files that likely contain installation instructions for a GitHub repository.

Repository: {{ repo\_name }}

Below is a list of files in the repository that may be helpful for understanding the installation and setup process:

{{ repo\_name }}

Please analyze this list and identify the files that are most likely to contain information about:

\* Installation instructions

\* Dependencies or requirements

\* Setup procedures

\* Development environment configuration

\* Testing setup

Think step by step:

\* Identify README files, as they often contain installation instructions.

\* Look for setup.py, pyproject.toml, requirements.txt, environment.yml.

\* Consider files in directories like \\texttt{docs} that might contain installation guides.

\* Look for test configuration files that might help understand how to run tests.

\* Consider \\textbf{only} files from the list provided above.

\* Prioritize files in the repository root (top-level directory).

\* Only include files from subdirectories if they are clearly relevant to installation or setup.

Return \*\*only\*\* a JSON array containing the paths to the most relevant files for installation and setup. Include only files that are directly relevant to the tasks above. Sort the files from most to least relevant and limit your response to no more than 10 files, preferring fewer files that are truly essential.

For example:

\[

"README.md",

"setup.py",

"requirements.txt"

\]

### B.2 Extracting installation recipe

Following the identification of relevant files, the next step in our automated pipeline (Section 2.2) is to extract a concrete installation recipe. For this, the LLM receives the concatenated content of the previously selected relevant files. The primary task of the LLM is to analyze this content and synthesize a structured JSON object, installation recipe, which encapsulates all necessary commands and configurations to set up the project environment, install dependencies, and execute tests. The prompt presented below guides the LLM through this extraction process, emphasizing the expected JSON structure and providing context on how the generated recipe will be utilized by downstream automation scripts.

Listing 2: LLM prompt for extracting installation recipe

⬇

You are tasked with extracting detailed installation instructions from the following repository files. Repository: {{ repo\_name }}

Please analyze the content of these files and extract comprehensive installation instructions: {{ rendered\[:100000\] }}

First, think step by step. After your analysis, return your findings in the following JSON format:

{

"python": "3.9",

"packages": "requirements.txt",

"install": "pip install -e.\[dev\]",

"pre\_install": \["apt-get update", "apt-get install -y gcc"\],

"reqs\_path": \["requirements/base.txt"\],

"env\_yml\_path": \["environment.yml"\],

"pip\_packages": \["numpy>=1.16.0", "pandas>=1.0.0"\]

}

Here is how this JSON will be used:

\`\`\`bash

git clone <repo\_url> repo

cd repo

git checkout <base\_sha>

bash <pre\_install>

conda create -n \<repo> python=\<python> -y

conda activate \<repo>

if \<packages> == requirements.txt; then

for path in <reqs\_path>:

pip install -r $path

elif \<packages> == environment.yml; then

for path in <env\_yml\_path>:

conda env update -f $path

else:

pip install \<packages>

pip install <pip\_packages>

bash \<install>

bash <test\_cmd>

\`\`\`

\*\*IMPORTANT:\*\*

\* For the "install" field, always use local install commands like pip install -e.\[dev\]

\* Do NOT include packages in pip\_packages that will be installed by pip install -e.

\* Include only explicitly needed packages in pip\_packages.

\* reqs\_path and env\_yml\_path must match filenames from the provided files (e.g., \[File: filename\]).

\* If "packages" is requirements.txt, you must provide at least one reqs\_path.

\* Add relevant test frameworks to pip\_packages (e.g., pytest, nose).

\* Use -y in all conda commands.

\* Prefer direct and specific pytest commands over general wrappers.

\* Avoid test commands with placeholders like {test\_name}.

\* If a Makefile runs tests, extract the actual test command (e.g., pytest).

You must ensure the final JSON includes required fields (, install, test\_cmd), and optionally packages, pre\_install, reqs\_path, env\_yml\_path, pip\_packages if relevant.

Base your reasoning on all provided files and return it followed by the JSON output in a Markdown code block (without

[^1]: Haolin Jin, Linghan Huang, Haipeng Cai, Jun Yan, Bo Li, and Huaming Chen. From llms to llm-based agents for software engineering: A survey of current, challenges and future, 2025. URL [https://arxiv.org/abs/2408.02479](https://arxiv.org/abs/2408.02479).

[^2]: Xingyao Wang, Boxuan Li, Yufan Song, Frank F. Xu, Xiangru Tang, Mingchen Zhuge, Jiayi Pan, Yueqi Song, Bowen Li, Jaskirat Singh, Hoang H. Tran, Fuqiang Li, Ren Ma, Mingzhang Zheng, Bill Qian, Yanjun Shao, Niklas Muennighoff, Yizhe Zhang, Binyuan Hui, Junyang Lin, Robert Brennan, Hao Peng, Heng Ji, and Graham Neubig. OpenHands: An Open Platform for AI Software Developers as Generalist Agents, 2024. URL [https://arxiv.org/abs/2407.16741](https://arxiv.org/abs/2407.16741).

[^3]: Antonis Antoniades, Albert Örwall, Kexun Zhang, Yuxi Xie, Anirudh Goyal, and William Wang. Swe-search: Enhancing software agents with monte carlo tree search and iterative refinement, 2024. URL [https://arxiv.org/abs/2410.20285](https://arxiv.org/abs/2410.20285).

[^4]: Chunqiu Steven Xia, Yinlin Deng, Soren Dunn, and Lingming Zhang. Agentless: Demystifying llm-based software engineering agents, 2024. URL [https://arxiv.org/abs/2407.01489](https://arxiv.org/abs/2407.01489).

[^5]: Carlos E Jimenez, John Yang, Alexander Wettig, Shunyu Yao, Kexin Pei, Ofir Press, and Karthik R Narasimhan. SWE-bench: Can language models resolve real-world github issues? In *The Twelfth International Conference on Learning Representations*, 2024. URL [https://openreview.net/forum?id=VTF8yNQM66](https://openreview.net/forum?id=VTF8yNQM66).

[^6]: Xingyao Wang. Introducing openhands lm 32b – a strong, open coding agent model. *All Hands AI Blog*, March 2025. URL [https://www.all-hands.dev/blog/introducing-openhands-lm-32b----a-strong-open-coding-agent-model](https://www.all-hands.dev/blog/introducing-openhands-lm-32b----a-strong-open-coding-agent-model).

[^7]: John Yang, Kilian Leret, Carlos E. Jimenez, Alexander Wettig, Kabir Khandpur, Yanzhe Zhang, Binyuan Hui, Ofir Press, Ludwig Schmidt, and Diyi Yang. Swe-smith: Scaling data for software engineering agents, 2025. URL [https://arxiv.org/abs/2504.21798](https://arxiv.org/abs/2504.21798).

[^8]: Haoran Wang, Zhenyu Hou, Yao Wei, Jie Tang, and Yuxiao Dong. Swe-dev: Building software engineering agents with training and inference scaling. [https://ubecwang.notion.site/1bc32cf963e080b2a01df2895f66021f?v=1bc32cf963e0810ca07e000c86c4c1e1](https://ubecwang.notion.site/1bc32cf963e080b2a01df2895f66021f?v=1bc32cf963e0810ca07e000c86c4c1e1), 2025. Notion Blog.

[^9]: Vaibhav Aggarwal, Ojasv Kamal, Abhinav Japesh, Zhijing Jin, and Bernhard Schölkopf. Dars: Dynamic action re-sampling to enhance coding agent performance by adaptive tree traversal, 2025. URL [https://arxiv.org/abs/2503.14269](https://arxiv.org/abs/2503.14269).

[^10]: Yingwei Ma, Yongbin Li, Yihong Dong, Xue Jiang, Rongyu Cao, Jue Chen, Fei Huang, and Binhua Li. Thinking longer, not larger: Enhancing software engineering agents via scaling test-time compute, 2025. URL [https://arxiv.org/abs/2503.23803](https://arxiv.org/abs/2503.23803).

[^11]: Yuxiang Wei, Olivier Duchenne, Jade Copet, Quentin Carbonneaux, Lingming Zhang, Daniel Fried, Gabriel Synnaeve, Rishabh Singh, and Sida I. Wang. Swe-rl: Advancing llm reasoning via reinforcement learning on open software evolution, 2025. URL [https://arxiv.org/abs/2502.18449](https://arxiv.org/abs/2502.18449).

[^12]: Alexander Golubev, Sergey Polezhaev, Karina Zainullina, Maria Trofimova, Ibragim Badertdinov, Yuri Anapolskiy, Daria Litvintseva, Simon Karasik, Filipp Fisin, Sergey Skvortsov, Maxim Nekrashevich, Anton Shevtsov, Sergey Abramov, and Boris Yangel. Leveraging training and search for better software engineering agents. *Nebius blog*, 2024. URL [https://nebius.com/blog/posts/training-and-search-for-software-engineering-agents](https://nebius.com/blog/posts/training-and-search-for-software-engineering-agents).

[^13]: DeepSeek-AI. Deepseek-v3 technical report, 2024. URL [https://arxiv.org/abs/2412.19437](https://arxiv.org/abs/2412.19437).

[^14]: Meta AI. The llama 4 herd: The beginning of a new era of natively multimodal ai innovation, 2025. URL [https://ai.meta.com/blog/llama-4-multimodal-intelligence/](https://ai.meta.com/blog/llama-4-multimodal-intelligence/).

[^15]: Qwen Team. Qwen3, April 2025. URL [https://qwenlm.github.io/blog/qwen3/](https://qwenlm.github.io/blog/qwen3/).

[^16]: Anton Lozhkov, Raymond Li, Loubna Ben Allal, Federico Cassano, Joel Lamy-Poirier, Nouamane Tazi, Ao Tang, Dmytro Pykhtar, Jiawei Liu, Yuxiang Wei, Tianyang Liu, Max Tian, Denis Kocetkov, Arthur Zucker, Younes Belkada, Zijian Wang, Qian Liu, Dmitry Abulkhanov, Indraneil Paul, Zhuang Li, Wen-Ding Li, Megan Risdal, Jia Li, Jian Zhu, Terry Yue Zhuo, Evgenii Zheltonozhskii, Nii Osae Osae Dade, Wenhao Yu, Lucas Krauß, Naman Jain, Yixuan Su, Xuanli He, Manan Dey, Edoardo Abati, Yekun Chai, Niklas Muennighoff, Xiangru Tang, Muhtasham Oblokulov, Christopher Akiki, Marc Marone, Chenghao Mou, Mayank Mishra, Alex Gu, Binyuan Hui, Tri Dao, Armel Zebaze, Olivier Dehaene, Nicolas Patry, Canwen Xu, Julian McAuley, Han Hu, Torsten Scholak, Sebastien Paquet, Jennifer Robinson, Carolyn Jane Anderson, Nicolas Chapados, Mostofa Patwary, Nima Tajbakhsh, Yacine Jernite, Carlos Muñoz Ferrandis, Lingming Zhang, Sean Hughes, Thomas Wolf, Arjun Guha, Leandro von Werra, and Harm de Vries. Starcoder 2 and the stack v2: The next generation, 2024. URL [https://arxiv.org/abs/2402.19173](https://arxiv.org/abs/2402.19173).

[^17]: Yuxiang Wei, Federico Cassano, Jiawei Liu, Yifeng Ding, Naman Jain, Zachary Mueller, Harm de Vries, Leandro von Werra, Arjun Guha, and Lingming Zhang. Selfcodealign: Self-alignment for code generation, 2024. URL [https://arxiv.org/abs/2410.24198](https://arxiv.org/abs/2410.24198).

[^18]: Zhihong Shao, Peiyi Wang, Qihao Zhu, Runxin Xu, Junxiao Song, Xiao Bi, Haowei Zhang, Mingchuan Zhang, Y. K. Li, Y. Wu, and Daya Guo. Deepseekmath: Pushing the limits of mathematical reasoning in open language models, 2024. URL [https://arxiv.org/abs/2402.03300](https://arxiv.org/abs/2402.03300).

[^19]: Jiayi Pan, Yichi Zhang, Nicholas Tomlin, Yifei Zhou, Sergey Levine, and Alane Suhr. Autonomous evaluation and refinement of digital agents, 2024a. URL [https://arxiv.org/abs/2404.06474](https://arxiv.org/abs/2404.06474).

[^20]: Jiayi Pan, Xingyao Wang, Graham Neubig, Navdeep Jaitly, Heng Ji, Alane Suhr, and Yizhe Zhang. Training software engineering agents and verifiers with swe-gym, 2024b. URL [https://arxiv.org/abs/2412.21139](https://arxiv.org/abs/2412.21139).

[^21]: Muhammad Shihab Rashid, Christian Bock, Yuan Zhuang, Alexander Buchholz, Tim Esler, Simon Valentin, Luca Franceschi, Martin Wistuba, Prabhu Teja Sivaprasad, Woo Jung Kim, Anoop Deoras, Giovanni Zappella, and Laurent Callot. Swe-polybench: A multi-language benchmark for repository level evaluation of coding agents, 2025. URL [https://arxiv.org/abs/2504.08703](https://arxiv.org/abs/2504.08703).

[^22]: Ibragim Badertdinov, Maria Trofimova, Yuri Anapolskiy, Sergey Abramov, Karina Zainullina, Alexander Golubev, Sergey Polezhaev, Daria Litvintseva, Simon Karasik, Filipp Fisin, Sergey Skvortsov, Maxim Nekrashevich, Anton Shevtsov, and Boris Yangel. Scaling data collection for training software engineering agents. *Nebius blog*, 2024. URL [https://nebius.com/blog/posts/scaling-data-collection-for-training-swe-agents](https://nebius.com/blog/posts/scaling-data-collection-for-training-swe-agents).

[^23]: TractoAI. Tractoai: Cloud platform for deploying, scaling, and monitoring ai and big data workloads, 2025. URL [https://tracto.ai/](https://tracto.ai/).

[^24]: Ilya Grigorik. Gh archive, 2011. URL [https://www.gharchive.org/](https://www.gharchive.org/). Accessed 15 May 2025.

[^25]: Qwen,:, An Yang, Baosong Yang, Beichen Zhang, Binyuan Hui, Bo Zheng, Bowen Yu, Chengyuan Li, Dayiheng Liu, Fei Huang, Haoran Wei, Huan Lin, Jian Yang, Jianhong Tu, Jianwei Zhang, Jianxin Yang, Jiaxi Yang, Jingren Zhou, Junyang Lin, Kai Dang, Keming Lu, Keqin Bao, Kexin Yang, Le Yu, Mei Li, Mingfeng Xue, Pei Zhang, Qin Zhu, Rui Men, Runji Lin, Tianhao Li, Tianyi Tang, Tingyu Xia, Xingzhang Ren, Xuancheng Ren, Yang Fan, Yang Su, Yichang Zhang, Yu Wan, Yuqiong Liu, Zeyu Cui, Zhenru Zhang, and Zihan Qiu. Qwen2.5 technical report, 2025. URL [https://arxiv.org/abs/2412.15115](https://arxiv.org/abs/2412.15115).

[^26]: Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, and Yuan Cao. React: Synergizing reasoning and acting in language models, 2023. URL [https://arxiv.org/abs/2210.03629](https://arxiv.org/abs/2210.03629).

[^27]: Mark Chen, Jerry Tworek, Heewoo Jun, Qiming Yuan, Henrique Ponde de Oliveira Pinto, Jared Kaplan, Harri Edwards, Yuri Burda, Nicholas Joseph, Greg Brockman, Alex Ray, Raul Puri, Gretchen Krueger, Michael Petrov, Heidy Khlaaf, Girish Sastry, Pamela Mishkin, Brooke Chan, Scott Gray, Nick Ryder, Mikhail Pavlov, Alethea Power, Lukasz Kaiser, Mohammad Bavarian, Clemens Winter, Philippe Tillet, Felipe Petroski Such, Dave Cummings, Matthias Plappert, Fotios Chantzis, Elizabeth Barnes, Ariel Herbert-Voss, William Hebgen Guss, Alex Nichol, Alex Paino, Nikolas Tezak, Jie Tang, Igor Babuschkin, Suchir Balaji, Shantanu Jain, William Saunders, Christopher Hesse, Andrew N. Carr, Jan Leike, Josh Achiam, Vedant Misra, Evan Morikawa, Alec Radford, Matthew Knight, Miles Brundage, Mira Murati, Katie Mayer, Peter Welinder, Bob McGrew, Dario Amodei, Sam McCandlish, Ilya Sutskever, and Wojciech Zaremba. Evaluating large language models trained on code, 2021. URL [https://arxiv.org/abs/2107.03374](https://arxiv.org/abs/2107.03374).

[^28]: Jacob Austin, Augustus Odena, Maxwell Nye, Maarten Bosma, Henryk Michalewski, David Dohan, Ellen Jiang, Carrie Cai, Michael Terry, Quoc Le, and Charles Sutton. Program synthesis with large language models, 2021. URL [https://arxiv.org/abs/2108.07732](https://arxiv.org/abs/2108.07732).

[^29]: Dan Hendrycks, Steven Basart, Saurav Kadavath, Mantas Mazeika, Akul Arora, Ethan Guo, Collin Burns, Samir Puranik, Horace He, Dawn Song, and Jacob Steinhardt. Measuring coding challenge competence with apps, 2021. URL [https://arxiv.org/abs/2105.09938](https://arxiv.org/abs/2105.09938).

[^30]: Naman Jain, King Han, Alex Gu, Wen-Ding Li, Fanjia Yan, Tianjun Zhang, Sida Wang, Armando Solar-Lezama, Koushik Sen, and Ion Stoica. Livecodebench: Holistic and contamination free evaluation of large language models for code, 2024. URL [https://arxiv.org/abs/2403.07974](https://arxiv.org/abs/2403.07974).

[^31]: Reem Aleithan, Haoran Xue, Mohammad Mahdi Mohajer, Elijah Nnorom, Gias Uddin, and Song Wang. Swe-bench+: Enhanced coding benchmark for llms, 2024. URL [https://arxiv.org/abs/2410.06992](https://arxiv.org/abs/2410.06992).

[^32]: Daoguang Zan, Zhirong Huang, Wei Liu, Hanwu Chen, Linhao Zhang, Shulin Xin, Lu Chen, Qi Liu, Xiaojian Zhong, Aoyan Li, Siyao Liu, Yongsheng Xiao, Liangqiang Chen, Yuyu Zhang, Jing Su, Tianyu Liu, Rui Long, Kai Shen, and Liang Xiang. Multi-swe-bench: A multilingual benchmark for issue resolving, 2025. URL [https://arxiv.org/abs/2504.02605](https://arxiv.org/abs/2504.02605).

[^33]: Niklas Muennighoff, Qian Liu, Armel Zebaze, Qinkai Zheng, Binyuan Hui, Terry Yue Zhuo, Swayam Singh, Xiangru Tang, Leandro von Werra, and Shayne Longpre. Octopack: Instruction tuning code large language models, 2024. URL [https://arxiv.org/abs/2308.07124](https://arxiv.org/abs/2308.07124).

[^34]: Ziyang Luo, Can Xu, Pu Zhao, Qingfeng Sun, Xiubo Geng, Wenxiang Hu, Chongyang Tao, Jing Ma, Qingwei Lin, and Daxin Jiang. Wizardcoder: Empowering code large language models with evol-instruct, 2023. URL [https://arxiv.org/abs/2306.08568](https://arxiv.org/abs/2306.08568).

[^35]: Jiawei Liu, Thanh Nguyen, Mingyue Shang, Hantian Ding, Xiaopeng Li, Yu Yu, Varun Kumar, and Zijian Wang. Learning code preference via synthetic evolution, 2024. URL [https://arxiv.org/abs/2410.03837](https://arxiv.org/abs/2410.03837).

[^36]: DeepSeek-AI. Deepseek-r1: Incentivizing reasoning capability in llms via reinforcement learning, 2025. URL [https://arxiv.org/abs/2501.12948](https://arxiv.org/abs/2501.12948).

[^37]: ByteDance Seed,:, Jiaze Chen, Tiantian Fan, Xin Liu, Lingjun Liu, Zhiqi Lin, Mingxuan Wang, Chengyi Wang, Xiangpeng Wei, Wenyuan Xu, Yufeng Yuan, Yu Yue, Lin Yan, Qiying Yu, Xiaochen Zuo, Chi Zhang, Ruofei Zhu, Zhecheng An, Zhihao Bai, Yu Bao, Xingyan Bin, Jiangjie Chen, Feng Chen, Hongmin Chen, Riwei Chen, Liangqiang Chen, Zixin Chen, Jinsong Chen, Siyan Chen, Kaiyuan Chen, Zhi Chen, Jin Chen, Jiecao Chen, Jinxin Chi, Weinan Dai, Ning Dai, Jiahui Dai, Shihan Dou, Yantao Du, Zhengyin Du, Jianhui Duan, Chen Dun, Ting-Han Fan, Jiazhan Feng, Junda Feng, Ziyuan Feng, Yuwei Fu, Wenqi Fu, Hanjie Fu, Hao Ge, Hongyi Guo, Mingji Han, Li Han, Wenhao Hao, Xintong Hao, Qianyu He, Jerry He, Feng He, Wen Heng, Zehua Hong, Qi Hou, Liang Hu, Shengding Hu, Nan Hu, Kai Hua, Qi Huang, Ziyue Huang, Hongzhi Huang, Zihao Huang, Ting Huang, Wenhao Huang, Wei Jia, Bin Jia, Xiaoying Jia, Yuhua Jiang, Haobin Jiang, Ziheng Jiang, Kaihua Jiang, Chengquan Jiang, Jianpeng Jiao, Xiaoran Jin, Xing Jin, Xunhao Lai, Zheng Li, Xiang Li, Liyi Li, Hongkai Li, Zheng Li, Shengxian Wan, Ya Wang, Yunshui Li, Chenggang Li, Niuniu Li, Siyu Li, Xi Li, Xiao Li, Aoyan Li, Yuntao Li, Nianning Liang, Xinnian Liang, Haibin Lin, Weijian Lin, Ye Lin, Zhicheng Liu, Guanlin Liu, Guanlin Liu, Chenxiao Liu, Yan Liu, Gaohong Liu, Juncai Liu, Chundian Liu, Deyi Liu, Kaibo Liu, Siyao Liu, Qi Liu, Yongfei Liu, Kang Liu, Gan Liu, Boyi Liu, Rui Long, Weiqiang Lou, Chenwei Lou, Xiang Luo, Yao Luo, Caiping Lv, Heyang Lv, Bole Ma, Qianli Ma, Hongzhi Ma, Yiyuan Ma, Jin Ma, Wenchang Ma, Tingting Ma, Chen Mao, Qiyang Min, Zhe Nan, Guanghan Ning, Jinxiang Ou, Haojie Pan, Renming Pang, Yanghua Peng, Tao Peng, Lihua Qian, Lihua Qian, Mu Qiao, Meng Qu, Cheng Ren, Hongbin Ren, Yong Shan, Wei Shen, Ke Shen, Kai Shen, Guangming Sheng, Jinlong Shi, Wenlei Shi, Guang Shi, Shuai Shuai Cao, Yuxin Song, Zuquan Song, Jing Su, Yifan Sun, Tao Sun, Zewei Sun, Borui Wan, Zihan Wang, Xiaohui Wang, Xi Wang, Shuguang Wang, Jun Wang, Qinlong Wang, Chenyuan Wang, Shuai Wang, Zihan Wang, Changbao Wang, Jiaqiang Wang, Shihang Wang, Xuwu Wang, Zaiyuan Wang, Yuxuan Wang, Wenqi Wang, Taiqing Wang, Chengzhi Wei, Houmin Wei, Ziyun Wei, Shufa Wei, Zheng Wu, Yonghui Wu, Yangjun Wu, Bohong Wu, Shuang Wu, Jingqiao Wu, Ning Wu, Shuangzhi Wu, Jianmin Wu, Chenguang Xi, Fan Xia, Yuqiao Xian, Liang Xiang, Boren Xiang, Bowen Xiao, Zhen Xiao, Xia Xiao, Yongsheng Xiao, Chao Xin, Shulin Xin, Yuwen Xiong, Jingjing Xu, Ziwen Xu, Chenyin Xu, Jiayi Xu, Yifan Xu, Wei Xu, Yufei Xu, Shikun Xu, Shipeng Yan, Shen Yan, Qingping Yang, Xi Yang, Tianhao Yang, Yuehang Yang, Yuan Yang, Ximing Yang, Zeyu Yang, Guang Yang, Yifan Yang, Xuesong Yao, Bairen Yi, Fan Yin, Jianian Yin, Ziqiang Ying, Xiangyu Yu, Hongli Yu, Song Yu, Menghan Yu, Huan Yu, Siyu Yuan, Jun Yuan, Yutao Zeng, Tianyang Zhan, Zheng Zhang, Yun Zhang, Mofan Zhang, Wang Zhang, Ru Zhang, Zhi Zhang, Tianqi Zhang, Xinyi Zhang, Zhexi Zhang, Sijun Zhang, Wenqiang Zhang, Xiangxiang Zhang, Yongtao Zhang, Yuyu Zhang, Ge Zhang, He Zhang, Yue Zhang, Renjie Zheng, Ningxin Zheng, Zhuolin Zheng, Yaowei Zheng, Chen Zheng, Xiaoyun Zhi, Wanjun Zhong, Cheng Zhong, Zheng Zhong, Baoquan Zhong, Xun Zhou, Na Zhou, Huan Zhou, Hang Zhu, Defa Zhu, Wenjia Zhu, and Lei Zuo. Seed1.5-thinking: Advancing superb reasoning models with reinforcement learning, 2025. URL [https://arxiv.org/abs/2504.13914](https://arxiv.org/abs/2504.13914).

[^38]: Team OpenThoughts. Open Thoughts, January 2025. URL [https://open-thoughts.ai](https://open-thoughts.ai/).

[^39]: Yiqing Xie, Alex Xie, Divyanshu Sheth, Pengfei Liu, Daniel Fried, and Carolyn Rose. Repost: Scalable repository-level coding environment construction with sandbox testing, 2025. URL [https://arxiv.org/abs/2503.07358](https://arxiv.org/abs/2503.07358).

[^40]: Woosuk Kwon, Zhuohan Li, Siyuan Zhuang, Ying Sheng, Lianmin Zheng, Cody Hao Yu, Joseph E. Gonzalez, Hao Zhang, and Ion Stoica. Efficient memory management for large language model serving with pagedattention. In *Proceedings of the ACM SIGOPS 29th Symposium on Operating Systems Principles*, 2023.