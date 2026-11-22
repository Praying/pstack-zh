---
name: setup-benny
description: "配置 Benny 并准备它的分诊与复现自动化。在安装 Benny 或更改其 Slack、追踪系统、仓库、路由、控制、模型或预算设置时使用。"
disable-model-invocation: true
---

# 设置 Benny

Benny 以休眠的自动化包（automation pack）形式随 pstack 发布。插件清单只暴露 pstack 常规的技能（skill）根目录。这份文件和两份操作文件都不是斜杠技能。

人把 Cursor 指向包里的 `FOR_AGENTS.md` 进入安装流程。引导流程把整个包复制进目标仓库，然后直接在 `.cursor/automations/benny/skills/setup-benny/SKILL.md` 读这份文件。

Benny 需要外部配置和两个正式运行的 Cursor 自动化。

用户没明确要求之前，不要创建或修改任何自动化。绝不要把密钥值放进插件文件、提示词（prompt）或提交的配置里。

## 1. 复制包并启用共享的 pstack 技能

在向用户询问 Benny 配置、在调用内置的 `/automate` 技能之前，先做这一步。

问清楚哪个仓库跑这些自动化。源包是包含 `FOR_AGENTS.md` 的目录。目标位置是 `<target-repository>/.cursor/automations/benny/`。

把整个源包合并进目标位置：

1. 目标位置不存在就创建。
2. 把每个源文件复制到相同的相对路径。
3. 保留只在目标位置存在的文件。安装或刷新期间绝不要删除无关文件。
4. 用户自有的配置、功能地图和路由地图放在目标位置之外。绝不要覆盖它们。
5. 已有的受管源文件有差异时，检查 diff 再合并，不要丢掉本地修改。归属不明确就停下来，替换之前先问。
6. 确认目标位置里有 `FOR_AGENTS.md`、这份安装文件、两份操作文件、它们的参考文件和模板。

如果这份文件已经在从目标位置读取，就视为复制完成，继续之前跑同样的检查。

把 pstack 加进目标仓库的 `.cursor/settings.json`。文件或 `.cursor` 目录不存在就创建。

把这个条目合并进已有的 JSON 或 JSONC：

```json
{
	"plugins": {
		"pstack": { "enabled": true }
	}
}
```

保留所有无关的顶层设置和其他插件条目。`plugins.pstack` 已存在就只改它的 `enabled` 值。文件用 JSONC 就保留注释和合法的 JSONC 语法。编辑后校验文件。

重新加载目标项目，或在那里起一个全新的代理（agent）。确认这些共享的 pstack 技能从项目作用域解析出来：

- `why`
- `tdd`
- `unslop`
- `principle-separate-before-serializing-shared-state`
- `principle-minimize-reader-load`
- `principle-guard-the-context-window`
- `principle-sequence-verifiable-units`
- `principle-fix-root-causes`
- `principle-prove-it-works`

从当前会话或用户作用域插件加载的技能不算数。检查必须显示目标仓库里的全新代理通过项目设置拿到 pstack。

项目作用域的插件安装不可用，或任何共享依赖解析不出来，就停下来说明失败原因。

Benny 的文件直接从 `.cursor/automations/benny/` 读。不要把那个目录加进插件清单，也不要指望它的 `SKILL.md` 出现在斜杠技能列表里。

告诉用户 `.cursor/settings.json`、`.cursor/automations/benny/` 和任何被引用的无密钥配置必须先提交，两个自动化才能启用。用户没要求就不要提交它们。

这一步检查通过后，正式运行的自动化的提示词就可以按稳定的仓库相对路径读已提交的操作文件。它们不得内嵌插件缓存路径，也不得复制文件内容。

## 2. 调整配置

打开这些复制过来的示例：

- `../../templates/configuration.example.yaml`
- `../reproduce-and-fix-issues/references/feature-map.example.md`

在 `.cursor/automations/benny/` 之外创建用户自有的副本。这些是配置文件，不是包文件。示例位置：

- 项目配置，例如 `.cursor/benny/configuration.yaml`
- 项目功能地图，例如 `.cursor/benny/feature-map.md`
- 项目路由地图，例如 `.cursor/benny/routing.md`
- 用户配置，例如 `~/.config/benny/configuration.yaml`
- 用户功能地图，例如 `~/.config/benny/feature-map.md`

自动化可能复现的每个面向用户的功能，都要在功能地图里填一个小节。保持用户的视角。不要把实现细节或当前代码路径冻结在地图里。

不要编辑复制过来的示例。包刷新可以在冲突审查后更新受管源文件，但绝不能碰用户自有的副本。

新的自动化检出必须读这些文件时，优先用目标仓库里已提交的无密钥文件。否则就把需要的值转述进正式提示词。只有内置的 `/automate` 技能确认文件已提交在自动化运行的那个仓库里，才能引用仓库文件。

已提交的包文件和配置文件用稳定的仓库相对路径。绝不要从正式运行的自动化引用插件源目录或插件缓存路径。

## 3. 填写必需的选项

询问或确认：

- 源 Slack 频道 ID
- 可选的运维或状态频道 ID
- 仓库 URL 和默认分支
- 分诊（triage）身份或 Slack 用户 ID
- 问题追踪系统的类型、团队、项目、标签和接收状态
- 追踪系统适配器技能或 MCP 动作
- 可选的路由地图路径
- 必需的控制技能名
- 必需的用户功能地图路径
- 状态 emoji 字符串
- PR 的 URL 格式
- 轮询和工作量预算
- 分诊、复现、代码工作和媒体审查的模型 slug

只用用户 Cursor 模型选择器或支持列表里显示可用的模型 slug。不要猜 slug，也不要沿用私有默认值。

源频道、分诊身份、仓库、追踪适配器、控制技能和功能地图必须明确。任何必需的值含糊不清，就判定安装失败。

保存之前，用 pstack 的 `unslop` 技能过一遍最终的自动化名称、描述和提示词垫片。

## 4. 检查集成能力

分诊自动化需要：

- 对配置的源 Slack 频道及其线程的读权限
- 在该频道回线程的权限
- 报告带媒体时的附件元数据和文件下载权限
- 通过配置的问题追踪适配器做搜索、读取、创建和更新的权限

复现自动化需要：

- 对源线程的读权限
- 在源频道回线程的权限
- 在配置的运维频道发帖和编辑的可选权限
- 仓库读取和历史访问权限
- 一个能开草稿 PR 的 PR 动作
- 配置的控制适配器（control adapter）技能

读取和发帖优先用配置的 Cursor Slack 动作。可选的 `BENNY_SLACK_BOT_TOKEN` 可以补一个小缺口，比如编辑一条运维状态消息或下载一个附件。值存在密钥管理器或环境变量里，不要写进 YAML。

不要用没写进文档的集成端点。

## 5. 准备路由地图

用户想要改派或提醒负责人时：

1. 把 `../triage-issue-reports/references/routing.example.md` 复制到 `.cursor/automations/benny/` 之外。
2. 把每个占位符换成公开的或组织内部的值。
3. 给负责人发提醒默认关着。
4. 只对配置的功能负责人或确认很可能的回归作者发提醒。

没配路由地图时，分诊可以给报告分类，但绝不能猜去向或负责人。

## 6. 验证控制适配器

读 `../reproduce-and-fix-issues/references/control-adapter.md` 和用户填完的功能地图。

确认指定的技能能做到：

- 把目标应用拉起来
- 走真实 UI 到每个已映射的功能
- 用声明的适配器动作遍历已映射的状态
- 查看状态而不强改结果
- 截图
- 开始和停止录屏
- 清理自己的进程和临时数据

缺任何一项能力，就让复现自动化保持禁用。它必须在失败时保持关闭，而不是宣称一次没做过的复现。

## 7. 准备正式自动化

问清楚是首次创建，还是配置已有的自动化。

从复制的包里读 `../../FOR_AGENTS.md`，它是两条路共用的主要用户意图来源。用它来理解两个触发器、工具、指令、结果和共享规则。

### 首次创建

一次只建一个自动化。

每个自动化按下面来：

1. 读对应的复制过来的提示词模板，作为次要的内部素材。
2. 把 `FOR_AGENTS.md`、填好的 Benny 配置和模板意图，转成一份完整的自然语言请求。
3. 让正式提示词去读并遵循 `.cursor/automations/benny/` 下它那份准确的已提交操作文件。
4. 用稳定的仓库相对路径，不用插件源目录或缓存路径。不要把操作文件内容复制进正式提示词。
5. 读并遵循内置的 `automate` 技能。
6. 让 `automate` 去发现 Slack 频道、仓库和已连接的集成。
7. 让 `automate` 确认复制的包和所有被引用的配置文件，已提交在自动化将要运行的同一个仓库里。
8. 让 `automate` 展示它的草稿表，拿到批准，询问就绪状态，打开 Automations 编辑器。
9. 把这个自动化的编辑器交接做完，再开始下一个。

把下面这份从配置填好的完整分诊意图交给 `automate`：

- 名称 `benny-triage`。
- 每次运行都读并遵循 `.cursor/automations/benny/skills/triage-issue-reports/SKILL.md`。
- 配置的源 Slack 频道里每出一条新的顶层报告就触发。
- 读触发的那条线程，只在它里面回复。
- 用配置的问题追踪集成。
- 分类、查证据、追成因、查重，只对清楚的新缺陷（bug）建单。
- 以一条只发在线程里的裁定（verdict）收尾，带配置的 `[benny:bug]`、`[benny:performance]` 或 `[benny:other]` 标记，以及可选的追踪系统 URL。
- 绝不在源频道发根消息。

分诊的编辑器交接完成后，把下面这份完整的复现与修复意图交给 `automate`：

- 名称 `benny-reproduce`。
- 每次运行都读并遵循 `.cursor/automations/benny/skills/reproduce-and-fix-issues/SKILL.md`。
- 在配置的源 Slack 频道里对同样的新顶层报告触发。
- 用配置的仓库和默认分支。
- 读源线程，只在它里面回复。
- 包含创建 PR 以及配置的追踪系统、控制适配器和功能地图要求。已映射的用户路径和状态用转述，除非 `automate` 确认同一个仓库里有合格的已提交文件。
- 行动之前先等可信的分诊标记。
- 走已映射的真实 UI 把确切的症状复现两次，并捕捉证据。
- 验证已有的修复，不要另写盖过它。
- 只有复现确认后才尝试可选的有界修复，证据和检查通过了就开草稿 PR。
- 绝不在源频道发根消息。

不要重复 `automate` 在 Slack、仓库、集成、完整性、认证、草稿审查、批准、就绪和编辑器交接上的工作。

### 已有的自动化

内置的 `automate` 技能只能用来创建。不要用它搜索、查看或更新已有的自动化。

做完配置、路由、控制适配器和功能地图的检查。然后把这份简明的编辑器清单交给用户。

已有的分诊自动化，更新：

- 名称和描述
- 直接读 `.cursor/automations/benny/skills/triage-issue-reports/SKILL.md` 的指令
- 新的顶层 Slack 报告触发器和源频道
- Slack 线程的读取和回复能力
- 问题追踪集成
- 转述的分诊指令、只在线程里的规则和 Benny 裁定标记

已有的复现自动化，更新：

- 名称和描述
- 直接读 `.cursor/automations/benny/skills/reproduce-and-fix-issues/SKILL.md` 的指令
- 对应的 Slack 触发器和源频道
- 仓库和默认分支
- Slack 线程的读取和回复能力
- PR 动作
- 追踪系统、控制适配器和功能地图要求
- 转述的等标记、证据、验证和有界修复指令

请用户直接在 Automations 编辑器里更新每个已有自动化。不要建替代品或重复的自动化。

### 创建的边界

绝不要直接调用自动化后端服务或后端自动化工具。绝不要用带草稿字段的浏览器 URL。绝不要构造或打开 Cursor 协议深链接。新建自动化唯一的收尾路径，是内置 `automate` 技能那套经过审查的 Automations 编辑器交接。

编辑器保存后线程安全测试通过之前，两个自动化都不要启用。

## 8. 测试线程安全

用测试频道或无害的测试报告。

测试前，确认目标仓库的 `.cursor/settings.json`、`.cursor/automations/benny/` 和所有被引用的无密钥配置文件，已提交在自动化检出所用的分支上。确认两份正式提示词都指向它们准确的已提交操作文件。任何一项检查不过就停。告诉用户自动化还不能启用。

验证：

1. 分诊存下根 `thread_ts`，只发一条裁定作为回复。
2. 裁定里带一个配置的标记。
3. 复现只接受来自配置的分诊身份的标记。
4. 复现保持同一份不可变的源坐标。
5. 源频道不出现根消息。
6. 委派出去的工作单元（worker）用不了任何 Slack 写动作。
7. 缺坐标、父消息被删或预检（preflight）失败时，不发帖也不建追踪单。

七项检查全部通过后才放开正式流量。
