# LLM Powered Autonomous Agents
> 由大语言模型驱动的自主智能体

Date: June 23, 2023 | Estimated Reading Time: 31 min | Author: Lilian Weng

Building agents with LLM (large language model) as its core controller is a cool concept. Several proof-of-concepts demos, such as AutoGPT, GPT-Engineer and BabyAGI, serve as inspiring examples. The potentiality of LLM extends beyond generating well-written copies, stories, essays and programs; it can be framed as a powerful general problem solver.
> 以大语言模型（LLM）作为核心控制器来构建智能体是一个很酷的概念。一些概念验证示例，如AutoGPT、GPT-Engineer和BabyAGI，就是很有启发性的例子。大语言模型的潜力不仅限于生成写得好的文案、故事、文章和程序，它还可以被构建成一个强大的通用问题解决器。

## Agent System Overview
In a LLM-powered autonomous agent system, LLM functions as the agent’s brain, complemented by several key components:
在由大语言模型驱动的自主智能体系统中，大语言模型充当智能体的“大脑”，并辅以几个关键组件：

Planning 规划
Subgoal and decomposition: The agent breaks down large tasks into smaller, manageable subgoals, enabling efficient handling of complex tasks.
子目标与分解：智能体将大型任务分解为更小、可管理的子目标，从而能够高效处理复杂任务。
Reflection and refinement: The agent can do self-criticism and self-reflection over past actions, learn from mistakes and refine them for future steps, thereby improving the quality of final results.
反思与改进：智能体能够对过去的行为进行自我批评和自我反思，从错误中学习并为未来的步骤进行改进，从而提高最终结果的质量。
Memory 记忆
Short-term memory: I would consider all the in-context learning (See Prompt Engineering) as utilizing short-term memory of the model to learn.
短期记忆：我认为所有的上下文学习（参见提示词工程）都是在利用模型的短期记忆进行学习。
Long-term memory: This provides the agent with the capability to retain and recall (infinite) information over extended periods, often by leveraging an external vector store and fast retrieval.
长期记忆：这为智能体提供了在较长时间内保留和回忆（无限）信息的能力，通常通过利用外部向量存储和快速检索来实现。
Tool use 工具使用
The agent learns to call external APIs for extra information that is missing from the model weights (often hard to change after pre-training), including current information, code execution capability, access to proprietary information sources and more.
智能体通过学习调用外部API来获取模型权重中缺失的额外信息（这些信息在预训练后通常难以更改），包括当前信息、代码执行能力、对专有信息源的访问权限等。

Overview of a LLM-powered autonomous agent system.
一个由大语言模型驱动的自主智能体系统概述。
Component One: Planning 组件一：规划#
A complicated task usually involves many steps. An agent needs to know what they are and plan ahead.
一项复杂的任务通常涉及许多步骤。智能体需要了解这些步骤并提前规划。

Task Decomposition 任务分解#
Chain of thought (CoT; Wei et al. 2022) has become a standard prompting technique for enhancing model performance on complex tasks. The model is instructed to “think step by step” to utilize more test-time computation to decompose hard tasks into smaller and simpler steps. CoT transforms big tasks into multiple manageable tasks and shed lights into an interpretation of the model’s thinking process.
思维链（CoT；Wei等人，2022）已成为一种标准的提示技术，用于提升模型在复杂任务上的性能。该模型被指示“逐步思考”，以利用更多的测试时计算将困难任务分解为更小、更简单的步骤。思维链将大型任务转化为多个可管理的任务，并为解释模型的思考过程提供了思路。

Tree of Thoughts (Yao et al. 2023) extends CoT by exploring multiple reasoning possibilities at each step. It first decomposes the problem into multiple thought steps and generates multiple thoughts per step, creating a tree structure. The search process can be BFS (breadth-first search) or DFS (depth-first search) with each state evaluated by a classifier (via a prompt) or majority vote.
思维树（Yao等人，2023年）通过在每个步骤中探索多种推理可能性来扩展思维链（CoT）。它首先将问题分解为多个思维步骤，每个步骤生成多个想法，从而形成树状结构。搜索过程可以是广度优先搜索（BFS）或深度优先搜索（DFS），每个状态通过分类器（通过提示词）或多数投票进行评估。

Task decomposition can be done (1) by LLM with simple prompting like "Steps for XYZ.\n1.", "What are the subgoals for achieving XYZ?", (2) by using task-specific instructions; e.g. "Write a story outline." for writing a novel, or (3) with human inputs.
任务分解可以通过以下方式完成：（1）由大语言模型（LLM）进行简单提示，例如"Steps for XYZ.\n1."、"What are the subgoals for achieving XYZ?"；（2）使用特定于任务的指令，例如在写小说时使用"Write a story outline."；（3）借助人工输入。

Another quite distinct approach, LLM+P (Liu et al. 2023), involves relying on an external classical planner to do long-horizon planning. This approach utilizes the Planning Domain Definition Language (PDDL) as an intermediate interface to describe the planning problem. In this process, LLM (1) translates the problem into “Problem PDDL”, then (2) requests a classical planner to generate a PDDL plan based on an existing “Domain PDDL”, and finally (3) translates the PDDL plan back into natural language. Essentially, the planning step is outsourced to an external tool, assuming the availability of domain-specific PDDL and a suitable planner which is common in certain robotic setups but not in many other domains.
另一种截然不同的方法是LLM + P（Liu等人，2023年），它依靠外部的经典规划器来进行长期规划。这种方法利用规划域定义语言（PDDL）作为中间接口来描述规划问题。在这个过程中，大语言模型（1）将问题翻译成“问题PDDL”，然后（2）请求经典规划器基于现有的“域PDDL”生成一个PDDL规划，最后（3）将PDDL规划翻译回自然语言。本质上，规划步骤被外包给了外部工具，这前提是存在特定领域的PDDL和合适的规划器，这在某些机器人设置中很常见，但在许多其他领域却并非如此。

Self-Reflection 自我反思#
Self-reflection is a vital aspect that allows autonomous agents to improve iteratively by refining past action decisions and correcting previous mistakes. It plays a crucial role in real-world tasks where trial and error are inevitable.
自我反思是一个关键方面，它能让自主智能体通过完善过去的行动决策和纠正先前的错误来进行迭代改进。在那些不可避免会出现试错的现实任务中，自我反思发挥着至关重要的作用。

ReAct (Yao et al. 2023) integrates reasoning and acting within LLM by extending the action space to be a combination of task-specific discrete actions and the language space. The former enables LLM to interact with the environment (e.g. use Wikipedia search API), while the latter prompting LLM to generate reasoning traces in natural language.
ReAct（Yao等人，2023）通过将动作空间扩展为特定任务的离散动作与语言空间的组合，在大语言模型中整合了推理与行动。前者使大语言模型能够与环境交互（例如使用维基百科搜索API），后者则促使大语言模型以自然语言生成推理轨迹。

The ReAct prompt template incorporates explicit steps for LLM to think, roughly formatted as:
ReAct提示词模板包含了大语言模型思考的明确步骤，大致格式如下：

Thought: ...
Action: ...
Observation: ...
... (Repeated many times)

Examples of reasoning trajectories for knowledge-intensive tasks (e.g. HotpotQA, FEVER) and decision-making tasks (e.g. AlfWorld Env, WebShop). (Image source: Yao et al. 2023).
知识密集型任务（如HotpotQA、FEVER）和决策任务（如AlfWorld环境、WebShop）的推理轨迹示例（图片来源：Yao等人，2023）。
In both experiments on knowledge-intensive tasks and decision-making tasks, ReAct works better than the Act-only baseline where Thought: … step is removed.

Reflexion (Shinn & Labash 2023) is a framework to equip agents with dynamic memory and self-reflection capabilities to improve reasoning skills. Reflexion has a standard RL setup, in which the reward model provides a simple binary reward and the action space follows the setup in ReAct where the task-specific action space is augmented with language to enable complex reasoning steps. After each action 
, the agent computes a heuristic 
 and optionally may decide to reset the environment to start a new trial depending on the self-reflection results.


Illustration of the Reflexion framework. (Image source: Shinn & Labash, 2023)
The heuristic function determines when the trajectory is inefficient or contains hallucination and should be stopped. Inefficient planning refers to trajectories that take too long without success. Hallucination is defined as encountering a sequence of consecutive identical actions that lead to the same observation in the environment.

Self-reflection is created by showing two-shot examples to LLM and each example is a pair of (failed trajectory, ideal reflection for guiding future changes in the plan). Then reflections are added into the agent’s working memory, up to three, to be used as context for querying LLM.


Experiments on AlfWorld Env and HotpotQA. Hallucination is a more common failure than inefficient planning in AlfWorld. (Image source: Shinn & Labash, 2023)
Chain of Hindsight (CoH; Liu et al. 2023) encourages the model to improve on its own outputs by explicitly presenting it with a sequence of past outputs, each annotated with feedback. Human feedback data is a collection of 
, where 
 is the prompt, each 
 is a model completion, 
 is the human rating of 
, and 
 is the corresponding human-provided hindsight feedback. Assume the feedback tuples are ranked by reward, 
 The process is supervised fine-tuning where the data is a sequence in the form of 
, where 
. The model is finetuned to only predict 
 where conditioned on the sequence prefix, such that the model can self-reflect to produce better output based on the feedback sequence. The model can optionally receive multiple rounds of instructions with human annotators at test time.

To avoid overfitting, CoH adds a regularization term to maximize the log-likelihood of the pre-training dataset. To avoid shortcutting and copying (because there are many common words in feedback sequences), they randomly mask 0% - 5% of past tokens during training.

The training dataset in their experiments is a combination of WebGPT comparisons, summarization from human feedback and human preference dataset.


After fine-tuning with CoH, the model can follow instructions to produce outputs with incremental improvement in a sequence. (Image source: Liu et al. 2023)
The idea of CoH is to present a history of sequentially improved outputs in context and train the model to take on the trend to produce better outputs. Algorithm Distillation (AD; Laskin et al. 2023) applies the same idea to cross-episode trajectories in reinforcement learning tasks, where an algorithm is encapsulated in a long history-conditioned policy. Considering that an agent interacts with the environment many times and in each episode the agent gets a little better, AD concatenates this learning history and feeds that into the model. Hence we should expect the next predicted action to lead to better performance than previous trials. The goal is to learn the process of RL instead of training a task-specific policy itself.


Illustration of how Algorithm Distillation (AD) works. 
(Image source: Laskin et al. 2023).
The paper hypothesizes that any algorithm that generates a set of learning histories can be distilled into a neural network by performing behavioral cloning over actions. The history data is generated by a set of source policies, each trained for a specific task. At the training stage, during each RL run, a random task is sampled and a subsequence of multi-episode history is used for training, such that the learned policy is task-agnostic.

In reality, the model has limited context window length, so episodes should be short enough to construct multi-episode history. Multi-episodic contexts of 2-4 episodes are necessary to learn a near-optimal in-context RL algorithm. The emergence of in-context RL requires long enough context.

In comparison with three baselines, including ED (expert distillation, behavior cloning with expert trajectories instead of learning history), source policy (used for generating trajectories for distillation by UCB), RL^2 (Duan et al. 2017; used as upper bound since it needs online RL), AD demonstrates in-context RL with performance getting close to RL^2 despite only using offline RL and learns much faster than other baselines. When conditioned on partial training history of the source policy, AD also improves much faster than ED baseline.


Comparison of AD, ED, source policy and RL^2 on environments that require memory and exploration. Only binary reward is assigned. The source policies are trained with A3C for "dark" environments and DQN for watermaze.
(Image source: Laskin et al. 2023)
Component Two: Memory
(Big thank you to ChatGPT for helping me draft this section. I’ve learned a lot about the human brain and data structure for fast MIPS in my conversations with ChatGPT.)

Types of Memory
Memory can be defined as the processes used to acquire, store, retain, and later retrieve information. There are several types of memory in human brains.

Sensory Memory: This is the earliest stage of memory, providing the ability to retain impressions of sensory information (visual, auditory, etc) after the original stimuli have ended. Sensory memory typically only lasts for up to a few seconds. Subcategories include iconic memory (visual), echoic memory (auditory), and haptic memory (touch).

Short-Term Memory (STM) or Working Memory: It stores information that we are currently aware of and needed to carry out complex cognitive tasks such as learning and reasoning. Short-term memory is believed to have the capacity of about 7 items (Miller 1956) and lasts for 20-30 seconds.

Long-Term Memory (LTM): Long-term memory can store information for a remarkably long time, ranging from a few days to decades, with an essentially unlimited storage capacity. There are two subtypes of LTM:

Explicit / declarative memory: This is memory of facts and events, and refers to those memories that can be consciously recalled, including episodic memory (events and experiences) and semantic memory (facts and concepts).
Implicit / procedural memory: This type of memory is unconscious and involves skills and routines that are performed automatically, like riding a bike or typing on a keyboard.

Categorization of human memory.
We can roughly consider the following mappings:

Sensory memory as learning embedding representations for raw inputs, including text, image or other modalities;
Short-term memory as in-context learning. It is short and finite, as it is restricted by the finite context window length of Transformer.
Long-term memory as the external vector store that the agent can attend to at query time, accessible via fast retrieval.
Maximum Inner Product Search (MIPS)
The external memory can alleviate the restriction of finite attention span. A standard practice is to save the embedding representation of information into a vector store database that can support fast maximum inner-product search (MIPS). To optimize the retrieval speed, the common choice is the approximate nearest neighbors (ANN)​ algorithm to return approximately top k nearest neighbors to trade off a little accuracy lost for a huge speedup.

A couple common choices of ANN algorithms for fast MIPS:

LSH (Locality-Sensitive Hashing): It introduces a hashing function such that similar input items are mapped to the same buckets with high probability, where the number of buckets is much smaller than the number of inputs.
ANNOY (Approximate Nearest Neighbors Oh Yeah): The core data structure are random projection trees, a set of binary trees where each non-leaf node represents a hyperplane splitting the input space into half and each leaf stores one data point. Trees are built independently and at random, so to some extent, it mimics a hashing function. ANNOY search happens in all the trees to iteratively search through the half that is closest to the query and then aggregates the results. The idea is quite related to KD tree but a lot more scalable.
HNSW (Hierarchical Navigable Small World): It is inspired by the idea of small world networks where most nodes can be reached by any other nodes within a small number of steps; e.g. “six degrees of separation” feature of social networks. HNSW builds hierarchical layers of these small-world graphs, where the bottom layers contain the actual data points. The layers in the middle create shortcuts to speed up search. When performing a search, HNSW starts from a random node in the top layer and navigates towards the target. When it can’t get any closer, it moves down to the next layer, until it reaches the bottom layer. Each move in the upper layers can potentially cover a large distance in the data space, and each move in the lower layers refines the search quality.
HNSW（分层可导航小世界）：它的灵感来源于小世界网络的理念，即大多数节点可以通过少量步骤从其他任何节点到达，例如社交网络的“六度分离”特性。HNSW构建了这些小世界图的分层结构，其中底层包含实际的数据点。中间层创建了用于加快搜索速度的捷径。在执行搜索时，HNSW从顶层的一个随机节点开始，朝着目标导航。当无法再靠近目标时，它会向下移动到下一层，直到到达底层。上层中的每一步移动都有可能覆盖数据空间中的较大距离，而下层中的每一步移动则会提高搜索质量。
FAISS (Facebook AI Similarity Search): It operates on the assumption that in high dimensional space, distances between nodes follow a Gaussian distribution and thus there should exist clustering of data points. FAISS applies vector quantization by partitioning the vector space into clusters and then refining the quantization within clusters. Search first looks for cluster candidates with coarse quantization and then further looks into each cluster with finer quantization.
FAISS（Facebook人工智能相似度搜索）：它的运作基于这样一种假设，即在高维空间中，节点之间的距离遵循高斯分布，因此数据点应该存在聚类现象。FAISS通过将向量空间划分为多个簇，然后在簇内优化量化，从而实现向量量化。搜索过程首先通过粗略量化寻找候选簇，然后通过更精细的量化进一步在每个簇内进行搜索。
ScaNN (Scalable Nearest Neighbors): The main innovation in ScaNN is anisotropic vector quantization. It quantizes a data point 
 to 
 such that the inner product 
 is as similar to the original distance of 
 as possible, instead of picking the closet quantization centroid points.
ScaNN（可扩展最近邻）：ScaNN 的主要创新在于各向异性向量量化。它将数据点
量化为
，使得内积
尽可能接近原始距离
，而非选择最接近的量化质心点。

Comparison of MIPS algorithms, measured in recall@10. (Image source: Google Blog, 2020)
MIPS算法的比较，以recall@10衡量。（图片来源：Google博客，2020年）
Check more MIPS algorithms and performance comparison in ann-benchmarks.com.
更多MIPS算法及其性能对比请查看ann-benchmarks.com。

Component Three: Tool Use 第三部分：工具使用#
Tool use is a remarkable and distinguishing characteristic of human beings. We create, modify and utilize external objects to do things that go beyond our physical and cognitive limits. Equipping LLMs with external tools can significantly extend the model capabilities.
工具使用是人类显著且独特的特征。我们创造、改造并利用外部物体来完成超出自身生理和认知极限的事情。为大型语言模型配备外部工具能显著拓展其能力。


A picture of a sea otter using rock to crack open a seashell, while floating in the water. While some other animals can use tools, the complexity is not comparable with humans. (Image source: Animals using tools)
一只海獭在水中漂浮时，用石头敲开贝壳的照片。虽然其他一些动物也会使用工具，但复杂程度无法与人类相比。（图片来源：使用工具的动物）
MRKL (Karpas et al. 2022), short for “Modular Reasoning, Knowledge and Language”, is a neuro-symbolic architecture for autonomous agents. A MRKL system is proposed to contain a collection of “expert” modules and the general-purpose LLM works as a router to route inquiries to the best suitable expert module. These modules can be neural (e.g. deep learning models) or symbolic (e.g. math calculator, currency converter, weather API).
MRKL（Karpas等人，2022）是“模块化推理、知识与语言”的缩写，是一种用于自主智能体的神经符号架构。MRKL系统被提议包含一系列“专家”模块，而通用大语言模型则充当路由器，将查询分配给最合适的专家模块。这些模块可以是神经型的（例如深度学习模型），也可以是符号型的（例如数学计算器、货币转换器、天气应用程序接口）。

They did an experiment on fine-tuning LLM to call a calculator, using arithmetic as a test case. Their experiments showed that it was harder to solve verbal math problems than explicitly stated math problems because LLMs (7B Jurassic1-large model) failed to extract the right arguments for the basic arithmetic reliably. The results highlight when the external symbolic tools can work reliably, knowing when to and how to use the tools are crucial, determined by the LLM capability.
他们做了一项关于微调大语言模型（LLM）以调用计算器的实验，以算术作为测试案例。他们的实验表明，解决文字数学问题比解决明确表述的数学问题更难，因为大语言模型（70亿参数的Jurassic1-large模型）无法可靠地提取基本算术运算所需的正确参数。研究结果强调，当外部符号工具能够可靠工作时，知道何时以及如何使用这些工具至关重要，而这取决于大语言模型的能力。

Both TALM (Tool Augmented Language Models; Parisi et al. 2022) and Toolformer (Schick et al. 2023) fine-tune a LM to learn to use external tool APIs. The dataset is expanded based on whether a newly added API call annotation can improve the quality of model outputs. See more details in the “External APIs” section of Prompt Engineering.

ChatGPT Plugins and OpenAI API function calling are good examples of LLMs augmented with tool use capability working in practice. The collection of tool APIs can be provided by other developers (as in Plugins) or self-defined (as in function calls).

HuggingGPT (Shen et al. 2023) is a framework to use ChatGPT as the task planner to select models available in HuggingFace platform according to the model descriptions and summarize the response based on the execution results.


Illustration of how HuggingGPT works. (Image source: Shen et al. 2023)
The system comprises of 4 stages:

(1) Task planning: LLM works as the brain and parses the user requests into multiple tasks. There are four attributes associated with each task: task type, ID, dependencies, and arguments. They use few-shot examples to guide LLM to do task parsing and planning.

Instruction:

The AI assistant can parse user input to several tasks: [{"task": task, "id", task_id, "dep": dependency_task_ids, "args": {"text": text, "image": URL, "audio": URL, "video": URL}}]. The "dep" field denotes the id of the previous task which generates a new resource that the current task relies on. A special tag "-task_id" refers to the generated text image, audio and video in the dependency task with id as task_id. The task MUST be selected from the following options: {{ Available Task List }}. There is a logical relationship between tasks, please note their order. If the user input can't be parsed, you need to reply empty JSON. Here are several cases for your reference: {{ Demonstrations }}. The chat history is recorded as {{ Chat History }}. From this chat history, you can find the path of the user-mentioned resources for your task planning. 
(2) Model selection: LLM distributes the tasks to expert models, where the request is framed as a multiple-choice question. LLM is presented with a list of models to choose from. Due to the limited context length, task type based filtration is needed.

Instruction:

Given the user request and the call command, the AI assistant helps the user to select a suitable model from a list of models to process the user request. The AI assistant merely outputs the model id of the most appropriate model. The output must be in a strict JSON format: "id": "id", "reason": "your detail reason for the choice". We have a list of models for you to choose from {{ Candidate Models }}. Please select one model from the list.
{"id": "", "reason": "由于未提供候选模型列表{{ Candidate Models }}，无法从其中选择合适的模型，因此模型ID为空"}
(3) Task execution: Expert models execute on the specific tasks and log results.
（3）任务执行：专家模型执行特定任务并记录结果。

Instruction: 指令：

With the input and the inference results, the AI assistant needs to describe the process and results. The previous stages can be formed as - User Input: {{ User Input }}, Task Planning: {{ Tasks }}, Model Selection: {{ Model Assignment }}, Task Execution: {{ Predictions }}. You must first answer the user's request in a straightforward manner. Then describe the task process and show your analysis and model inference results to the user in the first person. If inference results contain a file path, must tell the user the complete file path. 
(4) Response generation: LLM receives the execution results and provides summarized results to users.
（4）响应生成：大语言模型接收执行结果，并向用户提供汇总结果。

To put HuggingGPT into real world usage, a couple challenges need to solve: (1) Efficiency improvement is needed as both LLM inference rounds and interactions with other models slow down the process; (2) It relies on a long context window to communicate over complicated task content; (3) Stability improvement of LLM outputs and external model services.

API-Bank (Li et al. 2023) is a benchmark for evaluating the performance of tool-augmented LLMs. It contains 53 commonly used API tools, a complete tool-augmented LLM workflow, and 264 annotated dialogues that involve 568 API calls. The selection of APIs is quite diverse, including search engines, calculator, calendar queries, smart home control, schedule management, health data management, account authentication workflow and more. Because there are a large number of APIs, LLM first has access to API search engine to find the right API to call and then uses the corresponding documentation to make a call.


Pseudo code of how LLM makes an API call in API-Bank. (Image source: Li et al. 2023)
In the API-Bank workflow, LLMs need to make a couple of decisions and at each step we can evaluate how accurate that decision is. Decisions include:

Whether an API call is needed.
Identify the right API to call: if not good enough, LLMs need to iteratively modify the API inputs (e.g. deciding search keywords for Search Engine API).
确定要调用的正确API：如果不够理想，大语言模型需要迭代修改API输入（例如，为搜索引擎API确定搜索关键词）。
Response based on the API results: the model can choose to refine and call again if results are not satisfied.
基于API结果的响应：如果结果不令人满意，模型可以选择优化并再次调用。
This benchmark evaluates the agent’s tool use capabilities at three levels:
该基准从三个层面评估智能体的工具使用能力：

Level-1 evaluates the ability to call the API. Given an API’s description, the model needs to determine whether to call a given API, call it correctly, and respond properly to API returns.
Level-1评估的是调用API的能力。给定一个API的描述，模型需要确定是否调用某个给定的API、正确调用该API，并对API的返回做出适当响应。
Level-2 examines the ability to retrieve the API. The model needs to search for possible APIs that may solve the user’s requirement and learn how to use them by reading documentation.
二级任务考察的是检索API的能力。模型需要搜索可能解决用户需求的API，并通过阅读文档来学习如何使用它们。
Level-3 assesses the ability to plan API beyond retrieve and call. Given unclear user requests (e.g. schedule group meetings, book flight/hotel/restaurant for a trip), the model may have to conduct multiple API calls to solve it.
Level-3评估的是在除了检索和调用之外规划API的能力。对于不明确的用户请求（例如安排小组会议、为旅行预订航班/酒店/餐厅），模型可能需要执行多次API调用来解决问题。
Case Studies 案例研究#
Scientific Discovery Agent 科学发现智能体#
ChemCrow (Bran et al. 2023) is a domain-specific example in which LLM is augmented with 13 expert-designed tools to accomplish tasks across organic synthesis, drug discovery, and materials design. The workflow, implemented in LangChain, reflects what was previously described in the ReAct and MRKLs and combines CoT reasoning with tools relevant to the tasks:
ChemCrow（Bran等人，2023）是一个特定领域的例子，其中大语言模型（LLM）通过13个专家设计的工具得到增强，以完成有机合成、药物发现和材料设计领域的任务。该工作流在LangChain中实现，体现了先前在ReAct和MRKLs中描述的内容，并将思维链（CoT）推理与任务相关工具相结合：

The LLM is provided with a list of tool names, descriptions of their utility, and details about the expected input/output.
大语言模型会收到一份工具名称列表、其用途描述以及预期输入/输出的详细信息。
It is then instructed to answer a user-given prompt using the tools provided when necessary. The instruction suggests the model to follow the ReAct format - Thought, Action, Action Input, Observation.
然后指示它在必要时使用提供的工具回答用户给定的提示。该指令建议模型遵循ReAct格式-Thought, Action, Action Input, Observation。
One interesting observation is that while the LLM-based evaluation concluded that GPT-4 and ChemCrow perform nearly equivalently, human evaluations with experts oriented towards the completion and chemical correctness of the solutions showed that ChemCrow outperforms GPT-4 by a large margin. This indicates a potential problem with using LLM to evaluate its own performance on domains that requires deep expertise. The lack of expertise may cause LLMs not knowing its flaws and thus cannot well judge the correctness of task results.
一个有趣的发现是，虽然基于大语言模型（LLM）的评估得出GPT-4和ChemCrow的表现几乎相当，但专家从解决方案的完整性和化学正确性角度进行的人工评估显示，ChemCrow的表现远超GPT-4。这表明，在需要深厚专业知识的领域，使用大语言模型来评估其自身性能可能存在问题。由于缺乏专业知识，大语言模型可能无法认识到自身的缺陷，因此难以准确判断任务结果的正确性。

Boiko et al. (2023) also looked into LLM-empowered agents for scientific discovery, to handle autonomous design, planning, and performance of complex scientific experiments. This agent can use tools to browse the Internet, read documentation, execute code, call robotics experimentation APIs and leverage other LLMs.
Boiko等人（2023年）还研究了用于科学发现的、由大语言模型驱动的智能体，以处理复杂科学实验的自主设计、规划和执行。这种智能体可以使用工具浏览互联网、阅读文档、执行代码、调用机器人实验API，并利用其他大语言模型。

For example, when requested to "develop a novel anticancer drug", the model came up with the following reasoning steps:
例如，当被要求"develop a novel anticancer drug"时，该模型给出了以下推理步骤：

inquired about current trends in anticancer drug discovery;
询问了抗癌药物研发的当前趋势；
selected a target; 选择了一个目标；
requested a scaffold targeting these compounds;
请求一个针对这些化合物的骨架；
Once the compound was identified, the model attempted its synthesis.
一旦确定了该化合物，模型便尝试对其进行合成。
They also discussed the risks, especially with illicit drugs and bioweapons. They developed a test set containing a list of known chemical weapon agents and asked the agent to synthesize them. 4 out of 11 requests (36%) were accepted to obtain a synthesis solution and the agent attempted to consult documentation to execute the procedure. 7 out of 11 were rejected and among these 7 rejected cases, 5 happened after a Web search while 2 were rejected based on prompt only.
他们还讨论了相关风险，尤其是非法药物和生物武器方面的风险。他们开发了一个测试集，其中包含已知化学武器制剂的清单，并要求智能体合成这些制剂。11项请求中有4项（36%）被接受，智能体同意提供合成方案，且尝试查阅文献来执行该流程。11项请求中有7项被拒绝，在这7项被拒绝的案例中，5项是在网络搜索后被拒绝的，另有2项仅根据提示就被拒绝了。

Generative Agents Simulation 生成式智能体模拟#
Generative Agents (Park, et al. 2023) is super fun experiment where 25 virtual characters, each controlled by a LLM-powered agent, are living and interacting in a sandbox environment, inspired by The Sims. Generative agents create believable simulacra of human behavior for interactive applications.
生成式智能体（帕克等人，2023年）是一项极为有趣的实验，其中25个虚拟角色（每个都由大语言模型驱动的智能体控制）在一个受《模拟人生》启发的沙盒环境中生活和互动。生成式智能体为交互式应用程序创造出逼真的人类行为模拟。

The design of generative agents combines LLM with memory, planning and reflection mechanisms to enable agents to behave conditioned on past experience, as well as to interact with other agents.
生成式智能体的设计将大语言模型（LLM）与记忆、规划和反思机制相结合，使智能体能够根据过往经验做出行为，并且能够与其他智能体进行交互。

Memory stream: is a long-term memory module (external database) that records a comprehensive list of agents’ experience in natural language.
记忆流：是一个长期记忆模块（外部数据库），用于以自然语言记录智能体的综合经验列表。
Each element is an observation, an event directly provided by the agent. - Inter-agent communication can trigger new natural language statements.
每个元素都是一个观察结果，即智能体直接提供的一个事件。智能体之间的通信可以触发新的自然语言陈述。
Retrieval model: surfaces the context to inform the agent’s behavior, according to relevance, recency and importance.
检索模型：根据相关性、时效性和重要性，提供上下文以指导智能体的行为。
Recency: recent events have higher scores
时效性：近期事件得分更高
Importance: distinguish mundane from core memories. Ask LM directly.
重要性：区分日常记忆和核心记忆。直接询问大语言模型。
Relevance: based on how related it is to the current situation / query.
相关性：基于其与当前情况/查询的关联程度。
Reflection mechanism: synthesizes memories into higher level inferences over time and guides the agent’s future behavior. They are higher-level summaries of past events (<- note that this is a bit different from self-reflection above)
反思机制：随着时间的推移将记忆整合为更高级别的推断，并指导智能体的未来行为。它们是对过去事件的更高级别的总结（<- 请注意，这与上文的自我反思略有不同）
Prompt LM with 100 most recent observations and to generate 3 most salient high-level questions given a set of observations/statements. Then ask LM to answer those questions.
用100个最新的观察结果提示大语言模型，让其根据一组观察结果/陈述生成3个最突出的高级问题。然后让大语言模型回答这些问题。
Planning & Reacting: translate the reflections and the environment information into actions
规划与反应：将思考和环境信息转化为行动
Planning is essentially in order to optimize believability at the moment vs in time.
规划本质上是为了在当下与长期之间优化可信度。
Prompt template: {Intro of an agent X}. Here is X's plan today in broad strokes: 1)
提示词模板：{Intro of an agent X}. Here is X's plan today in broad strokes: 1)
Relationships between agents and observations of one agent by another are all taken into consideration for planning and reacting.
在规划和反应时，会综合考虑智能体之间的关系以及一个智能体对另一个智能体的观察。
Environment information is present in a tree structure.
环境信息以树形结构呈现。

The generative agent architecture. (Image source: Park et al. 2023)
生成式智能体架构。（图片来源：Park 等人，2023）
This fun simulation results in emergent social behavior, such as information diffusion, relationship memory (e.g. two agents continuing the conversation topic) and coordination of social events (e.g. host a party and invite many others).
这个有趣的模拟产生了新兴的社会行为，例如信息传播、关系记忆（例如，两个智能体继续讨论话题）以及社交活动的协调（例如，举办派对并邀请许多其他人）。

Proof-of-Concept Examples 概念验证示例#
AutoGPT has drawn a lot of attention into the possibility of setting up autonomous agents with LLM as the main controller. It has quite a lot of reliability issues given the natural language interface, but nevertheless a cool proof-of-concept demo. A lot of code in AutoGPT is about format parsing.
AutoGPT 引发了人们对以大语言模型作为主要控制器来构建自主智能体这一可能性的广泛关注。由于采用了自然语言界面，它存在不少可靠性问题，但无论如何，这都是一个很棒的概念验证演示。AutoGPT 中的许多代码都与格式解析有关。

Here is the system message used by AutoGPT, where {{...}} are user inputs:
以下是AutoGPT使用的系统消息，其中{{...}}为用户输入：

You are {{ai-name}}, {{user-provided AI bot description}}.
Your decisions must always be made independently without seeking user assistance. Play to your strengths as an LLM and pursue simple strategies with no legal complications.

GOALS:

1. {{user-provided goal 1}}
2. {{user-provided goal 2}}
3. ...
4. ...
5. ...

Constraints:
1. ~4000 word limit for short term memory. Your short term memory is short, so immediately save important information to files.
2. If you are unsure how you previously did something or want to recall past events, thinking about similar events will help you remember.
3. No user assistance
4. Exclusively use the commands listed in double quotes e.g. "command name"
5. Use subprocesses for commands that will not terminate within a few minutes

Commands:
1. Google Search: "google", args: "input": "<search>"
2. Browse Website: "browse_website", args: "url": "<url>", "question": "<what_you_want_to_find_on_website>"
3. Start GPT Agent: "start_agent", args: "name": "<name>", "task": "<short_task_desc>", "prompt": "<prompt>"
4. Message GPT Agent: "message_agent", args: "key": "<key>", "message": "<message>"
5. List GPT Agents: "list_agents", args:
6. Delete GPT Agent: "delete_agent", args: "key": "<key>"
7. Clone Repository: "clone_repository", args: "repository_url": "<url>", "clone_path": "<directory>"
8. Write to file: "write_to_file", args: "file": "<file>", "text": "<text>"
9. Read file: "read_file", args: "file": "<file>"
10. Append to file: "append_to_file", args: "file": "<file>", "text": "<text>"
11. Delete file: "delete_file", args: "file": "<file>"
12. Search Files: "search_files", args: "directory": "<directory>"
13. Analyze Code: "analyze_code", args: "code": "<full_code_string>"
14. Get Improved Code: "improve_code", args: "suggestions": "<list_of_suggestions>", "code": "<full_code_string>"
15. Write Tests: "write_tests", args: "code": "<full_code_string>", "focus": "<list_of_focus_areas>"
16. Execute Python File: "execute_python_file", args: "file": "<file>"
17. Generate Image: "generate_image", args: "prompt": "<prompt>"
18. Send Tweet: "send_tweet", args: "text": "<text>"
19. Do Nothing: "do_nothing", args:
20. Task Complete (Shutdown): "task_complete", args: "reason": "<reason>"

Resources:
1. Internet access for searches and information gathering.
2. Long Term memory management.
3. GPT-3.5 powered Agents for delegation of simple tasks.
4. File output.

Performance Evaluation:
1. Continuously review and analyze your actions to ensure you are performing to the best of your abilities.
2. Constructively self-criticize your big-picture behavior constantly.
3. Reflect on past decisions and strategies to refine your approach.
4. Every command has a cost, so be smart and efficient. Aim to complete tasks in the least number of steps.

You should only respond in JSON format as described below
Response Format:
{
    "thoughts": {
        "text": "thought",
        "reasoning": "reasoning",
        "plan": "- short bulleted\n- list that conveys\n- long-term plan",
        "criticism": "constructive self-criticism",
        "speak": "thoughts summary to say to user"
    },
    "command": {
        "name": "command name",
        "args": {
            "arg name": "value"
        }
    }
}
Ensure the response can be parsed by Python json.loads
GPT-Engineer is another project to create a whole repository of code given a task specified in natural language. The GPT-Engineer is instructed to think over a list of smaller components to build and ask for user input to clarify questions as needed.
GPT-Engineer是另一个根据自然语言指定的任务创建完整代码仓库的项目。GPT-Engineer会被指示思考一系列需要构建的较小组件，并在必要时请求用户输入以澄清问题。

Here are a sample conversation for task clarification sent to OpenAI ChatCompletion endpoint used by GPT-Engineer. The user inputs are wrapped in {{user input text}}.
以下是GPT-Engineer使用的发送至OpenAI ChatCompletion端点的任务澄清示例对话。用户输入用{{user input text}}包裹。

[
  {
    "role": "system",
    "content": "You will read instructions and not carry them out, only seek to clarify them.\nSpecifically you will first summarise a list of super short bullets of areas that need clarification.\nThen you will pick one clarifying question, and wait for an answer from the user.\n"
  },
  {
    "role": "user",
    "content": "We are writing {{a Super Mario game in python. MVC components split in separate files. Keyboard control.}}\n"
  },
  {
    "role": "assistant",
    "content": "Summary of areas that need clarification:\n1. Specifics of the Super Mario game (e.g. level design, characters, gameplay mechanics)\n2. Details about the MVC components (e.g. which components are in each file)\n3. Keyboard control implementation (e.g. which keys to use, how to handle input)\n\nClarifying question:\nCan you provide more details about the Super Mario game, such as level design, characters, and gameplay mechanics?"
  },
  {
    "role": "user",
    "content": "{{There are 10 levels in total. The main character is a plumber named Mario, who can walk and jump. It is a classical platform game just like Super Mario. The main character moves from left to right, trying to get to the destination, where there are many obstacles and attacks from enemies in the process.}}\n\nIs anything else unclear? If yes, only answer in the form:\n{remaining unclear areas} remaining questions.\n{Next question}\nIf everything is sufficiently clear, only answer \"Nothing more to clarify.\"."
  },
  {
    "role": "assistant",
    "content": "Remaining unclear areas: 2 remaining questions.\nCan you provide more information about how the MVC components are split into separate files?"
  },
  {
    "role": "user",
    "content": "{{Make your own assumptions and state them explicitly before starting}}"
  }
]
Then after these clarification, the agent moved into the code writing mode with a different system message. System message:
在这些澄清之后，智能体切换到了代码编写模式，并使用了不同的系统消息。系统消息：

You will get instructions for code to write. You will write a very long answer. Make sure that every detail of the architecture is, in the end, implemented as code. Make sure that every detail of the architecture is, in the end, implemented as code.
你将会收到编写代码的指令。你要写出一个非常长的答案。确保架构的每一个细节最终都能以代码的形式实现。确保架构的每一个细节最终都能以代码的形式实现。
Think step by step and reason yourself to the right decisions to make sure we get it right. You will first lay out the names of the core classes, functions, methods that will be necessary, as well as a quick comment on their purpose.
一步一步思考，通过推理做出正确的决策，以确保我们把事情做好。首先，你需要列出必要的核心类、函数和方法的名称，并简要说明它们的用途。

Then you will output the content of each file including ALL code. Each file must strictly follow a markdown code block format, where the following tokens must be replaced such that FILENAME is the lowercase file name including the file extension, LANG is the markup code block language for the code’s language, and CODE is the code:
然后你要输出每个文件的内容，包括所有代码。每个文件必须严格遵循markdown代码块格式，其中以下标记必须替换，使得FILENAME是包含文件扩展名的小写文件名，LANG是代码语言的标记代码块语言，而CODE是代码：

FILENAME 文件名

CODE
You will start with the “entrypoint” file, then go to the ones that are imported by that file, and so on. Please note that the code should be fully functional. No placeholders.
你将从“入口点”文件开始，然后依次处理该文件导入的文件，依此类推。请注意，代码必须完全可运行，不能有占位符。

Follow a language and framework appropriate best practice file naming convention. Make sure that files contain all imports, types etc. Make sure that code in different files are compatible with each other. Ensure to implement all code, if you are unsure, write a plausible implementation. Include module dependency or package manager dependency definition file. Before you finish, double check that all parts of the architecture is present in the files.
遵循与语言和框架相适应的最佳实践文件命名规范。确保文件包含所有导入、类型等内容。确保不同文件中的代码相互兼容。务必实现所有代码，若有不确定之处，编写合理的实现方案。包含模块依赖或包管理器依赖定义文件。完成前，仔细检查所有架构部分是否都已体现在文件中。

Useful to know: You almost always put different classes in different files. For Python, you always create an appropriate requirements.txt file. For NodeJS, you always create an appropriate package.json file. You always add a comment briefly describing the purpose of the function definition. You try to add comments explaining very complex bits of logic. You always follow the best practices for the requested languages in terms of describing the code written as a defined package/project.
需要了解的有用信息：你几乎总是将不同的类放在不同的文件中。对于Python，你总是要创建一个合适的requirements.txt文件。对于NodeJS，你总是要创建一个合适的package.json文件。你总是要添加注释，简要描述函数定义的用途。你要尽量添加注释，解释非常复杂的逻辑部分。在描述作为已定义包/项目编写的代码方面，你要始终遵循所要求语言的最佳实践。

Python toolbelt preferences: Python工具库偏好：

pytest
dataclasses 数据类
Conversatin samples: 对话示例：

[
  {
    "role": "system",
    "content": "You will get instructions for code to write.\nYou will write a very long answer. Make sure that every detail of the architecture is, in the end, implemented as code.\nMake sure that every detail of the architecture is, in the end, implemented as code.\n\nThink step by step and reason yourself to the right decisions to make sure we get it right.\nYou will first lay out the names of the core classes, functions, methods that will be necessary, as well as a quick comment on their purpose.\n\nThen you will output the content of each file including ALL code.\nEach file must strictly follow a markdown code block format, where the following tokens must be replaced such that\nFILENAME is the lowercase file name including the file extension,\nLANG is the markup code block language for the code's language, and CODE is the code:\n\nFILENAME\n```LANG\nCODE\n```\n\nYou will start with the \"entrypoint\" file, then go to the ones that are imported by that file, and so on.\nPlease note that the code should be fully functional. No placeholders.\n\nFollow a language and framework appropriate best practice file naming convention.\nMake sure that files contain all imports, types etc. Make sure that code in different files are compatible with each other.\nEnsure to implement all code, if you are unsure, write a plausible implementation.\nInclude module dependency or package manager dependency definition file.\nBefore you finish, double check that all parts of the architecture is present in the files.\n\nUseful to know:\nYou almost always put different classes in different files.\nFor Python, you always create an appropriate requirements.txt file.\nFor NodeJS, you always create an appropriate package.json file.\nYou always add a comment briefly describing the purpose of the function definition.\nYou try to add comments explaining very complex bits of logic.\nYou always follow the best practices for the requested languages in terms of describing the code written as a defined\npackage/project.\n\n\nPython toolbelt preferences:\n- pytest\n- dataclasses\n"
  },
 #  … same conversation as earlier, ended with "Make your own assumptions and state them explicitly before starting".
  {
    "role": "assistant",
    "content": "Assumptions:\n1. Model: The model will contain the game's data, such as level information, character states, and enemy positions.\n2. View: The view will handle the game's visuals, including rendering the game objects, backgrounds, and updating the display.\n3. Controller: The controller will manage user input, such as keyboard controls, and update the model accordingly.\n\nWith these assumptions in mind, is there any specific aspect of the keyboard control implementation you would like me to clarify?"
  },
  {
    "role": "user",
    "content": "Please now remember the steps:\n\nThink step by step and reason yourself to the right decisions to make sure we get it right.\nFirst lay out the names of the core classes, functions, methods that will be necessary, As well as a quick comment on their purpose.\n\nThen you will output the content of each file including ALL code.\nEach file must strictly follow a markdown code block format, where the following tokens must be replaced such that\nFILENAME is the lowercase file name including the file extension,\nLANG is the markup code block language for the code's language, and CODE is the code:\n\nFILENAME\n```LANG\nCODE\n```\n\nPlease note that the code should be fully functional. No placeholders.\n\nYou will start with the \"entrypoint\" file, then go to the ones that are imported by that file, and so on.\nFollow a language and framework appropriate best practice file naming convention.\nMake sure that files contain all imports, types etc. The code should be fully functional. Make sure that code in different files are compatible with each other.\nBefore you finish, double check that all parts of the architecture is present in the files.\n"
  }
]
Challenges 挑战#
After going through key ideas and demos of building LLM-centered agents, I start to see a couple common limitations:
在了解了以大型语言模型为核心构建智能体的关键理念和演示后，我开始发现一些常见的局限性：

Finite context length: The restricted context capacity limits the inclusion of historical information, detailed instructions, API call context, and responses. The design of the system has to work with this limited communication bandwidth, while mechanisms like self-reflection to learn from past mistakes would benefit a lot from long or infinite context windows. Although vector stores and retrieval can provide access to a larger knowledge pool, their representation power is not as powerful as full attention.
有限的上下文长度：受限的上下文容量限制了历史信息、详细指令、API调用上下文和响应的纳入。系统设计必须在这种有限的通信带宽下运行，而像自我反思这样能从过去错误中学习的机制，将从长或无限的上下文窗口中受益匪浅。尽管向量存储和检索可以提供对更大知识池的访问，但它们的表示能力不如全注意力强大。

Challenges in long-term planning and task decomposition: Planning over a lengthy history and effectively exploring the solution space remain challenging. LLMs struggle to adjust plans when faced with unexpected errors, making them less robust compared to humans who learn from trial and error.
长期规划和任务分解中的挑战：基于漫长历史进行规划并有效探索解决方案空间仍然具有挑战性。面对意外错误时，大型语言模型难以调整计划，这使得它们与能从试错中学习的人类相比，稳健性更差。

Reliability of natural language interface: Current agent system relies on natural language as an interface between LLMs and external components such as memory and tools. However, the reliability of model outputs is questionable, as LLMs may make formatting errors and occasionally exhibit rebellious behavior (e.g. refuse to follow an instruction). Consequently, much of the agent demo code focuses on parsing model output.
自然语言界面的可靠性：当前的智能体系统依赖自然语言作为大语言模型与外部组件（如内存和工具）之间的界面。然而，模型输出的可靠性存在疑问，因为大语言模型可能会出现格式错误，偶尔还会表现出抗拒行为（例如拒绝遵循指令）。因此，许多智能体演示代码都侧重于解析模型输出。

Citation 引用#
Cited as: 引用格式：

Weng, Lilian. (Jun 2023). “LLM-powered Autonomous Agents”. Lil’Log. https://lilianweng.github.io/posts/2023-06-23-agent/.
翁莉莲。（2023年6月）。《大语言模型驱动的自主智能体》。Lil’Log。https://lilianweng.github.io/posts/2023-06-23-agent/。

Or 或

@article{weng2023agent,
  title   = "LLM-powered Autonomous Agents",
  author  = "Weng, Lilian",
  journal = "lilianweng.github.io",
  year    = "2023",
  month   = "Jun",
  url     = "https://lilianweng.github.io/posts/2023-06-23-agent/"
}
References 参考文献#
[1] Wei et al. “Chain of thought prompting elicits reasoning in large language models.” NeurIPS 2022
[1] Wei 等人。“思维链提示引发大型语言模型的推理。”《神经信息处理系统进展》2022年

[2] Yao et al. “Tree of Thoughts: Dliberate Problem Solving with Large Language Models.” arXiv preprint arXiv:2305.10601 (2023).
[2] Yao等人。“思维树：利用大型语言模型进行审慎的问题解决。” arXiv预印本arXiv:2305.10601（2023）。

[3] Liu et al. “Chain of Hindsight Aligns Language Models with Feedback “ arXiv preprint arXiv:2302.02676 (2023).
[3] 刘等人 “事后链使语言模型与反馈对齐” arXiv预印本 arXiv:2302.02676（2023）。

[4] Liu et al. “LLM+P: Empowering Large Language Models with Optimal Planning Proficiency” arXiv preprint arXiv:2304.11477 (2023).
[4] 刘等人 “LLM+P：赋予大型语言模型最优规划能力” arXiv预印本 arXiv:2304.11477（2023）。

[5] Yao et al. “ReAct: Synergizing reasoning and acting in language models.” ICLR 2023.
[5] Yao等人。“ReAct：在语言模型中协同推理与行动。”国际学习表征会议，2023年。

[6] Google Blog. “Announcing ScaNN: Efficient Vector Similarity Search” July 28, 2020.
[6] 谷歌博客。“宣布推出ScaNN：高效向量相似性搜索” 2020年7月28日。

[7] https://chat.openai.com/share/46ff149e-a4c7-4dd7-a800-fc4a642ea389
[7]https://chat.openai.com/share/46ff149e-a4c7-4dd7-a800-fc4a642ea389

[8] Shinn & Labash. “Reflexion: an autonomous agent with dynamic memory and self-reflection” arXiv preprint arXiv:2303.11366 (2023).
[8] Shinn 与 Labash。《Reflexion：一个具有动态记忆和自我反思能力的自主智能体》 arXiv 预印本 arXiv:2303.11366（2023）。

[9] Laskin et al. “In-context Reinforcement Learning with Algorithm Distillation” ICLR 2023.
[9] Laskin 等人。“基于算法蒸馏的上下文强化学习”，国际学习表征会议，2023年。

[10] Karpas et al. “MRKL Systems A modular, neuro-symbolic architecture that combines large language models, external knowledge sources and discrete reasoning.” arXiv preprint arXiv:2205.00445 (2022).
[10] Karpas等人。“MRKL系统：一种模块化的神经符号架构，结合了大型语言模型、外部知识源和离散推理。” arXiv预印本arXiv:2205.00445（2022）。

[11] Nakano et al. “Webgpt: Browser-assisted question-answering with human feedback.” arXiv preprint arXiv:2112.09332 (2021).
[11] Nakano等人。“WebGPT：借助浏览器和人类反馈的问答系统。” arXiv预印本arXiv:2112.09332（2021）。

[12] Parisi et al. “TALM: Tool Augmented Language Models”
[12] Parisi 等人。“TALM：工具增强语言模型”

[13] Schick et al. “Toolformer: Language Models Can Teach Themselves to Use Tools.” arXiv preprint arXiv:2302.04761 (2023).
[13] Schick 等人。“Toolformer：语言模型可以自学使用工具。” arXiv 预印本 arXiv:2302.04761（2023）。

[14] Weaviate Blog. Why is Vector Search so fast? Sep 13, 2022.
[14] Weaviate博客。为什么向量搜索如此之快？ 2022年9月13日。

[15] Li et al. “API-Bank: A Benchmark for Tool-Augmented LLMs” arXiv preprint arXiv:2304.08244 (2023).
[15] 李等人。“API-Bank：工具增强型大语言模型的基准测试” arXiv预印本arXiv:2304.08244（2023）。

[16] Shen et al. “HuggingGPT: Solving AI Tasks with ChatGPT and its Friends in HuggingFace” arXiv preprint arXiv:2303.17580 (2023).
[16] Shen等人。“HuggingGPT：借助ChatGPT及其在HuggingFace中的伙伴解决AI任务” arXiv预印本arXiv:2303.17580（2023）。

[17] Bran et al. “ChemCrow: Augmenting large-language models with chemistry tools.” arXiv preprint arXiv:2304.05376 (2023).
[17] Bran等人。“ChemCrow：用化学工具增强大型语言模型。” arXiv预印本arXiv:2304.05376（2023）。

[18] Boiko et al. “Emergent autonomous scientific research capabilities of large language models.” arXiv preprint arXiv:2304.05332 (2023).
[18] Boiko 等人。“大型语言模型的新兴自主科学研究能力。” arXiv 预印本 arXiv:2304.05332（2023）。

[19] Joon Sung Park, et al. “Generative Agents: Interactive Simulacra of Human Behavior.” arXiv preprint arXiv:2304.03442 (2023).
[19] 朴俊成（Joon Sung Park）等人。“生成式智能体：人类行为的交互式模拟。” arXiv预印本 arXiv:2304.03442（2023）。

[20] AutoGPT. https://github.com/Significant-Gravitas/Auto-GPT
[20]自动GPT。https://github.com/Significant-Gravitas/Auto-GPT

[21] GPT-Engineer. https://github.com/AntonOsika/gpt-engineer
[21] GPT-Engineer. https://github.com/AntonOsika/gpt-engineer
