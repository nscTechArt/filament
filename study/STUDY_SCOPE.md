# 研究范围

研究分支：`study/v1.73.0`（tag `v1.73.0`，commit `d006c55`）

## 研究目标

把 Filament 作为**生产级实时渲染器参考**，重点理解：

- 现代实时渲染器的整体架构与分层
- FrameGraph：渲染 Pass 编排与资源生命周期
- Material / MaterialInstance / `matc` 材料管线
- Driver API 与 CommandStream 命令提交抽象
- Vulkan Backend（主要研究对象）
- OpenGL Backend（作为 Vulkan 的对照）
- Renderer、View、Scene 与一帧渲染流程
- PBR、阴影、后处理等具体系统的工程实现

## 主要研究目录

| 目录 | 关注点 |
|------|--------|
| `filament/src` | Engine/Renderer/View/Scene 实现层 |
| `filament/src/fg` | FrameGraph |
| `filament/backend/include` | Driver API 公共接口 |
| `filament/backend/src` | CommandStream 与 Backend 公共实现 |
| `filament/backend/src/vulkan` | Vulkan Backend（主要） |
| `filament/backend/src/opengl` | OpenGL Backend（对照） |
| `libs/filabridge` | Engine 与 Backend 之间的桥接层 |
| `libs/utils` | 工具库（线程安全、JobSystem、EntityManager 等） |
| `libs/math` | 数学库（header-only） |
| 桌面端 Samples / `filamentapp` | 最小调试入口 |

## 次要 / 对照目录

以下目录在确认抽象设计时作为对照，不作为主要研究对象：

- `filament/backend/src/metal` — Metal Backend
- `filament/backend/src/webgpu` — WebGPU Backend
- Android / iOS 应用层与 JNI 接入
- WebGL / WebAssembly 与 JavaScript/TypeScript Binding
- Java/Kotlin Binding

## 暂时排除

- 与当前问题无关的第三方库内部实现（`third_party/`）
- Android 应用层和 JNI 接入
- iOS 应用层
- Metal / WebGPU / WebGL / WebAssembly
- JavaScript / Java / Kotlin Binding

> 这些代码**不删除、不修改**，仅在需要确认跨平台抽象设计时作为对照。

## Agent 分析规则

1. **默认只研究当前任务指定的模块**，不一次性分析整个仓库。
2. **每次分析必须引用文件路径与符号**（类名/函数名，最好带行号）。
3. **所有结论必须基于实际源码**，不要仅凭类名或目录名猜测。
4. **明确区分**：
   - `[事实]`：源码中已确认的内容
   - `[推断]`：基于源码做出的架构推断
   - `Status: Unverified` / `Status: Partially verified`：尚未完全确认
5. **完成分析后更新对应研究文档**，保持文档与源码同步。
6. **不要自动删除暂时不研究的平台代码**。
7. 遵守根目录 `AGENTS.md` 与 `skills/` 下的技能指令；如冲突，以根目录规则为准。

## 修改源码的边界

- **第一阶段不直接大规模修改或删除 Filament 源码。**
- 研究以阅读、文档化、调用链索引为主。
- 如需源码实验（如加日志、插桩），必须放在**独立实验 commit**中，与文档 commit 分离。
- 文档修改和源码实验**不要混在同一个 commit**。
- 不要修改许可证、版权声明和第三方依赖声明。
- 不要修改 `AGENTS.md`（根目录）与 `skills/` 目录。

## 事实与推断的记录格式

在研究文档中使用以下标记：

```text
[事实] Renderer::beginFrame 定义于 filament/src/Renderer.cpp:NN
[推断] beginFrame 负责帧起始的资源同步，依据是它调用了 DriverApi 的 wait/poll 类命令
Status: Partially verified — 已确认入口，未确认完整调用链
```
