# CounterRules

TLC 到达 2 个状态、1 步。图中画出 4 个状态、3 条箭头。
2 个约束之外的状态各自终结了一条规则的路径，图中包含它们。TLC 会检查约束之外状态里的规则，但不会继续向后探索。

## NotThree 被违反
最短路径有 1 步：
0. 0
1. M!Jump: 3

## AtMostTwo 被违反
最短路径有 1 步：
0. 0
1. M!Jump: 3

## NotTwo 被违反
最短路径有 2 步：
0. 0
1. Step: 1
2. Step: 2

## 成立的规则
NoDeadlock
