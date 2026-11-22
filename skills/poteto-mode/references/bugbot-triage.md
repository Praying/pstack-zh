# Bugbot triage（bugbot 分诊）

当 Babysit 手册（`../playbooks/babysit.md`）处理 Bugbot 或审查自动化评论时，用这份参考。目标不是默认无视 Bugbot。目标是不再把每条评论都当成必须改代码。

## 决策判据

动手之前先给每条 Bugbot 线程分类：

- `fix`：评论指出了一个说得通的正确性、安全、隐私、数据丢失、鉴权、计费、迁移、幂等、竞态或已交付行为的问题。在拥有它的最底层 PR 里修掉，然后带着提交（commit）SHA 回复并解决该线程。
- `dismiss`：评论命中了一个有记录的低风险嘈杂模式，且当前代码或上下文证明该疑虑不需要改代码。用一句简短理由回复并解决该线程。
- `ask`：评论是新面孔、高严重度、涉及安全/隐私/数据，或含义模糊。问用户，不要猜。

拿不准就问。跳过一条嘈杂的代码质量评论代价很小；跳过一个真实的数据或安全 bug 代价不小。

## 已学模式的格式

以后的新模式按这个形状添加：

```markdown
### <short pattern name>

- Confidence: candidate | recurring | strong
- Skip when: <conditions that must be true>
- Do not skip when: <risk boundaries>
- Example signal: <phrases or code context that identify the pattern>
- Source: <PR/comment URL or short historical note>
```

一两个例子用 `candidate`。多次真实驳回之后用 `recurring`。只有当模式收窄、反复验证过且低风险时才用 `strong`。

## 反复出现的跳过候选

### 有意的 UI 或设计系统视觉改动

- Confidence: candidate
- Skip when: PR 描述、截图、设计审查或邻近代码把这次视觉改动写得明明白白，而 Bugbot 评论只是复述某个共享视觉默认值变了。
- Do not skip when: 评论指向无障碍、焦点可见性、键盘导航、颜色对比度，或某个 PR 并没有有意更改的组件 API 契约。
- Example signal: 关于焦点轮廓、按钮尺寸、间距或共享组件视觉默认值的评论，所有者回复 "intentional" 或 "intended"。

### Bugbot 看不到的上层栈或栈内用法

- Confidence: candidate
- Skip when: Bugbot 标记某个导出、组件、辅助函数或文件未被使用，而当前托管平台（forge）的 PR 列表与 diff、上层栈的 diff 或 PR 上下文显示它被栈里更后面的 PR 用到了。
- Do not skip when: 当前 PR 不属于任何栈、该符号是公开 API，或所谓的上层栈用法无法验证。
- Example signal: "Exported component is never used"，配上人类回复 "used upstack" 这类。

### 并行实现期间的临时重复

- Confidence: candidate
- Skip when: PR 有意重复少量代码，让新路径与正在删除、替换或验证中的旧路径保持平行。
- Do not skip when: 重复的代码涉及安全、计费、数据访问、API 行为，或一个长期存活的共享抽象能明显降低风险。
- Example signal: "Significant duplication" 或 "duplicated validation logic"，所有者解释旧路径会被删掉，或重复逻辑是刻意留在本地的。

### 现有框架或组件不变量已覆盖该警告

- Confidence: candidate
- Skip when: 该疑虑已由共享组件、框架契约、类型不变量或单一事实来源保证，且在当前 diff 或邻近代码中看得见。
- Do not skip when: 不变量只是被假定而没有被强制，依赖时序，或跨越值可能分叉的异步/状态边界。
- Example signal: 关于内层 popover 缺 max-height 的评论，而共享 popover 已强制视口边界；或可空值的评论，而本地检查的值和传入的值同源。

### 所有者声明的后续处理或延后清理

- Confidence: candidate
- Skip when: PR 所有者明说这是已知的后续事项，当前 PR 没有让行为变糟，且评论不涉及高风险区域。
- Do not skip when: 代理（agent）在没有所有者意见的情况下行动、问题属于中/高严重度的产品行为，或延后会把新回归合并进去。
- Example signal: "I'll worry about that later" 或 "we'll delete this eventually"。

### 自我撤回或明示误报的规则类评论

- Confidence: recurring
- Skip when: 评论正文或 Bugbot 后续回复明说该发现已撤回、合规或属误报，且代理能在本地验证相关规则。
- Do not skip when: 唯一证据是人类在高风险问题上不加解释地说 "false positive"。
- Example signal: 一条文件命名规则评论，正文说该文件已经合规。

## 默认要问

这些类别不要自动跳过，哪怕之前的 PR 驳回过类似东西：

- 安全、隐私、鉴权、计费、数据留存、训练数据、权限边界方面的发现。
- 高严重度的发现。
- 迁移、schema、幂等、并发、跨系统行为方面的发现。
- 建议的修复很小、明显降低风险又不改变产品意图的评论。

历史数据表明人类有时会驳回安全/数据流评论。把这些当作所有者的个案判断，不是全队通用的跳过规则。

## 近期 babysits 的候选经验

在 babysit 期间或之后，把看起来对全队有用但还不成熟的新候选经验追加到这里。一旦多个 PR 确认了模式，优先把反复出现的候选提升到上面一节。

### 对原生浏览器行为的手工重实现

- Confidence: candidate
- Skip when: 几乎从不。当一个 diff 把原生浏览器行为换成手工等价物（原生 sticky → JS 定位的克隆、原生滚动定位 → 转发的 wheel/touch 事件、绘制顺序遮挡 → masks/clip-path），Bugbot 对那部分代码报的逻辑 bug 一直都很站得住。
- Do not skip when: 发现涉及这类代码里的事件转发缺口（wheel deltaMode、touch 平移、边缘滚动链、tap slop）、mask/clip 命中测试分歧，或 observer 与 React 之间的状态时序竞态。默认选 fix。
- Example signal: "masks do not affect hit-testing"、"overlay blocks wheel scroll"、"ignores deltaMode"、"runs in the IntersectionObserver callback before React applies state"。
- Source: 一个 sticky 遮挡 PR：Bugbot 跑了六轮，大约十八条发现，条条都修掉而不是驳回。

### 契约测试漂移的说法验证起来很便宜，先跑测试

- Confidence: candidate
- Skip when: 验证本身绝不跳过，它只花一条命令。当一个 PR
  交付了一个钉住协议或文档措辞的契约测试（对 SKILL.md 下的正则、
  文档措辞的快照），而 Bugbot 声称"测试不再匹配文档"
  （或反过来）时，在归类之前先在 PR 的 tip 上跑那个测试。
  跑红就是对这个说法的实证；跑绿就是驳回回复的
  具体反证。
- Do not skip when: n/a，这是验证捷径，不是驳回
  模式。注意"多轮通过就倾向驳回"的启发式在这里会失灵：
  钉住措辞的测试会漂移，恰恰正是因为早先的修复轮会去改措辞。
- Example signal: "Contract test omits the pre-fix wait"，出现在这样的 PR 上：早先的
  修复提交改写了被钉住的段落；在 tip 上跑测试，恰好在被引用的那条
  断言上失败。
- Source: 一个钉住措辞的 PR，Bugbot 跑了八轮；尽管前面每轮都修掉并解决了，
  这个说法在第 7 轮是真的。

### 过期的安全审查发现，同一 PR 后面已经修掉

- Confidence: candidate
- Skip when: 一次代理式安全审查（或类似机制）声称缺少 authz/validation 调用，而当前 PR 的 tip 明确包含那道检查关卡（带测试），通常是审查跑完之后的加固提交里加的。
- Do not skip when: 被引用的辅助函数对讨论中的主体（principal）是空操作、检查在它守卫的副作用之后才跑，或缺少对该主体的覆盖。
- Example signal: 一条 HIGH 级别的 "missing authorization check" 发现，而那道守卫在 tip 上早已在副作用之前被调用。
- Source: 一个 webhook 端点 PR，它的加固提交晚于审查运行。

### 放宽刻意收窄的错误条件会掩盖真正的错误

- Confidence: candidate
- Skip when: 发现要求把收窄的错误条件（某个具体
  `errno`、错误码或状态类）放宽成一网打尽，而这种收窄
  编码着一个真实区别。经典形状是一个只认 `ENOENT` 的
  依赖回退："binary is not installed" 和 "the command ran and failed"
  是两种情形。对任何非零退出重试都会把一次正当失败
  （找不到、鉴权过期、网络）对着回退重跑一遍，然后
  报告回退的错误，把真正的错误藏起来。
- Do not skip when: 收窄条件漏掉了同一类里的
  另一种情形（另一个"binary unusable" errno，如 `EACCES`，另一种
  传输层失败）、未处理路径丢数据或留下半截状态，或重试
  是幂等的而且原始错误仍然浮出。
- Example signal: "only retries when X fails with ENOENT … never tries the
  fallback even when a working Y exists"，指向这样的代码：它的回退是为
  缺失依赖准备的，而不是为操作失败准备的。
- Source: 一个 CLI 改名 PR，它的回退是为缺失二进制准备的，而不是
  为命令失败准备的。
