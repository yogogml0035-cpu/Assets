---
title: "LangChain DeepAgents 速通指南（一）—— 一文详解DeepAgents核心特性"
source: "https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485774&idx=1&sn=8012973f5cdb8ca0b6ea2448331e1fca&chksm=c4d000a2f3a789b4677c8a1b83ecb410dd97387fa383e4fde9883d60f1ab748d30377a2be53c&scene=178&cur_album_id=4073973355617976325&search_click_id=#rd"
author:
  - "[[手摸手教你大模型]]"
published:
created: 2026-04-28
description: "DeepAgents是LangChain团队开发的框架，封装任务规划、子代理管理、文件系统等通用能力，通过create_deep_agent函数让开发者仅需数行代码即可构建复杂智能体，实现搭积木式开发."
tags:
  - "clippings"
---
手摸手教你大模型 *2026年2月24日 17:30*

## 前言

上篇文章 [《LangChain不支持AgentSkills？那就从0到1实现一个》](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485709&idx=1&sn=bf8f44e745a15d9de8a8af8a59ccfc88&scene=21#wechat_redirect) 发布后，不少读者在评论区提到，deepagents已经集成了Skill机制。通过查阅官方文档，我发现实现这一机制的是LangChain团队推出的DeepAgents框架。尽管最终调研发现DeepAgents CLI中的Skill功能未能完全满足笔者工作中的实际需求，但我对DeepAgents框架本身产生了浓厚的兴趣。经过一段时间的学习与实践，我深深折服于其精妙的设计理念。因此，在完成LangChain/LangGraph 1.0系列分享后，我计划接下来系统性地介绍DeepAgents相关内容。本期分享将首先带大家了解DeepAgents是什么，以及它的设计初衷。

本系列相关内容均列于笔者的专栏 [《深入浅出LangChain&LangGraph AI Agent 智能体开发》](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk3NTA2OTMxNQ==&action=getalbum&album_id=4073973355617976325#wechat_redirect) ，该专栏适合所有对 LangChain 感兴趣的学习者，无论之前是否接触过 LangChain。该专栏基于笔者在实际项目中的深度使用经验，系统讲解了使用LangChain/LangGraph如何开发智能体，目前已更新 39 讲，并持续补充实战与拓展内容。欢迎感兴趣的同学关注笔者的微信公众号 **大模型真好玩** ，每期分享涉及的代码均可在公众号私信: **LangChain智能体开发** 免费获取。

PS:鉴于后台私信越来越多，我建了一些大模型交流群，大家在日常学习生活工作中遇到的大模型知识和问题都可以在群中分享出来大家一起解决！如果大家想交流大模型知识，可以关注我并回复加群

## 一、DeepAgents：让复杂智能体的开发像搭积木一样简单

## 1.1 DeepAgents框架开发溯源

大家平时接触最多的智能体，通常是这样工作的：让大语言模型在一个循环里反复推理、调用工具，直到完成任务。这种模式在简单场景下很管用，可一旦任务变得复杂就会出现很大问题， 例如步骤不合理、工具调用出错、对话上下文越积越多导致混乱，甚至智能体自己都忘了要干什么。

在LangChain生态中要解决这个问题通常需要大家使用LangGraph从零搭建工作流，精确控制每一步是让 AI 自己决策还是走固定流程，也能选择不同的思考模式（比如 ReAct 或“先计划后执行”）。但这种灵活性的代价就是代码复杂，上手门槛高（关于LangGraph1.0的速通指南可参考笔者文章 [LangGraph1.0速通指南（一）—— LangGraph1.0 核心概念、点、边）](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485396&idx=1&sn=496d97ab67a213da3ea687dafc8d07e6&scene=21#wechat_redirect) 。

作为全世界最懂智能体开发的LangChain团队也敏锐的注意到这个问题，开发了框架 **DeepAgents** （截止目前版本为0.4.3）。 它的核心思想很简单： **把那些让智能体变得“聪明”的通用能力——比如做计划、记笔记、分工协作——都打包好，让大家开箱即用。** 这样大家只需要关注自己的业务逻辑，甚至只需要几行代码，就能快速搭建出能处理复杂、多步骤、长时间任务的智能体。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

## 1.2 DeepAgents框架定位

要理解 DeepAgents，先要理清它和 LangChain、LangGraph 的关系。打个比方：

- **LangGraph：就像智能体的“操作系统内核”，负责底层的工作流调度、状态持久化和监控。**
- **LangChain：基于LangGraph内核的“高级开发工具包”，提供了很多现成的函数（比如 create\_agent）和组件，让大家写代码更顺手。**
- **DeepAgents：在 LangChain 的便利接口和 LangGraph 的强大运行时之上，又盖了一层“智能模块”——比如任务规划、文件系统、长期记忆、多智能体协作等。它的核心函数 create\_deep\_agent，其实就是给 LangChain 的标准智能体加上了这些模块。**
![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

理解了DeepAgents/LangChain/LangGraph三者的分层定位，笔者建议大家根据实际需求情况选择合适框架开发：

- 对于一些 **简单步骤的任务** ，直接使用LangChain 1.0提供的 `create_agent` 。
- 对于一些需要精细控制的 **定制化需求** ，使用LangGraph 1.0自定义工作流程实现。
- 对于一些需要构建复杂的多步骤、长时间运行的Agent；并希望拥有任务规划、文件系统、长期记忆等能力时，考虑DeepAgents。

## 1.3 DeepAgents核心原理

DeepAgents的核心原理其实非常直观。大家可以借助大家熟悉的复杂任务智能体（如Claude Code等编程智能体）来理解：它们通常都具有 **自主制定计划** 、 **保存中间结果** ，并能够 **调用子智能体协助完成任务** 等共性能力。DeepAgents正是将这些共性能力进行了抽象和封装，形成了一系列 **预置组件** 。这些组件以 **中间件** 和 **内置工具** 的形式实现，提供了任务规划、上下文管理、子代理生成和长期记忆等核心功能。此外，DeepAgents还专门设计了文件系统，用于高效管理大量的中间结果与上下文信息。以上所有功能都已集成在 `create_deep_agent` SDK 中，开发者可以快速调用，轻松构建功能强大的复杂智能体。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

## 二、 DeepAgents 代码示例讲解

理论的讲解总归还是有点虚，为了让大家更直观地体验DeepAgents的优点，笔者这里选用DeepAgents官方的一个 `deep_research` 案例来感受DeepAgents的强大功能——仅需数十行代码，即可完成DeepResearch智能体的相关构建。

## 2.1 代码概览

先整体看一下这段代码。它创建了一个能够进行深度研究的智能体（Research Agent），该智能体不仅自己能使用搜索和思考工具，还可以动态委派子任务给专门的研究子智能体（sub‑agent）。整个脚本结构清晰，核心逻辑都封装在 `create_deep_agent` 函数中。

```python
from datetime import datetime

from deepagents import create_deep_agent
from langchain.chat_models import init_chat_model
from langchain_google_genai import ChatGoogleGenerativeAI

from research_agent.prompts import (
    RESEARCHER_INSTRUCTIONS,
    RESEARCH_WORKFLOW_INSTRUCTIONS,
    SUBAGENT_DELEGATION_INSTRUCTIONS,
)
from research_agent.tools import tavily_search, think_tool

# 并发与迭代限制
max_concurrent_research_units = 3
max_researcher_iterations = 3

# 当前日期（用于提示词中的时间信息）
current_date = datetime.now().strftime("%Y-%m-%d")

# 组合主智能体的系统提示词
INSTRUCTIONS = (
    RESEARCH_WORKFLOW_INSTRUCTIONS
    + "\n\n"
    + "=" * 80
    + "\n\n"
    + SUBAGENT_DELEGATION_INSTRUCTIONS.format(
        max_concurrent_research_units=max_concurrent_research_units,
        max_researcher_iterations=max_researcher_iterations,
    )
)

# 定义研究子代理
research_sub_agent = {
    "name": "research-agent",
    "description": "Delegate research to the sub-agent researcher. Only give this researcher one topic at a time.",
    "system_prompt": RESEARCHER_INSTRUCTIONS.format(date=current_date),
    "tools": [tavily_search, think_tool],
}

# 选择底层大模型（此处使用 Claude 4.5，Gemini 3 备选）
# model = ChatGoogleGenerativeAI(model="gemini-3-pro-preview", temperature=0.0)
model = init_chat_model(model="anthropic:claude-sonnet-4-5-20250929", temperature=0.0)

# 创建深度智能体
agent = create_deep_agent(
    model=model,
    tools=[tavily_search, think_tool],
    system_prompt=INSTRUCTIONS,
    subagents=[research_sub_agent],
)
```

## 2.2 核心组件解析

上述代码虽短，却完整地构建了一个具备 **任务规划** 、 **子代理委派** 和 **内置思考能力** 的复杂研究智能体。下面笔者将逐一拆解其关键部分。

### 2.2.1 create\_deep\_agent：一切智能体的起点

`create_deep_agent` 是DeepAgents框架提供的核心工厂函数。它接受一个基础模型、工具列表、系统提示词和子智能体列表，返回一个 **开箱即用的深度智能体** 。这个智能体内部已经集成了：

- **任务规划器**
	将复杂任务拆解为可执行的步骤。
- **文件系统**
	管理中间结果和上下文，防止对话过长导致混乱。
- **子智能体管理器**
	负责子智能体的创建、通信和结果汇总。
- **长期记忆**
	跨对话保存重要信息。

开发者完全不需要关心这些底层逻辑的实现，只需像搭积木一样传入配置即可。

### 2.2.2 系统提示词：赋予智能体“灵魂”

提示词的设计是智能体行为的关键。主智能体的系统提示词由三部分拼接而成：

- `RESEARCH_WORKFLOW_INSTRUCTIONS`
	定义研究工作的整体流程，例如如何规划、如何组织搜索结果。
- `SUBAGENT_DELEGATION_INSTRUCTIONS`
	规定了子智能体的委派规则，其中动态插入了并发限制和最大迭代次数。这让子智能体知道它可以同时运行最多3个研究单元，每个研究任务最多迭代3次。

子智能体的提示词 `RESEARCHER_INSTRUCTIONS` 还嵌入了当前日期，确保研究过程中能够感知时间上下文（例如搜索“今年的诺贝尔奖得主”时能正确理解“今年”）。

### 2.2.3 子智能体：分工协作的体现

`research_sub_agent` 是一个典型的子智能体定义。它包含：

- `name`
	唯一标识，用于在主智能体中调用。
- `description`
	描述子代理的功能， **主智能体通过阅读理解这段描述来决定何时委派任务** 。
- `system_prompt`
	子智能体自身的系统提示词，专注于研究任务。
- `tools`
	子智能体可用的工具，此处与主智能体一样拥有搜索和思考能力。

这种设计完美体现了第一章提到的“多智能体协作”原理：主智能体负责全局规划与调度，子代理专注于单一研究主题，各自分工，最终汇总成果。当然，除了通过传入配置方式构建子智能体外，大家也可将使用LangChain `create_agent` 创建的智能体或使用LangGraph `compile` 后的智能体作为子智能体传入。

### 2.2.4 工具：扩展智能体的能力边界

本例中使用了两个工具：

- `tavily_search`
	一个专为大模型优化的搜索引擎工具，能够返回干净、结构化的搜索结果。
- `think_tool`
	一个思考工具，在每次搜索后使用此工具，以系统性地分析结果并规划后续步骤。这能在研究工作流中创建刻意暂停，以便进行高质量的决策。

DeepAgents框架允许开发者自由注册任何自定义工具，进一步扩展智能体的能力。

## 2.3 DeepAgents设计思想与原理

从这段代码可以很好的体现DeepAgents三大设计原则： **封装通用能力** 、 **简化开发** 、 **模块化组合** 。

- **封装通用能力**
	任务规划、子代理管理、文件系统等复杂逻辑全部隐藏在 `create_deep_agent` 内部，开发者无需编写任何LangGraph节点和边的代码。
- **简化开发**
	原本需要数百行LangGraph代码才能实现的深度研究智能体，现在只需几十行配置即可完成。开发者只需关注提示词工程和工具定义。
- **模块化组合**
	主智能体、子智能体、工具都是独立模块，可以像搭积木一样自由组合、复用。大家可以为其他领域（如数据分析、代码生成）定义不同的子代理，轻松扩展智能体的能力。

笔者通过LangChain的 `agent.get_graph().draw_mermaid_png()` 展示DeepAgents构造的deepresearch智能体的图结构，可以看到该图同样是ReACT的经典结构，并通过 `PathToolCallsMiddleware`, `SummarizationMiddleware` 等中间件扩展了LangChain create\_agent的能力。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

## 2.4 运行效果与扩展思考

下面通过一个具体示例展示DeepAgents的运行效果。假设用户提出如下请求： `research context engineering approaches used to build AI agents（研究用于构建AI代理的上下文工程方法）`

```python
result = agent.invoke(
    {
        "messages": [
            {
                "role": "user",
                "content": "research context engineering approaches used to build AI agents",
            }
        ],
    }
)
format_messages(result["messages"])
```

**1\. 任务规划** ： DeepAgents首先接收用户的研究请求，并调用内置的 `write_todos` 工具生成任务计划。每个任务项都带有状态标记，例如第一步是“设计研究请求”，并将该计划保存至 `research_request.md` 文件中。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**2\. 逐步执行与状态更新** ：智能体每完成一个步骤，都会再次调用 `write_todos` 工具更新对应任务的状态为 `completed` ，然后自动进入下一步。如图所示，下一步计划是调用一个子智能体来获取具体的研究内容

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**3\. 结果汇总与输出** ：当所有计划任务完成后，智能体将更新最终的待办列表，并使用 `write_file` 工具将整理好的研究报告写入 `final_report.md` 文件中

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E) ![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

上述示例中涉及的 `write_todos` 、 `task` 、 `write_file` 等工具均已内置在DeepAgents中。用户只需输入简单的需求，智能体便会自动调用相应工具和文件系统来完成整个任务。由此可见，DeepAgents真正实现了“让复杂智能体的开发像搭积木一样简单”。

以上就是本篇文章的全部内容，相关示例代码大家可关注笔者的微信公众号 **大模型真好玩** ，并在公众号私信: **LangChain智能体开发** 获取。

## 三、总结

DeepAgents是LangChain团队推出的智能体框架，它将任务规划、子代理管理、文件系统等通用能力封装为内置组件，开发者通过 `create_deep_agent` 函数仅需数行代码即可搭建复杂智能体，真正实现“搭积木式”开发。从上述分析可以看出，DeepAgents本质上是在LangChain `create_agent` 的基础上，通过中间件和内置工具增强了智能体的能力。为了帮助大家深入理解DeepAgents的底层机制，做到知其然更知其所以然，下一期分享笔者将暂不探讨使用技巧，而是从DeepAgents所依赖的中间件入手，全面解析其设计原理与实现细节，敬请期待！

[《深入浅出LangChain&LangGraph AI Agent 智能体开发》](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk3NTA2OTMxNQ==&action=getalbum&album_id=4073973355617976325#wechat_redirect) 专栏内容源自笔者在实际学习和工作中对 LangChain 与 LangGraph 的深度使用经验，旨在帮助大家系统性地、高效地掌握 AI Agent 的开发方法，在各大技术平台获得了不少关注与支持。目前已更新39讲，正在更新LangGraph1.0速通指南，并随时补充笔者在实际工作中总结的拓展知识点。如果大家感兴趣，欢迎关注笔者的微信公众号 大模型真好玩，每期分享涉及的代码均可在公众号私信: LangChain智能体开发免费获取。

2026年笔者的另一大重心还会放在 **[大模型训练](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk3NTA2OTMxNQ==&action=getalbum&album_id=4338985715301089302#wechat_redirect)** [专栏](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk3NTA2OTMxNQ==&action=getalbum&album_id=4331818996941979657#wechat_redirect) 分享上，当然，谈论训练无法绕过算力这道现实的门槛。昂贵的GPU是许多人探索技术的拦路石。正因如此笔者今年特意与一些可靠的算力平台展开了合作，希望能为大家解决算力瓶颈。大家可以扫描下方二维码注册 **Lab4ai算力平台** ，提供了包括英伟达H系列在内的多种选择，更有 **5小时的免费体验额度** 。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

深入浅出LangChain AI Agent 智能体开发 · 目录

继续滑动看下一个

大模型真好玩

向上滑动看下一个
