# 功能地图示例

把 Benny 可能复现的每个面向用户的功能都画进功能地图。驱动应用之前先读相关小节。这份地图保持在用户的视角。内部实现和当前代码路径在运行时去发现，不要冻结在这里。

把这份文件复制到 `.cursor/automations/benny/` 之外，例如 `.cursor/benny/feature-map.md`，再把 `control.feature_map_path` 指向副本。包（pack）刷新不得覆盖它。

## 每个功能的模板

### `<feature name>`

`<one-line user-visible purpose>`

#### 用户如何到达

- 点击路径：`<screen> -> <menu, tab, or panel> -> <control>`
- 键盘快捷键：`<shortcut or none>`

#### 控制适配器如何驱动它

- `<adapter action>` 配合 `<inputs>` 应该 `<visible result>`。
- 重置：`<how the adapter returns to a fresh state>`。

#### 稳定的选择器

- `<role and accessible name>`
- `<ARIA relationship>`
- `<data-component or purpose-named data attribute>`

绝不要用生成的 CSS 或 StyleX 类名、动态哈希、子元素下标，或脆弱的 DOM 位置。

#### 需要覆盖的状态

- 默认、悬停、focus-visible、激活、禁用
- 加载中、空、错误
- 选中、打开、展开
- `<relevant feature-specific variants>`

不适用的状态标出来。

#### 前置条件与准备

- 认证：`<account state>`
- 数据：`<fixture>`
- 权限：`<role>`
- 功能开关：`<flag or none>`
- 服务：`<required availability>`

#### 证据与交叉核对

- 截图：`<app identity, feature, and discriminating state>`
- 录屏：`<entry path, interaction, and final state>`
- 交叉核对：`<read-only state or value that confirms the UI>`

#### 陷阱

- `<known dead end or wrong surface>`
- `<safe environment translation>`

## 虚构示例

这些功能属于一个虚构的任务应用。它们是示例，不是必需的 Benny 功能。

### Sign in（登录）

让用户进入任务应用。

#### 用户如何到达

- 打开应用，选择 `Sign in`。没有快捷键。

#### 控制适配器如何驱动它

- `open_app`、`click Sign in`、`fill credentials` 和 `click Continue` 应该打开条目列表。
- 通过登出并清空一次性会话来重置。

#### 稳定的选择器

- 按钮 `Sign in`，文本框 `Email` 和 `Password`，`data-component="sign-in-form"`

#### 需要覆盖的状态

- 默认、focus-visible、提交中、禁用、加载中、错误

#### 前置条件与准备

- 一次性账号和可用的认证服务

#### 证据与交叉核对

- 把从落地页到条目列表的过程录下来。检查只读的会话状态。

#### 陷阱

- 营销页面是错误的界面（surface）。缺认证服务就是阻塞。

### Item list and detail（条目列表与详情）

让用户浏览条目并打开其中一条。

#### 用户如何到达

- 打开 `Items` 标签页，然后选择一行。

#### 控制适配器如何驱动它

- `select_tab Items` 和 `click <fixture item>` 应该打开它的详情。
- 通过关闭详情并清空选择来重置。

#### 稳定的选择器

- 名为 `Items` 的标签页和列表，以夹具（fixture）命名的行，`data-component="item-detail"`

#### 需要覆盖的状态

- 加载中、空、错误、选中、打开、展开

#### 前置条件与准备

- 具名的夹具条目、读权限、可用的条目服务

#### 证据与交叉核对

- 展示选中状态和匹配的详情标题。检查选中条目的 ID。

#### 陷阱

- 搜索结果可能长得很像，但走的是另一条路径。

### Item editor（条目编辑器）

让用户创建或编辑条目。

#### 用户如何到达

- 在详情里选 `Edit`，或在列表里选 `New item`。

#### 控制适配器如何驱动它

- `click Edit`、`fill <field>` 和 `click Save` 应该更新详情。
- 通过恢复夹具来重置。

#### 稳定的选择器

- 按钮 `Edit`、`New item`、`Save`，表单 `Item editor`，与标签关联的字段

#### 需要覆盖的状态

- 默认、focus-visible、有改动、校验中、禁用、保存中、错误、成功

#### 前置条件与准备

- 可编辑的夹具、写权限、可用的保存服务

#### 证据与交叉核对

- 展示从字段改动到更新后详情的过程。以只读方式检查存储的条目值。

#### 陷阱

- 不要注入表单状态。只读的详情字段不是编辑器。

### Settings（设置）

让用户改个人偏好设置。

#### 用户如何到达

- 打开个人资料菜单，然后选 `Settings`。

#### 控制适配器如何驱动它

- `open_menu Profile`、`click Settings` 和 `toggle <preference>` 应该更新该控件。
- 通过恢复起始偏好设置来重置。

#### 稳定的选择器

- 按钮 `Profile`，菜单项 `Settings`，区域 `Settings`，按用途命名的偏好属性

#### 需要覆盖的状态

- 关闭、打开、选中、focus-visible、禁用、加载中、错误

#### 前置条件与准备

- 已登录的测试账号、已知的偏好设置、可用的偏好服务

#### 证据与交叉核对

- 展示菜单路径和最终的控件状态。以只读方式检查偏好值。

#### 陷阱

- 操作系统设置是另一个界面。

## 完整性检查清单

- 每个可复现的面向用户的功能都有对应小节。
- 每个小节都写明用户路径、适配器动作和重置方式。
- 选择器用角色、名称、ARIA、稳定的组件标记，或按用途命名的属性。
- 没有选择器用生成的类名或 DOM 位置。
- 相关的交互、加载中、空、错误、选中和展开状态都被覆盖。
- 认证、夹具、权限、功能开关和服务都写明确。
- 截图、录屏和底层交叉核对的要求都写明确。
- 错误的界面、死胡同和安全的环境翻译都列出来。
- 实现细节留给运行时发现。
