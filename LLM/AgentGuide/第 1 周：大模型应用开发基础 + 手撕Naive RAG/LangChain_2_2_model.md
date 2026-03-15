# Models
LLMs are powerful AI tools that can interpret and generate text like humans. They’re versatile enough to write content, translate languages, summarize, and answer questions without needing specialized training for each task.
> 大型语言模型是强大的人工智能工具，能够像人类一样解读和生成文本。它们功能多样，无需为每项任务进行专门训练，就能完成内容创作、语言翻译、文本总结和问题解答等工作。

In addition to text generation, many models support:
- `Tool calling` - calling external tools (like databases queries or API calls) and use results in their responses.
> 工具调用——调用外部工具（如数据库查询或API调用）并在其响应中使用结果。
- `Structured output` - where the model’s response is constrained to follow a defined format.
> 结构化输出——模型的响应被限制为遵循特定格式。
- `Multimodality` - process and return data other than text, such as images, audio, and video.
> 多模态——处理并返回文本以外的数据，例如图像、音频和视频。
- `Reasoning` - models perform multi-step reasoning to arrive at a conclusion.
> 推理 - 模型通过多步推理得出结论。

Models are **the reasoning engine of agents**. They drive the agent’s decision-making process, determining which tools to call, how to interpret results, and when to provide a final answer.
> 模型是智能体的推理引擎。它们驱动智能体的决策过程，决定调用哪些工具、如何解读结果以及何时提供最终答案。

The quality and capabilities of the model you choose directly impact your agent’s **baseline reliability and performance**. Different models excel at different tasks - some are better at following complex instructions, others at structured reasoning, and some support larger context windows for handling more information.
> 你所选择的模型的质量和能力直接影响智能体的基准可靠性和性能。不同的模型在不同任务上各有所长——有些更擅长遵循复杂指令，有些则在结构化推理方面表现更佳，还有一些支持更大的上下文窗口以处理更多信息。

LangChain’s standard model interfaces give you access to many different provider integrations, which makes it easy to experiment with and switch between models to find the best fit for your use case.
> LangChain的标准模型接口让你能够使用许多不同的提供商集成，这使得你可以轻松尝试不同的模型并在它们之间切换，以找到最适合你用例的模型。
For provider-specific integration information and capabilities, see the [provider’s chat model page](https://docs.langchain.com/oss/python/integrations/chat).
​
## Basic usage
Models can be utilized in two ways:
1. **With agents** - Models can be dynamically specified when creating an agent.
> 在智能体中 - 创建智能体时，可以动态指定模型。
2. **Standalone** - Models can be called directly (outside of the agent loop) for tasks like text generation, classification, or extraction without the need for an agent framework.
> 独立式 - 模型可以直接调用（在智能体循环之外），用于文本生成、分类或提取等任务，无需智能体框架。

The same model interface works in both contexts, which gives you the flexibility to start simple and scale up to more complex agent-based workflows as needed.
> 相同的模型接口在两种情况下都能运行，这让你可以灵活地从简单开始，根据需要扩展到更复杂的基于智能体的工作流。
​
### Initialize a model
The easiest way to get started with a standalone model in LangChain is to use [init_chat_model](https://github.com/meng-yijie1996/myNotes/blob/master/LLM/AgentGuide/LangChainReference/03_init_chat_model.md) to initialize one from a chat model provider of your choice (examples below):
> 在LangChain中开始使用独立模型最简单的方法是使用init_chat_model从你选择的聊天模型提供商那里初始化一个（示例如下）：
#### OpenAI
Read the [OpenAI chat model integration docs](https://docs.langchain.com/oss/python/integrations/chat/openai)

``` bash
pip install -U "langchain[openai]"
```
##### init_chat_model
``` python
import os
from langchain.chat_models import init_chat_model

os.environ["OPENAI_API_KEY"] = "sk-..."

model = init_chat_model("gpt-5.2")
```
##### Model Class
``` python
import os
from langchain_openai import ChatOpenAI

os.environ["OPENAI_API_KEY"] = "sk-..."

model = ChatOpenAI(model="gpt-5.2")
```

#### Anthropic
Read the [Anthropic chat model integration docs](https://docs.langchain.com/oss/python/integrations/chat/anthropic)

``` bash
pip install -U "langchain[anthropic]"
```
##### init_chat_model
``` python
import os
from langchain.chat_models import init_chat_model

os.environ["ANTHROPIC_API_KEY"] = "sk-..."

model = init_chat_model("claude-sonnet-4-6")
```
##### Model Class
``` python
import os
from langchain_anthropic import ChatAnthropic

os.environ["ANTHROPIC_API_KEY"] = "sk-..."

model = ChatAnthropic(model="claude-sonnet-4-6")
```

#### Azure
Read the [Azure chat model integration docs](https://docs.langchain.com/oss/python/integrations/chat/azure_chat_openai)

``` bash
pip install -U "langchain[openai]"
```
##### init_chat_model
``` python
import os
from langchain.chat_models import init_chat_model

os.environ["AZURE_OPENAI_API_KEY"] = "..."
os.environ["AZURE_OPENAI_ENDPOINT"] = "..."
os.environ["OPENAI_API_VERSION"] = "2025-03-01-preview"

model = init_chat_model(
    "azure_openai:gpt-5.2",
    azure_deployment=os.environ["AZURE_OPENAI_DEPLOYMENT_NAME"],
)
```
##### Model Class
``` python
import os
from langchain_openai import AzureChatOpenAI

os.environ["AZURE_OPENAI_API_KEY"] = "..."
os.environ["AZURE_OPENAI_ENDPOINT"] = "..."
os.environ["OPENAI_API_VERSION"] = "2025-03-01-preview"

model = AzureChatOpenAI(
    model="gpt-5.2",
    azure_deployment=os.environ["AZURE_OPENAI_DEPLOYMENT_NAME"]
)
```

#### Google Gemini
Read the [Google GenAI chat model integration docs](https://docs.langchain.com/oss/python/integrations/chat/google_generative_ai)

``` bash
pip install -U "langchain[google-genai]"
```
##### init_chat_model
``` python
import os
from langchain.chat_models import init_chat_model

os.environ["GOOGLE_API_KEY"] = "..."

model = init_chat_model("google_genai:gemini-2.5-flash-lite")
```
##### Model Class
``` python
import os
from langchain_google_genai import ChatGoogleGenerativeAI

os.environ["GOOGLE_API_KEY"] = "..."

model = ChatGoogleGenerativeAI(model="gemini-2.5-flash-lite")
```

#### AWS Bedrock
Read the [AWS Bedrock chat model integration docs](https://docs.langchain.com/oss/python/integrations/chat/bedrock)

``` bash
pip install -U "langchain[aws]"
```
##### init_chat_model
``` python
from langchain.chat_models import init_chat_model

# Follow the steps here to configure your credentials:
# https://docs.aws.amazon.com/bedrock/latest/userguide/getting-started.html

model = init_chat_model(
    "anthropic.claude-3-5-sonnet-20240620-v1:0",
    model_provider="bedrock_converse",
)
```
##### Model Class
``` python
from langchain_aws import ChatBedrock

model = ChatBedrock(model="anthropic.claude-3-5-sonnet-20240620-v1:0")
```

#### HuggingFace
Read the [HuggingFace chat model integration docs](https://docs.langchain.com/oss/python/integrations/chat/huggingface)

``` bash
pip install -U "langchain[huggingface]"
```
##### init_chat_model
``` python
import os
from langchain.chat_models import init_chat_model

os.environ["HUGGINGFACEHUB_API_TOKEN"] = "hf_..."

model = init_chat_model(
    "microsoft/Phi-3-mini-4k-instruct",
    model_provider="huggingface",
    temperature=0.7,
    max_tokens=1024,
)
```
##### Model Class
``` python
import os
from langchain_huggingface import ChatHuggingFace, HuggingFaceEndpoint

os.environ["HUGGINGFACEHUB_API_TOKEN"] = "hf_..."

llm = HuggingFaceEndpoint(
    repo_id="microsoft/Phi-3-mini-4k-instruct",
    temperature=0.7,
    max_length=1024,
)
model = ChatHuggingFace(llm=llm)
```

``` python
response = model.invoke("Why do parrots talk?")
```

See `init_chat_model` for more detail, including information on how to pass model parameters.
有关更多详情（包括如何传递模型参数的信息），请参见init_chat_model。
​
### Supported models
LangChain supports all major model providers, including OpenAI, Anthropic, Google, Azure, AWS Bedrock, and more. Each provider offers a variety of models with different capabilities. For a full list of supported models in LangChain, see the [integrations page](https://docs.langchain.com/oss/python/integrations/providers/overview).
> LangChain支持所有主要的模型提供商，包括OpenAI、Anthropic、谷歌、Azure、AWS Bedrock等。每个提供商都提供了多种具有不同功能的模型。有关LangChain中支持的所有模型的完整列表，请参阅集成页面。
​
### Key methods
**Invoke**: **messages in, messages out**

The model takes messages as input and outputs messages after generating a complete response.
> 该模型将消息作为输入，并在生成完整响应后输出消息。

**Stream**

Invoke the model, but stream the output as it is generated **in real-time**.
> 调用模型，但在生成输出时实时流式传输。

**Batch**

Send **multiple requests** to a model in a batch for more efficient processing.
> 成批向模型发送多个请求，以实现更高效的处理。

In addition to chat models, LangChain provides support for other adjacent technologies, such as embedding models and vector stores. See the [integrations page](https://docs.langchain.com/oss/python/integrations/providers/overview) for details.
> 除了聊天模型外，LangChain 还支持其他相关技术，例如嵌入模型和向量存储。有关详细信息，请参见 集成页面。
​
## Parameters
A chat model takes parameters that can be used to configure its behavior. The full set of supported parameters varies by model and provider, but standard ones include:
> 聊天模型会接收一些可用于配置其行为的参数。受支持的完整参数集因模型和提供商而异，但标准参数包括：
​
##### model `string` `required`
The name or identifier of the specific model you want to use with a provider. You can also specify both the model and its provider in a single argument using the ’:’ format, for example, ‘openai:o1’.
> 你想要与提供商一起使用的特定模型的名称或标识符。你也可以在一个参数中使用“:”格式同时指定模型及其提供商，例如“openai:o1”。
​
##### api_key `string`
The key required for authenticating with the model’s provider. This is usually issued when you sign up for access to the model. Often accessed by setting an environment variable
> 与模型提供商进行身份验证所需的密钥。这通常是在你注册获取模型访问权限时颁发的。通常可以通过设置一个环境变量.
​
##### temperature `number`
Controls the randomness of the model’s output. A higher number makes responses more creative; lower ones make them more deterministic.
> 控制模型输出的随机性。数值越高，响应越具创造性；数值越低，响应越具确定性。
​
##### max_tokens `number`
Limits the total number of tokens **in the response**, effectively controlling how long the output can be.
> 限制响应中的总令牌数，从而有效地控制输出的长度。
​
##### timeout `number`
The maximum time (in seconds) to wait for a response from the model before canceling the request.
> 取消请求前等待模型响应的最长时间（以秒为单位）。
​
##### max_retries `number` `default:"6"`
The maximum number of attempts the system will make to resend a request if it fails due to issues like network timeouts or rate limits. Retries use exponential backoff with jitter. Network errors, rate limits (429), and server errors (5xx) are retried automatically. Client errors such as 401 (unauthorized) or 404 are not retried. For long-running agent tasks on unreliable networks, consider increasing this to 10–15.
> 如果请求因网络超时或速率限制等问题失败，系统将尝试重新发送请求的最大次数。重试采用带抖动的指数退避策略。网络错误、速率限制（429）和服务器错误（5xx）会自动重试。客户端错误（如401（未授权）或404）不会重试。对于在不可靠网络上运行的长时间智能体任务，可考虑将此值增加到10–15。

Using `init_chat_model`, pass these parameters as inline **kwargs:
> 使用init_chat_model，将这些参数作为内联参数传递

##### Initialize using model parameters

``` python
model = init_chat_model(
    "claude-sonnet-4-6",
    # Kwargs passed to the model:
    temperature=0.7,
    timeout=30,
    max_tokens=1000,
    max_retries=6,  # Default; increase for unreliable networks
)
```

Each chat model integration may have additional params used to control provider-specific functionality.
> 每个聊天模型集成可能有额外的参数，用于控制特定于提供商的功能。

For example, ChatOpenAI has use_responses_api to dictate whether to use the OpenAI Responses or Completions API.
> 例如，ChatOpenAI 具有 use_responses_api 来指定是使用 OpenAI Responses API 还是 Completions API。

To find all the parameters supported by a given chat model, head to the [chat model integrations page](https://docs.langchain.com/oss/python/integrations/chat).
> 要查找特定聊天模型支持的所有参数，请前往聊天模型集成页面。
​
## Invocation
A chat model must be invoked to generate an output. There are three primary invocation methods, each suited to different use cases.
> 聊天模型必须调用才能生成输出。主要有三种调用方法，每种方法适用于不同的使用场景。
​
### Invoke
The most straightforward way to call a model is to use `invoke()` with a single message or a list of messages.
> 调用模型最直接的方法是使用invoke()，传入一条消息或一个消息列表。

##### Single message
``` python
response = model.invoke("Why do parrots have colorful feathers?")
print(response)
```

**A list of messages** can be provided to a chat model to represent **conversation history**. Each message has a role that models use to indicate who sent the message in the conversation.
> 可以向聊天模型提供消息列表来表示对话历史。每条消息都有一个角色，模型用该角色来表明对话中是谁发送了这条消息。

##### Dictionary format
``` python
conversation = [
    {"role": "system", "content": "You are a helpful assistant that translates English to French."},
    {"role": "user", "content": "Translate: I love programming."},
    {"role": "assistant", "content": "J'adore la programmation."},
    {"role": "user", "content": "Translate: I love building applications."}
]

response = model.invoke(conversation)
print(response)  # AIMessage("J'adore créer des applications.")
```

##### Message objects
``` python
from langchain.messages import HumanMessage, AIMessage, SystemMessage

conversation = [
    SystemMessage("You are a helpful assistant that translates English to French."),
    HumanMessage("Translate: I love programming."),
    AIMessage("J'adore la programmation."),
    HumanMessage("Translate: I love building applications.")
]

response = model.invoke(conversation)
print(response)  # AIMessage("J'adore créer des applications.")
```

If the **return type** of your invocation is a **string**, ensure that you are using a chat model as opposed to a LLM. Legacy, text-completion LLMs return strings directly. LangChain chat models are prefixed with “Chat”, e.g., ChatOpenAI(/oss/integrations/chat/openai).
> 如果你的调用返回类型是字符串，请确保你使用的是聊天模型而非大语言模型（LLM）。传统的文本补全大语言模型会直接返回字符串。LangChain聊天模型都以“Chat”为前缀，例如ChatOpenAI（/oss/integrations/chat/openai）。
​
### Stream
Most models can stream their output content while it is being generated. By displaying output progressively, streaming significantly improves user experience, particularly for longer responses.
> 大多数模型在生成输出内容时都能进行流式传输。通过逐步显示输出，流式传输显著改善了用户体验，尤其是对于较长的响应而言。

Calling `stream()` returns an iterator that yields output chunks as they are produced. You can use a loop to process each chunk in real-time:
> 调用stream()会返回一个迭代器，会在输出块生成时逐个生成它们。您可以使用循环来实时处理每个块：

##### Basic text streaming
``` python
for chunk in model.stream("Why do parrots have colorful feathers?"):
    print(chunk.text, end="|", flush=True)
```
##### Stream tool calls, reasoning, and other content
``` python
for chunk in model.stream("What color is the sky?"):
    for block in chunk.content_blocks:
        if block["type"] == "reasoning" and (reasoning := block.get("reasoning")):
            print(f"Reasoning: {reasoning}")
        elif block["type"] == "tool_call_chunk":
            print(f"Tool call chunk: {block}")
        elif block["type"] == "text":
            print(block["text"])
        else:
            ...
```
As opposed to `invoke()`, which returns a single `AIMessage` after the model has finished generating its full response, `stream()` returns multiple `AIMessageChunk` objects, each containing a portion of the output text. Importantly, each chunk in a stream is designed to be gathered into a full message via summation:
> 与invoke()不同，后者会在模型完成其完整响应生成后返回单个AIMessage，而stream()会返回多个AIMessageChunk对象，每个对象都包含一部分输出文本。重要的是，流中的每个块都旨在通过汇总组合成一条完整的消息：

##### Construct an AIMessage
``` python
full = None  # None | AIMessageChunk
for chunk in model.stream("What color is the sky?"):
    full = chunk if full is None else full + chunk
    print(full.text)

# The
# The sky
# The sky is
# The sky is typically
# The sky is typically blue
# ...

print(full.content_blocks)
# [{"type": "text", "text": "The sky is typically blue..."}]
```

The resulting message can be treated the same as a message that was generated with invoke()—for example, it can be aggregated into a message history and passed back to the model as conversational context.
> 由此产生的消息可以与使用invoke()生成的消息同等对待——例如，它可以被聚合到消息历史中，并作为对话上下文传递回模型。

Streaming only works if all steps in the program know how to process a stream of chunks. For instance, an application that isn’t streaming-capable would be one that needs to store the entire output in memory before it can be processed.
> 只有当程序中的所有步骤都知道如何处理块流时，流式传输才能工作。例如，一个不具备流式传输能力的应用程序需要将整个输出存储在内存中才能进行处理。

#### Advanced streaming topics
##### Streaming events

##### "Auto-streaming" chat models

​
Batch 批量
Batching a collection of independent requests to a model can significantly improve performance and reduce costs, as the processing can be done in parallel:
将一组独立的请求批量发送给模型可以显著提升性能并降低成本，因为这些处理可以并行进行：
Batch 批量
responses = model.batch([
    "Why do parrots have colorful feathers?",
    "How do airplanes fly?",
    "What is quantum computing?"
])
for response in responses:
    print(response)
This section describes a chat model method batch(), which parallelizes model calls client-side.
本节介绍了一种聊天模型方法batch()，该方法在客户端并行化模型调用。
It is distinct from batch APIs supported by inference providers, such as OpenAI or Anthropic.
它与推理提供商（如OpenAI或Anthropic）支持的批量API不同。
By default, batch() will only return the final output for the entire batch. If you want to receive the output for each individual input as it finishes generating, you can stream results with batch_as_completed():
默认情况下，batch() 只会返回整个批次的最终输出。如果您希望在每个单独输入完成生成时就收到其输出，可以使用 batch_as_completed() 来流式传输结果：
Yield batch responses upon completion
完成后生成批量响应
for response in model.batch_as_completed([
    "Why do parrots have colorful feathers?",
    "How do airplanes fly?",
    "What is quantum computing?"
]):
    print(response)
When using batch_as_completed(), results may arrive out of order. Each includes the input index for matching to reconstruct the original order as needed.
使用batch_as_completed()时，结果可能会无序到达。每个结果都包含输入索引，以便在需要时进行匹配以重建原始顺序。
When processing a large number of inputs using batch() or batch_as_completed(), you may want to control the maximum number of parallel calls. This can be done by setting the max_concurrency attribute in the RunnableConfig dictionary.
使用batch()或batch_as_completed()处理大量输入时，您可能希望控制并行调用的最大数量。这可以通过在RunnableConfig字典中设置max_concurrency属性来实现。
Batch with max concurrency 具有最大并发量的批处理
model.batch(
    list_of_inputs,
    config={
        'max_concurrency': 5,  # Limit to 5 parallel calls
    }
)
See the RunnableConfig reference for a full list of supported attributes.
有关支持的完整属性列表，请参阅RunnableConfig参考文档。
For more details on batching, see the reference.
有关批处理的更多详细信息，请参阅参考资料。
​
Tool calling 工具调用
Models can request to call tools that perform tasks such as fetching data from a database, searching the web, or running code. Tools are pairings of:
模型可以请求调用工具来执行诸如从数据库获取数据、搜索网页或运行代码等任务。工具是以下两者的组合：
A schema, including the name of the tool, a description, and/or argument definitions (often a JSON schema)
一个模式，包括工具名称、描述和/或参数定义（通常是一个JSON模式）
A function or 函数或coroutine 协程 to execute. 执行。
You may hear the term “function calling”. We use this interchangeably with “tool calling”.
你可能会听到“函数调用”这个术语。我们将其与“工具调用”交替使用。
Here’s the basic tool calling flow between a user and a model:
以下是用户与模型之间的基本工具调用流程：







Tools
Model
User
Tools
Model
User
par
[Parallel Tool Calls]
par
[Tool Execution]
"What's the weather in SF and NYC?"
Analyze request & decide tools needed
get_weather("San Francisco")
get_weather("New York")
SF weather data
NYC weather data
Process results & generate response
"SF: 72°F sunny, NYC: 68°F cloudy"
To make tools that you have defined available for use by a model, you must bind them using bind_tools. In subsequent invocations, the model can choose to call any of the bound tools as needed.
要让你定义的工具可供模型使用，你必须使用bind_tools来绑定它们。在后续调用中，模型可以根据需要选择调用任何已绑定的工具。
Some model providers offer 一些模型提供商提供built-in tools 内置工具 that can be enabled via model or invocation parameters (e.g. ChatOpenAI, ChatAnthropic). Check the respective provider reference for details.
可以通过模型或调用参数启用（例如ChatOpenAI、ChatAnthropic）。有关详细信息，请查看相应的提供商参考。
See the tools guide for details and other options for creating tools.
有关创建工具的详细信息和其他选项，请参见工具指南。
Binding user tools 绑定用户工具
from langchain.tools import tool

@tool
def get_weather(location: str) -> str:
    """Get the weather at a location."""
    return f"It's sunny in {location}."


model_with_tools = model.bind_tools([get_weather])

response = model_with_tools.invoke("What's the weather like in Boston?")
for tool_call in response.tool_calls:
    # View tool calls made by the model
    print(f"Tool: {tool_call['name']}")
    print(f"Args: {tool_call['args']}")
When binding user-defined tools, the model’s response includes a request to execute a tool. When using a model separately from an agent, it is up to you to execute the requested tool and return the result back to the model for use in subsequent reasoning. When using an agent, the agent loop will handle the tool execution loop for you.
在绑定用户定义的工具时，模型的响应包含一个执行工具的请求。当将模型与智能体分开使用时，需要由您来执行所请求的工具，并将结果返回给模型，以便在后续推理中使用。当使用智能体时，智能体循环会为您处理工具执行循环。
Below, we show some common ways you can use tool calling.
下面，我们将展示一些使用工具调用的常见方法。
Tool execution loop 工具执行循环

Forcing tool calls 强制工具调用

Parallel tool calls 并行工具调用

Streaming tool calls 流式工具调用

​
Structured output 结构化输出
Models can be requested to provide their response in a format matching a given schema. This is useful for ensuring the output can be easily parsed and used in subsequent processing. LangChain supports multiple schema types and methods for enforcing structured output.
可以要求模型以符合给定模式的格式提供响应。这有助于确保输出能够被轻松解析并用于后续处理。LangChain支持多种模式类型和用于实施结构化输出的方法。
To learn about structured output, see Structured output.
要了解结构化输出，请参阅结构化输出。
Pydantic
皮丹蒂克
TypedDict
类型字典
JSON Schema
JSON 模式
Pydantic models provide the richest feature set with field validation, descriptions, and nested structures.
Pydantic 模型提供了最丰富的功能集，包括字段验证、描述和嵌套结构。
from pydantic import BaseModel, Field

class Movie(BaseModel):
    """A movie with details."""
    title: str = Field(description="The title of the movie")
    year: int = Field(description="The year the movie was released")
    director: str = Field(description="The director of the movie")
    rating: float = Field(description="The movie's rating out of 10")

model_with_structure = model.with_structured_output(Movie)
response = model_with_structure.invoke("Provide details about the movie Inception")
print(response)  # Movie(title="Inception", year=2010, director="Christopher Nolan", rating=8.8)
Key considerations for structured output
结构化输出的关键注意事项
Method parameter: Some providers support different methods for structured output:
方法参数：一些提供商支持不同的结构化输出方法：
'json_schema': Uses dedicated structured output features offered by the provider.
'json_schema'：使用提供商提供的专用结构化输出功能。
'function_calling': Derives structured output by forcing a tool call that follows the given schema.
'function_calling'：通过强制遵循给定模式的工具调用来生成结构化输出。
'json_mode': A precursor to 'json_schema' offered by some providers. Generates valid JSON, but the schema must be described in the prompt.
'json_mode'：一些提供商提供的'json_schema'的前身。它会生成有效的JSON，但必须在提示词中描述其模式。
Include raw: Set include_raw=True to get both the parsed output and the raw AI message.
包含原始内容：设置include_raw=True以同时获取解析后的输出和原始AI消息。
Validation: Pydantic models provide automatic validation. TypedDict and JSON Schema require manual validation.
验证：Pydantic模型提供自动验证。TypedDict和JSON模式需要手动验证。
See your provider’s integration page for supported methods and configuration options.
有关支持的方法和配置选项，请参阅您的提供商集成页面。
Example: Message output alongside parsed structure
示例：消息输出与解析后的结构一起显示

Example: Nested structures 示例：嵌套结构

​
Advanced topics 高级主题
​
Model profiles 模型配置文件
Model profiles require langchain>=1.1.
模型配置文件需要langchain>=1.1。
LangChain chat models can expose a dictionary of supported features and capabilities through a .profile attribute:
LangChain聊天模型可以通过.profile属性展示一个包含支持的功能和能力的字典：
model.profile
# {
#   "max_input_tokens": 400000,
#   "image_inputs": True,
#   "reasoning_output": True,
#   "tool_calling": True,
#   ...
# }
Refer to the full set of fields in the API reference.
请参考API 参考中的完整字段集。
Much of the model profile data is powered by the models.dev project, an open source initiative that provides model capability data. These data are augmented with additional fields for purposes of use with LangChain. These augmentations are kept aligned with the upstream project as it evolves.
模型配置文件的大部分数据由models.dev项目提供支持，这是一个提供模型能力数据的开源计划。为了与LangChain配合使用，这些数据还增加了额外的字段。随着上游项目的发展，这些新增内容会与之保持一致。
Model profile data allow applications to work around model capabilities dynamically. For example:
模型配置文件数据允许应用程序动态适应模型的能力。例如：
Summarization middleware can trigger summarization based on a model’s context window size.
摘要中间件可以根据模型的上下文窗口大小触发摘要生成。
Structured output strategies in create_agent can be inferred automatically (e.g., by checking support for native structured output features).
结构化输出策略在create_agent中可以自动推断（例如，通过检查对原生结构化输出功能的支持）。
Model inputs can be gated based on supported modalities and maximum input tokens.
可以根据支持的模态和最大输入令牌对模型输入进行控制。
Updating or overwriting profile data
更新或覆盖配置文件数据

Model profiles are a beta feature. The format of a profile is subject to change.
模型配置文件是一项测试版功能。配置文件的格式可能会发生变化。
​
Multimodal 多模态
Certain models can process and return non-textual data such as images, audio, and video. You can pass non-textual data to a model by providing content blocks.
某些模型可以处理并返回图像、音频和视频等非文本数据。您可以通过提供内容块将非文本数据传递给模型。
All LangChain chat models with underlying multimodal capabilities support:
所有具有潜在多模态能力的LangChain聊天模型都支持：
Data in the cross-provider standard format (see our messages guide)
跨提供商标准格式的数据（参见我们的消息指南）
OpenAI chat completions format OpenAI 聊天补全格式
Any format that is native to that specific provider (e.g., Anthropic models accept Anthropic native format)
任何特定提供商的原生格式（例如，Anthropic模型接受Anthropic原生格式）
See the multimodal section of the messages guide for details.
详见消息指南的多模态部分。
Some models 一些模型 can return multimodal data as part of their response. If invoked to do so, the resulting AIMessage will have content blocks with multimodal types.
可以在其响应中返回多模态数据。如果被调用执行此操作，生成的AIMessage将包含具有多模态类型的内容块。
Multimodal output 多模态输出
response = model.invoke("Create a picture of a cat")
print(response.content_blocks)
# [
#     {"type": "text", "text": "Here's a picture of a cat"},
#     {"type": "image", "base64": "...", "mime_type": "image/jpeg"},
# ]
See the integrations page for details on specific providers.
有关特定提供商的详细信息，请参见集成页面。
​
Reasoning 推理
Many models are capable of performing multi-step reasoning to arrive at a conclusion. This involves breaking down complex problems into smaller, more manageable steps.
许多模型能够执行多步推理以得出结论。这包括将复杂问题分解为更小、更易于处理的步骤。
If supported by the underlying model, you can surface this reasoning process to better understand how the model arrived at its final answer.
如果底层模型支持，你可以呈现这一推理过程，以更好地理解模型是如何得出最终答案的。

Stream reasoning output
流式推理输出

Complete reasoning output
完整推理输出
for chunk in model.stream("Why do parrots have colorful feathers?"):
    reasoning_steps = [r for r in chunk.content_blocks if r["type"] == "reasoning"]
    print(reasoning_steps if reasoning_steps else chunk.text)
Depending on the model, you can sometimes specify the level of effort it should put into reasoning. Similarly, you can request that the model turn off reasoning entirely. This may take the form of categorical “tiers” of reasoning (e.g., 'low' or 'high') or integer token budgets.
根据模型的不同，有时你可以指定它在推理时应投入的努力程度。同样，你也可以要求模型完全关闭推理功能。这可能表现为推理的明确“层级”（例如，'low'或'high'）或整数令牌预算。
For details, see the integrations page or reference for your respective chat model.
有关详情，请参阅您所用聊天模型的集成页面或参考资料。
​
Local models 本地模型
LangChain supports running models locally on your own hardware. This is useful for scenarios where either data privacy is critical, you want to invoke a custom model, or when you want to avoid the costs incurred when using a cloud-based model.
LangChain支持在您自己的硬件上本地运行模型。这在以下场景中非常有用：数据隐私至关重要、您希望调用自定义模型，或者您希望避免使用基于云的模型时产生的费用。
Ollama is one of the easiest ways to run chat and embedding models locally.
Ollama是在本地运行聊天和嵌入模型的最简单方法之一。
​
Prompt caching 提示词缓存
Many providers offer prompt caching features to reduce latency and cost on repeat processing of the same tokens. These features can be implicit or explicit:
许多提供商都提供提示词缓存功能，以减少重复处理相同令牌时的延迟和成本。这些功能可以是隐式的或显式的：
Implicit prompt caching: providers will automatically pass on cost savings if a request hits a cache. Examples: OpenAI and Gemini.
隐式提示缓存：如果请求命中缓存，提供商将自动传递成本节省。例如：OpenAI和Gemini。
Explicit caching: providers allow you to manually indicate cache points for greater control or to guarantee cost savings. Examples:
显式缓存：提供商允许您手动指定缓存点，以获得更大的控制权或确保成本节约。例如：
ChatOpenAI (via prompt_cache_key) ChatOpenAI（通过prompt_cache_key）
Anthropic’s AnthropicPromptCachingMiddleware
Anthropic的AnthropicPromptCachingMiddleware
Gemini. Gemini。
AWS Bedrock
Prompt caching is often only engaged above a minimum input token threshold. See provider pages for details.
提示词缓存通常仅在输入令牌超过最低阈值时启用。有关详细信息，请参阅提供商页面。
Cache usage will be reflected in the usage metadata of the model response.
缓存使用情况将反映在模型响应的使用元数据中。
​
Server-side tool use 服务器端工具使用
Some providers support server-side tool-calling loops: models can interact with web search, code interpreters, and other tools and analyze the results in a single conversational turn.
一些提供商支持服务器端的工具调用循环：模型可以在单次对话轮次中与网络搜索、代码解释器和其他工具进行交互，并分析结果。
If a model invokes a tool server-side, the content of the response message will include content representing the invocation and result of the tool. Accessing the content blocks of the response will return the server-side tool calls and results in a provider-agnostic format:
如果模型在服务器端调用工具，响应消息的内容将包含表示工具调用和结果的内容。访问响应的内容块将以与提供商无关的格式返回服务器端工具调用和结果：
Invoke with server-side tool use 使用服务器端工具调用
from langchain.chat_models import init_chat_model

model = init_chat_model("gpt-4.1-mini")

tool = {"type": "web_search"}
model_with_tools = model.bind_tools([tool])

response = model_with_tools.invoke("What was a positive news story from today?")
print(response.content_blocks)
Result 结果
[
    {
        "type": "server_tool_call",
        "name": "web_search",
        "args": {
            "query": "positive news stories today",
            "type": "search"
        },
        "id": "ws_abc123"
    },
    {
        "type": "server_tool_result",
        "tool_call_id": "ws_abc123",
        "status": "success"
    },
    {
        "type": "text",
        "text": "Here are some positive news stories from today...",
        "annotations": [
            {
                "end_index": 410,
                "start_index": 337,
                "title": "article title",
                "type": "citation",
                "url": "..."
            }
        ]
    }
]
See all 29 lines
查看全部29行
This represents a single conversational turn; there are no associated ToolMessage objects that need to be passed in as in client-side tool-calling.
这代表单个对话轮次；不存在像客户端工具调用中那样需要传入的相关工具消息对象。
See the integration page for your given provider for available tools and usage details.
有关可用工具和使用详情，请参阅您所用提供商的集成页面。
​
Rate limiting 速率限制
Many chat model providers impose a limit on the number of invocations that can be made in a given time period. If you hit a rate limit, you will typically receive a rate limit error response from the provider, and will need to wait before making more requests.
许多聊天模型提供商对特定时间段内的调用次数设置了限制。如果达到速率限制，您通常会收到提供商返回的速率限制错误响应，并且需要等待一段时间才能继续发送更多请求。
To help manage rate limits, chat model integrations accept a rate_limiter parameter that can be provided during initialization to control the rate at which requests are made.
为了帮助管理速率限制，聊天模型集成接受一个rate_limiter参数，该参数可在初始化期间提供，用于控制请求的发出速率。
Initialize and use a rate limiter 初始化并使用速率限制器

​
Base URL and proxy settings 基础URL和代理设置
You can configure a custom base URL for providers that implement the OpenAI Chat Completions API.
对于实现了OpenAI聊天补全API的提供商，你可以配置自定义的基础URL。
model_provider="openai" (or direct ChatOpenAI usage) targets the official OpenAI API specification. Provider-specific fields from routers and proxies may not be extracted or preserved.
model_provider="openai"（或直接使用ChatOpenAI）针对的是官方OpenAI API规范。来自路由器和代理的特定于提供商的字段可能不会被提取或保留。
For OpenRouter and LiteLLM, prefer the dedicated integrations:
对于OpenRouter和LiteLLM，建议优先使用专用集成：
OpenRouter via ChatOpenRouter (langchain-openrouter)
通过ChatOpenRouter （langchain-openrouter）
LiteLLM via ChatLiteLLM / ChatLiteLLMRouter (langchain-litellm)
LiteLLM通过ChatLiteLLM/ChatLiteLLMRouter（langchain-litellm）
Custom base URL 自定义基础URL

HTTP proxy configuration HTTP代理配置

​
Log probabilities 对数概率
Certain models can be configured to return token-level log probabilities representing the likelihood of a given token by setting the logprobs parameter when initializing the model:
通过在初始化模型时设置logprobs参数，可以将某些模型配置为返回表示给定标记可能性的标记级对数概率：
model = init_chat_model(
    model="gpt-4.1",
    model_provider="openai"
).bind(logprobs=True)

response = model.invoke("Why do parrots talk?")
print(response.response_metadata["logprobs"])
​
Token usage 令牌使用量
A number of model providers return token usage information as part of the invocation response. When available, this information will be included on the AIMessage objects produced by the corresponding model. For more details, see the messages guide.
许多模型提供商在调用响应中返回令牌使用信息。如果有此信息，它将包含在相应模型生成的AIMessage对象中。有关更多详细信息，请参阅messages指南。
Some provider APIs, notably OpenAI and Azure OpenAI chat completions, require users opt-in to receiving token usage data in streaming contexts. See the streaming usage metadata section of the integration guide for details.
一些提供商的API，特别是OpenAI和Azure OpenAI的聊天补全功能，要求用户在流式传输环境中选择接收令牌使用数据。详情请参见集成指南的流式传输使用元数据部分。
You can track aggregate token counts across models in an application using either a callback or context manager, as shown below:
您可以使用回调或上下文管理器跟踪应用程序中不同模型的总令牌计数，如下所示：
Callback handler
回调处理器
Context manager
上下文管理器
from langchain.chat_models import init_chat_model
from langchain_core.callbacks import UsageMetadataCallbackHandler

model_1 = init_chat_model(model="gpt-4.1-mini")
model_2 = init_chat_model(model="claude-haiku-4-5-20251001")

callback = UsageMetadataCallbackHandler()
result_1 = model_1.invoke("Hello", config={"callbacks": [callback]})
result_2 = model_2.invoke("Hello", config={"callbacks": [callback]})
print(callback.usage_metadata)
{
    'gpt-4.1-mini-2025-04-14': {
        'input_tokens': 8,
        'output_tokens': 10,
        'total_tokens': 18,
        'input_token_details': {'audio': 0, 'cache_read': 0},
        'output_token_details': {'audio': 0, 'reasoning': 0}
    },
    'claude-haiku-4-5-20251001': {
        'input_tokens': 8,
        'output_tokens': 21,
        'total_tokens': 29,
        'input_token_details': {'cache_read': 0, 'cache_creation': 0}
    }
}
​
Invocation config 调用配置
When invoking a model, you can pass additional configuration through the config parameter using a RunnableConfig dictionary. This provides run-time control over execution behavior, callbacks, and metadata tracking.
调用模型时，你可以通过config参数传递额外配置，该参数使用RunnableConfig字典。这提供了对执行行为、回调和元数据跟踪的运行时控制。
Common configuration options include:
常见的配置选项包括：
Invocation with config 带配置调用
response = model.invoke(
    "Tell me a joke",
    config={
        "run_name": "joke_generation",      # Custom name for this run
        "tags": ["humor", "demo"],          # Tags for categorization
        "metadata": {"user_id": "123"},     # Custom metadata
        "callbacks": [my_callback_handler], # Callback handlers
    }
)
These configuration values are particularly useful when:
以下情况下，这些配置值尤其有用：
Debugging with LangSmith tracing 使用LangSmith追踪进行调试
Implementing custom logging or monitoring
实施自定义日志记录或监控
Controlling resource usage in production
在生产环境中控制资源使用
Tracking invocations across complex pipelines
在复杂管道中跟踪调用
Key configuration attributes 关键配置属性

See full RunnableConfig reference for all supported attributes.
查看完整的RunnableConfig参考以了解所有支持的属性。
​
Configurable models 可配置模型
You can also create a runtime-configurable model by specifying configurable_fields. If you don’t specify a model value, then 'model' and 'model_provider' will be configurable by default.
你也可以通过指定configurable_fields来创建一个可在运行时配置的模型。如果你不指定模型值，那么'model'和'model_provider'将默认是可配置的。
from langchain.chat_models import init_chat_model

configurable_model = init_chat_model(temperature=0)

configurable_model.invoke(
    "what's your name",
    config={"configurable": {"model": "gpt-5-nano"}},  # Run with GPT-5-Nano
)
configurable_model.invoke(
    "what's your name",
    config={"configurable": {"model": "claude-sonnet-4-6"}},  # Run with Claude
)
Configurable model with default values
具有默认值的可配置模型

Using a configurable model declaratively
声明式使用可配置模型
