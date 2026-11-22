### Opening a PR（开 PR）

在其他每本手册（playbook）的末尾调用。

在从主干（trunk）分出的 git 工作树（worktree）里干活，别把无关的工作混进来。变基（rebase）成小的、可落地的提交（commit）。提交之前用 `cursor-team-kit` 的 `/deslop` 过一遍 diff。

标题：`type(scope): subject`，改动落在某个真实符号上时就在标题里点它的名。正文：一份一分钟能读完、不超过 40 行的简报，按顺序带这些 `##` 标题，空的就删掉：`## Why`、`## What changed`、`## Scope`、`## Tradeoffs`（只写被否掉的备选，而且只写审查者否则会来问的那些）、`## Blast Radius`、`## Verification`。

后续 PR 按 base 分支堆叠。PR 就绪就开，走当前运行的 PR 工具，带 `draft: false`。没有就用 `gh pr create`、不带 `--draft`，Origin 的 CLI 能解析这个仓库时用 `origin pr create --status open`。开 PR 并不启动一轮 babysit。
