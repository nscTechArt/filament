# 模块地图

> 一级模块地图，基于 `study/v1.73.0` 源码确认。
> 每个模块标注：职责、关键目录、关键类型（file:line）、关系、理解状态。
> 本版只到一级，不深入所有函数。

## 分层总览

```
Public API (filament/include/filament/)
    │  downcast / FILAMENT_DOWNCAST
    ▼
Details 实现层 (filament/src/details/)   ←─ F 前缀内部类型
    │
    ├── Material (filament/src/Material*)  ◀── filabridge ── matc
    ├── FrameGraph (filament/src/fg/)
    │       │
    │       └──▶ RenderPass / Resource
    │
    └── Backend (filament/backend/)
            │  DriverApi (CommandStream)
            ▼
        VulkanDriver / OpenGLDriver / MetalDriver
            │
            ▼
        Platform (Vulkan / EGL / Metal / WebGPU)
```

---

## 1. Public API 层

| 项 | 内容 |
|----|------|
| 目录 | `filament/include/filament/` |
| 职责 | 面向用户的稳定 C++ API，客户端直接使用 |
| 关键类型 | `Engine` (`Engine.h:188`)、`Renderer` (`Renderer.h:73`)、`View` (`View.h:72`)、`Scene` (`Scene.h:68`)、`Material` (`Material.h:62`)、`MaterialInstance` (`MaterialInstance.h:52`)、`RenderableManager`、`Camera`、`LightManager`、`Texture`、`VertexBuffer`、`IndexBuffer`、`Skybox` |
| 关系 | 所有类继承 `FilamentAPI`，通过 `downcast()` 委托给 `F` 前缀实现 |
| 状态 | [事实] 入口与命名约定已确认；各 API 细节待逐模块深入 |

## 2. Details 实现层

| 项 | 内容 |
|----|------|
| 目录 | `filament/src/details/`（52 文件） |
| 职责 | Public API 的真实实现，每个 public 类对应一个 `F` 前缀内部类 |
| 关键类型 | `FEngine` (`details/Engine.h:135`)、`FRenderer` (`details/Renderer.h:73`)、`FView` (`details/View.h:98`)、`FScene` (`details/Scene.h:51`)、`FMaterial` (`details/Material.h:64`)、`FMaterialInstance` (`details/MaterialInstance.h:56`) |
| 命名约定 | 内部类型 `F` 前缀，继承 public 对应类，用 `FILAMENT_DOWNCAST` 宏桥接 |
| 状态 | [事实] 分层与命名约定已确认 |

## 3. Renderer

| 项 | 内容 |
|----|------|
| 目录 | `filament/src/`、`filament/src/details/` |
| 关键类型 | `FRenderer` (`details/Renderer.h:73`) |
| 关键方法 | `beginFrame` (`details/Renderer.cpp:394`)、`render` (`details/Renderer.cpp:707`)、`renderInternal` (`details/Renderer.cpp:727`)、`renderJob` (`details/Renderer.cpp:755`) |
| 职责 | 管理帧生命周期（`beginFrame` → 各 View `render` → `endFrame`），持有 swap chain、帧调度器，通过 `renderInternal`/`renderJob` 发起渲染 |
| 关系 | 上接 Engine，下接 View 与 FrameGraph，向 DriverApi 发命令 |
| 状态 | [事实] 入口方法已确认；`renderJob` 内部完整调用链待跟踪 |

## 4. View

| 项 | 内容 |
|----|------|
| 目录 | `filament/src/`、`filament/src/details/` |
| 关键类型 | `FView` (`details/View.h:98`) |
| 关键方法 | `prepare` (`details/View.h:121`)、`setScene` (`details/View.h:125`)、`setCameraUser` (`details/View.h:462`)、`setViewport` (`details/View.h:135`)、`renderShadowMaps` (`details/View.h:235`) |
| 职责 | 表示一个 camera + scene + viewport 组合，管理剔除、LOD、阴影贴图、froxel 光照、后处理选项（bloom/TAA/SSAO/DoF/vignette）、动态分辨率、色彩分级，持有每帧 uniform 数据 |
| 关系 | 被 Renderer 调用 prepare 与 render；引用 Scene |
| 状态 | [事实] 类与方法签名已确认；prepare 内部流程待跟踪 |

## 5. Scene

| 项 | 内容 |
|----|------|
| 目录 | `filament/src/`、`filament/src/details/` |
| 关键类型 | `FScene` (`details/Scene.h:51`) |
| 关键方法 | `setSkybox`/`setIndirectLight` (`details/Scene.h:195-196`)、`addEntity`/`addEntities` (`:197-198`)、`remove` (`:199-200`)、`prepare` (`:71`) |
| 数据结构 | `RenderableSoA`（每帧 renderable 数据）、`LightSoA`（每帧光源数据） |
| 职责 | 持有场景实体集合，管理 skybox、indirect light，缓存每帧 renderable/light 数据于 `SceneCacheData` |
| 关系 | 被 View 引用；与 RenderableManager/LightManager 协作 |
| 状态 | [事实] 类与 SoA 结构已确认；prepare 细节待跟踪 |

## 6. FrameGraph

| 项 | 内容 |
|----|------|
| 目录 | `filament/src/fg/`（21 文件，含 `details/` 子目录） |
| 关键类型 | `FrameGraph` (`fg/FrameGraph.h:61`)、`RenderPassNode` (`fg/details/PassNode.h:76`)、`FrameGraphPass` (`fg/FrameGraphPass.h`)、`FrameGraphResources` (`fg/FrameGraphResources.h`) |
| 关键方法 | `addPass` (`FrameGraph.h:290`)、`compile` (`FrameGraph.h:313`)、`execute(DriverApi&)` (`FrameGraph.h:320`) |
| 子目录 | `fg/details/`：`PassNode.h`、`Resource.h`、`ResourceNode.h`、`ResourceAllocator.h`、`DependencyGraph.h`、`ResourceCreationContext.h`、`Utilities.h` |
| 职责 | 渲染 Pass 依赖图，管理资源分配、Pass 裁剪、执行排序。`compile()` 分析图并裁剪未引用 Pass；`execute()` 通过 DriverApi 执行结果命令序列 |
| 关系 | 由 Renderer/View 构建并执行；向 DriverApi 发命令 |
| 状态 | [事实] 类与入口方法已确认；compile/execute 内部算法待深入 |

## 7. Material

| 项 | 内容 |
|----|------|
| 目录 | `filament/src/Material.cpp`、`filament/src/MaterialInstance.cpp`、`filament/src/details/Material.cpp`、`filament/src/details/MaterialInstance.cpp` |
| 关键类型 | `FMaterial` (`details/Material.h:64`)、`FMaterialInstance` (`details/MaterialInstance.h:56`) |
| matc 数据消费 | `MaterialParser` (`filament/src/MaterialParser.h:55`) 解析编译后的 `.filamat` 文件；chunk 格式定义于 `libs/filabridge/include/filament/MaterialChunkType.h`；parser 由 `MaterialDefinition::createParser()` (`filament/src/MaterialDefinition.cpp`) 创建 |
| 职责 | `Material`/`FMaterial` 包装编译后的材质 blob，暴露参数反射，提供 `MaterialInstance` 工厂；`MaterialInstance`/`FMaterialInstance` 持有每实例参数值（uniform + sampler）覆盖材质默认 |
| 关系 | 依赖 filabridge 的 chunk 格式；与 Backend Shader 模块对接 |
| 状态 | [事实] 类与解析入口已确认；matc 编译流程与变体机制待深入 |

## 8. Backend 接口

| 项 | 内容 |
|----|------|
| 目录 | `filament/backend/include/backend/`（public）、`filament/backend/include/private/backend/`（private） |
| 关键类型 | `Driver` (`private/backend/Driver.h:64`)、`Handle<T>` (`backend/Handle.h:50+`)、`Platform` (`backend/Platform.h`) |
| 关键文件 | `DriverAPI.inc`（Driver 虚方法声明宏）、`DriverEnums.h`（枚举与常量） |
| Driver 虚方法 | 经 `DriverAPI.inc` 中 `DECL_DRIVER_API`/`DECL_DRIVER_API_SYNCHRONOUS`/`DECL_DRIVER_API_RETURN` 宏声明，如 `beginFrame` (`DriverAPI.inc:146`)、`endFrame` (`:165`)、`createVertexBuffer` (`:188`)、`createTexture` (`:225`)、`beginRenderPass` (`:565`)、`commit` (`:594`) |
| 职责 | 抽象 GPU 驱动接口，所有 backend 命令（资源创建、draw call、render pass）均声明于此，各 Backend 提供具体实现 |
| 状态 | [事实] 接口与宏机制已确认；完整方法清单待分类整理 |

## 9. CommandStream

| 项 | 内容 |
|----|------|
| 目录 | `filament/backend/src/`、`filament/backend/include/private/backend/` |
| 关键类型 | `CommandStream` (`private/backend/CommandStream.h:196`) |
| 关键方法 | `execute(void* buffer)` (`CommandStream.h:252`)、`queueCommand(std::function<void()>)` (`:258`)、`allocate()` (`:265`)；宏生成的内联录制方法 (`:212-241`) |
| 机制 | 线程安全命令缓冲前端。将 driver 方法调用序列化进 `CircularBuffer`（placement-new 命令对象），渲染线程调 `execute(buffer)` 经 `Dispatcher` 分发到具体 `Driver` |
| 关系 | 上接 Engine/Renderer 的 DriverApi 调用，下接具体 Driver 实现 |
| 状态 | [事实] 类与录制/执行机制已确认；CircularBuffer 与 Dispatcher 细节待深入 |

## 10. VulkanDriver

| 项 | 内容 |
|----|------|
| 目录 | `filament/backend/src/vulkan/` |
| 关键类型 | `VulkanDriver` (`vulkan/VulkanDriver.h:61`)，`class VulkanDriver final : public DriverBase` |
| 继承 | `DriverBase` (`backend/src/DriverBase.h:189`) → `Driver` |
| 工厂 | `create()` 静态方法 (`VulkanDriver.h:63`) 返回 `Driver*` |
| 持有资源 | `VulkanContext`、`VulkanFboCache`、`VulkanPipelineCache`、`VulkanStagePool`、`VulkanMemory`、`VulkanDescriptorSetCache` 等 |
| 职责 | 实现 Driver 接口的 Vulkan 版本，管理 Vulkan 实例/设备/队列、命令缓冲、资源、同步、Pipeline |
| 状态 | [事实] 类定义与依赖已确认；各子系统实现待逐个深入 |

## 11. OpenGLDriver

| 项 | 内容 |
|----|------|
| 目录 | `filament/backend/src/opengl/` |
| 关键类型 | `OpenGLDriver` (`opengl/OpenGLDriver.h:85`)，`class OpenGLDriver final : public OpenGLDriverBase` |
| 继承 | `OpenGLDriverBase`（同目录）→ `DriverBase` → `Driver` |
| 持有资源 | `OpenGLContext`、`OpenGLState`、`ShaderCompilerService`、`GLTexture`、`GLBufferObject`、`GLDescriptorSet`、`GLMemoryMappedBuffer` |
| 职责 | 实现 Driver 接口的 OpenGL 版本，作为 Vulkan 的对照研究对象 |
| 状态 | [事实] 类定义已确认；与 Vulkan 的抽象取舍对照待深入 |

## 12. Utils

| 项 | 内容 |
|----|------|
| 目录 | `libs/utils/include/utils/` |
| 关键类型 | `JobSystem` (`JobSystem.h:46`)、`EntityManager` (`EntityManager.h:42`)、`Entity` (`Entity.h:38`)、`LockGuard` (`Mutex.h:109`) |
| 线程安全注解 | `UTILS_GUARDED_BY` (`compiler.h:246`)、`UTILS_REQUIRES` (`compiler.h:258`) |
| 内存 | `Allocator.h`（Arena/Pool 等） |
| 职责 | 核心工具库：ECS（Entity/EntityManager）、work-stealing JobSystem、线程安全注解、内存分配器、CString/bitset/FixedCapacityVector 等 |
| 状态 | [事实] 关键类型已确认；JobSystem 调度细节待按需深入 |

## 13. Math

| 项 | 内容 |
|----|------|
| 目录 | `libs/math/` |
| include 路径 | `libs/math/include/math/` |
| 性质 | header-only 模板库 |
| 类型 | `vec2/3/4.h`、`mat2/3/4.h`、`quat.h`、`half.h`、`scalar.h`、`fast.h`、`norm.h` |
| 前向声明 | `mathfwd.h` |
| 模板辅助 | `TMatHelpers.h`、`TVecHelpers.h`、`TQuatHelpers.h` |
| 职责 | 提供全仓库使用的 `math::float3`、`math::mat4f` 等向量/矩阵/四元数类型 |
| 状态 | [事实] 性质与路径已确认 |

## 14. filabridge

| 项 | 内容 |
|----|------|
| 目录 | `libs/filabridge/` |
| CMake target | `filabridge`（static，`libs/filabridge/CMakeLists.txt:4`） |
| 依赖 | `utils`、`math`、`backend_headers` |
| 关键文件 | `include/filament/MaterialChunkType.h`（材质二进制 chunk 类型常量）、`include/filament/MaterialEnums.h`（shader model/feature 枚举）、`include/private/filament/UibStructs.h`、`BufferInterfaceBlock.h`、`SamplerInterfaceBlock.h`、`DescriptorSets.h`、`Variant.h`、`SibStructs.h`、`EngineEnums.h` |
| 源文件 | `BufferInterfaceBlock.cpp`、`SamplerInterfaceBlock.cpp`、`DescriptorSets.cpp`、`Variant.cpp` |
| 职责 | matc（材质编译器）与 Filament 运行时之间的桥接库，定义材质二进制 chunk 布局、uniform/sampler interface block schema、descriptor set layout 规范 |
| 状态 | [事实] 职责与关键文件已确认；chunk 格式细节待按需深入 |

## 15. filamentapp / Samples

| 项 | 内容 |
|----|------|
| filamentapp 目录 | `libs/filamentapp/` — `FilamentApp` 类、平台 display manager（SDL）、mesh 加载（`MeshAssimp`/`Sphere`/`Cube`/`Grid`）、IBL 加载 |
| 最小 sample | `hellotriangle`，源码 `samples/hellotriangle.cpp`（189 行），CMake 注册 `samples/CMakeLists.txt:276` |
| `add_demo` | `samples/CMakeLists.txt:251` → `add_executable(${NAME} ${NAME}.cpp)` |
| 链接依赖 | `sample-resources`、`filamentapp`、`samples-common` |
| 职责 | hellotriangle 创建 Engine + Renderer + View + Scene，用 VertexBuffer + IndexBuffer + 基础材质渲染一个三角形，是最小可运行 Filament 样例 |
| 状态 | [事实] target 与入口已确认；作为最小调试入口 |

---

## 模块关系速查

| 上游 | 下游 | 关系 |
|------|------|------|
| Public API | Details | downcast 委托 |
| FEngine | FRenderer | 创建并持有 |
| FRenderer | FView | 每帧 render 各 View |
| FView | FScene | 引用场景数据 |
| FView | FrameGraph | 构建并执行渲染图 |
| FrameGraph | DriverApi | execute 时发命令 |
| FRenderer | DriverApi | beginFrame/endFrame 等 |
| DriverApi (CommandStream) | Driver (Vulkan/OpenGL) | 序列化命令分发 |
| FMaterial | filabridge | 读取 matc 编译的 chunk 格式 |
| matc | filabridge | 共享 chunk 布局定义 |
| 全仓库 | Utils/Math | 基础设施 |
