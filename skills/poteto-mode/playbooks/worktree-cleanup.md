### Worktree and simulator cleanup（工作树与模拟器清理）

1. 快照并审计。记下 `df -h /`，然后跑 `scripts/worktree-audit.sh [-b <pr-base-ref>] [repo-path]`（principle-build-the-lever）。不带 `-b` 时它拿 `MERGED` 对照 `origin/HEAD` 检查，所以 PR 指向别的分支时要传 `-b origin/<branch>`。它从 `git worktree list` 读路径，从不手打，因为手打的 `myrepo-worktrees/x` 会漏掉住在 `.cursor/worktrees/myrepo/x` 的那个（principle-encode-lessons-in-structure）。它给每个工作树（worktree）按大小、年龄、合并状态、未提交的工作、PR 状态、以及碰过它的最新对话分门别类，然后建议一个桶。转录（transcript）扫描很慢，放到后台跑。
2. 桶只是建议，不是许可。被钉住的和活跃的对话才是真实产物（principle-prove-it-works）。从用户或侧边栏拿到那套名单，逐个交叉核对候选。杠杆（lever）曾把用户钉住的工作树标成 `safe`，所以钉住的名单说了算。
3. 删除之前先验证使用情况。每一个 `verify-recent-chat` 行、以及任何你拿不准的东西，都把子代理（subagent）扇出去读转录，报告该对话是被钉住的还是进行中的、以及它碰了哪些工作树（principle-guard-the-context-window，转录都是大块内容）。被钉住的对话会通过后台子代理把 arena 树和复现树生成到兄弟工作树里，就算它们的名字从没出现在侧边栏上，它们也在使用中。照子代理（Subagents）一节给每个子代理挑推理档位。
4. 遇到不可逆的损失就暂停。`wip:N` 是 N 个已跟踪的未提交编辑。先亮出 diff、拿到决定再动手，因为删一个干净的工作树还能从它的分支恢复，未提交的工作却会没了。`scratch:N` 是未跟踪的一次性东西，可以安全删掉，但要点名这些文件。按自主性（Autonomy）一节，干净、已合并、又没人用的可以直接清。`wip` 的和在用的要暂停。
5. 修剪确认过的那批。逐个路径执行 `git worktree remove --force <path>`。目录因为被忽略的构建产物还赖着，就 `rm -rf` 它，然后 `git worktree prune`。分支引用会活下来，所以不会有提交（commit）丢失。用 `df -h /` 确认，再重新列一遍。
6. 模拟器和其他回收项。模拟器通常是接下来最大的一块收益。`xcrun simctl --set testing delete all`（XCTestDevices 的克隆）、`xcrun simctl delete unavailable`、以及先 `xcrun simctl runtime list` 再 `runtime delete <id>` 清掉旧运行时。需要时还有更多：Xcode 的 `DerivedData` 和 `iOS DeviceSupport`、`~/Library/Application Support/Cursor`（`state.vscdb.backup`，以及 `snapshots/roots/<root>`，后者里以你打开过的某个工作区文件夹命名的 `<root>` 会膨胀起来）、包缓存（pnpm、uv、brew、yarn）。只清用户没说要留的缓存。

这是唯一一本会删用户状态、又没有代码审查来兜住失误的手册（playbook），所以上面这些检查关卡（gate）就是审查。

**回复：** 前后两次的 `df -h /`、回收的空间、清掉的工作树、以及每个被留下的一行理由（被哪个对话占用，或有未提交的工作）。
