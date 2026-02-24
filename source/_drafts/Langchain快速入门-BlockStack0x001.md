---
title: Langchain快速入门-BlockStack0x001
tags:
---

> 原链接：
>
> 1. https://www.cnblogs.com/ClownLMe/p/19519224
> 2. https://www.cnblogs.com/ClownLMe/p/19523156

## 写在前面

什么是块叠？就是从他人优秀的知识总结中搬出一块来，叠到自己的技术栈上；Put a block of knowledge onto your own coding stack。通过一篇文章（视频、书籍等），发现自己技术栈中的空缺，并发散开去，作最大化的填补。遵循原文顺序、最佳实践优先、扩展思维含量，这就是BlockStack的理念与追求。每月的1号和15号，我都会发布一期，希望能尽可能聊得彻底、翔实。

这是一个系列的文章，一共有六篇。介绍了Langchain的基本使用方法。对于Agent使用越来越广泛的今天，了解这些有很重要的意义。这也是我选择这些文章作为本期BlockStack的理由。

## （一）运行你第一个LLM模型

Langchain是一个开源库，可以让你以最快的方式来构建一个大模型应用，其可以将工具组件、提示词组件和大模型组件等有机融合，可以让一个AI过程抽象成一个个组件的调用流程。同时让大模型配合其他的代码来进行使用，从而达到1+1>2的效果。它支持python和js/ts语言。它通过一系列的规定，来对一个大模型的过程进行标准化。

多说无益，我们先来看一个最小的demo，然后再解读这个代码。为了和原链接一致，我们在这里用的都是python代码。

在demo之前，我们首先要进行一个安装。在当前版本（langchain-core==1.2.8）中，我们需要安装核心`langchain`，还有对应的供应商的专用包，二者缺一不可：例如使用openai，就需要安装`langchain-openai`。在本文中，我们使用硅基流动的模型（因为我还有几块钱的免费没用完），它可以直接使用openai的包，[参考文档在这里](https://reference.langchain.com/python/integrations/langchain_openai)。

```py
from langchain.agents import create_agent

def get_weather(city: str) -> str:
    """Get weather for a given city."""
    return f"It's always sunny in {city}!"

agent = create_agent(
    model="claude-sonnet-4-5-20250929",
    tools=[get_weather],
    system_prompt="You are a helpful assistant",
)

# Run the agent
agent.invoke(
    {"messages": [{"role": "user", "content": "what is the weather in sf"}]}
)
```

这是官网的示例。我们可以看到，首先我们先通过`create_agent`函数创建了一个AI助手，给了他一个工具也就是`get_weather`、模型名称和系统提示。然后直接调用这个Agent。为了能够兼容硅基流动，我们需要对代码进行改造，让它更适合这个供应商，例如下面的代码。

```py
from langchain.agents import create_agent
from langchain.chat_models import init_chat_model

def get_weather(city: str) -> str:
    """Get weather for a given city."""
    return f"It's always sunny in {city}!"

model =init_chat_model(
    model_provider='openai',
    model="Qwen/Qwen3-VL-8B-Instruct",
    api_key="", # 你的APIkey
    # 设置base_url为硅基流动
    base_url='https://api.siliconflow.cn/v1/'
)

agent = create_agent(
    model=model,
    tools=[get_weather],
    system_prompt="You are a helpful assistant"
)

# Run the agent
agent.invoke(
    {"messages": [{"role": "user", "content": "what is the weather in sf"}]}
)
```

目前很多市面上的参考资料（包括笔者最后列出的部分参考资料），根本就没有完全理解Langchain的设计哲学。例如在一些文章中，它会让你直接通过供应商模型类的构造函数来初始化模型，原文章的作者使用了`ChatTongyi`类的构造函数。实际上官方并不推荐这种过于具体的模型构建方式，取而代之的是官方的`init_chat_model`API。同时官方文档中的`model`参数直接给的是模型名称，这个就是一种比较偷懒的写法，对于国内这种无法访问原生URL的环境来说，先定义一个model再传进来反而是比较有利的选择。

上述的代码其实并没有走完一个大模型工作流的完成流程。我们先来剖析一下一个Langchain工作流的运行流程，然后在最后我们再补上一个完整的示例代码。

1. 创建模型

`init_chat_model`函数是Langchain官方推荐的一个通用的初始化模型的方式。它有以下的一些常用参数：

- `model`：模型名称，在不给出provider的情况下，函数也会根据模型名称来自动推断provider。
- `model_provider`：模型的供应商，如果你需要显示指定，不依赖自动推断的话，就需要给这个参数了。具体支持的列表可以参考[这个链接](<https://reference.langchain.com/python/langchain/models/?h=init_chat_model#langchain.chat_models.init_chat_model(model_provider)>)。
- `**kwargs`：这个就是给每个不同的模型和供应商准备的自留地了，根据供应商的不同，所需的名称和类型数量之类的也不相同。例如上文的代码中的`api_key`和`base_url`。可以参考[这个链接](https://reference.langchain.com/python/integrations/)来查看具体的细节。

2. 消息设置

prompt就是大模型所获得的提示词，你可以将他看做一种输入。在Langchain中，模型上下文的基本单位是消息（Messages），模型的所有输入和输出都是借助一条条消息来实现的
在Langchain中的消息分为三类，分别代表着三种不同的角色：system、user和ai。他们对应的类和作用说明如下表所示。

| 角色名称 (Role) | 对应的类        | 作用说明                                                                                                 |
| --------------- | --------------- | -------------------------------------------------------------------------------------------------------- |
| system          | `SystemMessage` | 系统提示词。用于设定 AI 的“人格”、专业背景、行为准则或约束条件。它通常优先级最高，决定了后续对话的基调。 |
| user            | `HumanMessage`  | 用户消息。代表人类发送的内容。这是模型需要直接回答或处理的问题。                                         |
| ai              | `AIMessage`     | AI 消息。代表模型之前的回复。在构建多轮对话（带记忆）时，需要把模型之前的回复传回去。                    |

那么我们到底如何声明一个Messages呢？有下面三种方法。

首先是直接一句话的文本提示，例如当你很简单的调用或者不需要对话上下文的时候，直接塞给模型一个字符串就行。

```py
response = model.invoke("写一个关于春天的俳句")
```

如果你需要上下文记忆等更复杂的使用，那么你就可以通过字典或者消息对象列表来进行更精细化的使用。笔者本人更喜欢使用消息对象列表，因为看起来比较简洁。虽然这里两边的代码都会给出，但是后续还是尽量不适用dict。

```py
# 消息对象列表
from langchain.messages import SystemMessage, HumanMessage, AIMessage

messages = [
    SystemMessage("你是一个诗词专家"),
    HumanMessage("写一个关于春天的俳句"),
    AIMessage("樱花飘落时...")
]
# dict
messages = [
    {"role": "system", "content": "你是一个诗词专家"},
    {"role": "user", "content": "写一个关于春天的俳句"},
    {"role": "assistant", "content": "樱花飘落时..."}
]
response = model.invoke(messages)
```

如果一个模型能够支持工具调用的话，那么我们还可以使用`ToolMessage`类来表示工具消息。等到后面的文章中我们还会再更加详细的介绍。下面我们来介绍一下模型的输出。

通常来讲，如果直接让模型进行输出，那么模型返回的是一个`AIMessage`类，里面包括了content等内容。不过同时，我们也可以让模型遵循特定的结构来进行输出。在Langchain中，模型的结构化输出分三种，一种是Pydantic Model，一种是TypedDict，还有就是JSON格式了。由于笔者不了解Pydantic和typeddict，因此在本文中统一使用JSON格式。



## （二）chain链的应用

Agent是将Model和工具结合起来的产物，他能创建各种任务，自己决定使用什么工具并查找解决方案。一个LLM Agent会循环运行各种不同的工具来实现目标，一直运行到满足停止条件为止。

## 参考资料

- [完整教程：一、初识 LangChain：架构、应用与开发环境部署 - clnchanpin - 博客园](https://www.cnblogs.com/clnchanpin/p/19531897)
- [深入浅出LangChain AI Agent智能体开发教程（一）—认识LangChain&LangGraph本篇分享从L - 掘金](https://juejin.cn/post/7526993716071202851)
- [](https://docs.langchain.com/oss/python/langchain/messages)
- [Models - Docs by LangChain](https://docs.langchain.com/oss/python/langchain/models)
