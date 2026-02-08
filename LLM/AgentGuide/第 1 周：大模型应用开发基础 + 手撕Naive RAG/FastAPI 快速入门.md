## 后端基础：FastAPI 快速入门
### Python Types Intro（类型简介）

Python has support for optional "type hints" (also called "type annotations").
> Python支持可选的“类型提示”（也称为“类型注解”）。

These "type hints" or annotations are a special syntax that allow declaring the type of a variable.
> 这些“类型提示”或注释是一种特殊的语法，可用于声明变量的类型。

By declaring types for your variables, editors and tools can give you better support.
> 通过为变量声明类型，编辑器和工具可以为你提供更好的支持。

FastAPI is all based on these type hints, they give it many advantages and benefits.
> FastAPI 完全基于这些类型提示，它们为其带来了诸多优势和益处。

### Motivation 动机
**Python 3.9+**

```python 3.9+
def get_full_name(first_name, last_name):
    full_name = first_name.title() + " " + last_name.title()
    return full_name

print(get_full_name("john", "doe"))

# output: John Doe
```

The function does the following: 

- Takes a first_name and last_name. 
- Converts the first letter of each one to upper case with title().
- Concatenates them with a space in the middle.

#### Edit it
It's a very simple program. 

But now imagine that you were writing it from scratch.
> 但现在想象一下，你要从头开始编写它。

At some point you would have started the definition of the function, you had the parameters ready...
> 在某个时刻，你可能已经开始定义这个函数，并且准备好了参数……

But then you have to call "that method that converts the first letter to upper case".
> 但接下来你必须调用“那个将首字母转换为大写的方法”。

Was it `upper`? Was it `uppercase`? `first_uppercase`? `capitalize`?
> 是upper吗？是uppercase吗？first_uppercase？capitalize？

Then, you try with the old programmer's friend, editor autocompletion.
> 然后，你会尝试使用程序员的老伙计——编辑器自动补全功能。

You type the first parameter of the function, `first_name`, then a dot (`.`) and then hit `Ctrl+Space` to trigger the completion.
> 你输入函数的第一个参数first_name，然后输入一个点（.），接着按Ctrl+Space来触发补全。

But, sadly, you get nothing useful:

![自动补全无效](../pics/自动补全无效.png "自动补全无效")

#### Add types
Let's modify a single line from the previous version.
> 让我们修改上一版本中的一行内容。

We will change exactly this fragment, the parameters of the function, from: `first_name, last_name` to `first_name: str, last_name: str`
> 我们将精确修改这部分内容，即函数的参数，  

Those are the "type hints": 
``` Python 3.9+
def get_full_name(first_name: str, last_name: str):
    full_name = first_name.title() + " " + last_name.title()
    return full_name

print(get_full_name("john", "doe"))
```
That is not the same as declaring default values like would be with: `first_name="john", last_name="doe"`
> 这与像这样声明默认值并不相同：
    
It's a different thing. We are using colons (:), not equals (=).

And adding type hints normally doesn't change what happens from what would happen without them.

But now, imagine you are again in the middle of creating that function, but with type hints.
> 但现在，想象一下你再次处于创建该函数的过程中，但这次带有类型提示。

At the same point, you try to trigger the autocomplete with `Ctrl+Space` and you see:

![自动补全生效](../pics/自动补全生效.png "自动补全生效")

With that, you can scroll, seeing the options, until you find the one that "rings a bell":
> 有了这个，你可以滚动查看选项，直到找到那个“听起来耳熟”的选项：

![通过自动补全找到目标方法](../pics/通过自动补全找到目标方法.png "通过自动补全找到目标方法")

### More motivation
Check this function, it already has type hints:

```Python 3.9+
def get_name_with_age(name: str, age: int):
    name_with_age = name + " is this old: " + age
    return name_with_age
```
Because the editor knows the types of the variables, you don't only get completion, you also get error checks:

![python类型帮助error check](../pics/python类型帮助error check.png "python类型帮助error check")

Now you know that you have to fix it, convert `age` to a string with `str(age)`:

```Python 3.9+
def get_name_with_age(name: str, age: int):
    name_with_age = name + " is this old: " + str(age)
    return name_with_age
```
### Declaring types
You just saw the main place to declare type hints. As function parameters.This is also the main place you would use them with FastAPI.
> 你刚刚看到了声明类型提示的主要位置，也就是函数参数处。这也是在 FastAPI 中使用它们的主要地方。

#### Simple types
You can declare all the standard Python types, not only str. You can use, for example:
- `int`
- `float`
- `bool`
- `bytes`

```Python 3.9+
def get_items(item_a: str, item_b: int, item_c: float, item_d: bool, item_e: bytes):
    return item_a, item_b, item_c, item_d, item_e
```

#### Generic types with type parameters
There are some data structures that can contain other values, like dict, list, set and tuple. And the internal values can have their own type too.
> 有些数据结构可以包含其他值，例如dict、list、set和tuple。而且内部的值也可以有它们自己的类型。

These types that have internal types are called "generic" types. And it's possible to declare them, even with their internal types.
> 这些包含内部类型的类型被称为“泛型”类型。我们甚至可以连同它们的内部类型一起声明这些泛型类型。

To declare those types and the internal types, you can use the standard Python module typing. It exists specifically to support these type hints.
> 要声明这些类型和内部类型，您可以使用标准的Python模块typing。它的存在就是专门为了支持这些类型提示。

##### Newer versions of Python
The syntax using `typing` is compatible with all versions, from Python 3.6 to the latest ones, including Python 3.9, Python 3.10, etc.
> 使用`typing`的语法与所有版本兼容，从Python 3.6到最新版本，包括Python 3.9、Python 3.10等。

As Python advances, newer versions come with improved support for these type annotations and in many cases you won't even need to import and use the `typing` module to declare the type annotations.
> 随着Python的发展，更新的版本对这些类型注解提供了更好的支持，而且在很多情况下，你甚至不需要导入和使用`typing`模块来声明类型注解。

If you can choose a more recent version of Python for your project, you will be able to take advantage of that extra simplicity.
> 如果你的项目可以选择更新版本的Python，你将能够利用这种额外的简洁性。

In all the docs there are examples compatible with each version of Python (when there's a difference).
> 所有文档中都包含与各个Python版本兼容的示例（当存在差异时）。

For example "Python 3.6+" means it's compatible with Python 3.6 or above (including 3.7, 3.8, 3.9, 3.10, etc). And "Python 3.9+" means it's compatible with Python 3.9 or above (including 3.10, etc).
> 例如，“Python 3.6+”表示它兼容Python 3.6及更高版本（包括3.7、3.8、3.9、3.10等）。而“Python 3.9+”表示它兼容Python 3.9及更高版本（包括3.10等）。

If you can use the latest versions of Python, use the examples for the latest version, those will have the best and simplest syntax, for example, "Python 3.10+".
> 如果你能使用最新版本的Python，请使用最新版本的示例，这些示例将具有最佳且最简单的语法，例如“Python 3.10+”。

##### List
For example, let's define a variable to be a `list` of `str`.
> 例如，我们来定义一个变量，使其成为一个`list`，其中包含`str`。

Declare the variable, with the same colon (`:`) syntax.
> 使用相同的冒号（`:`）语法声明变量。

As the type, put `list`. 
> 类型请填写 list。

As the list is a type that contains some internal types, you put them in square brackets:
> 由于列表是一种包含某些内部类型的类型，你需要将这些内部类型放在方括号中：

```Python 3.9+

def process_items(items: list[str]):
    for item in items:
        print(item)
```

> Those internal types in the square brackets are called "type parameters".
>> 方括号中的这些内部类型被称为“类型参数”。
>
> In this case, str is the type parameter passed to list.
>> 在这种情况下，str 是传递给 list 的类型参数。

That means: "the variable items is a list, and each of the items in this list is a str".
> 这意味着：“变量items是一个list，且该列表中的每个元素都是一个str”。

By doing that, your editor can provide support even while processing items from the list:
> 通过这种方式，你的编辑器即使在处理列表中的元素时也能提供支持：

![python泛型内部的自动补全](../pics/python泛型内部的自动补全.png "python泛型内部的自动补全")

Without types, that's almost impossible to achieve.
> 没有类型的话，这几乎是不可能实现的。

Notice that the variable item is one of the elements in the list items.
> 请注意，变量item是列表items中的元素之一。

And still, the editor knows it is a `str`, and provides support for that.
> 而且，编辑器知道它是一个字符串，并为此提供支持。

##### Tuple and Set
You would do the same to declare tuples and sets:
> 声明tuple和set时，你也会采用同样的方式：

```Python 3.9+
def process_items(items_t: tuple[int, int, str], items_s: set[bytes]):
    return items_t, items_s
```
This means:

- The variable items_t is a tuple with 3 items, an int, another int, and a str.
- > 变量items_t是一个包含3个元素的tuple，分别是一个int、另一个int和一个str。

- The variable items_s is a set, and each of its items is of type bytes.
- > 变量items_s是一个set，其每个元素的类型都是bytes。

##### Dict
To define a dict, you pass 2 type parameters, separated by commas: keys and values.
> 要定义一个dict，需要传入两个类型参数，用逗号分隔。

```Python 3.9+
def process_items(prices: dict[str, float]):
    for item_name, item_price in prices.items():
        print(item_name)
        print(item_price)
```

The variable prices is a dict: 

- The keys of this dict are of type str (let's say, the name of each item).
- > 这个dict的键是str类型（比如说，每个物品的名称）。

The values of this dict are of type float (let's say, the price of each item).

##### Union
You can declare that a variable can be any of several types, for example, an int or a str.
> 你可以声明一个变量可以是多种类型中的任意一种，例如，int或str。

In Python 3.6 and above (including Python 3.10) you can use the `Union` type from `typing` and put inside the square brackets the possible types to accept.
> 在Python 3.6及更高版本（包括Python 3.10）中，你可以使用typing模块中的Union类型，并在方括号中放入可接受的可能类型。

In Python 3.10 there's also a new syntax where you can put the possible types separated by a vertical bar (|).
> 在Python 3.10中，还有一种新语法，你可以用竖线（|）分隔可能的类型。


```Python 3.10+
def process_item(item: int | str):
    print(item)
```
```Python 3.9+
from typing import Union


def process_item(item: Union[int, str]):
    print(item)
```

In both cases this means that item could be an int or a str.
> 在这两种情况下，这意味着item可以是一个int或一个str。

##### Possibly `None`
You can declare that a value could have a type, like str, but that it could also be None.
> 你可以声明一个值可能具有某种类型，比如str，但它也可能是None。

In Python 3.6 and above (including Python 3.10) you can declare it by importing and using Optional from the typing module.
> 在Python 3.6及更高版本（包括Python 3.10）中，你可以通过从typing模块导入并使用Optional来声明它。

```python
from typing import Optional


def say_hi(name: Optional[str] = None):
    if name is not None:
        print(f"Hey {name}!")
    else:
        print("Hello World")
```
Using `Optional[str]` instead of just `str` will let the editor help you detect errors where you could be assuming that a value is always a str, when it could actually be `None` too.
> 使用Optional[str]而不是单纯的str，可以让编辑器帮助你检测一些错误，这些错误可能是由于你假设某个值始终是str类型，但实际上它也可能是None类型而导致的。

`Optional[Something]` is actually a shortcut for `Union[Something, None]`, they are equivalent.
> Optional[Something]实际上是Union[Something, None]的快捷方式，它们是等效的。

This also means that in Python 3.10, you can use `Something | None`:
> 这也意味着在Python 3.10中，你可以使用Something | None：

```Python 3.10+
def say_hi(name: str | None = None):
    if name is not None:
        print(f"Hey {name}!")
    else:
        print("Hello World")
```
```Python 3.9+
from typing import Optional


def say_hi(name: Optional[str] = None):
    if name is not None:
        print(f"Hey {name}!")
    else:
        print("Hello World")
```
```Python 3.9+ alternative
from typing import Union


def say_hi(name: Union[str, None] = None):
    if name is not None:
        print(f"Hey {name}!")
    else:
        print("Hello World")
```

##### Using Union or Optional
If you are using a Python version below 3.10, here's a tip from my very subjective point of view:
> 如果你使用的Python版本低于3.10，以下是从我非常主观的角度给出的一个建议：

- 🚨 Avoid using `Optional[SomeType]`.
- Instead, ✨ use `Union[SomeType, None]` ✨.

Both are equivalent and underneath they are the same, but I would recommend `Union` instead of `Optional` because the word "optional" would seem to imply that the value is optional, and it actually means "it can be None", even if it's not optional and is still required.
> l两者是等价的，本质上是一样的，但我建议使用`Union`而不是`Optional`，因为“optional”这个词似乎暗示该值是可选的，而实际上它的意思是“它可以是None”，即便它不是可选的，仍然是必填的。

I think `Union[SomeType, None]` is more explicit about what it means.
> 我认为Union[SomeType, None]的含义更为明确。

It's just about the words and names. But those words can affect how you and your teammates think about the code.
> 这只关乎词语和名称。但这些词语会影响你和你的队友对代码的看法。

As an example, let's take this function:
> 举个例子，让我们看看这个函数：

```Python 3.9+
from typing import Optional


def say_hi(name: Optional[str]):
    print(f"Hey {name}!")
```

The parameter name is defined as `Optional[str]`, but it is not optional, you cannot call the function without the parameter:
> 参数name被定义为Optional[str]，但它是非可选的，调用该函数时不能缺少此参数：

`say_hi()  # Oh, no, this throws an error! 😱`

The name parameter is still required (not optional) because it doesn't have a default value. Still, name accepts None as the value:
> name参数是仍然必需的（不是可选的），因为它没有默认值。不过，name接受None作为值：


`say_hi(name=None)  # This works, None is valid 🎉`
The good news is, once you are on Python 3.10 you won't have to worry about that, as you will be able to simply use | to define unions of types:
> 好消息是，一旦你使用Python 3.10，就不必担心这个问题了，因为你可以直接使用|来定义类型的并集：

```Python 3.10+

def say_hi(name: str | None):
    print(f"Hey {name}!")
```
And then you won't have to worry about names like `Optional` and `Union`. 😎
> 这样你就不必担心像Optional和Union这样的名称了。😎

##### Generic types
These types that take type parameters in square brackets are called `Generic` types or `Generics`, for example:
> 这些在方括号中带有类型参数的类型被称为泛型类型或泛型，例如：

**Python 3.9+**

You can use the same builtin types as generics (with square brackets and types inside):
> 你可以将相同的内置类型用作泛型（使用方括号并在其中包含类型）：
- list
- tuple
- set
- dict

And the same as with previous Python versions, from the typing module:
> 和之前的Python版本一样，来自typing模块：
- Union
- Optional
- ...and others.

In Python 3.10, as an alternative to using the generics Union and Optional, you can use the vertical bar (|) to declare unions of types, that's a lot better and simpler.
> 在Python 3.10中，作为使用泛型Union和Optional的替代方法，你可以使用竖线（|）来声明类型联合，这要更好也更简单。

#### Classes as types
You can also declare a class as the type of a variable.
> 你也可以将一个类声明为变量的类型。

Let's say you have a class Person, with a name:
> 假设你有一个 Person 类，它包含一个姓名：

Then you can declare a variable to be of type Person:
> 然后你可以声明一个类型为Person的变量：
```Python 3.9+
class Person:
    def __init__(self, name: str):
        self.name = name


def get_person_name(one_person: Person):
    return one_person.name
```
And then, again, you get all the editor support:
> 然后，同样，你会获得所有的编辑器支持：

![class_as_type支持自动补全](../pics/class_as_type支持自动补全.png "class_as_type支持自动补全")

Notice that this means "`one_perso`n is an instance of the class `Person`".
> 请注意，这意味着“one_person是Person类的一个实例”。

It doesn't mean "`one_person` is the class called `Person`".
> 这并不意味着“one_person是名为Person的类”。

### Pydantic models
Pydantic is a Python library to perform data validation.
> Pydantic 是一个用于执行数据验证的 Python 库。

You declare the "shape" of the data as classes with attributes.
> 你将数据的“形状”声明为带有属性的类。

And each attribute has a type. 
> 并且每个属性都有一个类型。

Then you create an instance of that class with some values and it will validate the values, convert them to the appropriate type (if that's the case) and give you an object with all the data.
> 然后你用一些值创建该类的实例，它会验证这些值，将它们转换为适当的类型（如果情况如此），并为你提供一个包含所有数据的对象。

And you get all the editor support with that resulting object.
> 并且你会获得针对该结果对象的所有编辑器支持。

An example from the official Pydantic docs:
> 来自Pydantic官方文档的一个示例：

```Python 3.10+

from datetime import datetime

from pydantic import BaseModel


class User(BaseModel):
    id: int
    name: str = "John Doe"
    signup_ts: datetime | None = None
    friends: list[int] = []


external_data = {
    "id": "123",
    "signup_ts": "2017-06-01 12:22",
    "friends": [1, "2", b"3"],
}
user = User(**external_data)
print(user)
# > User id=123 name='John Doe' signup_ts=datetime.datetime(2017, 6, 1, 12, 22) friends=[1, 2, 3]
print(user.id)
# > 123
```

```
Info 信息

To learn more about Pydantic, check its docs.
要了解更多关于Pydantic的信息，请查看其文档。
```
FastAPI is all based on Pydantic.

You will see a lot more of all this in practice in the Tutorial - User Guide.
> 在《教程 - 用户指南》中，你会在实践中看到更多这方面的内容。

```Tip 提示

Pydantic has a special behavior when you use Optional or Union[Something, None] without a default value, you can read more about it in the Pydantic docs about Required Optional fields.
当你使用没有默认值的Optional或Union[Something, None]时，Pydantic有一个特殊行为，你可以在Pydantic文档中关于必填可选字段的部分了解更多相关内容。
```
### Type Hints with Metadata Annotations
Python also has a feature that allows putting additional metadata in these type hints using `Annotated`.
> Python 还有一个特性，允许使用 Annotated 在这些类型提示中放入 附加元数据。

Since Python 3.9, Annotated is a part of the standard library, so you can import it from typing.
> 自Python 3.9起，Annotated成为标准库的一部分，因此你可以从typing中导入它。

```Python 3.9+

from typing import Annotated


def say_hello(name: Annotated[str, "this is just metadata"]) -> str:
    return f"Hello {name}"
```
Python itself doesn't do anything with this Annotated. And for editors and other tools, the type is still str.
> Python本身不会对这个Annotated做任何处理。对于编辑器和其他工具来说，其类型仍然是str。

But you can use this space in Annotated to provide FastAPI with additional metadata about how you want your application to behave.
> 但你可以在Annotated部分使用这个空间，向FastAPI提供关于你希望应用程序如何运行的额外元数据。

The important thing to remember is that the first type parameter you pass to Annotated is the actual type. The rest, is just metadata for other tools.
> 需要记住的重要一点是，传递给Annotated的第一个类型参数是实际类型。其余的都只是供其他工具使用的元数据。

For now, you just need to know that Annotated exists, and that it's standard Python. 😎
> 目前，你只需要知道Annotated 存在，并且它是标准的Python即可。😎

Later you will see how powerful it can be.
> 稍后你会看到它有多么强大。

```
Tip 提示

The fact that this is standard Python means that you will still get the best possible developer experience in your editor, with the tools you use to analyze and refactor your code, etc. ✨
这是标准Python这一事实意味着，在你的编辑器中，借助你用于分析和重构代码等的工具，你仍将获得尽可能好的开发体验。✨

And also that your code will be very compatible with many other Python tools and libraries. 🚀
而且你的代码也将与许多其他Python工具和库具有很好的兼容性。🚀
```

### Type hints in FastAPI
FastAPI takes advantage of these type hints to do several things.
> FastAPI 利用这些类型提示来实现多项功能。

With FastAPI you declare parameters with type hints and you get:
> 使用FastAPI时，你可以通过类型提示来声明参数，并且你会获得：

- Editor support.
- Type checks.
...and FastAPI uses the same declarations to:
> ……FastAPI使用相同的声明来：

- **Define requirements**: from request path parameters, query parameters, headers, bodies, dependencies, etc.
- > 定义需求：从请求路径参数、查询参数、标头、主体、依赖项等中获取。
- **Convert data**: from the request to the required type.
- > 转换数据：将请求中的数据转换为所需类型。
- **Validate data**: coming from each request:
- > 验证数据：来自每个请求：
- - Generating automatic errors returned to the client when the data is invalid.
- - > 当数据无效时，生成返回给客户端的自动错误。
- Document the API using OpenAPI:
- > 使用OpenAPI为API生成文档：
- - which is then used by the automatic interactive documentation user interfaces.
- - > 然后，这会被自动交互式文档用户界面所使用。

This might all sound abstract. Don't worry. You'll see all this in action in the Tutorial - User Guide.
> 这听起来可能有些抽象。别担心，你会在《教程 - 用户指南》中看到所有这些的实际应用。

The important thing is that by using standard Python types, in a single place (instead of adding more classes, decorators, etc), FastAPI will do a lot of the work for you.
> 重要的是，通过在一个地方使用标准的Python类型（而不是添加更多的类、装饰器等），FastAPI将为你完成大量工作。

```
Info

If you already went through all the tutorial and came back to see more about types, a good resource is the "cheat sheet" from `mypy`.
如果你已经看完了所有教程，回来想了解更多关于类型的内容，一个不错的资源是来自mypy的“速查表”。
```
