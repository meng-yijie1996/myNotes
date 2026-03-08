### Run the code
To run any of the examples, copy the code to a file `main.py`, and start `fastapi dev`:
> 要运行任何示例，请将代码复制到文件`main.py`中，然后启动`fastapi dev`
``` bash
fastapi dev
```

### Install FastAPI
Make sure you create a virtual environment, activate it, and then **install FastAPI**:
``` bash
pip install "fastapi[standard]"
```

``` bash
PS C:\Users\myj\Desktop\2025-2026\code\fastapi> pip install "fastapi[standard]"
Collecting fastapi[standard]
  Downloading https://files.pythonhosted.org/packages/4d/d2/3ad038a2365fefbac19d9a046cab7ce45f4c7bfa81d877cbece9707de9ce/fastapi-0.103.2-py3-none-any.whl (66kB)
    100% |████████████████████████████████| 71kB 566kB/s
  fastapi 0.103.2 does not provide the extra 'standard'
Collecting starlette<0.28.0,>=0.27.0 (from fastapi[standard])
  Downloading https://files.pythonhosted.org/packages/58/f8/e2cca22387965584a409795913b774235752be4176d276714e15e1a58884/starlette-0.27.0-py3-none-any.whl (66kB)
    100% |████████████████████████████████| 71kB 1.0MB/s
Requirement already satisfied: typing-extensions>=4.5.0 in d:\installsite\python\python37\lib\site-packages (from fastapi[standard]) (4.6.3)
Collecting pydantic!=1.8,!=1.8.1,!=2.0.0,!=2.0.1,!=2.1.0,<3.0.0,>=1.7.4 (from fastapi[standard])
  Downloading https://files.pythonhosted.org/packages/dd/b7/9aea7ee6c01fe3f3c03b8ca3c7797c866df5fecece9d6cb27caa138db2e2/pydantic-2.5.3-py3-none-any.whl (381kB)
    100% |████████████████████████████████| 389kB 3.5MB/s
Collecting anyio<4.0.0,>=3.7.1 (from fastapi[standard])
  Downloading https://files.pythonhosted.org/packages/19/24/44299477fe7dcc9cb58d0a57d5a7588d6af2ff403fdd2d47a246c91a3246/anyio-3.7.1-py3-none-any.whl (80kB)
    100% |████████████████████████████████| 81kB 2.2MB/s
Collecting importlib-metadata; python_version == "3.7" (from pydantic!=1.8,!=1.8.1,!=2.0.0,!=2.0.1,!=2.1.0,<3.0.0,>=1.7.4->fastapi[standard])
  Using cached https://files.pythonhosted.org/packages/ff/94/64287b38c7de4c90683630338cf28f129decbba0a44f0c6db35a873c73c4/importlib_metadata-6.7.0-py3-none-any.whl
Collecting annotated-types>=0.4.0 (from pydantic!=1.8,!=1.8.1,!=2.0.0,!=2.0.1,!=2.1.0,<3.0.0,>=1.7.4->fastapi[standard])
  Downloading https://files.pythonhosted.org/packages/d8/f0/a2ee543a96cc624c35a9086f39b1ed2aa403c6d355dfe47a11ee5c64a164/annotated_types-0.5.0-py3-none-any.whl
Collecting pydantic-core==2.14.6 (from pydantic!=1.8,!=1.8.1,!=2.0.0,!=2.0.1,!=2.1.0,<3.0.0,>=1.7.4->fastapi[standard])
  Downloading https://files.pythonhosted.org/packages/00/9f/bca85affee28fff3d8fd848643f6c7bbcbd0567c07f2fb2e5c8e5690f6e8/pydantic_core-2.14.6-cp37-none-win_amd64.whl (1.9MB)
    100% |████████████████████████████████| 1.9MB 1.0MB/s
Collecting exceptiongroup; python_version < "3.11" (from anyio<4.0.0,>=3.7.1->fastapi[standard])
  Downloading https://files.pythonhosted.org/packages/8a/0e/97c33bf5009bdbac74fd2beace167cab3f978feb69cc36f1ef79360d6c4e/exceptiongroup-1.3.1-py3-none-any.whl
Collecting sniffio>=1.1 (from anyio<4.0.0,>=3.7.1->fastapi[standard])
  Downloading https://files.pythonhosted.org/packages/e9/44/75a9c9421471a6c4805dbf2356f7c181a29c1879239abab1ea2cc8f38b40/sniffio-1.3.1-py3-none-any.whl
Collecting idna>=2.8 (from anyio<4.0.0,>=3.7.1->fastapi[standard])
  Downloading https://files.pythonhosted.org/packages/76/c6/c88e154df9c4e1a2a66ccf0005a88dfb2650c1dffb6f5ce603dfbd452ce3/idna-3.10-py3-none-any.whl (70kB)
    100% |████████████████████████████████| 71kB 4.4MB/s
Collecting zipp>=0.5 (from importlib-metadata; python_version == "3.7"->pydantic!=1.8,!=1.8.1,!=2.0.0,!=2.0.1,!=2.1.0,<3.0.0,>=1.7.4->fastapi[standard])
  Using cached https://files.pythonhosted.org/packages/5b/fa/c9e82bbe1af6266adf08afb563905eb87cab83fde00a0a08963510621047/zipp-3.15.0-py3-none-any.whl
Installing collected packages: exceptiongroup, sniffio, idna, anyio, starlette, zipp, importlib-metadata, annotated-types, pydantic-core, pydantic, fastapi
Successfully installed annotated-types-0.5.0 anyio-3.7.1 exceptiongroup-1.3.1 fastapi-0.103.2 idna-3.10 importlib-metadata-6.7.0 pydantic-2.5.3 pydantic-core-2.14.6 sniffio-1.3.1 starlette-0.27.0 zipp-3.15.0
You are using pip version 19.0.3, however version 24.0 is available.
You should consider upgrading via the 'python -m pip install --upgrade pip' command.
PS C:\Users\myj\Desktop\2025-2026\code\fastapi>
```

**Note:**
When you install with pip install "fastapi[standard]" it comes with some default optional standard dependencies, including fastapi-cloud-cli, which allows you to deploy to FastAPI Cloud.
> 当你使用 pip install "fastapi[standard]" 进行安装时，它会附带一些默认的可选标准依赖项，包括 fastapi-cloud-cli，这使你能够部署到 FastAPI Cloud。

If you don't want to have those optional dependencies, you can instead install pip install fastapi.
> 如果你不想安装那些可选依赖项，也可以改为安装pip install fastapi。

If you want to install the standard dependencies but without the fastapi-cloud-cli, you can install with pip install "fastapi[standard-no-fastapi-cloud-cli]".
> 如果你想安装标准依赖项但不包括fastapi-cloud-cli</b0，可以使用pip install "fastapi[standard-no-fastapi-cloud-cli]"进行安装。

---

# First Steps
The simplest FastAPI file could look like this:

``` Python 3.10+
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def root():
    return {"message": "Hello World"}
```
Copy that to a file main.py.

Run the live server:
``` bash
fastapi dev
```

In the output, there's a line with something like:
``` bash
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
```
That line shows the URL where your app is being served on your local machine.

## Check it
Open your browser at http://127.0.0.1:8000.

You will see the JSON response as:

``` bash
{"message": "Hello World"}
```

## Interactive API docs
Now go to http://127.0.0.1:8000/docs.

You will see the automatic interactive API documentation (provided by Swagger UI):

![index-01-swagger-ui-simple](../pics/index-01-swagger-ui-simple.png "index-01-swagger-ui-simple")

## Alternative API docs
And now, go to http://127.0.0.1:8000/redoc.

You will see the alternative automatic documentation (provided by ReDoc):

![index-02-redoc-simple](../pics/index-02-redoc-simple.png "index-02-redoc-simple")

## OpenAPI
**FastAPI** generates a "schema" with all your API using the **OpenAPI** standard for defining APIs.

### Schema
A "schema" is a definition or description of something. Not the code that implements it, but just an **abstract description**.

### API "schema"
In this case, OpenAPI is a specification that dictates how to define a schema of your API.

This schema definition includes your API paths, the possible parameters they take, etc.

### Data "schema"
The term "schema" might also refer to the shape of some data, like a JSON content.

In that case, it would mean the JSON attributes, and data types they have, etc.

### OpenAPI and JSON Schema
OpenAPI defines an API schema for your API. And that schema includes definitions (or "schemas") of the data sent and received by your API using JSON Schema, the standard for JSON data schemas.

### Check the openapi.json
http://127.0.0.1:8000/openapi.json

``` json
{
    "openapi": "3.1.0",
    "info": {
        "title": "FastAPI",
        "version": "0.1.0"
    },
    "paths": {
        "/items/": {
            "get": {
                "responses": {
                    "200": {
                        "description": "Successful Response",
                        "content": {
                            "application/json": {



...
```

### What is OpenAPI for OpenAPI有什么用
The OpenAPI schema is what powers the two interactive documentation systems included.

And there are dozens of alternatives, all based on OpenAPI. You could easily add any of those alternatives to your application built with FastAPI.

You could also use it to generate code automatically, for clients that communicate with your API. For example, frontend, mobile or IoT applications.

## Configure the app `entrypoint` in `pyproject.toml`
You can configure where your app is located in a `pyproject.toml` file like:

``` bash
[tool.fastapi]
entrypoint = "main:app"
```

That entrypoint will tell the fastapi command that it should import the app like:

``` python
from main import app
```

If your code was structured like:
``` bash
.
├── backend
│   ├── main.py
│   ├── __init__.py
```
Then you need:

``` bash
[tool.fastapi]
entrypoint = "backend.main:app"
```

``` python
from backend.main import app
```

## `fastapi dev` with path
You can also pass the file path to the fastapi dev command, and it will guess the FastAPI app object to use:

``` bash
$ fastapi dev main.py
```

But you would have to remember to pass the correct path every time you call the fastapi command.

Additionally, other tools might not be able to find it, for example the VS Code Extension or FastAPI Cloud, so it is recommended to use the entrypoint in pyproject.toml.

## Deploy your app (optional)
You can optionally deploy your FastAPI app to FastAPI Cloud, go and join the waiting list if you haven't. 🚀

If you already have a FastAPI Cloud account (we invited you from the waiting list 😉), you can deploy your application with one command.

Before deploying, make sure you are logged in:
``` bash
fastapi login
```

Then deploy your app:

``` bash
fastapi deploy

Deploying to FastAPI Cloud...

✅ Deployment successful!

🐔 Ready the chicken! Your app is ready at https://myapp.fastapicloud.dev
```
That's it! Now you can access your app at that URL.

## Recap, step by step
### Step 1: import FastAPI
``` Python 3.10+
from fastapi import FastAPI
```
FastAPI is a Python class that provides all the functionality for your API.
FastAPI is a class that inherits directly from Starlette. You can use all the Starlette functionality with FastAPI too.

### Step 2: create a FastAPI "instance"
``` Python 3.10+
app = FastAPI()
```
Here the app variable will be an "instance" of the class FastAPI.

This will be the main point of interaction to create all your API.

### Step 3: create a path operation
#### Path
"Path" here refers to the last part of the URL starting from the first `/`.

So, in a URL like:  `https://example.com/items/foo`, the path would be: `/items/foo`

A "path" is also commonly called an "**endpoint**" or a "**route**".

#### Operation
"Operation" here refers to one of the HTTP "methods".

One of:
- `POST`: to create data
- `GET`
- `PUT`: to update data
- `DELETE`

...and the more exotic ones:
- `OPTIONS`
- `HEAD`
- `PATCH`
- `TRACE`

In the HTTP protocol, you can communicate to each path using one (or more) of these "methods".

#### Define a path operation decorator
``` Python 3.10+
@app.get("/")
```

The @app.get("/") tells FastAPI that the function right below is in charge of handling requests that go to:
- the path /
- using a get operation

You can also use the other operations:
- @app.post()
- @app.put()
- @app.delete()
- ...

Tip:

You are free to use each operation (HTTP method) as you wish. FastAPI doesn't enforce any specific meaning.

The information here is presented as a guideline, not a requirement.

For example, when using GraphQL you normally perform all the actions using only POST operations.

### Step 4: define the path operation function
``` Python 3.10+
async def root():
```

This is our "**path operation function**":
- path: is /.
- operation: is get.
- function: is the function below the "decorator" (below @app.get("/")).

### Step 5: return the content
``` Python 3.10+
    return {"message": "Hello World"}
```

You can return a dict, list, singular values as str, int, etc.

You can also return Pydantic models (you'll see more about that later).

There are many other objects and models that will be automatically converted to JSON (including ORMs, etc). Try using your favorite ones, it's highly probable that they are already supported.
还有许多其他对象和模型会被自动转换为JSON（包括ORM等）。试试你常用的那些，它们很可能已经得到支持。

### Step 6: Deploy it
Deploy your app to FastAPI Cloud with one command: `fastapi deploy`. 

## Recap
- Import FastAPI.
- Create an app instance.
- Write a path operation decorator using decorators like @app.get("/").
- Define a path operation function; for example, def root(): ....
- Run the development server using the command fastapi dev.
- Optionally deploy your app with fastapi deploy.

--- 

# Path Parameters
You can declare path "parameters" or "variables" with the **same syntax used by Python format strings**:

``` python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id):
    return {"item_id": item_id}
```

The value of the path parameter item_id will be passed to your function as the argument item_id.

So, if you run this example and go to http://127.0.0.1:8000/items/foo, you will see a response of: `{"item_id":"foo"}`

## Path parameters with types 带类型的路径参数
``` python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def read_item(item_id: int):
    return {"item_id": item_id}
```

This will give you editor support inside of your function, with error checks, completion, etc.

## Data conversion 数据转换
If you run this example and open your browser at http://127.0.0.1:8000/items/3, you will see a response of:
``` python
{"item_id":3}
```

Notice that the value your function received (and returned) is 3, as a Python int, not a string "3".

So, with that type declaration, FastAPI gives you automatic request "parsing".

## Data validation 数据验证
But if you go to the browser at http://127.0.0.1:8000/items/foo, you will see a nice HTTP error of:
``` bash
{
  "detail": [
    {
      "type": "int_parsing",
      "loc": [
        "path",
        "item_id"
      ],
      "msg": "Input should be a valid integer, unable to parse string as an integer",
      "input": "foo"
    }
  ]
}
```
because the path parameter item_id had a value of "foo", which is not an int.

The same error would appear if you provided a float instead of an int, as in: http://127.0.0.1:8000/items/4.2

So, with the same Python type declaration, FastAPI gives you data validation.

Notice that the error also clearly states exactly the point where the validation didn't pass.

This is incredibly helpful while developing and debugging code that interacts with your API.

## Pydantic
All the data validation is performed under the hood by Pydantic, so you get all the benefits from it. And you know you are in good hands.
> 所有数据验证都在幕后由Pydantic执行，因此您可以从中获得所有优势。而且您知道自己得到了可靠的保障。

You can use the same type declarations with str, float, bool and many other complex data types.
> 你可以对str、float、bool以及许多其他复杂数据类型使用相同的类型声明。

Several of these are explored in the next chapters of the tutorial.
> 教程的后续章节将探讨其中的几个。

## Order matters 顺序很重要
When creating path operations, you can find situations where you have a fixed path.
> 在创建路径操作时，你会遇到存在固定路径的情况。

Like /users/me, let's say that it's to get data about the current user.
> 就像/users/me，假设它是用于获取当前用户的数据。

And then you can also have a path /users/{user_id} to get data about a specific user by some user ID.
> 然后你还可以有一个路径/users/{user_id}，通过某个用户ID获取特定用户的数据。

Because path operations are evaluated in order, you need to make sure that the path for /users/me is declared before the one for /users/{user_id}:
> 由于路径操作是按顺序进行评估的，因此你需要确保 `/users/me` 的路径在 `/users/{user_id}` 的路径之前声明：

``` Python 3.10+
from fastapi import FastAPI

app = FastAPI()


@app.get("/users/me")
async def read_user_me():
    return {"user_id": "the current user"}


@app.get("/users/{user_id}")
async def read_user(user_id: str):
    return {"user_id": user_id}
```
Otherwise, the path for /users/{user_id} would match also for /users/me, "thinking" that it's receiving a parameter user_id with a value of "me".
> 否则，/users/{user_id} 这个路径也会匹配 /users/me，“认为”它收到了一个值为 "me" 的参数 user_id。

Similarly, you cannot redefine a path operation:
> 同样，你不能重复定义路径操作：

``` Python 3.10+

from fastapi import FastAPI

app = FastAPI()


@app.get("/users")
async def read_users():
    return ["Rick", "Morty"]


@app.get("/users")
async def read_users2():
    return ["Bean", "Elfo"]
```

The first one will always be used since the path matches first.
> 由于路径首先匹配，第一个总是会被使用。

## Predefined values 预定义值
If you have a path operation that receives a path parameter, but you want the possible valid path parameter values to be **predefined**, you can use a standard Python **Enum**.
> 如果你有一个接收路径参数的路径操作，但你希望路径参数可能的有效值是预定义的，你可以使用标准的Python枚举（Enum）。

``` python
from enum import Enum

from fastapi import FastAPI


class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"


app = FastAPI()


@app.get("/models/{model_name}")
async def get_model(model_name: ModelName):
    if model_name is ModelName.alexnet:
        return {"model_name": model_name, "message": "Deep Learning FTW!"}

    if model_name.value == "lenet":
        return {"model_name": model_name, "message": "LeCNN all the images"}

    return {"model_name": model_name, "message": "Have some residuals"}
```

### Create an Enum class
Import Enum and create a sub-class that inherits from str and from Enum.
> 导入Enum并创建一个继承自str和Enum的子类。

By inheriting from str the API docs will be able to know that the values must be of type string and will be able to render correctly.
> 通过继承str，API文档将能够知晓这些值的类型必须为string，并能够正确呈现。

Then create class attributes with fixed values, which will be the available valid values:
> 然后创建具有固定值的类属性，这些属性将是可用的有效值：

``` python
from enum import Enum

class ModelName(str, Enum):
    alexnet = "alexnet"
    resnet = "resnet"
    lenet = "lenet"
```

### Declare a path parameter
Then create a path parameter with a type annotation using the enum class you created (ModelName):
``` python

async def get_model(model_name: ModelName):
```

### Working with Python enumerations
The value of the path parameter will be an enumeration member.

#### Compare enumeration members
You can compare it with the enumeration member in your created enum ModelName:
``` python
    if model_name is ModelName.alexnet:
```

#### Get the enumeration value
You can get the actual value (a str in this case) using model_name.value, or in general, your_enum_member.value:
``` python
    if model_name.value == "lenet":
```

#### Return enumeration members
You can return enum members from your path operation, even nested in a JSON body (e.g. a dict).

They will be converted to their corresponding values (strings in this case) before returning them to the client:

``` python
        return {"model_name": model_name, "message": "Deep Learning FTW!"}

        return {"model_name": model_name, "message": "LeCNN all the images"}

    return {"model_name": model_name, "message": "Have some residuals"}
```
In your client you will get a JSON response like:

``` json
{
  "model_name": "alexnet",
  "message": "Deep Learning FTW!"
}
```

## Path parameters containing paths 包含路径的路径参数
Let's say you have a path operation with a path /files/{file_path}.

But you need file_path itself to **contain a path**, like home/johndoe/myfile.txt.
> 但你需要file_path本身包含一个路径，例如home/johndoe/myfile.txt。

So, the URL for that file would be something like: /files/home/johndoe/myfile.txt.

### OpenAPI support
OpenAPI doesn't support a way to declare a path parameter to contain a path inside, as that could lead to scenarios that are difficult to test and define.
> OpenAPI不支持声明一个路径参数包含路径的方式，因为这可能会导致难以测试和定义的场景。

Nevertheless, you can still do it in FastAPI, using one of the internal tools from Starlette.
> 不过，你仍然可以在FastAPI中使用Starlette的一个内部工具来实现这一点。

And the docs would still work, although not adding any documentation telling that the parameter should contain a path.
> 而且文档仍然可以使用，尽管没有添加任何说明来表明该参数应包含一个路径。

### Path convertor
Using an option directly from Starlette you can declare a path parameter containing a path using a URL like:
通过直接使用Starlette中的一个选项，你可以使用如下所示的URL来声明包含路径的路径参数：

``` bash
/files/{file_path:path}
```

In this case, the name of the parameter is file_path, and the last part, :path, tells it that the parameter should match any path.
> 在这种情况下，参数名称是file_path，而最后一部分:path表明该参数应匹配任何路径。

So, you can use it with:

``` Python 3.10+

from fastapi import FastAPI

app = FastAPI()


@app.get("/files/{file_path:path}")
async def read_file(file_path: str):
    return {"file_path": file_path}
```
Tip:

You might need the parameter to contain /home/johndoe/myfile.txt, with a leading slash (/).
> 你可能需要该参数包含/home/johndoe/myfile.txt，且以斜杠（/）开头。

In that case, the URL would be: /files//home/johndoe/myfile.txt, with a double slash (//) between files and home.
> 在这种情况下，URL 会是：/files//home/johndoe/myfile.txt，其中在 files 和 home 之间有一个双斜杠（//）。

## Recap
With FastAPI, by using short, intuitive and standard Python type declarations, you get:
> 使用FastAPI，通过使用简短、直观且标准的Python类型声明，你将获得：

- Editor support: error checks, autocompletion, etc.
- Data "parsing"
- Data validation
- API annotation and automatic documentation
- And you only have to declare them once.

That's probably the main visible advantage of FastAPI compared to alternative frameworks (apart from the raw performance).
> 这可能是 FastAPI 与其他框架相比的主要明显优势（除了原始性能之外）。
