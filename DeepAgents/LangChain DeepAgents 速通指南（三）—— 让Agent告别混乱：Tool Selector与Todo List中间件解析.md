---
title: "LangChain DeepAgents 速通指南（三）—— 让Agent告别混乱：Tool Selector与Todo List中间件解析"
source: "https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485820&idx=1&sn=5a1383f51019086be8078a2e464e7ef2&chksm=c4d00090f3a78986b0b878320dd6d2d85b97db14619e20f351708ce5cfd03f7cbd542f3026d1&scene=178&cur_album_id=4073973355617976325&search_click_id=#rd"
author:
  - "[[手摸手教你大模型]]"
published:
created: 2026-04-28
description: "本期介绍ToolSelector与TodoList中间件：ToolSelector智能筛选相关工具；TodoList自动拆解子任务并维护状态，二者共同提升LangChain DeepAgents处理复杂任务的性能。"
tags:
  - "clippings"
---
手摸手教你大模型 *2026年3月3日 08:01*

## 前言

上篇文章 [《LangChain DeepAgents 速通指南（二）—— Summarization中间件为Agent作记忆加减法》](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485784&idx=1&sn=86e84857c606ce6e76410d97e1027580&scene=21#wechat_redirect) 深入探讨了LangChain DeepAgents内置的 **Summarization中间件** 。该中间件能够自动压缩对话历史，有效解决大模型上下文窗口限制的问题。本期笔者将继续深入介绍LangChain DeepAgents框架预置的两个非常实用的中间件—— **Tool Selector** （工具选择器）和 **Todo List** （待办列表）。这两个中间件能够帮助Agent智能体在面对复杂任务和庞大工具集时，依然保持高效、专注与准确，让Agent的决策过程更加条理清晰。

PS:鉴于后台私信越来越多，我建了一些大模型交流群，大家在日常学习生活工作中遇到的大模型知识和问题都可以在群中分享出来大家一起解决！如果大家想交流大模型知识，可以关注我并回复加群

## 一、Tool Selector 中间件：为Agent的工具箱做减法

## 1.1 为什么需要Tool Selector？

在构建面向真实业务场景的智能体应用时，大家常常需要为大模型赋予大量的工具，例如搜索引擎、计算器、数据库查询器、API调用器等。当任务环境变得复杂，智能体可能拥有成百上千个工具。然而在执行一个具体任务时，实际用到的工具往往只是其中的一小部分。大量无关的工具不仅不会在每一步都发挥作用，反而会持续占据宝贵的模型上下文窗口。这不仅造成了Token的浪费，更可能引入噪声，干扰主模型的判断，降低决策的准确率。

Tool Selector正是为了解决这一痛点而生。它是一个覆写了 `wrap_model_call` 钩子函数的中间件。其核心机制是： **在每次调用主模型之前，ToolSelector中间件会基于当前的对话消息列表及用户问题，对全部工具列表进行一次智能预筛选，只保留与当前任务最相关的一小部分工具** 。随后，这个精简后的工具子集才会被传递给主模型，让主模型能够排除干扰，更专注、更准确地做出下一步的决策。

LangChain DeepAgents 框架通过内置的 `LLMToolSelectorMiddleware` 中间件实现了这一功能，其工作流程如下图所示：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

## 1.2 Tool Selector中间件适用场景

Tool Selector中间件尤其适合以下场景：

- **多工具管理**
	Agent拥有庞大的工具集，但每次查询仅涉及其中少数几个。
- **成本控制**
	希望通过过滤无关的工具信息，显著减少Token消耗，优化API调用成本。
- **精度提升**
	通过减少上下文中的冗余信息，提升模型在关键任务上的专注度与决策准确性。

## 1.3 如何使用Tool Selector？

在LangChain DeepAgents中使用Tool Selector非常简单，只需几个步骤即可完成配置。

**1\. 环境准备与依赖引入：** 首先在 `langchain>=1.0.5` 、 `python>=3.12` 的环境中编写脚本，引入必要的依赖，并定义模型和工具。本例中使用 DeepSeek 模型。

```javascript
from dotenv import load_dotenvfrom langchain_core.tools import toolfrom langchain.agents import create_agentfrom langchain.agents.middleware import LLMToolSelectorMiddlewarefrom langchain_deepseek import ChatDeepSeek
load_dotenv()
model = ChatDeepSeek(    model="deepseek-chat",)
```

**2\. 为 Agent 定义工具列表：** 编写一个核心的 `calculate` 工具，并同时定义 `tool_1` 至 `tool_4` 作为备用工具，以模拟一个拥有较多工具的场景。

```python
@tooldef tool_1(input:str) -> str:    """    This is a useless tool, intended solely as an example.    """    return "This is a useless tool, intended solely as an example."@tooldef tool_2(input:str) -> str:    """    This is a useless tool, intended solely as an example.    """    return "This is a useless tool, intended solely as an example."@tooldef tool_3(input:str) -> str:    """    This is a useless tool, intended solely as an example.    """    return "This is a useless tool, intended solely as an example."@tooldef tool_4(input:str) -> str:    """    This is a useless tool, intended solely as an example.    """    return "This is a useless tool, intended solely as an example."

@tooldef calculate(expression: str) -> str:    """Perform mathematical calculations and return the result.    Args:        expression: Mathematical expression to evaluate        (e.g., "2 + 3 * 4", "sqrt(16)", "sin(pi/2)")    Returns:        The calculated result as a string    """    result = str(eval(expression))    return result
```

3. **集成 LLMToolSelectorMiddleware 中间件：**
	使用Tool Selector功能需要引入 `LLMToolSelectorMiddleware` 中间件。配置该中间件需要以下几个关键参数：
- `model`
	负责执行工具筛选的模型实例，本例中与主模型共用同一个 `model` 。
- `max_tools`
	每次为主模型筛选出的最大工具数量。
- `always_include`
	一个列表，用于指定 **无论如何都必须被选中** 的工具名称。需要注意的是， `create_agent` 的 `tools` 参数传入的是工具对象列表，而 `always_include` 中传入的是这些工具对应的名称字符串。

```nginx
agent = create_agent(    model=model,    tools=[tool_1, tool_2, tool_3, tool_4,calculate],    middleware=[        LLMToolSelectorMiddleware(            model=model,            max_tools=2,            always_include=['tool_1'],        ),    ],)
```

**4\. 运行测试：** 最后通过一个数学计算任务来测试效果。从运行结果可以看到，尽管工具列表庞大，但智能体依然能够准确地选择并使用 `calculate` 工具来完成计算，这证明了Tool Selector在筛选工具方面的有效性。

```bash
status = {    'messages': '请计算2+3*4的值'}
result = agent.invoke(status)print(result)
```

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

## 二、Todo List 中间件：为复杂任务绘制路线图

## 2.1 为什么需要 Todo List 中间件？

当智能体需要执行一个多步骤、跨工具的复杂任务时，如果没有任务规划能力，Agent很容易在步骤中迷失，或者忘记已经完成的部分。大家工作中都使用Trae、Claude Code等编程智能体时也能观察到，当用户下达一个复杂任务时，这些智能体的首要工作往往是生成一份清晰的任务规划清单。

Todo List中间件的设计正是为了赋予Agent这种规划能力。它的实现方式与传统钩子函数不同，而是以 **额外工具** 的形式，向Agent注入一个名为 `write_todo` 的工具。其工作流程如下：

当用户输入一个复杂任务后，主模型会首先判断该任务是否需要多步处理。如果需要，它会主动调用 `write_todo` 工具，生成结构化的子任务列表，并将其保存在Agent的持久化状态（ `todos` 字段）中。随后，Agent会严格按照这个任务列表的顺序执行：当前正在处理的子任务状态标记为 `progress` ，完成后更新为 `completed` ，并自动推进到下一个状态为 `pending` 的子任务，如此循环，直至所有子任务完成。最终，Agent汇总所有步骤收集到的信息，给出完整的最终回复。这一机制确保了Agent能够有条不紊地完成复杂任务。整个流程可以用下图清晰地展示：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

## 2.2 Todo List 中间件适用场景

- **复杂多步骤任务**
	需要调用多个工具协同完成，且步骤之间存在明确的前后依赖关系。
- **需要进度可见性的长期运行任务**
	通过查看 `todos` 状态，开发者或用户能够实时了解任务执行的进展，知道当前进行到哪一步，哪些步骤已经完成。

## 2.3 如何使用 Todo List 中间件？

下面通过代码示例，详细讲解如何在LangChain的 `create_agent` API中集成并使用Todo List功能。

**1\. 环境准备与依赖引入：** 在 `langchain>=1.0.5` 、 `python>=3.12` 的环境中编写脚本，引入必要的依赖，并定义模型。本例中使用 DeepSeek 模型。

```javascript
from dotenv import load_dotenvfrom langchain.agents import create_agentfrom langchain.agents.middleware import TodoListMiddlewarefrom langchain_deepseek import ChatDeepSeek
load_dotenv()
model = ChatDeepSeek(    model="deepseek-chat",)
```

**2\. 集成 TodoListMiddleware 中间件：** 使用 `TodoListMiddleware` 中间件的方法极其简单，只需在创建Agent时，将其加入到 `middleware` 列表中即可， `write_todo` 工具会由中间件自动注入。

```makefile
agent = create_agent(    model=model,    tools=[],    middleware=[TodoListMiddleware()],)
```

**3\. 通过复杂案例测试：** 设计了一个高度复杂的、需要多步分析和计算的综合性问题来测试。当用户下达这个任务后，Todo List机制会自动生效。在Agent的最终响应中，除了常规的 `messages` （包含对话历史和最终答案），还会发现一个新增的 `todos` 字段。这个字段中包含了Agent对原始任务进行拆解后的详细子任务列表。每个子任务条目都包含 `content` （任务描述）和 `status` （状态）。

子任务的状态共有三种：

- `pending`
	尚未执行的子任务。
- `progress`
	当前正在执行的子任务。
- `completed`
	已经完成的子任务。

**注意** ：由于在这里使用的是 `invoke` 方法进行同步调用，Agent会执行完所有任务后才返回最终结果，因此最终打印出的所有子任务状态均为 `completed` 。若想观察子任务状态在执行过程中的动态变化（例如从 `pending` 到 `progress` 再到 `completed` 的流转），可以参考笔者的另一篇文章 [《LangChain1.0速通指南（二）——LangChain1.0 create\_agent api 基础知识》](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485146&idx=1&sn=1397839d622039d1665868f97a329f43&scene=21#wechat_redirect) 中介绍的流式输出方法

```python
res = agent.invoke({"messages":["""你要一步一步的详细规划以下内容再进行回答。请分析美国加利福尼亚中央谷地的杏仁种植业在未来30年面临的气候变化风险,并估算其经济影响。具体需要回答,:“假设当前气候趋势持续,到2050年,加利福尼亚中央谷地杏仁产量可能减少的百分比及其对该州经济的潜在年度损失是多少美元?这些美元按照2025年11月的汇率能够购买多少比特币?"""]})
print(res["messages"])print("\n---------TODO---------------\n")print(res["todos"])
```

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

以上就是笔者今天的全部内容。完整的源码示例已整理好，欢迎大家关注笔者的微信公众号 **大模型真好玩** ，在公众号后台私信发送 **LangChain智能体开发** 即可免费获取。

## 三、总结

本期内容介绍了ToolSelector和TodoList两个中间件，ToolSelector中间件能够在调用主模型前，利用LLM智能筛选相关工具。TodoList中间件通过注入 `write_todo` 工具，让Agent在遇到复杂任务时自动拆解子任务并维护状态。LangChain DeepAgents框架正是通过这两个中间件提升了智能体执行复杂任务的性能。除了这两个中间件外，DeepAgents框架还需要有一个文件系统来存储中间结果、记忆结果等，下期内容笔者会分享LangChain DeepAgents框架 FileSystem的相关知识，大家敬请期待~

[《深入浅出LangChain&LangGraph AI Agent 智能体开发》](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk3NTA2OTMxNQ==&action=getalbum&album_id=4073973355617976325#wechat_redirect) 专栏内容源自笔者在实际学习和工作中对 LangChain 与 LangGraph 的深度使用经验，旨在帮助大家系统性地、高效地掌握 AI Agent 的开发方法，在各大技术平台获得了不少关注与支持。目前已更新40讲，正在更新LangGraph1.0速通指南，并随时补充笔者在实际工作中总结的拓展知识点。如果大家感兴趣，欢迎关注笔者的微信公众号 大模型真好玩，每期分享涉及的代码均可在公众号私信: LangChain智能体开发免费获取。

2026年笔者的另一大重心还会放在 **[大模型训练](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk3NTA2OTMxNQ==&action=getalbum&album_id=4338985715301089302#wechat_redirect)** [专栏](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk3NTA2OTMxNQ==&action=getalbum&album_id=4331818996941979657#wechat_redirect) 分享上，当然，谈论训练无法绕过算力这道现实的门槛。昂贵的GPU是许多人探索技术的拦路石。正因如此笔者今年特意与一些可靠的算力平台展开了合作，希望能为大家解决算力瓶颈。大家可以扫描下方二维码注册 **Lab4ai算力平台** ，提供了包括英伟达H系列在内的多种选择，更有 **5小时的免费体验额度** 。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**微信扫一扫赞赏作者**

深入浅出LangChain AI Agent 智能体开发 · 目录

继续滑动看下一个

大模型真好玩

向上滑动看下一个