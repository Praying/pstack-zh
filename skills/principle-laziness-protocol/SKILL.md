---
name: principle-laziness-protocol
description: "在重构、评估 diff 大小，或想加抽象、加分层、加信号传递时应用。偏向删减，选择能解决问题的最小改动。"
disable-model-invocation: true
---

# Laziness Protocol（懒惰协议）

- **合并决策。**同一个选择不要在多处重复。把它收敛到唯一的事实来源，再把结果作为一个简单标志传下去。
- **让 diff 最小。**做出能解决问题的最小改动。行数更少胜过"优雅"的样板代码。
