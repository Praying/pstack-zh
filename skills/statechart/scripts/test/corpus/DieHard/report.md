# DieHard

TLC 到达 16 个状态、96 步。图中画出 16 个状态、58 条箭头。
有 38 步停留在同一个图状态内，图中不画它们。

## NotSolved 被违反
最短路径有 6 步：
0. big=0 small=0
1. FillBig: big=5 small=0
2. BigToSmall: big=2 small=3
3. EmptySmall: big=2 small=0
4. BigToSmall: big=0 small=2
5. FillBig: big=5 small=2
6. BigToSmall: big=4 small=3

## 成立的规则
TypeOK, NoDeadlock
