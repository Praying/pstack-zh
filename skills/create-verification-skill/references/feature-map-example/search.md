# 搜索笔记

搜索让用户能按标题或正文找笔记、查看一条匹配的笔记、并区分没有匹配和搜索不可用这两种情况。

## 子功能

- `search-open` 从每个受支持的浏览器入口打开搜索。
- `search-match` 返回标题和正文的匹配，不改动笔记数据。
- `search-open-result` 在笔记编辑器里打开一条结果。
- `search-empty` 为没有匹配的查询显示完整的空状态。
- `search-clear` 移除查询，恢复最近笔记视图。
- `search-cli` 从终端返回同样的匹配笔记。

## 怎么到达它（用户视角）

- 在浏览器工具栏里点 `Search` 按钮。
- 焦点不在可编辑字段里时，在浏览器里按 `/`。
- 在终端里跑 `notes search <query>`。

## 用 control-notes 驱动它

前置条件：

- Notes 在 `http://127.0.0.1:4173` 上是健康的。
- 一次性数据目录里有 `Quarterly plan`，正文是 `Draft budget`。
- `control-notes doctor` 报告的 URL 和数据目录符合预期。

- **工具栏入口。** 点 `Search` 按钮。跑 `control-notes browser click --role button --name "Search"`。一个名为 `Search notes` 的对话框出现，焦点落在它的搜索框里。
- **键盘入口。** 关掉对话框，聚焦页面，按 `/`。跑 `control-notes browser press --key "/"`。同一个对话框出现，页面没有插入斜杠。
- **标题匹配。** 输入 `quarterly`。跑 `control-notes browser fill --role searchbox --name "Search notes" --value "quarterly"`。`Search results` 列表包含 `Quarterly plan`，不包含 `Grocery list`。
- **正文匹配。** 把查询换成 `budget`。跑 `control-notes browser fill --role searchbox --name "Search notes" --value "budget"`。结果 `Quarterly plan` 仍然可见，带着一段正文匹配的摘录。
- **打开结果。** 点 `Quarterly plan`。跑 `control-notes browser click --role link --name "Quarterly plan"`。对话框关闭，编辑器标题读作 `Quarterly plan`。
- **空状态。** 重新打开搜索，输入 `volcano`。跑 `control-notes browser fill --role searchbox --name "Search notes" --value "volcano"`。搜索完成后出现名为 `No matching notes` 的状态。
- **清除查询。** 点 `Clear search`。跑 `control-notes browser click --role button --name "Clear search"`。搜索框空了，`Recent notes` 区域取代了结果列表。
- **CLI 匹配。** 从终端搜索。跑 `control-notes cli -- notes search "quarterly" --format json`。退出码 `0`，stdout 里有一个对象，标题是 `Quarterly plan`。
- **CLI 未命中。** 搜一个不存在的值。跑 `control-notes cli -- notes search "volcano" --format json`。退出码 `0`，stdout 是 `[]`。
- **证明。** 抓下填满结果的状态。跑 `control-notes browser snapshot --aria --path artifacts/search/results.aria.txt` 和 `control-notes browser screenshot --path artifacts/search/results.png`。两个产物都能认出 Notes、查询词、和 `Quarterly plan`。

## 坑

- 编辑器或搜索框有焦点时按 `/`，插入的是字符，不会打开搜索。
- 结果在一小段防抖之后才更新。等结果列表或空状态出现，不要靠固定时长的 sleep 去等。
- 归档的笔记默认被排除，除非用户启用 `Include archived`。
- CLI 默认输出人类可读格式。要稳定的断言就用 `--format json`。
- 打开一条结果会改变浏览器状态。证明另一个查询前先重新打开搜索。
