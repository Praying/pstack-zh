# pstack 指南

pstack 在你停止微观管理代理（agent）的时候效果最好。你描述想要什么，以及怎么算做完。`/poteto-mode` 挑选手册（playbook），按步骤需要运行其他技能（skill），把证据摆给你看。本指南用贴近真实的提示词（prompt）教你养成这个习惯。

你将学到：

1. [配置 pstack](./01-setup.md)。安装插件并挑选你的模型。
2. [用 `/poteto-mode` 分派工作](./02-poteto-mode.md)。给它一个目标，看它挑中哪本手册。
3. [理解代码](./03-understand.md)。先做一次只读调查，然后在动任何代码之前用 `/how`、`/why`、`/teach`、`/recall`。
4. [设计改动](./04-design.md)。在代码把形状锁死之前，用 `/architect`、`/arena`、`/swarm`、`/interrogate`，做原型和计划。
5. [构建并清理改动](./05-build-and-clean.md)。构建类手册、`/tdd`、`/unslop` 和 `/no-comments`。
6. [验证并交付](./06-verify-and-ship.md)。在真实应用上证明行为，对测试很难碰到的部分做形式化验证，用 `/benchmark-checklist` 审查数字，然后开出聚焦的 PR 一路推到合并。
7. [睡觉时让工作继续跑](./07-overnight.md)。开循环之前先建立信任，一份通宵契约，一份你可审计的决策日志，还有能扩展到单个代理之外的 Projects 与自动化。
8. [用原则（principle）名字掌舵](./08-principles.md)。24 个能在任务中途给代理改道的名字。
9. [定制成你自己的](./09-make-it-yours.md)。属于你自己的模式，用 `/correct` 治重复犯的错，以及怎么测试一次技能改动。
10. [食谱与坑](./10-recipes-and-pitfalls.md)。可以直接抄的提示词，和可以绕开的错误。

第一次请按顺序读。之后每页都能单独读。

卡住了，或拿不准哪个技能合适时，把你的问题输给 [`/poteto-help`](../../skills/poteto-help/SKILL.md)：

```text
/poteto-help which skill should i use to review this branch?
```

它给出回答，递给你一段可以直接发出去的提示词，并链接答案出处的技能或指南页。它不开工，因为 pstack 每次运行都要花真金白银的 token，所以你准备好了再发那段提示词。只有你输入它才会运行。

## 如果你只记住一件事

用你自己的话，给代理一个目标和一个检查它的办法：

```text
/poteto-mode the export writes duplicate rows when a retry lands mid-run. repro first, then fix and verify.
```

你不用点名手册，也不用列技能。“先复现”加上一个可检查的结果，就是 `/poteto-mode` 需要的全部路由信号。它会匹配到 Bug fix 手册，把步骤抄进待办清单，在每步触发时叫来对的技能。

下一章：[配置 pstack](./01-setup.md)。
