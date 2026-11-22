# 分诊自动化提示词

> 这是复制出的设置流程的素材来源。在 `automate` 确认复制出的包（pack）已提交到自动化将要运行的仓库之后，把这份意图转述进内置 `automate` 的草稿里。

本次运行请读取并遵循 `.cursor/automations/benny/skills/triage-issue-reports/SKILL.md`。

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

创建意图里要把这件事描述成配置的源 Slack 频道里一条新的顶层报告（top-level report）。

把源频道和根线程时间戳视为不可变。任何一项缺失或与配置不符，就停下，不要发帖，也不要写工单追踪系统。

已提交的运维文件负责归类、附件审查、成因追溯、路由、查重、追踪系统写入，以及最终裁定。不要发进度消息。绝不在源频道发根消息（root message）。

协调者是唯一的 Slack 发帖者。任何被委派的工作单元（worker）都必须是只读的，只返回发现，并且明确禁止一切 Slack 写操作。

这条唯一的裁定消息结尾放且只放一个配置的标记：

```text
[benny:bug]
[benny:performance]
[benny:other]
```

缺陷或性能标记可以加上 `tracker=<URL>`。
