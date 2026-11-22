# TwoPhaseFixed

TLC 到达 56 个状态、142 步。图中画出 10 个状态、12 条箭头。
图中没有画出停留在同一个图状态里的 45 个步。
RMPrepare 步每次只改变 r1 和 r2 中的一个。
RMChooseToAbort 步每次只改变 r1 和 r2 中的一个。
RMRcvAbortMsg 步每次只改变 r1 和 r2 中的一个。
RMRcvCommitMsg 步每次只改变 r1 和 r2 中的一个。

## 成立的规则
TPTypeOK, TCConsistent, NoDeadlock
