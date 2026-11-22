# benny 自动化意图

## 我想自动化什么

我想要两个相互配合的 Cursor 自动化，运行在同一个 Slack 问题频道里。

### 自动化 1：分诊问题报告

- 触发条件：当有人在我配置的源 Slack 频道里发一条新的顶层报告（top-level report）时，我要这份自动化从那份报告启动，并保留它原来的线程坐标（thread coordinates）。
- 行为：我要它读取线程和附件，把报告归类为缺陷或性能问题、功能请求、问题或反馈、或转派（reroute），并在路由前追溯可能的归属层（owning layer）。
- 追踪系统（tracker）：我要它在我配置的追踪系统里搜索重复项，有把握时更新那个重复项，只对明确的全新缺陷创建工单。
- 工具：我要 Slack 线程的读取和回复权限、我配置的追踪系统集成，以及我可选的路由映射（routing map）。
- 结果：我要源线程里恰好一条回复，带一句简短裁定和 `[benny:bug]`、`[benny:performance]` 或 `[benny:other]` 之一。缺陷或性能标记可以附上追踪系统的 URL。
- 边界：我绝不想要这份自动化在源频道里发根消息（root message）。

### 自动化 2：复现并修复已确认的缺陷

- 触发条件：我要这份自动化从同一条新的顶层报告启动，或从设置时选定的另一种受支持触发方式启动，然后等待原线程里出现可信的分诊标记（triage marker）。
- 检查关卡（gate）：我要它在有人明确接手修复时停下。如果某个已有的 PR 或已合并的提交可能已经修掉了这份报告，我要验证，而不是做一个竞争性改动。
- 行为：我要它使用我配置的控制适配器（control adapter）和功能地图（feature map），通过真实 UI 把确切症状复现两次，并截屏、录屏，做一次只读状态交叉核对。
- 修复：我要它验证已有的 PR，而不是另写改动盖过它们。复现确认之后，它可以尝试一次有边界的根因修复，测试成本低时用 TDD，对影响范围做冒烟测试，只有前后对比证据通过才开草稿 PR。
- 工具：我要 Slack 线程的读取和回复权限、仓库和历史访问权限、草稿 PR 创建权限、我配置的追踪系统，以及我的控制适配器。
- 结果：我要证据和验证结果发到源线程或可选的运维线程（operations thread），外加一份可选的草稿 PR。更新要简明。
- 边界：我绝不想要这份自动化在源频道里发根消息。

### 共享规则

- 我要源频道和根线程坐标在整个运行期间保持不可变。
- 我把工具型和调试机器人当作证据来源，不当作委派对象，也不把修复所有权交给它们。
- 我允许子代理（subagent）帮忙，但它们不能向 Slack 发帖，也不能拿到 Slack 凭据。
- 我要这整包都提交到目标仓库的 `.cursor/automations/benny/`。它的 `SKILL.md` 文件是直接的自动化指令，不是注册过的插件技能（skill）。
- 我要 pstack 只通过目标仓库里已提交的 `.cursor/settings.json` 为共享依赖启用，例如 `why`、`tdd`、`unslop`，以及必需的原则（principle）技能。
- 我要每份线上自动化的提示词（prompt）直接读取已提交的运维文件。我不要插件缓存路径、复制的摘录，也不要斜杠技能发现。
- 我把用户自有的配置、功能地图、路由映射和密钥都放在 `.cursor/automations/benny/` 之外，这样包刷新不会覆盖它们。
- 当频道坐标、追踪系统访问、控制适配器或功能地图缺失或不确定时，我要两份自动化都按失效关闭（fail closed）处理。
- 我只要草稿 PR。不要合并，也不要部署。

### 我的配置

- 源 Slack 频道：`<channel>`
- 可选的运维频道：`<channel or none>`
- 仓库与默认分支：`<repo>`，`<branch>`
- 追踪系统：`<type, team, project, labels, intake status>`
- 路由映射：`<path or none>`
- 分诊身份：`<slack identity>`
- 控制技能：`<configured skill or adapter>`
- 功能地图：`<committed same-repo path outside the copied pack, or behavior to paraphrase>`
- 模型：`<triage, reproduce, code, media review>`
- 状态 emoji 字符串：`<seen, reproducing, reproduced, blocked, fixing, failed, pull request opened>`
- 预算：`<polling, verdict wait, follow-up, repro, rejection, fix>`
- 可选的机器人令牌能力：`<none, file download, or editable operations status>`

从 [`configuration.example.yaml`](./templates/configuration.example.yaml) 和 [`feature-map.example.md`](./skills/reproduce-and-fix-issues/references/feature-map.example.md) 开始。把它们复制到包外填写，例如 `.cursor/benny/` 下。密钥值放在密钥管理器或环境变量里。

## 致代理（agent）

人类通过让 Cursor 指向本文件进入设置流程。不要去找或调用搜索出来的 benny 斜杠技能。

1. 询问哪个仓库将运行这些自动化。
2. 把包含这份 `FOR_AGENTS.md` 的目录当作源包。
3. 把整个源包合并进 `<target-repository>/.cursor/automations/benny/`。
4. 保留每个只存在于目标位置的文件。绝不删除无关文件，也不覆盖用户自有的配置、功能地图或路由映射。
5. 当源管理路径上已有的目标文件与源不一致时，审查 diff 并合并，不要丢弃本地改动。如果归属不明确，先停下询问再替换。
6. 验证复制过去的 `FOR_AGENTS.md` 和 `skills/setup-benny/SKILL.md` 在目标仓库里存在。
7. 直接从目标仓库读取并遵循 `.cursor/automations/benny/skills/setup-benny/SKILL.md`。

我要你把这一条合并进目标仓库的 `.cursor/settings.json`：

```json
{
	"plugins": {
		"pstack": { "enabled": true }
	}
}
```

保留所有无关的设置和插件。如果现有文件用的是 jsonc，注释和合法的 jsonc 语法都要保留。

我要由一个以目标仓库为根的全新代理来做验证。确认 pstack 的 `why`、`tdd`、`unslop`，以及 benny 用到的原则技能都能在项目范围内解析。从当前会话或用户级安装加载的技能不算数。

如果项目级插件不可用，或任何共享依赖无法解析，就停下并说明失败原因。不要把 `.cursor/automations/benny/skills/` 加进插件清单，也不要指望它的文件出现在斜杠技能列表里。

告诉我在启用任一自动化之前，必须先提交 `.cursor/settings.json`、`.cursor/automations/benny/`，以及所有引用到的不含密钥的配置。在我明确要求之前，不要创建或更新任何自动化。

首次创建时，分诊用一次内置的 `/automate`，复现与修复再用一次。第一份自动化的草稿审查、批准、就绪检查和 Automations 编辑器交接全部完成之后，再开始第二份。

把这份意图和最终配置转述进每份草稿。分诊提示词必须读取并遵循 `.cursor/automations/benny/skills/triage-issue-reports/SKILL.md`。复现提示词必须读取并遵循 `.cursor/automations/benny/skills/reproduce-and-fix-issues/SKILL.md`。只有当 `/automate` 确认这些路径已提交到自动化将要运行的仓库之后，才能使用这些仓库相对路径。

对已有的自动化，不要用 `/automate` 查看或更新它们。先校验配置，然后用复制出的设置文件里的简明字段清单，让我能在各自的编辑器里直接修改每份自动化。不要创建重复项。
