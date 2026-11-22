# CounterLoop

TLC 到达 3 个状态、3 步。图中画出 3 个状态、3 条箭头。

## ReachesThree 被违反
TLC 的反例在 tlc-ReachesThree.out 里。它有 3 步：
0. 0
1. Up: 1
2. Up: 2
3. Back: 1
运行永远在 "1" 和 "2" 之间循环。

## 成立的规则
NoDeadlock
