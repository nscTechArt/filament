# Vulkan Backend

> Status: Pending analysis
> 待分析源码：`filament/backend/src/vulkan/`

## 计划内容

本文档将记录：

- VulkanDriver 类定义与 Driver 接口实现（文件:行号）
- Vulkan 实例 / 物理设备 / 逻辑设备 / 队列初始化
- Swapchain 与呈现
- 命令缓冲录制与提交
- 资源管理：Buffer、Image、Sampler、DescriptorSet
- 同步：Semaphore、Fence、Barrier
- Shader 模块与 Pipeline 创建
- 与 FrameGraph 的对接点

## 待跟踪符号

- `VulkanDriver`
- Vulkan context / device 相关结构
- 命令录制与提交入口

> 以上为计划跟踪目标，尚未在源码中确认。确认后更新本文档。
