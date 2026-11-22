### Refactoring（重构）

**你拥有契约。结构要变。行为不能变。** 区别于 Feature（它加行为）和 Bug fix（它修行为）。

如果清理过程中发现缺失的功能或真 bug，把它拆出去，先对着钉住的契约交付结构改动。允许重设计，但要给它点名并路由到 Feature。大型或横切的结构工作归 **figure-it-out** 技能。本手册（playbook）管的是聚焦到中等规模的改动。

1. 先钉住行为契约。追踪受影响的子系统来弄懂契约，然后写一个特征化测试、快照或等价工装（harness），在任何结构挪动之前抓住当前行为。这块没有覆盖，就先写钉子再动结构。类型检查和 lint 不算钉子。
2. 按 **principle-model-the-domain** 给代码缺的那个结构命名。形状已经清楚又局部时，无聊的代码就留着。重塑必须删掉分支或非法状态，而不是加间接层。
3. 说出目标形状。讲清如果今天重来，模块布局、类型和调用图应该长什么样（**principle-foundational-thinking**、**principle-redesign-from-first-principles**）。目标跨函数边界，就在动手前跑 **architect** 技能（skill）定形状。
4. 先减后加。在引入新形状之前，删死代码、折叠只有一个调用者的包装层、丢掉冗余校验器、移除孤儿引用（**principle-subtract-before-you-add**）。抵达目标形状的最小改动才交付（**principle-laziness-protocol**）。"可能有用"的投机清理一律回退。
5. 用小步走，每步都保持行为不变、钉子保持绿色。API 重塑时，同一波里迁移每个调用方并删掉旧 API（**principle-migrate-callers-then-delete-legacy-apis**）。不留兼容垫片，不留新旧并行路径。每个改名都对照实际文件抽查。改名会悄悄漏掉字符串、散文和反向引用里的用法。把机械编辑委派给一个子代理（subagent），用你配置的 refactoring 模型（默认 `claude-opus-5-5-xhigh-fast`），给出具体范围（文件路径、要挪的名字、要保持的行为）。照子代理（Subagents）一节给每个子代理挑推理档位。
6. 在真实产物上证明行为没变，不是"能编译"（**principle-prove-it-works**）。更大的重塑跑一次等价检查：用脚本 diff 新旧输出、拿录制的基线对着新代码回放、或用相关控制技能在相配的界面上跑冒烟。
7. 确认这个改动值得留。成功度量是读者负担下降（**principle-minimize-reader-load**）。diff 没能在任何地方降低读者负担，就回退它。
8. 变基（rebase）成小而有序的提交（commit）。一个删减提交，然后重塑，然后后续清理。用 **sequence-verifiable-units** 原则（principle）技能把它们塑形，让每个保持行为的切片在下一片之前保持绿色。跑 **Opening a PR** 手册。

**回复：** 变了的结构、你拿它对照的钉子、等价证明、读者负担差值、交付了什么、回退了什么。不许有新行为。
