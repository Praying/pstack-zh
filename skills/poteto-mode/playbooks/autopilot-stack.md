### Autopilot-stack（自动堆栈）

跑 Autopilot-full 手册（playbook），但不合并。

1. **代替第 5 步，停在已验证。** 每个 PR 在当前 head 上验证过且绿了就停，然后把 PR 交给操作者。交接之后，没有操作者的放行，任何人不许推送。循环（loop）每一拍（tick）的提示词（prompt）都点名这个文件，所以每一拍都看得到不合并的规则。
2. **只有当一个 PR 建在另一个 PR 之上时，才把它堆进栈。** 这是对第 3 步唯一的例外。按 `playbooks/opening-a-pr.md`，把这个 PR 的 base 放到另一个 PR 的分支上，代替主干（trunk）。代替第 2 步里向主干的变基（rebase），每个 PR 变基到父 PR 的分支上，栈根变基到主干上。下面的 PR 出了新 head，就要把它上面的 PR 重新堆一遍，而 `git patch-id` 在重堆后没变的 PR 保留它的裁定（verdict）。

**回复：** 列出每个 PR 和它的裁定，代替合并了什么那一项。
