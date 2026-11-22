# pstack 在 Claude Code 中

pstack 的技能（skill）沿用 Cursor 的名字。在 Claude Code 里，每个会话先读一遍本文件，再用下表把技能里出现的每个 Cursor 名字映射过去。`<pstack>` 指放着 `skills/` 和 `docs/` 的那个文件夹。某个 Cursor 字段在表里没有对应行，就丢掉它并说明一声。绝不要传工具 schema 会拒绝的字段或值。

## 工具

| Cursor | Claude Code |
| --- | --- |
| `Task` | `Agent` |
| `AskQuestion` | `AskUserQuestion`。在 `-p` 模式下它不存在，改用纯文本提问。 |
| 打开待办清单，或把步骤抄进待办清单工具 | 会话提供 `TodoWrite` 就用它。否则把清单写在你的回复里，每个阶段更新一次。 |
| 运行另一个 pstack 技能（`/name` 或 "the **name** skill"） | 用 `Read` 打开 `<pstack>/skills/<name>/SKILL.md` 然后照做。`Skill` 工具会拒绝大多数 pstack 技能。让用户自己输入时，写 `/pstack:<name>`。 |
| `Grep`、`Glob` | 会话没有 `Grep` 或 `Glob` 时，用 `Bash` 跑 `rg`。 |

## 子代理字段

| Cursor | Claude Code |
| --- | --- |
| `subagent_type: generalPurpose` | `subagent_type: "general-purpose"` |
| `poteto-agent`, `comment-sicko` | `pstack:poteto-agent`, `pstack:comment-sicko` |
| `poteto-agent` 文件里的 `model:` 行 | 它放的是 Cursor slug。每次生成 `pstack:poteto-agent` 的 `Agent` 调用都要带上 `model` 和 `effort`，按模型一表映射，别让那个 slug 直达 API。 |
| `readonly: true` | `subagent_type: "Explore"`。它保留 `Bash`，所以简报里还要写明 "do not edit files"，事后你自己查 `git status`。 |
| `readonly: false` | `general-purpose`。它保留会话的 MCP 服务器。 |
| `environment: "cloud"` | `isolation: "worktree"`。说清工作单元检出的是哪个分支或提交。 |
| `environment: "local"` | 省略。 |
| `cloud_base_branch: <b>` | 告诉工作单元先检出 `<b>`。 |
| `run_in_background: true` | 省略。子代理（subagent）默认就在后台运行，完成时会通知你。 |
| `resume` | 生成一个新代理（agent），把之前的结果贴进它的简报。 |
| `file_attachments` | 在简报里列出路径。 |

## 模型

| Cursor | Claude Code |
| --- | --- |
| `claude-<family>-<version>-<effort>[-fast]`，例如 `claude-opus-5-5-xhigh` | `model: "opus"` 加 `effort: "xhigh"`。`model` 只接受 `opus`、`sonnet`、`haiku`、`fable`。丢掉 `-fast`。 |
| `gemini-*` 或其他非 Claude 的 slug | 省略 `model`，让代理跑在会话模型上，并说明一声。 |
| `auto` 或 `inherit-parent` | 省略 `model`。 |
| `pstack-models.mdc` 规则，`~/.cursor/rules/pstack-models.mdc` | `~/.claude/rules/pstack-models.md`。Claude Code 会把它载入每个新会话，所以各角色行已经在上下文里。不在的话就去读该文件。文件或该角色的行缺失时，用技能的默认 slug。 |
| 角色值 `<model> <effort>`，例如 `opus high` | `model: "opus"` 加 `effort: "high"`。 |

## setup-pstack（配置 pstack）

Claude Code 会把 `~/.claude/rules/` 里每个 `.md` 文件载入新会话，所以 `setup-pstack` 把规则写在那里。按它的步骤走，但做如下调整：

- 检测。检测到的模型就是 `Agent` 工具 schema 列出的 `model` 取值，每个再配上其 `effort` 取值中的一个档位。真实模型写成 `<model> <effort>`，例如 `opus xhigh`。档位是最后一个词，所以预算那一步照原样就能用。
- 默认值。按上面几行映射每个默认 slug：`claude-opus-5-5-xhigh-fast` 对 `opus xhigh`，`claude-opus-5-5-high-fast` 对 `opus high`，`claude-fable-5-2-high` 对 `fable high`，`gemini-4-neon-xhigh` 对 `inherit-parent`。
- 写入。写 `~/.claude/rules/pstack-models.md`，不写 frontmatter，然后是注释行、`# budget` 行，以及第 5 步里每个角色一行。整份文件覆盖重写。绝不要把角色行写进 `CLAUDE.md` 或 `AGENTS.md`。

## 路径

| Cursor | Claude Code |
| --- | --- |
| `.cursor/skills/`, `~/.cursor/skills/` | `.claude/skills/`, `~/.claude/skills/` |
| `~/.cursor/plugins/`, `.cursor-plugin/plugin.json` | `~/.claude/plugins/`, `.claude-plugin/plugin.json` |
| `.cursor/rules/*.mdc`，总是生效的规则 | `CLAUDE.md`，没有 `CLAUDE.md` 时用 `AGENTS.md`。Claude Code 从不读 `.mdc` 文件。模型规则是唯一例外。见模型一节。 |
| `https://github.com/cursor/plugins/blob/main/pstack/<path>` | 该文件存在时就是 `<pstack>/<path>`。插件自带技能、`docs/guide/` 和 `README.md`。读本地拷贝，绝不要去抓取。用户需要链接时把 URL 给他们。 |
| 转录（transcript）在 `~/.cursor/projects/<slug>/agent-transcripts/` | `~/.claude/projects/<slug>/<session>.jsonl`。slug 是工作区路径把每个 `/` 和 `.` 换成 `-`，所以 `/Users/you/proj` 就是 `-Users-you-proj`。该文件夹下最新的 `.jsonl` 就是当前会话。它的 `<session>/subagents/` 文件夹放子代理转录。只读这个工作区的文件夹。 |

## 仅 Cursor 有的功能

Claude Code 没有 Cursor Automations、Bugbot、Cursor 云端代理，也没有内置 PR 工具。某一步需要它们时，在回复里放一行 `Skipped in Claude Code: <step>. <reason>.`，然后继续下一步。不要停下、提问或重试。后面的步骤要用到被跳过那步的产出（比如某个 URL 或密钥）时，留个占位符并在这一行里点名。

| Cursor | Claude Code |
| --- | --- |
| Cursor Automations：例程、`update_state` 例程调用、`api2.cursor.sh/automations` webhook、例程唤醒和 `SendToUser` 密钥请求 | 用那一行跳过。 |
| Bugbot | GitHub PR 上的 Bugbot 评论就是普通的评审线索，用该步骤给出的 forge 命令分类处理。PR 上没有 Bugbot 检查或评论时，跳过启动、等待或配置 Bugbot 的步骤。 |
| Cursor 云端代理 | `environment: "cloud"` 映射为工作树（worktree），见子代理字段一表。需要云端代理本体的步骤（比如它的仪表盘状态或它自己的机器）直接跳过。 |
| 内置 PR 工具 | 用该步骤给出的 forge 命令，比如 `gh pr create`。这不算跳过，所以不写那一行。 |
