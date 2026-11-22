# 复现自动化提示词

> 这是复制出的设置流程的素材来源。在 `automate` 确认复制出的包（pack）已提交到自动化将要运行的仓库之后，把这份意图转述进内置 `automate` 的草稿里。

本次运行请读取并遵循 `.cursor/automations/benny/skills/reproduce-and-fix-issues/SKILL.md`。

配置来源。只有当这个仓库相对路径已提交到同一个目标仓库时才包含它。否则就把配置的值转述出来。绝不要使用插件源路径或缓存路径：

```text
{{BENNY_CONFIG_PATH}}
```

触发条件：

```json
{
	"source_channel_id": "{{SLACK_CHANNEL_ID}}",
	"message_ts": "{{SLACK_MESSAGE_TS}}",
	"thread_ts": "{{SLACK_THREAD_TS_OR_EMPTY}}"
}
```

创建意图里要把这件事描述成配置的源 Slack 频道里一条新的顶层报告（top-level report）。其中要包含配置的仓库、默认分支、工单追踪系统、控制适配器（control adapter）、功能地图（feature map），以及草稿 PR 创建能力。

把源频道和根线程时间戳视为不可变。任何一项缺失或与配置不符，就停下，不要发帖。

在这个确切线程里，等待来自配置的分诊身份的分诊标记（triage marker）。只有出现 `[benny:bug]` 或 `[benny:performance]` 才继续。

尝试复现之前，先要求配置的控制适配器技能就位。通过真实 UI 把那个有区分度的确切症状复现两次。验证已有的 PR 或提交，而不是另写改动盖过它们。只有在复现确认、并且运维文件的修复检查关卡（gate）通过之后，才能尝试一次有边界的修复。

协调者是唯一的 Slack 发帖者。每份子代理（subagent）的提示词（prompt）都必须禁止 `SendSlackMessage`、`PostToSlack`、`chat.postMessage`，以及其他一切 Slack 写操作。子代理只返回发现。

绝不在源频道发根消息（root message）。
