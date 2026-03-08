LangChain is an open source framework with a pre-built agent architecture and integrations for any model or tool — so you can build agents that adapt as fast as the ecosystem evolves
> LangChain 是一个开源框架，它拥有预构建的智能体架构，并且能够与任何模型或工具集成，因此你可以构建出能像生态系统一样快速演进的智能体。

LangChain is the easy way to start building completely custom agents and applications powered by LLMs. With under 10 lines of code, you can connect to OpenAI, Anthropic, Google, and more. LangChain provides a pre-built agent architecture and model integrations to help you get started quickly and seamlessly incorporate LLMs into your agents and applications.
> LangChain是轻松开始构建由大语言模型驱动的完全自定义智能体和应用程序的方法。只需不到10行代码，你就可以连接到OpenAI、Anthropic、谷歌以及更多平台。LangChain提供了预构建的智能体架构和模型集成，帮助你快速入门，并将大语言模型无缝整合到你的智能体和应用程序中。

**更多平台**
https://docs.langchain.com/oss/python/integrations/providers/overview

### LangChain vs. LangGraph vs. Deep Agents

If you are looking to build an agent, we recommend you start with Deep Agents which comes “batteries-included”, with modern features like automatic compression of long conversations, a virtual filesystem, and subagent-spawning for managing and isolating context.
> 如果你想构建一个智能体，我们建议你从深度智能体开始，它“内置了各种功能”，具备现代特性，例如自动压缩长对话、虚拟文件系统以及用于管理和隔离上下文的子智能体生成功能。

Deep Agents are implementations of LangChain agents. If you don’t need these capabilities or would like to customize your own for your agents and autonomous applications, start with LangChain.
> 深度智能体是LangChain智能体的实现。如果您不需要这些功能，或者希望为您的智能体和自主应用定制自己的功能，请从LangChain开始。
Use LangGraph, our low-level agent orchestration framework and runtime, when you have more advanced needs that require a combination of deterministic and agentic workflows and heavy customization.
> 当您有更高级的需求，需要结合确定性工作流和智能体工作流，并进行大量定制时，请使用LangGraph——我们的低级智能体编排框架和运行时。

LangChain agents are built on top of LangGraph in order to provide durable execution, streaming, human-in-the-loop, persistence, and more. You do not need to know LangGraph for basic LangChain agent usage.
LangChain 智能体 构建在 LangGraph 之上，以提供持久执行、流式传输、人机协作、持久性等功能。对于 LangChain 智能体的基本使用，您无需了解 LangGraph。

We recommend you use LangChain if you want to quickly build agents and autonomous applications.
如果您想快速构建智能体和自主应用，我们建议您使用LangChain。
​
### Create an agent 创建智能体

``` python
# pip install -qU langchain "langchain[anthropic]"
from langchain.agents import create_agent

def get_weather(city: str) -> str:
    """Get weather for a given city."""
    return f"It's always sunny in {city}!"

agent = create_agent(
    model="claude-sonnet-4-6",
    tools=[get_weather],
    system_prompt="You are a helpful assistant",
)

# Run the agent
agent.invoke(
    {"messages": [{"role": "user", "content": "what is the weather in sf"}]}
)
```

### pip install -qU langchain "langchain[anthropic]"
在 Python 环境中安装 / 升级 LangChain 框架，以及 LangChain 对 Anthropic（克劳德）大模型的扩展支持

PyPI（Python Package Index）是 Python 包的官方仓库，所有包的可选依赖都会在 PyPI 页面明确标注。

操作步骤：

打开 PyPI 官网：https://pypi.org/

搜索目标包（如「langchain」），进入包的详情页（https://pypi.org/project/langchain/）；

找到「Project details」或「Dependencies」区域，查看「Extra Requirements」（额外依赖）部分。

See the Installation instructions and Quickstart guide to get started building your own agents and applications with LangChain.
> 请参阅安装说明和快速入门指南，开始使用LangChain构建您自己的智能体和应用程序。

Use LangSmith to trace requests, debug agent behavior, and evaluate outputs. Set LANGSMITH_TRACING=true and your API key to get started.
> 使用LangSmith来追踪请求、调试智能体行为并评估输出。设置LANGSMITH_TRACING=true并配置您的API密钥即可开始使用。
​
### Core benefits 核心优势
Standard model interface 标准模型接口
Different providers have unique APIs for interacting with models, including the format of responses. LangChain standardizes how you interact with models so that you can seamlessly swap providers and avoid lock-in.
不同的提供商拥有独特的API用于与模型交互，包括响应格式。LangChain标准化了与模型的交互方式，使你能够无缝切换提供商，避免被锁定。
Learn more
了解更多
Easy to use, highly flexible agent
易于使用、高度灵活的智能体
LangChain’s agent abstraction is designed to be easy to get started with, letting you build a simple agent in under 10 lines of code. But it also provides enough flexibility to allow you to do all the context engineering your heart desires.
LangChain的智能体抽象设计旨在让用户易于上手，只需不到10行代码就能构建一个简单的智能体。但它也提供了足够的灵活性，让你能够随心所欲地进行各种上下文设计。
Learn more
了解更多

Built on top of LangGraph 基于LangGraph构建
LangChain’s agents are built on top of LangGraph. This allows us to take advantage of LangGraph’s durable execution, human-in-the-loop support, persistence, and more.
LangChain的智能体构建在LangGraph之上。这使我们能够利用LangGraph的持久执行、人机协同支持、持久性等功能。
Learn more
了解更多

Debug with LangSmith 使用LangSmith进行调试
Gain deep visibility into complex agent behavior with visualization tools that trace execution paths, capture state transitions, and provide detailed runtime metrics.
借助可视化工具深入了解复杂的智能体行为，这些工具可追踪执行路径、捕捉状态转换并提供详细的运行时指标。
Learn more
了解更多
