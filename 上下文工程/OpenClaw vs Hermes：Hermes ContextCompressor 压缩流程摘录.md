---
title: "OpenClaw vs Hermes：Hermes ContextCompressor 压缩流程摘录"
source: "https://mp.weixin.qq.com/s/DLZFEsqxRMq0ntilgqpEjA"
author:
  - "[[叶小钗]]"
published:
created: 2026-04-28
description: "一条消息在 Hermes Agent 里经历了什么？万字拆解五层架构"
tags:
  - "clippings"
---
# OpenClaw vs Hermes：Hermes ContextCompressor 压缩流程摘录

> 同源摘录：本文仅保存 `ContextCompressor` 的压缩流程片段。
>
> 主文档入口：[OpenClaw vs Hermes：拆解 Hermes Agent 五层架构](../多Agent工程/OpenClaw%20vs%20Hermes：拆解%20Hermes%20Agent%20五层架构.md)
>
> 相关摘录：[Hermes 上下文压缩与 Session 链摘录](<OpenClaw vs Hermes：Hermes 上下文压缩与 Session 链摘录.md>)

## 上下文压缩

`ContextCompressor` 的压缩流程:

1. **裁旧工具输出**(不调 LLM):替换成 `[Old tool output cleared to save context space]`。很多时候这一步就够降到阈值下。
2. **保护头部**:系统提示词 + 前 3 条消息不动(通常是系统提示词 + 第一条用户消息 + 第一条助手回复,即第一轮完整交换)。
3. **保护尾部按 token 预算**:最近的完整对话不动,预算是两步链式推导出来的,先算压缩触发阈值 `context_length × 0.50`(上下文用掉一半时触发压缩),再从阈值里拿出 20% 给尾部保护(`threshold_tokens × 0.20`)。200K 上下文模型的阈值是 100K,尾部预算 = 100K × 20% = 20K token。源码注释这样写道:"ratio is relative to the threshold, not total context"。**不是按消息数**,一条长代码和一句"好的" token 差 100 倍,按数字算没意义。
4. **中间摘要**:配置里指定的便宜模型做摘要。摘要前拼 `SUMMARY_PREFIX`:**"这是来自前一个上下文窗口的交接"**。暗示这是**另一个助手**留下的笔记,让模型不会把摘要里的旧请求当新指令再执行一遍。
5. **增量更新**:二次压缩在已有摘要上更新,不从头重压。摘要 token 上限 12000,防自己膨胀。

压缩触发时主动调 `_invalidate_system_prompt()` + `_build_system_prompt()` 重建系统提示词,冻结的记忆快照重新生成，加载最新的记忆内容。
