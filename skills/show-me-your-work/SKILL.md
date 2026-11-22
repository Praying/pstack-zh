---
name: show-me-your-work
description: "为长时间运行或无人值守的工作保留一份可评审的决策轨迹。一份 TSV 日志，每个决策一行（做了什么、为什么、证据、结果）。默认留在本地。评审需要凭轨迹才能信任结果时才提交。用于 /show-me-your-work、自主运行或多阶段运行，以及人走开后回来评审的工作。"
disable-model-invocation: true
---

# 把你的工作亮出来

始终只保留一份正本日志。

## 格式

单个 TSV 文件，每个决策一行。单元格保持单行。证据是指针，不是散文。

把 `references/decision-log-template.tsv`（表头行）复制过来，开一份干净的日志。各列：

- **ts.** ISO8601 时间戳。
- **phase.** 所属阶段或工作流。
- **decision.** 选了什么或做了什么，一行写完。
- **why.** 用大白话写原因。如果做决定是受某条原则（principle）驱动，就照直说，不要写成行话标签。
- **evidence.** 能证明它的链接或路径：提交 SHA、PR 编号、`file:line`，或者产物、追踪、截图的路径。绝不写成一段话。
- **result.** 结果或谓词状态：`tests green`、`reverted`、`pixel-diff 0`、`INCONCLUSIVE`、`open`。

下面是一个例子。用大白话写成，评审一眼就能读。

```
ts	phase	decision	why	evidence	result
2026-05-24T09:02:00Z	frame	counted the work first, about 100 components and roughly 75 hours	wanted to know the size before starting a long run	commit 3a9f1c2	found 5 things to sort out before starting
2026-05-24T09:40:00Z	harness	took screenshots of the old version before changing anything	so we can compare old against new and catch any visual change	scripts/snapshot.sh, baseline/	saved 120 reference screenshots
2026-05-24T11:15:00Z	widget	moved the widget styles over without changing how it looks	keep the change small and the result identical	commit 7c21e0a, pixel-diff 0	looks identical, tests pass
2026-05-24T12:30:00Z	widget	threw out a helper's work because its screenshots were blank	checked the real files instead of trusting its summary	worktree reset	reverted, tightened the instructions for next time
```

## 记录一行

每条记录写得像告诉队友你做了什么。大白话，具体动作，不写 AI 腔和抽象行话（**unslop** 技能同样适用于日志文字）。

用辅助脚本 `scripts/log.sh <logfile> <phase> <decision> <why> <evidence> <result>`。它会盖上 `ts` 时间戳，首次使用时写入表头，清理游离的制表符和换行，还会给任何以 `=`、`+`、`-` 或 `@` 开头的单元格加上单引号前缀。用裸的 `printf` 追加一行也行，但如果单元格来自生成文本或用户提供的文本，同样要当心这些字节。

记录决策点和检查点，不是每个动作都记。选定的分岔、带验证结果完成的单元、带触发原因的转向或回退、暴露出来的阻塞、修好的检查关卡（gate）。对于循环运行，每次迭代一行。琐碎和不言自明的跳过不记。

一次运行就是一次代理（agent）对话，包括它的后续轮次和对它的任何总结。接续、替换代理或新对话都会开启一次新的运行。当一次运行向已有记录的日志追加时，它的第一行 phase 为 `start`；在另一个运行的 `start` 行之后，它的第一行也是如此。所以在后续轮次回到日志的运行，要先读日志末尾几行，看看期间有没有别的运行写过。`start` 行要指明它之前、本次运行没有写过的那些行的 `ts` 范围，evidence 则指明本次运行，例如它的代理 id。phase `start` 不作他用。

## 日志放在哪

默认情况下日志是工作产物，不提交。放在工作目录的 `decisions.tsv`，多项工作并行时放 `.audit/<task-slug>.tsv`，并且把它排除在 git 之外。

只有当工作规模大到评审需要凭轨迹才能信任结果时，才提交它。

## 规则

- 只追加。判断错了就写新行覆盖它。绝不编辑或删除历史。
- 证据优先用已提交脚本产出的，而不是手工做的一次性产物（**encode-lessons-in-structure** 原则技能）。

## 对照转录审计日志

运行收尾、交回结果之前，核对日志说的是不是真话。读取本次运行的转录（transcript），它在活动工作区的 `agent-transcripts/` 目录下（系统提示词命名了该路径）。不要用 glob 扫 `~/.cursor/projects/*/`。那会读到无关的私人对话。把本次运行的行对照实际发生的事情走一遍。每一段从本次运行的一个 `start` 行开始，若日志由本次运行创建则从第一行开始，到另一个运行的下一个 `start` 行为止：

- 确认每一行都对应真实的决策或动作。
- 确认每行的 evidence 都能解析到实际内容，并且证明该行的说法。
- 影响了工作走向却没有记录的分岔、转向或半途放弃的方案，就是缺口。补上它。

改日志，不改故事。审计绝不编辑或删除任何行，哪怕是编造出来的行。当某一行既没记录真实决策也没记录真实动作，或者它的说法或 evidence 有误，就加一行覆盖它，写清实际发生了什么，并给出能解析的指针。这次审计不检查本次运行各段之外的行。如果本次运行自己的工作证明其中某行有错，就照错误判断一样用新行覆盖它。

## 轨迹的跨模型评审

交回之前，派一个子代理（subagent）来审，它的模型要与做这份工作的模型来自不同厂商。自审不能替代它。子代理读审计轨迹和本次运行的转录，标出用户该留意的地方。不是重做工作，而是扫描哪里次优或有风险。按任务为每个子代理挑选推理等级，见 **poteto-mode** 技能（skill）的子代理（Subagents）一节。

- 记录的决策证据薄弱或干脆没有证据。
- 转录里被跳过的验证步骤，或者没有凭据的声称。
- 事后看有风险的选择（动手过早、范围蔓延、粉饰症状）。
- 用户随便浏览就会漏掉的缺口。

产出轨迹的运行，其每条回复都以"注意"一节收尾。第一行单独写评审用的模型（`reviewed by <model>`），然后逐条列出标记，每条指向具体的行或时刻。"没有标记"是合法取值。模型名不是。

## 评审轨迹

从头读到尾，跟着证据指针走，抽查。GitHub 会把已提交的 TSV 渲染成表格。`column -s$'\t' -t decisions.tsv` 在终端里渲染它。

## 编写本技能

其他技能把审计轨迹路由到这里，而不是自创一套。按名字引用它，让格式归它所有。不要复述各列。
