# 创建一条笔记

创建笔记让用户能从浏览器或 CLI 保存一条带标题的笔记、取消一份没写完的草稿、并从第二个面向用户的视图确认保存的笔记。

## 子功能

- `create-open` 从每个浏览器入口打开一个空白编辑器。
- `create-save` 持久化标题和正文。
- `create-cancel` 丢弃一份没写完的浏览器草稿。
- `create-cli` 从终端创建同样形状的笔记。

## 怎么到达它（用户视角）

- 在浏览器工具栏里点 `New note` 按钮。
- 焦点不在可编辑字段里时，在浏览器里按 `n`。
- 在终端里跑 `notes create --title <title> --body <body>`。

## 用 control-notes 驱动它

前置条件：

- Notes 在 `http://127.0.0.1:4173` 上是健康的。
- 没有标题为 `Release checklist` 的笔记。
- `control-notes doctor` 报告的 URL 和一次性数据目录符合预期。

- **打开编辑器。** 点 `New note`。跑 `control-notes browser click --role button --name "New note"`。一个名为 `Note editor` 的表单出现，焦点落在 `Title` 文本框里。
- **输入内容。** 敲入标题和正文。跑 `control-notes browser fill --role textbox --name "Title" --value "Release checklist"` 和 `control-notes browser fill --role textbox --name "Body" --value "Tag and publish"`。`Save note` 按钮变成可用。
- **保存笔记。** 点 `Save note`。跑 `control-notes browser click --role button --name "Save note"`。一个名为 `Note saved` 的状态出现，标题读作 `Release checklist`。
- **确认持久化。** 回到笔记列表，重新打开那条笔记。跑 `control-notes browser click --role link --name "All notes"` 和 `control-notes browser click --role link --name "Release checklist"`。编辑器显示两个已保存的值。
- **取消草稿。** 打开一条新笔记，输入 `Discard me`，点 `Cancel`。跑 `control-notes browser click --role button --name "New note"`、`control-notes browser fill --role textbox --name "Title" --value "Discard me"`、和 `control-notes browser click --role button --name "Cancel"`。回到笔记列表，里面没有 `Discard me` 链接。
- **CLI 入口。** 创建第二条笔记。跑 `control-notes cli -- notes create --title "CLI note" --body "Created from terminal" --format json`。退出码 `0`，stdout 里有新笔记的 ID 和标题。
- **证明。** 从 `All notes` 重新打开两条保存的笔记。跑 `control-notes browser snapshot --aria --path artifacts/create-note/list.aria.txt` 和 `control-notes browser screenshot --path artifacts/create-note/list.png`。产物里能看见 `Release checklist` 和 `CLI note`。

## 坑

- 文本框有焦点时按 `n`，打出的是这个字符，不会打开新编辑器。
- 标题保存时会被裁剪。断言渲染出来的标题，不是草稿输入值。
- 只有保存状态不算充分证明。要从列表重新打开那条笔记。
- 在夹具清理时删掉 `Release checklist` 和 `CLI note`，但保留它们的证明产物。
