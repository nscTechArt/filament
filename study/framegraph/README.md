# FrameGraph 研究子目录

本目录记录 Filament FrameGraph 的研究笔记。

## 文档

| 文档 | 内容 | 状态 |
|------|------|------|
| [RESOURCE_LIFETIME.md](RESOURCE_LIFETIME.md) | FrameGraph 资源生命周期与 Pass 依赖 | Pending analysis |

## 关键源码目录

- `filament/src/fg/` — FrameGraph 实现

## 计划研究点

- FrameGraph 类结构与编译/执行入口
- RenderPass 声明与依赖
- 资源（Texture/RenderTarget）的声明、别名与生命周期
- FrameGraph 如何与 Renderer::render 对接
- 与 DriverApi 命令的映射关系

> 所有具体结论待源码分析后填充。
