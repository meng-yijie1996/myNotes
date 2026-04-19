# Short-term memory

## Overview
Memory is a system that remembers information about previous interactions. For AI agents, memory is crucial because it lets them remember previous interactions, learn from feedback, and adapt to user preferences. As agents tackle more complex tasks with numerous user interactions, this capability becomes essential for both efficiency and user satisfaction.
记忆是一个存储过往交互信息的系统。对于人工智能智能体而言，记忆至关重要，因为它能让智能体记住过往的交互、从反馈中学习并适应用户的偏好。随着智能体处理包含大量用户交互的复杂任务时，这一能力对提升效率和用户满意度都变得不可或缺。
Short term memory lets your application remember previous interactions within a single thread or conversation.
短期记忆让你的应用程序能够记住单个线程或对话中的先前交互。
A thread organizes multiple interactions in a session, similar to the way email groups messages in a single conversation.
线程会将会话中的多次互动整合在一起，其组织方式类似于电子邮件将同一场对话中的消息归为一类。
Conversation history is the most common form of short-term memory. Long conversations pose a challenge to today’s LLMs; a full history may not fit inside an LLM’s context window, resulting in an context loss or errors.
对话历史是最常见的短期记忆形式。长对话对当前的大语言模型构成挑战；完整的对话历史可能无法放入大语言模型的上下文窗口，从而导致上下文丢失或出现错误。
Even if your model supports the full context length, most LLMs still perform poorly over long contexts. They get “distracted” by stale or off-topic content, all while suffering from slower response times and higher costs.
即使你的模型支持完整的上下文长度，大多数大语言模型在长上下文场景下的表现依然很差。它们会被陈旧或偏离主题的内容“干扰”，同时还会面临响应速度变慢、成本更高的问题。
Chat models accept context using messages, which include instructions (a system message) and inputs (human messages). In chat applications, messages alternate between human inputs and model responses, resulting in a list of messages that grows longer over time. Because context windows are limited, many applications can benefit from using techniques to remove or “forget” stale information.
聊天模型通过消息来接收上下文，消息包含指令（系统消息）和输入内容（人类消息）。在聊天应用中，消息会在人类输入和模型回复之间交替，形成一个随时间不断变长的消息列表。由于上下文窗口存在限制，许多应用都能从使用技术手段来清除或“遗忘”过时信息中获益。
Need to remember information across conversations? Use long-term memory to store and recall user-specific or application-level data across different threads and sessions.
需要记住跨对话的信息？使用长期记忆在不同的对话线程和会话中存储和召回用户专属或应用层面的数据。
​
Usage 使用方法
To add short-term memory (thread-level persistence) to an agent, you need to specify a checkpointer when creating an agent.
要为智能体添加短期记忆（线程级持久化），你需要在创建智能体时指定一个checkpointer。
LangChain’s agent manages short-term memory as a part of your agent’s state.
LangChain 的智能体将短期记忆作为自身状态的一部分进行管理。
By storing these in the graph’s state, the agent can access the full context for a given conversation while maintaining separation between different threads.
通过将这些信息存储在图的状态中，智能体可以在保持不同对话线程相互独立的同时，访问给定对话的完整上下文。
State is persisted to a database (or memory) using a checkpointer so the thread can be resumed at any time.
通过检查点机制将状态持久化到数据库（或内存）中，以便线程可随时恢复执行。
Short-term memory updates when the agent is invoked or a step (like a tool call) is completed, and the state is read at the start of each step.
智能体被调用或完成某个步骤（如工具调用）时，短期记忆会更新，且每个步骤开始时都会读取该状态。
from langchain.agents import create_agent
from langgraph.checkpoint.memory import InMemorySaver  


agent = create_agent(
    "gpt-5",
    tools=[get_user_info],
    checkpointer=InMemorySaver(),
)

agent.invoke(
    {"messages": [{"role": "user", "content": "Hi! My name is Bob."}]},
    {"configurable": {"thread_id": "1"}},
)
​
In production 生产环境中
In production, use a checkpointer backed by a database:
在生产环境中，请使用数据库支持的检查点工具：
pip install langgraph-checkpoint-postgres
from langchain.agents import create_agent

from langgraph.checkpoint.postgres import PostgresSaver  


DB_URI = "postgresql://postgres:postgres@localhost:5442/postgres?sslmode=disable"
with PostgresSaver.from_conn_string(DB_URI) as checkpointer:
    checkpointer.setup() # auto create tables in PostgreSQL
    agent = create_agent(
        "gpt-5",
        tools=[get_user_info],
        checkpointer=checkpointer,
    )
For more checkpointer options including SQLite, Postgres, and Azure Cosmos DB, see the list of checkpointer libraries in the Persistence documentation.
有关包括 SQLite、Postgres 和 Azure Cosmos DB 在内的更多检查点选项，请参阅持久性文档中的检查点库列表。
​
Customizing agent memory 自定义智能体记忆
By default, agents use AgentState to manage short term memory, specifically the conversation history via a messages key.
默认情况下，智能体使用AgentState来管理短期记忆，具体是通过messages键来存储对话历史。
You can extend AgentState to add additional fields. Custom state schemas are passed to create_agent using the state_schema parameter.
你可以扩展AgentState以添加额外字段。自定义状态架构通过state_schema参数传递给create_agent。
from langchain.agents import create_agent, AgentState
from langgraph.checkpoint.memory import InMemorySaver


class CustomAgentState(AgentState):
    user_id: str
    preferences: dict

agent = create_agent(
    "gpt-5",
    tools=[get_user_info],
    state_schema=CustomAgentState,
    checkpointer=InMemorySaver(),
)

# Custom state can be passed in invoke
result = agent.invoke(
    {
        "messages": [{"role": "user", "content": "Hello"}],
        "user_id": "user_123",
        "preferences": {"theme": "dark"}
    },
    {"configurable": {"thread_id": "1"}})
​
Common patterns 常见模式
With short-term memory enabled, long conversations can exceed the LLM’s context window. Common solutions are:
启用短期记忆后，长对话可能会超出 LLM 的上下文窗口。常见的解决方案包括：
Trim messages 修剪消息
Remove first or last N messages (before calling LLM)
在调用 LLM 之前删除前 N 条或后 N 条消息
Delete messages 删除消息
Delete messages from LangGraph state permanently
从 LangGraph 状态中永久删除消息
Summarize messages 总结消息
Summarize earlier messages in the history and replace them with a summary
总结对话历史中较早的消息，并用摘要替换它们
Custom strategies 自定义策略
Custom strategies (e.g., message filtering, etc.)
自定义策略（例如消息过滤等）
This allows the agent to keep track of the conversation without exceeding the LLM’s context window.
这使得智能体能够在不超出 LLM 上下文窗口的情况下跟踪对话进程。
​
Trim messages 修剪消息
Most LLMs have a maximum supported context window (denominated in tokens).
大多数大语言模型都有一个支持的最大上下文窗口（以标记为单位）。
One way to decide when to truncate messages is to count the tokens in the message history and truncate whenever it approaches that limit. If you’re using LangChain, you can use the trim messages utility and specify the number of tokens to keep from the list, as well as the strategy (e.g., keep the last max_tokens) to use for handling the boundary.
决定何时截断消息的一种方法是统计消息历史中的令牌数量，一旦接近该限制就进行截断。如果你在使用 LangChain，可以使用消息修剪工具，指定要从列表中保留的令牌数量，以及用于处理边界的strategy（例如，保留最后的max_tokens）。
To trim message history in an agent, use the @before_model middleware decorator:
要在智能体中修剪消息历史，请使用@before_model中间件装饰器：
from langchain.messages import RemoveMessage
from langgraph.graph.message import REMOVE_ALL_MESSAGES
from langgraph.checkpoint.memory import InMemorySaver
from langchain.agents import create_agent, AgentState
from langchain.agents.middleware import before_model
from langgraph.runtime import Runtime
from langchain_core.runnables import RunnableConfig
from typing import Any


@before_model
def trim_messages(state: AgentState, runtime: Runtime) -> dict[str, Any] | None:
    """Keep only the last few messages to fit context window."""
    messages = state["messages"]

    if len(messages) <= 3:
        return None  # No changes needed

    first_msg = messages[0]
    recent_messages = messages[-3:] if len(messages) % 2 == 0 else messages[-4:]
    new_messages = [first_msg] + recent_messages

    return {
        "messages": [
            RemoveMessage(id=REMOVE_ALL_MESSAGES),
            *new_messages
        ]
    }

agent = create_agent(
    your_model_here,
    tools=your_tools_here,
    middleware=[trim_messages],
    checkpointer=InMemorySaver(),
)

config: RunnableConfig = {"configurable": {"thread_id": "1"}}

agent.invoke({"messages": "hi, my name is bob"}, config)
agent.invoke({"messages": "write a short poem about cats"}, config)
agent.invoke({"messages": "now do the same but for dogs"}, config)
final_response = agent.invoke({"messages": "what's my name?"}, config)

final_response["messages"][-1].pretty_print()
"""
================================== Ai Message ==================================

Your name is Bob. You told me that earlier.
If you'd like me to call you a nickname or use a different name, just say the word.
"""
​
Delete messages 删除消息
You can delete messages from the graph state to manage the message history.
你可以从图状态中删除消息以管理消息历史记录。
This is useful when you want to remove specific messages or clear the entire message history.
当你想要删除特定消息或清除整个消息历史记录时，这会非常有用。
To delete messages from the graph state, you can use the RemoveMessage.
要从图状态中删除消息，你可以使用RemoveMessage。
For RemoveMessage to work, you need to use a state key with add_messages reducer.
要使RemoveMessage生效，你需要在add_messages的reducer中使用状态键。
The default AgentState provides this.
默认的 AgentState 就提供了这一点。
To remove specific messages: 要删除特定消息：
from langchain.messages import RemoveMessage  

def delete_messages(state):
    messages = state["messages"]
    if len(messages) > 2:
        # remove the earliest two messages
        return {"messages": [RemoveMessage(id=m.id) for m in messages[:2]]}
To remove all messages: 要删除所有消息：
from langgraph.graph.message import REMOVE_ALL_MESSAGES  

def delete_messages(state):
    return {"messages": [RemoveMessage(id=REMOVE_ALL_MESSAGES)]}
When deleting messages, make sure that the resulting message history is valid. Check the limitations of the LLM provider you’re using. For example:
删除消息时，请确保生成的消息历史记录是有效的。请查看你所使用的 LLM 提供商的限制。例如：
Some providers expect message history to start with a user message
部分提供商要求消息历史记录以user消息开头
Most providers require assistant messages with tool calls to be followed by corresponding tool result messages.
大多数提供商要求带有工具调用的assistant消息后面必须跟上对应的tool结果消息。
from langchain.messages import RemoveMessage
from langchain.agents import create_agent, AgentState
from langchain.agents.middleware import after_model
from langgraph.checkpoint.memory import InMemorySaver
from langgraph.runtime import Runtime
from langchain_core.runnables import RunnableConfig


@after_model
def delete_old_messages(state: AgentState, runtime: Runtime) -> dict | None:
    """Remove old messages to keep conversation manageable."""
    messages = state["messages"]
    if len(messages) > 2:
        # remove the earliest two messages
        return {"messages": [RemoveMessage(id=m.id) for m in messages[:2]]}
    return None


agent = create_agent(
    "gpt-5-nano",
    tools=[],
    system_prompt="Please be concise and to the point.",
    middleware=[delete_old_messages],
    checkpointer=InMemorySaver(),
)

config: RunnableConfig = {"configurable": {"thread_id": "1"}}

for event in agent.stream(
    {"messages": [{"role": "user", "content": "hi! I'm bob"}]},
    config,
    stream_mode="values",
):
    print([(message.type, message.content) for message in event["messages"]])

for event in agent.stream(
    {"messages": [{"role": "user", "content": "what's my name?"}]},
    config,
    stream_mode="values",
):
    print([(message.type, message.content) for message in event["messages"]])
[('human', "hi! I'm bob")]
[('human', "hi! I'm bob"), ('ai', 'Hi Bob! Nice to meet you. How can I help you today? I can answer questions, brainstorm ideas, draft text, explain things, or help with code.')]
[('human', "hi! I'm bob"), ('ai', 'Hi Bob! Nice to meet you. How can I help you today? I can answer questions, brainstorm ideas, draft text, explain things, or help with code.'), ('human', "what's my name?")]
[('human', "hi! I'm bob"), ('ai', 'Hi Bob! Nice to meet you. How can I help you today? I can answer questions, brainstorm ideas, draft text, explain things, or help with code.'), ('human', "what's my name?"), ('ai', 'Your name is Bob. How can I help you today, Bob?')]
[('human', "what's my name?"), ('ai', 'Your name is Bob. How can I help you today, Bob?')]
​
Summarize messages 总结消息
The problem with trimming or removing messages, as shown above, is that you may lose information from culling of the message queue. Because of this, some applications benefit from a more sophisticated approach of summarizing the message history using a chat model.
如上所示，精简或删除消息存在一个问题：你可能会因消息队列的筛选而丢失信息。正因为如此，一些应用程序可以从更复杂的方法中获益——即使用聊天模型来总结聊天历史。
Summary
To summarize message history in an agent, use the built-in SummarizationMiddleware:
要在智能体中汇总消息历史，请使用内置的 SummarizationMiddleware：
from langchain.agents import create_agent
from langchain.agents.middleware import SummarizationMiddleware
from langgraph.checkpoint.memory import InMemorySaver
from langchain_core.runnables import RunnableConfig


checkpointer = InMemorySaver()

agent = create_agent(
    model="gpt-4.1",
    tools=[],
    middleware=[
        SummarizationMiddleware(
            model="gpt-4.1-mini",
            trigger=("tokens", 4000),
            keep=("messages", 20)
        )
    ],
    checkpointer=checkpointer,
)

config: RunnableConfig = {"configurable": {"thread_id": "1"}}
agent.invoke({"messages": "hi, my name is bob"}, config)
agent.invoke({"messages": "write a short poem about cats"}, config)
agent.invoke({"messages": "now do the same but for dogs"}, config)
final_response = agent.invoke({"messages": "what's my name?"}, config)

final_response["messages"][-1].pretty_print()
"""
================================== Ai Message ==================================

Your name is Bob!
"""
See SummarizationMiddleware for more configuration options.
有关更多配置选项，请参阅SummarizationMiddleware。
​
Access memory 访问记忆
You can access and modify the short-term memory (state) of an agent in several ways:
你可以通过多种方式访问和修改智能体的短期记忆（状态）：
​
Tools 工具
​
Read short-term memory in a tool 读取工具中的短期记忆
Access short term memory (state) in a tool using the runtime parameter (typed as ToolRuntime).
使用runtime参数（类型为ToolRuntime）在工具中访问短期记忆（状态）。
The runtime parameter is hidden from the tool signature (so the model doesn’t see it), but the tool can access the state through it.
runtime参数对工具签名隐藏（因此模型无法看到它），但工具可以通过它访问状态。
from langchain.agents import create_agent, AgentState
from langchain.tools import tool, ToolRuntime


class CustomState(AgentState):
    user_id: str

@tool
def get_user_info(
    runtime: ToolRuntime
) -> str:
    """Look up user info."""
    user_id = runtime.state["user_id"]
    return "User is John Smith" if user_id == "user_123" else "Unknown user"

agent = create_agent(
    model="gpt-5-nano",
    tools=[get_user_info],
    state_schema=CustomState,
)

result = agent.invoke({
    "messages": "look up user information",
    "user_id": "user_123"
})
print(result["messages"][-1].content)
# > User is John Smith.
​
Write short-term memory from tools 通过工具写入短时记忆
To modify the agent’s short-term memory (state) during execution, you can return state updates directly from the tools.
要在执行过程中修改智能体的短期记忆（状态），你可以直接从工具中返回状态更新。
This is useful for persisting intermediate results or making information accessible to subsequent tools or prompts.
这对于保留中间结果或让后续工具或提示能够访问信息非常有用。
from langchain.tools import tool, ToolRuntime
from langchain_core.runnables import RunnableConfig
from langchain.messages import ToolMessage
from langchain.agents import create_agent, AgentState
from langgraph.types import Command
from pydantic import BaseModel


class CustomState(AgentState):
    user_name: str

class CustomContext(BaseModel):
    user_id: str

@tool
def update_user_info(
    runtime: ToolRuntime[CustomContext, CustomState],
) -> Command:
    """Look up and update user info."""
    user_id = runtime.context.user_id
    name = "John Smith" if user_id == "user_123" else "Unknown user"
    return Command(update={
        "user_name": name,
        # update the message history
        "messages": [
            ToolMessage(
                "Successfully looked up user information",
                tool_call_id=runtime.tool_call_id
            )
        ]
    })

@tool
def greet(
    runtime: ToolRuntime[CustomContext, CustomState]
) -> str | Command:
    """Use this to greet the user once you found their info."""
    user_name = runtime.state.get("user_name", None)
    if user_name is None:
       return Command(update={
            "messages": [
                ToolMessage(
                    "Please call the 'update_user_info' tool it will get and update the user's name.",
                    tool_call_id=runtime.tool_call_id
                )
            ]
        })
    return f"Hello {user_name}!"

agent = create_agent(
    model="gpt-5-nano",
    tools=[update_user_info, greet],
    state_schema=CustomState,
    context_schema=CustomContext,
)

agent.invoke(
    {"messages": [{"role": "user", "content": "greet the user"}]},
    context=CustomContext(user_id="user_123"),
)
​
Prompt 提示词
Access short term memory (state) in middleware to create dynamic prompts based on conversation history or custom state fields.
在中间件中访问短期记忆（状态），以基于对话历史或自定义状态字段创建动态提示词。
from langchain.agents import create_agent
from typing import TypedDict
from langchain.agents.middleware import dynamic_prompt, ModelRequest


class CustomContext(TypedDict):
    user_name: str


def get_weather(city: str) -> str:
    """Get the weather in a city."""
    return f"The weather in {city} is always sunny!"


@dynamic_prompt
def dynamic_system_prompt(request: ModelRequest) -> str:
    user_name = request.runtime.context["user_name"]
    system_prompt = f"You are a helpful assistant. Address the user as {user_name}."
    return system_prompt


agent = create_agent(
    model="gpt-5-nano",
    tools=[get_weather],
    middleware=[dynamic_system_prompt],
    context_schema=CustomContext,
)

result = agent.invoke(
    {"messages": [{"role": "user", "content": "What is the weather in SF?"}]},
    context=CustomContext(user_name="John Smith"),
)
for msg in result["messages"]:
    msg.pretty_print()

Output 输出
================================ Human Message =================================

What is the weather in SF?
================================== Ai Message ==================================
Tool Calls:
  get_weather (call_WFQlOGn4b2yoJrv7cih342FG)
 Call ID: call_WFQlOGn4b2yoJrv7cih342FG
  Args:
    city: San Francisco
================================= Tool Message =================================
Name: get_weather

The weather in San Francisco is always sunny!
================================== Ai Message ==================================

Hi John Smith, the weather in San Francisco is always sunny!
​
Before model 模型前
Access short term memory (state) in @before_model middleware to process messages before model calls.
在 @before_model 中间件中访问短期记忆（状态），以在模型调用前处理消息。







__start__

before_model

model

tools

__end__

from langchain.messages import RemoveMessage
from langgraph.graph.message import REMOVE_ALL_MESSAGES
from langgraph.checkpoint.memory import InMemorySaver
from langchain.agents import create_agent, AgentState
from langchain.agents.middleware import before_model
from langchain_core.runnables import RunnableConfig
from langgraph.runtime import Runtime
from typing import Any


@before_model
def trim_messages(state: AgentState, runtime: Runtime) -> dict[str, Any] | None:
    """Keep only the last few messages to fit context window."""
    messages = state["messages"]

    if len(messages) <= 3:
        return None  # No changes needed

    first_msg = messages[0]
    recent_messages = messages[-3:] if len(messages) % 2 == 0 else messages[-4:]
    new_messages = [first_msg] + recent_messages

    return {
        "messages": [
            RemoveMessage(id=REMOVE_ALL_MESSAGES),
            *new_messages
        ]
    }


agent = create_agent(
    "gpt-5-nano",
    tools=[],
    middleware=[trim_messages],
    checkpointer=InMemorySaver()
)

config: RunnableConfig = {"configurable": {"thread_id": "1"}}

agent.invoke({"messages": "hi, my name is bob"}, config)
agent.invoke({"messages": "write a short poem about cats"}, config)
agent.invoke({"messages": "now do the same but for dogs"}, config)
final_response = agent.invoke({"messages": "what's my name?"}, config)

final_response["messages"][-1].pretty_print()
"""
================================== Ai Message ==================================

Your name is Bob. You told me that earlier.
If you'd like me to call you a nickname or use a different name, just say the word.
"""
​
After model 模型后
Access short term memory (state) in @after_model middleware to process messages after model calls.
在 @after_model 中间件中访问短期记忆（状态），以在模型调用后处理消息。







__start__

model

after_model

tools

__end__

from langchain.messages import RemoveMessage
from langgraph.checkpoint.memory import InMemorySaver
from langchain.agents import create_agent, AgentState
from langchain.agents.middleware import after_model
from langgraph.runtime import Runtime


@after_model
def validate_response(state: AgentState, runtime: Runtime) -> dict | None:
    """Remove messages containing sensitive words."""
    STOP_WORDS = ["password", "secret"]
    last_message = state["messages"][-1]
    if any(word in last_message.content for word in STOP_WORDS):
        return {"messages": [RemoveMessage(id=last_message.id)]}
    return None

agent = create_agent(
    model="gpt-5-nano",
    tools=[],
    middleware=[validate_response],
    checkpointer=InMemorySaver(),
)
