> 译注。此中文版仅为便于中文用户阅读和理解pstack完整内容。非官方翻译，仅供学习阅读，如有出入以英文原文为准。原文是 cursor/plugins 仓库中的 pstack 目录，https://github.com/cursor/plugins/tree/main/pstack ，版权 (c) 2026 Lauren Tan，MIT 许可，许可全文见 LICENSE 文件。

# pstack

我是 [poteto](https://x.com/poteto)。我不是总统也不是 CEO，但我在 Meta、Netflix 和 Cursor 摸过几百万行代码。我也是 react 核心团队的成员，帮忙构建和维护 react compiler。

越来越多人觉得 ai 写了太多垃圾代码。我同意。我不想像一支二十人的垃圾代码艺术家队伍那样交付。有吞吐没质量不是我追求的目标。想快，先往深里走。

**pstack 就是我的答案。**这就是我每天在 Cursor 交付高质量代码用的同一批技能（skill）。它把 cursor 变成一支真正的工程团队。目标不是最大化代码行数，恰恰相反。pstack 帮你写得更少，但质量更高。

**pstack 给你无所畏惧的并行。**只要你能在一个代理（agent）身上往深里走，信任它写出好的、可验证的代码，你就能真正放心地并行。用 `poteto-mode` 同时开多个代理，信任它们会把严谨的工程原则（principle）用到工作里。

**cursor 给你兼得众家之长。**每个前沿模型都有长有短。pstack 什么模型都能用。事实上我的很多技能都用多模型工作流，吃各家模型独有的长处。

随便 fork。随便改。改成你自己的。欢迎 PR！

## 安装

```bash
/add-plugin pstack
```

在 claude code 里：

```bash
/plugin marketplace add cursor/plugins
/plugin install pstack@cursor-plugins
```

claude code 对第三方市场默认不开自动更新，所以 pstack 停在你安装的那个版本。想打开，运行 `/plugin`，切到 **marketplaces** 标签，选 `cursor-plugins`，再选 **enable auto-update**。想手动更新一次，运行 `claude plugin update pstack@cursor-plugins`。

claude code 给插件命令和代理加前缀，所以 `/poteto-mode` 在那里叫 `/pstack:poteto-mode`。[claude code 适配器](./docs/harness/claude-code.md) 映射其余所有 cursor 名字。在 claude code 里，`/pstack:setup-pstack` 把你的模型选择写到 `~/.claude/rules/pstack-models.md`，需要 cursor 独有功能（比如 bugbot 或自动化）的步骤会带一行说明跳过。

claude code 从仓库根的 [`harness/pstack/`](https://github.com/cursor/plugins/tree/main/harness/pstack) 装一份生成的 pstack 拷贝，本 readme 和指南都在内。改完 pstack 后，在仓库根运行 `node scripts/generate-pstack-harness-trees.mjs` 并提交结果。生成拷贝过期时 ci 会红。

## 快速上手

两步：

1. 运行 [`/setup-pstack`](./skills/setup-pstack/SKILL.md)，挑一个推理预算，选好你想用的模型。
2. 任何需要严谨的事，都用 [`/poteto-mode`](./skills/poteto-mode/SKILL.md)。

新来的？[pstack 指南](./docs/guide/README.md) 带你走完第一个真实任务，从配置、写提示词（prompt）一路到验证和通宵运行。卡住了，或拿不准哪个技能合适？问 [`/poteto-help`](./skills/poteto-help/SKILL.md)。

就这么简单。其他技能看情况用，mode 技能会按需替你调用。开箱即用时，写代码的受托任务（feature、refactoring、bug fix、perf、hillclimb）、写文字和判断都去 opus 5.5 的 xhigh，没有角色超过 xhigh。评审面板之外，fable 5.2 只在你点名它，或同一份简报两次 opus 5.5 都失败后才出场。默认面板是 fable 5.2 / opus 5.5 / gemini 4 neon。[`/setup-pstack`](./skills/setup-pstack/SKILL.md) 可以改掉任何一项。

## 用法

任务开始就用 [`/poteto-mode`](./skills/poteto-mode/SKILL.md)。它读你的请求，从一套手册（playbook）里挑一本，按步骤需要运行其他技能。

打字之前先从 `/` 菜单选 `/poteto-mode`，cursor 就会把技能挂到消息上。

### 只用 [`/poteto-mode`](./skills/poteto-mode/SKILL.md)

这个技能是主快捷方式。只要我要代理干严谨的工程活，我就用它。它自带二十二本手册：

```
/poteto-mode this pr has a subtle bug where the scroll drifts every 750ms even when idle. repro
first, then fix and verify.
```

```
/poteto-mode i'm going to bed. land the stack even if ci flakes. i want everything merged by
morning.
```

<details>
<summary>二十二本手册</summary>

| 手册 | 用途 |
|---|---|
| [investigation](./skills/poteto-mode/playbooks/investigation.md) | 一个只读的问题。x 怎么工作，y 为什么建成这样，我们确定吗。 |
| [bug fix](./skills/poteto-mode/playbooks/bug-fix.md) | 复现缺陷，找到根因，用运行时证据修复。 |
| [perf](./skills/poteto-mode/playbooks/perf-issue.md) | 追踪测出来的慢，对着基线改进。 |
| [hillclimb](./skills/poteto-mode/playbooks/hillclimb.md) | 对一个指标做持续的、科学的爬坡改进，朝目标推进，循环验证假设，每次带前后测量，每个被采纳的胜利配一个提交。 |
| [runtime forensics](./skills/poteto-mode/playbooks/runtime-forensics.md) | 从插桩数据诊断活的症状（泄漏、空转 cpu 打转、毛刺）。 |
| [trace forensics](./skills/poteto-mode/playbooks/trace-forensics.md) | 诊断抓下来的剖析工件（cpuprofile、trace、spindump、heap snapshot）。 |
| [feature](./skills/poteto-mode/playbooks/feature.md) | 新的或改动的行为，从一个有名字的数据形状构建。 |
| [refactoring](./skills/poteto-mode/playbooks/refactoring.md) | 保行为的结构或形状改动。 |
| [prototype](./skills/poteto-mode/playbooks/prototype.md) | 用完即弃的草样，便宜地定下设计或行为决策，或靠观察定夺经验性分岔。 |
| [visual parity](./skills/poteto-mode/playbooks/visual-parity.md) | 两套实现之间像素级一致的 ui 等价。 |
| [authoring a skill](./skills/poteto-mode/playbooks/authoring-a-skill.md) | 写或改一份 SKILL.md。 |
| [eval](./skills/poteto-mode/playbooks/eval.md) | 盲测一次技能或提示词改动怎么影响代理行为。 |
| [babysit](./skills/poteto-mode/playbooks/babysit.md) | 把一个 PR 或一摞推到可合并：冲突、评审线索、ci。 |
| [shipping](./skills/poteto-mode/playbooks/shipping.md) | 独立验证绿的一摞，然后把连续验证过的部分自底向上落地，默认走 github，可用时走 origin。 |
| [autonomous run](./skills/poteto-mode/playbooks/autonomous-run.md) | 把一个长任务不停顿地干到完成。 |
| [autopilot-full](./skills/poteto-mode/playbooks/autopilot-full.md) | 把独立的 PR 跑到合并，一个 PR 一个负责人。swarm 验证每个代码就绪的头部，补丁之后的每一处新改动都重新检查。 |
| [autopilot-stack](./skills/poteto-mode/playbooks/autopilot-stack.md) | autopilot-full 去掉合并。构建并验证 PR，交操作者审阅和落地，PR 只叠在它确实基于的 PR 上。 |
| [session pickup](./skills/poteto-mode/playbooks/session-pickup.md) | 续上或接手之前代理的在途工作。 |
| [pause safely](./skills/poteto-mode/playbooks/pause-safely.md) | 把在途工作干净地挂起，之后能续。 |
| [multi-phase plan](./skills/poteto-mode/playbooks/multi-phase-plan.md) | 跨阶段或跨叠放 PR 的工作。 |
| [worktree cleanup](./skills/poteto-mode/playbooks/worktree-cleanup.md) | 回收磁盘：剪掉已合并或废弃的工作树（worktree）和过期 ios 模拟器，经安全检查关卡（gate）把关。 |
| [opening a pr](./skills/poteto-mode/playbooks/opening-a-pr.md) | 用排好序的小提交开出就绪的 PR，标题走 conventional commits，正文像简报。在其他每本手册收尾时调用。 |

</details>



它被调用时会：

1. 把你的任务匹配到一本[手册](./skills/poteto-mode/playbooks/)，打开待办清单，开头几项就是手册步骤原样抄入。
2. 步骤触发时路由到其他技能。
3. 写出不带垃圾的回复，面向使用者和维护者框定内容。

完整规则和手册都在 [`skills/poteto-mode/SKILL.md`](./skills/poteto-mode/SKILL.md)。

想让 [`/poteto-mode`](./skills/poteto-mode/SKILL.md) 跨轮次一直开着，就从 `/` 菜单选它，按 option+enter（mac）或 alt+enter（windows）代替回车。这会把它变成一种[自定义模式](https://cursor.com/docs/skills#using-a-skill-as-a-custom-mode)，cursor 在 agents 窗口和 cli 里都提供。退出该模式之前它每轮都留在上下文里。单按回车只挂到一条消息上。

[`/poteto-mode`](./skills/poteto-mode/SKILL.md) 和 cursor 的 `/loop` 命令是绝配。你可以让 cursor 干很多个小时，还不牺牲严谨。

## 技能

步骤需要时，[`/poteto-mode`](./skills/poteto-mode/SKILL.md) 会替你跑掉大多数（`why`、`architect`、`arena`、`swarm`、`interrogate`、`unslop`、`no-comments`、`technical-writing`、`tdd`，以及各条原则）。下表供你想直接点名时用：

```
/how do we cancel runs? do we have an n+1 when we look up every run to cancel?
```

```
/interrogate review this pr.
```

<details>
<summary>全部技能</summary>

| 技能 | 什么时候用 |
|---|---|
| [`/poteto-mode`](./skills/poteto-mode/SKILL.md) | 任何非平凡任务的默认入口。 |
| [`/poteto-help`](./skills/poteto-help/SKILL.md) | 你是 pstack 新手，或拿不准哪个技能、手册或原则合适。它先弄清你想干什么，把那部分答了，再递一条提示词给你输入。只在你输入 `/poteto-help` 时运行。 |
| [`/in-your-own-words`](./skills/in-your-own-words/SKILL.md) | 你想让代理开工之前先把你的目标和问题复述一遍。只在你输入 `/in-your-own-words` 时运行。 |
| [`/sharpen`](./skills/sharpen/SKILL.md) | 你想在发出之前把提示词改得更深，而不是更长：顶尖高手会做而你没要求的事、他们起步的原材料、他们的方法和工具，以及一个外部检查来确认结果对。它递回重写后的提示词，里面要求代理把实验能定夺的选择都做原型，产品和偏好类选择留给你。不做任务本身。只在你输入 `/sharpen` 时运行。 |
| [`/how`](./skills/how/SKILL.md) | 你想走一遍某个子系统怎么工作。 |
| [`/why`](./skills/why/SKILL.md) | 你想知道某样东西为什么建成这样。答案来自记录而非代码：引入它的提交和 PR 讨论、之后的每次改动，线索太薄时再看别的来源。它引用作者写过的话，标出自己的推断，并说出记录回答不了的部分。 |
| [`/recall`](./skills/recall/SKILL.md) | 你要开工或续工，想把自己在某主题上的近期上下文从你的聊天历史和共享记录里重建出来，拿回一份紧凑的现状简报。 |
| [`/ketchup`](./skills/ketchup/SKILL.md) | 你想知道这条对话从你上条消息之后变了什么，用大白话加小图讲清，每件等你拍板的事都列出来、只看回复就能定，哪怕回复因此很长。 |
| [`/ketchup-video`](./skills/ketchup-video/SKILL.md) | 你想把 `/ketchup` 的补课做成一段带字幕的幻灯短片，等你拍板的决定以文字列在片下。 |
| [`/blast-radius`](./skills/blast-radius/SKILL.md) | 一个改动看着很小，你想知道它还能弄坏什么，而且那条让它安全的关键事实要由跑起来的代码证明，不是嘴上断言。 |
| [`/architect`](./skills/architect/SKILL.md) | 你要写的代码跨函数边界，想先把调用方的用法、类型和模块形状定下来。 |
| [`/arena`](./skills/arena/SKILL.md) | 你想对同一件事开 N 路并行尝试，再把各家最好的部分抓过来。 |
| [`/swarm`](./skills/swarm/SKILL.md) | 你想把 N 个工作单元铺到不同切片或竞速上，最后拿一份汇总报告。 |
| [`/interrogate`](./skills/interrogate/SKILL.md) | 你有一份 diff，想让不同模型来攻破它，只留运行证实过的发现。 |
| [`/automate-me`](./skills/automate-me/SKILL.md) | 你想要自己的 `-mode` 技能，按你实际干活的样子起草。 |
| [`/make-bot-ui`](./skills/make-bot-ui/SKILL.md) | 你想要一个页面或仪表盘，按钮通过 webhook 唤醒 Grok Bot，含 sender-key 交接和 Tailscale。 |
| [`/setup-pstack`](./skills/setup-pstack/SKILL.md) | 你想按角色挑 pstack 用哪些模型。它检测你的模型并写一条配置规则。 |
| [`/reflect`](./skills/reflect/SKILL.md) | 长任务落地了，你想把食谱沉淀成一次技能改动。 |
| [`/correct`](./skills/correct/SKILL.md) | 你老在纠正代理同一批错。它从历史里挖出错误类别，在管用的最高层修掉每个（先架构，再类型、lint 和 ci，然后测试，文档垫后），并留一张表，把每条规则和它的执行手段配对。 |
| [`/teach`](./skills/teach/SKILL.md) | 你想真正弄懂一个改动或子系统，不只是听个摘要。它跑 how 加 why，织成一份平实讲解，一张图一张图搭起来。 |
| [`/tdd`](./skills/tdd/SKILL.md) | 你在修 bug，而且有便宜的本地测试路径。先写失败的测试，再写修复。 |
| [`/benchmark-checklist`](./skills/benchmark-checklist/SKILL.md) | 你跑了一次基准测试，或测出某个加速或回退。它在你报告或据此行动之前审查这个数字（限制器、调参、误差、重复运行、端到端相关性）。 |
| [`/no-comments`](./skills/no-comments/SKILL.md) | 评审前剥掉注释。生成 Comment Sicko，修掉被接受的发现，给声称的约束提供编码方案。 |
| [`/typescript-best-practices`](./skills/typescript-best-practices/SKILL.md) | 你在读或改 typescript。把 type-system-discipline 原则落到语法上。 |
| [`/figure-it-out`](./skills/figure-it-out/SKILL.md) | 内置手册都不合适。它为这个任务设计一本严谨、可审计的手册。 |
| [`/show-me-your-work`](./skills/show-me-your-work/SKILL.md) | 你想要一份可评审的决策轨迹。它把决策记进一份可提交的 tsv。 |
| [`/create-verification-skill`](./skills/create-verification-skill/SKILL.md) | 你的项目没有脚本化的办法证明应用行为。它生成一个项目内的 verify 技能，带功能地图，语言平台不限。 |
| [`/maintain-verification-skill`](./skills/maintain-verification-skill/SKILL.md) | 你的 verify 技能的功能地图和应用漂移了。扫一轮源码加一轮实跑，最多一个 PR 的经证实修正。 |
| [`/create-formal-verification-skill`](./skills/create-formal-verification-skill/SKILL.md) | 测试够不着能击穿竞态、重试、状态机、解析器或 unsafe 代码的用例。它用合适的工具验证一条性质，并生成一个项目内技能，每次改动都重查。 |
| [`/maintain-formal-verification-skill`](./skills/maintain-formal-verification-skill/SKILL.md) | 你的形式化验证覆盖的代码自上次检查后变了。它更新模型、工装（harness）或性质来对齐代码，重跑，并说出每个失败是验证漂移还是代码 bug。 |
| [`/statechart`](./skills/statechart/SKILL.md) | 负责这块行为的人需要看见它，而不是读规格。它检查 tla+ 模型，把用户看到的东西画成 mermaid 状态图，标出每个违反规则的状态，并给出到每个状态的最短路径。 |
| [`/unslop`](./skills/unslop/SKILL.md) | 你在清理文字。去掉 AI 腔。 |
| [`/bro`](./skills/bro/SKILL.md) | 你想把上一条消息用纯人话重述一遍，不带黑话。 |
| [`/technical-writing`](./skills/technical-writing/SKILL.md) | 分层文档标准（Diátaxis + Google developer style + STE + Global English），用于文档、RFC、readme、PR 描述、提交信息。 |

</details>



### 示例

多数时候我在任务开始时敲 [`/poteto-mode`](./skills/poteto-mode/SKILL.md)，让它路由到一本手册。其他技能按步骤需要触发。有几个我会直接点名。


<details>
<summary>全部示例</summary>

```
bug fix:           /poteto-mode this pr has a subtle bug where the scroll drifts every 750ms even
                   when idle. repro first, then fix and verify.
perf:              /poteto-mode a big list takes a second or two to load even though we virtualize.
                   run a cpu trace and tell me why.
feature:           /poteto-mode build a small feature behind a feature flag. verify it really works.
prototype:         /poteto-mode build two prototypes of the markdown renderer so we can compare.
                   spawn an agent for each.
multi-phase:       /poteto-mode open source these skills as a plugin. nothing internal leaks, work
                   in a temp dir, show me the dependency graph first.
overnight run:     /poteto-mode i'm going to bed. land the stack even if ci flakes. i want
                   everything merged by morning.
babysit:           /poteto-mode check on pr 123. anything outstanding?
visual parity:     /poteto-mode the row spacing is too tall when this flag is on. the second image
                   is correct. repro and fix until it matches.
figure it out:     /poteto-mode i'm stepping away. migrate every caller from the synchronous store
                   to the new async one, keeping behavior identical. i want to trust it was done
                   right when i'm back.
how:               /how do we cancel runs? do we have an n+1 when we look up every run to cancel?
why:               /why is this feature flag not on yet?
architect:         design this instrumentation to be high signal with no false positives. /architect
                   this first.
arena:             /arena take my prompt to the arena verbatim. i want to compare their proposals
                   with yours.
swarm:             /swarm check every package under packages/ against its check.sh. one worker per
                   package. one report.
interrogate:       /interrogate review this pr.
tdd:               /tdd implement
unslop:            can we unslop and tighten the new changes?
reflect:           /reflect that took too long. capture what we learned so the next run doesn't
                   repeat it.
correct:           /correct
show-me-your-work: /show-me-your-work keep a decision trail i can review when i'm back.
automate-me:       /automate-me
help:              /poteto-help which skill should i use to review this branch?
```

</details>

## `poteto-agent` 与 Comment Sicko 子代理（subagent）

[`/poteto-mode`](./skills/poteto-mode/SKILL.md) 在接收它的那个对话里运行。那个对话持有手册、待办清单和每一次子代理生成。

pstack 还附带 [`subagent_type: "poteto-agent"`](./agents/poteto-agent.md)，作为运行中派出的受托子代理的包装。它开工前先完整读 `poteto-mode`，含内联原则索引。换用 `generalPurpose` 会跳过这读，行为会漂移。别用 `poteto-agent` 来跑手册本身。

[`/poteto-mode`](./skills/poteto-mode/SKILL.md) 和 [`subagent_type: "poteto-agent"`](./agents/poteto-agent.md) 走同一个包装。

pstack 还附带 [Comment Sicko](./agents/comment-sicko.md)，一个只读的注释评审，以 `subagent_type: "comment-sicko"` 提供。一般通过 [`/no-comments`](./skills/no-comments/SKILL.md) 调用，别直接叫。

## 原则

二十四条短技能，一条原则一条。`poteto-mode` 把它们内联成索引，任务开始时读这份索引。独立文件存在的意义是让别的技能能按名字引用某条原则，也是为了让索引能指向每条的完整规则。

<details>
<summary>全部二十四条原则</summary>

| 原则 | 分组 | 规则 |
|---|---|---|
| [laziness-protocol](./skills/principle-laziness-protocol/SKILL.md) | 核心 | 偏向删减，选能解决问题的最小改动。 |
| [foundational-thinking](./skills/principle-foundational-thinking/SKILL.md) | 核心 | 写逻辑之前用：选核心类型与数据结构、排定脚手架和功能的先后、问清并发各方共用什么。数据结构对了，下游代码就自然明朗。 |
| [redesign-from-first-principles](./skills/principle-redesign-from-first-principles/SKILL.md) | 核心 | 当作这条需求从第一天起就是基础假设来重新设计，而不是往上硬焊。 |
| [attack-the-premise](./skills/principle-attack-the-premise/SKILL.md) | 核心 | 两个以上共享同一前提的修复在同一个检查关卡（gate）失败时用。把前提写下来直接验它，别再写下一个默认它成立的修复。 |
| [subtract-before-you-add](./skills/principle-subtract-before-you-add/SKILL.md) | 核心 | 先删死重、多余的校验器和桩引用，再在更简单的地基上建。 |
| [minimize-reader-load](./skills/principle-minimize-reader-load/SKILL.md) | 核心 | 数一数问题与答案之间的层数，和读者脑子里的隐藏状态。收掉只有一个调用方的包装，缩小小可变范围。 |
| [outcome-oriented-execution](./skills/principle-outcome-oriented-execution/SKILL.md) | 核心 | 用于阶段边界明确的计划性重写和迁移。向目标架构收敛，别用一次性兼容代码去保平滑的中间态。 |
| [experience-first](./skills/principle-experience-first/SKILL.md) | 核心 | 选用户的愉悦，不选实现的方便。少而精的功能胜过多而糙的。 |
| [exhaust-the-design-space](./skills/principle-exhaust-the-design-space/SKILL.md) | 核心 | 先做 2-3 个互相竞争的原型，并排比过再定。 |
| [build-the-lever](./skills/principle-build-the-lever/SKILL.md) | 核心 | 用于任何非平凡工作，不只大批量活：编辑、迁移、分析、检查。造一个能把活干掉或把活证明掉的工具（codemod、脚本、生成器，或一个你子代理照做的技能），别手工干。工具是评审者能重跑的工件。 |
| [model-the-domain](./skills/principle-model-the-domain/SKILL.md) | 架构 | 把领域编码进结构，而不是散落的条件分支。 |
| [boundary-discipline](./skills/principle-boundary-discipline/SKILL.md) | 架构 | 把防护集中在系统边界（CLI、配置、网络、外部 API）。信任内部类型，业务逻辑放纯函数。 |
| [type-system-discipline](./skills/principle-type-system-discipline/SKILL.md) | 架构 | 让非法状态无法表达，给语义原语打品牌，外部数据在边界解析，穷尽变体，从权威 schema 派生。 |
| [make-operations-idempotent](./skills/principle-make-operations-idempotent/SKILL.md) | 架构 | 无论之前跑了一半多少次，都收敛到同一个终态。 |
| [migrate-callers-then-delete-legacy-apis](./skills/principle-migrate-callers-then-delete-legacy-apis/SKILL.md) | 架构 | 同一波里迁完调用方并删掉旧 API，而不是留着兼容层。 |
| [separate-before-serializing-shared-state](./skills/principle-separate-before-serializing-shared-state/SKILL.md) | 架构 | 先消灭共享。只有单一共享写者真是不变量时，才做结构化串行。 |
| [prove-it-works](./skills/principle-prove-it-works/SKILL.md) | 验证 | 任务完成后、宣布做完之前用。对着真工件验证（跑功能、读真实值、看 diff），不是对着代理物、自述，或"能编译"。 |
| [fix-root-causes](./skills/principle-fix-root-causes/SKILL.md) | 验证 | 把每个症状追回系统最初出错的地方，在那里修。只有它处理的状态合法时，防护才算修复。 |
| [sequence-verifiable-units](./skills/principle-sequence-verifiable-units/SKILL.md) | 验证 | 用于多步工作（清扫（sweep）、迁移、成串的相似编辑），也用于提交与 PR 的堆叠顺序。把工作拆成小单元，各自收在可验证的状态，验完一个再下一个，交付顺序本身要能向评审证明工作成立。 |
| [test-behavior-not-implementation](./skills/principle-test-behavior-not-implementation/SKILL.md) | 验证 | 写、改或留测试时用。断言代码使用者观察得到的东西，行为破坏时测试就红，只改内部时测试保持绿。 |
| [explain-the-number](./skills/principle-explain-the-number/SKILL.md) | 验证 | 在你信任、报告或依据某个测得数字行动之前用：加速、回退、吞吐、延迟或评测结果。找出是什么限制了它，排除它测的其实是别的东西。 |
| [guard-the-context-window](./skills/principle-guard-the-context-window/SKILL.md) | 委派 | 大宗内容路由给子代理，主线程只留摘要，不留原始货。 |
| [never-block-on-the-human](./skills/principle-never-block-on-the-human/SKILL.md) | 委派 | 先干，摆出结果，让人事后纠偏。确认只留给不可逆动作。 |
| [encode-lessons-in-structure](./skills/principle-encode-lessons-in-structure/SKILL.md) | 元 | 把规则编码成 lint、元数据标志、运行时检查或脚本，而不是更多的文字。 |

</details>

## 这里不附带的东西

`poteto-mode` 引用但不打包的几样东西：

- `/deslop` 和 `deslop` 技能在 `cursor-team-kit` 插件里。
- `control-cli`（管 CLI 和 TUI）和 `control-ui`（管浏览器、Electron、web）也在 `cursor-team-kit` 里。
- `/create-skill` 是 cursor 内置。cursor 还内置一个看护 PR 的 `/autopilot`。在 `poteto-mode` 里，PR 状态类请求改走 [babysit 手册](./skills/poteto-mode/playbooks/babysit.md)。

想要全套，就把 `cursor-team-kit` 和 pstack 一起装。

## 为什么没有规划技能？

cursor 已经有一个很好的计划模式，和 pstack 配合得很好。但就我个人而言，我不信规划。最好的规格就是代码。你真要做计划，[`/poteto-mode`](./skills/poteto-mode/SKILL.md) 也罩得住，但它不是默认动作。

## 改成你自己的

`poteto-mode` 是我的风格。你未必想要一模一样的。

敲 [`/automate-me`](./skills/automate-me/SKILL.md)。它挖你最近的转录（transcript），按你实际干活的样子起草一份 `<your-name>-mode` 技能，底下仍走 pstack 路由。pstack 仍是你的地基，最终你得到一个和 `poteto-mode` 并列的自家路由技能。

模型也可配置。敲 [`/setup-pstack`](./skills/setup-pstack/SKILL.md)。它检测你能用的模型，写一条总是生效的小规则，把每个角色（写代码、判断、评审面板）映射到一个模型。每个技能都读它，规则缺失时退回合理默认值，所以你只覆盖想改的。

默认值变了之后，改动前写的规则仍钉着旧默认值，直到你再跑一次 `/setup-pstack`。那一轮会替换它认得的旧默认值，保留其他模型与默认值不同的角色。要拔掉它不认得的钉子，先删那行或整个文件。

## 自动化

pstack 还附带一套休眠的 [benny 自动化包](./automations/benny/)。benny 给 slack 的 issue 报告分诊，然后用真实 ui 证据复现并修复确认的 bug。它的文件没有注册成斜杠技能。

要配置它，让 cursor 读 [`FOR_AGENTS.md`](./automations/benny/FOR_AGENTS.md)。配置会把包拷进目标仓库的 `.cursor/automations/benny/`，在那里为共享技能启用 pstack，用户配置留在拷贝的包之外。benny 跑在 cursor 自动化上，所以只此一家。

## 许可证

MIT
