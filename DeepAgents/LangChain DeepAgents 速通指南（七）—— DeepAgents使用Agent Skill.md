---
title: "LangChain DeepAgents 速通指南（七）—— DeepAgents使用Agent Skill"
source: "https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247486027&idx=1&sn=79e034e9fbd03fd8f532b7abef3d6411&chksm=c4d003a7f3a78ab1fefbb87c4db0ff53b884d75c3d5f5498a69f87f875f1923ab0d00e8f3027&scene=178&cur_album_id=4073973355617976325&search_click_id=#rd"
author:
  - "[[手摸手教你大模型]]"
published:
created: 2026-04-28
description: "本期全面解析了DeepAgents中的Agent Skill机制。DeepAgents通过SkillsMiddleware与FileSystemMiddleware的协同，完整实现了Skills的发现、激活与执行流程。本文从Skills快速回顾入手，剖析了工程实现的四个核心步骤，并通过两个实战案例展示了具体接入方法。"
tags:
  - "clippings"
---
手摸手教你大模型 *2026年4月22日 17:30*

## 前言

上篇文章 [《LangChain DeepAgents 速通指南（六）—— DeepAgents SubAgent 子智能体机制》](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485975&idx=1&sn=c9c1fbde0ac31ff6cca02fe17abeaee4&scene=21#wechat_redirect) 详细解析了 DeepAgents 中的子智能体系统。今天，笔者将继续和大家探讨智能体工程中一项必备能力—— **Agent Skill** ，以及如何在 DeepAgents 中快速接入 Skills。DeepAgents 提供了非常便捷的 Skills 接入方式，通过整合各种 Skill，可以极大地扩展智能体的能力边界。接下来就和笔者一起学习吧！（原计划本期介绍流式输出机制，经过权衡，决定优先讲解 Skills 的接入与应用。）

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/I5P6NLY2q6VfYEZWHDIkL8MjQ8aqoCWCTSicq0XiaEufc1T1ibLswKbCk2PEV3ZRey2h4f2WbphX3ghzaV67NZsc2joxK3qQjiaXksMyuuOndlw/640?wx_fmt=webp&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=0)

PS:鉴于后台私信越来越多，我建了一些大模型交流群，大家在日常学习生活工作中遇到的大模型知识和问题都可以在群中分享出来大家一起解决！如果大家想交流大模型知识，可以关注我并回复加群

## 一、Skills快速回顾

为了照顾还不熟悉 Skills 概念的读者，这里先快速回顾一下。（建议先阅读笔者的文章 [《 Agent Skills完全指南：核心概念丨设计模式丨实战代码》](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485633&idx=1&sn=07eaae7b630762ef19f9d8857dbc87a0&scene=21#wechat_redirect) ，对 Skills 有一个全面的理解后，再继续看本篇文章，会事半功倍。）

Skills 是 Anthropic 提出的一种全新的 Agent 能力注入方式，可以看作是一种轻量级的开放格式。通过 Skills，开发者能够将特定领域的专业知识打包成可被智能体发现和调用的功能模块。

无论是 Claude Code、OpenClaw，还是正在学习的 DeepAgents，Skills 都遵循统一的目录结构规范。一个 Skill 本质上就是一个文件夹，其中至少包含一个 `SKILL.md` 描述文件，此外还可以附带可执行脚本、参考文档以及资源文件等。

```text
my-skill/
├── SKILL.md          # 必须: name + description
├── scripts/          # 可选: 执行代码
├── references/       # 可选: 参考资料文档
└── assets/           # 可选: 模板、图片等资源文件
```

每个 Skill 文件夹中必须包含 `SKILL.md` 文件（注意文件名大小写不要出错）。下面是一个通用的 `SKILL.md` 示例：

```markdown
---
name: pdf-processing
description: Extract text and tables from PDF files, fill forms, merge documents.
---

# PDF Processing

## When to use this skill

Use this skill when the user needs to work with PDF files...

## How to extract text

1. Use pdfplumber for text extraction...

## How to fill forms  ...
```

其中， `name` 是 Skill 的名称， `description` 是该 Skill 的简短描述，这两项 **不可为空** 。 `SKILL.md` 的正文部分用于定义该 Skill 的专业知识、操作流程或解决思路。此外，用户还可以在文件夹下增加 `scripts` 工具脚本、 `references` 参考文档或 `assets` 资源文件，以进一步丰富 Skill 的能力。

Skill 的核心设计理念是 **“渐进式加载”** 。具体来说：智能体在初始化时，只会将每个 Skill 的基本信息（即 `name` 和 `description` ）加载到系统提示词中。只有当后续对话中智能体根据用户意图判断出需要用到某个具体 Skill 时，才会进一步加载该 Skill 的详细信息（即 `SKILL.md` 的正文内容），甚至按需加载脚本、模板等额外资源。

## 二、 DeepAgents Skills详解

## 2.1 DeepAgents Skills使用说明

DeepAgents 作为 LangChain 团队的明星框架，对 Skill 的支持相当完善。框架内部已经封装好了 **发现、激活、执行** 这一完整流程，因此开发者只需专注于定义 Skill，然后将 Skill 所在的目录路径传递给 `DeepAgents` 即可。例如：

```python
agent = create_deep_agent(
    model=llm,
    skills=["/skills"],  # 技能包所在目录
)
agent.invoke("你有哪些技能？")
```

## 2.2 Skills工程化的实现思路

对于屏幕前好学的读者而言，仅仅会使用 Agent Skill 是不够的，大家还需要理解其工程实现。总体来看，Agent Skill 的工程化实现基本都遵循以下四个步骤：

1. **发现与识别 Skills**
	Agent 需要能够管理文件系统，在配置好的目录中发现 Skills 文件夹。系统会扫描每个子文件夹，读取其中的 `SKILL.md` ，并提取文件头部的 YAML 元数据（即 `name` 和 `description` ）。
2. **系统提示词注入**
	将所有 Skill 的元数据（名称 + 描述）注入到系统提示词中，使得大模型在每一轮对话开始时都能清楚看到有哪些技能可用，以及各自的简要用途。
3. **渐进式加载**
	当模型决定使用某个 Skill 时，系统才会进一步读取该 Skill 的完整说明（即 `SKILL.md` 的正文），将其加载到上下文中，使后续行动有据可依。
4. **任务执行与完成**
	模型按照 `SKILL.md` 中的详细说明，调用必要的工具来访问附加资源，并最终完成任务。

## 2.3 DeepAgent 的 Skills 实现机制

DeepAgents 框架正是按照上述通用思路实现的。笔者此前提到过，DeepAgents 的本质是基于 LangChain 1.0 的 `create_agent` 进行深度定制，并通过 **中间件机制** 完成了对 Skills 的工程化支持。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

下面结合中间件，逐一对上述四个步骤进行解析：

### 1\. 发现与识别 Skills

DeepAgents 通过 **`FileSystemMiddleware`** 获得了操作本地文件系统目录的能力（该中间件的详细实现可参考笔者之前的文章 [《LangChain DeepAgents 速通指南（四）—— FileSystem 中间件：让 AI Agent 拥有系统级记忆管理能力》](https://mp.weixin.qq.com/s?__biz=Mzk3NTA2OTMxNQ==&mid=2247485864&idx=1&sn=c23ae9fa7a37ea4136d4124c61160488&scene=21#wechat_redirect) ）。框架内置的实现代码会扫描指定的 Skills 目录，读取每个子目录内的 `SKILL.md` ，解析其顶部的 YAML 区域，提取 `name` 、 `description` 等信息，并格式化为一个 `SkillMetadata` 列表。

### 2\. 系统提示词注入

DeepAgents 编写了一个 **`SkillsMiddleware`** 中间件。在 `before_agent` 钩子函数中，该中间件会将 `SkillMetadata` 列表组合成一段文本，并附上使用 Skills 的指令与提示。最终形成的提示片段类似如下结构：

```text
**可用 Skill**
- fullstack-template-generator: ...
- web-research: ...

**如何使用（渐进式加载原则）**
Skills follow a progressive disclosure pattern...

**什么时候使用 Skill**
...

**如何执行 Skill 中的脚本**
...

**Skill 使用流程示例**
...

**Skill 使用注意点**
...
```

同时该中间件还会在 `wrap_model_call` 方法中将上述 Skills 提示附加到 `system_prompt` 中，确保模型在每次调用时都能看到这些信息。

### 3\. 渐进式加载

模型通过 `SkillMetadata` 已经知道了如何选择和使用某个 Skill。当模型的输出与使用某个 Skill 相关时，就会触发渐进式加载流程。由于 DeepAgents 的 `FileSystemMiddleware` 已经配置了 `read_file` 等文件系统读取工具，此时系统可以读取对应 `SKILL.md` 的完整正文，从而获得该 Skill 的详细说明书。

### 4\. 任务执行与完成

拿到完整的 Skill 说明之后，智能体会在任务执行过程中按需加载必要的附加资源（如 `scripts/` 、 `references/` 、 `assets/` 中的文件），并调用相应的工具来完成任务。

---

以上就是 DeepAgents 实现 Agent Skill 机制的完整解析。是不是非常简单明了？

## 三、DeepAgents Skill实战

掌握了 DeepAgents Agent Skill 的原理之后，接下来大家进入实战环节，看看如何真正在 DeepAgents 框架中使用 Agent Skill。

## 3.1 简单 Skill 尝试

笔者先从一个简单的示例入手，演示 DeepAgents 中 Skills 的接入流程。

**1\. 准备示例 Skill**  
选用 GitHub 上官方提供的示例 Skill：  
`https://github.com/langchain-ai/deepagents/tree/main/libs/cli/examples/skills/langgraph-docs`  
将该 Skill 下载到项目文件夹下的 `skills` 目录中，目录结构如下图所示：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**2\. 编写 Python 代码**  
新建一个 Python 文件，导入所需依赖，并定义模型和持久化存储：

```python
from dotenv import load_dotenv
from deepagents import create_deep_agent
from deepagents.backends.filesystem import FilesystemBackend
from langchain_deepseek import ChatDeepSeek
from langgraph.checkpoint.memory import MemorySaver

load_dotenv()
model = ChatDeepSeek(model="deepseek-chat")
checkpointer = MemorySaver()
```

**3\. 创建 Deep Agent 并指定 Skills 目录**  
使用 `create_deep_agent` 创建深度智能体，设置文件系统后端 `FilesystemBackend` 的根目录为当前项目目录，并通过 `skills` 参数指定 Skills 存放路径为当前目录下的skills文件夹（`./skills/` ）。  
大家可能会问：为什么这里没有显式引入 `SkillsMiddleware` ？因为 `create_deep_agent` 在传入 `skills` 参数时会 **自动加载** `SkillsMiddleware` ，省去了手动配置的麻烦（有兴趣可以查看源码确认）

```python
agent = create_deep_agent(
    model=model,
    backend=FilesystemBackend(root_dir="./", virtual_mode=True),
    skills=["./skills/"],
    checkpointer=checkpointer,  # Required!
)
```

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**4\. 运行测试:**

**向智能体提问：“What is langgraph?”。 DeepAgents 会根据** `langgraph-docs` Skill 的 `description` 识别用户意图，然后渐进式加载该 Skill 的完整内容并回答

```python
result = agent.invoke(
    {
        "messages": [
            {
                "role": "user",
                "content": "What is langgraph?",
            }
        ]
    },
    config={"configurable": {"thread_id": "12345"}},
)
print(result)
```

从执行结果（下图）可以看到，DeepAgents 准确识别了 Skills 并给出了回答。  
**注意** ：如果无法识别 Skills，请优先检查路径配置是否正确。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

## 3.2 复杂 Skill 尝试

上面的示例只是一个比较简单的 Skill。接下来笔者尝试一个日常办公中常用的 Skill —— docx文档处理 Skill。

**1\. 获取 Skill**  
访问 SkillHub 官网 https://www.skillhub.cn/，搜索 `doc` ，然后通过 ZIP 方式下载对应的 Skill：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

将下载的 ZIP 包解压，内容同样放到项目目录下的 `skills` 文件夹中：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**2\. 编写代码**

- **导入依赖**
	注意某些 Skill 可能需要执行命令行脚本（例如安装 `scripts` 中所需的依赖），因此需要引入 `LocalShellBackend` 来支持命令执行。同时定义模型和上下文记忆。

```python
from pathlib import Path

from dotenv import load_dotenv
from deepagents import create_deep_agent
from deepagents.backends import LocalShellBackend
from langchain_deepseek import ChatDeepSeek
from langgraph.checkpoint.memory import MemorySaver

load_dotenv()
model = ChatDeepSeek(model="deepseek-chat")
# 上下文记忆
checkpointer = MemorySaver()
```

- **配置后端**
	定义项目根目录和命令执行目录。注意使用 `inherit_env=True` 继承系统环境变量，以便命令可以正常执行。同时将路径中的分隔符统一转换为 `/` （Windows 下避免使用 \`\\\`）。

```python
root_dir = Path.cwd().as_posix() # 转换文件分隔符为/格式而不是windows的\格式
print(root_dir)
backend = LocalShellBackend(
    root_dir=root_dir,
    inherit_env=True,
    timeout=120,  # 命令超时秒数
    max_output_bytes=100000,
)
```

- **定义系统提示词并创建智能体**
	再次强调，传入 `skills` 参数后 `create_deep_agent` 会自动加载 `SkillsMiddleware` 。提示词中特别提醒： `read_file` 工具不支持 Windows 绝对路径（如 `D:\xxx\SKILL.md` ），应使用 POSIX 格式（如 `/xxx/xxx/SKILL.md` ）。

```python
system_prompt = '''
## 角色设定
你是一位专业、高效、多领域的超级智能助手，具备强大的知识整合与问题解决能力。

## 核心任务
- 根据用户提问，结合你的专业知识库与可用工具（skills），提供高质量解答。
- 回答需遵循：准确性 > 实用性 > 简洁性 > 友好性 的优先级原则。
- 遇到模糊问题时，主动澄清需求；遇到复杂问题时，分步骤拆解说明。

## 注意事项
read_file 工具不支持 Windows 绝对地址，如错误写法 D:\xxx\xxx\SKILL.md；
正确写法为 /xxx/xxx/SKILL.md。
'''
agent = create_deep_agent(
    model=model,
    backend=backend,
    skills=[root_dir + r"/skills"],
    system_prompt=system_prompt,
    checkpointer=checkpointer,
)
```

- **编写流式输出并测试**
	以“编写 100 字左右的笑话并保存到 笑话.docx 中”作为测试输入，观察智能体是否调用 `doc` Skill 并完成任务。

```python
while True:
    question = input("请输入:")
    if not question:
        continue
    if question == "q":
        break

    for type, chunk in agent.stream(
        {
            "messages": [
                {
                    "role": "user",
                    "content": question,
                },
            ]
        },
        config={"configurable": {"thread_id": "12345"}},
        stream_mode=["updates"],
    ):
        if "SkillsMiddleware.before_agent" in chunk and chunk["SkillsMiddleware.before_agent"]:
            skills = chunk["SkillsMiddleware.before_agent"]["skills_metadata"]
            print(">" * 10, "加载Skills", "<" * 30)
            for skill in skills:
                print("Load Skill:", skill["name"])

        if "model" in chunk:
            message = chunk["model"]["messages"][0]
            content = message.content
            if content:
                print(">" * 10, "AIMessage", "<" * 30)
                print(content)

            tool_calls = message.tool_calls
            if tool_calls:
                print(">" * 10, "Call Tools", "<" * 30)
                for t in tool_calls:
                    print(f"Tool:{t['name']}, Args:{t['args']}")

        if "tools" in chunk:
            print(">" * 20, "Tools Output", "<" * 20)
            for m in chunk["tools"]["messages"]:
                print(f"Tool:{m.name}, Output: \n{m.content}")
```

**3\. 运行结果**

测试结果显示，智能体成功调用了 `doc` Skill，并按要求生成了笑话文档：

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E) ![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E) ![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

以上就是本期分享的全部内容，本文的全部代码可关注笔者的微信公众号 **大模型真好玩** ，每期分享涉及的代码均可在公众号私信: **LangChain智能体开发** 免费获取。

## 四、总结

本期全面解析了DeepAgents中的Agent Skill机制。DeepAgents通过 `SkillsMiddleware` 与 `FileSystemMiddleware` 的协同，完整实现了Skills的发现、激活与执行流程。本文从Skills快速回顾入手，剖析了工程实现的四个核心步骤（发现、注入、渐进加载、执行），并通过两个实战案例（基础技能识别与文档处理Skill）展示了具体接入方法。合理使用Skills能极大扩展智能体能力边界，但需注意路径配置与系统提示词规范。下期将分享DeepAgents的流式输出机制，敬请期待！

[《深入浅出LangChain&LangGraph AI Agent 智能体开发》](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk3NTA2OTMxNQ==&action=getalbum&album_id=4073973355617976325#wechat_redirect) 专栏内容源自笔者在实际学习和工作中对 LangChain 与 LangGraph 的深度使用经验，旨在帮助大家系统性地、高效地掌握 AI Agent 的开发方法，在各大技术平台获得了不少关注与支持。目前已更新43讲，正在更新LangChain 最新 DeepAgents框架的相关内容，并随时补充笔者在实际工作中总结的拓展知识点。如果大家感兴趣，欢迎关注笔者的微信公众号 大模型真好玩，每期分享涉及的代码均可在公众号私信: LangChain智能体开发免费获取。

2026年笔者的另一大重心还会放在 **[大模型训练](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk3NTA2OTMxNQ==&action=getalbum&album_id=4338985715301089302#wechat_redirect)** [专栏](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=Mzk3NTA2OTMxNQ==&action=getalbum&album_id=4331818996941979657#wechat_redirect) 分享上，当然，谈论训练无法绕过算力这道现实的门槛。昂贵的GPU是许多人探索技术的拦路石。正因如此笔者今年特意与一些可靠的算力平台展开了合作，希望能为大家解决算力瓶颈。大家可以扫描下方二维码注册 **Lab4ai算力平台** ，提供了包括英伟达H系列在内的多种选择，更有 **5小时的免费体验额度** 。

![图片](data:image/svg+xml,%3C%3Fxml version='1.0' encoding='UTF-8'%3F%3E%3Csvg width='1px' height='1px' viewBox='0 0 1 1' version='1.1' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink'%3E%3Ctitle%3E%3C/title%3E%3Cg stroke='none' stroke-width='1' fill='none' fill-rule='evenodd' fill-opacity='0'%3E%3Cg transform='translate(-249.000000, -126.000000)' fill='%23FFFFFF'%3E%3Crect x='249' y='126' width='1' height='1'%3E%3C/rect%3E%3C/g%3E%3C/g%3E%3C/svg%3E)

**微信扫一扫赞赏作者**

深入浅出LangChain AI Agent 智能体开发 · 目录

继续滑动看下一个

大模型真好玩

向上滑动看下一个
