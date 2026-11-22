# CounterJump

TLC 到达 6 个状态、5 步。图中画出 7 个状态、6 条箭头。
1 个约束之外的状态终结了一条规则的路径，图中包含了它。TLC 会在约束之外的状态里检查规则，但不会越过它继续探索。

## NotThree 被违反
最短路径有 1 步：
0. jumps=0 x=0
1. Jump: jumps=1 x=3

## 成立的规则
NoDeadlock
