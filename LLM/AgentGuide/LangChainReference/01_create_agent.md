# create_agent
Creates an agent graph that calls tools in a loop until a stopping condition is met.
> 创建一个智能体图，该图会循环调用工具，直到满足停止条件。

``` python
create_agent(
  model: str | BaseChatModel,
  tools: Sequence[BaseTool | Callable[..., Any] | dict[str, Any]] | None = None,
  *,
  system_prompt: str | SystemMessage | None = None,
  middleware: Sequence[AgentMiddleware[StateT_co, ContextT]] = (),
  response_format: ResponseFormat[ResponseT] | type[ResponseT] | dict[str, Any] | None = None,
  state_schema: type[AgentState[ResponseT]] | None = None,
  context_schema: type[ContextT] | None = None,
  checkpointer: Checkpointer | None = None,
  store: BaseStore | None = None,
  interrupt_before: list[str] | None = None,
  interrupt_after: list[str] | None = None,
  debug: bool = False,
  name: str | None = None,
  cache: BaseCache[Any] | None = None
) -> CompiledStateGraph[AgentState[ResponseT], ContextT, _InputAgentState, _OutputAgentState[ResponseT]]
```

|概念|核心作用|
|:---|:---|
|*|分隔符，强制后续参数必须用关键字传参，避免参数顺序混乱|
|Sequence|类型注解，表示「有序可迭代对象」（list/tuple 等），兼容多种有序集合类型|

The agent node calls the language model with the **messages list** (after applying the system prompt). If the resulting `AIMessage` contains `tool_calls`, the graph will then call the tools. The tools node executes the tools and adds the responses to the **messages list** as `ToolMessage` objects. The agent node then calls the language model again. The process repeats until **no more  `tool_calls`** are present in the response. The agent then returns the full list of messages.
> 智能体节点使用消息列表调用语言模型（在应用系统提示之后）。如果生成的AIMessage包含tool_calls，则图会调用工具。工具节点执行这些工具，并将响应作为ToolMessage对象添加到消息列表中。之后，智能体节点再次调用语言模型。这个过程会重复进行，直到响应中不再存在tool_calls。然后，智能体会返回完整的消息列表。

### Example

``` python
from langchain.agents import create_agent

def check_weather(location: str) -> str:
    '''Return the weather forecast for the specified location.'''
    return f"It's always sunny in {location}"

graph = create_agent(
    model="anthropic:claude-sonnet-4-5-20250929",
    tools=[check_weather],
    system_prompt="You are a helpful assistant",
)
inputs = {"messages": [{"role": "user", "content": "what is the weather in sf"}]}
for chunk in graph.stream(inputs, stream_mode="updates"):
    print(chunk)
```

## Parameters
|Name|Type|Description|
|:---|:---|:---|
|model*|str \| BaseChatModel|The language model for the agent. <br> Can be a string identifier(标识符) (e.g., "openai:gpt-4") or a direct chat model instance (e.g., ChatOpenAI or other another LangChain chat model). <br> For a full list of supported model strings, see [init_chat_model](https://reference.langchain.com/python/langchain/chat_models/base/init_chat_model#model_provider).|
|tools|Sequence[BaseTool \| Callable[..., Any] \| dict[str, Any]] \| None|Default: None <br> A list of tools, dict, or Callable. <br> If None or an empty list, the agent will consist of a model node without a tool calling loop.|
|system_prompt|str \| SystemMessage \| None|Default: None <br> An optional system prompt for the LLM. <br> Can be a str (which will be converted to a SystemMessage) or a SystemMessage instance directly. The system message is added to the **beginning of the message list** when calling the model.|
|middleware|Sequence[AgentMiddleware[StateT_co, ContextT]]|Default: () <br> A sequence of middleware instances to apply to the agent. <br> Middleware can intercept(拦截) and modify agent behavior at various stages.|
|response_format|ResponseFormat[ResponseT] \| type[ResponseT] \| dict[str, Any] \| None|Default: None <br> An optional configuration for structured responses. <br> Can be a ToolStrategy, ProviderStrategy, or a Pydantic model class. <br> If provided, the agent will handle structured output during the conversation flow. <br> Raw schemas will be wrapped in an appropriate strategy based on model capabilities.|
|state_schema|type[AgentState[ResponseT]] \| None|Default: None <br> An optional TypedDict schema that extends AgentState. <br> When provided, this schema is used instead of AgentState **as the base schema for merging with middleware state schemas**. This allows users to **add custom state** fields without needing to create custom middleware. <br> Generally, it's recommended to use state_schema extensions via middleware to keep relevant extensions scoped to corresponding hooks / tools.|
|context_schema|type[ContextT] \| None|Default: None <br> An optional schema for runtime context.|
|checkpointer|Checkpointer \| None|Default: None <br> An optional checkpoint saver object. <br> Used for persisting the state of the graph (e.g., as chat memory) for a **single thread** (e.g., a single conversation).|
|store|BaseStore \| None|Default: None <br> An optional store object. <br> Used for persisting data **across multiple threads** (e.g., multiple conversations / users).|
|interrupt_before|list[str] \| None|Default: None <br> An optional list of node names to interrupt before. <br> Useful if you want to add a **user confirmation or other interrupt** before taking an action.|
|interrupt_after|list[str] \| None|Default: None <br> An optional list of node names to interrupt after. <br> Useful if you want to **return directly or run additional processing** on an output.|
|debug|bool|Default: False <br> Whether to enable verbose logging for graph execution. <br>When enabled, prints detailed information about each node execution, state updates, and transitions during agent runtime. Useful for debugging middleware behavior and understanding agent execution flow.|
|name|str \| None|Default: None <br> An optional name for the CompiledStateGraph. <br> This name will be automatically used when adding the agent graph to another graph as a **subgraph node** - particularly useful for building **multi-agent systems**.|
|cache|BaseCache[Any] \| None|Default: None <br> An optional BaseCache instance to enable caching of graph execution.|
