---
name: principle-migrate-callers-then-delete-legacy-apis
description: "适用于在旧调用方还在时引入新的内部 API。同一波里迁移调用方并删除旧 API，不要保留兼容层。"
disable-model-invocation: true
---

# Migrate Callers Then Delete Legacy APIs（先迁移调用方，再删除旧 API）

- 更新测试去断言新契约，删掉那些只为保护重构前实现细节的测试

**适用条件：**
- 没有外部用户依赖向后兼容
- 项目能吸收协调一致的破坏性变更
- 新 API 是某次简化或重构计划的一部分
