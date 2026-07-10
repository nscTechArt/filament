# Shader 变体

> Status: Pending analysis
> 待分析源码：`filament/src/material/`、`tools/matc/`

## 计划内容

本文档将记录：

- Shader 变体生成机制（matc 端）
- 运行时变体选择逻辑
- 材质属性如何影响变体（如 shading model、blend mode、双面等）
- 变体与 Backend Pipeline 的对应关系
- 变体数量与编译开销的工程取舍

## 待跟踪符号

- 变体相关枚举与 key 类型
- 变体选择入口

> 以上为计划跟踪目标，尚未在源码中确认。确认后更新本文档。
