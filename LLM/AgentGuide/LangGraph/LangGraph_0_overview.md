# LangGraph overview
Trusted by companies shaping the future of agents— including Klarna, Replit, Elastic, and more— LangGraph is a low-level orchestration framework and runtime for building, managing, and deploying long-running, stateful agents.
> LangGraph 是一个底层编排框架和运行时。用于构建、管理和部署长期运行的有状态智能体。受到那些塑造智能体未来的公司的信任，其中包括 Klarna、Replit、Elastic 等。

LangGraph is very low-level, and focused entirely on agent **orchestration**. Before using LangGraph, we recommend you familiarize yourself with some of the components used to build agents, starting with **models** and **tools**.
> LangGraph非常底层，并且完全专注于智能体的编排。在使用LangGraph之前，我们建议您先熟悉一些用于构建智能体的组件，从模型和工具开始。

We will commonly use LangChain components throughout the documentation to integrate models and tools, but you don’t need to use LangChain to use LangGraph. If you are just getting started with agents or want a higher-level abstraction, we recommend you use LangChain’s agents that provide prebuilt architectures for common LLM and tool-calling loops.
> 在整个文档中，我们通常会使用LangChain组件来集成模型和工具，但使用LangGraph并不一定需要用到LangChain。如果您刚刚开始接触智能体，或者希望使用更高层次的抽象，我们建议您使用LangChain的智能体，它们为常见的大语言模型和工具调用循环提供了预制架构。

LangGraph is focused on the underlying capabilities important for agent orchestration: durable execution, streaming, human-in-the-loop, and more.
> LangGraph专注于智能体编排所需的核心底层能力：持久执行、流处理、人机协同等。​

## Install
### pip
``` bash
pip install -U langgraph
```

### uv
``` bash
uv add langgraph
```

Then, create a simple hello world example:
``` python
from langgraph.graph import StateGraph, MessagesState, START, END

def mock_llm(state: MessagesState):
    return {"messages": [{"role": "ai", "content": "hello world"}]}

graph = StateGraph(MessagesState)
graph.add_node(mock_llm)
graph.add_edge(START, "mock_llm")
graph.add_edge("mock_llm", END)
graph = graph.compile()

graph.invoke({"messages": [{"role": "user", "content": "hi!"}]})
```

Use LangSmith to trace requests, debug agent behavior, and evaluate outputs. Set LANGSMITH_TRACING=true and your API key to get started.
> 使用LangSmith追踪请求、调试智能体行为并评估输出。设置LANGSMITH_TRACING=true并配置您的API密钥即可开始使用。
​
## Core benefits
LangGraph provides low-level supporting infrastructure for any long-running, stateful workflow or agent. LangGraph does not abstract prompts or architecture, and provides the following central benefits:
> LangGraph 为任何长期运行的、有状态的工作流或智能体提供底层支持架构。LangGraph 不抽象提示词或架构，它具有以下核心优势：

- **Durable execution**: Build agents that persist through failures and can run for extended periods, resuming from where they left off.
> 持久执行：构建能够在故障中保持运行且可以长时间运行的智能体，它们能够从停止的地方恢复运行。

- **Human-in-the-loop**: Incorporate human oversight by inspecting and modifying agent state at any point.
> 人机协同：通过在任何时候检查和修改智能体状态，纳入人类监督。

- **Comprehensive memory**: Create stateful agents with both short-term working memory for ongoing reasoning and long-term memory across sessions.
> 全面的记忆能力：创建具有状态的智能体，既具备用于持续推理的短期工作记忆，又拥有跨会话的长期记忆。

- **Debugging with LangSmith**: Gain deep visibility into complex agent behavior with visualization tools that trace execution paths, capture state transitions, and provide detailed runtime metrics.
> 使用LangSmith进行调试：借助可视化工具深入了解复杂的智能体行为，这些工具可追踪执行路径、捕获状态转换并提供详细的运行时指标。

- **Production-ready deployment**: Deploy sophisticated agent systems confidently with scalable infrastructure designed to handle the unique challenges of stateful, long-running workflows.
> 可投入生产的部署：借助可扩展的基础设施，自信地部署复杂的智能体系统，这些基础设施旨在应对有状态、长时间运行的工作流所带来的独特挑战。
​
## LangGraph ecosystem
While LangGraph can be used standalone, it also integrates seamlessly with any LangChain product, giving developers a full suite of tools for building agents. To improve your LLM application development, pair LangGraph with:
> 虽然LangGraph可以单独使用，但它也能与任何LangChain产品无缝集成，为开发者提供一整套用于构建智能体的工具。为了改进你的大语言模型应用开发，可以将LangGraph与以下工具结合使用：

### LangSmith Observability
Trace requests, evaluate outputs, and monitor deployments in one place. Prototype locally with LangGraph, then move to production with integrated observability and evaluation to build more reliable agent systems.
> 在一个地方追踪请求、评估输出并监控部署。使用LangGraph在本地制作原型，然后借助集成的可观测性和评估功能投入生产，以构建更可靠的智能体系统。

### LangSmith Deployment
Deploy and scale agents effortlessly with a purpose-built deployment platform for long running, stateful workflows. Discover, reuse, configure, and share agents across teams — and iterate quickly with visual prototyping in Studio.
> 借助专为长时间运行的有状态工作流打造的部署平台，轻松部署和扩展智能体。在团队间发现、重用、配置和共享智能体，并通过Studio中的可视化原型设计快速迭代。

### LangChain
Provides integrations and composable components to streamline LLM application development. Contains agent abstractions built on top of LangGraph.
> 提供集成和可组合组件，以简化大语言模型应用程序的开发。包含基于LangGraph构建的智能体抽象。
​
## Acknowledgements
LangGraph is inspired by Pregel and Apache Beam. The public interface draws inspiration from NetworkX. LangGraph is built by LangChain Inc, the creators of LangChain, but can be used without LangChain.
> LangGraph 的灵感来源于 Pregel 和 Apache Beam。其公共接口的设计借鉴了 NetworkX。LangGraph 由 LangChain 的创建者 LangChain 公司开发，但也可在不使用 LangChain 的情况下独立使用。
