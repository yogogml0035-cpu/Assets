---
title: "LangChain DeepAgents 速通指南（二）—— Summarization中间件为Agent作记忆加减法"
source: "https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485784&idx=1&sn=86e84857c606ce6e76410d97e1027580&chksm=c4d000b4f3a789a2b5cb08ac02a5ce2aeb634f1ad1dba2f059567e291ea85ddf1543d4937bff&scene=178&cur_album_id=4073973355617976325&search_click_id=#rd"
author:
  - "[[手摸手教你大模型]]"
published: 2025-12-01
created: 2026-04-28
description: "本文深入讲解LangChain DeepAgents内置的Summarization中间件，它能自动压缩对话历史，解决大模型上下文窗口限制问题。通过配置trigger、keep、model等参数，中间件在达到阈值时将旧消息摘要并重组列表，为Agent记忆“做减法”，助力高效处理长任务。"
tags:
  - "clippings"
---
手摸手教你大模型 *2026年2月25日 16:49*

## 前言

上篇分享 [LangChain DeepAgents 速通指南（一）—— 一文详解DeepAgents核心特性](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485774&idx=1&sn=8012973f5cdb8ca0b6ea2448331e1fca&scene=21#wechat_redirect) 笔者介绍了DeepAgents的强大特性。借助 `create_deep_agent` 函数，开发者仅需少量代码即可搭建出功能复杂的智能体，真正实现了“搭积木式”的开发体验。而 `create_deep_agent` 之所以如此强大，其核心原因在于 LangChain DeepAgents 团队在 LangChain 1.0 的 `create_agent` API 基础上，内置了丰富的中间件和工具函数。从本期开始笔者将逐一剖析这些经典组件，帮助大家更深入地理解它们的设计与用途。

本期内容将聚焦 **Summarization（摘要中间件）** 。在实际开发中，由于大模型上下文窗口的限制，一个常见且棘手的问题逐渐浮现： **随着任务推进，对话历史的不断累积会迅速耗尽上下文窗口，导致信息丢失甚至任务中断** 。

当 Agent 需要处理长篇文档、进行多轮交互，或从冗余信息（如网页内容）中提取关键数据时，消息列表很容易突破模型的 token 限制。尤其是在 DeepAgents 这类复杂智能体中，中间消息和上下文信息更是急剧增加，进一步加剧了上下文窗口的压力。为解决这一痛点，LangChain 1.0 提供了一个非常实用的中间件—— **Summarization** （摘要中间件），它也被 DeepAgents 默认集成到智能体构建流程中。今天笔者就和大家一起来深入探讨它的使用方法与实现原理，看看它是如何为 Agent 的记忆“做减法”的。

PS:鉴于后台私信越来越多，我建了一些大模型交流群，大家在日常学习生活工作中遇到的大模型知识和问题都可以在群中分享出来大家一起解决！如果大家想交流大模型知识，可以关注我并回复加群

## 一、Summarization中间件核心原理

## 1.1 为什么需要Summarization？

在基于大模型与工具构建的Agent中，每次调用模型都需要将之前的对话历史一并传入，以便模型理解当前上下文。然而，模型的上下文窗口是有限的（例如常见的 8K、32K token 限制）。当任务变得复杂，或工具返回的信息量很大时，消息列表很容易就会“爆表”，超出模型的处理能力。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

Summarization中间件的作用，正是 **在接近Token限制或其他预设条件时，自动对较旧的对话记录进行摘要** 。它会在保留近期关键消息的同时，将早期内容压缩成一段精炼的总结。这样一来，Agent既能维持对任务背景的连贯理解，又不会因Token溢出而导致执行中断。

## 1.2 适用场景

- **长文本处理**
	当模型需要阅读并分析长文档、多页网页时，工具返回的内容可能一次就占满上下文。Summarization可以在每次工具调用前对已积累的内容做摘要，防止溢出。
- **多轮次对话**
	在客服、咨询等场景中，对话轮次可能多达几十轮。Summarization可以将早期的寒暄或确认信息压缩，让模型聚焦当前核心问题。
- **高冗余工具调用**
	某些工具（如搜索引擎、爬虫）返回的信息往往包含大量无关内容（广告、导航栏等）。Summarization可以提炼关键信息，减少噪声。

## 1.3 Summarization中间件原理

Summarization中间件继承自 `AgentMiddleware` 基类，通过覆写 `before_model` 与 `abefore_model` 方法，在信息进入模型前执行压缩处理。对中间件机制不太熟悉的读者，可以参考我之前的文章： [LangChain1.0速通指南（三）——LangChain1.0 create\_agent api 高阶功能](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485162&idx=1&sn=570bdb6fc1f6a88971e9b787d3fa35d8&scene=21#wechat_redirect) 和 [LangChain不支持AgentSkills？那就从0到1实现一个！](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485709&idx=1&sn=bf8f44e745a15d9de8a8af8a59ccfc88&scene=21#wechat_redirect) 。

Summarization属于 **before model钩子** 类型的中间件，意味着它会在 **每一次模型调用前被触发执行** 。它的核心工作流程如下：

1. **检查消息列表**
	获取当前Agent的完整消息列表（包括历史消息、用户最新输入、工具响应等）。
2. **判断触发条件**
	根据用户配置的 `trigger` 参数（例如消息数量超过阈值、Token总数超限、达到模型上下文窗口的一定比例等），判断当前是否需要进行摘要。
3. **执行摘要**
	如果条件满足，中间件会先根据 `keep` 参数保留最新的一定数量的消息（例如保留最近的3条），然后将剩余的历史消息一并发送给摘要模型，生成一段概括性的总结。
4. **重组消息列表**
	用新生成的摘要消息（通常包装为一条 `HumanMessage` ）与之前保留的最近消息组合，形成一个新的、更精简的消息列表，再传递给模型进行后续的处理。
![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

## 二、快速上手：配置你的第一个Summarization中间件

下面笔者通过实际的代码示例，演示如何在 LangChain 1.0 中使用 Summarization 中间件。

**1\. 环境准备与依赖引入** ：在 `langchain>=1.0.5` 、 `python>=3.12` 的环境中编写以下脚本，引入必要的依赖，并定义模型和工具。本例中使用 DeepSeek 模型

```python
import os
from typing import Literal

from dotenv import load_dotenv
from langchain.agents import create_agent
from langchain.agents.middleware import SummarizationMiddleware
from langchain.messages import AIMessage, HumanMessage, ToolMessage
from langchain_core.tools import tool
from langchain_deepseek import ChatDeepSeek
from tavily import TavilyClient

load_dotenv()
tavily_client = TavilyClient(api_key=os.environ["TAVILY_API_KEY"])

@tool
def internet_search(
    query: str,
    max_results: int = 5,
    topic: Literal["general", "news", "finance"] = "general",
    include_raw_content: bool = False,
):
    """Search the internet for information using Tavily search engine."""
    search_docs = tavily_client.search(
        query,
        max_results=max_results,
        include_raw_content=include_raw_content,
        topic=topic,
    )
    return search_docs

@tool
def calculate(expression: str) -> str:
    """Perform mathematical calculations and return the result."""
    result = str(eval(expression))
    return result

model = ChatDeepSeek(model="deepseek-chat")
```

**2\. 为 Agent 配置 Summarization 中间件** ： 与其他中间件类似，使用 Summarization 中间件需要先将其引入，并添加到 Agent 的中间件列表中。配置后，中间件会按照设定的规则，在任务执行过程中自动精简消息列表。在下面的代码中笔者为 `SummarizationMiddleware` 传入了三个关键配置

- `model：指定用于生成摘要的模型。摘要的质量直接取决于此模型的能力。`
- `trigger：设置触发摘要的条件。例如 ("messages", 5)表示当消息列表中的消息数量大于 5 条时，触发摘要。`
- `keep：设置摘要后需要保留的最新消息数量。例如 ("messages", 3) 表示摘要完成后，只保留最近的 3 条原始消息，其余压缩成一条摘要。`

```python
agent = create_agent(
    model=model,
    tools=[internet_search, calculate],
    middleware=[
        SummarizationMiddleware(
            model=model,
            trigger=("messages", 5),
            keep=("messages", 3),
        ),
    ],
)
```

**3\. 测试调用与效果验证** ：笔者构造一个包含四条历史消息的初始状态，然后追加一条新的用户询问，最后调用 Agent 进行处理。从返回的结果可以看到，Agent 的回复中包含了五条消息，其中第一条 `HumanMessage` 正是 Summarization 中间件对历史信息进行总结后生成的摘要内容

```python
status = {
    "messages": [
        HumanMessage(content="deepseek公司最近有什么最新的资讯?"),
        AIMessage(
            content="",
            tool_calls=[
                {
                    "name": "internet_search",
                    "args": {"query": "deepseek 最新资讯", "topic": "news"},
                    "id": "call_search_001",
                    "type": "tool_call",
                }
            ],
        ),
        ToolMessage(
            content="DeepSeek 发布了新的数学推理模型 DeepSeek-Math-V2。",
            name="internet_search",
            tool_call_id="call_search_001",
        ),
        AIMessage(content="已根据搜索结果整理出 DeepSeek 最新资讯。"),
    ]
}
status["messages"].append(
    HumanMessage("deepseek的新模型有哪些特点与突破?")
)
result = agent.invoke(status)
print(result)
```

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**4\. 内部流程解析** ：详细拆解一下在上述调用过程中，Agent 内部的消息流转与 Summarization 中间件的工作细节：

- **初始状态：用户调用 Agent 时，消息列表的初始状态包含 4 条历史消息和 1 条新询问，共计 5 条消息。此时，消息数量刚好达到我们设置的 trigger阈值（5 条），但并未超过。**
- **第一次模型调用：模型接收这 5 条消息后，判断需要调用工具（internet\_search）。**
- **工具返回结果：工具执行完毕并将结果（ToolMessage）返回后，消息列表中新增了这条工具消息。此时，列表中的消息总数变为 7 条，已超过 trigger 阈值。**
- **第二次模型调用前（触发摘要）：在将工具返回的结果和之前的消息再次送入模型进行最终回复前，Summarization 中间件会检查消息列表。发现 7 > 5，触发摘要条件。**
- **执行摘要：中间件根据我们配置的 keep=3，首先保留最新的 3 条消息（即工具返回的消息、引发工具调用的 AI 消息，以及用户的最新提问？）。然后，将剩余的所有更早的消息发送给摘要模型，生成一段总结。**
- **重组消息列表：这段新生成的总结被包装为一条 HumanMessage（即我们在最终输出中看到的第一条消息），并与之前保留的 3 条最新消息组合，形成一个新的、共 4 条消息的列表，传递给模型。**
- **最终回复：模型基于这个精简后的消息列表，生成对用户提问的最终回答。**
![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

## 三、 Summarization配置详解

Summarization中间件提供了四个核心配置参数： `trigger` 、 `keep` 、 `model` 以及可选的 `summary_prompt` 。下面逐一详解其用法与注意事项。

## 3.1 trigger：触发条件

`trigger` 参数决定了何时对消息列表进行摘要。它可以是一个 **单一条件元组** ，也可以是一个由多个条件元组组成的 **列表** （采用“或”逻辑，满足任一条件即触发）。LangChain内置了以下几种条件类型：

- **基于消息数量**
	例如 `("messages", ">=", 10)` 表示当消息总数达到或超过10条时触发。
- **基于Token数量**
	例如 `("tokens", ">=", 3000)` 表示当所有消息的总Token数达到或超过3000时触发（LangChain会自动对消息进行编码以计算Token数）。
- **基于上下文窗口占比**
	例如 `("fraction", ">=", 0.7)` 表示当前消息的总Token数占模型最大上下文Token数的比例达到或超过70%时触发。

除了单一条件，还可以组合多个条件以实现更灵活的触发策略。当使用列表时，条件之间为“或”关系，即任意一个条件满足就会触发摘要。示例如下：

```ini
trigger = [
    ("messages", ">=", 8),
    ("fraction", ">=", 0.8),
]
# 当消息数量≥8 **或** 上下文占用≥80%时触发
```

## 3.2 keep：保留最新消息的数量

`keep` 参数指定在触发摘要后，需要保留多少条 **最新的原始消息** （这些消息不会被摘要）。与 `trigger` 不同， `keep` 只能使用一个 **单一条件元组** ，不支持列表，因为保留策略必须具有确定性。

常见的配置方式包括：

- **基于消息数量**
	`keep = ("messages", "=", 5)` 表示保留最新的5条消息。
- **基于Token数量**
	`keep = ("tokens", "<=", 1000)` 表示保留最新的消息，直到其总Token数不超过1000为止。
- **基于上下文占比**
	`keep = ("fraction", "<=", 0.3)` 表示保留最新的消息，直到其占用模型上下文的比例不超过30%为止。

通常情况下，基于消息数量来配置 `keep` 最为直观和常用。

**注意** ： `keep` 不能像 `trigger` 那样配置多个条件，因为保留策略必须是单一且明确的。

## 3.3 model：摘要模型

`model` 参数用于指定生成摘要所使用的语言模型。它可以是一个模型名称字符串（例如 `"gpt-3.5-turbo"` ），也可以是一个已经初始化的模型实例。在实际应用中，通常建议选用轻量、快速的模型来执行摘要任务，以避免占用Agent主模型的计算资源，从而提升整体效率。

## 3.4 summary\_prompt：自定义摘要提示词

LangChain为摘要模型预设了一个默认的系统提示词，其内容大致为“请总结以下对话历史，保留关键信息”。如果你希望摘要以特定格式输出，或需要加入额外的指令，可以通过 `summary_prompt` 参数传入自定义的提示词模板。在自定义提示词时， **必须包含 `{messages}` 占位符** ，以便中间件将待总结的消息列表传入模板。示例如下：

```python
from langchain.middleware.summarization import DEFAULT_SUMMARY_PROMPT
from langchain.prompts import PromptTemplate

custom_prompt = PromptTemplate.from_template(
    "请将以下对话浓缩成一段简洁的总结，重点关注事实和行动项：\n\n{messages}"
)
summarization_mw = SummarizationMiddleware(
    model=summary_llm,
    trigger=trigger,
    keep=keep,
    summary_prompt=custom_prompt,
)
```

以上就是笔者今天的全部内容。完整的源码示例已整理好，欢迎大家关注笔者的微信公众号 **大模型真好玩** ，在公众号后台私信发送 **LangChain智能体开发** 即可免费获取。

## 四、总结

本文深入讲解LangChain DeepAgents内置的Summarization中间件，它能自动压缩对话历史，解决大模型上下文窗口限制问题。通过配置trigger（触发条件）、keep（保留最新消息）、model（摘要模型）等参数，中间件在达到阈值时将旧消息摘要并重组列表。支持消息数、token数、占比等多条件触发，可自定义摘要提示词，为Agent记忆“做减法”，助力高效处理长任务。除了Summarization中间件，DeepAgents还内置了其它许多重要的中间件，下期内容笔者将分享任务规划中间件与工具，大家敬请期待~

[《深入浅出LangChain&LangGraph AI Agent 智能体开发》](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk3NTA2OTMxNQ==&action=getalbum&album_id=4073973355617976325#wechat_redirect) 专栏内容源自笔者在实际学习和工作中对 LangChain 与 LangGraph 的深度使用经验，旨在帮助大家系统性地、高效地掌握 AI Agent 的开发方法，在各大技术平台获得了不少关注与支持。目前已更新40讲，正在更新LangGraph1.0速通指南，并随时补充笔者在实际工作中总结的拓展知识点。如果大家感兴趣，欢迎关注笔者的微信公众号 大模型真好玩，每期分享涉及的代码均可在公众号私信: LangChain智能体开发免费获取。

2026年笔者的另一大重心还会放在 **[大模型训练](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk3NTA2OTMxNQ==&action=getalbum&album_id=4338985715301089302#wechat_redirect)** [专栏](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk3NTA2OTMxNQ==&action=getalbum&album_id=4331818996941979657#wechat_redirect) 分享上，当然，谈论训练无法绕过算力这道现实的门槛。昂贵的GPU是许多人探索技术的拦路石。正因如此笔者今年特意与一些可靠的算力平台展开了合作，希望能为大家解决算力瓶颈。大家可以扫描下方二维码注册 **Lab4ai算力平台** ，提供了包括英伟达H系列在内的多种选择，更有 **5小时的免费体验额度** 。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

深入浅出LangChain AI Agent 智能体开发 · 目录

继续滑动看下一个

大模型真好玩

向上滑动看下一个
