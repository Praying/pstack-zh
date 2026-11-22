---
name: principle-guard-the-context-window
description: "适用于上下文快满的时候：大输出、长文件、反复读取、扇出规划。把大块内容转给子代理，只把摘要留在主线程，别留原始载荷。"
disable-model-invocation: true
---

# Guard the Context Window（守护上下文窗口）

**为什么：** 上下文窗口（context window）溢出会拉低推理质量，制造压缩痕迹，还会让工作停摆。

**模式：**
- **给阶段定规模，给范围设上限。** 限制每个阶段的文件数，设定轮次预算，把机制成本算进去。
