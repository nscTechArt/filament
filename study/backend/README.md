# Backend 研究子目录

本目录记录 Filament Backend 层的研究笔记，包括 Driver API 抽象、Vulkan Backend（主要研究对象）与 OpenGL Backend（对照研究对象）。

## 文档

| 文档 | 内容 | 状态 |
|------|------|------|
| [DRIVER_API.md](DRIVER_API.md) | Driver 公共接口、虚函数表、与 Engine 的交互边界 | Pending analysis |
| [VULKAN_BACKEND.md](VULKAN_BACKEND.md) | VulkanDriver 实现：队列、命令录制、资源、同步 | Pending analysis |
| [OPENGL_BACKEND.md](OPENGL_BACKEND.md) | OpenGLDriver 实现，作为 Vulkan 的对照 | Pending analysis |

## 研究顺序建议

1. 先读 `DRIVER_API.md` 理解抽象接口与 CommandStream。
2. 再读 `VULKAN_BACKEND.md` 理解一个完整 Backend 如何实现 Driver 接口。
3. 最后读 `OPENGL_BACKEND.md` 做对照，理解抽象的跨平台取舍。

## 关键源码目录

- `filament/backend/include/backend/` — Driver 公共接口
- `filament/backend/src/` — CommandStream 与 Backend 公共实现
- `filament/backend/src/vulkan/` — Vulkan Backend
- `filament/backend/src/opengl/` — OpenGL Backend

> 所有具体结论待源码分析后填充，不预先编造。
