# 定制成你自己的

poteto-mode 是一个人的风格。它底下的手册（playbook）、路由和模型角色，配你的风格照样好用。这一页讲这些内容：生成一个个人模式、从一次会话里捕获教训、修好仓库让代理（agent）不再重复犯错、写一份聚焦的技能（skill），以及在信任一次技能改动之前先测它。

起步要比你想的更小。第一天不需要多少技能，甚至不需要整个插件。提示词（prompt）写得平实，盯着代理在哪里失败，同样的失败出现两次，就加一份技能或一条检查。

## 用 `/automate-me` 生成你自己的模式

```text
/automate-me
```

你不用描述自己的风格，因为 [`/automate-me`](../../skills/automate-me/SKILL.md) 会从你的历史里把它读出来。它在当前工作区挖你最近的转录（transcript），找反复出现的偏好，覆盖你喜欢的回复方式、委派、验证、代码、文字和流程，然后问你哪些模式真的是你。它通过 Cursor 内置的 `create-skill` 流程起草 `.cursor/skills/<your-name>-mode/SKILL.md`，把草稿过一遍 [`/unslop`](../../skills/unslop/SKILL.md)，再从一棵工作树（worktree）开出 PR，让你像审任何改动一样审它。

习惯漂移了就再跑一次：

```text
/automate-me update my mode skill with everything since its last edit
```

更新模式只挖技能上次改动以来的历史。你没反驳过的规则保留，有新证据的规则修订，只给新模式加小节。

## 用 `/reflect` 捕获一次会话的教训

在一个教会你什么的任务刚结束时，跑：

```text
/reflect that took way too long. capture what we learned so the next run doesn't repeat it.
```

[`/reflect`](../../skills/reflect/SKILL.md) 读转录，只保留会改变未来代理工作的教训，并把提案分进 `Accepted`、`Rejected` 和 `Backlog`。任何技能改动之前它都等你批准。只有当一条提案会改变未来决策时才批准它。一次怪异的会话是轶事，不是规则。

<a id="fix-the-environment-with-correct"></a>
## 用 `/correct` 修好环境

当你为同一个错误一遍遍纠正代理时，这个修复属于仓库，不属于你下一条提示词。按牢靠程度给选项排序：

1. 用架构或更好的数据结构让这个错误不可能发生。
2. 用类型拦住它，或者用一条 lint 或 CI 检查拦住它，报错信息要点名怎么修。
3. 用测试抓住它。
4. 把它写成文档或代理规则。代理跳过一条规则时什么都不会失败，所以这条排最后。

人工审查不在清单上。一个必须在每个 PR 上抓住同一个错误的审查者，正是这里要修的问题。[`/correct`](../../skills/correct/SKILL.md) 干这个活：

```text
/correct agents keep calling the database client directly instead of going through the repository layer
```

它读最近的提交、回退、审查评论，以及解释变通做法的注释，然后把错误归成类。一个类出现两次才算数。它按频率从高到低修类，每个类一个提交，修在管用的最高层级上，并证明每条新检查会在一个真实的过往错误上失败。它还在代理指令文件里维护一张表，把每条规则和执行它的东西配对，这样没有任何东西执行的规则就会以复发的形式冒出来。回复里列出每个类，附上它的证据、选定的层级，以及更高层级为什么不行。

不带参数跑它，它就自己从历史里找出这些类。`/reflect` 和 `/correct` 分工。`/reflect` 从一次会话改进技能。`/correct` 改仓库，让一个错误类再也回不来。修复是一条新边界时，搭配 `/architect`。[并行跑多个 Project](./07-overnight.md#run-many-projects-in-parallel) 里有一条提示词，能在整个仓库上把两件事都做了。

## 写一份聚焦的技能

你已经知道自己想捕获的工作流时：

```text
/poteto-mode write a skill for verifying database migrations in this repo
```

写一份技能对上的是 [Authoring or modifying a skill 手册](../../skills/poteto-mode/playbooks/authoring-a-skill.md)，它路由到 Cursor 内置的 `create-skill`，校验 frontmatter 和链接，再通过 Opening a PR 手册把结果交付出去。面向代理的文字比面向人类的文字门槛更高，因为一句没用的话会变成某个未来代理照做的指令。让手册替你守住这条线，别徒手写 `SKILL.md`。

两种特殊情况有自己的生成器。必须驱动你的应用并证明行为的技能是验证技能，改用 [`/create-verification-skill`](../../skills/create-verification-skill/SKILL.md) 和 [`/maintain-verification-skill`](../../skills/maintain-verification-skill/SKILL.md)。用模型检查器、证明或别的验证器检查代码性质的技能是形式化验证技能，改用 [`/create-formal-verification-skill`](../../skills/create-formal-verification-skill/SKILL.md) 和 [`/maintain-formal-verification-skill`](../../skills/maintain-formal-verification-skill/SKILL.md)。[验证并交付](./06-verify-and-ship.md#prove-a-property-with-a-formal-verification-skill) 两类都讲。

## 用 `/technical-writing` 按标准写文档

技能不是你交付的唯一文字。对文档、RFC、readme、PR 描述和提交信息：

```text
/technical-writing review the readme changes
```

[`/technical-writing`](../../skills/technical-writing/SKILL.md) 施行一套分层标准。它的目标是让一个疲惫的工程师读第一遍就能懂的文字。它先定文档的模式（教程、操作指南、参考或讲解），然后逐句处理：谁做什么，一句话一个意思，没有两种读法。用它审查你或代理刚写的东西，或者要文档时先点它的名。

## 盲测一次技能改动

一次技能编辑影响之后每一次会话，所以要像对待实验一样测它。收养别人的技能也一样。留下它之前，先检查它有没有让你的活干得更好。

```text
/poteto-mode run the eval playbook on this skill change. same task for both variants, candidates stay blind.
```

当一份技能老是落空，而你知道它该做什么时，在一个任务里又改又测：

```text
/poteto-mode update the review skill so it flags missing migrations, and eval the change.
```

开口先要评测（eval），能让编辑保持诚实。从一次糟糕会话写出的修复，往往过拟合那场会话，编辑一多技能就漂移。评测在它出门之前抓住漂移。

[Eval 手册](../../skills/poteto-mode/playbooks/eval.md) 围绕一种失败模式打造，观察者效应。一个知道自己正被评测的代理行为会不一样。所以候选代理拿到的是看起来很自然的任务，在消过毒的目录里，见不到 "eval" 或 "candidate" 这两个词，也见不到彼此的存在。一个评审（judge）在中立标签下给所有输出打分，是否遵循链条则看每个候选实际读了哪些文件来评分，而不是看它自称读了什么。

接受裁定之前，把每份输出亲自读一遍。不同意评审的结论时，先怀疑评分标准，再怀疑你的判断。

## 用 `/make-bot-ui` 造一个机器人 UI

有一份场景技能是给 Grok Bot 用户的。[`/make-bot-ui`](../../skills/make-bot-ui/SKILL.md) 造一个小页面，页面上的按钮通过一个 webhook routine 唤醒机器人。比如你可以刷过一条审查队列，每刷一下就让机器人处理那一项。你机器上运行的服务器持有 webhook 的发送方密钥，所以密钥永远不会到浏览器或聊天那边。这份技能还讲怎么用 Tailscale 把页面暴露出去。

**坑：**别因为技能行为不对于是任务干到一半去改它。在它自己的 PR 里修好，让任务继续走。混进功能工作里一起交付的技能编辑，审查看不见它，也没法评测。

下一章：[食谱与坑](./10-recipes-and-pitfalls.md)。
