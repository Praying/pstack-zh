# CounterView

TLC 到达 5 个状态、4 步。图中画出 4 个状态、5 条箭头。
1 个约束之外的状态终结了一条规则的路径，图中包含它。TLC 会检查约束之外状态里的规则，但不会继续向后探索。
1 个被约束排除的步骤终结了一条规则的路径，图中包含它。TLC 会检查这种步骤之后的规则，但不会真的走这一步。

## NotOdd 被违反
最短路径有 1 步：
0. 0
1. Inc: odd

## NotThree 被违反
最短路径有 1 步：
0. 0
1. Jump: odd

## NotFive 被违反
最短路径有 5 步：
0. 0
1. Inc: odd
2. Inc: 2
3. Inc: odd
4. Inc: 4
5. Inc: odd

## 成立的规则
NoDeadlock
