# CounterAction

TLC 到达 5 个状态、4 步。图中画出 5 个状态、5 条箭头。
1 个被约束排除的步骤终结了一条规则的路径，图中包含它。TLC 会检查这种步骤之后的规则，但不会真的走这一步。

## NotThree 被违反
最短路径有 1 步：
0. 0
1. Jump: 3

## NotFour 被违反
最短路径有 4 步：
0. 0
1. Inc: 1
2. Inc: 2
3. Inc: 3
4. Inc: 4

## 成立的规则
NoDeadlock
