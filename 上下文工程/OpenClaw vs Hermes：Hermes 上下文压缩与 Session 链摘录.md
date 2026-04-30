---
title: "OpenClaw vs Hermes：Hermes 上下文压缩与 Session 链摘录"
source: "https://mp.weixin.qq.com/s/DLZFEsqxRMq0ntilgqpEjA"
author:
  - "[[叶小钗]]"
published:
created: 2026-04-28
description: "一条消息在 Hermes Agent 里经历了什么？万字拆解五层架构"
tags:
  - "clippings"
---
# OpenClaw vs Hermes：Hermes 上下文压缩与 Session 链摘录

> 同源摘录：本文仅保存上下文压缩后的 Session 链、历史保留和记忆边界片段。
>
> 主文档入口：[OpenClaw vs Hermes：拆解 Hermes Agent 五层架构](../多Agent工程/OpenClaw%20vs%20Hermes：拆解%20Hermes%20Agent%20五层架构.md)
>
> 相关摘录：[Hermes ContextCompressor 压缩流程摘录](<OpenClaw vs Hermes：Hermes ContextCompressor 压缩流程摘录.md>)

## 上下文压缩

上下文压缩有一个很容易被忽略的副作用:**摘要是有损的**。

用户今天聊了一大段,明天回来问"昨天你说的那个函数名叫什么来着",模型在当前 session 里看到的只是摘要,细节可能已经被压缩掉,答不上来了。如果压缩是"就地覆盖"旧对话,那对用户来说就是历史丢了。

Hermes 的做法是，每次上下文压缩时,SessionDB 里做三件事:

1. 结束当前 session,**原始对话完整保留在数据库里,不删**
2. 开一个新 session,把压缩后的摘要作为新 session 的起点
3. 新 session 的 `parent_session_id` 指回旧 session 的 ID

连续压缩几次就会形成一条链:新 session 的 parent 指回上一次的 session,一路能追溯到最初的那一轮对话。

这样设计之后,"省成本"和"不丢历史"两个看起来矛盾的目标,用**分层**各自满足:

- **模型层**(当前 session):只装系统提示词 + 摘要 + 近期对话,token 成本不会随对话无限膨胀,也不会撞到模型上下文窗口的上限
- **数据层**(SQLite):所有 session 的原始消息全部留着,FTS5 索引全文可搜。用户再问"昨天那个函数",`session_search` 工具直接命中老 session 的原文,把片段返给模型,模型就能答得上来

模型看到的是压缩版,数据库存的是完整版。两个目标不是真的矛盾,只是**不该用同一份数据同时扛**。用数据模型把它们分开承载,矛盾就消解了。

但要注意,"能搜到历史"和"Agent 记住了"是两回事。

`session_search` 是按需检索:模型得主动调用这个工具才能拿到旧对话的片段,搜索结果只是当次推理的临时上下文,不会自动写入 MEMORY.md,也不会更新系统提示词里的记忆快照。下次遇到类似问题,模型还得再搜一次。

真正持久的"记忆"只有一条路:模型主动调 `memory` 工具写入,下次新会话启动时才从磁盘加载进快照。

换句话说,**session 链保的是原始数据不丢,记忆系统保的是经验沉淀不丢,两条通道各管各的**。

有了 session 链不代表可以不写记忆，前者是被动存档,后者是主动学习。
