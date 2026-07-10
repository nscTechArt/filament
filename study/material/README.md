# Material 研究子目录

本目录记录 Filament Material 管线的研究笔记。

## 文档

| 文档 | 内容 | 状态 |
|------|------|------|
| [MATERIAL_PIPELINE.md](MATERIAL_PIPELINE.md) | Material/MaterialInstance 体系与 matc 编译管线 | Pending analysis |
| [SHADER_VARIANTS.md](SHADER_VARIANTS.md) | Shader 变体与材质参数化 | Pending analysis |

## 关键源码目录

- `filament/src/material/` — 运行时 Material 实现
- `tools/matc/` — 材质编译器（离线）

## 计划研究点

- Material 与 MaterialInstance 的关系
- matc 如何将 `.mat` 材质文件编译为运行时可消费的数据
- 材质参数（uniform）如何绑定到 Shader
- Shader 变体生成与选择机制

> 所有具体结论待源码分析后填充。
