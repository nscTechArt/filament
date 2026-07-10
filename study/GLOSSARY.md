# 术语表

> Filament 专用术语表。解释优先采用源码语义，而非泛化图形学定义。
> 每条标注来源（文件:行号）。未在源码中完全确认的标记状态。

## 核心类型

### Engine
- [事实] Public API 类，定义于 `filament/include/filament/Engine.h:188`。
- 语义：渲染器主入口，创建并持有 Renderer、Scene、View、Material 等 resource，管理 Backend Driver 生命周期。
- 内部实现：`FEngine` (`filament/src/details/Engine.h:135`)。

### FEngine
- [事实] Engine 的内部实现类，`F` 前缀命名约定。
- 持有 DriverApi（CommandStream）与具体 Driver，是 Engine 与 Backend 的桥接点。

### Renderer
- [事实] Public API 类，定义于 `filament/include/filament/Renderer.h:73`。
- 语义：管理帧生命周期（`beginFrame` → `render` 各 View → `endFrame`），持有 swap chain。
- 内部实现：`FRenderer` (`filament/src/details/Renderer.h:73`)。

### FRenderer
- [事实] Renderer 的内部实现。
- 关键方法：`beginFrame` (`details/Renderer.cpp:394`)、`render` (`:707`)、`renderInternal` (`:727`)、`renderJob` (`:755`)。

### View
- [事实] Public API 类，定义于 `filament/include/filament/View.h:72`。
- 语义：一个 camera + scene + viewport 组合，管理剔除、LOD、阴影、froxel 光照、后处理选项、动态分辨率、色彩分级。
- 内部实现：`FView` (`filament/src/details/View.h:98`)。

### FView
- [事实] View 的内部实现。
- 关键方法：`prepare` (`details/View.h:121`)、`renderShadowMaps` (`:235`)。

### Scene
- [事实] Public API 类，定义于 `filament/include/filament/Scene.h:68`。
- 语义：持有场景实体集合（renderable + light），管理 skybox 与 indirect light。
- 内部实现：`FScene` (`filament/src/details/Scene.h:51`)。
- 数据结构：`RenderableSoA`、`LightSoA`（Structure of Arrays）。

## Backend 抽象

### Driver
- [事实] 抽象 GPU 驱动接口，定义于 `filament/backend/include/private/backend/Driver.h:64`。
- 语义：所有 Backend 命令（资源创建、draw call、render pass）的虚方法声明于此，各 Backend 提供具体实现。
- 方法经 `DriverAPI.inc` 中的 `DECL_DRIVER_API`/`DECL_DRIVER_API_SYNCHRONOUS`/`DECL_DRIVER_API_RETURN` 宏声明。

### DriverApi
- [事实] CommandStream 的类型别名/封装，Engine/Renderer/FrameGraph 通过它录制命令。
- 语义：面向调用方的"录制端"，线程安全，将调用序列化进 CircularBuffer。
- 与 Driver 的关系：DriverApi 录制 → CommandStream 序列化 → execute → Driver 虚方法。

### DriverBase
- [事实] Driver 的中间基类，定义于 `filament/backend/src/DriverBase.h:189`。
- `VulkanDriver` 与 `OpenGLDriverBase` 均继承自它。

### CommandStream
- [事实] 线程安全命令缓冲前端，定义于 `filament/backend/include/private/backend/CommandStream.h:196`。
- 机制：将 driver 方法调用序列化进 `CircularBuffer`（placement-new 命令对象），渲染线程 `execute(buffer)` (`:252`) 经 `Dispatcher` 分发到具体 `Driver`。
- 辅助：`queueCommand` (`:258`)、`allocate` (`:265`)。

### Handle
- [事实] Backend 资源句柄模板，定义于 `filament/backend/include/backend/Handle.h:50+`。
- 语义：类型安全的 GPU 资源引用，配合 `Hw*` 结构（如 `HwTexture`、`HwVertexBuffer`）使用。

### VulkanDriver
- [事实] Driver 的 Vulkan 实现，定义于 `filament/backend/src/vulkan/VulkanDriver.h:61`。
- `class VulkanDriver final : public DriverBase`。
- 持有 `VulkanContext`、`VulkanPipelineCache`、`VulkanStagePool`、`VulkanMemory`、`VulkanDescriptorSetCache` 等。

### OpenGLDriver
- [事实] Driver 的 OpenGL 实现，定义于 `filament/backend/src/opengl/OpenGLDriver.h:85`。
- `class OpenGLDriver final : public OpenGLDriverBase`。
- 持有 `OpenGLContext`、`OpenGLState`、`ShaderCompilerService` 等。

## FrameGraph

### FrameGraph
- [事实] 渲染 Pass 依赖图，定义于 `filament/src/fg/FrameGraph.h:61`。
- 关键方法：`addPass` (`:290`)、`compile` (`:313`)、`execute(DriverApi&)` (`:320`)。
- 语义：管理资源分配、Pass 裁剪、执行排序。`compile()` 裁剪未引用 Pass，`execute()` 通过 DriverApi 执行。

### RenderPass / RenderPassNode
- [事实] FrameGraph 中的 Pass 节点，`RenderPassNode` 定义于 `filament/src/fg/details/PassNode.h:76`。
- 语义：声明一组输入/输出资源与一个 execute 回调。

## Material 体系

### Material
- [事实] Public API 类，定义于 `filament/include/filament/Material.h:62`。
- 语义：包装 matc 编译后的材质 blob，暴露参数反射，提供 MaterialInstance 工厂。
- 内部实现：`FMaterial` (`filament/src/details/Material.h:64`)。

### MaterialInstance
- [事实] Public API 类，定义于 `filament/include/filament/MaterialInstance.h:52`。
- 语义：持有每实例参数值（uniform + sampler）覆盖材质默认。
- 内部实现：`FMaterialInstance` (`filament/src/details/MaterialInstance.h:56`)。

### matc
- [事实] 材质编译器（离线工具），位于 `tools/matc/`。
- 语义：将 `.mat` 材质文件编译为运行时可消费的 `.filamat` 二进制。
- chunk 格式定义于 `libs/filabridge/include/filament/MaterialChunkType.h`。

### MaterialParser
- [事实] 运行时解析 `.filamat` 的组件，定义于 `filament/src/MaterialParser.h:55`。
- 由 `MaterialDefinition::createParser()` (`filament/src/MaterialDefinition.cpp`) 创建。

## 渲染相关

### Froxel
- [推断] Filament 的分块光源分配技术，将视锥体划分为 3D 网格（frustum + voxel = froxel）以高效计算光源影响。
- 依据：`FView` 中有 froxel 光照相关字段（`details/View.h`），具体实现待源码确认。
- Status: Partially verified — 概念确认，实现入口待跟踪。

### RenderPrimitive
- [推断] 表示一个可渲染的图元（vertex buffer + index buffer + material binding 组合）。
- 依据：`RenderableManager` 公共 API 涉及 renderable 与 primitive 概念。
- Status: Partially verified — 待定位具体类型定义。

### Renderable
- [事实] 场景中可渲染对象的统称，通过 `RenderableManager` 构建，存储于 `FScene::RenderableSoA`。
- 语义：一个 Entity + transform + visibility + skinning + AABB + mesh primitive 列表。

## 基础设施

### Entity / EntityManager
- [事实] ECS 基础，`Entity` (`libs/utils/include/utils/Entity.h:38`)、`EntityManager` (`EntityManager.h:42`)。
- 语义：轻量实体标识与分配，renderable/light/camera 等均以 Entity 为键。

### JobSystem
- [事实] work-stealing 并行任务系统，定义于 `libs/utils/include/utils/JobSystem.h:46`。
- 语义：Filament 内部的并行调度框架，用于多线程剔除、阴影、光照计算等。

### UTILS_GUARDED_BY / UTILS_REQUIRES
- [事实] 线程安全注解宏，定义于 `libs/utils/include/utils/compiler.h:246`/`:258`。
- 语义：标注 mutex 保护的成员与所需锁前提条件，详见 `skills/cpp_static_thread_safety/SKILL.md`。

### LockGuard
- [事实] RAII 锁守卫，定义于 `libs/utils/include/utils/Mutex.h:109`。

### filabridge
- [事实] 桥接库，位于 `libs/filabridge/`，CMake target `filabridge`。
- 语义：matc 与运行时之间的共享定义层（材质 chunk 布局、interface block schema、descriptor set layout、variant key）。

### RootArenaScope
- [事实] 帧级内存 arena 的作用域对象，出现在 `FView::prepare`、`FRenderer::renderJob` 等签名中。
- 语义：每帧临时内存分配，帧结束自动释放，避免堆分配开销。
- Status: Partially verified — 用途确认，分配器实现细节待深入。

## 命名约定

### F 前缀
- [事实] Filament 内部实现类的命名约定：public 类 `Engine` → 内部类 `FEngine`。
- 桥接通过 `FILAMENT_DOWNCAST` 宏（`downcast(this)`）。

### Hw 前缀
- [事实] Backend 硬件资源结构命名约定，如 `HwTexture`、`HwVertexBuffer`，定义于 `filament/backend/include/backend/Handle.h` 附近。
- 语义：Driver 实现层持有的具体 GPU 资源对象，由 `Handle<T>` 引用。

### DECL_DRIVER_API 宏
- [事实] `DriverAPI.inc` 中声明 Driver 虚方法的三种宏之一：
  - `DECL_DRIVER_API` — 异步命令（经 CommandStream 序列化）
  - `DECL_DRIVER_API_SYNCHRONOUS` — 同步命令（直接调用 Driver）
  - `DECL_DRIVER_API_RETURN` — 有返回值的命令
