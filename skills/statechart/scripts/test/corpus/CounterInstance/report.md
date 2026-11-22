# CounterInstance

TLC 到达 3 个状态、3 步。图中画出 3 个状态、3 条箭头。

## ReachesThree 被违反
TLC 的反例在 tlc-ReachesThree.out 里。它有 3 步：
0. 0
1. C!Up: 1
2. C!Up: 2
3. C!Back: 1
这次运行永远循环经过 "1" 和 "2"。

## 成立的规则
NoDeadlock
