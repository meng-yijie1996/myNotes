# init_chat_model
Initialize a chat model from any supported provider using a unified interface.

Two main use cases:

1. Fixed model – specify the model upfront and get a ready-to-use chat model.
> 固定模型——预先指定模型，即可获得一个随时可用的聊天模型。
2. Configurable model – choose to specify parameters (including model name) at runtime via `config`. Makes it easy to switch between models/providers without changing your code
> 可配置模型——选择在运行时通过config指定参数（包括模型名称）。无需更改代码即可轻松在不同模型/提供商之间切换

Installation requirements
- Requires the integration package for the chosen model provider to be installed.
> 需要安装所选模型提供商的集成包。
- See the `model_provider` parameter below for specific package names (e.g., pip install langchain-openai).
> 请参见下方的model_provider参数，了解具体的包名称（例如：pip install langchain-openai）。
- Refer to the [provider integration's API reference](https://docs.langchain.com/oss/python/integrations/providers?_gl=1*115p00n*_gcl_au*Mzc5MTc3ODg5LjE3NzEwNTk5ODM.*_ga*MTg3MTg2ODE2NS4xNzcxMDU5OTg0*_ga_47WX3HKKY2*czE3NzM1NzM4MTUkbzExJGcxJHQxNzczNTc1MDA3JGo2MCRsMCRoMA..) for supported model parameters to use as **kwargs.
> 有关可用作**kwargs的受支持模型参数，请参阅提供商集成的API参考。

``` python
init_chat_model(
  model: str | None = None,
  *,
  model_provider: str | None = None,
  configurable_fields: Literal['any'] | list[str] | tuple[str, ...] | None = None,
  config_prefix: str | None = None,
  **kwargs: Any = {}
) -> BaseChatModel | _ConfigurableModel
```

### Initialize a non-configurable model
> 初始化一个不可配置的模型
``` python
# pip install langchain langchain-openai langchain-anthropic langchain-google-vertexai

from langchain.chat_models import init_chat_model

o3_mini = init_chat_model("openai:o3-mini", temperature=0)
claude_sonnet = init_chat_model("anthropic:claude-sonnet-4-5-20250929", temperature=0)
gemini_2-5_flash = init_chat_model("google_vertexai:gemini-2.5-flash", temperature=0)

o3_mini.invoke("what's your name")
claude_sonnet.invoke("what's your name")
gemini_2-5_flash.invoke("what's your name")
```

### Partially configurable model with no default
> 部分可配置且无默认值的模型
``` python
# pip install langchain langchain-openai langchain-anthropic

from langchain.chat_models import init_chat_model

# (We don't need to specify configurable=True if a model isn't specified.)
configurable_model = init_chat_model(temperature=0)

configurable_model.invoke("what's your name", config={"configurable": {"model": "gpt-4o"}})
# Use GPT-4o to generate the response

configurable_model.invoke(
    "what's your name",
    config={"configurable": {"model": "claude-sonnet-4-5-20250929"}},
)
```

### Fully configurable model with a default
> 具有默认值的完全可配置模型
``` python
# pip install langchain langchain-openai langchain-anthropic

from langchain.chat_models import init_chat_model

configurable_model_with_default = init_chat_model(
    "openai:gpt-4o",
    configurable_fields="any",  # Note: This allows us to configure other params like temperature, max_tokens, etc at runtime.
    config_prefix="foo",
    temperature=0,
)

configurable_model_with_default.invoke("what's your name")
# GPT-4o response with temperature 0 (as set in default)

configurable_model_with_default.invoke(
    "what's your name",
    config={
        "configurable": {
            "foo_model": "anthropic:claude-sonnet-4-5-20250929",
            "foo_temperature": 0.6,
        }
    },
)
# Override default to use Sonnet 4.5 with temperature 0.6 to generate response
```

### Bind tools to a configurable model
> 将工具绑定到可配置模型
You can call any chat model declarative methods on a configurable model in the same way that you would with a normal model:
> 你可以像使用普通模型一样，在可配置模型上调用任何聊天模型的声明式方法：

``` python
# pip install langchain langchain-openai langchain-anthropic

from langchain.chat_models import init_chat_model
from pydantic import BaseModel, Field

class GetWeather(BaseModel):
    '''Get the current weather in a given location'''

    location: str = Field(..., description="The city and state, e.g. San Francisco, CA")

class GetPopulation(BaseModel):
    '''Get the current population in a given location'''

    location: str = Field(..., description="The city and state, e.g. San Francisco, CA")

configurable_model = init_chat_model(
    "gpt-4o", configurable_fields=("model", "model_provider"), temperature=0
)

configurable_model_with_tools = configurable_model.bind_tools(
    [
        GetWeather,
        GetPopulation,
    ]
)
configurable_model_with_tools.invoke(
    "Which city is hotter today and which is bigger: LA or NY?"
)
# Use GPT-4o

configurable_model_with_tools.invoke(
    "Which city is hotter today and which is bigger: LA or NY?",
    config={"configurable": {"model": "claude-sonnet-4-5-20250929"}},
)
# Use Sonnet 4.5
```

## Parameters
|Name|Type|Description|
|:---|:---|:---|
|model|str \| None|Default: None <br> The model name, optionally prefixed with provider (e.g., 'openai:gpt-4o'). <br> Prefer exact model IDs from provider docs over aliases for reliable behavior (e.g., dated versions like '...-20250514' instead of '...-latest'). <br> 为确保行为可靠，相比别名，更推荐使用提供商文档中的精确模型ID（例如，带有日期的版本，如'...-20250514'，而非'...-latest'） <br> Will attempt to infer model_provider from model if not specified. <br> 如果未指定，将尝试从模型中推断出model_provider。 <br> The following providers will be inferred based on these model prefixes: <br> gpt-... \| o1... \| o3... -> openai <br> claude... -> anthropic <br> amazon... -> bedrock <br> gemini... -> google_vertexai <br> command... -> cohere <br> accounts/fireworks... -> fireworks <br> mistral... -> mistralai <br> deepseek... -> deepseek <br> grok... -> xai <br> sonar... -> perplexity <br> solar... -> upstage|
|model_provider|str \| None|Default: None <br> The model provider if not specified as part of the model arg (see above). <br> 如果模型参数中未指定模型提供商（见上文）。 <br> Supported `model_provider` values and the corresponding integration package are: <br> 支持的model_provider值及相应的集成包如下： <br> openai -> langchain-openai <br> anthropic -> langchain-anthropic <br> azure_openai -> langchain-openai <br> azure_ai -> langchain-azure-ai <br> google_vertexai -> langchain-google-vertexai <br> google_genai -> langchain-google-genai <br> anthropic_bedrock -> langchain-aws <br> bedrock -> langchain-aws <br> bedrock_converse -> langchain-aws <br> cohere -> langchain-cohere <br> fireworks -> langchain-fireworks <br> together -> langchain-together <br> mistralai -> langchain-mistralai <br> huggingface -> langchain-huggingface <br> groq -> langchain-groq <br> ollama -> langchain-ollama <br> google_anthropic_vertex -> langchain-google-vertexai <br> deepseek -> langchain-deepseek <br> ibm -> langchain-ibm <br> nvidia -> langchain-nvidia-ai-endpoints <br> xai -> langchain-xai <br> openrouter -> langchain-openrouter <br> perplexity -> langchain-perplexity <br> upstage -> langchain-upstage|
|configurable_fields|Literal['any'] \| list[str] \| tuple[str, ...] \| None|Default: None <br> Which model parameters are configurable at runtime: <br>None: No configurable fields (i.e., a fixed model). <br> 'any': All fields are configurable. See security note below. <br> list[str] | Tuple[str, ...]: Specified fields are configurable. <br>  <br> Fields are assumed to have config_prefix stripped if a config_prefix is specified. 即不需要体现前缀 <br> If model is specified, then defaults to None(针对configurable_fields，下同). <br> If model is not specified, then defaults to ("model", "model_provider"). <br>  <br> Security note <br> Setting configurable_fields="any" means fields like api_key, base_url, etc., can be altered at runtime, potentially redirecting model requests to a different service/user. <br> configurable_fields <br> Make sure that if you're accepting untrusted configurations that you enumerate the configurable_fields=(...) explicitly. <br> 请确保，如果您要接受不受信任的配置，请明确列出|
|config_prefix|str \| None|Default: None <br> Optional prefix for configuration keys. <br> Useful when you have multiple configurable models in the same application. <br> 当同一应用程序中有多个可配置模型时，这会很有用。 <br> If 'config_prefix' is a non-empty string then model will be configurable at runtime via the config["configurable"]["{config_prefix}_{param}"] keys. See examples below. <br> 如果'config_prefix'是一个非空字符串，那么model将在运行时通过config["configurable"]["{config_prefix}_{param}"]键进行配置。参见下面的示例。 <br> If 'config_prefix' is an empty string then model will be configurable via config["configurable"]["{param}"]. <br> 如果'config_prefix'是空字符串，那么模型可通过config["configurable"]["{param}"]进行配置。|
|**kwargs|Any|Default: {} <br> Additional model-specific keyword args to pass to the underlying chat model's __init__ method. Common parameters include: <br> temperature: Model temperature for controlling randomness. <br> max_tokens: Maximum number of output tokens. <br> timeout: Maximum time (in seconds) to wait for a response. <br> max_retries: Maximum number of retry attempts for failed requests. <br> base_url: Custom API endpoint URL. <br> rate_limiter: A BaseRateLimiter instance to control request rate. <br> Refer to the specific model provider's integration reference for all available parameters.|
