---
title: "LangChain DeepAgents 速通指南（四）—— FileSystem中间件：让AI Agent拥有系统级记忆管理能力"
source: "https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485864&idx=1&sn=c23ae9fa7a37ea4136d4124c61160488&chksm=c4d00044f3a7895284218d8b56cc569705574ee2142b646b11b8a9eac7162cc69fc96c1fb520&scene=178&cur_album_id=4073973355617976325&search_click_id=#rd"
author:
  - "[[手摸手教你大模型]]"
published:
created: 2026-04-28
description: "FileSystem 中间件为 LangChain Agent 赋予文件管理能力，通过四种后端实现不同层级的记忆模式：线程级短期记忆、跨线程长期记忆、本地磁盘持久化和混合路由，从而灵活应对从临时草稿到长期记忆的存储需求，有效管理上下文窗口。"
tags:
  - "clippings"
---
手摸手教你大模型 *2026年3月11日 17:35*

## 前言

上篇文章 [《LangChain DeepAgents 速通指南（三）—— 让Agent告别混乱：Tool Selector与Todo List中间件解析》](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485820&idx=1&sn=5a1383f51019086be8078a2e464e7ef2&scene=21#wechat_redirect) 深入探讨了 `ToolSelector` 和 `TodoList` 两个中间件， `ToolSelector` 中间件能够在调用主模型前，利用大模型智能筛选相关工具。 `TodoList` 中间件通过注入 `write_todo` 工具函数，让Agent在遇到复杂任务时自动拆解子任务并维护状态。正是这两者的协同，显著提升了 LangChain DeepAgents 框架下智能体执行复杂任务的效能。

通过前几篇的学习大家已经具备了构建“能调用工具、执行任务的智能体”的基本能力。然而，当任务复杂度进一步提升，一个棘手的问题逐渐浮出水面—— **上下文窗口的管理** 。智能体依赖的工具（如网页搜索、RAG 检索）常常返回大量信息，这些冗长的内容会迅速填满有限的上下文窗口，导致模型性能下降，甚至遗忘关键指令。此时需要一种机制，让智能体能够将重要信息“存档”下来，待需要时再读取——这正是 **FileSystem 中间件** 的用武之地。

本文将带大家深入理解 LangChain DeepAgents 中的 **FileSystem 中间件** ，学习如何为智能体赋予文件管理能力，并通过四种不同的后端配置，实现从短期草稿到长期记忆的灵活存储。

PS:鉴于后台私信越来越多，我建了一些大模型交流群，大家在日常学习生活工作中遇到的大模型知识和问题都可以在群中分享出来大家一起解决！如果大家想交流大模型知识，可以关注我并回复加群

## 一、为什么需要 FileSystem 中间件？

在构建具备长期运行能力的智能体时， **记忆** 是维持智能体状态一致性的核心要素。以近期备受关注的小龙虾 OpenClaw 项目为例（关于其原理，可参考笔者的文章 [《一文读懂 OpenClaw 核心特性与原理解析》](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485669&idx=1&sn=e91413a87ae9c3767efd04b0292c8068&scene=21#wechat_redirect) ），该项目正是通过将用户与大模型的对话历史、用户偏好等内容写入后端的文件系统，实现了对用户记忆的持久化保持。而作为“宇宙最强智能体框架”的 LangChain，自然也提供了类似的能力。

LangChain DeepAgents 框架中的 **FileSystem 中间件** ，正是为解决智能体的记忆问题而设计的。它允许智能体将关键信息以 **文件形式** 写入本地或内存中的“文件系统”，并在后续步骤中随时读取、修改或追加内容。这相当于为 Agent 配备了一个私人笔记本，使其能够跨越多次交互保持记忆一致性，从而支撑起更加复杂、长期的自动化任务流程。

## 二、快速上手FileSystem中间件

## 1\. 安装依赖

在使用 FileSystem 中间件之前，需要先安装 LangChain DeepAgents 的相关依赖。执行以下命令即可安装 `deepagents` 包：

```nginx
pip install deepagents
```

## 2\. 基本使用示例

接下来笔者通过示例代码带大家了解 `FileSystemMiddleware` 中间件的基本用法。与其他中间件一样，它需要传入 `create_agent` API 的 `middlewares` 配置中。配置完成后，Agent 将自动获得以下四个文件操作工具：

- **`ls`**
	列出当前文件系统中的文件列表。
- **`read_file`**
	读取指定文件的全部内容或特定行数。
- **`write_file`**
	创建新文件并写入内容。
- **`edit_file`**
	对已有文件进行修改（支持追加、替换等操作）。

```makefile
from langchain.agents import create_agentfrom langchain_deepagents import FileSystemMiddleware
agent = create_agent(    model=model,    middlewares=[        FileSystemMiddleware(            # 核心配置参数：BACKEND，决定记忆的存储方式            backend=None,            # 可选：自定义系统提示和工具描述            system_prompt="Write to the filesystem when...",            custom_tool_descriptions={                "ls": "Use the 1s tool when..."                "read file":"Use the read file tool to.        )    ])
```

### 3\. 参数说明

`FileSystemMiddleware` 的主要参数如下：

- **`backend`**
	（可选）：该参数决定记忆的存储模式，是 FileSystem 中间件的核心配置，后续章节将重点展开讲解。
- **`system_prompt`**
	（可选）：允许开发者重写默认的系统提示词，以引导 Agent 在何时使用文件操作。
- **`custom_tool_descriptions`**
	（可选）：可用于自定义各个工具的描述文本，以适应特定场景的需求。

在初步使用时，通常无需额外配置 `system_prompt` 和 `custom_tool_descriptions` ，直接使用默认设置即可快速体验文件系统管理能力。

## 三、Backend 参数详解：四种存储模式

`FileSystemMiddleware` 的核心在于 `backend` 参数，它决定了文件的存储位置和生命周期，直接影响智能体记忆的持久化程度。LangChain DeepAgents 针对不同应用场景，提供了四种内置后端类型，下面笔者将逐一介绍。

### 3.1 FileSystemBackend：访问本地磁盘

`FileSystemBackend` 赋予 Agent 直接访问本地文件系统的能力，允许它在指定目录下读写真实文件。配置时需要设置 `backend` 为 `FileSystemBackend` 实例，并通过 `root_dir` 参数限定可操作的根目录，确保访问范围受控。此外， `virtual_mode` 参数可以隐藏绝对路径，进一步提升安全性。

笔者编写了如下示例代码测试 `FileSystemBackend` 的使用：

```python
from dotenv import load_dotenvfrom langchain.agents import create_agentfrom langchain_deepseek import ChatDeepSeekfrom deepagents import FilesystemMiddlewarefrom deepagents.backends import FilesystemBackendfrom langchain.messages import HumanMessage
load_dotenv()
model = ChatDeepSeek(    model="deepseek-chat",)
agent_local = create_agent(    model=model,    tools=[],    middleware=[        FilesystemMiddleware(            backend=FilesystemBackend(root_dir="./test_dir", virtual_mode=True)        )    ])
res = agent_local.invoke(    {        'messages': [HumanMessage("调用工具写入一个文件，文件名为:测试.txt, 内容为: '测试'")]    })
print(res)
```

运行结果如下图所示，可以看到在 `test_dir` 目录下生成了 `测试.txt` 文件，内容为 `测试` ：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E) ![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**注意：** 使用 `FileSystemBackend` 时需格外注意文件安全性。由于大模型存在幻觉和不稳定性，建议对敏感文件提前进行备份和保护。

### 3.2 StateBackend：线程级短期记忆

`StateBackend` 将文件系统“嵌入”到 Agent 的运行状态（State）中，文件内容仅存在于当前线程的生命周期内。它就像一个 **草稿纸** ，适合在单次任务中临时记录中间信息，任务结束后内容自动释放，不留痕迹。

配置时需要使用 Lambda 表达式延迟创建后端对象，因为状态是在运行时动态绑定的。示例如下：

```python
from dotenv import load_dotenvfrom langchain.agents import create_agentfrom langchain_deepseek import ChatDeepSeekfrom deepagents import FilesystemMiddlewarefrom deepagents.backends import FilesystemBackend, StateBackendfrom langchain.messages import HumanMessage
load_dotenv()
model = ChatDeepSeek(    model="deepseek-chat",)agent_local = create_agent(    model=model,    tools=[],    middleware=[        FilesystemMiddleware(            backend=lambda runtime: StateBackend(runtime)        )    ])
res = agent_local.invoke(    {        'messages': [            HumanMessage("调用工具写入一个文件，文件名为:测试.txt, 内容为: '你好帅'"),            HumanMessage('调用工具读取名为测试.txt的文件，告诉我里面的内容')        ]    },)
print(res['messages'][-1].content)
```

运行结果如下，单个线程中 Agent 成功读取了之前写入的内容：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

### 3.3 StoreBackend：跨线程长期记忆

若希望文件内容能在多次对话、多个线程中共享， `StoreBackend` 是最佳选择。它基于一个独立的 **存储对象** 来持久化文件，该存储对象的生命周期决定了文件的生命周期，非常适合需要长期保存记忆或跨会话执行指令的场景。

使用步骤如下：

1. 实例化一个存储对象，例如 `InMemoryStore` （也可自定义持久化存储，如数据库存储）。
2. 将存储对象通过 `store` 参数传入后端。

关于 `InMemoryStore` 的使用涉及到 LangGraph 的长短期记忆原理，大家可参考笔者文章： [深入浅出 LangGraph AI Agent 智能体开发教程（九）—— LangGraph 长短期记忆管理](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247484904&idx=1&sn=f008706fb4131831c709ef4340a0d82f&scene=21#wechat_redirect) 。

示例代码如下：

```python
from dotenv import load_dotenvfrom langchain.agents import create_agentfrom langchain_deepseek import ChatDeepSeekfrom deepagents import FilesystemMiddlewarefrom deepagents.backends import FilesystemBackend, StateBackend, StoreBackendfrom langchain.messages import HumanMessagefrom langgraph.store.memory import InMemoryStore
load_dotenv()
model = ChatDeepSeek(    model="deepseek-chat",)store = InMemoryStore()agent_local1 = create_agent(    model=model,    tools=[],    store=store,    middleware=[        FilesystemMiddleware(            backend=lambda runtime: StoreBackend(runtime)        )    ])
agent_local1.invoke({    'messages':[        HumanMessage("调用工具写入一个文件，文件名为:测试.txt, 内容为: '你好帅'"),    ]})
agent_local2 = create_agent(    model=model,    tools=[],    store=store, # 同一个store实例    middleware=[        FilesystemMiddleware(            backend=lambda runtime: StoreBackend(runtime)        )    ])
res = agent_local2.invoke(    {        'messages': [            HumanMessage('调用工具读取名为测试.txt的文件，告诉我里面的内容')        ]    },)
print(res['messages'][-1].content)
```

执行结果如下图所示，第二个 Agent 能够读取第一个 Agent 写入的内容，说明存储对象独立于线程，写入的内容不会随线程结束而消失：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

### 3.4 CompositeBackend：复合后端（混合存储）

在实际应用中，往往同时需要短期草稿和长期存储。 `CompositeBackend` 允许将 `StateBackend` 和 `StoreBackend` 组合使用，实现“临时文件进状态，重要文件进仓库”的混合模式。智能体可根据文件路径，将不同数据路由到对应的后端。

**工作原理** ：  
创建 `CompositeBackend` 时，通过 `routes` 参数配置一个路径前缀到后端实例的映射字典，并指定一个默认后端：

- **`default` （默认后端）**
	所有不匹配任何特殊前缀的文件操作，都会被路由到此后端处理。通常设置为 `StateBackend` ，用于存放会话内的临时文件。
- **`routes` （路由规则）**
	字典形式，键为路径前缀（如 `"/memories/"` ），值为对应的后端实例（如 `StoreBackend` ）。当 Agent 操作文件（如 `read_file` 、 `write_file` ）时， `CompositeBackend` 会检查文件路径是否以某个路由前缀开头，若是，则将该操作转发给对应的后端。

**路径前缀剥离** ：值得注意的是， `CompositeBackend` 在将文件传递给具体后端存储之前，会 **自动剥离匹配到的路由前缀** 。例如，Agent 写入 `/memories/preferences.txt` 时，实际存储在 `StoreBackend` 中的文件路径为 `/preferences.txt` 。这一设计实现了逻辑路径（智能体看到的）与物理存储路径的解耦，使路由规则更加灵活。

下面通过示例演示 `CompositeBackend` 的使用：

```python
from dotenv import load_dotenvfrom langchain.agents import create_agentfrom langchain_deepseek import ChatDeepSeekfrom deepagents import FilesystemMiddlewarefrom deepagents.backends import FilesystemBackend, StateBackend, StoreBackend, CompositeBackendfrom langchain.messages import HumanMessagefrom langgraph.store.memory import InMemoryStore
load_dotenv()
model = ChatDeepSeek(    model="deepseek-chat",)
store = InMemoryStore()
composite_backend = lambda runtime: CompositeBackend(    default=StateBackend(runtime),    routes={        "/memories/": StoreBackend(runtime)    })
agent = create_agent(    model=model,    store=store,    middleware=[        FilesystemMiddleware(            backend=composite_backend        )    ])
config1 = {"configurable": {"thread_id": '1'}}
# 智能体将 "preferences.txt" 写入 /memories/ 路径agent.invoke({    "messages": [{"role": "user", "content": "我最爱的水果是草莓, 请把我的偏好保存在/memories/preferences.txt"}]}, config=config1)
config2 = {"configurable": {"thread_id": '2'}}
res = agent.invoke({    "messages": [{"role": "user", "content": "请从/memories/获取我最爱的水果是什么?"}]}, config=config2)
print(res['messages'][-1].content)
```

执行结果如下图所示，第二个线程的 Agent 成功读取了第一个线程中写入的 `/memories/` 下的文件，说明复合后端成功将持久化数据路由到了共享存储。这也揭示了中间件命名为 `FileSystemMiddleware` 的缘由：无论后端是内存还是真实文件，都可以被抽象成一个统一的虚拟文件系统供 Agent 使用。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

灵活的应用场景利用 `routes` 参数，大家可以轻松实现各种复杂的记忆策略，甚至可以构建出类似小龙虾 OpenClaw 项目的个性化记忆系统：

1. **用户偏好记忆**
	将用户在不同对话中表达的偏好（如“用简洁的语言回答”）持久化保存在 `/memories/preferences.txt` 中，让智能体在未来的所有互动中自动遵循。
2. **自改进指令**
	智能体可根据用户反馈，动态更新存储在 `/memories/instructions.txt` 中的系统指令，实现自我迭代优化。
3. **跨会话知识库**
	在进行研究项目时，将收集的资料、阶段性笔记保存在 `/memories/research/` 目录下，确保进度跨越多天、多次对话也不会丢失。
4. **混合存储策略**
	可将 `/memories/` 路由到生产级的 `PostgresStore` 实现可靠持久化，同时默认后端仍使用快速、临时的 `StateBackend` 处理会话内临时文件，满足不同数据对持久性的要求。

## 四、理解不同后端的存储原理

选择合适的后端类型，关键在于评估 Agent 的实际需求：是否需要跨对话保持记忆？记忆的敏感程度如何？不同后端对应不同的存储机制和生命周期，理解其原理有助于做出正确决策。

- **StateBackend**
	基于运行时（Runtime）的 `state` 对象实现文件存储。 `state` 对象与当前线程绑定，线程销毁时，其中存储的文件内容也随之释放。这种后端适用于 **单次会话内的临时记忆** ，例如记录中间计算结果或上下文缓存，任务结束后无需保留。
- **StoreBackend**
	利用运行时聚合的 `store` 对象进行文件存储。 `store` 对象独立于线程，其生命周期由开发者控制，因此可以实现跨线程、跨会话的 **长期记忆共享** 。适合需要持久化用户偏好、全局配置或跨多次对话积累知识库的场景。
- **FileSystemBackend**
	直接操作操作系统文件，文件持久化在磁盘上，具有最高的持久性。但使用时需特别注意并发写入冲突和文件权限管理，适用于需要与外部系统交换文件、或存储大规模非结构化数据的场景。
- **CompositeBackend**
	组合多个后端，通过路由规则将不同路径的文件操作分发到对应的后端实例。文件写入时，根据路径前缀决定目标存储；读取时，则可能需要定义优先级或合并策略。这种后端适合构建 **分层记忆系统** ，例如将临时文件保存在内存状态中，而将重要记忆持久化到共享存储或磁盘。

理解这些后端的底层存储原理，可以帮助大家在设计智能体时，根据记忆的持久性要求和数据敏感性，灵活选择最合适的存储方案。

## 五、总结

FileSystem 中间件为 Agent 赋予文件管理能力，通过四种后端实现不同层级的记忆：StateBackend（线程级短期记忆）、StoreBackend（跨线程长期记忆）、FileSystemBackend（本地磁盘持久化）和 CompositeBackend（混合路由），从而灵活应对从临时草稿到长期记忆的存储需求，有效管理上下文窗口。本期内容结束后，关于LangChain DeepAgents框架的典型中间件介绍也就基本结束啦，从下期分享开始，笔者将和大家一起正式开始 `deepagents` 框架的学习，大家敬请期待~

[《深入浅出LangChain&LangGraph AI Agent 智能体开发》](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk3NTA2OTMxNQ==&action=getalbum&album_id=4073973355617976325#wechat_redirect) 专栏内容源自笔者在实际学习和工作中对 LangChain 与 LangGraph 的深度使用经验，旨在帮助大家系统性地、高效地掌握 AI Agent 的开发方法，在各大技术平台获得了不少关注与支持。目前已更新41讲，正在更新LangGraph1.0速通指南，并随时补充笔者在实际工作中总结的拓展知识点。如果大家感兴趣，欢迎关注笔者的微信公众号 大模型真好玩，每期分享涉及的代码均可在公众号私信: LangChain智能体开发免费获取。

2026年笔者的另一大重心还会放在 **[大模型训练](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk3NTA2OTMxNQ==&action=getalbum&album_id=4338985715301089302#wechat_redirect)** [专栏](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk3NTA2OTMxNQ==&action=getalbum&album_id=4331818996941979657#wechat_redirect) 分享上，当然，谈论训练无法绕过算力这道现实的门槛。昂贵的GPU是许多人探索技术的拦路石。正因如此笔者今年特意与一些可靠的算力平台展开了合作，希望能为大家解决算力瓶颈。大家可以扫描下方二维码注册 **Lab4ai算力平台** ，提供了包括英伟达H系列在内的多种选择，更有 **5小时的免费体验额度** 。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

深入浅出LangChain AI Agent 智能体开发 · 目录

继续滑动看下一个

大模型真好玩

向上滑动看下一个