# 验证结果并开出 PR

"能编译"不算证据。[Prove It Works（证明它有效）](../../skills/principle-prove-it-works/SKILL.md) 这条原则（principle）要求代理（agent）在报成功之前检查真实产物，而你的工作就是让"真实产物"变得可检查。这一页讲这些内容：声明完成条件、审查测得的数字、为你的应用生成一份验证技能（skill）、形式化验证测试很难碰到的性质、开出 PR，以及把 PR 一路推到合并。

验证是大多数代理工作里最慢的一步，因为它通常在等人。让代理有能力做它，你就不再是瓶颈。跳过它，多跑代理只会带来更多没人检查的工作，堆给你审。

![一架原型飞机飞过真实的测试航线，她手拿秒表计时，机器人在旁拍摄，并在检查单上勾记这次飞行；终端上写着 verify: pass, evidence: captured。](./images/verification.jpg)

## 事先把完成条件说清

把"完成"的含义写进第一份提示词（prompt），措辞随你：

```text
/poteto-mode add json output to this command. text output stays byte-identical, the json parses, both run against the sample project. show me the evidence.
```

现在代理手里是三项跑得动的检查，不是一种要对上的感觉。回复回来时应该带上确切的命令与输出。某项检查跑不了，好的回复会写"无法确定"。没有证据却满口笃定的回复是个危险信号。

检查要跟改动对上：

- CLI 改动就跑真实命令。
- UI 改动就在运行中的应用里走一遍改过的流程。必须和参考图逐像素对齐时，[Visual parity 手册（playbook）](../../skills/poteto-mode/playbooks/visual-parity.md) 拿截图和冻结的基线做 diff，而不是靠肉眼判断。
- 解析器或迁移就重放一份保存好的输入。
- 性能改动就对比改动前后的剖析结果。
- 存储改动就把写进去的值读回来。

要证据，就以你自己能检视的产物形式来要：先失败后通过的那个测试、一段前后对比的视频、那份 trace、那张截图。修复已经合并的话，就在 main 上再跑一次同样的检查。产物胜过一个听起来合理的解释，因为你不必重放全程就能对它提出质疑。

一个你不敢全信的小 diff，用 [`/blast-radius`](../../skills/blast-radius/SKILL.md) 找出它可能在别处弄坏什么。它找出让这次改动安全的那一个事实，然后用跑代码来证明这个事实，而不是为它争辩。

<a id="vet-a-measured-number-with-benchmark-checklist"></a>
## 用 `/benchmark-checklist` 审查测得的数字

前后对比的数字是最容易无意中搞错的证据。缓存是热的、某一侧跑的是 debug 构建、有些工作根本没跑进计时区间，三者各自都能造出一个像模像样的加速。在你报告一个数字或依据它行动之前，输入：

```text
/benchmark-checklist vet the export speedup before it goes in the pr
```

[`/benchmark-checklist`](../../skills/benchmark-checklist/SKILL.md) 问七个问题，每一个都要来自某次运行的证据：

1. 是什么限制了这个数字？为什么它没有翻倍？
2. 每一侧都按生产环境的调优方式跑了吗？
3. 结果是否打破了物理极限，比如磁盘带宽或核心数？
4. 有没有步骤报错或返回错误输出？
5. 交替多轮运行下它能复现吗？有中位数和区间吗？
6. 在端到端层面，在用户实际等待的路径上，它要紧吗？
7. 那些工作真的发生在计时区间内吗？

裁定只有四种：更快、更慢、无可测差异、无法确定，并附上运行次数、区间和限制因素。说不出限制因素，或者有一侧没按生产方式调优，就判为无法确定。`/poteto-mode` 已经在 Perf issue 与 Hillclimb 手册里跑这份检查单，所以只有当你在它们之外自己测了东西，或者别人给的数字好得可疑时，才需要你亲手输入它。这就是 [Explain the Number（解释这个数字）](../../skills/principle-explain-the-number/SKILL.md) 原则的实战形态。

<a id="create-a-project-verification-skill"></a>
## 创建项目验证技能

上面讲 UI 的那条藏着一个真实需求。代理需要一种脚本化的方式来驱动你的应用。项目里已经有，那最好。没有，就跑：

```text
/create-verification-skill
```

[`/create-verification-skill`](../../skills/create-verification-skill/SKILL.md) 访谈的是仓库，不是你。它弄清楚用户会碰什么、应用在本地怎么启动、什么能驱动它（先找现成的工装，否则用浏览器加 CDP、一个 PTY，或者干脆 HTTP）、什么证据能证明行为，以及两个实例能不能并排跑。它只问你代码回答不了的问题。

它会写出 `.cursor/skills/verify-<app>/`，里面是面向代理的操作说明，带有确切的 Launch、Doctor、Drive、Evidence 和 Cleanup 小节，外加一个 Helpers 小节装它附带的脚本，还有 `features/` 下的一份功能地图，索引应用能做什么，以及什么结果能证明每个功能正常。这份技能自带一份[现成的功能地图示例](../../skills/create-verification-skill/references/feature-map-example/)，有 README 索引和每个功能一个文件，文件用规定的四个 H2。交付之前，生成器把这份技能端到端证明一次：启动、doctor 检查、驱动一个功能、采集证据、清理，然后确认证据在清理之后仍然留存。这次证明失败，就别用它的产出。

从今往后，"在应用里验证它"就是任何代理都能执行的一步，在这个仓库里不需要任何前置沟通。想要特定形式的证据，就在提示词里点名：

```text
/poteto-mode build the bulk-archive action. use /verify-<app> to verify your changes and show me a video and screenshots as proof.
```

```text
/poteto-mode repro this with /verify-<app>. if it repros on main, fix it and show me a video as proof.
```

验证技能一旦跑得通，[`/swarm`](../../skills/swarm/SKILL.md) 就能按功能地图条目切分一轮完整验证，再把结果汇总。一群验证代理还能在足够大的样本上确认一次性能收益，或在 PR 发布前对应用做模糊测试找回归。

把验证技能当基础设施，不是一次性用品。把它提交进仓库，让团队里每个人、每个代理都用同一种方式驱动应用。然后[打造杠杆](../../skills/principle-build-the-lever/SKILL.md)。当代理总写用完即扔的脚本去点应用时，就开口要一个小的控制 CLI，让技能去调用它。代理省下 token，每次运行也变得可重复。一个能让代理用好的 CLI 有这些特点：

- 少量能组合的命令，每个都干实事，而不是一堆单薄的命令。
- 任何破坏性的东西都带 `--dry-run` 选项。
- 子命令逐步展露功能，而不是一次全摊开。
- 报错信息会说该怎么办。
- 内容丰富的 `--help` 文本。
- 机器可读的输出，比如 JSON。

顺手把开发环境的搭建也做成可重复的：预置数据、测试用户，外加一条每次都以同样方式把环境拉起的命令。

## 让验证技能保持诚实

应用会变，功能地图会过时。下面这条至少每天跑一次，最好放进定时自动化里，省得靠人记：

```text
/maintain-verification-skill
```

[`/maintain-verification-skill`](../../skills/maintain-verification-skill/SKILL.md) 审计生成的技能：每个功能派一个只读的源码阅读器并行读，再跑一轮实测，驱动每个映射到的功能。结局恰好是三种之一。`clean` 表示覆盖完整，没有要交付的东西。`changed` 表示产出一个 PR，里面全是经过证明的修正，且限定在验证技能自己的目录里。`blocked` 则点名卡住的东西。它从不编辑产品代码。实测抓到产品回归时，它上报这个回归，而不是用一次文档修改把它藏起来。

<a id="prove-a-property-with-a-formal-verification-skill"></a>
## 用形式化验证技能证明一条性质

验证技能证明的是你驱动到的路径。有些 bug 藏在没人走的路径上：两次重试撞在一起、一把锁在写到一半时过期、一次状态变更乱序到达。测试很少够得着这些情况。对这类 bug 代价高昂的代码，跑：

```text
/create-formal-verification-skill
```

[`/create-formal-verification-skill`](../../skills/create-formal-verification-skill/SKILL.md) 从代码里挑目标：代码做出的某个承诺，一旦打破就会伤害用户，而测试很难够到打破它的那种情况。它挑最强的目标，不是最容易验证的目标。想从某个区域或某个 bug 入手，就点名：

```text
/create-formal-verification-skill the job lease and retry logic. a job must never run twice at once.
```

问题决定工具。事件会不会以坏的顺序发生，适合用模型检查器对着设计的一个小型模型来查，任何语言下这件事的默认选择都是 TLA+。代码是否对每个输入都行为正确，适合用跑在代码上的验证器。你的语言没有可用的验证器时，代理会直说，并用生成的输入去测真实代码，而不是手写一个函数模型。一条在任何规模下都必须成立的性质，可能需要一个证明。代理会告诉你它选了哪个工具，以及为什么。

它验证一条性质，并展示这条性质在一份弄坏的代码变体上会失败，因为你从没见过它失败的检查什么也证明不了。当前代码已经打破这条性质时，你会拿到一个反例，用你代码的说法写成一小串步骤。性质按原样保留，所以这个发现仍然是发现。除非你开口，产品代码不会改动。

它写一份项目内的技能，告诉下一个代理某次改动是否碰到被验证的代码、怎么用一条命令跑检查、怎么区分真 bug 和建模错误。交付之前，生成器拿一个临时改动去打破该性质，照着这份技能走一遍，检查必须抓住它。交接说明写清这些：验证了什么、用什么工具和什么边界、检查发现了什么、它不覆盖什么。一次通过覆盖的是模型或边界，不是整个系统。

这份技能还记录每个检查覆盖哪些代码，以及检查最后一次与代码对得上的那个提交。绕过这份技能的合并和重构，会让检查对着一段已经消失的代码的描述继续通过。这些操作之后，跑 [`/maintain-formal-verification-skill`](../../skills/maintain-formal-verification-skill/SKILL.md)。它从代码出发更新验证，重跑，并展示每个更新后的检查仍然能够失败，再告诉你每次失败究竟是验证漂移还是代码里的 bug。

## 用状态图展示行为

用模型语汇讲的反例能说服你。负责该行为的产品经理或设计师，得用他们自己的语汇看到它。跑：

```text
/statechart what the user sees when they edit a note offline
```

[`/statechart`](../../skills/statechart/SKILL.md) 画出用户所见内容的 Mermaid 状态图，图由 TLA+ 模型生成，所以图和被检查的模型不会各走各的。读者的规则（"离线时你永远看不到 Synced"）变成不变量。图上标出每个打破规则的状态，报告用读者的话给出到达该状态的最短路径。交接之前，代理会在真实代码里复现每条 bug 路径，这样建模错误就不会以 bug 的面目走到读者面前。

## 开出 PR

```text
/poteto-mode open the pr. small ordered commits, evidence in the description.
```

[Opening a PR 手册](../../skills/poteto-mode/playbooks/opening-a-pr.md) 从一棵工作树（worktree）出发，把工作变基成一串小而可落地的提交，清理 diff，把一份一分钟读完的简报写成正文，然后开出一个就绪的 PR。堆叠的后续 PR 胜过一条不断变长的分支。

## 用 Babysit 把 PR 推到可合并

一个开着的 PR 会立刻开始积攒阻碍。检查失败、审查者评论、主干在动。把这摊乱子交给 [Babysit 手册](../../skills/poteto-mode/playbooks/babysit.md)：

```text
/poteto-mode babysit this pr. get it green.
```

Babysit 看着这个 PR，在 GitHub 上用自带的观察器，Origin CLI 可用时改用 Origin CLI，按顺序处理阻碍：先冲突，再审查线程，然后 CI。它从不变基，也从不强推。遇到冲突或过时的基线，它连同需要变基的分支一起退回给你。评论分诊保持怀疑，因为人类和机器人在同一个清单里既报真货也报噪音。真发现就修，噪音则在贴出证伪后驳回。你只想要状态时，问小一点，Babysit 会直接作答，不启动循环：

```text
/poteto-mode check on pr 123. anything outstanding?
```

Babysit 停在可合并。即使全绿它也从不合并，因为合并是另一个决策。

## 用 Shipping 落地这摞 PR

全绿不等于安全。准备好落地时，说一声：

```text
/poteto-mode land the stack.
```

[Shipping 手册](../../skills/poteto-mode/playbooks/shipping.md) 在武装任何东西之前先独立验证每个 PR。每个 PR 派一个全新的代理现场证明行为，评判改动的代理绝不是写改动的那个。然后 Shipping 只把从底部起连续已验证的那一段落地，一次一个 PR，默认走 GitHub，Origin CLI 可用时走 Origin，并报告第一个打断链条的 PR。一个已验证的 PR 坐在一个未验证的 PR 上面就要等，因为合上它会把那段缺口一并拉进底下。

下一章：[睡觉时让工作继续跑](./07-overnight.md)。
