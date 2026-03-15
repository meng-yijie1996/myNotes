# ChatOpenAI integration
You can find information about OpenAI’s latest models, their costs, context windows, and supported input types in the OpenAI Platform docs.
> 你可以在OpenAI平台文档中找到关于OpenAI最新模型、其成本、上下文窗口以及支持的输入类型的信息。

##### API Reference
For detailed documentation of all features and configuration options, head to the [ChatOpenAI](https://reference.langchain.com/python/langchain-openai/chat_models/base/ChatOpenAI?_gl=1*tydokn*_gcl_au*Mzc5MTc3ODg5LjE3NzEwNTk5ODM.*_ga*MTg3MTg2ODE2NS4xNzcxMDU5OTg0*_ga_47WX3HKKY2*czE3NzM1ODYwNTEkbzE0JGcwJHQxNzczNTg2MDUxJGo2MCRsMCRoMA..) API reference.
> 有关所有功能和配置选项的详细文档，请查阅ChatOpenAI API参考。

##### API scope
ChatOpenAI targets [official OpenAI API specifications](https://github.com/openai/openai-openapi) only. Non-standard response fields from third-party providers (e.g., reasoning_content, reasoning, reasoning_details) are not extracted or preserved. If you are using a provider that extends the Chat Completions or Responses formats, such as OpenRouter, LiteLLM, vLLM, or DeepSeek, use a provider-specific package instead. See Chat Completions API compatibility for details.
> ChatOpenAI仅以OpenAI官方API规范为目标。第三方提供商的非标准响应字段（例如reasoning_content、reasoning、reasoning_details）不会被提取或保留。如果您使用的提供商扩展了聊天补全或响应格式，例如OpenRouter、LiteLLM、vLLM或DeepSeek<，请改用特定于提供商的包。有关详细信息，请参见聊天补全API兼容性。
​
## Overview
### Integration details

|Class|Package|Serializable|JS/TS Support|Downloads|Latest Version|
|:---|:---|:---|:---|:---|:---|
|ChatOpenAI|langchain-openai|beta|✅ (npm)|[Downloads](https://pypi.org/project/langchain-openai/)|[v1.1.11](https://pypi.org/project/langchain-openai/)|

​
### Model features
|Tool calling|Structured output|Image input|Audio input|Video input|Token-level streaming|Native async|Token usage|Logprobs|
|:---|:---|:---|:---|:---|:---|:---|:---|:---|
|✅|✅|✅|✅|❌|✅|✅|✅|✅|​

## Setup
To access OpenAI models you’ll need to install the langchain-openai integration package and acquire an OpenAI Platform API key.
要使用OpenAI模型，你需要安装langchain-openai集成包并获取一个OpenAI平台的API密钥。
​
Installation 安装

pip

uv
pip install -U langchain-openai
​
Credentials 凭证
Head to the OpenAI Platform to sign up and generate an API key. Once you’ve done this set the OPENAI_API_KEY environment variable in your environment:
前往OpenAI平台注册并生成API密钥。完成后，请在你的环境中设置OPENAI_API_KEY环境变量：
import getpass
import os

if not os.environ.get("OPENAI_API_KEY"):
    os.environ["OPENAI_API_KEY"] = getpass.getpass("Enter your OpenAI API key: ")
If you want to get automated tracing of your model calls you can also set your LangSmith API key:
如果您希望自动追踪模型调用，还可以设置您的LangSmith API密钥：
os.environ["LANGSMITH_API_KEY"] = getpass.getpass("Enter your LangSmith API key: ")
os.environ["LANGSMITH_TRACING"] = "true"
​
Instantiation 实例化
Now we can instantiate our model object and generate responses:
现在我们可以实例化模型对象并生成响应：
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="gpt-5-nano",
    # stream_usage=True,
    # temperature=None,
    # max_tokens=None,
    # timeout=None,
    # reasoning_effort="low",
    # max_retries=2,
    # api_key="...",  # If you prefer to pass api key in directly
    # base_url="...",
    # organization="...",
    # other params...
)
See the ChatOpenAI API Reference for the full set of available model parameters.
有关所有可用的模型参数，请参阅ChatOpenAI API参考。
Token parameter deprecation 令牌参数弃用
OpenAI deprecated max_tokens in favor of max_completion_tokens in September 2024. While max_tokens is still supported for backwards compatibility, it’s automatically converted to max_completion_tokens internally.
OpenAI于2024年9月弃用了max_tokens，转而采用max_completion_tokens。虽然为了向后兼容，max_tokens仍受支持，但它在内部会自动转换为max_completion_tokens。
​
Invocation 调用
messages = [
    (
        "system",
        "You are a helpful assistant that translates English to French. Translate the user sentence.",
    ),
    ("human", "I love programming."),
]
ai_msg = llm.invoke(messages)
ai_msg
AIMessage(content="J'adore la programmation.", additional_kwargs={'refusal': None}, response_metadata={'token_usage': {'completion_tokens': 5, 'prompt_tokens': 31, 'total_tokens': 36}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_3aa7262c27', 'finish_reason': 'stop', 'logprobs': None}, id='run-63219b22-03e3-4561-8cc4-78b7c7c3a3ca-0', usage_metadata={'input_tokens': 31, 'output_tokens': 5, 'total_tokens': 36})
print(ai_msg.text)
J'adore la programmation.
​
Streaming usage metadata 流式使用元数据
OpenAI’s Chat Completions API does not stream token usage statistics by default (see the OpenAI API reference for stream options).
OpenAI的聊天补全API默认不会流式传输令牌使用统计信息（请参阅OpenAI API中关于流式选项的参考）。
To recover token counts when streaming with ChatOpenAI or AzureChatOpenAI, set stream_usage=True as an initialization parameter or on invocation:
使用ChatOpenAI或AzureChatOpenAI进行流式传输时，若要恢复令牌计数，请在初始化参数中或调用时设置stream_usage=True：
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4.1-mini", stream_usage=True)
​
Using with Azure OpenAI 与Azure OpenAI一起使用
Azure OpenAI v1 API support Azure OpenAI v1 API 支持
As of langchain-openai>=1.0.1, ChatOpenAI can be used directly with Azure OpenAI endpoints using the new v1 API. This provides a unified way to use OpenAI models whether hosted on OpenAI or Azure.
自langchain-openai>=1.0.1起，ChatOpenAI可通过新的v1 API直接与Azure OpenAI端点配合使用。这提供了一种统一的方式来使用OpenAI模型，无论这些模型托管在OpenAI还是Azure上。
For the traditional Azure-specific implementation, continue to use AzureChatOpenAI.
对于传统的Azure特定实现，请继续使用AzureChatOpenAI。
Using Azure OpenAI v1 API with API Key
使用 Azure OpenAI v1 API 与 API 密钥

Using Azure OpenAI with Microsoft Entra ID
将Azure OpenAI与Microsoft Entra ID结合使用

​
Tool calling 工具调用
OpenAI has a tool calling (we use “tool calling” and “function calling” interchangeably here) API that lets you describe tools and their arguments, and have the model return a JSON object with a tool to invoke and the inputs to that tool. tool-calling is extremely useful for building tool-using chains and agents, and for getting structured outputs from models more generally.
OpenAI有一个工具调用（在这里，我们交替使用“工具调用”和“函数调用”）API，它允许你描述工具及其参数，并让模型返回一个JSON对象，其中包含要调用的工具以及该工具的输入。工具调用对于构建使用工具的链和智能体，以及更广泛地从模型中获取结构化输出都非常有用。
​
Bind tools 绑定工具
With ChatOpenAI.bind_tools, we can easily pass in Pydantic classes, dict schemas, LangChain tools, or even functions as tools to the model. Under the hood these are converted to an OpenAI tool schemas, which looks like:
通过ChatOpenAI.bind_tools，我们可以轻松地将Pydantic类、字典模式、LangChain工具甚至函数作为工具传递给模型。在底层，这些会被转换为OpenAI工具模式，如下所示：
{
    "name": "...",
    "description": "...",
    "parameters": {...}  # JSONSchema
}
…and are passed in every model invocation.
……并在每次模型调用时传入。
from pydantic import BaseModel, Field


class GetWeather(BaseModel):
    """Get the current weather in a given location"""

    location: str = Field(description="The city and state, e.g. San Francisco, CA")


llm_with_tools = llm.bind_tools([GetWeather])
ai_msg = llm_with_tools.invoke(
    "what is the weather like in San Francisco",
)
ai_msg
AIMessage(content='', additional_kwargs={'refusal': None}, response_metadata={'token_usage': {'completion_tokens': 17, 'prompt_tokens': 68, 'total_tokens': 85}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_3aa7262c27', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-1617c9b2-dda5-4120-996b-0333ed5992e2-0', tool_calls=[{'name': 'GetWeather', 'args': {'location': 'San Francisco, CA'}, 'id': 'call_o9udf3EVOWiV4Iupktpbpofk', 'type': 'tool_call'}], usage_metadata={'input_tokens': 68, 'output_tokens': 17, 'total_tokens': 85})
​
Strict mode 严格模式
Requires langchain-openai>=0.1.21 需要 langchain-openai>=0.1.21
As of Aug 6, 2024, OpenAI supports a strict argument when calling tools that will enforce that the tool argument schema is respected by the model. See more.
截至2024年8月6日，OpenAI在调用工具时支持一个strict参数，该参数将确保模型遵守工具的参数 schema。查看更多。
If strict=True the tool definition will also be validated, and a subset of JSON schema are accepted. Crucially, schema cannot have optional args (those with default values).
如果strict=True，工具定义也将被验证，并且仅接受一部分JSON模式。关键是，模式不能包含可选参数（那些带有默认值的参数）。
Read the full docs on what types of schema are supported.
阅读关于支持哪些类型的模式的完整文档。
llm_with_tools = llm.bind_tools([GetWeather], strict=True)
ai_msg = llm_with_tools.invoke(
    "what is the weather like in San Francisco",
)
ai_msg
AIMessage(content='', additional_kwargs={'tool_calls': [{'id': 'call_jUqhd8wzAIzInTJl72Rla8ht', 'function': {'arguments': '{"location":"San Francisco, CA"}', 'name': 'GetWeather'}, 'type': 'function'}], 'refusal': None}, response_metadata={'token_usage': {'completion_tokens': 17, 'prompt_tokens': 68, 'total_tokens': 85}, 'model_name': 'gpt-4o-2024-05-13', 'system_fingerprint': 'fp_3aa7262c27', 'finish_reason': 'tool_calls', 'logprobs': None}, id='run-5e3356a9-132d-4623-8e73-dd5a898cf4a6-0', tool_calls=[{'name': 'GetWeather', 'args': {'location': 'San Francisco, CA'}, 'id': 'call_jUqhd8wzAIzInTJl72Rla8ht', 'type': 'tool_call'}], usage_metadata={'input_tokens': 68, 'output_tokens': 17, 'total_tokens': 85})
​
Tool calls 工具调用
Notice that the AIMessage has a tool_calls attribute. This contains in a standardized ToolCall format that is model-provider agnostic.
请注意，AIMessage 具有 tool_calls 属性。它包含一种标准化的 ToolCall 格式，这种格式与模型提供商无关。
ai_msg.tool_calls
[{'name': 'GetWeather',
  'args': {'location': 'San Francisco, CA'},
  'id': 'call_jUqhd8wzAIzInTJl72Rla8ht',
  'type': 'tool_call'}]
For more on binding tools and tool call outputs, head to the tool calling docs.
有关绑定工具和工具调用输出的更多信息，请参阅工具调用文档。
​
Custom tools 自定义工具
Requires langchain-openai>=0.3.29 需要 langchain-openai>=0.3.29
Custom tools support tools with arbitrary string inputs. They can be particularly useful when you expect your string arguments to be long or complex.
自定义工具支持具有任意字符串输入的工具。当您期望字符串参数较长或较复杂时，它们会特别有用。
from langchain_openai import ChatOpenAI, custom_tool
from langchain.agents import create_agent


@custom_tool
def execute_code(code: str) -> str:
    """Execute python code."""
    return "27"


llm = ChatOpenAI(model="gpt-5", use_responses_api=True)

agent = create_agent(llm, [execute_code])

input_message = {"role": "user", "content": "Use the tool to calculate 3^3."}
for step in agent.stream(
    {"messages": [input_message]},
    stream_mode="values",
):
    step["messages"][-1].pretty_print()
================================ Human Message =================================

Use the tool to calculate 3^3.
================================== Ai Message ==================================

[{'id': 'rs_68b7336cb72081a080da70bf5e980e4e0d6082d28f91357a', 'summary': [], 'type': 'reasoning'}, {'call_id': 'call_qyKsJ4XlGRudbIJDrXVA2nQa', 'input': 'print(3**3)', 'name': 'execute_code', 'type': 'custom_tool_call', 'id': 'ctc_68b7336f718481a0b39584cd35fbaa5d0d6082d28f91357a', 'status': 'completed'}]
Tool Calls:
  execute_code (call_qyKsJ4XlGRudbIJDrXVA2nQa)
 Call ID: call_qyKsJ4XlGRudbIJDrXVA2nQa
  Args:
    __arg1: print(3**3)
================================= Tool Message =================================
Name: execute_code

[{'type': 'custom_tool_call_output', 'output': '27'}]
================================== Ai Message ==================================

[{'type': 'text', 'text': '27', 'annotations': [], 'id': 'msg_68b73371e9e081a0927f54f88f2cd7a20d6082d28f91357a'}]
Context-free grammars 无上下文语法

​
Structured output 结构化输出
OpenAI supports a native structured output feature, which guarantees that its responses adhere to a given schema.
OpenAI支持原生的结构化输出功能，该功能确保其响应符合给定的模式。
You can access this feature in individual model calls, or by specifying the response format of a LangChain agent. See below for examples.
你可以在单独的模型调用中使用此功能，或者通过指定LangChain智能体的响应格式来使用。下面是一些示例。
Individual model calls 单个模型调用

Agent response format 智能体响应格式

​
Structured output with tool calls 带有工具调用的结构化输出
OpenAI’s structured output feature can be used simultaneously with tool-calling. The model will either generate tool calls or a response adhering to a desired schema. See example below:
OpenAI的结构化输出功能可以与工具调用同时使用。模型将生成工具调用，或者生成符合所需模式的响应。请看下面的示例：
from langchain_openai import ChatOpenAI
from pydantic import BaseModel


def get_weather(location: str) -> None:
    """Get weather at a location."""
    return "It's sunny."


class OutputSchema(BaseModel):
    """Schema for response."""

    answer: str
    justification: str


llm = ChatOpenAI(model="gpt-4.1")

structured_llm = llm.bind_tools(
    [get_weather],
    response_format=OutputSchema,
    strict=True,
)

# Response contains tool calls:
tool_call_response = structured_llm.invoke("What is the weather in SF?")

# structured_response.additional_kwargs["parsed"] contains parsed output
structured_response = structured_llm.invoke(
    "What weighs more, a pound of feathers or a pound of gold?"
)
​
Responses API 响应API
Requires langchain-openai>=0.3.9 需要 langchain-openai>=0.3.9
OpenAI supports a Responses API that is oriented toward building agentic applications. It includes a suite of built-in tools, including web and file search. It also supports management of conversation state, allowing you to continue a conversational thread without explicitly passing in previous messages, as well as the output from reasoning processes.
OpenAI支持一个面向构建智能体应用的响应API。它包含一套内置工具，包括网络和文件搜索功能。它还支持对对话状态的管理，使你能够继续对话线程而无需显式传入之前的消息，以及推理过程的输出。
ChatOpenAI will route to the Responses API if one of these features is used. You can also specify use_responses_api=True when instantiating ChatOpenAI.
ChatOpenAI 如果使用了这些功能之一，将路由到响应 API。在实例化 ChatOpenAI 时，你也可以指定 use_responses_api=True。
​
Web search 网页搜索
To trigger a web search, pass {"type": "web_search_preview"} to the model as you would another tool.
要触发网页搜索，请像使用其他工具一样，向模型传递{"type": "web_search_preview"}。
You can also pass built-in tools as invocation params:
你也可以将内置工具作为调用参数传递：
llm.invoke("...", tools=[{"type": "web_search_preview"}])
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4.1-mini")

tool = {"type": "web_search_preview"}
llm_with_tools = llm.bind_tools([tool])

response = llm_with_tools.invoke("What was a positive news story from today?")
Note that the response includes structured content blocks that include both the text of the response and OpenAI annotations citing its sources. The output message will also contain information from any tool invocations:
请注意，该回复包含结构化的内容块，其中既包括回复文本，也包括引用其来源的OpenAI 注释。输出消息还将包含来自任何工具调用的信息：
response.content_blocks
[{'type': 'server_tool_call',
  'name': 'web_search',
  'args': {'query': 'positive news stories today', 'type': 'search'},
  'id': 'ws_68cd6f8d72e4819591dab080f4b0c340080067ad5ea8144a'},
 {'type': 'server_tool_result',
  'tool_call_id': 'ws_68cd6f8d72e4819591dab080f4b0c340080067ad5ea8144a',
  'status': 'success'},
 {'type': 'text',
  'text': 'Here are some positive news stories from today...',
  'annotations': [{'end_index': 410,
    'start_index': 337,
    'title': 'Positive News | Real Stories. Real Positive Impact',
    'type': 'citation',
    'url': 'https://www.positivenews.press/?utm_source=openai'},
   {'end_index': 969,
    'start_index': 798,
    'title': "From Green Innovation to Community Triumphs: Uplifting US Stories Lighting Up September 2025 | That's Great News",
    'type': 'citation',
    'url': 'https://info.thatsgreatnews.com/from-green-innovation-to-community-triumphs-uplifting-us-stories-lighting-up-september-2025/?utm_source=openai'},
  'id': 'msg_68cd6f8e8d448195a807b89f483a1277080067ad5ea8144a'}]
You can recover just the text content of the response as a string by using response.text. For example, to stream response text:
你可以通过使用response.text将响应的纯文本内容作为字符串进行恢复。例如，要流式传输响应文本：
for token in llm_with_tools.stream("..."):
    print(token.text, end="|")
See the streaming guide for more detail.
有关更多详细信息，请参阅流式传输指南。
​
Image generation 图像生成
Requires langchain-openai>=0.3.19 需要 langchain-openai>=0.3.19
To trigger an image generation, pass {"type": "image_generation"} to the model as you would another tool.
要触发图像生成，可像使用其他工具一样，向模型传递{"type": "image_generation"}。
You can also pass built-in tools as invocation params:
你也可以将内置工具作为调用参数传递：
llm.invoke("...", tools=[{"type": "image_generation"}])
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4.1-mini")

tool = {"type": "image_generation", "quality": "low"}

llm_with_tools = llm.bind_tools([tool])

ai_message = llm_with_tools.invoke(
    "Draw a picture of a cute fuzzy cat with an umbrella"
)
import base64

from IPython.display import Image

image = next(
    item for item in ai_message.content_blocks if item["type"] == "image"
)
Image(base64.b64decode(image["base64"]), width=200)


​
File search 文件搜索
To trigger a file search, pass a file search tool to the model as you would another tool. You will need to populate an OpenAI-managed vector store and include the vector store ID in the tool definition. See OpenAI documentation for more detail.
要触发文件搜索，可像使用其他工具一样，向模型传递一个文件搜索工具。你需要填充一个由OpenAI管理的向量存储，并在工具定义中包含该向量存储的ID。更多详情请参阅OpenAI文档。
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="gpt-4.1-mini",
    include=["file_search_call.results"],  # optionally include search results
)

openai_vector_store_ids = [
    "vs_...",  # your IDs here
]

tool = {
    "type": "file_search",
    "vector_store_ids": openai_vector_store_ids,
}
llm_with_tools = llm.bind_tools([tool])

response = llm_with_tools.invoke("What is deep research by OpenAI?")
print(response.text)
Deep Research by OpenAI is...
As with web search, the response will include content blocks with citations:
与网络搜索一样，响应将包含带有引用的内容块：
[block["type"] for block in response.content_blocks]
['server_tool_call', 'server_tool_result', 'text']
text_block = next(block for block in response.content_blocks if block["type"] == "text")

text_block["annotations"][:2]
[{'type': 'citation',
  'title': 'deep_research_blog.pdf',
  'extras': {'file_id': 'file-3UzgX7jcC8Dt9ZAFzywg5k', 'index': 2712}},
 {'type': 'citation',
  'title': 'deep_research_blog.pdf',
  'extras': {'file_id': 'file-3UzgX7jcC8Dt9ZAFzywg5k', 'index': 2712}}]
It will also include information from the built-in tool invocations:
它还将包含来自内置工具调用的信息：
response.content_blocks[0]
{'type': 'server_tool_call',
 'name': 'file_search',
 'id': 'fs_68cd704c191c81959281b3b2ec6b139908f8f7fb31b1123c',
 'args': {'queries': ['deep research by OpenAI']}}
​
Tool search 工具搜索
Requires langchain-openai>=1.1.11 需要 langchain-openai>=1.1.11
OpenAI supports a tool search feature allowing models to search for and load tools into its context as needed. OpenAI injects the retrieved tool definitions at the end of the active context to preserve its cache.
OpenAI支持一项工具搜索功能，允许模型根据需要搜索工具并将其加载到自身语境中。OpenAI会将检索到的工具定义注入到活动语境的末尾，以保留其缓存。
To engage tool search, mark tools with @tool(extras={"defer_loading": True}) and add OpenAI’s search tool to available tools. See below for examples.
要启用工具搜索，请用@tool(extras={"defer_loading": True})标记工具，并将OpenAI的搜索工具添加到可用工具中。示例如下。
Server-side tool search 服务器端工具搜索

Client-executed tool search 客户端执行的工具搜索

​
Computer use 计算机使用
ChatOpenAI supports the "computer-use-preview" model, which is a specialized model for the built-in computer use tool. To enable, pass a computer use tool as you would pass another tool.
ChatOpenAI支持"computer-use-preview"模型，这是一个用于内置计算机使用工具的专用模型。要启用该模型，请像传递其他工具一样传递一个计算机使用工具。
Currently, tool outputs for computer use are present in the message content field. To reply to the computer use tool call, construct a ToolMessage with {"type": "computer_call_output"} in its additional_kwargs. The content of the message will be a screenshot. Below, we demonstrate a simple example.
目前，计算机使用的工具输出位于消息的content字段中。要回复计算机使用工具调用，请构建一个ToolMessage，并在其additional_kwargs中包含{"type": "computer_call_output"}。该消息的内容将是一张截图。下面，我们展示一个简单的示例。
First, load two screenshots: 首先，加载两张截图：
import base64


def load_png_as_base64(file_path):
    with open(file_path, "rb") as image_file:
        encoded_string = base64.b64encode(image_file.read())
        return encoded_string.decode("utf-8")


screenshot_1_base64 = load_png_as_base64(
    "/path/to/screenshot_1.png"
)  # perhaps a screenshot of an application
screenshot_2_base64 = load_png_as_base64(
    "/path/to/screenshot_2.png"
)  # perhaps a screenshot of the Desktop
from langchain_openai import ChatOpenAI

# Initialize model
llm = ChatOpenAI(model="computer-use-preview", truncation="auto")

# Bind computer-use tool
tool = {
    "type": "computer_use_preview",
    "display_width": 1024,
    "display_height": 768,
    "environment": "browser",
}
llm_with_tools = llm.bind_tools([tool])

# Construct input message
input_message = {
    "role": "user",
    "content": [
        {
            "type": "text",
            "text": (
                "Click the red X to close and reveal my Desktop. "
                "Proceed, no confirmation needed."
            ),
        },
        {
            "type": "input_image",
            "image_url": f"data:image/png;base64,{screenshot_1_base64}",
        },
    ],
}

# Invoke model
response = llm_with_tools.invoke(
    [input_message],
    reasoning={
        "generate_summary": "concise",
    },
)
The response will include a call to the computer-use tool in its content:
响应将在其content中包含对计算机使用工具的调用：
response.content
[{'id': 'rs_685da051742c81a1bb35ce46a9f3f53406b50b8696b0f590',
  'summary': [{'text': "Clicking red 'X' to show desktop",
    'type': 'summary_text'}],
  'type': 'reasoning'},
 {'id': 'cu_685da054302481a1b2cc43b56e0b381706b50b8696b0f590',
  'action': {'button': 'left', 'type': 'click', 'x': 14, 'y': 38},
  'call_id': 'call_zmQerFBh4PbBE8mQoQHkfkwy',
  'pending_safety_checks': [],
  'status': 'completed',
  'type': 'computer_call'}]
We next construct a ToolMessage with these properties:
接下来，我们构建一个具有以下属性的ToolMessage：
It has a tool_call_id matching the call_id from the computer-call.
它有一个与计算机调用中的call_id相匹配的tool_call_id。
It has {"type": "computer_call_output"} in its additional_kwargs.
其在additional_kwargs中包含{"type": "computer_call_output"}。
Its content is either an image_url or an input_image output block (see OpenAI docs for formatting).
其内容要么是一个image_url，要么是一个input_image输出块（格式请参见OpenAI文档）。
from langchain.messages import ToolMessage

tool_call_id = next(
    item["call_id"] for item in response.content if item["type"] == "computer_call"
)

tool_message = ToolMessage(
    content=[
        {
            "type": "input_image",
            "image_url": f"data:image/png;base64,{screenshot_2_base64}",
        }
    ],
    # content=f"data:image/png;base64,{screenshot_2_base64}",  # <-- also acceptable
    tool_call_id=tool_call_id,
    additional_kwargs={"type": "computer_call_output"},
)
We can now invoke the model again using the message history:
我们现在可以使用消息历史再次调用该模型：
messages = [
    input_message,
    response,
    tool_message,
]

response_2 = llm_with_tools.invoke(
    messages,
    reasoning={
        "generate_summary": "concise",
    },
)
response_2.text
'VS Code has been closed, and the desktop is now visible.'
Instead of passing back the entire sequence, we can also use the previous_response_id:
我们也可以使用previous_response_id，而不是返回整个序列：
previous_response_id = response.response_metadata["id"]

response_2 = llm_with_tools.invoke(
    [tool_message],
    previous_response_id=previous_response_id,
    reasoning={
        "generate_summary": "concise",
    },
)
response_2.text
'The VS Code window is closed, and the desktop is now visible. Let me know if you need any further assistance.'
​
Code interpreter 代码解释器
OpenAI implements a code interpreter tool to support the sandboxed generation and execution of code.
OpenAI 实施了一个代码解释器工具，以支持代码的沙盒式生成和执行。
Example use 示例用法
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="gpt-4.1-mini",
    include=["code_interpreter_call.outputs"],  # optionally include outputs
)

llm_with_tools = llm.bind_tools(
    [
        {
            "type": "code_interpreter",
            # Create a new container
            "container": {"type": "auto"},
        }
    ]
)
response = llm_with_tools.invoke(
    "Write and run code to answer the question: what is 3^3?"
)
Note that the above command created a new container. We can also specify an existing container ID:
请注意，上述命令创建了一个新容器。我们也可以指定一个现有的容器ID：
code_interpreter_calls = [
    item for item in response.content if item["type"] == "code_interpreter_call"
]
assert len(code_interpreter_calls) == 1
container_id = code_interpreter_calls[0]["extras"]["container_id"]

llm_with_tools = llm.bind_tools(
    [
        {
            "type": "code_interpreter",
            # Use an existing container
            "container": container_id,
        }
    ]
)
​
Remote MCP 远程MCP
OpenAI implements a remote MCP tool that allows for model-generated calls to MCP servers.
OpenAI 实现了一个 远程 MCP 工具，该工具允许模型生成对 MCP 服务器的调用。
Example use 示例用法
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4.1-mini")

llm_with_tools = llm.bind_tools(
    [
        {
            "type": "mcp",
            "server_label": "deepwiki",
            "server_url": "https://mcp.deepwiki.com/mcp",
            "require_approval": "never",
        }
    ]
)
response = llm_with_tools.invoke(
    "What transport protocols does the 2025-03-26 version of the MCP "
    "spec (modelcontextprotocol/modelcontextprotocol) support?"
)
MCP Approvals MCP 审批

​
Managing conversation state 管理对话状态
The Responses API supports management of conversation state.
Responses API 支持对对话状态的管理。
​
Manually manage state 手动管理状态
You can manage the state manually or using LangGraph, as with other chat models:
你可以手动管理状态，也可以像使用其他聊天模型一样，使用LangGraph来管理状态：
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4.1-mini", use_responses_api=True)

first_query = "Hi, I'm Bob."
messages = [{"role": "user", "content": first_query}]

response = llm.invoke(messages)
print(response.text)
Hi Bob! Nice to meet you. How can I assist you today?
second_query = "What is my name?"

messages.extend(
    [
        response,
        {"role": "user", "content": second_query},
    ]
)
second_response = llm.invoke(messages)
print(second_response.text)
You mentioned that your name is Bob. How can I assist you further, Bob?
You can use LangGraph to manage conversational threads for you in a variety of backends, including in-memory and Postgres. See this tutorial to get started.
你可以使用LangGraph在多种后端中为你管理对话线程，包括内存和Postgres。查看本教程开始使用。
​
Passing previous_response_id 传递previous_response_id
When using the Responses API, LangChain messages will include an "id" field in its metadata. Passing this ID to subsequent invocations will continue the conversation. Note that this is equivalent to manually passing in messages from a billing perspective.
使用Responses API时，LangChain消息的元数据中将包含一个"id"字段。将此ID传递给后续调用将继续对话。请注意，从计费角度而言，这与手动传入消息是等效的。
second_response = llm.invoke(
    "What is my name?",
    previous_response_id=response.id,
)
print(second_response.text)
Your name is Bob. How can I help you today, Bob?
ChatOpenAI can also automatically specify previous_response_id using the last response in a message sequence:
ChatOpenAI 还可以使用消息序列中的最后一个响应自动指定 previous_response_id：
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="gpt-4.1-mini",
    use_previous_response_id=True,
)
If we set use_previous_response_id=True, input messages up to the most recent response will be dropped from request payloads, and previous_response_id will be set using the ID of the most recent response.
如果我们设置use_previous_response_id=True，那么直到最近一次响应的输入消息将从请求负载中删除，并且previous_response_id将使用最近一次响应的ID来设置。
That is, 也就是说，
llm.invoke(
    [
        HumanMessage("Hello"),
        AIMessage("Hi there!", id="resp_123"),
        HumanMessage("How are you?"),
    ]
)
…is equivalent to: ……等价于：
llm.invoke([HumanMessage("How are you?")], previous_response_id="resp_123")
​
Context management 上下文管理
The Responses API supports automatic server-side context compaction. This reduces conversation size when it reaches a token threshold, allowing for support of long-running interactions:
Responses API支持自动的服务器端上下文压缩。当对话达到令牌阈值时，这会减小对话规模，从而支持长时间运行的交互：
from langchain_openai import ChatOpenAI

model = ChatOpenAI(
    model="gpt-5.2",
    context_management=[
        {"type": "compaction", "compact_threshold": 100_000}
    ],
)
When enabled, AIMessage responses may contain blocks with "type": "compaction" in content. These should be retained in the conversation history, and can be appended to the message sequence in the usual way. Messages prior to the most recent compaction item can be kept, or discarded to improve latency.
启用后，AIMessage的响应在内容中可能包含带有"type": "compaction"的区块。这些区块应保留在对话历史中，并且可以按常规方式添加到消息序列中。最近的compaction项之前的消息可以保留，也可以丢弃以减少延迟。
​
Reasoning output 推理输出
Some OpenAI models will generate separate text content illustrating their reasoning process. See OpenAI’s reasoning documentation for details.
一些OpenAI模型会生成单独的文本内容来说明其推理过程。详情请参见OpenAI的推理文档。
OpenAI can return a summary of the model’s reasoning (although it doesn’t expose the raw reasoning tokens). To configure ChatOpenAI to return this summary, specify the reasoning parameter. ChatOpenAI will automatically route to the Responses API if this parameter is set.
OpenAI 可以返回模型推理的摘要（尽管它不会暴露原始的推理令牌）。要配置ChatOpenAI以返回此摘要，请指定reasoning参数。如果设置了此参数，ChatOpenAI将自动路由到 Responses API。
from langchain_openai import ChatOpenAI

reasoning = {
    "effort": "medium",  # 'low', 'medium', or 'high'
    "summary": "auto",  # 'detailed', 'auto', or None
}

llm = ChatOpenAI(model="gpt-5-nano", reasoning=reasoning)
response = llm.invoke("What is 3^3?")

# Output
response.text
'3³ = 3 × 3 × 3 = 27.'
# Reasoning
for block in response.content_blocks:
    if block["type"] == "reasoning":
        print(block["reasoning"])
**Calculating the power of three**

The user is asking about 3 raised to the power of 3. That's a pretty simple calculation! I know that 3^3 equals 27, so I can say, "3 to the power of 3 equals 27." I might also include a quick explanation that it's 3 multiplied by itself three times: 3 × 3 × 3 = 27. So, the answer is definitely 27.
Troubleshooting: Empty responses from reasoning models
故障排除：推理模型返回空响应
If you’re getting empty responses from reasoning models like gpt-5-nano, this is likely due to restrictive token limits. The model uses tokens for internal reasoning and may not have any left for the final output.
如果你从像gpt-5-nano这样的推理模型那里得到空白响应，这很可能是由于严格的token限制。该模型会将token用于内部推理，可能没有剩余的token来生成最终输出。
Ensure max_tokens is set to None or increase the token limit to allow sufficient tokens for both reasoning and output generation.
确保将max_tokens设置为None，或者提高令牌限制，以便为推理和输出生成提供足够的令牌。
​
Fine-tuning 微调
You can call fine-tuned OpenAI models by passing in your corresponding modelName parameter.
您可以通过传入相应的modelName参数来调用微调后的OpenAI模型。
This generally takes the form of ft:{OPENAI_MODEL_NAME}:{ORG_NAME}::{MODEL_ID}. For example:
这通常采用ft:{OPENAI_MODEL_NAME}:{ORG_NAME}::{MODEL_ID}的形式。例如：
fine_tuned_model = ChatOpenAI(
    temperature=0, model_name="ft:gpt-3.5-turbo-0613:langchain::7qTVM5AR"
)

fine_tuned_model.invoke(messages)
AIMessage(content="J'adore la programmation.", additional_kwargs={'refusal': None}, response_metadata={'token_usage': {'completion_tokens': 8, 'prompt_tokens': 31, 'total_tokens': 39}, 'model_name': 'ft:gpt-3.5-turbo-0613:langchain::7qTVM5AR', 'system_fingerprint': None, 'finish_reason': 'stop', 'logprobs': None}, id='run-0f39b30e-c56e-4f3b-af99-5c948c984146-0', usage_metadata={'input_tokens': 31, 'output_tokens': 8, 'total_tokens': 39})
​
Multimodal inputs (images, PDFs, audio)
多模态输入（图像、PDF、音频）
OpenAI has models that support multimodal inputs. You can pass in images, PDFs, or audio to these models. For more information on how to do this in LangChain, head to the multimodal inputs docs.
OpenAI拥有支持多模态输入的模型。你可以向这些模型传入图像、PDF或音频。有关如何在LangChain中实现此操作的更多信息，请查阅多模态输入文档。
You can see the list of models that support different modalities in OpenAI’s documentation.
你可以在OpenAI的文档中查看支持不同模态的模型列表。
For all modalities, LangChain supports both its cross-provider standard as well as OpenAI’s native content-block format.
对于所有模态，LangChain既支持其跨提供商标准，也支持OpenAI的原生内容块格式。
To pass multimodal data into ChatOpenAI, create a content block containing the data and incorporate it into a message, e.g., as below:
要将多模态数据传入ChatOpenAI，请创建一个包含该数据的内容块并将其纳入消息中，例如如下所示：
message = {
    "role": "user",
    "content": [
        {
            "type": "text",
            # Update prompt as desired
            "text": "Describe the (image / PDF / audio...)",
        },
        content_block,
    ],
}
See below for examples of content blocks.
以下是内容块的示例。
Images 图像

PDFs PDF文档

Audio 音频

​
Predicted output 预测输出
Requires langchain-openai>=0.2.6 需要 langchain-openai>=0.2.6
Some OpenAI models (such as their gpt-4o and gpt-4o-mini series) support Predicted Outputs, which allow you to pass in a known portion of the LLM’s expected output ahead of time to reduce latency. This is useful for cases such as editing text or code, where only a small part of the model’s output will change.
一些OpenAI模型（例如其gpt-4o和gpt-4o-mini系列）支持预测输出功能，该功能允许你提前传入大语言模型预期输出的已知部分，以减少延迟。这在诸如编辑文本或代码等场景中非常有用，因为在这些场景中，模型输出中只有一小部分会发生变化。
Here’s an example: 以下是一个示例：
code = """
/// <summary>
/// Represents a user with a first name, last name, and username.
/// </summary>
public class User
{
    /// <summary>
    /// Gets or sets the user's first name.
    /// </summary>
    public string FirstName { get; set; }

    /// <summary>
    /// Gets or sets the user's last name.
    /// </summary>
    public string LastName { get; set; }

    /// <summary>
    /// Gets or sets the user's username.
    /// </summary>
    public string Username { get; set; }
}
"""

llm = ChatOpenAI(model="gpt-4.1")
query = (
    "Replace the Username property with an Email property. "
    "Respond only with code, and with no markdown formatting."
)
response = llm.invoke(
    [{"role": "user", "content": query}, {"role": "user", "content": code}],
    prediction={"type": "content", "content": code},
)
print(response.content)
print(response.response_metadata)
/// <summary>
/// Represents a user with a first name, last name, and email.
/// </summary>
public class User
{
    /// <summary>
    /// Gets or sets the user's first name.
    /// </summary>
    public string FirstName { get; set; }

    /// <summary>
    /// Gets or sets the user's last name.
    /// </summary>
    public string LastName { get; set; }

    /// <summary>
    /// Gets or sets the user's email.
    /// </summary>
    public string Email { get; set; }
}
{'token_usage': {'completion_tokens': 226, 'prompt_tokens': 166, 'total_tokens': 392, 'completion_tokens_details': {'accepted_prediction_tokens': 49, 'audio_tokens': None, 'reasoning_tokens': 0, 'rejected_prediction_tokens': 107}, 'prompt_tokens_details': {'audio_tokens': None, 'cached_tokens': 0}}, 'model_name': 'gpt-4o-2024-08-06', 'system_fingerprint': 'fp_45cf54deae', 'finish_reason': 'stop', 'logprobs': None}
Predictions are billed as additional tokens and may increase your usage and costs in exchange for this reduced latency.
预测会被计为额外的令牌，为了换取这种延迟的降低，可能会增加您的使用量和成本。
​
Audio generation (Preview) 音频生成（预览版）
Requires langchain-openai>=0.2.3 需要 langchain-openai>=0.2.3
OpenAI has a new audio generation feature that allows you to use audio inputs and outputs with the gpt-4o-audio-preview model.
OpenAI推出了一项新的音频生成功能</b0，允许你在gpt-4o-audio-preview模型中使用音频输入和输出。
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="gpt-4o-audio-preview",
    temperature=0,
    model_kwargs={
        "modalities": ["text", "audio"],
        "audio": {"voice": "alloy", "format": "wav"},
    },
)

output_message = llm.invoke(
    [
        ("human", "Are you made by OpenAI? Just answer yes or no"),
    ]
)
output_message.additional_kwargs['audio'] will contain a dictionary like
output_message.additional_kwargs['audio'] 将包含一个类似的字典
{
    'data': '<audio data b64-encoded',
    'expires_at': 1729268602,
    'id': 'audio_67127d6a44348190af62c1530ef0955a',
    'transcript': 'Yes.'
}
…and the format will be what was passed in model_kwargs['audio']['format'].
……其格式将是在model_kwargs['audio']['format']中传入的格式。
We can also pass this message with audio data back to the model as part of a message history before openai expires_at is reached.
在openai的expires_at到期之前，我们还可以将这条包含音频数据的消息作为消息历史的一部分传递回模型。
**Output audio is stored under the audio key in AIMessage.additional_kwargs, but input content blocks are typed with an input_audio type and key in HumanMessage.content lists. **
**输出音频存储在AIMessage.additional_kwargs的audio键下，而输入内容块在HumanMessage.content列表中以input_audio类型和键进行标注。**
For more information, see OpenAI’s audio docs.
有关更多信息，请参阅OpenAI的音频文档。
history = [
    ("human", "Are you made by OpenAI? Just answer yes or no"),
    output_message,
    ("human", "And what is your name? Just give your name."),
]
second_output_message = llm.invoke(history)
​
Prompt caching 提示词缓存
OpenAI’s prompt caching feature automatically caches prompts longer than 1024 tokens to reduce costs and improve response times. This feature is enabled for all recent models (gpt-4o and newer).
OpenAI的提示词缓存功能会自动缓存超过1024个token的提示词，以降低成本并提高响应速度。所有最新模型（gpt-4o及更新版本）均支持此功能。
​
Manual caching 手动缓存
You can use the prompt_cache_key parameter to influence OpenAI’s caching and optimize cache hit rates:
你可以使用prompt_cache_key参数来影响OpenAI的缓存并优化缓存命中率：
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4.1")

# Use a cache key for repeated prompts
messages = [
    {"role": "system", "content": "You are a helpful assistant that translates English to French."},
    {"role": "user", "content": "I love programming."},
]

response = llm.invoke(
    messages,
    prompt_cache_key="translation-assistant-v1"
)

# Check cache usage
cache_read_tokens = response.usage_metadata.input_token_details.cache_read
print(f"Cached tokens used: {cache_read_tokens}")
Cache hits require the prompt prefix to match exactly
缓存命中要求提示词前缀完全匹配
​
Cache key strategies 缓存键策略
You can use different cache key strategies based on your application’s needs:
您可以根据应用程序的需求使用不同的缓存键策略：
# Static cache keys for consistent prompt templates
customer_response = llm.invoke(
    messages,
    prompt_cache_key="customer-support-v1"
)

support_response = llm.invoke(
    messages,
    prompt_cache_key="internal-support-v1"
)

# Dynamic cache keys based on context
user_type = "premium"
cache_key = f"assistant-{user_type}-v1"
response = llm.invoke(messages, prompt_cache_key=cache_key)
​
Model-level caching 模型级缓存
You can also set a default cache key at the model level using model_kwargs:
你也可以使用model_kwargs在模型级别设置默认缓存键：
llm = ChatOpenAI(
    model="gpt-4.1-mini",
    model_kwargs={"prompt_cache_key": "default-cache-v1"}
)

# Uses default cache key
response1 = llm.invoke(messages)

# Override with specific cache key
response2 = llm.invoke(messages, prompt_cache_key="override-cache-v1")
​
Flex processing 弹性处理
OpenAI offers a variety of service tiers. The “flex” tier offers cheaper pricing for requests, with the trade-off that responses may take longer and resources might not always be available. This approach is best suited for non-critical tasks, including model testing, data enhancement, or jobs that can be run asynchronously.
OpenAI 提供多种服务层级。“灵活”层级的请求定价更低，但相应地，响应时间可能更长，且资源未必随时可用。这种方式最适合非关键任务，包括模型测试、数据增强或可异步运行的作业。
To use it, initialize the model with service_tier="flex":
要使用它，请通过service_tier="flex"初始化模型：
llm = ChatOpenAI(model="o4-mini", service_tier="flex")
Note that this is a beta feature that is only available for a subset of models. See OpenAI docs for more detail.
请注意，这是一项测试版功能，仅适用于部分模型。更多详情请参阅OpenAI文档。
​
API reference API 参考
For detailed documentation of all features and configuration options, head to the ChatOpenAI API reference.
有关所有功能和配置选项的详细文档，请查阅ChatOpenAI API参考。
