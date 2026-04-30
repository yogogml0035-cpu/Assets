---
title: "LangChain DeepAgents 速通指南（六）—— DeepAgents SubAgent 子智能体机制"
source: "https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485975&idx=1&sn=c9c1fbde0ac31ff6cca02fe17abeaee4&chksm=c4d003fbf3a78aed1c374b34ba3fc7c403a141a31e6889ff046d22e748b52f8ec8495f1540f2&scene=178&cur_album_id=4073973355617976325&search_click_id=#rd"
author:
  - "[[手摸手教你大模型]]"
published:
created: 2026-04-30
description: "DeepAgents 子智能体通过上下文隔离实现任务分工，本文介绍DeepAgents 字典配置与编译配置两种子智能体创建方式，并通过“伊朗老美分析系统”的搜索+报告案例演示子智能体系统搭建流程。"
tags:
  - "clippings"
---
手摸手教你大模型 *2026年4月3日 20:20*

## 前言

上篇文章 [《LangChain DeepAgents 速通指南（五）—— 快速了解DeepAgents框架及其核心特性》](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485906&idx=1&sn=1024589caa410a2b36aff4f320eba259&scene=21#wechat_redirect) 介绍了 DeepAgents 在任务规划、上下文管理、子智能体并行执行等方面的强大能力，仅需少量代码即可构建出复杂的智能体。上篇的案例演示也展示了一个有趣的现象：即便没有明确定义子智能体，DeepAgents 也能自行创建并执行子智能体。这种自动行为在简单任务场景下非常实用。

不过，当面对更复杂的任务时，更好的做法是主动定义好各个子智能体以及它们之间的协同方式。为此，DeepAgents 提供了完善的子智能体功能支持。本期分享，笔者就和大家一起深入探讨 DeepAgents 的子智能体系统：它到底是什么？如何创建？以及在什么场景下该使用它？

PS:鉴于后台私信越来越多，我建了一些大模型交流群，大家在日常学习生活工作中遇到的大模型知识和问题都可以在群中分享出来大家一起解决！如果大家想交流大模型知识，可以关注我并回复加群

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/I5P6NLY2q6VfYEZWHDIkL8MjQ8aqoCWCTSicq0XiaEufc1T1ibLswKbCk2PEV3ZRey2h4f2WbphX3ghzaV67NZsc2joxK3qQjiaXksMyuuOndlw/640?wx_fmt=webp&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

## 一、子智能体的核心价值

## 1.1 为什么需要子智能体参与？

上期文章提到，DeepAgents 实现了 LangChain `create_agent` 从简单到复杂应用的华丽升级。而子智能体功能，正是 DeepAgents 框架的核心能力之一。

在构建复杂的智能体应用时，大家常常面临一个难题：随着任务步骤增多，单一智能体的上下文会变得臃肿，不仅影响性能，还容易让模型“迷失”在细节中。对此，DeepAgents 给出的解决方案之一就是引入子智能体。

在 DeepAgents 中，主智能体可以将任务委派给各个子智能体，这样做有两个明显的好处：

1. **保持主智能体的上下文清爽**
	子任务的执行细节不会挤占主智能体的上下文 token 窗口，实现了上下文的隔离。
2. **子智能体更专注**
	每个子智能体只负责某一特定职责，配合针对性的工具，可以显著提升任务执行的效率与成功率。

当构建一个 DeepAgent 时，除了配置主智能体的模型和工具，还可以为其配备各种职责的子智能体，而每个子智能体要配置与其职责相对应的工具。

## 1.2 DeepAgents 子智能体的执行原理

DeepAgents 在构建时会自动注入一个名为 `task` 的工具，该工具负责选择合适的子智能体去执行某项任务。子智能体的上下文与主智能体完全隔离——子智能体的中间执行过程不会暴露给主智能体，只将最终结果传递给主智能体。这就保证了主智能体的上下文中不包含过多不必要的子任务细节，使其能够更专注于高层的任务协调与规划。

大家不妨回忆一下上期文章 [《LangChain DeepAgents 速通指南（五）—— 快速了解DeepAgents框架及其核心特性》](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485906&idx=1&sn=1024589caa410a2b36aff4f320eba259&scene=21#wechat_redirect) 中的案例：主智能体使用 `task` 工具，调用两个子智能体分别去搜集伊朗和美国的情况，得到结果后，主智能体只需对结果进行总结提炼即可。

## 二、DeepAgents 子智能体的创建与配置方式

DeepAgents 作为一款便捷高效的深度智能体搭建框架，提供了两种创建子智能体的方式，能够满足从快速原型验证到精细定制开发的不同需求。

## 方法一：基于字典的基本创建

1\. 首先创建模型与工具对象。这里同样使用 DeepSeek 模型，并创建一个模拟网络搜索的工具

```python
from dotenv import load_dotenv
from deepagents import create_deep_agent
from langchain_core.tools import tool
from langchain_deepseek import ChatDeepSeek

load_dotenv()

@tool
def internet_search():
    print("模拟网络搜索功能")

model = ChatDeepSeek(model="deepseek-chat")
```

2\. 使用 Python 字典形式配置子智能体（SubAgent），包括 `name` 、 `description` 、 `system_prompt` 和 `tools` 等字段。其中 `name` 是子智能体的名字； `description` 非常重要，主智能体会通过它了解每个子智能体的功能； `system_prompt` 用于让子智能体明确自身的职责； `tools` 则用于为每个子智能体配备针对性的工具。除了这些必备参数，还有一些可选参数： `model` 可以为该子智能体单独配置模型，若不配置则默认与主智能体使用同一模型； `middleware` 用于为子智能体配置所需的中间件； `interrupt_on` 用于为子智能体配置“人在回路”机制。这些与普通 Agent 的创建方式相同，这里不再赘述。

```python
internet_subagent = {
    "name": "internet_agent",
    "description": "实用网络工具从网络中搜索信息",
    "system_prompt": "你是一个网络搜索智能体，擅长从网络中搜索主智能体需要的关键信息",
    "tools": [internet_search],
    "model": model,
}
```

3\. 最后，将配置好的子智能体列表传入 `create_deep_agent` 函数，即可完成智能体的创建。

```python
agent = create_deep_agent(
    model=model,
    subagents=[internet_subagent],
)
```

## 方法二：编译后的子智能体

大家很多时候已经使用langchain `create_agent` 创建了很多子智能体，要想把这些子智能体有机结合起来解决复杂任务，当然也可以使用DeepAgent。

1\. 当遇到更灵活、更具定制性的需求时，可以先通过标准的 `create_agent` 函数在外部创建一个子智能体对象，参数配置与大家熟悉的 LangChain `create_agent` 方式完全相同。

```python
from dotenv import load_dotenv
from deepagents import CompiledSubAgent, create_deep_agent
from langchain.agents import create_agent
from langchain_core.tools import tool
from langchain_deepseek import ChatDeepSeek

load_dotenv()

@tool
def internet_search():
    print("模拟网络搜索功能")

model = ChatDeepSeek(model="deepseek-chat")
custom_agent = create_agent(
    model=model,
    tools=[internet_search],
    system_prompt="你是一个网络搜索智能体",
)
```

2\. 为创建好的智能体通过 `CompiledSubAgent` 编译为子智能体，并为该子智能体添加名称与描述，以确保主智能体能够理解其职责。

```python
research_subagent = {
    "name": "research-agent",
    "description": "用于深度搜索网络信息",
    "system_prompt": "你是一个网络搜索大师，可以调用网络搜索工具搜索用户想了解的内容",
    "tools": [internet_search],
}
summary_agent = create_agent(
    model=model,
    system_prompt="你用来根据现有资料总结并提供用户想要的短篇报告",
)
summary_subagent = CompiledSubAgent(
    name="summary-agent",
    description="用来根据提供的新闻或搜索信息编写短篇报告，500字以内",
    runnable=summary_agent,
)
```

3\. 最后，将编译好的子智能体对象传入主智能体。

```python
agent = create_deep_agent(
    model=model,
    subagents=[custom_subagent],
)
```

这种方式可以实现更精细的自定义，比如为子智能体单独配置回调、检查点等高级功能。

## 三、DeepAgents 子智能体实战

要充分掌握 DeepAgents 子智能体的开发方法，动手实践是最好的途径。笔者将通过一个名为“国际情报分析师”的小 Demo，带大家掌握 DeepAgents SubAgent 功能的使用方法。

## 1\. 下载依赖包

本项目会用到 `tavily` 搜索工具。在 `deepagents` 环境目录下执行 `pip install tavily-python` 安装 Tavily 搜索依赖，同时将 Tavily 官网上注册的 API Key 填入项目文件夹下的 `.env` 环境文件中。关于 Tavily 搜索工具的注册与使用，可以参考笔者的文章： [《深入浅出 LangChain AI Agent 智能体开发教程（六）——两行代码 LangChain Agent API 快速搭建智能体》](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247484602&idx=1&sn=4fc3063e74abefad70b3b11440cbd480&scene=21#wechat_redirect) 。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

## 2\. 导入依赖包，定义模型和搜索工具

```python
import os
from typing import Literal

from dotenv import load_dotenv
from deepagents import CompiledSubAgent, create_deep_agent
from langchain.agents import create_agent
from langchain_core.tools import tool
from langchain_deepseek import ChatDeepSeek
from tavily import TavilyClient

load_dotenv()
tavily_client = TavilyClient(api_key=os.getenv("TAVILY_API_KEY"))
model = ChatDeepSeek(model="deepseek-chat")

@tool
def internet_search(
    query: str,
    max_results: int = 5,
    topic: Literal["general", "news", "finance"] = "general",
    include_raw_content: bool = False,
):
    """使用 Tavily API 执行互联网搜索，获取实时或最新的网络信息。"""
    return tavily_client.search(
        query,
        max_results=max_results,
        include_raw_content=include_raw_content,
        topic=topic,
    )
```

## 3\. 编写搜索子智能体和总结分析智能体

这里分别使用基于字典创建与编译子智能体两种方法。

```python
research_subagent = {
    "name": "research-agent",
    "description": "用于深度搜索网络信息",
    "system_prompt": "你是一个网络搜索大师，可以调用网络搜索工具搜索用户想了解的内容",
    "tools": [internet_search],
}
summary_agent = create_agent(
    model=model,
    system_prompt="你用来根据现有资料总结并提供用户想要的短篇报告",
)
summary_subagent = CompiledSubAgent(
    name="summary-agent",
    description="用来根据提供的新闻或搜索信息编写短篇报告，500字以内",
)
```

## 4\. 创建 DeepAgent 智能体并传入子智能体

```python
agent = create_deep_agent(
    model=model,
    subagents=[research_subagent, summary_subagent],
)
```

## 5\. 执行任务并观察输出

不知道大家最近有没有和笔者一样，每天都在为“朗哥”与“老美”的对抗而揪心？今天笔者就通过一个任务，来论证“伊朗必胜，美国必败”的原因。格式化输出的程序与上篇文章 [《LangChain DeepAgents 速通指南（五）—— 快速了解 DeepAgents 框架及其核心特性》](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485906&idx=1&sn=1024589caa410a2b36aff4f320eba259&scene=21#wechat_redirect) 中的一致，这里不再赘述：

```python
step_num = 0
query = "请分析2026年4月3日伊朗和美国战事的情况，并撰写短篇报告分析为什么美国注定失败，500字以内的报告"
for event in agent.stream(
    {"messages": [{"role": "user", "content": query}]},
    stream_mode="values",
):
    ...
```

## 6\. 观察执行结果

首先，主智能体将任务分配给子智能体进行查询并总结结果：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

信息搜集完成后，主智能体调用了另一个子智能体来撰写报告：

最终撰写的报告分析了美国必定失败的原因——只能说“朗哥”厉害！

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

## 四、子智能体适用场景 vs. 不适用场景

子智能体能够有效解决上下文膨胀问题，将子任务中的局部冗余信息与主任务流程隔离开来。它主要适用于以下场景：

1. **多步骤任务**
	—— 避免单一智能体的上下文变得杂乱。
2. **专业领域**
	—— 需要特殊说明或使用特定工具的子任务。
3. **多模型混合**
	—— 需要综合使用不同模型以提升整体性能。
4. **高层协调**
	—— 希望主智能体更专注于规划与协调，而非执行细节。

但子智能体也不是万能的。以下场景 **不建议** 使用：

1. **简单的单步任务**
	—— 引入子智能体反而会增加复杂度和开销。
2. **依赖中间细节**
	—— 如果任务的最终完成高度依赖中间步骤的大量详尽信息，上下文的隔离会导致关键信息丢失。
3. **成本敏感任务**
	—— 由于子智能体的上下文完全独立，主智能体中的一些历史信息可能会在子智能体中被重复获取，从而导致不必要的成本增加。

**注意：** 任何 DeepAgents 对象在创建后，内部都已默认配备了一个名为 `general_purpose` 的子智能体。这个默认子智能体共享与主智能体完全相同的模型、工具和系统提示词。这既解释了上期文章 [《LangChain DeepAgents 速通指南（五）—— 快速了解DeepAgents框架及其核心特性》](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485906&idx=1&sn=1024589caa410a2b36aff4f320eba259&scene=21#wechat_redirect) 中“未传入子智能体也能并行调用子智能体完成任务”的现象，也提醒大家： **对于每一个你自定义的 SubAgent，其 `description` 字段必须写得清晰明确** ，否则主智能体可能会错误地选择默认的通用子智能体，而不是你期望的那一个。

以上就是本期分享的全部内容，本文的全部代码可关注笔者的微信公众号 **大模型真好玩** ，每期分享涉及的代码均可在公众号私信: **LangChain智能体开发** 免费获取。

## 五、总结

本期分享了DeepAgents的子智能体机制，子智能体机制通过上下文隔离与职责分离，有效解决复杂任务中的上下文膨胀问题。本文介绍了两种创建方式（字典配置与编译子智能体），并通过实战案例展示了搜索与报告撰写分工。合理使用子智能体能提升效率，但简单任务或成本敏感场景需谨慎，确保务必为每个子智能体配置清晰的 `description` ~。下期内容笔者将分享DeepAgents的流式输出机制，学会流式输出机制大家就可以合理的控制智能体系统后端的输出，敬请期待！

[《深入浅出LangChain&LangGraph AI Agent 智能体开发》](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk3NTA2OTMxNQ==&action=getalbum&album_id=4073973355617976325#wechat_redirect) 专栏内容源自笔者在实际学习和工作中对 LangChain 与 LangGraph 的深度使用经验，旨在帮助大家系统性地、高效地掌握 AI Agent 的开发方法，在各大技术平台获得了不少关注与支持。目前已更新43讲，正在更新LangChain 最新 DeepAgents框架的相关内容，并随时补充笔者在实际工作中总结的拓展知识点。如果大家感兴趣，欢迎关注笔者的微信公众号 大模型真好玩，每期分享涉及的代码均可在公众号私信: LangChain智能体开发免费获取。

2026年笔者的另一大重心还会放在 **[大模型训练](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk3NTA2OTMxNQ==&action=getalbum&album_id=4338985715301089302#wechat_redirect)** [专栏](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk3NTA2OTMxNQ==&action=getalbum&album_id=4331818996941979657#wechat_redirect) 分享上，当然，谈论训练无法绕过算力这道现实的门槛。昂贵的GPU是许多人探索技术的拦路石。正因如此笔者今年特意与一些可靠的算力平台展开了合作，希望能为大家解决算力瓶颈。大家可以扫描下方二维码注册 **Lab4ai算力平台** ，提供了包括英伟达H系列在内的多种选择，更有 **5小时的免费体验额度** 。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**微信扫一扫赞赏作者**

深入浅出LangChain AI Agent 智能体开发 · 目录

继续滑动看下一个

大模型真好玩

向上滑动看下一个
