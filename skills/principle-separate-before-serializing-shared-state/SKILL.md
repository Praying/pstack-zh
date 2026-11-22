---
name: principle-separate-before-serializing-shared-state
description: "适用于并发行动者可能写同一个文件、分支、键或状态对象时。先消除共享，只有单一共享写入者是真正不变量时才做结构性串行化。"
disable-model-invocation: true
---

# Separate Before Serializing Shared State（先拆分，再串行化共享状态）

当并发行动者可能写同一个文件、分支、键或对象时，先问一句：它们真的需要一个共享对象，还是在各自发布独立的事实。默认给每个行动者自己的文件、键、分支或状态目录，只在读取状态的地方合并。两个工作单元把各自的字段写进同一个 `state.json`，这仍然是共享修改。

只有当单一共享写入目标是真正的不变量时，才结构性地串行访问（锁文件、单一写入者、顺序阶段、比较并交换）。指令不是并发控制，"我们需要一把锁"是先要检查的坏味道。
