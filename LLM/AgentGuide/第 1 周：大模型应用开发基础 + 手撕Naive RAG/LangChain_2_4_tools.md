# Tools

Tools extend what agents can do—letting them fetch real-time data, execute code, query external databases, and take actions in the world.
> 工具拓展了智能体的能力——让它们能够获取实时数据、执行代码、查询外部数据库，并在现实世界中采取行动。

Under the hood, tools are callable functions with well-defined inputs and outputs that get passed to a chat model. The model decides when to invoke a tool based on the conversation context, and what input arguments to provide.
> 在底层，工具是具有明确定义的输入和输出的可调用函数，这些函数会被传递给聊天模型。模型会根据对话上下文决定何时调用工具，以及提供哪些输入参数。

For details on how models handle tool calls, see [Tool calling](https://github.com/meng-yijie1996/myNotes/blob/master/LLM/AgentGuide/%E7%AC%AC%201%20%E5%91%A8%EF%BC%9A%E5%A4%A7%E6%A8%A1%E5%9E%8B%E5%BA%94%E7%94%A8%E5%BC%80%E5%8F%91%E5%9F%BA%E7%A1%80%20%2B%20%E6%89%8B%E6%92%95Naive%20RAG/LangChain_2_2_model.md#tool-calling).
> 有关模型如何处理工具调用的详细信息，请参见工具调用。
​
## Create tools
### Basic tool definition
The simplest way to create a tool is with the `@tool` decorator. By default, the function’s docstring becomes the tool’s description that helps the model understand when to use it:
> 创建工具最简单的方法是使用 @tool 装饰器。默认情况下，函数的文档字符串会成为工具的描述，帮助模型了解何时使用该工具：

``` python
from langchain.tools import tool

@tool
def search_database(query: str, limit: int = 10) -> str:
    """Search the customer database for records matching the query.

    Args:
        query: Search terms to look for
        limit: Maximum number of results to return
    """
    return f"Found {limit} results for '{query}'"
```

**Type hints** are required as they define the tool’s input schema. The docstring should be **informative and concise** to help the model understand the tool’s purpose.
> 类型提示是必需的，因为它们定义了工具的输入模式。文档字符串应具有信息性和简洁性，以帮助模型理解工具的用途。

Server-side tool use: Some chat models feature built-in tools (web search, code interpreters) that are executed server-side. See Server-side tool use for details.
> 服务端工具使用：部分聊天模型内置了在服务端执行的工具（网络搜索、代码解释器）。有关详细信息，请参阅服务端工具使用。

Prefer `snake_case` for tool names (e.g., web_search instead of Web Search). Some model providers have issues with or reject names containing spaces or special characters with errors. Sticking to alphanumeric characters, underscores, and hyphens helps to improve compatibility across providers.
> 工具名称优先使用 snake_case（例如，用 web_search 而非 Web Search）。部分模型提供商对包含空格或特殊字符的名称存在问题或会因报错拒绝使用这类名称。只使用字母数字、下划线和连字符有助于提升跨提供商的兼容性。
​
### Customize tool properties
#### Custom tool name
By default, the tool name comes from the function name. Override it when you need something more descriptive:
> 默认情况下，工具名称来源于函数名称。如需更具描述性的名称，请对其进行覆盖：

``` python
@tool("web_search")  # NOTE: Custom name
def search(query: str) -> str:
    """Search the web for information."""
    return f"Results for: {query}"

print(search.name)  # web_search
```

#### Custom tool description
Override the auto-generated tool description for clearer model guidance:
> 重写自动生成的工具描述，以获得更清晰的模型指导：

``` python
@tool("calculator", description="Performs arithmetic calculations. Use this for any math problems.")
def calc(expression: str) -> str:
    """Evaluate mathematical expressions."""
    return str(eval(expression))​
```

### Advanced schema definition
Define **complex inputs** with Pydantic models or JSON schemas:

##### Pydantic model
``` python
from pydantic import BaseModel, Field
from typing import Literal

class WeatherInput(BaseModel):
    """Input for weather queries."""
    location: str = Field(description="City name or coordinates")
    units: Literal["celsius", "fahrenheit"] = Field(
        default="celsius",
        description="Temperature unit preference"
    )
    include_forecast: bool = Field(
        default=False,
        description="Include 5-day forecast"
    )

@tool(args_schema=WeatherInput)
def get_weather(location: str, units: str = "celsius", include_forecast: bool = False) -> str:
    """Get current weather and optional forecast."""
    temp = 22 if units == "celsius" else 72
    result = f"Current weather in {location}: {temp} degrees {units[0].upper()}"
    if include_forecast:
        result += "\nNext 5 days: Sunny"
    return result

```

##### JSON Schema
``` python
weather_schema = {
    "type": "object",
    "properties": {
        "location": {"type": "string"},
        "units": {"type": "string"},
        "include_forecast": {"type": "boolean"}
    },
    "required": ["location", "units", "include_forecast"]
}

@tool(args_schema=weather_schema)
def get_weather(location: str, units: str = "celsius", include_forecast: bool = False) -> str:
    """Get current weather and optional forecast."""
    temp = 22 if units == "celsius" else 72
    result = f"Current weather in {location}: {temp} degrees {units[0].upper()}"
    if include_forecast:
        result += "\nNext 5 days: Sunny"
    return result
```
​
### Reserved argument names
The following parameter names are reserved and cannot be used as tool arguments. Using these names will cause runtime errors.
> 以下参数名称为保留名称，不能用作工具参数。使用这些名称会导致运行时错误。

|Parameter name|Purpose|
|:---|:---|
|config|Reserved for passing RunnableConfig to tools internally<br>保留用于在内部将 RunnableConfig 传递给工具|
|runtime|Reserved for ToolRuntime parameter (accessing state, context, store)|

To access runtime information, use the ToolRuntime parameter instead of naming your own arguments config or runtime.
> 要访问运行时信息，请使用ToolRuntime参数，而不是自行命名config或runtime参数。
​
## Access context
Tools are most powerful when they can access runtime information like conversation history, user data, and persistent memory. This section covers how to **access and update** this information from within your tools.
> 工具在能够访问对话历史、用户数据和持久化内存等运行时信息时，功能最为强大。本节将介绍如何在工具内部访问和更新这些信息。

Tools can access runtime information through the `ToolRuntime` parameter, which provides:
> 工具可通过ToolRuntime参数访问运行时信息，该参数提供以下内容：

|Component|Description|Use case|
|:---|:---|:---|
|**State**|Short-term memory - mutable data that exists for the current conversation (messages, counters, custom fields) <br> 短期记忆——存在于当前对话中的可变数据（消息、计数器、自定义字段）|Access conversation history, track tool call counts|

访问对话历史，记录工具调用次数
Context 上下文	Immutable configuration passed at invocation time (user IDs, session info)
调用时传入的不可变配置（用户 ID、会话信息）
Personalize responses based on user identity
根据用户身份个性化回复
Store 存储	Long-term memory - persistent data that survives across conversations
长期记忆——可在多次对话中留存的持久化数据
Save user preferences, maintain knowledge base
保存用户偏好设置，维护知识库
Stream Writer 流式写入器	Emit real-time updates during tool execution
在工具执行期间发送实时更新
Show progress for long-running operations
为耗时操作显示进度
Execution Info 执行信息	Identity and retry information for the current execution (thread ID, run ID, attempt number)
当前执行的身份和重试信息（线程 ID、运行 ID、尝试次数）
Access thread/run IDs, adjust behavior based on retry state
获取线程/运行ID，根据重试状态调整行为
Server Info 服务器信息	Server-specific metadata when running on LangGraph Server (assistant ID, graph ID, authenticated user)
在 LangGraph Server 上运行时的服务器特定元数据（助手 ID、图表 ID、已认证用户）
Access assistant ID, graph ID, or authenticated user info
获取助手ID、图表ID或已认证的用户信息
Config 配置	RunnableConfig for the execution RunnableConfig 用于执行	Access callbacks, tags, and metadata
访问回调函数、标签和元数据
Tool Call ID 工具调用ID	Unique identifier for the current tool invocation
当前工具调用的唯一标识符
Correlate tool calls for logs and model invocations
为日志和模型调用关联工具调用







⚡ Enhanced Tool Capabilities

📊 Available Resources

🔧 Tool Runtime Context

Tool Call

ToolRuntime

State Access

Context Access

Store Access

Stream Writer

Messages

Custom State

User ID

Session Info

Long-term Memory

User Preferences

Context-Aware Tools

Stateful Tools

Memory-Enabled Tools

Streaming Tools

​
Short-term memory (State) 短期记忆（状态）
State represents short-term memory that exists for the duration of a conversation. It includes the message history and any custom fields you define in your graph state.
状态表示在一次对话期间存在的短期记忆。它包含消息历史以及你在图状态中定义的任何自定义字段。
Add runtime: ToolRuntime to your tool signature to access state. This parameter is automatically injected and hidden from the LLM - it won’t appear in the tool’s schema.
在你的工具签名中添加 runtime: ToolRuntime 以访问状态。此参数会被自动注入并对 LLM 隐藏——它不会出现在工具的架构中。
​
Access state 访问状态
Tools can access the current conversation state using runtime.state:
工具可以通过runtime.state访问当前的对话状态：
from langchain.tools import tool, ToolRuntime
from langchain.messages import HumanMessage

@tool
def get_last_user_message(runtime: ToolRuntime) -> str:
    """Get the most recent message from the user."""
    messages = runtime.state["messages"]

    # Find the last human message
    for message in reversed(messages):
        if isinstance(message, HumanMessage):
            return message.content

    return "No user messages found"

# Access custom state fields
@tool
def get_user_preference(
    pref_name: str,
    runtime: ToolRuntime
) -> str:
    """Get a user preference value."""
    preferences = runtime.state.get("user_preferences", {})
    return preferences.get(pref_name, "Not set")
The runtime parameter is hidden from the model. For the example above, the model only sees pref_name in the tool schema.
runtime参数对模型隐藏。以上示例中，模型在工具架构中只能看到pref_name。
​
Update state 更新状态
Use Command to update the agent’s state. This is useful for tools that need to update custom state fields:
使用Command更新智能体的状态。这对于需要更新自定义状态字段的工具非常有用：
from langgraph.types import Command
from langchain.tools import tool

@tool
def set_user_name(new_name: str) -> Command:
    """Set the user's name in the conversation state."""
    return Command(update={"user_name": new_name})
When tools update state variables, consider defining a reducer for those fields. Since LLMs can call multiple tools in parallel, a reducer determines how to resolve conflicts when the same state field is updated by concurrent tool calls.
当工具更新状态变量时，考虑为这些字段定义一个reducer。由于大语言模型（LLMs）可以并行调用多个工具，当同一状态字段被并发的工具调用更新时，reducer 会确定如何解决冲突。
​
Context 上下文
Context provides immutable configuration data that is passed at invocation time. Use it for user IDs, session details, or application-specific settings that shouldn’t change during a conversation.
上下文提供在调用时传递的不可变配置数据。可将其用于用户 ID、会话详情或对话期间不应更改的特定于应用程序的设置。
Access context through runtime.context:
通过runtime.context访问上下文：
from dataclasses import dataclass
from langchain_openai import ChatOpenAI
from langchain.agents import create_agent
from langchain.tools import tool, ToolRuntime


USER_DATABASE = {
    "user123": {
        "name": "Alice Johnson",
        "account_type": "Premium",
        "balance": 5000,
        "email": "alice@example.com"
    },
    "user456": {
        "name": "Bob Smith",
        "account_type": "Standard",
        "balance": 1200,
        "email": "bob@example.com"
    }
}

@dataclass
class UserContext:
    user_id: str

@tool
def get_account_info(runtime: ToolRuntime[UserContext]) -> str:
    """Get the current user's account information."""
    user_id = runtime.context.user_id

    if user_id in USER_DATABASE:
        user = USER_DATABASE[user_id]
        return f"Account holder: {user['name']}\nType: {user['account_type']}\nBalance: ${user['balance']}"
    return "User not found"

model = ChatOpenAI(model="gpt-4.1")
agent = create_agent(
    model,
    tools=[get_account_info],
    context_schema=UserContext,
    system_prompt="You are a financial assistant."
)

result = agent.invoke(
    {"messages": [{"role": "user", "content": "What's my current balance?"}]},
    context=UserContext(user_id="user123")
)
​
Long-term memory (Store) 长时记忆（存储）
The BaseStore provides persistent storage that survives across conversations. Unlike state (short-term memory), data saved to the store remains available in future sessions.
BaseStore 提供跨对话持续存在的持久化存储。与状态（短期记忆）不同，保存到该存储中的数据在未来会话中仍然可用。
Access the store through runtime.store. The store uses a namespace/key pattern to organize data:
通过 runtime.store 访问存储库。该存储库使用命名空间/键模式来组织数据：
For production deployments, use a persistent store implementation like PostgresStore instead of InMemoryStore. See the memory documentation for setup details.
对于生产环境部署，请使用 PostgresStore 等持久化存储实现，而非 InMemoryStore。有关设置的详细信息，请参阅 内存文档。
from typing import Any
from langgraph.store.memory import InMemoryStore
from langchain.agents import create_agent
from langchain.tools import tool, ToolRuntime
from langchain_openai import ChatOpenAI

# Access memory
@tool
def get_user_info(user_id: str, runtime: ToolRuntime) -> str:
    """Look up user info."""
    store = runtime.store
    user_info = store.get(("users",), user_id)
    return str(user_info.value) if user_info else "Unknown user"

# Update memory
@tool
def save_user_info(user_id: str, user_info: dict[str, Any], runtime: ToolRuntime) -> str:
    """Save user info."""
    store = runtime.store
    store.put(("users",), user_id, user_info)
    return "Successfully saved user info."

model = ChatOpenAI(model="gpt-4.1")

store = InMemoryStore()
agent = create_agent(
    model,
    tools=[get_user_info, save_user_info],
    store=store
)

# First session: save user info
agent.invoke({
    "messages": [{"role": "user", "content": "Save the following user: userid: abc123, name: Foo, age: 25, email: foo@langchain.dev"}]
})

# Second session: get user info
agent.invoke({
    "messages": [{"role": "user", "content": "Get user info for user with id 'abc123'"}]
})
# Here is the user info for user with ID "abc123":
# - Name: Foo
# - Age: 25
# - Email: foo@langchain.dev
See all 44 lines
查看全部44行
​
Stream writer 流写入器
Stream real-time updates from tools during execution. This is useful for providing progress feedback to users during long-running operations.
在执行过程中流式传输工具的实时更新。这对于在长时间运行的操作期间向用户提供进度反馈非常有用。
Use runtime.stream_writer to emit custom updates:
使用runtime.stream_writer来输出自定义更新：
from langchain.tools import tool, ToolRuntime

@tool
def get_weather(city: str, runtime: ToolRuntime) -> str:
    """Get weather for a given city."""
    writer = runtime.stream_writer

    # Stream custom updates as the tool executes
    writer(f"Looking up data for city: {city}")
    writer(f"Acquired data for city: {city}")

    return f"It's always sunny in {city}!"
If you use runtime.stream_writer inside your tool, the tool must be invoked within a LangGraph execution context. See Streaming for more details.
如果在你的工具中使用 `runtime.stream_writer`，则该工具必须在 LangGraph 执行上下文中被调用。有关更多详细信息，请参阅 `流式处理`。
​
Execution info 执行信息
Access thread ID, run ID, and retry state from within a tool via runtime.execution_info:
通过 runtime.execution_info 从工具内部访问线程 ID、运行 ID 和重试状态：
from langchain.tools import tool, ToolRuntime

@tool
def log_execution_context(runtime: ToolRuntime) -> str:
    """Log execution identity information."""
    info = runtime.execution_info
    print(f"Thread: {info.thread_id}, Run: {info.run_id}")
    print(f"Attempt: {info.node_attempt}")
    return "done"
Requires deepagents>=0.5.0 (or langgraph>=1.1.5).
需要 deepagents>=0.5.0（或 langgraph>=1.1.5）。
​
Server info 服务器信息
When your tool runs on LangGraph Server, access the assistant ID, graph ID, and authenticated user via runtime.server_info:
当你的工具在 LangGraph Server 上运行时，可通过 runtime.server_info 访问助手 ID、图 ID 和已认证用户：
from langchain.tools import tool, ToolRuntime

@tool
def get_assistant_scoped_data(runtime: ToolRuntime) -> str:
    """Fetch data scoped to the current assistant."""
    server = runtime.server_info
    if server is not None:
        print(f"Assistant: {server.assistant_id}, Graph: {server.graph_id}")
        if server.user is not None:
            print(f"User: {server.user.identity}")
    return "done"
server_info is None when the tool is not running on LangGraph Server (e.g., during local development or testing).
当工具未在 LangGraph 服务器上运行时（例如在本地开发或测试期间），server_info 为 None。
Requires deepagents>=0.5.0 (or langgraph>=1.1.5).
需要 deepagents>=0.5.0（或 langgraph>=1.1.5）。
​
ToolNode 工具节点
ToolNode is a prebuilt node that executes tools in LangGraph workflows. It handles parallel tool execution, error handling, and state injection automatically.
ToolNode 是一个用于在 LangGraph 工作流中执行工具的预构建节点。它会自动处理并行工具执行、错误处理和状态注入。
For custom workflows where you need fine-grained control over tool execution patterns, use ToolNode instead of create_agent. It’s the building block that powers agent tool execution.
对于需要对工具执行模式进行精细控制的自定义工作流，请使用 ToolNode 而非 create_agent。它是驱动智能体工具执行的基础组件。
​
Basic usage 基本用法
from langchain.tools import tool
from langgraph.prebuilt import ToolNode
from langgraph.graph import StateGraph, MessagesState, START, END

@tool
def search(query: str) -> str:
    """Search for information."""
    return f"Results for: {query}"

@tool
def calculator(expression: str) -> str:
    """Evaluate a math expression."""
    return str(eval(expression))

# Create the ToolNode with your tools
tool_node = ToolNode([search, calculator])

# Use in a graph
builder = StateGraph(MessagesState)
builder.add_node("tools", tool_node)
# ... add other nodes and edges
​
Tool return values 工具返回值
You can choose different return values for your tools:
你可以为工具选择不同的返回值：
Return a string for human-readable results.
返回一个string以获取人类可读的结果。
Return an object for structured results the model should parse.
返回一个object，用于获取模型需要解析的结构化结果。
Return a Command with optional message when you need to write to state.
当你需要写入状态时，返回一个Command并附带可选消息。
​
Return a string 返回字符串
Return a string when the tool should provide plain text for the model to read and use in its next response.
当工具需要为模型提供纯文本供其阅读并在下一次回复中使用时，返回一个字符串。
from langchain.tools import tool


@tool
def get_weather(city: str) -> str:
    """Get weather for a city."""
    return f"It is currently sunny in {city}."
Behavior: 行为：
The return value is converted to a ToolMessage.
返回值会被转换为ToolMessage。
The model sees that text and decides what to do next.
模型会查看该文本并决定下一步的操作。
No agent state fields are changed unless the model or another tool does so later.
除非模型或其他工具后续进行修改，否则不会更改任何智能体状态字段。
Use this when the result is naturally human-readable text.
当结果是天然的人类可读文本时，使用此方式。
​
Return an object 返回一个对象
Return an object (for example, a dict) when your tool produces structured data that the model should inspect.
当你的工具生成模型需要检查的结构化数据时，返回一个对象（例如 dict）。
from langchain.tools import tool


@tool
def get_weather_data(city: str) -> dict:
    """Get structured weather data for a city."""
    return {
        "city": city,
        "temperature_c": 22,
        "conditions": "sunny",
    }
Behavior: 行为：
The object is serialized and sent back as tool output.
该对象会被序列化并作为工具输出返回。
The model can read specific fields and reason over them.
模型可以读取特定字段并对其进行推理。
Like string returns, this does not directly update graph state.
与字符串返回一样，这不会直接更新图状态。
Use this when downstream reasoning benefits from explicit fields instead of free-form text.
当下游推理得益于明确的字段而非自由格式的文本时，请使用此方式。
​
Return a Command 返回一个命令
Return a Command when the tool needs to update graph state (for example, setting user preferences or app state). You can return a Command with or without including a ToolMessage. If the model needs to see that the tool succeeded (for example, to confirm a preference change), include a ToolMessage in the update, using runtime.tool_call_id for the tool_call_id parameter.
当工具需要更新图表状态（例如，设置用户首选项或应用程序状态）时，返回一个Command。你可以返回包含ToolMessage或不包含ToolMessage的Command。如果模型需要确认工具执行成功（例如，确认首选项已更改），请在更新中包含ToolMessage，并将runtime.tool_call_id用作tool_call_id参数。
from langchain.messages import ToolMessage
from langchain.tools import ToolRuntime, tool
from langgraph.types import Command


@tool
def set_language(language: str, runtime: ToolRuntime) -> Command:
    """Set the preferred response language."""
    return Command(
        update={
            "preferred_language": language,
            "messages": [
                ToolMessage(
                    content=f"Language set to {language}.",
                    tool_call_id=runtime.tool_call_id,
                )
            ],
        }
    )
Behavior: 行为：
The command updates state using update.
该命令使用update更新状态。
Updated state is available to subsequent steps in the same run.
更新后的状态在同一次运行的后续步骤中均可用。
Use reducers for fields that may be updated by parallel tool calls.
为可能被并行工具调用更新的字段使用reducer。
Use this when the tool is not just returning data, but also mutating agent state.
当工具不仅返回数据，还会修改智能体状态时，请使用此方法。
​
Error handling 错误处理
Configure how tool errors are handled. See the ToolNode API reference for all options.
配置工具错误的处理方式。有关所有选项，请参阅 ToolNode API 参考。
from langgraph.prebuilt import ToolNode

# Default: catch invocation errors, re-raise execution errors
tool_node = ToolNode(tools)

# Catch all errors and return error message to LLM
tool_node = ToolNode(tools, handle_tool_errors=True)

# Custom error message
tool_node = ToolNode(tools, handle_tool_errors="Something went wrong, please try again.")

# Custom error handler
def handle_error(e: ValueError) -> str:
    return f"Invalid input: {e}"

tool_node = ToolNode(tools, handle_tool_errors=handle_error)

# Only catch specific exception types
tool_node = ToolNode(tools, handle_tool_errors=(ValueError, TypeError))
​
Route with tools_condition 使用工具条件进行路由
Use tools_condition for conditional routing based on whether the LLM made tool calls:
使用tools_condition，根据大模型是否调用了工具进行条件路由：
from langgraph.prebuilt import ToolNode, tools_condition
from langgraph.graph import StateGraph, MessagesState, START, END

builder = StateGraph(MessagesState)
builder.add_node("llm", call_llm)
builder.add_node("tools", ToolNode(tools))

builder.add_edge(START, "llm")
builder.add_conditional_edges("llm", tools_condition)  # Routes to "tools" or END
builder.add_edge("tools", "llm")

graph = builder.compile()
​
State injection 状态注入
Tools can access the current graph state through ToolRuntime:
工具可通过ToolRuntime访问当前的图状态：
from langchain.tools import tool, ToolRuntime
from langgraph.prebuilt import ToolNode

@tool
def get_message_count(runtime: ToolRuntime) -> str:
    """Get the number of messages in the conversation."""
    messages = runtime.state["messages"]
    return f"There are {len(messages)} messages."

tool_node = ToolNode([get_message_count])
For more details on accessing state, context, and long-term memory from tools, see Access context.
有关如何从工具访问状态、上下文和长期记忆的更多详细信息，请参阅 访问上下文。
​
Prebuilt tools 预构建工具
LangChain provides a large collection of prebuilt tools and toolkits for common tasks like web search, code interpretation, database access, and more. These ready-to-use tools can be directly integrated into your agents without writing custom code.
LangChain 提供了大量针对常见任务的预构建工具和工具包，例如网络搜索、代码解释、数据库访问等。这些即用型工具可直接集成到你的智能体中，无需编写自定义代码。
See the tools and toolkits integration page for a complete list of available tools organized by category.
请查看工具和工具包集成页面，获取按类别整理的可用工具完整列表。
​
Server-side tool use 服务端工具使用
Some chat models feature built-in tools that are executed server-side by the model provider. These include capabilities like web search and code interpreters that don’t require you to define or host the tool logic.
部分聊天模型配备了由模型提供商在服务器端执行的内置工具。这些工具包括网页搜索、代码解释器等功能，无需你定义或托管工具逻辑。
Refer to the individual chat model integration pages and the tool calling documentation for details on enabling and using these built-in tools.
有关启用和使用这些内置工具的详细信息，请参阅单独的聊天模型集成页面和工具调用文档。
