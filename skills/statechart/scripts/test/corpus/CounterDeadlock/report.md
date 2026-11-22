# CounterDeadlock

TLC 到达 4 个状态、3 步。图中画出 4 个状态、4 条箭头。
1 个被约束排除的步骤终结了一条规则的路径，图中包含它。TLC 会检查这种步骤之后的规则，但不会真的走这一步。
图中只标记 TLC 报告的死锁。在 CONSTRAINT 或 ACTION_CONSTRAINT 之下，图中没有任何步骤离开的状态仍可能有被约束略去的步骤，TLC 不把它算作死锁。

## NotThree 被违反
最短路径有 1 步：
0. 0
1. Jump: 3

## NoDeadlock 被违反
最短路径有 3 步：
0. 0
1. Inc: 1
2. Inc: 2
3. Inc: 3
