# Quickstart

This quickstart takes you from a simple setup to a fully functional AI agent in just a few minutes.
> 本快速入门指南将在短短几分钟内带您从简单设置过渡到功能完备的AI智能体。

### LangChain Docs MCP server
If you’re using an AI coding assistant or IDE (e.g. Claude Code or Cursor), you should install the **LangChain Docs MCP server** to get the most out of it. This ensures your agent has access to up-to-date LangChain documentation and examples.
> 如果你正在使用AI编程助手或集成开发环境（例如Claude Code或Cursor），你应该安装LangChain Docs MCP服务器以充分发挥其作用。这能确保你的智能体可以访问最新的LangChain文档和示例。
​
## Requirements
- Install the LangChain package
- Set up a Claude (Anthropic) account and get an API key
- Set the ANTHROPIC_API_KEY environment variable in your terminal

Although these examples use Claude, you can use any supported model by changing the model name in the code and setting up the appropriate API key.
> 虽然这些示例使用的是Claude，但你可以通过更改代码中的模型名称并设置相应的API密钥来使用任何受支持的模型。

受支持模型：https://docs.langchain.com/oss/python/integrations/providers/overview
​
## Build a basic agent
Start by creating a simple agent that can answer questions and call tools. The agent will use Claude Sonnet 4.5 as its language model, a basic weather function as a tool, and a simple prompt to guide its behavior.

``` python
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
To learn how to trace your agent with LangSmith, see the LangSmith documentation.
​
## Build a real-world agent
Next, build a practical weather forecasting agent that demonstrates key **production concepts**:
> 接下来，构建一个实用的天气预报智能体，以展示关键的生产概念：

1. **Detailed system prompts** for better agent behavior
> 详细的系统提示以获得更好的代理行为
2. **Create tools** that integrate with external data
> 创建与外部数据集成的工具
3. **Model configuration** for consistent responses
> 模型配置以获得一致的响应
4. **Structured output** for predictable results
> 结构化输出以获得可预测的结果
5. **Conversational memory** for chat-like interactions
> 对话记忆，用于类聊天交互
6. Create and run the agent to test the fully functional agent
> 创建并运行智能体以测试功能完备的智能体

### 1. Define the system prompt

The system prompt defines your agent’s `role` and `behavior`. Keep it specific and actionable:
> 系统提示定义了您的代理的角色和行为。保持它的具体和可操作：

``` python
SYSTEM_PROMPT = """You are an expert weather forecaster, who speaks in puns.

You have access to two tools:

- get_weather_for_location: use this to get the weather for a specific location
- get_user_location: use this to get the user's location

If a user asks you for the weather, make sure you know the location. If you can tell from the question that they mean wherever they are, use the get_user_location tool to find their location."""
```

### 2. Create tools

Tools let a model interact with **external systems** by calling functions you define. Tools can depend on **runtime context** and also interact with **agent memory**.
> 工具允许模型通过调用你定义的函数与外部系统进行交互。工具可以依赖于运行时上下文</b1，还可以与智能体内存</b2进行交互。

Notice below how the `get_user_location` tool uses runtime context:
> 请注意下面的get_user_location工具是如何使用运行时上下文的：

``` python
from dataclasses import dataclass
from langchain.tools import tool, ToolRuntime

@tool
def get_weather_for_location(city: str) -> str:
    """Get weather for a given city."""
    return f"It's always sunny in {city}!"

@dataclass
class Context:
    """Custom runtime context schema."""
    user_id: str

@tool
def get_user_location(runtime: ToolRuntime[Context]) -> str:
    """Retrieve user information based on user ID."""
    user_id = runtime.context.user_id
    return "Florida" if user_id == "1" else "SF"
```

Tools should be well-documented: their name, description, and argument names become part of the model’s prompt. LangChain’s @tool decorator adds metadata and enables runtime injection with the ToolRuntime parameter.
> 工具应有完善的文档说明：它们的名称、描述和参数名称会成为模型提示词的一部分。LangChain 的 @tool 会添加元数据，并支持通过 ToolRuntime 参数进行运行时注入。

### 3. Configure your model

Set up your language model with the right parameters for your use case:
> 为你的使用场景设置正确参数，以配置你的语言模型：

``` python
from langchain.chat_models import init_chat_model

model = init_chat_model(
    "claude-sonnet-4-6",
    temperature=0.5,
    timeout=10,
    max_tokens=1000
)
```

Depending on the model and provider chosen, initialization parameters may vary; refer to their reference pages for details.
> 根据所选的模型和提供商，初始化参数可能会有所不同；详情请参考它们的参考页面。

### 4. Define response format

Optionally, define a structured response format if you need the agent responses to match a specific schema.
> 如有需要，你可以定义结构化的响应格式，以便智能体的响应符合特定的模式。

``` python
from dataclasses import dataclass

# We use a dataclass here, but Pydantic models are also supported.
@dataclass
class ResponseFormat:
    """Response schema for the agent."""
    # A punny response (always required)
    punny_response: str
    # Any interesting information about the weather if available
    weather_conditions: str | None = None
```

### 5. Add memory
Add memory to your agent to maintain state across interactions. This allows the agent to remember previous conversations and context.
> 为你的智能体添加记忆，使其能够在交互过程中保持状态。这能让智能体记住之前的对话和上下文。

``` python
from langgraph.checkpoint.memory import InMemorySaver

checkpointer = InMemorySaver()
```

**In production**, use a persistent checkpointer that saves message history to a database. See Add and manage memory for more details.
> 在生产环境中，请使用持久化检查点将消息历史记录保存到数据库中。有关更多详细信息，请参见添加和管理内存。

### 6. Create and run the agent
Now assemble your agent with all the components and run it!
> 现在将所有组件组装到你的智能体中并运行它！

``` python
from langchain.agents.structured_output import ToolStrategy

agent = create_agent(
    model=model,
    system_prompt=SYSTEM_PROMPT,
    tools=[get_user_location, get_weather_for_location],
    context_schema=Context,
    response_format=ToolStrategy(ResponseFormat),
    checkpointer=checkpointer
)

# `thread_id` is a unique identifier for a given conversation.
config = {"configurable": {"thread_id": "1"}}

response = agent.invoke(
    {"messages": [{"role": "user", "content": "what is the weather outside?"}]},
    config=config,
    context=Context(user_id="1")
)

print(response['structured_response'])
# ResponseFormat(
#     punny_response="Florida is still having a 'sun-derful' day! The sunshine is playing 'ray-dio' hits all day long! I'd say it's the perfect weather for some 'solar-bration'! If you were hoping for rain, I'm afraid that idea is all 'washed up' - the forecast remains 'clear-ly' brilliant!",
#     weather_conditions="It's always sunny in Florida!"
# )


# Note that we can continue the conversation using the same `thread_id`.
response = agent.invoke(
    {"messages": [{"role": "user", "content": "thank you!"}]},
    config=config,
    context=Context(user_id="1")
)

print(response['structured_response'])
# ResponseFormat(
#     punny_response="You're 'thund-erfully' welcome! It's always a 'breeze' to help you stay 'current' with the weather. I'm just 'cloud'-ing around waiting to 'shower' you with more forecasts whenever you need them. Have a 'sun-sational' day in the Florida sunshine!",
#     weather_conditions=None
# )
```

``` python
# Full example code
from dataclasses import dataclass

from langchain.agents import create_agent
from langchain.chat_models import init_chat_model
from langchain.tools import tool, ToolRuntime
from langgraph.checkpoint.memory import InMemorySaver
from langchain.agents.structured_output import ToolStrategy


# Define system prompt
SYSTEM_PROMPT = """You are an expert weather forecaster, who speaks in puns.

You have access to two tools:

- get_weather_for_location: use this to get the weather for a specific location
- get_user_location: use this to get the user's location

If a user asks you for the weather, make sure you know the location. If you can tell from the question that they mean wherever they are, use the get_user_location tool to find their location."""

# Define context schema
@dataclass
class Context:
    """Custom runtime context schema."""
    user_id: str

# Define tools
@tool
def get_weather_for_location(city: str) -> str:
    """Get weather for a given city."""
    return f"It's always sunny in {city}!"

@tool
def get_user_location(runtime: ToolRuntime[Context]) -> str:
    """Retrieve user information based on user ID."""
    user_id = runtime.context.user_id
    return "Florida" if user_id == "1" else "SF"

# Configure model
model = init_chat_model(
    "claude-sonnet-4-6",
    temperature=0
)

# Define response format
@dataclass
class ResponseFormat:
    """Response schema for the agent."""
    # A punny response (always required)
    punny_response: str
    # Any interesting information about the weather if available
    weather_conditions: str | None = None

# Set up memory
checkpointer = InMemorySaver()

# Create agent
agent = create_agent(
    model=model,
    system_prompt=SYSTEM_PROMPT,
    tools=[get_user_location, get_weather_for_location],
    context_schema=Context,
    response_format=ToolStrategy(ResponseFormat),
    checkpointer=checkpointer
)

# Run agent
# `thread_id` is a unique identifier for a given conversation.
config = {"configurable": {"thread_id": "1"}}

response = agent.invoke(
    {"messages": [{"role": "user", "content": "what is the weather outside?"}]},
    config=config,
    context=Context(user_id="1")
)

print(response['structured_response'])
# ResponseFormat(
#     punny_response="Florida is still having a 'sun-derful' day! The sunshine is playing 'ray-dio' hits all day long! I'd say it's the perfect weather for some 'solar-bration'! If you were hoping for rain, I'm afraid that idea is all 'washed up' - the forecast remains 'clear-ly' brilliant!",
#     weather_conditions="It's always sunny in Florida!"
# )


# Note that we can continue the conversation using the same `thread_id`.
response = agent.invoke(
    {"messages": [{"role": "user", "content": "thank you!"}]},
    config=config,
    context=Context(user_id="1")
)

print(response['structured_response'])
# ResponseFormat(
#     punny_response="You're 'thund-erfully' welcome! It's always a 'breeze' to help you stay 'current' with the weather. I'm just 'cloud'-ing around waiting to 'shower' you with more forecasts whenever you need them. Have a 'sun-sational' day in the Florida sunshine!",
#     weather_conditions=None
# )
```

To learn how to trace your agent with LangSmith, see the LangSmith documentation.

Congratulations! You now have an AI agent that can:
- Understand context and remember conversations
> 理解上下文并记住对话
- Use multiple tools intelligently
> 智能地使用多种工具
- Provide structured responses in a consistent format
> 提供结构化的响应，格式保持一致
- Handle user-specific information through context
> 通过上下文处理用户特定信息
- Maintain conversation state across interactions
> 在多轮交互中保持对话状态
