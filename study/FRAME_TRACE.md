# 一帧调用链框架

> Status: Partially verified
> 本文档建立一帧渲染的调用链框架，逐项标注源码确认状态。
> 尚未完全确认的步骤明确标记，并列出下一步需跟踪的符号。

## 框架（待逐项验证）

```
Application / Sample (hellotriangle)
  → Engine
  → Renderer::beginFrame
  → Renderer::render
  → View preparation
  → FrameGraph construction
  → DriverApi commands
  → CommandStream
  → VulkanDriver / OpenGLDriver
```

## 逐项状态

### 1. Application / Sample 入口

| 项 | 内容 |
|----|------|
| 状态 | [事实] 已确认 |
| 来源 | `samples/hellotriangle.cpp`（189 行） |
| 说明 | hellotriangle 通过 `FilamentApp` 框架创建 Engine、Renderer、View、Scene，在帧回调中调用渲染 |

**待跟踪**：hellotriangle 的帧回调具体调用 `Renderer::beginFrame`/`render`/`endFrame` 的位置与顺序。

### 2. Engine

| 项 | 内容 |
|----|------|
| 状态 | [事实] 入口已确认 |
| 来源 | `FEngine` (`filament/src/details/Engine.h:135`) |
| 说明 | Engine 是渲染器主入口，创建 Renderer、Backend Driver |

**待跟踪**：`FEngine` 如何创建并持有 `DriverApi`（CommandStream）与具体 `Driver`；`Engine::create()` 到 Driver 初始化的完整路径。

### 3. Renderer::beginFrame

| 项 | 内容 |
|----|------|
| 状态 | [事实] 签名已确认 |
| 来源 | `FRenderer::beginFrame` (`filament/src/details/Renderer.cpp:394`) |
| 签名 | `bool FRenderer::beginFrame(FSwapChain* swapChain, uint64_t vsyncSteadyClockTimeNano)` |
| 说明 | 帧起始，处理 swap chain 与 vsync 时序 |

**待跟踪**：`beginFrame` 内部对 DriverApi 的调用（如 wait/poll/present 语义）；与 swap chain 的同步细节。

### 4. Renderer::render

| 项 | 内容 |
|----|------|
| 状态 | [事实] 签名已确认 |
| 来源 | `FRenderer::render` (`filament/src/details/Renderer.cpp:707`) |
| 签名 | `void FRenderer::render(FView const* view)` |
| 说明 | 对一个 View 执行渲染 |

**待跟踪**：`render` → `renderInternal` (`:727`) → `renderJob` (`:755`) 的内部调用链与各阶段职责。

### 5. View preparation

| 项 | 内容 |
|----|------|
| 状态 | [事实] 方法已确认，内部流程 Partially verified |
| 来源 | `FView::prepare` (`filament/src/details/View.h:121`) |
| 签名 | `prepare(FEngine&, DriverApi&, RootArenaScope&, Viewport, CameraInfo, ...)` |
| 说明 | View 的每帧准备：剔除、LOD、froxel 光照、阴影贴图、uniform 更新 |

**待跟踪**：
- `prepare` 内部调用顺序
- `FView::renderShadowMaps` (`details/View.h:235`) 的阴影 Pass 编排
- froxel 光照计算入口
- per-frame uniform 如何写入 DriverApi

### 6. FrameGraph construction

| 项 | 内容 |
|----|------|
| 状态 | [事实] 入口已确认，构建过程 Partially verified |
| 来源 | `FrameGraph::addPass` (`filament/src/fg/FrameGraph.h:290`)、`compile` (`:313`)、`execute(DriverApi&)` (`:320`) |
| 说明 | FrameGraph 在 render 阶段被构建：`addPass` 声明 Pass，`compile` 裁剪，`execute` 执行 |

**待跟踪**：
- `renderJob` 中 FrameGraph 的具体构建位置与顺序
- 哪些 Pass 被添加（阴影、GBuffer、光照、透明、后处理）
- `compile()` 的裁剪算法
- `execute()` 如何将 Pass 转为 DriverApi 命令

### 7. DriverApi commands

| 项 | 内容 |
|----|------|
| 状态 | [事实] 接口已确认，具体命令序列 Partially verified |
| 来源 | `DriverAPI.inc`（`filament/backend/include/private/backend/DriverAPI.inc`） |
| 关键命令 | `beginFrame` (`:146`)、`endFrame` (`:165`)、`beginRenderPass` (`:565`)、`commit` (`:594`)、`createVertexBuffer` (`:188`)、`createTexture` (`:225`) |
| 说明 | DriverApi 是 CommandStream 的类型别名/封装，Engine/Renderer/FrameGraph 通过它录制命令 |

**待跟踪**：一帧中 DriverApi 命令的实际录制顺序与分类（资源创建 vs 状态设置 vs draw call vs 同步）。

### 8. CommandStream

| 项 | 内容 |
|----|------|
| 状态 | [事实] 机制已确认 |
| 来源 | `CommandStream` (`filament/backend/include/private/backend/CommandStream.h:196`) |
| 机制 | 将 DriverApi 调用序列化进 `CircularBuffer`（placement-new 命令对象，`:212-241` 宏生成），渲染线程 `execute(buffer)` (`:252`) 经 `Dispatcher` 分发到具体 `Driver` |
| 辅助 | `queueCommand(std::function<void()>)` (`:258`)、`allocate()` (`:265`) |

**待跟踪**：
- `CircularBuffer` 的内存管理与多线程模型
- `Dispatcher` 如何将命令映射到 Driver 虚方法
- 录制线程与渲染线程的同步点

### 9. VulkanDriver / OpenGLDriver

| 项 | 内容 |
|----|------|
| 状态 | [事实] 类定义已确认 |
| Vulkan | `VulkanDriver` (`filament/backend/src/vulkan/VulkanDriver.h:61`)，`final : public DriverBase` |
| OpenGL | `OpenGLDriver` (`filament/backend/src/opengl/OpenGLDriver.h:85`)，`final : public OpenGLDriverBase` |
| 继承 | 均经 `DriverBase` (`backend/src/DriverBase.h:189`) → `Driver` (`private/backend/Driver.h:64`) |
| 说明 | CommandStream 的 `execute` 最终调用具体 Driver 的虚方法，转为 Vulkan/GL 调用 |

**待跟踪**：
- `VulkanDriver::beginRenderPass` 等关键方法的实现
- Vulkan 命令缓冲录制与提交
- OpenGL 的立即模式语义与 Vulkan 延迟模式的对照

## 下一步需跟踪的符号清单

| 符号 | 文件 | 目的 |
|------|------|------|
| `FRenderer::renderJob` | `filament/src/details/Renderer.cpp:755` | 帧渲染核心编排 |
| `FView::prepare` 内部 | `filament/src/details/View.h:121` | View 准备流程 |
| `FView::renderShadowMaps` | `filament/src/details/View.h:235` | 阴影 Pass 编排 |
| `FrameGraph::compile` | `filament/src/fg/FrameGraph.h:313` | Pass 裁剪算法 |
| `FrameGraph::execute` | `filament/src/fg/FrameGraph.h:320` | Pass → DriverApi 命令 |
| `CommandStream::execute` | `filament/backend/include/private/backend/CommandStream.h:252` | 命令分发机制 |
| `Dispatcher` | `filament/backend/src/`（待定位） | 命令到 Driver 虚方法映射 |
| `FEngine` Driver 初始化 | `filament/src/details/Engine.h:135` | Engine → Driver 创建链 |

## 建议的下一步专题（二选一）

1. **从 Sample 跟踪到 `Renderer::render` 的完整帧调用链**
   - 跟踪 hellotriangle 帧回调 → `beginFrame` → `render` → `renderInternal` → `renderJob`
   - 确认 View prepare 与 FrameGraph 构建在 renderJob 中的位置
   - 产出：完整的、源码确认的一帧时序图

2. **分析 `DriverApi → CommandStream → VulkanDriver` 的命令提交路径**
   - 跟踪 DriverApi 调用如何被 CommandStream 序列化
   - `execute` 如何经 Dispatcher 分发到 VulkanDriver 虚方法
   - VulkanDriver 如何转为 Vulkan 命令缓冲
   - 产出：命令录制到 GPU 提交的完整路径文档

> 本次不开始执行下一专题，仅给出建议。
