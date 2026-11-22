# 用原则（principle）名字掌舵

pstack 把 24 条原则做成一份份独立技能（skill）。`/poteto-mode` 在每个多步任务开始时读它们的索引，应用任务触发到的那些，并在回复里点名每条用上的原则，以及它改变了哪个决策。

你不调用原则。你用它们的名字掌舵。每个名字指向一条代理（agent）早已读过的完整规则，所以一个短语就能比一段指令更精确地把工作改道。

## 掌舵实战

假设代理正要把一个新适配器硬接到现有三个上：

```text
use subtract before you add. delete the obsolete adapters first, then design what's left.
```

假设它因为构建通过就宣称成功：

```text
apply prove it works. run the real import flow and show me the written records.
```

假设两次并行尝试马上要写同一个分支：

```text
separate before serializing shared state. give each attempt its own worktree, no locks.
```

每个短语都打得中，因为它背后的规则足够具体。代理仍然必须在回复里说清这条规则改变了哪个决策。只引原则名而背后没有决策，就说明代理只是报了名字，没有真的应用。

## 24 条，一句话版

核心原则决定造多少、何时重新思考设计：

- [Laziness Protocol（懒惰协议）](../../skills/principle-laziness-protocol/SKILL.md) 偏向删减，选能解决问题的最小改动。
- [Foundational Thinking（基础思考）](../../skills/principle-foundational-thinking/SKILL.md) 在写逻辑之前先选定核心数据结构。
- [Redesign from First Principles（从第一性原理重新设计）](../../skills/principle-redesign-from-first-principles/SKILL.md) 整合新需求时，当作它从第一天就在场。
- [Attack the Premise（攻击前提）](../../skills/principle-attack-the-premise/SKILL.md) 把两个以上失败修复共用的前提写下来，在下一次修复之前直接检验它。
- [Subtract Before You Add（先做减法，再做加法）](../../skills/principle-subtract-before-you-add/SKILL.md) 在其上建造之前先去掉死重。
- [Minimize Reader Load（最小化阅读负担）](../../skills/principle-minimize-reader-load/SKILL.md) 折叠读者必须在脑子里记着的层和隐藏状态。
- [Outcome-Oriented Execution（面向结果的执行）](../../skills/principle-outcome-oriented-execution/SKILL.md) 让重写向目标设计收敛，而不是保着那些用完即弃的兼容状态。
- [Experience First（体验优先）](../../skills/principle-experience-first/SKILL.md) 选用户得到的结果，不选实现上的方便。
- [Exhaust the Design Space（穷尽设计空间）](../../skills/principle-exhaust-the-design-space/SKILL.md) 在没有先例时造两三个互相竞争的原型。
- [Build the Lever（打造杠杆）](../../skills/principle-build-the-lever/SKILL.md) 造出能干成或证明工作的脚本，让审查者可以重跑。当代理老是手工做同一件事时，让它写出自己想要的那个工具或技能。脚本能把一个步骤每次都做得一模一样时，就用脚本，把代理留给需要判断的决策。

架构原则决定状态、验证和兼容性住在哪里：

- [Model the Domain（为领域建模）](../../skills/principle-model-the-domain/SKILL.md) 把反复出现的规则编码进一个结构，而不是散落的条件分支。
- [Boundary Discipline（边界纪律）](../../skills/principle-boundary-discipline/SKILL.md) 在边界处验证，信任内部类型。
- [Type System Discipline（类型系统纪律）](../../skills/principle-type-system-discipline/SKILL.md) 让非法状态无法被表示。
- [Make Operations Idempotent（让操作幂等）](../../skills/principle-make-operations-idempotent/SKILL.md) 让重试收敛到同一个终态。
- [Migrate Callers Then Delete Legacy APIs（先迁移调用方，再删除旧 API）](../../skills/principle-migrate-callers-then-delete-legacy-apis/SKILL.md) 迁移和删除一波做完。
- [Separate Before Serializing Shared State（先拆分，再串行化共享状态）](../../skills/principle-separate-before-serializing-shared-state/SKILL.md) 在加协调之前先消灭共享。

验证原则定义什么才算证据：

- [Prove It Works（证明它有效）](../../skills/principle-prove-it-works/SKILL.md) 验证真实产物，不是替代物。
- [Fix Root Causes（修复根因）](../../skills/principle-fix-root-causes/SKILL.md) 改代码之前先复现并追到原因。
- [Sequence Work into Verifiable Units（把工作排成可验证的单元序列）](../../skills/principle-sequence-verifiable-units/SKILL.md) 让每个小单元收尾在一次检查上，再开始下一个。
- [Test Behavior, Not Implementation（测行为，不测实现）](../../skills/principle-test-behavior-not-implementation/SKILL.md) 断言代码的使用者能观察到什么，这样行为一坏测试就失败，动内部却不受影响。
- [Explain the Number（解释这个数字）](../../skills/principle-explain-the-number/SKILL.md) 在任何人信任或报告一个测得数字之前，点名什么限制了它，并排除它测的是别的东西。[`/benchmark-checklist`](../../skills/benchmark-checklist/SKILL.md) 把它变成七个用真实运行来回答的问题。

委派原则让并行工作保持清醒：

- [Guard the Context Window（守护上下文窗口）](../../skills/principle-guard-the-context-window/SKILL.md) 把大块阅读路由给子代理（subagent），把发现留在主会话。
- [Never Block on the Human（绝不阻塞在人类身上）](../../skills/principle-never-block-on-the-human/SKILL.md) 在可逆工作上继续推进，并把结果摆出来。

还有一条元原则：

- [Encode Lessons in Structure（把教训编码进结构）](../../skills/principle-encode-lessons-in-structure/SKILL.md) 把你重复过两遍的建议变成一个 lint、检查或脚本。[`/correct`](../../skills/correct/SKILL.md) 把它应用到整个仓库，详见 [定制成你自己的](./09-make-it-yours.md#fix-the-environment-with-correct)。

别背这张清单。现在扫一遍，等你抓到代理干出这里某个名字本可拦住的事，再回来翻。词汇就是这么记住的。

下一章：[定制成你自己的](./09-make-it-yours.md)。
