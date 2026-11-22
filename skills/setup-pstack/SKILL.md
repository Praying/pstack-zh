---
name: setup-pstack
description: 配置 pstack 各角色用什么模型、推理预算给多少。检测你可用的模型，写一条始终生效的规则来覆盖技能默认值。用于 /setup-pstack、"configure pstack models"、"pstack budget"，或者更改 pstack 的模型选择。
---

# 配置 pstack

写入 `~/.cursor/rules/pstack-models.mdc`。这是一条始终生效的规则，设定 pstack 每个角色用的模型。

## 步骤

### 1. 检测可用模型

枚举本会话中你能传给 `Task` 子代理（subagent）的模型 slug。这是可靠的来源。如果 Cursor 还提供列出用户有权使用的模型的 models API 或 CLI，优先用它来查全。如果一个都检测不到，就请用户把有权使用的 slug 贴给你。绝不要写下你没有确认可用的真实 slug。别名 `inherit-parent` 和 `auto` 永远合法，虽然它们不是检测到的 slug。

### 2. 读取当前状态

默认的角色到模型映射就是下面第 5 步展示的规则形态。如果 `~/.cursor/rules/pstack-models.mdc` 已经存在，就读它，把它的 `# budget` 行和各角色取值当作当前选择。否则从那些默认值开始。某行的角色不在第 5 步之列（例如 `hardest tasks`、`architect runners`、`reflect tooling`、`why investigators`），说明它来自已废弃的角色。删掉它。

更早的默认值先后把另一个 Gemini 模型和 `gemini-4-neon-high` 放在评审团席位上。所以在任何规则里，`gemini-4-neon-high` 条目，或者 `gemini-4-*` 中不属于 `gemini-4-neon-*` 的条目，出现在评审团角色（`arena runners`、`arena cross-judge pool`、`interrogate reviewers`）里，就是旧默认值。每个都换成 `gemini-4-neon-xhigh`。其他档位上的 Gemini 4 Neon 条目是用户的选择，保留。告诉用户你替换了哪些条目。

没有 `# defaults: 3` 行的规则来自更旧的安装。那种安装给每个角色都写了行，所以其中一些取值是旧默认值，不是选择。第 3 步之前，替换这些取值：

- 若 `arena cross-judge pool` 列表恰好是一个 `claude-fable-*` 条目、一个 `claude-opus-*` 条目和一个 `gemini-*` 条目，顺序不限，就换成该角色第 5 步的列表。Fable 和 Opus 是同一家厂商，所以 Fable 条目能评的地方，Opus 条目本来也能评。
- `# budget: unlimited` 行换成 `large`，因为没有角色跑在 `xhigh` 之上。
- 如果规则连 `# defaults: 2` 行也没有，那么匹配 `grok-*` 的就是旧 slug。`feature, refactoring`、`bug-fix`、`perf-issue`、`hillclimb` 或 `swarm workers` 里的旧 slug，换成该角色第 5 步的默认值。在评审团角色（`arena runners`、`arena cross-judge pool`、`interrogate reviewers`）里，恰好由一个 `claude-opus-*` 条目和一个旧 slug 组成的列表（顺序不限），换成该角色第 5 步的列表。其他列表里，每个旧 slug 都换成 `claude-opus-5-5-high-fast`。

其余取值都是用户的选择，一律保留，包括其他角色里的旧 slug 和所有 `inherit-parent` 或 `auto` 取值。告诉用户你替换了哪些取值。

### 3. 定预算、做映射、确认

**(a) 询问预算。** 优先用 AskQuestion 而不是自由文本。用下面这三个选项，标签一字不差。规则里记录了当前预算时，把它说出来。没有规则时，说明 `large` 与技能（skill）默认值一致。

- `large — xhigh reasoning`
- `medium — high reasoning`
- `small — medium reasoning`

**(b) 应用预算。** 从技能默认值构建工作表。重跑时，保留你按系列、列表或别名（`inherit-parent`、`auto`）改过的所有角色。系列就是 **arena** 技能 Phase A 定义的那套，所以 Fable 自成一个系列。每个角色有一个目标 effort，等于它默认的 effort 按预算平移后的档位。`large` 不动，`medium` 降一档，`small` 降两档，阶梯是 `xhigh` > `high` > `medium` > `low`。没有角色跑在 `xhigh` 之上，目标落到阶梯底部的就停在底部。多数默认值在 `xhigh`，每个预算标签标明它们落到哪里。把角色里每个真实 slug（含评审团条目）的 effort 段设为角色的目标值。effort 段是 slug 的最后一段，或者结尾 `fast` 之前的那一段。如果结果不是检测到的 slug，就用同系列里检测到的、effort 不高于目标且最高的 slug；否则把该角色标为待选择。`inherit-parent` 和 `auto` 不变。所以 `medium` 把 `claude-opus-5-5-xhigh-fast` 变成 `claude-opus-5-5-high-fast`，把 `claude-opus-5-5-high-fast` 变成 `claude-opus-5-5-medium-fast`；`small` 把它们分别变成 `claude-opus-5-5-medium-fast` 和 `claude-opus-5-5-low-fast`。Opus 5.5 始终跑它的 `-fast` slug，所以每个 Opus 5.5 取值都以 `-fast` 结尾。Fable 5.2 和 Gemini 没有 fast slug，所以绝不要写出来。Gemini 4 Neon 的 slug 从 `low` 到 `xhigh`，没有 `max` 或 `fast` slug，所以它的席位跑在 `gemini-4-neon-xhigh`，也就是它的最高 slug。评审团角色里的 Gemini 4 Neon 条目取它自己的目标值，即按预算平移的 `xhigh`。所以 `large` 保持 `gemini-4-neon-xhigh`，`medium` 写 `gemini-4-neon-high`，`small` 写 `gemini-4-neon-medium`。

**(c) 展示角色并确认。** 列出每个角色和它的模型，凡是不在检测集合里的真实 slug 都标为待选择。同时列出第 2 步删掉的每一行。问用户是照单全收还是改某些角色。可选项是检测到的 `xhigh` 及以下的模型，外加 `inherit-parent` 和 `auto`（两者含义一样：该角色跑在父级对话模型上，Auto 用户就是这样留在 Auto 上的）。优先用 AskQuestion 而不是自由文本。对评审团角色（arena runners、interrogate reviewers）来说，取值是一个列表，每个条目跑一个子代理，别名条目也算，所以列表长度决定并发数量。`arena cross-judge pool` 也是列表，但 Arena 只跑一个评审。它只尝试与父模型不同厂商的取值，按列表顺序试，所以按偏好排序。Fable 和其他 Claude 模型是同一家厂商。`inherit-parent` 或 `auto` 取值永远不参与评审。`swarm workers` 是每个工作单元的默认模型，除非竞速或对比给每一路指派了别的模型。

### 4. 校验

写下的每个真实 slug 都必须在检测集合内，且不高于 `xhigh`。`inherit-parent` 和 `auto` 永远通过。如果选中的真实 slug 不可用，停下来重新问。

### 5. 写规则

写入 `~/.cursor/rules/pstack-models.mdc`。带 `alwaysApply: true`、`# defaults: 3` 行、一条 `# budget` 行（写所选标签和该标签对应的 effort），每个角色一行，用 poteto-mode 使用的同样标签。整文件覆盖，保证重跑幂等。形态如下：

```
---
description: pstack per-role model choices (overrides skill defaults)
alwaysApply: true
---
# pstack model configuration. One line per role. Delete a line to fall back to the skill default.
# `inherit-parent` or `auto` as a value: the role runs on the parent chat model (omit Task `model` and spawn `generalPurpose`). Alias entries in a panel list still count toward its fan-out.
# defaults: 3
# budget: large (xhigh)
feature, refactoring: claude-opus-5-5-xhigh-fast
bug-fix: claude-opus-5-5-xhigh-fast
perf-issue: claude-opus-5-5-xhigh-fast
hillclimb: claude-opus-5-5-xhigh-fast
judgment and prose: claude-opus-5-5-xhigh-fast
arena runners: claude-fable-5-2-high, claude-opus-5-5-high-fast, gemini-4-neon-xhigh
arena cross-judge pool: claude-opus-5-5-high-fast, gemini-4-neon-xhigh
swarm workers: claude-opus-5-5-high-fast
interrogate reviewers: claude-fable-5-2-high, claude-opus-5-5-high-fast, gemini-4-neon-xhigh
```

### 6. 确认

告诉用户规则已写好，对新会话生效。重跑本技能会更新它。

### 7. 提供一个验证技能（可选）

看看项目有没有驱动真实应用来做证明的手段（某个 `verify-*` 技能，或现成的工装）。如果没有，就提供一次："要一个项目内的验证技能吗？这样代理（agent）就能像用户一样驱动应用，证明改动确实有效。我可以用 /create-verification-skill 生成一个。"答应就调用 `/create-verification-skill`（在 pstack 安装的任何位置解析：工作区、用户目录或插件）。拒绝就继续，不强推。

### 8. 提供一个形式化验证技能（可选）

看看项目是否已经有形式化验证技能（跑模型检查器、证明器或其他验证器来检查代码的项目技能）。如果没有，就提供一次："要一个形式化验证技能吗？这样代理就能证明你的代码的某个关键性质在范围内所有情况下都成立，而不只是测试够得到的那些用例。我可以用 /create-formal-verification-skill 生成一个。"答应就调用 `/create-formal-verification-skill`（在 pstack 安装的任何位置解析：工作区、用户目录或插件）。拒绝就继续，不强推。
