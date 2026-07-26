---
title: "A-Mem: Agentic Memory for LLM Agents"
source: "https://ar5iv.labs.arxiv.org/html/2502.12110"
author:
published:
created: 2026-07-26
description: "While large language model (LLM) agents can effectively use external tools for complex real-world tasks, they require memory systems to leverage historical experiences. Current memory systems enable basic storage and r…"
tags:
  - "clippings"
---
Wujiang Xu <sup>1</sup>  Zujie Liang <sup>2</sup>  Kai Mei <sup>1</sup>    
Hang Gao <sup>1</sup>  Juntao Tan <sup>1</sup>  Yongfeng Zhang <sup>1</sup>  
<sup>1</sup> Rutgers University    <sup>2</sup> Ant Group Corresponding Email: yongfeng.zhang@rutgers.edu.

###### Abstract

While large language model (LLM) agents can effectively use external tools for complex real-world tasks, they require memory systems to leverage historical experiences. Current memory systems enable basic storage and retrieval but lack sophisticated memory organization, despite recent attempts to incorporate graph databases. Moreover, these systems’ fixed operations and structures limit their adaptability across diverse tasks. To address this limitation, this paper proposes a novel agentic memory system for LLM agents that can dynamically organize memories in an agentic way. Following the basic principles of the Zettelkasten method, we designed our memory system to create interconnected knowledge networks through dynamic indexing and linking. When a new memory is added, we generate a comprehensive note containing multiple structured attributes, including contextual descriptions, keywords, and tags. The system then analyzes historical memories to identify relevant connections, establishing links where meaningful similarities exist. Additionally, this process enables memory evolution - as new memories are integrated, they can trigger updates to the contextual representations and attributes of existing historical memories, allowing the memory network to continuously refine its understanding. Our approach combines the structured organization principles of Zettelkasten with the flexibility of agent-driven decision making, allowing for more adaptive and context-aware memory management. Empirical experiments on six foundation models show superior improvement against existing SOTA baselines. The source code for evaluating performance is available at [https://github.com/WujiangXu/AgenticMemory](https://github.com/WujiangXu/AgenticMemory), while the source code of agentic memory system is available at [https://github.com/agiresearch/A-mem](https://github.com/agiresearch/A-mem).

A-Mem: Agentic Memory for LLM Agents

Wujiang Xu <sup>1</sup>  Zujie Liang <sup>2</sup>  Kai Mei <sup>1</sup> Hang Gao <sup>1</sup>  Juntao Tan <sup>1</sup>  Yongfeng Zhang <sup>1</sup> <sup>†</sup> <sup>1</sup> Rutgers University    <sup>2</sup> Ant Group

## 1 Introduction

Large Language Model (LLM) agents have demonstrated remarkable capabilities in various tasks, with recent advances enabling them to interact with environments, execute tasks, and make decisions autonomously [^20] [^31] [^5]. They integrate LLMs with external tools and delicate workflows to improve reasoning and planning abilities. Though LLM agent has strong reasoning performance, it still needs a memory system to provide long-term interaction ability with the external environment [^33].

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2502.12110/assets/x1.png)

(a) Traditional memory system.

Existing memory systems [^22] [^43] [^25] [^18] for LLM agents provide basic memory storage functionality. These systems require agent developers to predefine memory storage structures, specify storage points within the workflow, and establish retrieval timing. Meanwhile, to improve structured memory organization, Mem0 [^6], following the principles of RAG [^7] [^15] [^27], incorporates graph databases for storage and retrieval processes. While graph databases provide structured organization for memory systems, their reliance on predefined schemas and relationships fundamentally limits their adaptability. This limitation manifests clearly in practical scenarios - when an agent learns a novel mathematical solution, current systems can only categorize and link this information within their preset framework, unable to forge innovative connections or develop new organizational patterns as knowledge evolves. Such rigid structures, coupled with fixed agent workflows, severely restrict these systems’ ability to generalize across new environments and maintain effectiveness in long-term interactions. The challenge becomes increasingly critical as LLM agents tackle more complex, open-ended tasks, where flexible knowledge organization and continuous adaptation are essential. Therefore, how to design a flexible and universal memory system that supports LLM agents’ long-term interactions remains a crucial challenge.

In this paper, we introduce a novel agentic memory system, named as A-Mem, for LLM agents that enables dynamic memory structuring without relying on static, predetermined memory operations. Our approach draws inspiration from the Zettelkasten method [^13] [^1], a sophisticated knowledge management system that creates interconnected information networks through atomic notes and flexible linking mechanisms. Our system introduces an agentic memory architecture that enables autonomous and flexible memory management for LLM agents. For each new memory, we construct comprehensive notes, which integrates multiple representations: structured textual attributes including several attributes and embedding vectors for similarity matching. Then A-Mem analyzes the historical memory repository to establish meaningful connections based on semantic similarities and shared attributes. This integration process not only creates new links but also enables dynamic evolution when new memories are incorporated, they can trigger updates to the contextual representations of existing memories, allowing the entire memories to continuously refine and deepen its understanding over time. The contributions are summarized as:

$\bullet$ We present A-Mem, an agentic memory system for LLM agents that enables autonomous generation of contextual descriptions, dynamic establishment of memory connections, and intelligent evolution of existing memories based on new experiences. This system equips LLM agents with long-term interaction capabilities without requiring predetermined memory operations.

$\bullet$ We design an agentic memory update mechanism where new memories automatically trigger two key operations: (1) Link Generation - automatically establishing connections between memories by identifying shared attributes and similar contextual descriptions, and (2) Memory Evolution - enabling existing memories to dynamically evolve as new experiences are analyzed, leading to the emergence of higher-order patterns and attributes.

$\bullet$ We conduct comprehensive evaluations of our system using a long-term conversational dataset, comparing performance across six foundation models using six distinct evaluation metrics, demonstrating significant improvements. Moreover, we provide T-SNE visualizations to illustrate the structured organization of our agentic memory system.

## 2 Related Work

### 2.1 Memory for LLM Agents

Prior works on LLM agent memory systems have explored various mechanisms for memory management and utilization [^20] [^18] [^6] [^43]. Some approaches complete interaction storage, which maintains comprehensive historical records through dense retrieval models [^43] or read-write memory structures [^21]. Moreover, MemGPT [^22] leverages cache-like architectures to prioritize recent information. Similarly, SCM [^29] proposes a Self-Controlled Memory framework that enhances LLMs’ capability to maintain long-term memory through a memory stream and controller mechanism. However, these approaches face significant limitations in handling diverse real-world tasks. While they can provide basic memory functionality, their operations are typically constrained by predefined structures and fixed workflows. These constraints stem from their reliance on rigid operational patterns, particularly in memory writing and retrieval processes. Such inflexibility leads to poor generalization in new environments and limited effectiveness in long-term interactions. Therefore, designing a flexible and universal memory system that supports agents’ long-term interactions remains a crucial challenge.

### 2.2 Retrieval-Augmented Generation

Retrieval-Augmented Generation (RAG) has emerged as a powerful approach to enhance LLMs by incorporating external knowledge sources [^15] [^4] [^8]. The standard RAG [^38] [^32] process involves indexing documents into chunks, retrieving relevant chunks based on semantic similarity, and augmenting the LLM’s prompt with this retrieved context for generation. Advanced RAG systems [^17] [^9] have evolved to include sophisticated pre-retrieval and post-retrieval optimizations. Building upon these foundations, recent researches has introduced agentic RAG systems that demonstrate more autonomous and adaptive behaviors in the retrieval process. These systems can dynamically determine when and what to retrieve [^2] [^11], generate hypothetical responses to guide retrieval, and iteratively refine their search strategies based on intermediate results [^28] [^26].

However, while agentic RAG approaches demonstrate agency in the retrieval phase by autonomously deciding when and what to retrieve [^2] [^11] [^39], our agentic memory system exhibits agency at a more fundamental level through the autonomous evolution of its memory structure. Inspired by the Zettelkasten method, our system allows memories to actively generate their own contextual descriptions, form meaningful connections with related memories, and evolve both their content and relationships as new experiences emerge. This fundamental distinction in agency between retrieval versus storage and evolution distinguishes our approach from agentic RAG systems, which maintain static knowledge bases despite their sophisticated retrieval mechanisms.

## 3 Methodolodgy

Our proposed agentic memory system draws inspiration from the Zettelkasten method, implementing a dynamic and self-evolving memory system that enables LLM agents to maintain long-term memory without predetermined operations. The system’s design emphasizes atomic note-taking, flexible linking mechanisms, and continuous evolution of knowledge structures.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2502.12110/assets/x3.png)

Figure 2: Our A-Mem architecture comprises three integral parts in memory storage. During note construction, the system processes new interaction memories and stores them as notes with multiple attributes. The link generation process first retrieves the most relevant historical memories and then decide whether to establish connections between them. The concept of a ’box’ describes that related memories become interconnected through their similar contextual descriptions, analogous to the Zettelkasten method. However, our approach allows individual memories to exist simultaneously within multiple different boxes. In the memory retrieval stage, the system analyzes queries into constituent keywords and utilizes these keywords to search through the memory network.

### 3.1 Note Construction

Building upon the Zettelkasten method’s principles of atomic note-taking and flexible organization, we introduce an LLM-driven approach to memory note construction. When an agent interacts with its environment, we construct structured memory notes that capture both explicit information and LLM-generated contextual understanding. Each memory note $m_{i}$ in our collection $\mathcal{M}=\{m_{1},m_{2},...,m_{N}\}$ is represented as:

$$
m_{i}=\{c_{i},t_{i},K_{i},G_{i},X_{i},e_{i},L_{i}\}
$$

where $c_{i}$ represents the original interaction content, $t_{i}$ is the timestamp of the interaction, $K_{i}$ denotes LLM-generated keywords that capture key concepts, $G_{i}$ contains LLM-generated tags for categorization, $X_{i}$ represents the LLM-generated contextual description that provides rich semantic understanding, and $L_{i}$ maintains the set of linked memories that share semantic relationships. To enrich each memory note with meaningful context beyond its basic content and timestamp, we leverage an LLM to analyze the interaction and generate these semantic components. The note construction process involves prompting the LLM with carefully designed templates $P_{s1}$:

$$
K_{i},G_{i},X_{i}\leftarrow\text{LLM}(c_{i}\;\|t_{i}\;\|P_{s1})
$$

Following the Zettelkasten principle of atomicity, each note captures a single, self-contained unit of knowledge. To enable efficient retrieval and linking, we compute a dense vector representation via a text encoder [^24] that encapsulates all textual components of the note:

$$
e_{i}=f_{\text{enc}}[\;\text{concat}(c_{i},K_{i},G_{i},X_{i})\;]
$$

By using LLMs to generate enriched components, we enable autonomous extraction of implicit knowledge from raw interactions. The multi-faceted note structure ($K_{i}$, $G_{i}$, $X_{i}$) creates rich representations that capture different aspects of the memory, facilitating nuanced organization and retrieval. Additionally, the combination of LLM-generated semantic components with dense vector representations provides both human-interpretable context and computationally efficient similarity matching.

### 3.2 Link Generation

Our system implements an autonomous link generation mechanism that enables new memory notes to form meaningful connections without predefined rules. When the constrctd memory note $m_{n}$ is added to the system, we first leverage its semantic embedding for similarity-based retrieval. For each existing memory note $m_{j}\in\mathcal{M}$, we compute a similarity score:

$$
s_{n,j}=\frac{e_{n}\cdot e_{j}}{|e_{n}||e_{j}|}
$$

The system then identifies the top- $k$ most relevant memories:

$$
\mathcal{M}_{\text{near}}^{n}=\{m_{j}|\;\text{rank}(s_{n,j})\leq k,m_{j}\in\mathcal{M}\}
$$

Based on these candidate nearest memories, we prompt the LLM to analyze potential connections based on their potential common attributes. Formally, the link set of memory $m_{n}$ update like:

$$
L_{i}\leftarrow\text{LLM}(m_{n}\;\|\mathcal{M}_{\text{near}}^{n}\;\|P_{s2})
$$

Each generated link $l_{i}$ is structured as: $L_{i}=\{m_{i},...,m_{k}\}$. By using embedding-based retrieval as an initial filter, we enable efficient scalability while maintaining semantic relevance. A-Mem can quickly identify potential connections even in large memory collections without exhaustive comparison. More importantly, the LLM-driven analysis allows for nuanced understanding of relationships that goes beyond simple similarity metrics. The language model can identify subtle patterns, causal relationships, and conceptual connections that might not be apparent from embedding similarity alone. We implements the Zettelkasten principle of flexible linking while leveraging modern language models. The resulting network emerges organically from memory content and context, enabling natural knowledge organization.

### 3.3 Memory Evolution

After creating links for the new memory, A-Mem evolves the retrieved memories based on their textual information and relationships with the new memory. For each memory $m_{j}$ in the nearest neighbor set $\mathcal{M}_{\text{near}}^{n}$, the system determines whether to update its context, keywords, and tags. This evolution process can be formally expressed as:

$$
m_{j}^{*}\leftarrow\text{LLM}(m_{n}\;\|\mathcal{M}_{\text{near}}^{n}\setminus m_{j}\;\|m_{j}\;\|P_{s3})
$$

The evolved memory $m_{j}^{*}$ then replaces the original memory $m_{j}$ in the memory set $\mathcal{M}$. This evolutionary approach enables continuous updates and new connections, mimicking human learning processes. As the system processes more memories over time, it develops increasingly sophisticated knowledge structures, discovering higher-order patterns and concepts across multiple memories. This creates a foundation for autonomous memory learning where knowledge organization becomes progressively richer through the ongoing interaction between new experiences and existing memories.

### 3.4 Retrieve Relative Memory

In each interaction, our A-Mem performs context-aware memory retrieval to provide the agent with relevant historical information. Given a query text $q$ from the current interaction, we first compute its dense vector representation using the same text encoder used for memory notes:

$$
e_{q}=f_{\text{enc}}(q)
$$

The system then computes similarity scores between the query embedding and all existing memory notes in $\mathcal{M}$ using cosine similarity:

$$
s_{q,i}=\frac{e_{q}\cdot e_{i}}{|e_{q}||e_{i}|},\text{where}\;e_{i}\in m_{i},\;\forall m_{i}\in\mathcal{M}
$$

Then we retrieve the k most relevant memories from the historical memory storage to construct a contextually appropriate prompt.

$$
\mathcal{M}_{\text{retrieved}}=\{m_{i}|\text{rank}(s_{q,i})\leq k,m_{i}\in\mathcal{M}\}
$$

These retrieved memories provide relevant historical context that helps the agent better understand and respond to the current interaction. The retrieved context enriches the agent’s reasoning process by connecting the current interaction with related past experiences and knowledge stored in the memory system.

## 4 Experiment

### 4.1 Dataset and Evaluation

To evaluate the effectiveness of instruction-aware recommendation in long-term conversations, we utilize the LoCoMo dataset [^19], which contains significantly longer dialogues compared to existing conversational datasets [^34] [^10]. While previous datasets contain dialogues with around 1K tokens over 4-5 sessions, LoCoMo features much longer conversations averaging 9K tokens spanning up to 35 sessions, making it particularly suitable for evaluating models’ ability to handle long-range dependencies and maintain consistency over extended conversations. The LoCoMo dataset comprises diverse question types designed to comprehensively evaluate different aspects of model understanding: (1) single-hop questions answerable from a single session; (2) multi-hop questions requiring information synthesis across sessions; (3) temporal reasoning questions testing understanding of time-related information; (4) open-domain knowledge questions requiring integration of conversation context with external knowledge; and (5) adversarial questions assessing models’ ability to identify unanswerable queries. In total, LoCoMo contains 7,512 question-answer pairs across these categories.

For evaluation, we employ two primary metrics: the F1 score to assess answer accuracy by balancing precision and recall, and BLEU-1 [^23] to evaluate generated response quality by measuring word overlap with ground truth responses. Also, we report the average token length for answering one question. Besides, we report the experiment results with four extra metrics including ROUGE-L, ROUGE-2, METEOR and SBERT Similarity in the Appendix B.2.

### 4.2 Implementation Details

For all baselines and our proposed method, we maintain consistency by employing identical system prompts as detailed in Appendix C. The deployment of Qwen-1.5B/3B and Llama 3.2 1B/3B models is accomplished through local instantiation using Ollama <sup>1</sup>, with LiteLLM <sup>2</sup> managing structured output generation. For GPT models, we utilize the official structured output API. In our memory retrieval process, we primarily employ $k$ =10 for top- $k$ memory selection to maintain computational efficiency, while adjusting this parameter for specific categories to optimize performance. The detailed configurations of $k$ can be found in Appendix B.4. For text embedding, we implement the all-minilm-l6-v2 model across all experiments.

Table 1: Experimental results on LoCoMo dataset of QA tasks across five categories (Single Hop, Multi Hop, Temporal, Open Domain, and Adversial) using different methods. Results are reported in F1 and BLEU-1 (%) scores. The best performance is marked in bold, and our proposed method A-MEM (highlighted in gray) demonstrates competitive performance across six foundation language models.

<table><tbody><tr><td colspan="2" rowspan="3">Model</td><td rowspan="3">Method</td><td colspan="10">Category</td><td colspan="3">Average</td></tr><tr><td colspan="2">Single Hop</td><td colspan="2">Multi Hop</td><td colspan="2">Temporal</td><td colspan="2">Open Domain</td><td colspan="2">Adversial</td><td colspan="2">Ranking</td><td>Token</td></tr><tr><td>F1</td><td>BLEU-1</td><td>F1</td><td>BLEU-1</td><td>F1</td><td>BLEU-1</td><td>F1</td><td>BLEU-1</td><td>F1</td><td>BLEU-1</td><td>F1</td><td>BLEU-1</td><td>Length</td></tr><tr><td rowspan="10"><p></p><p>GPT</p><p></p></td><td rowspan="5"><p></p><p>4o-mini</p><p></p></td><td>LoCoMo</td><td>25.02</td><td>19.75</td><td>18.41</td><td>14.77</td><td>12.04</td><td>11.16</td><td>40.36</td><td>29.05</td><td>69.23</td><td>68.75</td><td>2.4</td><td>2.4</td><td>16,910</td></tr><tr><td>ReadAgent</td><td>9.15</td><td>6.48</td><td>12.60</td><td>8.87</td><td>5.31</td><td>5.12</td><td>9.67</td><td>7.66</td><td>9.81</td><td>9.02</td><td>4.2</td><td>4.2</td><td>643</td></tr><tr><td>MemoryBank</td><td>5.00</td><td>4.77</td><td>9.68</td><td>6.99</td><td>5.56</td><td>5.94</td><td>6.61</td><td>5.16</td><td>7.36</td><td>6.48</td><td>4.8</td><td>4.8</td><td>432</td></tr><tr><td>MemGPT</td><td>26.65</td><td>17.72</td><td>25.52</td><td>19.44</td><td>9.15</td><td>7.44</td><td>41.04</td><td>34.34</td><td>43.29</td><td>42.73</td><td>2.4</td><td>2.4</td><td>16,977</td></tr><tr><td>A-Mem</td><td>27.02</td><td>20.09</td><td>45.85</td><td>36.67</td><td>12.14</td><td>12.00</td><td>44.65</td><td>37.06</td><td>50.03</td><td>49.47</td><td>1.2</td><td>1.2</td><td>2,520</td></tr><tr><td rowspan="5"><p></p><p>4o</p><p></p></td><td>LoCoMo</td><td>28.00</td><td>18.47</td><td>9.09</td><td>5.78</td><td>16.47</td><td>14.80</td><td>61.56</td><td>54.19</td><td>52.61</td><td>51.13</td><td>2.0</td><td>2.0</td><td>16,910</td></tr><tr><td>ReadAgent</td><td>14.61</td><td>9.95</td><td>4.16</td><td>3.19</td><td>8.84</td><td>8.37</td><td>12.46</td><td>10.29</td><td>6.81</td><td>6.13</td><td>4.0</td><td>4.0</td><td>805</td></tr><tr><td>MemoryBank</td><td>6.49</td><td>4.69</td><td>2.47</td><td>2.43</td><td>6.43</td><td>5.30</td><td>8.28</td><td>7.10</td><td>4.42</td><td>3.67</td><td>5.0</td><td>5.0</td><td>569</td></tr><tr><td>MemGPT</td><td>30.36</td><td>22.83</td><td>17.29</td><td>13.18</td><td>12.24</td><td>11.87</td><td>60.16</td><td>53.35</td><td>34.96</td><td>34.25</td><td>2.4</td><td>2.4</td><td>16,987</td></tr><tr><td>A-Mem</td><td>32.86</td><td>23.76</td><td>39.41</td><td>31.23</td><td>17.10</td><td>15.84</td><td>48.43</td><td>42.97</td><td>36.35</td><td>35.53</td><td>1.6</td><td>1.6</td><td>1,216</td></tr><tr><td rowspan="10"><p></p><p>Qwen2.5</p><p></p></td><td rowspan="5"><p></p><p>1.5b</p><p></p></td><td>LoCoMo</td><td>9.05</td><td>6.55</td><td>4.25</td><td>4.04</td><td>9.91</td><td>8.50</td><td>11.15</td><td>8.67</td><td>40.38</td><td>40.23</td><td>3.4</td><td>3.4</td><td>16,910</td></tr><tr><td>ReadAgent</td><td>6.61</td><td>4.93</td><td>2.55</td><td>2.51</td><td>5.31</td><td>12.24</td><td>10.13</td><td>7.54</td><td>5.42</td><td>27.32</td><td>4.6</td><td>4.6</td><td>752</td></tr><tr><td>MemoryBank</td><td>11.14</td><td>8.25</td><td>4.46</td><td>2.87</td><td>8.05</td><td>6.21</td><td>13.42</td><td>11.01</td><td>36.76</td><td>34.00</td><td>2.6</td><td>2.6</td><td>284</td></tr><tr><td>MemGPT</td><td>10.44</td><td>7.61</td><td>4.21</td><td>3.89</td><td>13.42</td><td>11.64</td><td>9.56</td><td>7.34</td><td>31.51</td><td>28.90</td><td>3.4</td><td>3.4</td><td>16,953</td></tr><tr><td>A-Mem</td><td>18.23</td><td>11.94</td><td>24.32</td><td>19.74</td><td>16.48</td><td>14.31</td><td>23.63</td><td>19.23</td><td>46.00</td><td>43.26</td><td>1.0</td><td>1.0</td><td>1,300</td></tr><tr><td rowspan="5"><p></p><p>3b</p><p></p></td><td>LoCoMo</td><td>4.61</td><td>4.29</td><td>3.11</td><td>2.71</td><td>4.55</td><td>5.97</td><td>7.03</td><td>5.69</td><td>16.95</td><td>14.81</td><td>3.2</td><td>3.2</td><td>16,910</td></tr><tr><td>ReadAgent</td><td>2.47</td><td>1.78</td><td>3.01</td><td>3.01</td><td>5.57</td><td>5.22</td><td>3.25</td><td>2.51</td><td>15.78</td><td>14.01</td><td>4.2</td><td>4.2</td><td>776</td></tr><tr><td>MemoryBank</td><td>3.60</td><td>3.39</td><td>1.72</td><td>1.97</td><td>6.63</td><td>6.58</td><td>4.11</td><td>3.32</td><td>13.07</td><td>10.30</td><td>4.2</td><td>4.2</td><td>298</td></tr><tr><td>MemGPT</td><td>5.07</td><td>4.31</td><td>2.94</td><td>2.95</td><td>7.04</td><td>7.10</td><td>7.26</td><td>5.52</td><td>14.47</td><td>12.39</td><td>2.4</td><td>2.4</td><td>16,961</td></tr><tr><td>A-Mem</td><td>12.57</td><td>9.01</td><td>27.59</td><td>25.07</td><td>7.12</td><td>7.28</td><td>17.23</td><td>13.12</td><td>27.91</td><td>25.15</td><td>1.0</td><td>1.0</td><td>1,137</td></tr><tr><td rowspan="10"><p></p><p>Llama 3.2</p><p></p></td><td rowspan="5"><p></p><p>1b</p><p></p></td><td>LoCoMo</td><td>11.25</td><td>9.18</td><td>7.38</td><td>6.82</td><td>11.90</td><td>10.38</td><td>12.86</td><td>10.50</td><td>51.89</td><td>48.27</td><td>3.4</td><td>3.4</td><td>16,910</td></tr><tr><td>ReadAgent</td><td>5.96</td><td>5.12</td><td>1.93</td><td>2.30</td><td>12.46</td><td>11.17</td><td>7.75</td><td>6.03</td><td>44.64</td><td>40.15</td><td>4.6</td><td>4.6</td><td>665</td></tr><tr><td>MemoryBank</td><td>13.18</td><td>10.03</td><td>7.61</td><td>6.27</td><td>15.78</td><td>12.94</td><td>17.30</td><td>14.03</td><td>52.61</td><td>47.53</td><td>2.0</td><td>2.0</td><td>274</td></tr><tr><td>MemGPT</td><td>9.19</td><td>6.96</td><td>4.02</td><td>4.79</td><td>11.14</td><td>8.24</td><td>10.16</td><td>7.68</td><td>49.75</td><td>45.11</td><td>4.0</td><td>4.0</td><td>16,950</td></tr><tr><td>A-Mem</td><td>19.06</td><td>11.71</td><td>17.80</td><td>10.28</td><td>17.55</td><td>14.67</td><td>28.51</td><td>24.13</td><td>58.81</td><td>54.28</td><td>1.0</td><td>1.0</td><td>1,376</td></tr><tr><td rowspan="5"><p></p><p>3b</p><p></p></td><td>LoCoMo</td><td>6.88</td><td>5.77</td><td>4.37</td><td>4.40</td><td>10.65</td><td>9.29</td><td>8.37</td><td>6.93</td><td>30.25</td><td>28.46</td><td>2.8</td><td>2.8</td><td>16,910</td></tr><tr><td>ReadAgent</td><td>2.47</td><td>1.78</td><td>3.01</td><td>3.01</td><td>5.57</td><td>5.22</td><td>3.25</td><td>2.51</td><td>15.78</td><td>14.01</td><td>4.2</td><td>4.2</td><td>461</td></tr><tr><td>MemoryBank</td><td>6.19</td><td>4.47</td><td>3.49</td><td>3.13</td><td>4.07</td><td>4.57</td><td>7.61</td><td>6.03</td><td>18.65</td><td>17.05</td><td>3.2</td><td>3.2</td><td>263</td></tr><tr><td>MemGPT</td><td>5.32</td><td>3.99</td><td>2.68</td><td>2.72</td><td>5.64</td><td>5.54</td><td>4.32</td><td>3.51</td><td>21.45</td><td>19.37</td><td>3.8</td><td>3.8</td><td>16,956</td></tr><tr><td>A-Mem</td><td>17.44</td><td>11.74</td><td>26.38</td><td>19.50</td><td>12.53</td><td>11.83</td><td>28.14</td><td>23.87</td><td>42.04</td><td>40.60</td><td>1.0</td><td>1.0</td><td>1,126</td></tr></tbody></table>

### 4.3 Baselines

LoCoMo [^19] takes a direct approach by leveraging foundation models without memory mechanisms for question answering tasks. For each query, it incorporates the complete preceding conversation and questions into the prompt, evaluating the model’s reasoning capabilities.

ReadAgent [^14] tackles long-context document processing through a sophisticated three-step methodology: it begins with episode pagination to segment content into manageable chunks, followed by memory gisting to distill each page into concise memory representations, and concludes with interactive look-up to retrieve pertinent information as needed.

MemoryBank [^43] introduces an innovative memory management system that maintains and efficiently retrieves historical interactions. The system features a dynamic memory updating mechanism based on the Ebbinghaus Forgetting Curve theory, which intelligently adjusts memory strength according to time and significance. Additionally, it incorporates a user portrait building system that progressively refines its understanding of user personality through continuous interaction analysis.

MemGPT [^22] presents a novel virtual context management system drawing inspiration from traditional operating systems’ memory hierarchies. The architecture implements a dual-tier structure: a main context (analogous to RAM) that provides immediate access during LLM inference, and an external context (analogous to disk storage) that maintains information beyond the fixed context window.

### 4.4 Empricial Results

Table 2: An ablation study was conducted to evaluate our proposed method against the GPT-4-mini base model. The notation ’w/o’ indicates experiments where specific modules were removed. The abbreviations LG and ME denote the link generation module and memory evolution module, respectively.

<table><tbody><tr><th rowspan="3">Method</th><td colspan="10">Category</td></tr><tr><td colspan="2">Single Hop</td><td colspan="2">Multi Hop</td><td colspan="2">Temporal</td><td colspan="2">Open Domain</td><td colspan="2">Adversial</td></tr><tr><td>F1</td><td>BLEU-1</td><td>F1</td><td>BLEU-1</td><td>F1</td><td>BLEU-1</td><td>F1</td><td>BLEU-1</td><td>F1</td><td>BLEU-1</td></tr><tr><th>w/o LG & ME</th><td>9.65</td><td>7.09</td><td>24.55</td><td>19.48</td><td>7.77</td><td>6.70</td><td>13.28</td><td>10.30</td><td>15.32</td><td>18.02</td></tr><tr><th>w/o ME</th><td>21.35</td><td>15.13</td><td>31.24</td><td>27.31</td><td>10.13</td><td>10.85</td><td>39.17</td><td>34.70</td><td>44.16</td><td>45.33</td></tr><tr><th>A-Mem</th><td>27.02</td><td>20.09</td><td>45.85</td><td>36.67</td><td>12.14</td><td>12.00</td><td>44.65</td><td>37.06</td><td>50.03</td><td>49.47</td></tr></tbody></table>

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2502.12110/assets/x4.png)

(a) Single Hop

In our empirical evaluation, we compared A-MEM with four competitive baselines including LoCoMo, ReadAgent, MemoryBank, and MemGPT on the LoCoMo dataset. For non-GPT foundation models, our A-Mem consistently outperforms all baselines across different categories, demonstrating the effectiveness of our agentic memory approach. For GPT-based models, while LoCoMo and MemGPT show strong performance in certain categories like Open Domain and Adversial tasks due to their robust pre-trained knowledge in simple fact retrieval, our A-MEM demonstrates superior performance in Multi-Hop tasks achieves at least two times better performance that require complex reasoning chains. The effectiveness of A-Mem stems from its novel agentic memory architecture that enables dynamic and structured memory management. Unlike traditional approaches that use static memory operations, our system creates interconnected memory networks through atomic notes with rich contextual descriptions, enabling more effective multi-hop reasoning. The system’s ability to dynamically establish connections between memories based on shared attributes and continuously update existing memory descriptions with new contextual information allows it to better capture and utilize the relationships between different pieces of information. Notably, A-Mem achieves these improvements while maintaining significantly lower token length requirements compared to LoCoMo and MemGPT (around 1,200-2,500 tokens versus 16,900 tokens) through our selective top-k retrieval mechanism. In conclusion, our empirical results demonstrate that A-Mem successfully combines structured memory organization with dynamic memory evolution, leading to superior performance in complex reasoning tasks while maintaining computational efficiency.

### 4.5 Ablation Study

To evaluate the effectiveness of the Link Generation (LG) and Memory Evolution (ME) modules, we conduct the ablation study by systematically removing key components of our model. When both LG and ME modules are removed, the system exhibits substantial performance degradation, particularly in Multi Hop reasoning and Open Domain tasks. The system with only LG active (w/o ME) shows intermediate performance levels, maintaining significantly better results than the version without both modules, which demonstrates the fundamental importance of link generation in establishing memory connections. Our full model, A-MEM, consistently achieves the best performance across all evaluation categories, with particularly strong results in complex reasoning tasks. These results reveal that while the link generation module serves as a critical foundation for memory organization, the memory evolution module provides essential refinements to the memory structure. The ablation study validates our architectural design choices and highlights the complementary nature of these two modules in creating an effective memory system.

### 4.6 Hyperparameter Analysis

We conducted extensive experiments to analyze the impact of the memory retrieval parameter k, which controls the number of relevant memories retrieved for each interaction. As shown in Figure 3, we evaluated performance across different k values (10, 20, 30, 40, 50) on five categories of tasks using GPT-4-mini as our base model. The results reveal an interesting pattern: while increasing k generally leads to improved performance, this improvement gradually plateaus and sometimes slightly decreases at higher values. This trend is particularly evident in Multi Hop and Open Domain tasks. The observation suggests a delicate balance in memory retrieval - while larger k values provide richer historical context for reasoning, they may also introduce noise and challenge the model’s capacity to process longer sequences effectively. Our analysis indicates that moderate k values strike an optimal balance between context richness and information processing efficiency.

### 4.7 Memory Analysis

We present the t-SNE visualization in Figure 4 of memory embeddings to demonstrate the structural advantages of our agentic memory system. Analyzing two dialogues sampled from long-term conversations in LoCoMo [^19], we observe that A-Mem (shown in blue) consistently exhibits more coherent clustering patterns compared to the baseline system (shown in red). This structural organization is particularly evident in Dialogue 2, where well-defined clusters emerge in the central region, providing empirical evidence for the effectiveness of our memory evolution mechanism and contextual description generation. In contrast, the baseline memory embeddings display a more dispersed distribution, demonstrating that memories lack structural organization without our link generation and memory evolution components. These visualization results validate that A-Mem can autonomously maintain meaningful memory structures through dynamic evolution and linking mechanisms. More results can be seen in Appendix B.3.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2502.12110/assets/x9.png)

(a) Dialogue 1

## 5 Conclusion

In this work, we introduced A-Mem, a novel agentic memory system that enables LLM agents to dynamically organize and evolve their memories without relying on predefined structures. Drawing inspiration from the Zettelkasten method, our system creates an interconnected knowledge network through dynamic indexing and linking mechanisms that adapt to diverse real-world tasks. The system’s core architecture features autonomous generation of contextual descriptions for new memories and intelligent establishment of connections with existing memories based on shared attributes. Furthermore, our approach enables continuous evolution of historical memories by incorporating new experiences and developing higher-order attributes through ongoing interactions. Through extensive empirical evaluation across six foundation models, we demonstrated that A-Mem achieves superior performance compared to existing state-of-the-art baselines in long-term conversational tasks. Visualization analysis further validates the effectiveness of our memory organization approach. These results suggest that agentic memory systems can significantly enhance LLM agents’ ability to utilize long-term knowledge in complex environments.

## 6 Limitation

While our agentic memory system achieves promising results, we acknowledge several areas for potential future exploration. First, although our system dynamically organizes memories, the quality of these organizations may still be influenced by the inherent capabilities of the underlying language models. Different LLMs might generate slightly different contextual descriptions or establish varying connections between memories. Additionally, while our current implementation focuses on text-based interactions, future work could explore extending the system to handle multimodal information, such as images or audio, which could provide richer contextual representations.

## References

## APPENDIX

## Appendix A Detailed Related Work

### A.1 Memory for LLM Agents

Large Language Models (LLMs) have demonstrated remarkable capabilities across various domains, including natural language processing, code generation, and recommender systems [^30] [^40] [^36] [^37] [^35]. LLM-based agents further extend these capabilities by enabling interactive decision-making and executing complex workflows through structured interaction patterns [^12] [^41] [^42]. Prior works on LLM agent memory systems have explored various mechanisms for memory management and utilization [^20] [^18] [^6] [^43]. Some approaches complete interaction storage, which maintains comprehensive historical records through dense retrieval models [^43] or read-write memory structures [^21]. Moreover, MemGPT [^22] leverages cache-like architectures to prioritize recent information. Similarly, SCM [^29] proposes a Self-Controlled Memory framework that enhances LLMs’ capability to maintain long-term memory through a memory stream and controller mechanism. However, these approaches face significant limitations in handling diverse real-world tasks. While they can provide basic memory functionality, their operations are typically constrained by predefined structures and fixed workflows. These constraints stem from their reliance on rigid operational patterns, particularly in memory writing and retrieval processes. Such inflexibility leads to poor generalization in new environments and limited effectiveness in long-term interactions. Therefore, designing a flexible and universal memory system that supports agents’ long-term interactions remains a crucial challenge.

## Appendix B Experiment

### B.1 Evaluation Metric

The F1 score represents the harmonic mean of precision and recall, offering a balanced metric that combines both measures into a single value. This metric is particularly valuable when we need to balance between complete and accurate responses:

$$
F1=2\cdot\frac{\text{precision}\cdot\text{recall}}{\text{precision}+\text{recall}}
$$

where

$$
\text{precision}=\frac{\text{true positives}}{\text{true positives}+\text{false positives}}
$$
 
$$
\text{recall}=\frac{\text{true positives}}{\text{true positives}+\text{false negatives}}
$$

In question-answering systems, the F1 score serves a crucial role in evaluating exact matches between predicted and reference answers. This is especially important for span-based QA tasks, where systems must identify precise text segments while maintaining comprehensive coverage of the answer.

BLEU-1 [^23] provides a method for evaluating the precision of unigram matches between system outputs and reference texts:

$$
\text{BLEU-1}=BP\cdot\exp(\sum_{n=1}^{1}w_{n}\log p_{n})
$$

where

$$
BP=\begin{cases}1&\text{if }c>r\\
e^{1-r/c}&\text{if }c\leq r\end{cases}
$$
 
$$
p_{n}=\frac{\sum_{i}\sum_{k}\min(h_{ik},m_{ik})}{\sum_{i}\sum_{k}h_{ik}}
$$

Here, $c$ is candidate length, $r$ is reference length, $h_{ik}$ is the count of n-gram i in candidate k, and $m_{ik}$ is the maximum count in any reference. In QA, BLEU-1 evaluates the lexical precision of generated answers, particularly useful for generative QA systems where exact matching might be too strict.

ROUGE-L [^16] measures the longest common subsequence between the generated and reference texts.

$$
\text{ROUGE-L}=\frac{(1+\beta^{2})R_{l}P_{l}}{R_{l}+\beta^{2}P_{l}}
$$
 
$$
R_{l}=\frac{\text{LCS}(X,Y)}{|X|}
$$
 
$$
P_{l}=\frac{\text{LCS}(X,Y)}{|Y|}
$$

where $X$ is reference text, $Y$ is candidate text, and LCS is the Longest Common Subsequence.

ROUGE-2 [^16] calculates the overlap of bigrams between the generated and reference texts.

$$
\text{ROUGE-2}=\frac{\sum_{\text{bigram}\in\text{ref}}\min(\text{Count}_{\text{ref}}(\text{bigram}),\text{Count}_{\text{cand}}(\text{bigram}))}{\sum_{\text{bigram}\in\text{ref}}\text{Count}_{\text{ref}}(\text{bigram})}
$$

Both ROUGE-L and ROUGE-2 are particularly useful for evaluating the fluency and coherence of generated answers, with ROUGE-L focusing on sequence matching and ROUGE-2 on local word order.

Table 3: Experimental results on LoCoMo dataset of QA tasks across five categories (Single Hop, Multi Hop, Temporal, Open Domain, and Adversial) using different methods. Results are reported in ROUGE-2 and ROUGE-L scores, abbreviated to RGE-2 and RGE-L. The best performance is marked in bold, and our proposed method A-MEM (highlighted in gray) demonstrates competitive performance across six foundation language models.

<table><tbody><tr><td colspan="2" rowspan="3">Model</td><td rowspan="3">Method</td><td colspan="10">Category</td></tr><tr><td colspan="2">Single Hop</td><td colspan="2">Multi Hop</td><td colspan="2">Temporal</td><td colspan="2">Open Domain</td><td colspan="2">Adversial</td></tr><tr><td>RGE-2</td><td>RGE-L</td><td>RGE-2</td><td>RGE-L</td><td>RGE-2</td><td>RGE-L</td><td>RGE-2</td><td>RGE-L</td><td>RGE-2</td><td>RGE-L</td></tr><tr><td rowspan="10"><p></p><p>GPT</p><p></p></td><td rowspan="5"><p></p><p>4o-mini</p><p></p></td><td>LoCoMo</td><td>9.64</td><td>23.92</td><td>2.01</td><td>18.09</td><td>3.40</td><td>11.58</td><td>26.48</td><td>40.20</td><td>60.46</td><td>69.59</td></tr><tr><td>ReadAgent</td><td>2.47</td><td>9.45</td><td>0.95</td><td>13.12</td><td>0.55</td><td>5.76</td><td>2.99</td><td>9.92</td><td>6.66</td><td>9.79</td></tr><tr><td>MemoryBank</td><td>1.18</td><td>5.43</td><td>0.52</td><td>9.64</td><td>0.97</td><td>5.77</td><td>1.64</td><td>6.63</td><td>4.55</td><td>7.35</td></tr><tr><td>MemGPT</td><td>10.58</td><td>25.60</td><td>4.76</td><td>25.22</td><td>0.76</td><td>9.14</td><td>28.44</td><td>42.24</td><td>36.62</td><td>43.75</td></tr><tr><td>A-Mem</td><td>10.61</td><td>25.86</td><td>21.39</td><td>44.27</td><td>3.42</td><td>12.09</td><td>29.50</td><td>45.18</td><td>42.62</td><td>50.04</td></tr><tr><td rowspan="5"><p></p><p>4o</p><p></p></td><td>LoCoMo</td><td>11.53</td><td>30.65</td><td>1.68</td><td>8.17</td><td>3.21</td><td>16.33</td><td>45.42</td><td>63.86</td><td>45.13</td><td>52.67</td></tr><tr><td>ReadAgent</td><td>3.91</td><td>14.36</td><td>0.43</td><td>3.96</td><td>0.52</td><td>8.58</td><td>4.75</td><td>13.41</td><td>4.24</td><td>6.81</td></tr><tr><td>MemoryBank</td><td>1.84</td><td>7.36</td><td>0.36</td><td>2.29</td><td>2.13</td><td>6.85</td><td>3.02</td><td>9.35</td><td>1.22</td><td>4.41</td></tr><tr><td>MemGPT</td><td>11.55</td><td>30.18</td><td>4.66</td><td>15.83</td><td>3.27</td><td>14.02</td><td>43.27</td><td>62.75</td><td>28.72</td><td>35.08</td></tr><tr><td>A-Mem</td><td>12.76</td><td>31.71</td><td>9.82</td><td>25.04</td><td>6.09</td><td>16.63</td><td>33.67</td><td>50.31</td><td>30.31</td><td>36.34</td></tr><tr><td rowspan="10"><p></p><p>Qwen2.5</p><p></p></td><td rowspan="5"><p></p><p>1.5b</p><p></p></td><td>LoCoMo</td><td>1.39</td><td>9.24</td><td>0.00</td><td>4.68</td><td>3.42</td><td>10.59</td><td>3.25</td><td>11.15</td><td>35.10</td><td>43.61</td></tr><tr><td>ReadAgent</td><td>0.74</td><td>7.14</td><td>0.10</td><td>2.81</td><td>3.05</td><td>12.63</td><td>1.47</td><td>7.88</td><td>20.73</td><td>27.82</td></tr><tr><td>MemoryBank</td><td>1.51</td><td>11.18</td><td>0.14</td><td>5.39</td><td>1.80</td><td>8.44</td><td>5.07</td><td>13.72</td><td>29.24</td><td>36.95</td></tr><tr><td>MemGPT</td><td>1.16</td><td>11.35</td><td>0.00</td><td>7.88</td><td>2.87</td><td>14.62</td><td>2.18</td><td>9.82</td><td>23.96</td><td>31.69</td></tr><tr><td>A-Mem</td><td>4.88</td><td>17.94</td><td>5.88</td><td>27.23</td><td>3.44</td><td>16.87</td><td>12.32</td><td>24.38</td><td>36.32</td><td>46.60</td></tr><tr><td rowspan="5"><p></p><p>3b</p><p></p></td><td>LoCoMo</td><td>0.49</td><td>4.83</td><td>0.14</td><td>3.20</td><td>1.31</td><td>5.38</td><td>1.97</td><td>6.98</td><td>12.66</td><td>17.10</td></tr><tr><td>ReadAgent</td><td>0.08</td><td>4.08</td><td>0.00</td><td>1.96</td><td>1.26</td><td>6.19</td><td>0.73</td><td>4.34</td><td>7.35</td><td>10.64</td></tr><tr><td>MemoryBank</td><td>0.43</td><td>3.76</td><td>0.05</td><td>1.61</td><td>0.24</td><td>6.32</td><td>1.03</td><td>4.22</td><td>9.55</td><td>13.41</td></tr><tr><td>MemGPT</td><td>0.69</td><td>5.55</td><td>0.05</td><td>3.17</td><td>1.90</td><td>7.90</td><td>2.05</td><td>7.32</td><td>10.46</td><td>14.39</td></tr><tr><td>A-Mem</td><td>2.91</td><td>12.42</td><td>8.11</td><td>27.74</td><td>1.51</td><td>7.51</td><td>8.80</td><td>17.57</td><td>21.39</td><td>27.98</td></tr><tr><td rowspan="10"><p></p><p>Llama 3.2</p><p></p></td><td rowspan="5"><p></p><p>1b</p><p></p></td><td>LoCoMo</td><td>2.51</td><td>11.48</td><td>0.44</td><td>8.25</td><td>1.69</td><td>13.06</td><td>2.94</td><td>13.00</td><td>39.85</td><td>52.74</td></tr><tr><td>ReadAgent</td><td>0.53</td><td>6.49</td><td>0.00</td><td>4.62</td><td>5.47</td><td>14.29</td><td>1.19</td><td>8.03</td><td>34.52</td><td>45.55</td></tr><tr><td>MemoryBank</td><td>2.96</td><td>13.57</td><td>0.23</td><td>10.53</td><td>4.01</td><td>18.38</td><td>6.41</td><td>17.66</td><td>41.15</td><td>53.31</td></tr><tr><td>MemGPT</td><td>1.82</td><td>9.91</td><td>0.06</td><td>6.56</td><td>2.13</td><td>11.36</td><td>2.00</td><td>10.37</td><td>38.59</td><td>50.31</td></tr><tr><td>A-Mem</td><td>4.82</td><td>19.31</td><td>1.84</td><td>20.47</td><td>5.99</td><td>18.49</td><td>14.82</td><td>29.78</td><td>46.76</td><td>60.23</td></tr><tr><td rowspan="5"><p></p><p>3b</p><p></p></td><td>LoCoMo</td><td>0.98</td><td>7.22</td><td>0.03</td><td>4.45</td><td>2.36</td><td>11.39</td><td>2.85</td><td>8.45</td><td>25.47</td><td>30.26</td></tr><tr><td>ReadAgent</td><td>2.47</td><td>1.78</td><td>3.01</td><td>3.01</td><td>5.07</td><td>5.22</td><td>3.25</td><td>2.51</td><td>15.78</td><td>14.01</td></tr><tr><td>MemoryBank</td><td>1.83</td><td>6.96</td><td>0.25</td><td>3.41</td><td>0.43</td><td>4.43</td><td>2.73</td><td>7.83</td><td>14.64</td><td>18.59</td></tr><tr><td>MemGPT</td><td>0.72</td><td>5.39</td><td>0.11</td><td>2.85</td><td>0.61</td><td>5.74</td><td>1.45</td><td>4.42</td><td>16.62</td><td>21.47</td></tr><tr><td>A-Mem</td><td>6.02</td><td>17.62</td><td>7.93</td><td>27.97</td><td>5.38</td><td>13.00</td><td>16.89</td><td>28.55</td><td>35.48</td><td>42.25</td></tr></tbody></table>

Table 4: Experimental results on LoCoMo dataset of QA tasks across five categories (Single Hop, Multi Hop, Temporal, Open Domain, and Adversial) using different methods. Results are reported in METEOR and SBERT Similarity scores, abbreviated to ME and SBERT. The best performance is marked in bold, and our proposed method A-MEM (highlighted in gray) demonstrates competitive performance across six foundation language models.

<table><tbody><tr><td colspan="2" rowspan="3">Model</td><td rowspan="3">Method</td><td colspan="10">Category</td></tr><tr><td colspan="2">Single Hop</td><td colspan="2">Multi Hop</td><td colspan="2">Temporal</td><td colspan="2">Open Domain</td><td colspan="2">Adversial</td></tr><tr><td>ME</td><td>SBERT</td><td>ME</td><td>SBERT</td><td>ME</td><td>SBERT</td><td>ME</td><td>SBERT</td><td>ME</td><td>SBERT</td></tr><tr><td rowspan="10"><p></p><p>GPT</p><p></p></td><td rowspan="5"><p></p><p>4o-mini</p><p></p></td><td>LoCoMo</td><td>15.81</td><td>47.97</td><td>7.61</td><td>52.30</td><td>8.16</td><td>35.00</td><td>40.42</td><td>57.78</td><td>63.28</td><td>71.93</td></tr><tr><td>ReadAgent</td><td>5.46</td><td>28.67</td><td>4.76</td><td>45.07</td><td>3.69</td><td>26.72</td><td>8.01</td><td>26.78</td><td>8.38</td><td>15.20</td></tr><tr><td>MemoryBank</td><td>3.42</td><td>21.71</td><td>4.07</td><td>37.58</td><td>4.21</td><td>23.71</td><td>5.81</td><td>20.76</td><td>6.24</td><td>13.00</td></tr><tr><td>MemGPT</td><td>15.79</td><td>49.33</td><td>13.25</td><td>61.53</td><td>4.59</td><td>32.77</td><td>41.40</td><td>58.19</td><td>39.16</td><td>47.24</td></tr><tr><td>A-Mem</td><td>16.36</td><td>49.46</td><td>23.43</td><td>70.49</td><td>8.36</td><td>38.48</td><td>42.32</td><td>59.38</td><td>45.64</td><td>53.26</td></tr><tr><td rowspan="5"><p></p><p>4o</p><p></p></td><td>LoCoMo</td><td>16.34</td><td>53.82</td><td>7.21</td><td>32.15</td><td>8.98</td><td>43.72</td><td>53.39</td><td>73.40</td><td>47.72</td><td>56.09</td></tr><tr><td>ReadAgent</td><td>7.86</td><td>37.41</td><td>3.76</td><td>26.22</td><td>4.42</td><td>30.75</td><td>9.36</td><td>31.37</td><td>5.47</td><td>12.34</td></tr><tr><td>MemoryBank</td><td>3.22</td><td>26.23</td><td>2.29</td><td>23.49</td><td>4.18</td><td>24.89</td><td>6.64</td><td>23.90</td><td>2.93</td><td>10.01</td></tr><tr><td>MemGPT</td><td>16.64</td><td>55.12</td><td>12.68</td><td>35.93</td><td>7.78</td><td>37.91</td><td>52.14</td><td>72.83</td><td>31.15</td><td>39.08</td></tr><tr><td>A-Mem</td><td>17.53</td><td>55.96</td><td>13.10</td><td>45.40</td><td>10.62</td><td>38.87</td><td>41.93</td><td>62.47</td><td>32.34</td><td>40.11</td></tr><tr><td rowspan="10"><p></p><p>Qwen2.5</p><p></p></td><td rowspan="5"><p></p><p>1.5b</p><p></p></td><td>LoCoMo</td><td>4.99</td><td>32.23</td><td>2.86</td><td>34.03</td><td>5.89</td><td>35.61</td><td>8.57</td><td>29.47</td><td>40.53</td><td>50.49</td></tr><tr><td>ReadAgent</td><td>3.67</td><td>28.20</td><td>1.88</td><td>27.27</td><td>8.97</td><td>35.13</td><td>5.52</td><td>26.33</td><td>24.04</td><td>34.12</td></tr><tr><td>MemoryBank</td><td>5.57</td><td>35.40</td><td>2.80</td><td>32.47</td><td>4.27</td><td>33.85</td><td>10.59</td><td>32.16</td><td>32.93</td><td>42.83</td></tr><tr><td>MemGPT</td><td>5.40</td><td>35.64</td><td>2.35</td><td>39.04</td><td>7.68</td><td>40.36</td><td>7.07</td><td>30.16</td><td>27.24</td><td>40.63</td></tr><tr><td>A-Mem</td><td>9.49</td><td>43.49</td><td>11.92</td><td>61.65</td><td>9.11</td><td>42.58</td><td>19.69</td><td>41.93</td><td>40.64</td><td>52.44</td></tr><tr><td rowspan="5"><p></p><p>3b</p><p></p></td><td>LoCoMo</td><td>2.00</td><td>24.37</td><td>1.92</td><td>25.24</td><td>3.45</td><td>25.38</td><td>6.00</td><td>21.28</td><td>16.67</td><td>23.14</td></tr><tr><td>ReadAgent</td><td>1.78</td><td>21.10</td><td>1.69</td><td>20.78</td><td>4.43</td><td>25.15</td><td>3.37</td><td>18.20</td><td>10.46</td><td>17.39</td></tr><tr><td>MemoryBank</td><td>2.37</td><td>17.81</td><td>2.22</td><td>21.93</td><td>3.86</td><td>20.65</td><td>3.99</td><td>16.26</td><td>15.49</td><td>20.77</td></tr><tr><td>MemGPT</td><td>3.74</td><td>24.31</td><td>2.25</td><td>27.67</td><td>6.44</td><td>29.59</td><td>6.24</td><td>22.40</td><td>13.19</td><td>20.83</td></tr><tr><td>A-Mem</td><td>6.25</td><td>33.72</td><td>14.04</td><td>62.54</td><td>6.56</td><td>30.60</td><td>15.98</td><td>33.98</td><td>27.36</td><td>33.72</td></tr><tr><td rowspan="10"><p></p><p>Llama 3.2</p><p></p></td><td rowspan="5"><p></p><p>1b</p><p></p></td><td>LoCoMo</td><td>5.77</td><td>38.02</td><td>3.38</td><td>45.44</td><td>6.20</td><td>42.69</td><td>9.33</td><td>34.19</td><td>46.79</td><td>60.74</td></tr><tr><td>ReadAgent</td><td>2.97</td><td>29.26</td><td>1.31</td><td>26.45</td><td>7.13</td><td>39.19</td><td>5.36</td><td>26.44</td><td>42.39</td><td>54.35</td></tr><tr><td>MemoryBank</td><td>6.77</td><td>39.33</td><td>4.43</td><td>45.63</td><td>7.76</td><td>42.81</td><td>13.01</td><td>37.32</td><td>50.43</td><td>60.81</td></tr><tr><td>MemGPT</td><td>5.10</td><td>32.99</td><td>2.54</td><td>41.81</td><td>3.26</td><td>35.99</td><td>6.62</td><td>30.68</td><td>45.00</td><td>61.33</td></tr><tr><td>A-Mem</td><td>9.01</td><td>45.16</td><td>7.50</td><td>54.79</td><td>8.30</td><td>43.42</td><td>22.46</td><td>47.07</td><td>53.72</td><td>68.00</td></tr><tr><td rowspan="5"><p></p><p>3b</p><p></p></td><td>LoCoMo</td><td>3.69</td><td>27.94</td><td>2.96</td><td>20.40</td><td>6.46</td><td>32.17</td><td>6.58</td><td>22.92</td><td>29.02</td><td>35.74</td></tr><tr><td>ReadAgent</td><td>1.21</td><td>17.40</td><td>2.33</td><td>12.02</td><td>3.39</td><td>19.63</td><td>2.46</td><td>14.63</td><td>14.37</td><td>21.25</td></tr><tr><td>MemoryBank</td><td>3.84</td><td>25.06</td><td>2.73</td><td>13.65</td><td>3.05</td><td>21.08</td><td>6.35</td><td>22.02</td><td>17.14</td><td>24.39</td></tr><tr><td>MemGPT</td><td>2.78</td><td>22.06</td><td>2.21</td><td>14.97</td><td>3.63</td><td>23.18</td><td>3.47</td><td>17.81</td><td>20.50</td><td>26.87</td></tr><tr><td>A-Mem</td><td>9.74</td><td>39.32</td><td>13.19</td><td>59.70</td><td>8.09</td><td>32.27</td><td>24.30</td><td>42.86</td><td>39.74</td><td>46.76</td></tr></tbody></table>

METEOR [^3] computes a score based on aligned unigrams between the candidate and reference texts, considering synonyms and paraphrases.

$$
\text{METEOR}=F_{\text{mean}}\cdot(1-\text{Penalty})
$$
 
$$
F_{\text{mean}}=\frac{10P\cdot R}{R+9P}
$$
 
$$
\text{Penalty}=0.5\cdot(\frac{\text{ch}}{m})^{3}
$$

where $P$ is precision, $R$ is recall, ch is number of chunks, and $m$ is number of matched unigrams. METEOR is valuable for QA evaluation as it considers semantic similarity beyond exact matching, making it suitable for evaluating paraphrased answers.

SBERT Similarity [^24] measures the semantic similarity between two texts using sentence embeddings.

$$
\text{SBERT\_Similarity}=\cos(\text{SBERT}(x),\text{SBERT}(y))
$$
 
$$
\cos(a,b)=\frac{a\cdot b}{\|a\|\|b\|}
$$

SBERT($x$ ) represents the sentence embedding of text. SBERT Similarity is particularly useful for evaluating semantic understanding in QA systems, as it can capture meaning similarities even when the lexical overlap is low.

### B.2 Comparison Results

Our comprehensive evaluation using ROUGE-2, ROUGE-L, METEOR, and SBERT metrics demonstrates that A-Mem achieves superior performance while maintaining remarkable computational efficiency. Through extensive empirical testing across various model sizes and task categories, we have established A-Mem as a more effective approach compared to existing baselines, supported by several compelling findings. In our analysis of non-GPT models, specifically Qwen2.5 and Llama 3.2, A-Mem consistently outperforms all baseline approaches across all metrics. The Multi-Hop category showcases particularly striking results, where Qwen2.5-15b with A-Mem achieves a ROUGE-L score of 27.23, dramatically surpassing LoComo’s 4.68 and ReadAgent’s 2.81 - representing a nearly six-fold improvement. This pattern of superiority extends consistently across METEOR and SBERT scores. When examining GPT-based models, our results reveal an interesting pattern. While LoComo and MemGPT demonstrate strong capabilities in Open Domain and Adversarial tasks, A-Mem shows remarkable superiority in Multi-Hop reasoning tasks. Using GPT-4o-mini, A-Mem achieves a ROUGE-L score of 44.27 in Multi-Hop tasks, more than doubling LoComo’s 18.09. This significant advantage maintains consistency across other metrics, with METEOR scores of 23.43 versus 7.61 and SBERT scores of 70.49 versus 52.30. The significance of these results is amplified by A-Mem’s exceptional computational efficiency. Our approach requires only 1,200-2,500 tokens, compared to the substantial 16,900 tokens needed by LoComo and MemGPT. This efficiency stems from two key architectural innovations: First, our novel agentic memory architecture creates interconnected memory networks through atomic notes with rich contextual descriptions, enabling more effective capture and utilization of information relationships. Second, our selective top-k retrieval mechanism facilitates dynamic memory evolution and structured organization. The effectiveness of these innovations is particularly evident in complex reasoning tasks, as demonstrated by the consistently strong Multi-Hop performance across all evaluation metrics.

### B.3 Memory Analysis

In addition to the memory visualizations of the first two dialogues shown in the main text, we present additional visualizations in Fig.5 that demonstrate the structural advantages of our agentic memory system. Through analysis of two dialogues sampled from long-term conversations in LoCoMo [^19], we observe that A-Mem (shown in blue) consistently produces more coherent clustering patterns compared to the baseline system (shown in red). This structural organization is particularly evident in Dialogue 2, where distinct clusters emerge in the central region, providing empirical support for the effectiveness of our memory evolution mechanism and contextual description generation. In contrast, the baseline memory embeddings exhibit a more scattered distribution, indicating that memories lack structural organization without our link generation and memory evolution components. These visualizations validate that A-Mem can autonomously maintain meaningful memory structures through its dynamic evolution and linking mechanisms.

![Refer to caption](https://ar5iv.labs.arxiv.org/html/2502.12110/assets/x11.png)

(a) Dialogue 3

### B.4 Hyperparameters setting

All hyperparameter k values are presented in Table 5. For models that have already achieved state-of-the-art (SOTA) performance with k=10, we maintain this value without further tuning.

Table 5: Selection of k values in retriever across specific categories and model choices.

| Model | Single Hop | Multi Hop | Temporal | Open Domain | Adversial |
| --- | --- | --- | --- | --- | --- |
| GPT-4o-mini | 40 | 40 | 50 | 50 | 40 |
| GPT-4o | 40 | 40 | 50 | 50 | 40 |
| Qwen2.5-1.5b | 10 | 10 | 10 | 10 | 10 |
| Qwen2.5-3b | 10 | 10 | 50 | 10 | 10 |
| Llama3.2-1b | 10 | 10 | 10 | 10 | 10 |
| Llama3.2-3b | 10 | 20 | 10 | 10 | 10 |

## Appendix C Prompt Templates and Examples

### C.1 Prompt Template of Note Construction

<svg id="A3.SS1.p1.pic1" height="288.92" overflow="visible" version="1.1" width="600"><g transform="translate(0,288.92) matrix(1 0 0 -1 0 0)" fill="#000000" stroke="#000000" stroke-width="0.4pt"><g fill="#000000" fill-opacity="1.0"><path d="M 0 17.72 L 0 271.21 C 0 280.99 7.93 288.92 17.72 288.92 L 582.28 288.92 C 592.07 288.92 600 280.99 600 271.21 L 600 17.72 C 600 7.93 592.07 0 582.28 0 L 17.72 0 C 7.93 0 0 7.93 0 17.72 Z" style="stroke:none"></path></g><g fill="#F9F9F9" fill-opacity="1.0"><path d="M 1.97 17.72 L 1.97 271.21 C 1.97 279.91 9.02 286.96 17.72 286.96 L 582.28 286.96 C 590.98 286.96 598.03 279.91 598.03 271.21 L 598.03 17.72 C 598.03 9.02 590.98 1.97 582.28 1.97 L 17.72 1.97 C 9.02 1.97 1.97 9.02 1.97 17.72 Z" style="stroke:none"></path></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignObject width="556.69" height="261.37" transform="matrix(1 0 0 -1 0 16.6)" overflow="visible" color="#000000"><span style="width:402.3pt;"><span id="A3.SS1.p1.pic1.p1">The prompt template in Note Construction: <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="P_{s1}"><semantics><msub><mi>P</mi> <mrow><mi>s</mi> <mo lspace="0em" rspace="0em"></mo><mn>1</mn></mrow></msub> <annotation encoding="application/x-tex">P_{s1}</annotation></semantics></math><br>Generate a structured analysis of the following content by:<br>1. Identifying the most salient keywords (focus on nouns, verbs, and key concepts)<br>2. Extracting core themes and contextual elements<br>3. Creating relevant categorical tags<br>Format the response as a JSON object:<br>{<br>"keywords": [ // several specific, distinct keywords that capture key concepts and terminology // Order from most to least important // Don’t include keywords that are the name of the speaker or time // At least three keywords, but don’t be too redundant. ],<br>"context": // one sentence summarizing: // - Main topic/domain // - Key arguments/points // - Intended audience/purpose,<br>"tags": [ // several broad categories/themes for classification // Include domain, format, and type tags // At least three tags, but don’t be too redundant. ]<br>}<br>Content for analysis:</span></span></foreignObject></g></g></svg>

### C.2 Prompt Template of Link Generation

<svg id="A3.SS2.p1.pic1" height="205.9" overflow="visible" version="1.1" width="600"><g transform="translate(0,205.9) matrix(1 0 0 -1 0 0)" fill="#000000" stroke="#000000" stroke-width="0.4pt"><g fill="#000000" fill-opacity="1.0"><path d="M 0 17.72 L 0 188.19 C 0 197.97 7.93 205.9 17.72 205.9 L 582.28 205.9 C 592.07 205.9 600 197.97 600 188.19 L 600 17.72 C 600 7.93 592.07 0 582.28 0 L 17.72 0 C 7.93 0 0 7.93 0 17.72 Z" style="stroke:none"></path></g><g fill="#F9F9F9" fill-opacity="1.0"><path d="M 1.97 17.72 L 1.97 188.19 C 1.97 196.88 9.02 203.93 17.72 203.93 L 582.28 203.93 C 590.98 203.93 598.03 196.88 598.03 188.19 L 598.03 17.72 C 598.03 9.02 590.98 1.97 582.28 1.97 L 17.72 1.97 C 9.02 1.97 1.97 9.02 1.97 17.72 Z" style="stroke:none"></path></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignObject width="556.69" height="178.34" transform="matrix(1 0 0 -1 0 16.6)" overflow="visible" color="#000000"><span style="width:402.3pt;"><span id="A3.SS2.p1.pic1.p1">The prompt template in Link Generation: <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="P_{s2}"><semantics><msub><mi>P</mi> <mrow><mi>s</mi> <mo lspace="0em" rspace="0em"></mo><mn>2</mn></mrow></msub> <annotation encoding="application/x-tex">P_{s2}</annotation></semantics></math><br>You are an AI memory evolution agent responsible for managing and evolving a knowledge base.<br>Analyze the the new memory note according to keywords and context, also with their several nearest neighbors memory.<br>The new memory context:<br>{context} content: {content}<br>keywords: {keywords}<br>The nearest neighbors memories: {nearest_neighbors_memories}<br>Based on this information, determine:<br>Should this memory be evolved? Consider its relationships with other memories.</span></span></foreignObject></g></g></svg>

### C.3 Prompt Template of Memory Evolution

<svg id="A3.SS3.p1.pic1" height="506.32" overflow="visible" version="1.1" width="600"><g transform="translate(0,506.32) matrix(1 0 0 -1 0 0)" fill="#000000" stroke="#000000" stroke-width="0.4pt"><g fill="#000000" fill-opacity="1.0"><path d="M 0 17.72 L 0 488.6 C 0 498.39 7.93 506.32 17.72 506.32 L 582.28 506.32 C 592.07 506.32 600 498.39 600 488.6 L 600 17.72 C 600 7.93 592.07 0 582.28 0 L 17.72 0 C 7.93 0 0 7.93 0 17.72 Z" style="stroke:none"></path></g><g fill="#F9F9F9" fill-opacity="1.0"><path d="M 1.97 17.72 L 1.97 488.6 C 1.97 497.3 9.02 504.35 17.72 504.35 L 582.28 504.35 C 590.98 504.35 598.03 497.3 598.03 488.6 L 598.03 17.72 C 598.03 9.02 590.98 1.97 582.28 1.97 L 17.72 1.97 C 9.02 1.97 1.97 9.02 1.97 17.72 Z" style="stroke:none"></path></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignObject width="556.69" height="478.76" transform="matrix(1 0 0 -1 0 16.6)" overflow="visible" color="#000000"><span style="width:402.3pt;"><span id="A3.SS3.p1.pic1.p1">The prompt template in Memory Evolution: <math xmlns="http://www.w3.org/1998/Math/MathML" display="inline" data-latex="P_{s3}"><semantics><msub><mi>P</mi> <mrow><mi>s</mi> <mo lspace="0em" rspace="0em"></mo><mn>3</mn></mrow></msub> <annotation encoding="application/x-tex">P_{s3}</annotation></semantics></math><br>You are an AI memory evolution agent responsible for managing and evolving a knowledge base.<br>Analyze the the new memory note according to keywords and context, also with their several nearest neighbors memory.<br>Make decisions about its evolution.<br>The new memory context:{context}<br>content: {content}<br>keywords: {keywords}<br>The nearest neighbors memories:{nearest_neighbors_memories}<br>Based on this information, determine:<br>1. What specific actions should be taken (strengthen, update_neighbor)?<br>1.1 If choose to strengthen the connection, which memory should it be connected to? Can you give the updated tags of this memory?<br>1.2 If choose to update neighbor, you can update the context and tags of these memories based on the understanding of these memories.<br>Tags should be determined by the content of these characteristic of these memories, which can be used to retrieve them later and categorize them.<br>All the above information should be returned in a list format according to the sequence: [[new_memory],[neighbor_memory_1],...[neighbor_memory_n]]<br>These actions can be combined.<br>Return your decision in JSON format with the following structure: {{<br>"should_evolve": true/false,<br>"actions": ["strengthen", "merge", "prune"],<br>"suggested_connections": ["neighbor_memory_ids"],<br>"tags_to_update": ["tag_1",..."tag_n"],<br>"new_context_neighborhood": ["new context",...,"new context"],<br>"new_tags_neighborhood": [["tag_1",...,"tag_n"],...["tag_1",...,"tag_n"]],<br>}}</span></span></foreignObject></g></g></svg>

### C.4 Examples of Q/A with A-Mem

<svg id="A3.SS4.p1.pic1" height="426.06" overflow="visible" version="1.1" width="600"><g transform="translate(0,426.06) matrix(1 0 0 -1 0 0)" fill="#000000" stroke="#000000" stroke-width="0.4pt"><g fill="#000000" fill-opacity="1.0"><path d="M 0 17.72 L 0 408.35 C 0 418.13 7.93 426.06 17.72 426.06 L 582.28 426.06 C 592.07 426.06 600 418.13 600 408.35 L 600 17.72 C 600 7.93 592.07 0 582.28 0 L 17.72 0 C 7.93 0 0 7.93 0 17.72 Z" style="stroke:none"></path></g><g fill="#F9F9F9" fill-opacity="1.0"><path d="M 1.97 17.72 L 1.97 408.35 C 1.97 417.05 9.02 424.1 17.72 424.1 L 582.28 424.1 C 590.98 424.1 598.03 417.05 598.03 408.35 L 598.03 17.72 C 598.03 9.02 590.98 1.97 582.28 1.97 L 17.72 1.97 C 9.02 1.97 1.97 9.02 1.97 17.72 Z" style="stroke:none"></path></g><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignObject width="556.69" height="398.51" transform="matrix(1 0 0 -1 0 16.6)" overflow="visible" color="#000000"><span style="width:402.3pt;"><span id="A3.SS4.p1.pic1.p1">Example:<br>Question 686: Which hobby did Dave pick up in October 2023?<br>Prediction: photography<br>Reference: photography<br>talk start time:10:54 am on 17 November, 2023<br>memory content: Speaker Davesays: Hey Calvin, long time no talk! A lot has happened. I’ve taken up photography and it’s been great - been taking pics of the scenery around here which is really cool.<br>memory context: The main topic is the speaker’s new hobby of photography, highlighting their enjoyment of capturing local scenery, aimed at engaging a friend in conversation about personal experiences.<br>memory keywords: [’photography’, ’scenery’, ’conversation’, ’experience’, ’hobby’]<br>memory tags: [’hobby’, ’photography’, ’personal development’, ’conversation’, ’leisure’]<br>talk start time:6:38 pm on 21 July, 2023<br>memory content: Speaker Calvinsays: Thanks, Dave! It feels great having my own space to work in. I’ve been experimenting with different genres lately, pushing myself out of my comfort zone. Adding electronic elements to my songs gives them a fresh vibe. It’s been an exciting process of self-discovery and growth!<br>memory context: The speaker discusses their creative process in music, highlighting experimentation with genres and the incorporation of electronic elements for personal growth and artistic evolution.<br>memory keywords: [’space’, ’experimentation’, ’genres’, ’electronic’, ’self-discovery’, ’growth’]<br>memory tags: [’music’, ’creativity’, ’self-improvement’, ’artistic expression’]<br></span></span></foreignObject></g></g></svg>

[^1]: Sönke Ahrens. 2017. *How to Take Smart Notes: One Simple Technique to Boost Writing, Learning and Thinking*. Amazon. Second Edition.

[^2]: Akari Asai, Zeqiu Wu, Yizhong Wang, Avirup Sil, and Hannaneh Hajishirzi. 2023. Self-rag: Learning to retrieve, generate, and critique through self-reflection. *arXiv preprint arXiv:2310.11511*.

[^3]: Satanjeev Banerjee and Alon Lavie. 2005. Meteor: An automatic metric for mt evaluation with improved correlation with human judgments. In *Proceedings of the acl workshop on intrinsic and extrinsic evaluation measures for machine translation and/or summarization*, pages 65–72.

[^4]: Sebastian Borgeaud, Arthur Mensch, Jordan Hoffmann, Trevor Cai, Eliza Rutherford, Katie Millican, George Bm Van Den Driessche, Jean-Baptiste Lespiau, Bogdan Damoc, Aidan Clark, et al. 2022. Improving language models by retrieving from trillions of tokens. In *International conference on machine learning*, pages 2206–2240. PMLR.

[^5]: Xiang Deng, Yu Gu, Boyuan Zheng, Shijie Chen, Sam Stevens, Boshi Wang, Huan Sun, and Yu Su. 2023. Mind2web: Towards a generalist agent for the web. *Advances in Neural Information Processing Systems*, 36:28091–28114.

[^6]: Khant Dev and Singh Taranjeet. 2024. mem0: The memory layer for ai agents. [https://github.com/mem0ai/mem0](https://github.com/mem0ai/mem0).

[^7]: Darren Edge, Ha Trinh, Newman Cheng, Joshua Bradley, Alex Chao, Apurva Mody, Steven Truitt, and Jonathan Larson. 2024. From local to global: A graph rag approach to query-focused summarization. *arXiv preprint arXiv:2404.16130*.

[^8]: Yunfan Gao, Yun Xiong, Xinyu Gao, Kangxiang Jia, Jinliu Pan, Yuxi Bi, Yi Dai, Jiawei Sun, and Haofen Wang. 2023. Retrieval-augmented generation for large language models: A survey. *arXiv preprint arXiv:2312.10997*.

[^9]: I. Ilin. 2023. [Advanced rag techniques: An illustrated overview](https://pub.towardsai.net/advanced-rag-techniques-an-illustrated-overview-04d193d8fec6).

[^10]: Jihyoung Jang, Minseong Boo, and Hyounghun Kim. 2023. Conversation chronicles: Towards diverse temporal and relational dynamics in multi-session conversations. *arXiv preprint arXiv:2310.13420*.

[^11]: Zhengbao Jiang, Frank F Xu, Luyu Gao, Zhiqing Sun, Qian Liu, Jane Dwivedi-Yu, Yiming Yang, Jamie Callan, and Graham Neubig. 2023. Active retrieval augmented generation. *arXiv preprint arXiv:2305.06983*.

[^12]: Mingyu Jin, Qinkai Yu, Dong Shu, Haiyan Zhao, Wenyue Hua, Yanda Meng, Yongfeng Zhang, and Mengnan Du. 2024. [The impact of reasoning step length on large language models](https://aclanthology.org/2024.findings-acl.108). In *Findings of the Association for Computational Linguistics ACL 2024*, pages 1830–1842, Bangkok, Thailand and virtual meeting.

[^13]: David Kadavy. 2021. *Digital Zettelkasten: Principles, Methods, & Examples*. Google Books.

[^14]: Kuang-Huei Lee, Xinyun Chen, Hiroki Furuta, John Canny, and Ian Fischer. 2024. A human-inspired reading agent with gist memory of very long contexts. *arXiv preprint arXiv:2402.09727*.

[^15]: Patrick Lewis, Ethan Perez, Aleksandra Piktus, Fabio Petroni, Vladimir Karpukhin, Naman Goyal, Heinrich Küttler, Mike Lewis, Wen-tau Yih, Tim Rocktäschel, et al. 2020. Retrieval-augmented generation for knowledge-intensive nlp tasks. *Advances in Neural Information Processing Systems*, 33:9459–9474.

[^16]: Chin-Yew Lin. 2004. Rouge: A package for automatic evaluation of summaries. In *Text summarization branches out*, pages 74–81.

[^17]: Xi Victoria Lin, Xilun Chen, Mingda Chen, Weijia Shi, Maria Lomeli, Rich James, Pedro Rodriguez, Jacob Kahn, Gergely Szilvasy, Mike Lewis, et al. 2023. Ra-dit: Retrieval-augmented dual instruction tuning. *arXiv preprint arXiv:2310.01352*.

[^18]: Zhiwei Liu, Weiran Yao, Jianguo Zhang, Liangwei Yang, Zuxin Liu, Juntao Tan, Prafulla K Choubey, Tian Lan, Jason Wu, Huan Wang, et al. 2024. Agentlite: A lightweight library for building and advancing task-oriented llm agent system. *arXiv preprint arXiv:2402.15538*.

[^19]: Adyasha Maharana, Dong-Ho Lee, Sergey Tulyakov, Mohit Bansal, Francesco Barbieri, and Yuwei Fang. 2024. Evaluating very long-term conversational memory of llm agents. *arXiv preprint arXiv:2402.17753*.

[^20]: Kai Mei, Zelong Li, Shuyuan Xu, Ruosong Ye, Yingqiang Ge, and Yongfeng Zhang. 2024. Aios: Llm agent operating system. *arXiv e-prints, pp. arXiv–2403*.

[^21]: Ali Modarressi, Ayyoob Imani, Mohsen Fayyaz, and Hinrich Schütze. 2023. Ret-llm: Towards a general read-write memory for large language models. *arXiv preprint arXiv:2305.14322*.

[^22]: Charles Packer, Sarah Wooders, Kevin Lin, Vivian Fang, Shishir G Patil, Ion Stoica, and Joseph E Gonzalez. 2023. Memgpt: Towards llms as operating systems. *arXiv preprint arXiv:2310.08560*.

[^23]: Kishore Papineni, Salim Roukos, Todd Ward, and Wei-Jing Zhu. 2002. Bleu: a method for automatic evaluation of machine translation. In *Proceedings of the 40th annual meeting of the Association for Computational Linguistics*, pages 311–318.

[^24]: Nils Reimers and Iryna Gurevych. 2019. [Sentence-bert: Sentence embeddings using siamese bert-networks](https://arxiv.org/abs/1908.10084). In *Proceedings of the 2019 Conference on Empirical Methods in Natural Language Processing*. Association for Computational Linguistics.

[^25]: Aymeric Roucher, Albert Villanova del Moral, Thomas Wolf, Leandro von Werra, and Erik Kaunismäki. 2025. ‘smolagents‘: a smol library to build great agentic systems. [https://github.com/huggingface/smolagents](https://github.com/huggingface/smolagents).

[^26]: Zhihong Shao, Yeyun Gong, Yelong Shen, Minlie Huang, Nan Duan, and Weizhu Chen. 2023. Enhancing retrieval-augmented large language models with iterative retrieval-generation synergy. *arXiv preprint arXiv:2305.15294*.

[^27]: Zeru Shi, Kai Mei, Mingyu Jin, Yongye Su, Chaoji Zuo, Wenyue Hua, Wujiang Xu, Yujie Ren, Zirui Liu, Mengnan Du, et al. 2024. From commands to prompts: Llm-based semantic file system for aios. *arXiv preprint arXiv:2410.11843*.

[^28]: Harsh Trivedi, Niranjan Balasubramanian, Tushar Khot, and Ashish Sabharwal. 2022. Interleaving retrieval with chain-of-thought reasoning for knowledge-intensive multi-step questions. *arXiv preprint arXiv:2212.10509*.

[^29]: Bing Wang, Xinnian Liang, Jian Yang, Hui Huang, Shuangzhi Wu, Peihao Wu, Lu Lu, Zejun Ma, and Zhoujun Li. 2023a. Enhancing large language model with self-controlled memory framework. *arXiv preprint arXiv:2304.13343*.

[^30]: Kun Wang, Yuxuan Liang, Xinglin Li, Guohao Li, Bernard Ghanem, Roger Zimmermann, Zhengyang Zhou, Huahui Yi, Yudong Zhang, and Yang Wang. 2023b. Brave the wind and the waves: Discovering robust and generalizable graph lottery tickets. *IEEE Transactions on Pattern Analysis and Machine Intelligence*, 46(5):3388–3405.

[^31]: Xingyao Wang, Boxuan Li, Yufan Song, Frank F Xu, Xiangru Tang, Mingchen Zhuge, Jiayi Pan, Yueqi Song, Bowen Li, Jaskirat Singh, et al. 2024. Openhands: An open platform for ai software developers as generalist agents. *arXiv preprint arXiv:2407.16741*.

[^32]: Zhiruo Wang, Jun Araki, Zhengbao Jiang, Md Rizwan Parvez, and Graham Neubig. 2023c. Learning to filter context for retrieval-augmented generation. *arXiv preprint arXiv:2311.08377*.

[^33]: Lilian Weng. 2023. [Llm-powered autonomous agents](https://lilianweng.github.io/posts/2023-06-23-agent/). *lilianweng.github.io*.

[^34]: J Xu. 2021. Beyond goldfish memory: Long-term open-domain conversation. *arXiv preprint arXiv:2107.07567*.

[^35]: Wujiang Xu, Shaoshuai Li, Mingming Ha, Xiaobo Guo, Qiongxu Ma, Xiaolei Liu, Linxun Chen, and Zhenfeng Zhu. 2023. Neural node matching for multi-target cross domain recommendation. In *2023 IEEE 39th International Conference on Data Engineering (ICDE)*, pages 2154–2166. IEEE.

[^36]: Wujiang Xu, Qitian Wu, Zujie Liang, Jiaojiao Han, Xuying Ning, Yunxiao Shi, Wenfang Lin, and Yongfeng Zhang. 2024a. Slmrec: empowering small language models for sequential recommendation. *arXiv preprint arXiv:2405.17890*.

[^37]: Wujiang Xu, Qitian Wu, Runzhong Wang, Mingming Ha, Qiongxu Ma, Linxun Chen, Bing Han, and Junchi Yan. 2024b. Rethinking cross-domain sequential recommendation under open-world assumptions. In *Proceedings of the ACM on Web Conference 2024*, pages 3173–3184.

[^38]: Wenhao Yu, Hongming Zhang, Xiaoman Pan, Kaixin Ma, Hongwei Wang, and Dong Yu. 2023a. Chain-of-note: Enhancing robustness in retrieval-augmented language models. *arXiv preprint arXiv:2311.09210*.

[^39]: Zichun Yu, Chenyan Xiong, Shi Yu, and Zhiyuan Liu. 2023b. Augmentation-adapted retriever improves generalization of language models as generic plug-in. *arXiv preprint arXiv:2305.17331*.

[^40]: Guibin Zhang, Haonan Dong, Yuchen Zhang, Zhixun Li, Dingshuo Chen, Kai Wang, Tianlong Chen, Yuxuan Liang, Dawei Cheng, and Kun Wang. 2024a. Gder: Safeguarding efficiency, balancing, and robustness via prototypical graph pruning. *arXiv preprint arXiv:2410.13761*.

[^41]: Guibin Zhang, Yanwei Yue, Zhixun Li, Sukwon Yun, Guancheng Wan, Kun Wang, Dawei Cheng, Jeffrey Xu Yu, and Tianlong Chen. 2024b. Cut the crap: An economical communication pipeline for llm-based multi-agent systems. *arXiv preprint arXiv:2410.02506*.

[^42]: Guibin Zhang, Yanwei Yue, Xiangguo Sun, Guancheng Wan, Miao Yu, Junfeng Fang, Kun Wang, and Dawei Cheng. 2024c. G-designer: Architecting multi-agent communication topologies via graph neural networks. *arXiv preprint arXiv:2410.11782*.

[^43]: Wanjun Zhong, Lianghong Guo, Qiqi Gao, He Ye, and Yanlin Wang. 2024. Memorybank: Enhancing large language models with long-term memory. In *Proceedings of the AAAI Conference on Artificial Intelligence*, volume 38, pages 19724–19731.