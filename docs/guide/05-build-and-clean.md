# 构建改动并清理 diff

构建类手册（playbook）共用一条纪律。你说观察到了什么，让手册去要证据。这一页讲每种常见构建任务的提示词（prompt）里写什么，再讲让 diff 保持可评审的清理习惯。

## 给每个构建手册的提示词写上你所知的

bug 提示词说清症状，并先要一个复现：

```text
/poteto-mode this command emits two records after a retry. repro first, then fix and verify.
```

功能提示词说清行为，以及什么不许变：

```text
/poteto-mode add a --json flag. text output stays byte-identical. verify both forms.
```

重构提示词在结构挪动之前钉死行为：

```text
/poteto-mode move parsing into one module, zero behavior change. record the current output first and prove it's unchanged after.
```

性能提示词报的是测量，不是印象：

```text
/poteto-mode startup takes 1.8s on this fixture. trace it, fix the measured cause, show me before and after.
```

每种都会路由到自己的手册（[Bug fix](../../skills/poteto-mode/playbooks/bug-fix.md)、[Feature](../../skills/poteto-mode/playbooks/feature.md)、[Refactoring](../../skills/poteto-mode/playbooks/refactoring.md)、[Perf issue](../../skills/poteto-mode/playbooks/perf-issue.md)），手册补上你没打的步骤：修复之前先复现，实现之前先给数据形状命名，重构之前先钉死行为，优化之前先做剖析。

要把一个数字长期往上拱，有 [Hillclimb 手册](../../skills/poteto-mode/playbooks/hillclimb.md)。给它指标、目标和尝试次数的下限，它就用冻结的测量工装（harness）一次循环验证一个假设。留下赢的，其余全部回退。

两本性能手册都会对它们的数字跑 [`/benchmark-checklist`](../../skills/benchmark-checklist/SKILL.md)。Perf issue 把关基线和其后的每个数字，Hillclimb 在冻结之前把关它的执行环境。[验证并交付](./06-verify-and-ship.md#vet-a-measured-number-with-benchmark-checklist) 讲了什么时候该自己输这条命令。

有时你想在任何修复之前先拿到原因。活的症状，比如泄漏、空转 CPU 打转或画面毛刺，用 [Runtime forensics 手册](../../skills/poteto-mode/playbooks/runtime-forensics.md) 给运行中的进程插桩。已经抓好的剖析文件，用 [Trace forensics 手册](../../skills/poteto-mode/playbooks/trace-forensics.md) 读工件，把热点帧映射回源码。两者都只交诊断，不交修复：

```text
/poteto-mode here's a cpuprofile from the slow startup. tell me where the time goes and which source lines own it. no fix yet.
```

## 用 `/tdd` 先写失败的测试

bug 有便宜的本地测试路径时，整条提示词可以只有两个词：

```text
/tdd implement
```

上下文里，这就够了。[`/tdd`](../../skills/tdd/SKILL.md) 先写一个因为正确原因失败的最小测试，然后写修复，再重跑测试。测试需要大摊子执行环境搭台或脆弱的 mock 时，技能（skill）会直说，改用最接近的可执行检查。真实命令是更强证据的地方，别硬塞测试。

## 按名字加载 TypeScript 规则

[`typescript-best-practices`](../../skills/typescript-best-practices/SKILL.md) 把类型系统原则（principle）变成具体规则：可辨识联合、边界的 `unknown`、穷尽变体、从 schema 派生类型。它不会自己加载，所以任务碰到 `.ts` 或 `.tsx` 文件时输入 `/typescript-best-practices`。

## 提交之前先清理

[Opening a PR 手册](../../skills/poteto-mode/playbooks/opening-a-pr.md) 在每次提交前对 diff 跑 `/deslop`，poteto-mode 会对 PR 描述和提交正文用 [`/unslop`](../../skills/unslop/SKILL.md)。`/deslop` 在 `cursor-team-kit` 插件里，不在 pstack。没有它就用大白话要同样的结果：删掉叙事式注释、没依据的防护、死掉的兼容路径和无关改动。

对文字，`/unslop` 接受一个目标，外加你的任何额外规则：

```text
/unslop the readme changes, no emdashes
```

你会磨出自己的简写。像 `unslop that, tighten it` 这种简短提示词，技能照样读得懂意图。

## 用 `/no-comments` 剥掉注释

注释要单独过一遍，而且不能由写它的代理（agent）来过。作者护自己的注释，就像你护你的一样。所以评审之前，把它们交给一双新鲜的眼睛：

```text
/no-comments the diff
```

[`/no-comments`](../../skills/no-comments/SKILL.md) 会生成 [Comment Sicko](../../agents/comment-sicko.md)，一个只读评审，手握一份短短的保留清单：许可证声明头、公共 API 的文档注释、解释代码表达不了的东西的链接、外部依赖强加而你重塑不了的行为。其余全删。你自己代码里的意外没有这等优待。注释会以重构标记的形式回来，`/no-comments` 对接受的标记在根因上修好。注释声称有约束（"do not remove"）时，技能会提议把这条主张编码成类型、测试或 lint。无论走哪条路，注释都会被拿掉。

分工值得记清楚。`/deslop` 清代码里的垃圾，`/unslop` 清文字里的垃圾，`/no-comments` 把注释交给没写它们的评审。

**坑：**清理不是可选的抛光。带着叙事式注释和防御性死重的 diff，在评审眼里就是没做完，而多出来的代码正是下一个 bug 的藏身处。diff 看着虚胖，就在提交前说 `deslop it`，别等评审点破。

下一章：[验证并交付](./06-verify-and-ship.md)。
