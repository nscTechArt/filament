# Material 管线

> Status: Pending analysis
> 待分析源码：`filament/src/material/`、`tools/matc/`

## 计划内容

本文档将记录：

- Material 类定义与职责（文件:行号）
- MaterialInstance 类定义与职责
- Material 与 MaterialInstance 的关系
- matc 编译流程：`.mat` → 编译产物 → 运行时加载
- 材质参数绑定机制
- 与 Backend Shader 模块的对接

## 待跟踪符号

- `Material` / `FMaterial`
- `MaterialInstance` / `FMaterialInstance`
- matc 入口与编译产物结构

> 以上为计划跟踪目标，尚未在源码中确认。确认后更新本文档。
