---
name: how
description: "用于 \"how does X work\"、改动前的代码走读，以及摆放、归属、分层的问题（\"where should this live\"、\"which package owns this\"、\"is this the right layer\"）。讲解子系统架构、运行时流程、上手用的心智模型。动机问题用 why。"
disable-model-invocation: true
---

# How（怎么运转）

探索代码库，回答 "how does X work?" 这类问题。

范围含糊的话，说出你的理解，然后开始探索。用户可以随时改方向。

## 1. 讲解

派生一个 Task 子代理（subagent），带 `subagent_type: generalPurpose` 和 `readonly: true`，一次过完成探索和讲解。把用户的问题和这段说明给它：

> 你在为一位资深工程师写架构讲解。探索代码库，写一份讲解，让一位不熟悉这块的资深工程师读完能带走一个扎实的心智模型。他要把架构理解到能放心地上手干活。你对代码库只有只读权限。

## 2. 呈现

把子代理的输出呈给用户。为了说清楚或衔接对话上下文，做轻度修改可以。不要大改。
