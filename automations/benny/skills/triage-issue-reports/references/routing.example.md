# 路由地图示例

把这份文件复制到 `.cursor/automations/benny/` 之外，例如 `.cursor/benny/routing.md`，换掉每个占位符。把 `routing.map_path` 指向副本。包（pack）刷新不得覆盖它。

分诊（triage）技能（skill）把它当数据看。一条路由需要来自报告或成因追查的证据。光靠关键词匹配不够。

```yaml
routes:
  - name: "billing-example"
    match:
      product_areas:
        - "billing-area-placeholder"
      code_paths:
        - "billing-code-path-placeholder"
      error_signatures:
        - "billing-error-placeholder"
    destination:
      slack_channel: "billing-channel-placeholder"
      tracker_team: "billing-team-placeholder"
    owners:
      - "billing-owner-placeholder"
    allow_feature_owner_ping: false

  - name: "desktop-example"
    match:
      product_areas:
        - "desktop-area-placeholder"
      code_paths:
        - "desktop-code-path-placeholder"
      error_signatures:
        - "desktop-error-placeholder"
    destination:
      slack_channel: "desktop-channel-placeholder"
      tracker_team: "desktop-team-placeholder"
    owners:
      - "desktop-owner-placeholder"
    allow_feature_owner_ping: false

fallback:
  destination: ""
  owners: []
  allow_feature_owner_ping: false

ping_policy:
  default: "off"
  allow:
    - "configured-feature-owner"
    - "confirmed-regression-author"
  deny:
    - "broad-on-call-group"
    - "unverified-owner"
```

## 规则

- `fallback.destination` 留空，除非有一个团队接收所有未匹配的报告。
- 用稳定的产品领域、代码路径和错误特征。
- 公开的副本里不要放私有数据。
- 不要把原始的用户或频道 ID 贴进会被发布的示例。
- 功能负责人提醒保持关闭，直到目标团队同意。
- 改派是告诉报告者该去哪里。自动化绝不跨频道发帖。
