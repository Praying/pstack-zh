# 配置 pstack

这一页里你安装插件，挑好 pstack 用哪些模型，然后跑第一个任务。配置就是一条命令加一小段对话。

## 安装插件

在 Cursor 对话里运行：

```text
/add-plugin pstack
```

Cursor 会确认插件已安装。

在 Claude Code 里，先运行 `/plugin marketplace add cursor/plugins`，再运行 `/plugin install pstack@cursor-plugins`。Claude Code 会给插件命令加前缀，所以本指南里每条命令在那里都要带 `pstack:` 前缀，比如 `/pstack:setup-pstack`，它把模型规则写到 `~/.claude/rules/pstack-models.md`。[Claude Code 适配器](../harness/claude-code.md) 讲了其余差异。

## 挑选你的模型

运行：

```text
/setup-pstack
```

[`/setup-pstack`](../../skills/setup-pstack/SKILL.md) 检测你能用的模型，问你要多大的推理预算，把每个角色（写代码的受托任务、判断、评审面板）摆给你看，问你想要什么。回答问题就好。它会写 `~/.cursor/rules/pstack-models.mdc`，一份每个 pstack 技能（skill）都会读的小规则。

默认值与 `large` 预算相同。写代码、写文字和判断跑在 Opus 5.5 上，推理档位 `xhigh`，评审面板和 swarm 工作单元跑在 `high`。没有角色超过 `xhigh`。评审面板之外，Fable 5.2 只在你为任务点名它，或同一份简报两次 Opus 5.5 运行都没通过检查时才出场。`medium` 把每个角色降一档，`small` 降两档，两者都更省 token。

你只覆盖真正关心的角色。规则里没写行的角色保持技能默认值。想恢复默认，删掉那个角色的行。重跑 `/setup-pstack` 会保留任何模型与默认值不同的角色。默认值变了之后，改动之前写的规则仍会钉住旧默认值，直到你再跑一次 `/setup-pstack`。那一轮会替换它认得的旧默认值，告诉你换掉了哪些，并保留你的其他选择。要拔掉它不认得的钉子，先删掉那个角色的行或整个文件。

你可能想问用 Auto 会怎样。把角色设成 `inherit-parent` 或 `auto`，pstack 就省略子代理（subagent）的 `model` 字段，子代理于是继承你主对话的模型。两个取值含义相同，都不是模型 slug。面板角色的取值是列表，每个条目跑一个子代理，列表长度决定面板规模。配置还会设置 `swarm workers`，这是每个 `/swarm` 工作单元的默认模型，除非某场竞速给每个分支各自点名了模型。

## 验证提议，随你接不接受

配置收尾时，`/setup-pstack` 会在你的项目里找证明应用行为的办法，找 `verify-*` 技能或现成的工装（harness）。两样都没有，它会提议一次，用 [`/create-verification-skill`](../../skills/create-verification-skill/SKILL.md) 生成一个。

答应的话，它会写 `.cursor/skills/verify-<app>/`，一个项目内的技能，教代理（agent）像用户那样驱动你的应用。交接之前它会先证明这个技能确实管用一次。拒绝的话，配置继续往下走。你随时可以自己跑 `/create-verification-skill`。[验证并交付](./06-verify-and-ship.md#create-a-project-verification-skill) 有深入讲解。

你是 pstack 新手的话，答应它。能自查工作的代理会一直干到检查通过为止。不能自查的代理会把每个结果都丢回给你手工检查。本指南讲的所有东西里，验证技能回报最高。

然后配置会提议一次，用 [`/create-formal-verification-skill`](../../skills/create-formal-verification-skill/SKILL.md) 生成形式化验证技能，除非项目里已经有了。测试检查的是有人想到要写的用例。形式化验证技能检查你代码的一条关键性质，比如 "一个任务绝不会同时跑两遍"，覆盖声明范围内的每一种情况。你的代码有并发、重试、状态机或解析器时就答应，这类地方一个 bug 代价高昂，而测试很少碰到真正击穿的用例。否则拒绝，情况变了再跑。[验证并交付](./06-verify-and-ship.md#prove-a-property-with-a-formal-verification-skill) 有讲解。

配置完成后，开一个新对话。模型规则只对新会话生效。

## 控制成本

pstack 在子代理和评审面板上要多花 token。这是严谨的价钱。想少花：

- 重跑 `/setup-pstack`，选更小的推理预算或更便宜的模型。主对话用强模型，写代码的角色用更便宜更快的模型，这种划分不错。
- 把某个角色设成 `auto` 或 `inherit-parent`，让它跑在对话自己的模型上。
- 缩短面板列表。每个条目跑一个子代理。
- 把 `/poteto-mode` 留给需要严谨的工作。又小又明显的改动不需要。

<a id="run-your-first-task"></a>
## 跑你的第一个任务

挑一件真实但小的事，像跟同事说话那样描述它：

```text
/poteto-mode add a --json flag to this command. text output stays byte-identical. verify both.
```

盯着待办清单。开头几项就是匹配到的手册（playbook）的步骤原样抄进来，这条提示词（prompt）匹配的是 Feature 手册。

接下来你正常追问就行。想让 `/poteto-mode` 在整个对话里一直开着，就从 `/` 菜单选它，然后按 Option+Enter（Mac）或 Alt+Enter（Windows）代替回车。这会把它变成一种 [Custom Mode（自定义模式）](https://cursor.com/docs/skills#using-a-skill-as-a-custom-mode)，在你退出之前每一轮都留在上下文里。Custom Mode 在 Agents 窗口和 CLI 里都可用。单按回车只把技能挂到一条消息上，对话往后走它就淡出了。

下一章：[用 `/poteto-mode` 分派工作](./02-poteto-mode.md)。
