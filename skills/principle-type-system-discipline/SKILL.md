---
name: principle-type-system-discipline
description: "适用于设计类型、审查函数签名，或在任意静态类型语言里写代码时。让非法状态无法表示，给语义原语打上品牌标记，在边界解析外部数据，穷尽所有变体，从权威 schema 推导。"
disable-model-invocation: true
---

# Type System Discipline（类型系统纪律）

让编译器拒绝不可能的状态、不匹配的原语和未处理的变体。

- **给语义原语打上品牌标记。** 两个参数共用一个原语类型却含义不同时，给各自独立的类型。在创建时校验一次。
- **外部数据在解析之前没有类型。** JSON、RPC 载荷、CLI 参数、配置、环境变量和数据库行，都要在边界处经过一个解析函数，返回带类型的模型。
- **从权威 schema 推导类型。** schema 文件拥有某个形状时，从它推导类型，不要手工另造一份平行版本。

TypeScript 语法参见 **typescript-best-practices** 技能（skill）。
