---
name: typescript-best-practices
description: TypeScript 最佳实践。在阅读或编辑任何 .ts 或 .tsx 文件时使用。
paths: ["**/*.ts", "**/*.tsx"]
disable-model-invocation: true
---

# TypeScript 最佳实践

先应用 **type-system-discipline** 原则（principle）技能（skill）。

| 规则 | 摘要 |
|------|---------|
| 可辨识联合 | 用 `kind` 字面量判别字段给变体建模，让不可能的状态无法被表示。不要可选字段包。 |
| 品牌类型 | 用 `& { readonly __brand: "X" }` 给原始类型打品牌，让它们无法混淆。在边界处验证一次。品牌来自一个会验证的类型守卫（`input is AgentId`），绝不出自 `as`。 |
| 构造式建模 | 把类型构造得让非法值根本造不出来。非空用 `[T, ...T[]]`，偶数长度用 `[T, T][]`，区间用 `start` 加 `duration`。这不是运行时守卫，也不是对精化类型的奢望。 |
| 最简单的全类型 | 只要所有操作都保持全函数，就保留 `T[]`。只在宽松类型逼出 `!`、一次强制转换、或一个"should never happen"抛错时，才强化为 `NonEmpty<T>`。 |
| 用 `unknown` 不用 `any` | 外部数据一律是 `unknown`。 |
| 先 schema 后守卫 | 在手写逐属性的类型守卫之前，先用仓库的运行时 schema 库，从 schema 推导出类型，例如 `z.infer`。 |
| 禁用 `as` 断言 | 每个 `as` 都是一次伺机发生的运行时崩溃。只在验证之后才做断言。 |
| 收窄层级 | 判别 switch > `in` 运算符 > `typeof`/`instanceof` > 用户自定义类型守卫 > `as`。 |
| 类型守卫 | 必须真的验证它所声称的东西。撒谎的守卫比 `as` 更糟，因为 bug 藏在一个自称安全的名字后面。命名为 `isX` 或 `hasX`。 |
| 穷尽性 | 在 default 分支里内联 `const _exhaustive: never = x;`，这样新增变体时编译器会报错。 |
| 用 `satisfies` 不用 `as` | 验证值，同时不拓宽字面量类型。 |
| 边界验证 | 在数据跨界进入处解析成一个具名的领域类型。`Record<string, unknown>`（无论写成什么形式）到这次解析为止。内部则信任类型。参见 **boundary-discipline** 原则技能。 |
| 从 schema 派生类型 | 声明新 interface 之前，先找 `Pick`/`Omit`/`Parameters`/`ReturnType`/`Awaited`/`typeof`。 |
| 对象参数 | 传对象，不用位置参数，参数顺序因此不言自明。热路径除外（逐帧渲染、词法分析器、解析器）。 |
| 真实测试 | 能跑真东西就不要 mock。优先用框架的真实测试原语，带上泄漏与可释放对象检查，并在运行中的构建里验证 UI。只 mock 本地跑不了的东西。 |
| 结构化遥测 | 优先用结构化日志器的诊断输出，上下文要足到能凭一个 id 排查问题。发布的代码里不留 `console.log`。 |

示例：`references/patterns.md`。
