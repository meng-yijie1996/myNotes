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
Let’s walk through each step:
1
Define the system prompt

The system prompt defines your agent’s role and behavior. Keep it specific and actionable:
SYSTEM_PROMPT = """You are an expert weather forecaster, who speaks in puns.

You have access to two tools:

- get_weather_for_location: use this to get the weather for a specific location
- get_user_location: use this to get the user's location

If a user asks you for the weather, make sure you know the location. If you can tell from the question that they mean wherever they are, use the get_user_location tool to find their location."""
2
Create tools

Tools let a model interact with external systems by calling functions you define. Tools can depend on runtime context and also interact with agent memory.
Notice below how the get_user_location tool uses runtime context:
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
Tools should be well-documented: their name, description, and argument names become part of the model’s prompt. LangChain’s @tool decorator adds metadata and enables runtime injection with the ToolRuntime parameter.
3
Configure your model

Set up your language model with the right parameters for your use case:
from langchain.chat_models import init_chat_model

model = init_chat_model(
    "claude-sonnet-4-6",
    temperature=0.5,
    timeout=10,
    max_tokens=1000
)
Depending on the model and provider chosen, initialization parameters may vary; refer to their reference pages for details.
4
Define response format

Optionally, define a structured response format if you need the agent responses to match a specific schema.
from dataclasses import dataclass

# We use a dataclass here, but Pydantic models are also supported.
@dataclass
class ResponseFormat:
    """Response schema for the agent."""
    # A punny response (always required)
    punny_response: str
    # Any interesting information about the weather if available
    weather_conditions: str | None = None
5
Add memory

Add memory to your agent to maintain state across interactions. This allows the agent to remember previous conversations and context.
from langgraph.checkpoint.memory import InMemorySaver

checkpointer = InMemorySaver()
In production, use a persistent checkpointer that saves message history to a database. See Add and manage memory for more details.
6
Create and run the agent

Now assemble your agent with all the components and run it!
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
Show Full example code

To learn how to trace your agent with LangSmith, see the LangSmith documentation.
Congratulations! You now have an AI agent that can:
Understand context and remember conversations
Use multiple tools intelligently
Provide structured responses in a consistent format
Handle user-specific information through context
Maintain conversation state across interactions
