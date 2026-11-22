# benny

benny 给你两个处理 Slack 问题报告的 Cursor 自动化。一个给每份报告分诊。另一个复现已确认的缺陷，可以准备一个小的草稿修复。

这个目录里的文件是休眠的设置与自动化来源。它们不会以斜杠技能（skill）的形式出现。

## 设置

1. 让 Cursor 指向 [`FOR_AGENTS.md`](./FOR_AGENTS.md)，并说出目标仓库。
2. 让设置流程把这个目录整体合并到目标的 `.cursor/automations/benny/`。它必须保留只存在于目标位置的文件，冲突要审查，而不是覆盖本地改动。
3. 让设置流程在目标仓库的 `.cursor/settings.json` 里为共享依赖启用 pstack：

```json
{
	"plugins": {
		"pstack": { "enabled": true }
	}
}
```

4. 把用户自有的配置放在复制出的包（pack）之外，例如 `.cursor/benny/`。按需改写 [`configuration.example.yaml`](./templates/configuration.example.yaml) 和 [`feature-map.example.md`](./skills/reproduce-and-fix-issues/references/feature-map.example.md)。
5. 在启用任一自动化之前，先提交 `.cursor/settings.json`、`.cursor/automations/benny/`，以及所有不含密钥的配置。
6. 在各自的编辑器里审查每份新的自动化草稿，或更新已有自动化。然后发一份无害的测试报告，验证源频道里的每条消息都留在原线程内。
