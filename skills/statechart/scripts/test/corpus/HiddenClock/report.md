# HiddenClock

TLC 到达 2 个状态、2 步。图中画出 2 个状态、1 条箭头。
图中没有画出停留在同一个图状态里的 1 个步。
用了 VIEW 之后，TLC 为每一组它算作相同的状态保留一个代表。一条箭头可能指向这个代表，而不是这一步实际到达的状态。

## EndsAtZero 被违反
TLC 的反例在 tlc-EndsAtZero.out 里。它有 1 步：
0. x=0 clock=0
1. Tick: x=1 clock=1
这次运行永远停在 "x=1 clock=1"。
