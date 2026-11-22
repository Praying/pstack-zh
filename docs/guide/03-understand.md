# 改代码之前先理解代码

改动你不理解的代码，微妙的回归就是这样上线的。这对代理（agent）和对你一样成立。代理通常败在两种地方。一是读错你要什么，二是没有把活干对所需的上下文。[提示词里该写什么](./02-poteto-mode.md#what-goes-in-a-prompt) 治第一种。这一页治第二种。

pstack 给你四条入口。`/how` 讲代码现在做什么。`/why` 挖出它长成这样的缘由。`/teach` 把两者融成一份讲解。`/recall` 重建你自己在某个主题上的近期上下文。每一个都会让代理用你能核对的话解释自己。代理可能比你更懂这份代码，你就这样监督它。

![一位侦探用放大镜研究机器图纸，机器人取来案卷，她身后的证据板在 /how 和 /why 之下串联线索。](./images/understanding.jpg)

## 从只读调查开始

原因不明时，要调查结果，不要修复：

```text
/poteto-mode investigate why background jobs time out every few hours. give me what we know, what data you used, and your best hypotheses. don't change any code yet.
```

"先别改任何代码"把这一发路由到 [Investigation 手册（playbook）](../../skills/poteto-mode/playbooks/investigation.md)。它追踪代码，对动机类问题补上 `/why`，带回一份有引用的解释。要在选项之间做选择时，它带回一份附权衡表的建议。问一句 "你用了哪些数据"，代理就会把证据和猜测分开摆。调查结果指向某个修复时，把修复当作新任务开。

## 用 `/how` 追踪行为

```text
/how do we dedupe notifications? is there an n+1 when we look up subscribers?
```

问你真正想问的问题。[`/how`](../../skills/how/SKILL.md) 读代码，回答的水准相当于一位资深工程师带你上手这个子系统，讲清运行时的流程、关键类型和不显然的部分。它会起一个只读子代理（subagent）去探索代码并写出讲解。

## 用 `/why` 挖历史

```text
/why was the retry limit set to five? does the reason still hold?
```

[`/why`](../../skills/why/SKILL.md) 从记录里找答案，不从代码里。它读引入这段代码的提交与 PR 讨论，以及之后的每次改动，那条线索太薄时再看别的来源。回答引用作者写过的话，把代理自己的推断标成推断，并说出记录回答不了的部分，因为"没人写下为什么"本身就是一个答案。

两者天然可组合。怀疑历史能解释这团乱麻时，`do why first then how` 就是条好提示词（prompt）。

## 用 `/teach` 真正弄懂它

```text
/teach me how this PR changes retries. convince me it fixes the cause and not the symptom.
```

[`/teach`](../../skills/teach/SKILL.md) 用在摘要不够的时候。它跑 `/how` 和 `/why`，小改动也许只跑一个，把发现织成一份平实的讲解，一张图一张图搭起来。"说服我"这个提法值得复用。它把讲解变成一场你能检验的论证，而不是走马观花。

它对代理自己的选择也好使：

```text
/teach me why you implemented it this way and not with a queue. what did you trade off, and why?
```

教，对代理和对你一样有好处。一个必须解释自己工作的代理，就得读代码，给每个论断配上证据，而不是自信满满说一句就走。

## 用 `/recall` 重建你自己的上下文

```text
/recall catch me up on the export work from last week
```

[`/recall`](../../skills/recall/SKILL.md) 挖你自己的近期聊天，加上共享记录（issue、旧修复、还在报的错），带回一份简报，讲清现状和下一步。你的旧聊天里有新代理没有的上下文，所以旧主题上的新工作，先加载它，再递上新输入：

```text
/recall my work on the virtualized list from yesterday, then read this bug report.
```

想续上某一个具体对话，那该用下面的 Session pickup 手册，不是 `/recall`。

## 用 Session pickup 接手之前的工作

另一个代理（或上周的你）把一个分支丢在半空时：

```text
/poteto-mode take over this branch. read the decision log, figure out what's done, and continue from there. don't redo finished work.
```

[Session pickup 手册](../../skills/poteto-mode/playbooks/session-pickup.md) 把之前的轨迹当权威。它重建分支状态和当时的决定，指出续跑点，对照原始目标验证继承来的论断，而不是把一切从头推导一遍。

**坑：**别因为"代理反正会读代码"就跳过这一页的技能（skill）。没建起行为模型就动手改的代理，往往在第一个看起来像的地方治症状。先跑 `/how` 比再吃一个 bug 便宜。

下一章：[设计改动](./04-design.md)。
