# OpenGL Backend

> Status: Pending analysis
> 待分析源码：`filament/backend/src/opengl/`

## 计划内容

本文档作为 Vulkan Backend 的对照，记录：

- OpenGLDriver 类定义与 Driver 接口实现（文件:行号）
- GL context 初始化与能力查询
- 资源管理：Buffer、Texture、Sampler、Program
- 命令录制语义（GL 的立即模式 vs Vulkan 的延迟命令缓冲）
- 同步模型差异（GL fence/sync vs Vulkan semaphore/fence）
- 与 Vulkan Backend 的抽象取舍对照

## 待跟踪符号

- `OpenGLDriver`
- GL context / 能力查询相关结构
- Program / Pipeline 对应物

> 以上为计划跟踪目标，尚未在源码中确认。确认后更新本文档。
