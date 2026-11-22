# 食谱与坑

值得照抄的提示词（prompt），然后是每个人都会犯一次的错误。换成你自己的路径和完成条件。这些食谱刻意写得不正式。实际动手时大家就是这么敲的，技能（skill）照样读得懂意图。

![她尝着一道做好的菜，机器人照食谱盒做菜，柜台上方钉着写有 /how、/tdd 和 /loop 的卡片。](./images/recipes.jpg)

## 理解一个陌生子系统

```text
use /how first to understand how this initialization works. then use /why to figure out why it broke recently.
```

先懂机制，再看历史。每份技能的报告都会告诉你它搜了哪些来源，这样你就知道答案立足在哪。

## 动代码之前先复述嘈杂的报告

```text
/poteto-mode read this thread. restate the underlying issue in your own words, in plain english. don't change any code yet.
```

误读会在复述里现形，在那里纠它只花一条消息。在代理（agent）说出它自己的判断之前，先把你自己的猜想揣着。`/in-your-own-words` 只要复述这一件事，不走手册（playbook）。

## 发提示词之前先把它磨快

```text
/sharpen write the onboarding doc for the new billing service.
```

代理会点名：专家会做而你没要求的事、他们会用的原材料、他们的方法和工具，以及一个从外部检查结果是否正确的办法。它把你的提示词改写成会要这些东西的样子递回来，不动手做任务。重写读着对了，再发出去。

## 挑选之前先做原型

```text
/poteto-mode prototype a few options for the settings layout. put them behind a switcher and send me screenshots of each.
```

你从能跑起来的东西里挑，不是从描述里挑。代理沿途自己回答自己的布局和时机问题。

## 把定稿的设计变成计划

```text
/poteto-mode turn this design into a plan. small verifiable PRs, each with its own proof.
```

设计定了再开口。计划就是交付物，而且它点名执行要用的手册。

## 给设计要第二意见

```text
ask /arena for a second opinion on this thread and our approach
```

你当前的设计变成若干候选之一，综合结论会告诉你评审团是找到了更好的东西，还是确认了你已有的设计。相比一个推倒重来代价高昂的设计，这点成本不值一提。

## 并行检查互相独立的切片

```text
/swarm check every package under packages/ against its check.sh. one worker per package. one report.
```

每个工作单元（worker）负责一个包。父级等所有切片做完，返回一份 `PASS`、`ISSUES` 或 `BLOCKED` 报告，而不是一堆工作单元的原始输出。

## 用怀疑眼光审一条分支

```text
/interrogate the whole branch, but skeptically. don't change anything yet. no nitpicks unless it's an actual bug or regression in behavior.
```

这些限定词干的是实事。"don't change anything yet" 让它保持只读，吹毛求疵规则提前滤掉噪音，这样经过验证的发现才值得你花时间。

## 通过失败的测试修 bug

```text
/poteto-mode repro the duplicate write first. if there's a cheap test path, /tdd it. then fix and rerun.
```

"if there's a cheap test path" 这句很关键。硬推一个靠脆 mock 撑着的测试，证明力不如跑真实命令，手册可以直说这一点。

## 用证据复现并修复一份报告

```text
/poteto-mode repro this with /verify-<app>. if it repros on main, fix it and show me a video as proof.
```

"if it repros on main" 让运行在 bug 已经消失时早点停。视频让你在读 diff 之前就能检查修复。

## 证明一条测试很难够到的性质

```text
/create-formal-verification-skill the retry queue. a message must never be delivered twice.
```

你得到这些：一条在声明范围内每个情形都被检查的性质，如果代码已经打破它，就有一个用你代码语汇讲的反例，还有一份下一个代理在那段代码有任何改动时都会跑的项目技能。

## 发出去之前审查一个数字

```text
/benchmark-checklist vet this 40% speedup before it goes in the pr description
```

结果只有四种：更快、更慢、无可测差异、无法确定，附上运行次数、区间，以及是什么限制了这个数字。

## 别再纠正同一个错误

```text
/correct agents keep adding new config flags without registering them in the schema
```

修复以架构、类型、lint 或测试的形态落进仓库，让下一个代理犯不了这个错。

## 只问怎么做，不开工

```text
/poteto-help how do i get poteto-mode to stay on every turn?
```

你得到一个答案、一段可以直接发出去的提示词，和一个指向出处的链接。你不发那段提示词，什么都不会跑。

## 你不在时让一次运行保持诚实

```text
im going to bed, keep going autonomously until every fixture passes. do not stop. keep a decision log i can audit in the morning.
```

完整契约在 [通宵那一页](./07-overnight.md)。只要任务和完成条件已经在对话里，短版就能用。

## 给跑偏的运行改道

改道用的提示词就一行：

```text
i said the goal is to repro. i did not ask for a fix yet.
```

```text
apply prove it works. show me the real output, not the build log.
```

```text
/unslop that, no emdashes
```

你很少需要更多字。你需要的是对的名字，而 [原则（principle）页](./08-principles.md) 就是那本词汇表。

## 要一份大白话回复

```text
/bro
```

这就是提示词全文。[`/bro`](../../skills/bro/SKILL.md) 把上一条消息像人跟人说话那样复述一遍，不带术语，更短。当一条回复技术上很全面，你却还是不知道它说了什么时候用。

## 追上你丢下继续跑的会话

```text
/ketchup
```

[`/ketchup`](../../skills/ketchup/SKILL.md) 用大白话告诉你自你上条消息以来变了什么，然后列出每一个等你拍板的决策，附上它的推荐、证据和风险。它报告时什么都不改。

想看而不是读，改跑 [`/ketchup-video`](../../skills/ketchup-video/SKILL.md)。它用一段带字幕的幻灯片短视频演一遍发生了什么，视频下方用文字列出等你拍板的决策。

## 那些坑

- **在提示词里罗列技能。**"use /architect then /arena then /interrogate" 把手册已经排好序的步骤重新排了一遍。说清目标和约束。只有要覆盖默认值时才点名技能。
- **含糊的完成条件。**"make it better" 没给 `/loop` 任何可检查的东西。给一条能通过或失败的命令或产物。
- **一上来就抛出你的原因理论。**你指哪代理就搜哪。先让它复述问题，再分享你的直觉。
- **拿下第一个设计。**只做一次尝试，就把模型想到的第一个形状锁死了。要原型或 `/architect`，从证据里挑。
- **打磨一份抽象计划。**对一份背后没有代码的计划做对抗性审查，只会发明永远不会发生的风险。先用原型定下悬而未决的问题，再审建出来的东西。
- **多个并行代理挤一棵工作树（worktree）。**它们互相覆盖，谁也说不清哪个代理改了什么。把它们当云端代理跑，或者明说 "own worktree per attempt"。
- **还没信任循环就先开循环。**一个验证不了自身工作的循环只会更快产出没人检查的工作。先把验证技能跑通。
- **信任一个没审过的数字。**热缓存或被跳过的代码路径都能伪造加速。数字去任何地方之前先跑 `/benchmark-checklist`。
- **手工纠正同一个错误。**聊天里的一次纠正只帮一次运行。`/correct` 修仓库，让以后的运行不再重犯。
- **拿 `/arena` 做覆盖。**`/arena` 重复同一份设计或代码任务书，然后选一个基底，把最好的部分嫁接上去。`/swarm` 把工作切成切片或已声明的竞赛组别，聚合成一份报告。
- **照单全收每条审查评论。**机器人和人类都在同一张清单里既报真货也报噪音。`/interrogate` 只保留重跑能证明的发现，并列出它丢掉了什么、为什么丢，两个方向你都可以推翻。
- **把 `auto` 当成模型 slug。**`auto` 和 `inherit-parent` 的意思是"省略 model 字段，让子代理（subagent）继承父会话的模型"。[配置](./01-setup.md) 讲了各个角色。
- **凭绿色构建报成功。**构建只能证明它编译得过。要真实命令、流程、存下来的值或 profile，并且要求证据出现在回复里。
- **徒手写 `SKILL.md`。**把它路由过 [Authoring or modifying a skill 手册](../../skills/poteto-mode/playbooks/authoring-a-skill.md)，让校验和审查真正发生。

指南到此为止。如果你是跳着读的，回到 [配置](./01-setup.md) 跑一个真实任务。习惯靠用记住，不靠读记住。

回到 [指南索引](./README.md)。
