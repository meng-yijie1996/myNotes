# bind_tools
Bind tools to the model.

``` python
bind_tools(
  self,
  tools: Sequence[builtins.dict[str, Any] | type | Callable | BaseTool],
  *,
  tool_choice: str | None = None,
  **kwargs: Any = {}
) -> Runnable[LanguageModelInput, AIMessage]
```

## Parameters
|Name|Type|Description|
|:---|:---|:---|
|tools*|Sequence[builtins.dict[str, Any] \| type \| Callable \| BaseTool]|Sequence of tools to bind to the model.|
|tool_choice|str \| None|Default: None <br> The tool to use. If "any" then any tool can be used.|

---

在 LangChain 中，`bind_tools()` 是**给大模型绑定「工具调用能力」的核心方法**，本质是将「工具描述」封装成模型能识别的格式（如 OpenAI 函数调用规范），并绑定到模型实例上，让模型在生成响应时自动判断是否调用工具、调用哪个工具。

### 一、核心作用（通俗理解）
你可以把 `bind_tools()` 理解为：**给大模型「装备工具箱」** —— 告诉模型「你现在有这些工具可用，什么时候该用、怎么用」，后续调用模型时无需重复传入工具列表，简化工具调用流程。

### 二、核心特点 & 解决的问题
1. **一次绑定，多次复用**：
   无需在每次 `invoke()`/`stream()` 调用时都传 `tools` 参数，绑定后模型实例会记住工具列表；
2. **标准化工具格式**：
   自动将 LangChain 标准 `Tool` 对象转换为模型兼容的格式（如 OpenAI 函数调用格式、Anthropic/Claude 工具格式、百炼/Qwen 兼容格式）；
3. **强制/自动工具调用**：
   可配置模型「必须调用工具」「自动判断」或「禁止调用」，精准控制工具调用逻辑。

### 三、基础用法（结合百炼/Qwen 示例）
```python
import os
from langchain_community.chat_models import ChatDashScope
from langchain_core.tools import Tool

# 1. 定义工具
def get_weather(city: str) -> str:
    """Get weather for a given city."""
    return f"It's sunny in {city}!"

weather_tool = Tool(
    name="get_weather",
    func=get_weather,
    description="获取指定城市的天气信息，输入参数为城市名称（如sf、beijing）"
)

# 2. 初始化模型并绑定工具（核心：bind_tools）
os.environ["DASHSCOPE_API_KEY"] = "你的百炼API Key"
llm = ChatDashScope(model="qwen3.5-plus")

# 绑定工具到模型（一次绑定，后续复用）
llm_with_tools = llm.bind_tools(
    tools=[weather_tool],  # 要绑定的工具列表
    tool_choice="auto"     # 调用策略：auto=自动判断 / none=禁用 / {"name": "get_weather"}=强制调用指定工具
)

# 3. 调用模型（无需再传工具列表）
response = llm_with_tools.invoke("what is the weather in sf?")
print(response)
```

### 四、关键参数说明
| 参数         | 作用                                                                 |
|--------------|----------------------------------------------------------------------|
| `tools`      | 必传，工具列表（LangChain `Tool` 对象/工具描述字典）                  |
| `tool_choice`| 可选，工具调用策略：<br>- `"auto"`：模型自动判断是否调用（默认）<br>- `"none"`：禁用工具调用<br>- `{"name": "工具名"}`：强制调用指定工具 |
| `kwargs`     | 可选，额外参数（如给模型加温度 `temperature=0.1`）                   |

### 五、`bind_tools()` vs 直接传 `tools` 参数（对比）
| 方式                  | 优点                          | 缺点                          |
|-----------------------|-------------------------------|-------------------------------|
| `llm.bind_tools(tools)` | 一次绑定，多次复用；代码简洁  | 绑定后工具列表固定，无法动态改 |
| `llm.invoke(input, tools=[])` | 每次可传不同工具，灵活        | 重复传参，代码冗余            |

### 六、进阶场景：绑定工具后解析调用结果
绑定工具后，模型会返回包含「工具调用指令」的响应，需解析后执行工具：
```python
# 接上面的代码
from langchain_core.messages import ToolMessage

# 解析模型的工具调用指令
if hasattr(response, "tool_calls") and response.tool_calls:
    for tool_call in response.tool_calls:
        if tool_call["name"] == "get_weather":
            # 提取参数
            city = tool_call["args"]["city"]
            # 执行工具
            weather_result = get_weather(city)
            # 将结果回传给模型
            final_response = llm.invoke([
                {"role": "user", "content": "what is the weather in sf?"},
                response,  # 模型的工具调用指令
                ToolMessage(content=weather_result, tool_call_id=tool_call["id"])
            ])
            print("最终回答：", final_response.content)
```

### 总结
1. **核心功能**：给 LangChain 模型实例绑定工具列表，让模型具备工具调用能力；
2. **核心优势**：一次绑定、多次复用，自动标准化工具格式，简化调用逻辑；
3. **适用场景**：需要重复调用同一批工具的场景（如 Agent、智能助手）；
4. **关键注意**：绑定后模型仅「判断是否调用工具」，需手动解析并执行工具函数，再将结果回传给模型。

`bind_tools()` 是 LangChain 工具调用的「极简模式」，比手动构造工具调用格式、重复传参更高效，是学习 LangChain 工具调用的核心方法。
