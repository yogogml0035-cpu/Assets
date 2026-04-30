---
title: "LangChain DeepAgents 速通指南（五）—— 快速了解DeepAgents框架及其核心特性"
source: "https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485906&idx=1&sn=1024589caa410a2b36aff4f320eba259&chksm=c4d0003ef3a78928f868f456357204f80bd070ac35c282fa87499442c929edb184622c88135c&scene=178&cur_album_id=4073973355617976325&search_click_id=#rd"
author:
  - "[[手摸手教你大模型]]"
published:
created: 2026-04-28
description: "本文分享了DeepAgents 框架的诞生背景与核心优势，并通过实战案例演示了其快速上手的完整流程。可以看到，DeepAgents 在任务规划、上下文管理、子代理并行执行等方面展现出强大的能力，仅需少量代码即可构建复杂智能体，让一键创建Claude Code成为可能。"
tags:
  - "clippings"
---
手摸手教你大模型 *2026年3月25日 08:00*

## 前言

上篇文章 [《LangChain DeepAgents 速通指南（四）—— FileSystem中间件：让AI Agent拥有系统级记忆管理能力》](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485864&idx=1&sn=c23ae9fa7a37ea4136d4124c61160488&scene=21#wechat_redirect) 深入探讨了LangChain的文件管理中间件。回顾整个系列，截至目前笔者已陆续讲解了四种关键中间件：Summarization（记忆管理中间件）、TodoList（任务规划中间件）、ToolSelector（工具选择中间件）以及FileSystem（文件管理中间件）。

这些中间件与LangChain的 `create_agent` API 相结合，共同构成了LangChain团队推出的深度框架—— **DeepAgents** 。借助这一框架，仅需数十行代码，大家便可构建出一个类似Claude Code的深度智能体，堪称未来开发中必须掌握的关键技术之一。

从本期内容开始，笔者将和大家一起系统学习DeepAgents框架的使用技巧。作为开篇，今天笔者先快速介绍DeepAgents框架的基本使用方法及其核心特性。

PS:鉴于后台私信越来越多，我建了一些大模型交流群，大家在日常学习生活工作中遇到的大模型知识和问题都可以在群中分享出来大家一起解决！如果大家想交流大模型知识，可以关注我并回复加群

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

## 一、为什么需要DeepAgents框架

当大家使用 LangChain 的 `create_agent` API 构建智能体时，其底层本质是利用 LangGraph 创建了一个 ReAct 图（不熟悉 ReAct 图的朋友，可以参考我的文章 [《深入浅出LangGraph AI Agent智能体开发教程（八）——LangGraph底层API实现ReACT智能体》](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247484880&idx=1&sn=d608e1dd364420a527f921d2fb292bae&scene=21#wechat_redirect) ）。在这种架构下，智能体会循环调用工具，直到问题解决——这其实是 Agent 最简单的工作模式。然而，这种简单的架构限制了 Agent 的“深度”，使其难以有效规划并执行更长时序、更复杂的任务。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

如何解决构建复杂深度智能体的难题呢？LangChain 团队的 CEO Harrison Chase 从 Claude Code、Manus 等表现出色的深度智能体项目中汲取灵感，编写了最初的 DeepAgents 框架。其核心思路是将 **详尽提示、任务规划、子代理和文件系统** 四个能力相结合，推动 Agent 实现从“浅”至“深”的转变。该项目随后经过不断迭代完善，最终被纳入 LangChain 官方库，成为基于 LangChain 和 LangGraph 底层能力的开源框架。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

DeepAgents 的核心思想其实非常直观：为了让智能体能够更深入地探讨并完成用户需求，它需要具备规划复杂解决方案、并在更长的时间范围内逐个解决子任务的能力，最终整合达成整体目标。 **DeepAgents 的核心优势，正是对以下四个基本能力的合理运用：**

1. **规划与任务分解能力**
	DeepAgents 内置了 **TodoList** 和 **File System 工具** ，使 Agent 能够将复杂任务拆解为离散步骤，实时跟踪进展，并根据新信息的出现动态调整计划。
2. **上下文管理能力**
	通过 File System 提供的 `ls` 、 `read_file` 、 `write_file` 、 `edit_file` 等工具，Agent 可以将复杂的上下文内容“卸载”到外部存储中，有效防止上下文窗口溢出。同时，借助 **Summarization 记忆压缩** 中间件，Agent 可以灵活处理长度不确定的工具返回结果。
3. **子代理生成能力**
	DeepAgents 内置了 **Task 工具** ，使主 Agent 能够针对特定任务动态生成子代理。这样一来，主 Agent 的上下文可以保持“干净”，同时又具备深入处理复杂子任务的能力。（子代理能力笔者会在后续章节详细展开讲解）
4. **长期记忆能力**
	利用不同类型的后端（Backend）进行信息存储，DeepAgents 可以将具备持久记忆的 Agent 扩展到跨线程场景。这意味着 Agent 能够保存并检索过往对话中的信息，为当前任务提供更智能的辅助。

## 二、DeepAgents快速上手：基本用法与核心特性

## 2.1 DeepAgents的定位

在具体实现上，大家可以直接导入 `deepagents` 包中的 `create_deep_agent` 函数，快速构建深度智能体。该函数本质上是对 LangChain 中 `create_agent` 函数的一层封装，额外配备了处理复杂任务所需的常用配置（例如笔者之前介绍过的各种中间件及内置函数）。

总体而言， `create_agent` 与 `create_deep_agent` 是两种 **形式相似但定位不同** 的创建方式，大家可以根据实际场景灵活选用：

- 当面对 **简单任务** 时，直接使用 `create_agent` 即可快速满足需求。
- 而 `create_deep_agent` 则更适合以下 **复杂场景** ：
- 需要多步规划和拆解的多步骤任务；
	- 需要借助文件系统工具管理大量上下文的任务；
	- 需要将子任务委托给专门子代理（Sub-agent）执行的任务；
	- 需要跨对话、跨线程持久化保存记忆的任务。

## 2.2 DeepAgents的基本使用

下面笔者将通过一个具体案例，帮助大家快速上手并理解 DeepAgents 的用法。

**1\. 环境配置** ：使用 Anaconda 创建虚拟环境，并在其中安装 `deepagents` 包:

```bash
conda create -n langchaindeepagents python=3.12
conda activate langchaindeepagents
pip install -U deepagents
```

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**2\. 创建项目文件** ：编写如下代码。这里笔者让 DeepAgents 协助撰写一份关于伊朗和美国文化的分析报告。模型同样使用 DeepSeek，并引入 `rich` 库来美化命令行输出（可通过 `pip install rich` 安装）:

```python
from dotenv import load_dotenv
from deepagents import create_deep_agent
from deepagents.backends import FilesystemBackend
from langchain_deepseek import ChatDeepSeek

load_dotenv()
model = ChatDeepSeek(model="deepseek-chat")
```

**3\. 使用 `create_deep_agent` 构建智能体：** 该 API 有几个关键参数值得关注：

- **`system_prompt`**
	用户自定义的系统提示。DeepAgents 本身已配置了一段默认系统提示，用户提供的提示会被追加在其之前。
- **`tools`**
	与 `create_agent` 一致，接收一个工具对象列表。
- **`backend`**
	DeepAgents 默认集成了 File System 中间件，其后端配置可通过该参数直接传入。
- **`subagents`**
	DeepAgents 支持通过子智能体完成复杂任务，这些子智能体可以是由 LangChain 或 LangGraph 构建的智能体实例

```python
research_instruction = """ 你是一位从事国际关系研究的专家，能够分析不同国家的国情，并按照用户的要求生成报告"""
agent = create_deep_agent(
    model=model,
    tools=[],
    system_prompt=research_instruction,
    subagents=[],
    backend=FilesystemBackend(root_dir="./test_dir", virtual_mode=True),
)
```

**4\. 执行任务** 。笔者这里给智能体一个具体任务：“请分析伊朗和美国的国情，并帮我撰写一份1500字左右的为什么伊朗和美国会对立的报告”。DeepAgents 的执行方式与 `create_agent` 完全一致，这里采用流式输出，并借助 Rich 库优化显示效果，方便直观感受运行过程。（关于 LangChain 流式输出的更多细节，可参考文章 [《LangChain1.0速通指南（二）——LangChain1.0 create\_agent api 基础知识](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485146&idx=1&sn=1397839d622039d1665868f97a329f43&scene=21#wechat_redirect) [》](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485146&idx=1&sn=1397839d622039d1665868f97a329f43&scene=21#wechat_redirect) ， 笔者之后也会撰写专门文章分析如何处理DeepAgents的流式输出：

```python
for event in agent.stream(
    {"messages": [{"role": "user", "content": query}]},
    stream_mode="values",
):
    step_num += 1
    console.print(f"\n[bold yellow]{'─' * 80}[/bold yellow]")
    console.print(f"[bold yellow]步骤 {step_num}[/bold yellow]")
    console.print(f"[bold yellow]{'─' * 80}[/bold yellow]")

    if "messages" in event:
        messages = event["messages"]
        if messages:
            msg = messages[-1]

            # 保存最终响应
            if hasattr(msg, "content") and msg.content and not hasattr(msg, "tool_calls"):
                final_response = msg.content

            # AI 思考
            if hasattr(msg, "content") and msg.content:
                content = msg.content
                console.print(Panel(
                    content,
                    title="[bold green]AI 思考[/bold green]",
                    border_style="green",
                ))

            # 工具调用
            if hasattr(msg, "tool_calls") and msg.tool_calls:
                for tool_call in msg.tool_calls:
                    tool_info = {
                        "工具名称": tool_call.get("name", "unknown"),
                        "参数": tool_call.get("args", {}),
                    }
                    console.print(Panel(
                        JSON(json.dumps(tool_info, ensure_ascii=False)),
                        title="[bold blue]工具调用[/bold blue]",
                        border_style="blue",
                    ))

            # 工具响应
            if hasattr(msg, "name") and msg.name:
                response = str(msg.content)
                console.print(Panel(
                    response,
                    title=f"[bold magenta]工具响应: {msg.name}[/bold magenta]",
                    border_style="magenta",
                ))
console.print("\n[bold green]任务完成![/bold green]\n")
```

**5\. 执行过程分析** ：一起来完整分析上述代码的执行步骤：

- 收到任务后，智能体会首先调用 `write_todos` 工具，编写任务规划。
![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)
- 随后，智能体按照规划列表开始执行。这里值得注意的是：虽然并未传入自定义的 `subagents` ，但智能体在分析伊朗和美国国情时，感知到这两部分可以并行处理，于是自动调用 `task` 工具，利用当前 DeepSeek 模型开启了两个并行的子智能体同时进行资料整理。这正是 DeepAgents 框架的一大特色。
![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)
- 完成相关子任务后，智能体会更新任务状态，将已完成项标记为 `completed` ，并继续迭代，直至生成最终报告。
![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)
- 最终生成的报告如下，DeepAgents还会进一步的审阅和修改报告，确保报告生成的完整性
![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

以上就是本期分享的全部内容，本文的全部代码可关注笔者的微信公众号 **大模型真好玩** ，每期分享涉及的代码均可在公众号私信: **LangChain智能体开发** 免费获取。

## 三、总结

本文分享了 LangChain DeepAgents 框架的诞生背景与核心优势，并通过实战案例演示了其快速上手的完整流程。可以看到，DeepAgents 在任务规划、上下文管理、子代理并行执行等方面展现出强大的能力，仅需少量代码即可构建复杂智能体。下一章节，笔者将深入探究 DeepAgents 中子智能体的运行机制，包括其创建方式、通信机制与任务委托流程，大家敬请期待~

[《深入浅出LangChain&LangGraph AI Agent 智能体开发》](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk3NTA2OTMxNQ==&action=getalbum&album_id=4073973355617976325#wechat_redirect) 专栏内容源自笔者在实际学习和工作中对 LangChain 与 LangGraph 的深度使用经验，旨在帮助大家系统性地、高效地掌握 AI Agent 的开发方法，在各大技术平台获得了不少关注与支持。目前已更新42讲，正在更新LangChain 最新 DeepAgents框架的相关内容，并随时补充笔者在实际工作中总结的拓展知识点。如果大家感兴趣，欢迎关注笔者的微信公众号 大模型真好玩，每期分享涉及的代码均可在公众号私信: LangChain智能体开发免费获取。

2026年笔者的另一大重心还会放在 **[大模型训练](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk3NTA2OTMxNQ==&action=getalbum&album_id=4338985715301089302#wechat_redirect)** [专栏](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk3NTA2OTMxNQ==&action=getalbum&album_id=4331818996941979657#wechat_redirect) 分享上，当然，谈论训练无法绕过算力这道现实的门槛。昂贵的GPU是许多人探索技术的拦路石。正因如此笔者今年特意与一些可靠的算力平台展开了合作，希望能为大家解决算力瓶颈。大家可以扫描下方二维码注册 **Lab4ai算力平台** ，提供了包括英伟达H系列在内的多种选择，更有 **5小时的免费体验额度** 。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

深入浅出LangChain AI Agent 智能体开发 · 目录

继续滑动看下一个

大模型真好玩

向上滑动看下一个
