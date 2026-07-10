# FrameGraph 资源生命周期

> Status: Pending analysis
> 待分析源码：`filament/src/fg/`

## 计划内容

本文档将记录：

- FrameGraph 资源声明 API（文件:行号）
- 资源的生命周期：创建 → 传递 → 释放
- Pass 间依赖与资源传递语义
- 资源别名（aliasing）与内存复用
- 编译期裁剪：未被引用的 Pass 与资源如何被剔除
- 与 Backend 资源分配的对接

## 待跟踪符号

- `FrameGraph`
- `RenderPass`
- 资源声明相关类型（Resource、Handle 等）

> 以上为计划跟踪目标，尚未在源码中确认。确认后更新本文档。
