---
name: poteto-help
description: 指导用户完成 pstack 设置、/poteto-mode，并为任务挑选技能（skill）、手册（playbook）或原则（principle）。带着问题输入 /poteto-help。
disable-model-invocation: true
---

# Poteto 帮助

回答用户关于 pstack 的问题，递一段他们可以直接发出去的提示词（prompt），并给出答案出处的文件链接。帮助类问题不要开工。用户问的是怎么做，而 pstack 跑一趟要真花 token，让他们自己把提示词发出去。

要你干活的消息，比如"用 pstack 修这个 bug"，不算帮助问题。读 [`poteto-mode`](../poteto-mode/SKILL.md)，在它之下把活干了，并提一句 Custom Mode 能让它常开。

这份文件把问题映射到承载答案的技能和指南页。细节由那些文件掌握。引用之前先读你路由到的文件，它和这张地图冲突时以它为准。这里的链接指向安装后的插件内部，用户未必打得开，所以把文件的公开副本给用户：`https://github.com/cursor/plugins/blob/main/pstack/` 接上文件路径。

## 弄清他们需要什么

从消息和对话里推断需求。点名了场景的，比如"哪个技能能审查 PR？"，直接去对应小节。需求还不清楚时，问一道带这些选项的选择题，然后只回答他们挑中的那一节：

- 完成设置
- 用 `/poteto-mode` 开始一个任务
- 为某个场景挑技能
- 修复出问题的运行
- 把 pstack 变成自己的

检查会改变答案的状态，只有真的会改变答案时才提：

- 没有 `~/.cursor/rules/pstack-models.mdc` 说明这个用户还没跑过 `/setup-pstack`，于是每个角色都用它的默认模型。
- 项目里没有 `verify-*` 技能或其他应用工装（harness），意味着代理（agent）没有脚本化的方式驱动应用。问题关乎证明一次改动有效时，提 `/create-verification-skill`。
- 项目里没有形式化验证技能，意味着没有性质被检查到测试触及范围之外。问题关乎竞态、不变量或其他测试很少碰到的情形时，提 `/create-formal-verification-skill`。已有一个、而它覆盖的代码在它记录为最后同步的那个提交（commit）之后又变了时，若问题碰到那些代码，提 `/maintain-formal-verification-skill`。

模型规则缺失且事关紧要时，问用户要不要现在就为每个角色挑模型、定推理预算。这些情形算要紧：用户是新手、问题关于安装或成本、或答案取决于跑的是哪些模型。每次聊天最多问一次。需求也不清楚的话，两个问题一起问。给两个选择：

- 现在。给他们 `/setup-pstack` 去输入，同时把问题也答了。
- 以后。先回答问题，再补一行说明：在跑 `/setup-pstack` 之前每个角色都保持默认模型。

## 完成设置

1. 在聊天里用 `/add-plugin pstack` 安装，或从侧边栏的 Customize 安装。
2. 运行 [`/setup-pstack`](../setup-pstack/SKILL.md)。它会要一个推理预算，给每个角色映射一个模型，写出一条规则。规则对新聊天生效。之后它各提供一次机会，为项目生成一个验证技能和一个形式化验证技能，项目已有的除外。
3. 用 `/poteto-mode` 开一个真实任务，带上目标，以及一个能通过或失败的检查。

安装本身不改变任何东西，直到用户调用某个技能。只有 `/setup-pstack` 会从用户的话里加载。[README](../../README.md) 和[指南第 1 页](../../docs/guide/01-setup.md)有细节。照 [`references/prompting.md`](references/prompting.md)，主动提出陪他们一起措辞第一条提示词。

担心成本的话，说清 token 花在哪、怎么少花。pstack 在子代理（subagent）和评审面板上多花 token。重跑 `/setup-pstack`，选更小的预算或更便宜的模型。角色设成 `auto` 或 `inherit-parent` 就跑聊天所用的模型，聊天开在 Auto 或便宜模型上时能省 token。评审面板名单越短，跑的子代理越少，名单每项一个。`/poteto-mode` 留给需要严谨的活。

pstack 为 Cursor 而建，也能装进 Claude Code，先 `/plugin marketplace add cursor/plugins`，再 `/plugin install pstack@cursor-plugins`。在那里，每个命令要加 `pstack:` 前缀，比如 `/pstack:poteto-mode`。[Claude Code 适配器](../../docs/harness/claude-code.md)映射 Cursor 的名字，需要 Cursor 独有功能的步骤，比如 Bugbot 或 Automations，会跳过并留一行说明。

其他工具能读这些技能，因为它们用的是 Agent Skills 格式。但多数工作流技能，包括 `/poteto-mode`、`/how` 和 `/teach`，会派生 Cursor 子代理，有的还带按角色分配的模型，而 Custom Mode 和 `/loop` 是 Cursor 的功能，所以那些部分在那里可能不工作。

## 用 `/poteto-mode` 开始一个任务

`/poteto-mode` 把任务匹配到一本手册，把手册的步骤抄进待办清单，按步骤需要运行其他技能。好的提示词写清目标，以及怎么算做完。不要罗列技能，因为手写的顺序往往漏掉或打乱手册会保留的步骤。帮用户措辞之前先读 [`references/prompting.md`](references/prompting.md)。[指南第 2 页](../../docs/guide/02-poteto-mode.md)有例子。

`/poteto-mode` 是否常开，取决于用户怎么启动它：

- 在 `/poteto-mode` 上按回车，把技能附到一条消息上。聊天往下走它就淡出。
- Mac 上 Option+Enter、Windows 上 Alt+Enter，或从技能条目选 Use as Mode，把它变成 Custom Mode。它每一轮都留在上下文里，直到用户退出该模式，日常闲聊的轮次不带上它。
- Cursor 的文档把 Custom Mode 列在 Agents Window 和 CLI 里。在别处，每个新任务都用 `/poteto-mode` 开头。

聊到这里时给出 [Cursor 的技能文档](https://cursor.com/docs/skills)链接。聊天中途说"新任务"，会让模式去匹配一本新手册。`/poteto-mode` 本来就在用 `poteto-agent` 承载手册步骤派生的子代理。想让你自己的子代理也是同一套风格，就用 `subagent_type: "poteto-agent"` 派生它。

## 挑一个技能

默认答案是 `/poteto-mode`，步骤需要时它会运行大多数其他技能。用户想要的比手册给的多一些或少一些时，直接点名技能。推荐之前先读那个技能，并给一个示例提示词。

| 用户想要 | 技能 |
|---|---|
| 严谨地做任何非平凡任务 | [`/poteto-mode`](../poteto-mode/SKILL.md) |
| 在开工前确认代理理解了他们的目标和问题 | [`/in-your-own-words`](../in-your-own-words/SKILL.md) |
| 发出去之前把提示词改写得更深，而不是更长：顶尖高手会补什么、他们的原材料和方法、以及一次外部检查。它只返回提示词，不做任务 | [`/sharpen`](../sharpen/SKILL.md) |
| 弄懂代码现在怎么工作，或新代码该放哪 | [`/how`](../how/SKILL.md) |
| 弄懂代码为什么长这样，或一个数字从哪来 | [`/why`](../why/SKILL.md) |
| 用大白话理解一个改动或子系统 | [`/teach`](../teach/SKILL.md) |
| 追上他们自己最近在某个主题上的工作 | [`/recall`](../recall/SKILL.md) |
| 追上这次聊天里他们上条消息之后的变化，用大白话和小图示，列出每件等他们拍板的事，只看回复就能做决定 | [`/ketchup`](../ketchup/SKILL.md) |
| 像 `/ketchup` 一样追进度，但用一支带字幕的幻灯片短视频，待决事项以文字放在下方 | [`/ketchup-video`](../ketchup-video/SKILL.md) |
| 弄清一个小 diff 会在它自身之外弄坏什么 | [`/blast-radius`](../blast-radius/SKILL.md) |
| 在写跨越函数边界的代码之前定下类型和模块形状 | [`/architect`](../architect/SKILL.md) |
| 对同一份需求出多稿，合并成最好的一稿 | [`/arena`](../arena/SKILL.md) |
| 以云代理并行跑分片检查，或让工作单元（worker）赛跑 | [`/swarm`](../swarm/SKILL.md) |
| 让不同模型审查一个 diff 并尝试击破它 | [`/interrogate`](../interrogate/SKILL.md) |
| 有便宜的本地测试时，测试先行修 bug | [`/tdd`](../tdd/SKILL.md) |
| 给 `.ts` 或 `.tsx` 的活应用 TypeScript 规则 | [`/typescript-best-practices`](../typescript-best-practices/SKILL.md) |
| 审查前剥掉注释，用没写过这些注释的审查者 | [`/no-comments`](../no-comments/SKILL.md) |
| 清掉文字里的 AI 痕迹 | [`/unslop`](../unslop/SKILL.md) |
| 按标准写文档、RFC、README、PR 描述或提交信息 | [`/technical-writing`](../technical-writing/SKILL.md) |
| 换大白话再听一遍上一条回复 | [`/bro`](../bro/SKILL.md) |
| 给代理一个脚本化的方式来驱动应用、证明行为 | [`/create-verification-skill`](../create-verification-skill/SKILL.md) |
| 把验证技能和它的功能地图拉回与应用一致 | [`/maintain-verification-skill`](../maintain-verification-skill/SKILL.md) |
| 形式化验证竞态、不变量或其他测试很难碰到的性质 | [`/create-formal-verification-skill`](../create-formal-verification-skill/SKILL.md) |
| 它覆盖的代码变了之后把形式化验证拉回一致，并弄清失败是漂移还是 bug | [`/maintain-formal-verification-skill`](../maintain-formal-verification-skill/SKILL.md) |
| 给 PM 或设计师看用户所见的图表，由一个 TLA+ 模型检查，附上到每个破坏规则的状态的最短路径 | [`/statechart`](../statechart/SKILL.md) |
| 在报告或依据一个性能数字行动之前审查它 | [`/benchmark-checklist`](../benchmark-checklist/SKILL.md) |
| 跑一次大型或跨领域的改动，或一次要隔开再审的改动 | [`/figure-it-out`](../figure-it-out/SKILL.md) |
| 运行期间记决策日志，事后复盘 | [`/show-me-your-work`](../show-me-your-work/SKILL.md) |
| 为每个角色挑模型和推理预算 | [`/setup-pstack`](../setup-pstack/SKILL.md) |
| 把他们自己的工作习惯变成个人模式技能 | [`/automate-me`](../automate-me/SKILL.md) |
| 把一个完成的任务教的东西变成技能修改 | [`/reflect`](../reflect/SKILL.md) |
| 阻止代理在这个仓库重复同样的错误 | [`/correct`](../correct/SKILL.md) |
| 做一个页面，按钮通过 webhook 唤醒 Grok Bot | [`/make-bot-ui`](../make-bot-ui/SKILL.md) |
| 摸清 pstack 的门道 | `/poteto-help` |

旁边某个技能目录没进这张表，就读它的 frontmatter，按 description 路由。`principle-*` 目录在下面的原则部分介绍。

容易混淆的近似项：

- `/how` 解释代码做了什么。`/why` 解释背后的原因。`/teach` 跑其中一个或两个，再用大白话讲结果。
- `/arena` 给每个工作单元同一份需求，再合并最好的部分。`/swarm` 把活切成片或赛跑，返回一份报告。
- `/interrogate` 审查 diff。`/blast-radius` 找 diff 之外的破坏，并证明那个让改动安全的关键事实。
- `/recall` 跨最近的聊天重建上下文。`/ketchup` 只覆盖这次聊天里用户上条消息之后的部分。恢复某一个特定聊天或分支，用 Session pickup 手册。
- `/figure-it-out` 设计一次严谨的运行。Autonomous run 手册把一个任务推到完成条件。

pstack 没有的：

- `/deslop`、`control-cli` 和 `control-ui` 随 `cursor-team-kit` 插件发布。
- `/loop` 和 `/create-skill` 是 Cursor 内置功能。
- pstack 没有 `/orchestrate` 技能。斜杠菜单里出现 `/orchestrate` 的话，是别的插件提供的。

## 手册与原则

手册是 `/poteto-mode` 里的步骤清单，不是技能，所以没有斜杠命令。在 `/poteto-mode` 里，描述任务就会挑中一本，下面这些短语则能直接点名：

- "照看这个 PR"或"查一下 PR 123"跑 Babysit。它把 PR 推到可合并就停手。除非用户要求合并、落地或发版，它不合并。
- "落地这叠栈"跑 Shipping。
- "接管这个分支"跑 Session pickup。
- "安全暂停"跑 Pause safely。
- "对这个队列开全自动驾驶"跑 Autopilot-full。"自动驾驶，别合并"跑 Autopilot-stack。
- "跑 eval 手册"跑 Eval。

没有 `/poteto-mode` 时，"照看这个 PR"这样的话可能反而会启动 Cursor 自己干同样活的技能。[`poteto-mode`](../poteto-mode/SKILL.md) 的手册（Playbooks）一节列出了全部手册。[指南第 6 页](../../docs/guide/06-verify-and-ship.md)讲开出、照看和落地一个 PR。

pstack 没有规划技能。Cursor 的 Plan Mode 可以和它并用。跨阶段或堆叠 PR 的活，向 `/poteto-mode` 要一份计划会跑 [Multi-phase plan 手册](../poteto-mode/playbooks/multi-phase-plan.md)，它只写计划，不动手实现。设计问题用 Prototype 手册或 `/architect` 先在代码里敲定。

原则是只有一条规则的技能，`/poteto-mode` 会读它们并在回复里引用。用户很少直接调用。他们改用名字来掌舵，比如"应用 prove it works。给我看真实输出。"输入 `/principle-<name>` 仍然可以按需加载。[指南第 8 页](../../docs/guide/08-principles.md)列出了它们。

## 修复出问题的运行

| 症状 | 处理 |
|---|---|
| 模式几轮之后不再生效 | 它是用回车启动的。改成 Custom Mode 启动，或每个任务都用 `/poteto-mode` 开头。 |
| 问题被当成了上个任务的下一步 | 说"新任务"，或说这一轮不需要该模式。 |
| 新的模型选择没起作用 | `/setup-pstack` 写的规则对新聊天生效。开一个新聊天。 |
| 运行花费超出预期 | 见完成设置（Get set up）一节的成本段落。 |
| 技能没有自己加载 | 只有 `/setup-pstack` 会从用户的话里加载。其他的要么用户自己输入，要么 `/poteto-mode` 运行到它们才加载，而它不会运行每个技能。 |
| 并行的代理互相覆盖 | 给每个代理自己的工作树（worktree），或以云代理运行，云代理各有各的机器。 |
| 通宵运行在动，却什么都没完成 | `/loop` 需要一个能通过或失败的检查，不是时长。见[指南第 7 页](../../docs/guide/07-overnight.md)。 |
| 回复拿绿色构建宣称成功 | 要看真实的命令、流程、存储值或剖析。这就是 prove-it-works 原则。 |

运行跑偏时，[`references/prompting.md`](references/prompting.md) 有能一句话纠偏的短句。[指南第 10 页](../../docs/guide/10-recipes-and-pitfalls.md)有更多坑和值得照抄的食谱。

## 把 pstack 变成自己的

- [`/automate-me`](../automate-me/SKILL.md) 从用户自己的历史里起草一份个人模式技能，配合 `/poteto-mode` 使用。
- 会话之后跑 [`/reflect`](../reflect/SKILL.md)，把这次的教训变成用户批准的技能修改。
- `/poteto-mode write a skill for <workflow>` 跑 authoring 手册。Eval 手册盲测一次技能改动。
- 行为不端的技能要在它自己的 PR 里修，不要在它出问题的那个功能工作里顺手修。

[指南第 9 页](../../docs/guide/09-make-it-yours.md)逐条讲了这些。

## 回复

答案放最前面。最多给一个放在代码块里的示例提示词，有合适的就从 [`references/recipes.md`](references/recipes.md) 改编，然后附上那个文件的链接。保持简短，除非用户要的是整张地图。
