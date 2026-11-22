### Shipping（交付）

**你拥有落地的一切。每个 PR 独立验证，只落地从栈底生根的已验证连续段（run），然后把手拿开，别碰这个栈。**

1. **先定下托管平台，然后在每个 PR 落地之前独立验证它。** 默认用 GitHub CLI（`gh`）。`command -v origin` 成功、且 Origin 能解析这个仓库时，所有 PR 操作都用 `origin pr`。否则留在 `gh` 上，并记录这次回退。永远不要求 Graphite（`gt`）。每个 PR 一名验证者，每名都是 Cursor 云代理，用配套的控制技能（比如 `cursor-team-kit` 的 `control-ui` 或 `control-cli`）像用户一样跑这个改动，parent 对 head，然后在那个 PR 上贴出 `PASS`、`PASS+NOTES` 或 `FAIL`。只有没写这段代码的代理（agent）给出的裁定（verdict）才算数，因为作者检查的是它本来想造出来的东西。绿的 CI 和机器人审查给出的批准都不算裁定，因为它们都不像用户那样跑这个改动。
2. **只落地以栈底为根的、连续已验证的连续段。** 从最低的未合并 PR 往上走，停在第一个没有 `PASS` 或 `PASS+NOTES` 的 PR 上。它下面的 PR 构成连续段，最上面那个就是天花板。已验证的 PR 压在未验证的上面，就得等，因为它没有下面的代码就落不了地。分支上既没有也不需要那段代码的 PR，可以挪到主干（trunk）上独立落地。但它要先拿一个新裁定，因为旧裁定是在那段代码垫在下面时跑的，而主干上没有它。
3. **复查裁定是否仍然描述这份代码。** 裁定覆盖的是一个 patch，而链式变基（rebase）会把它上面的每个 PR 挪到新的 SHA 上，且不让任何检查失败。在信任一个较旧的裁定之前，把裁定 SHA 处的 `git patch-id` 和当前 head 对比一下。当两个 patch 只在测试、文档或 lint 配置上有差别时，只要某条通道跑的东西在两个 SHA 上构建结果除了噪声外相同，它的结果就成立。噪声就是那种在裁定 SHA 的两次构建之间也会冒出来的差异，或者干脆是一个内嵌的提交（commit）SHA。每类噪声都要连同涉及的文件报告出来，让读者能核查这个判断。改动本身仍然要跑新检查、过一遍审查，因为旧裁定从没见过它。像 dev server 这种没有构建输出的通道证明不了这一点，所以要重跑它，其他漂移了的也都要重新验证。
4. **自底向上落地，一次一个 PR。** 永远先合并连续段里最低的未合并 PR，用 `origin pr merge <pr>` 或 `gh pr merge <pr> --squash`。在 Origin 上，这条裸命令还会一并合并你点名的那个之下的每一个 PR；在 GitHub 上，base 还挂在父分支上的 PR 会合并进那个分支，而不是主干。把 PR 读回来确认结果（`origin pr view <pr>` 或 `gh pr view <pr> --json state,mergedAt,baseRefName`），因为合并命令可能报成功，而合并队列只是把 PR 排进了队，请求了 auto-merge 也不等于已合并。落地的 PR 读回来应该是已合并，base 是主干，不是父分支。只对裁定覆盖其当前 head 的 PR 启用 `--auto`（Origin 的 merge-when-ready 或 GitHub 的 auto-merge），因为它落地的是当下就绪的 head。一次合并落地后，把 base 还指着被合并分支的 PR 改指到主干（`origin pr edit <pr> --base <trunk>` 或 `gh pr edit <pr> --base <trunk>`），把它的 patch 挪到合并结果之上，再用 `--force-with-lease` 推上去。裁定覆盖的是 patch，不是它的提交，所以第 3 步的 patch-id 规则决定裁定能不能幸存。它的检查过了、裁定也幸存了，就合并它。
5. **栈在落地期间，别碰它。** 第 4 步挪动下一个 PR 是唯一允许的对分支或 base 的改动，因为一次合并落地的是那一刻它的分支和 base 上装着的一切，不管有没有裁定。在 GitHub 上，在 `/loop` 下用 `scripts/watch-pr/watch-pr --queued-stack` 盯着连续段，直到它在天花板处报出 `COMPLETE`。把连续段的 PR 编号按从底到顶的顺序传进 `--stack-prs`，每次重启监视器都传同一份列表。在 Origin 上，每次 `origin pr checks <pr> --watch` 返回时，都用 `origin pr view <pr> --checks` 重读当前 PR，直到天花板读出来是已合并。父分支合并后 base 自动挪到主干是预期行为，不是要修的损伤。合并卡住了，先弄清原因，再动任何东西。
6. **停在天花板处。** 延长连续段需要新的裁定，所以那是重走一遍第 1 步。

**回复：** 已验证的连续段和它的天花板、每个 PR 的裁定及出自谁手、落了什么地以及你怎么确认的、下一个缺口需要什么。
