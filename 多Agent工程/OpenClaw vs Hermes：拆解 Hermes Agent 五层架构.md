---
title: "OpenClaw vs Hermes：拆解 Hermes Agent 五层架构"
source: "https://mp.weixin.qq.com/s/DLZFEsqxRMq0ntilgqpEjA"
author:
  - "[[叶小钗]]"
published:
created: 2026-04-28
description: "一条消息在 Hermes Agent 里经历了什么？万字拆解五层架构"
tags:
  - "clippings"
---
## Agent 主循环

消息到 `AIAgent`，进入整个项目最核心的地方，这个地方一定要详细读、重复读：

#### 主循环骨架

```
while iteration_budget.remaining > 0:    response = client.chat.completions.create(        model=model, messages=messages, tools=tool_schemas, stream=True    )    if response 有 tool_calls:        执行工具(可能并行)        iteration_budget.consume()    else:        return response.content  # 没有工具调用,返回最终结果
```

主循环有三种退出路径：

- **模型给最终文本**:本轮没有 tool\_calls,走 else 分支把 `response.content` 返回给用户,正常完结。
- **预算耗尽**:while 条件不再成立,`iteration_budget.remaining` 归零。这是硬上限,防止模型在错误循环或幻觉里把 token 烧光。
- **用户中断**:`_interrupt_requested` 被外部置位。用户 Ctrl+C 或者直接发新消息都会触发,Agent 在每轮开头检查这个标志。收到中断后**不是 raise 抛异常**,而是 break 出循环,持久化已有结果并补齐消息结构。

#### 迭代预算

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/6Uzn2S5AAyRmkZaouibBXPFr06joUVC03fhUMxSnGWacEZso8QicLAlWVOByicWT0dAm9IPjPwr23wLr8JfYcojRm65OxibkZdBpVyWubrDNpL4/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=4)

系统设计有循环迭代预算限制:父 Agent 上限 90 轮,子 Agent 50 轮。

**模型每推理一轮消耗 1 次迭代预算**,不管这一轮并行调了几个工具。

有意思的是 `refund()` 的触发条件:

```
_tc_names = {tc.function.name for tc in assistant_message.tool_calls}if _tc_names == {"execute_code"}:    self.iteration_budget.refund()
```

**当本轮工具调用里只有 `execute_code` 一种**,刚扣掉的那 1 次迭代会被退还,这轮等于白送。

`execute_code` 是 PTC(Programmatic Tool Calling):模型不是直接挨个调工具,而是**写一段 Python 脚本**,脚本内部通过 RPC 把 web\_search、read\_file、write\_file 这些工具串起来跑。

换个角度对比一下:同样要做 8 次信息获取,

- 走普通工具调用:模型调 web\_search → 拿结果 → 再推理下一步调什么 → 调 read\_file → 拿结果 → 再推理 ……8 次工具执行要 **8 轮模型推理**,吃掉 8 次迭代预算。
- 走 PTC:模型一轮里写出一整段脚本,脚本自己连调 8 次工具。**1 轮模型推理**就打包干完。

PTC 已经把 8 次工具调用折成 1 轮推理,系统再把这 1 轮也免掉,执行脚本在预算里等于零成本。

退还的真正作用是**预算管理**:脚本密集型任务可能要连写十几个脚本才做完,一次扣 1 轮的话,90 轮预算很快被脚本执行吃掉,留给真正推理轮次的就不够了。索性让脚本执行零成本,预算就能全留给需要推理的轮次。

#### 工具并行执行

系统维护三个集合决定一批工具能不能并行:

```
_NEVER_PARALLEL_TOOLS = frozenset({"clarify"})           # 会跟用户交互_PARALLEL_SAFE_TOOLS = frozenset({                        # 只读,无共享状态    "read_file", "search_files", "session_search",    "skill_view", "skills_list",    "vision_analyze", "web_extract", "web_search",    "ha_get_state", "ha_list_entities", "ha_list_services",})_PATH_SCOPED_TOOLS = frozenset({"read_file", "write_file", "patch"})  # 路径不重叠才能并行
```

路径工具的冲突检查原理:提取每次调用的目标路径,**两两比对看有没有重叠**。

重叠的判定包括两种情况:同一个路径,或者一个路径是另一个的祖先(比如 `/a` 和 `/a/b.txt`)。

只要重叠,就可能撞上读写竞态(一个线程正在写,另一个读到半拉状态),必须排队串行;路径完全独立则放并行。

举两个例子:

- `read_file("/a/b.txt")` + `write_file("/a/b.txt")`:同一个文件,一个读一个写,并行会出乱子,必须串行
- `read_file("/a/x.txt")` + `read_file("/b/y.txt")`:两条路径完全独立,可以并行

并行池最多 8 个工作线程同时跑。

如果模型判断要同时读 5 个文件、搜 2 个关键词、查 3 个网页时,串行要 10 次 API 往返,并行可能 2-3 次搞定。每次 API 调用都是时间 + 金钱。

#### delegate\_task

![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/6Uzn2S5AAyT5jvRpwllTROe0mFc5MUUylv2JA2oTsYCtCe3a9LyEDhOO6QrYJzWtG1YP9YDAwebrb9A24BHp9nhTZ0GRpnaOMhVbGu9wxgw/640?wx_fmt=jpeg&from=appmsg&tp=webp&wxfrom=5&wx_lazy=1#imgIndex=5)

`delegate_task` 是个特殊工具:模型选它的时候,会 fork 一个新的 `AIAgent`。

子 Agent 有**自己独立的上下文**,也有**自己独立的 50 轮迭代预算**,父子之间只通过任务描述(传入)和最终摘要(传出)通信,除此之外彼此看不见。子 Agent 被禁用 5 个工具:

- `delegate_task`:防套娃。Agent 嵌套本身就有开销,再允许无限递归成本会爆
- `clarify`:子 Agent 不能反问人,因为用户不在场,只有父 Agent 是跟人对话的那一层
- `memory`:子 Agent 不能写共享记忆,避免一次临时委托里抓到的噪声,污染所有未来会话
- `send_message`:子 Agent 不能直接往平台发消息,对外沟通只能经由父 Agent
- `execute_code`:子 Agent 定位就是一步步推理把事做完,不该再用 PTC 折叠(PTC 是主 Agent 用来节省轮次的,子 Agent 本来就分到了独立预算,用不着)

结构上还有两条硬约束:**委托深度只有 1 层**(父→子,子 Agent 禁用了 `delegate_task` 无法再委托)、**并发上限 3 个**。

源码里虽然设了 `MAX_DEPTH = 2`,注释写"parent(0)→child(1)→grandchild rejected(2)",但子 Agent 已经拿不到 `delegate_task` 工具了,这个深度检查是双重保险，防的是工具集被手动调整绕过黑名单的极端情况。

父 Agent 每 30 秒给子发一次心跳,一旦父被用户中断或者自己挂了,心跳断开,子 Agent 就会连锁停下,这就是"级联中断"。没有这个机制,用户按了 Ctrl+C 之后,后台还会有一堆子 Agent 继续烧 token。

子 Agent 的系统提示词强调的是**边界**而不是人格:做这一件事、给摘要、不用关心父 Agent 在干什么。

主 Agent 的上下文**只会看到委托调用本身和最终摘要**,看不到子 Agent 那可能 20 次工具调用的中间过程。

主 Agent 能处理多少轮用户消息才触顶上下文压缩,取决于它的上下文保持得多干净,一次把重活甩给子 Agent、只把摘要收回来,等于用一点并行开销换主 Agent 的长寿。

回到开头那个新闻例子:主 Agent 给科技/财经/国际各委托一个子 Agent 并行跑,拿摘要自己汇总分类。主 Agent 只花 1 次迭代预算,子 Agent 的 50 次预算各自独立。