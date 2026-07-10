# Filament 研究基线

本目录是 Technical Artist 视角下对 Google Filament 的长期研究工作区。
研究分支：`study/v1.73.0`（精确匹配 tag `v1.73.0`，commit `d006c55`）。

## 为什么研究 Filament

我已有两个作品集项目：

1. **Unity 自定义 SRP** — 了解上层渲染管线编排与脚本化抽象。
2. **C++ Vulkan 渲染器** — 了解底层 GPU API、资源屏障、命令录制。

Filament 填补两者之间的关键空白：一个**生产级、跨平台、有完整 FrameGraph 与多 Backend 的实时渲染器**。
它同时具备：

- 清晰的 Public API / details 实现分层（`filament/include` vs `filament/src/details`）
- 显式 FrameGraph（`filament/src/fg`）做渲染 Pass 编排与资源生命周期管理
- 统一 Driver API + CommandStream 抽象，下接 Vulkan / OpenGL / Metal / WebGPU
- 工业级 PBR、阴影（Froxel/PCF/VSM）、后处理实现
- `matc` 材料编译器与运行时 Material / MaterialInstance 体系

## 与现有项目的关系

| 项目 | 关注层 | Filament 对照点 |
|------|--------|-----------------|
| Unity SRP | 管线编排、RenderPass 调度 | FrameGraph、Renderer::render |
| Vulkan Renderer | 命令录制、资源屏障、同步 | DriverApi → CommandStream → VulkanDriver |
| Filament | 两者之间的完整工程闭环 | 本研究的主体 |

## 文档导航

| 文档 | 内容 |
|------|------|
| [STUDY_SCOPE.md](STUDY_SCOPE.md) | 研究范围、排除项、Agent 分析规则、事实/推断区分 |
| [BUILD_BASELINE.md](BUILD_BASELINE.md) | 构建基线：环境、工具版本、CMake 配置、构建命令与结果 |
| [MODULE_MAP.md](MODULE_MAP.md) | 一级模块地图：职责、关键类型、目录、关系 |
| [FRAME_TRACE.md](FRAME_TRACE.md) | 一帧调用链框架（待逐项验证） |
| [GLOSSARY.md](GLOSSARY.md) | Filament 专用术语表（源码语义优先） |
| [AGENTS.md](AGENTS.md) | study 目录下的 Agent 研究规则 |
| [backend/](backend/) | Driver API、Vulkan Backend、OpenGL Backend |
| [framegraph/](framegraph/) | FrameGraph 与资源生命周期 |
| [material/](material/) | Material 管线与 Shader 变体 |
| [diagrams/](diagrams/) | 架构图（后续补充） |

## 当前研究阶段

**阶段 0：研究基线建立（本次完成）**

- 仓库状态确认
- 构建方式分析（文档级，未实际构建）
- 模块地图一级
- 一帧调用链框架
- 术语表初版

**阶段 1（建议下一步，二选一）：**

1. 从 Sample 跟踪到 `Renderer::render` 的完整帧调用链
2. 分析 `DriverApi → CommandStream → VulkanDriver` 的命令提交路径

## 如何使用这些文档

- 所有结论标注来源：`文件路径:行号` + 类名/函数名。
- 区分 **[事实]**（源码确认）与 **[推断]**（基于源码的架构推断）。
- 未验证内容显式标记 `Status: Unverified` 或 `Status: Partially verified`。
- 文档随研究推进迭代更新，不追求一次填满。

## 已确认的构建入口

仓库根目录 `build.sh` 封装脚本（见 `skills/filament_build_clean/SKILL.md`）：

```bash
# Debug 构建（开发调试）
./build.sh -ip desktop debug

# Release 构建（性能分析）
./build.sh -ip desktop release

# 全量清理
./build.sh -C
```

> 当前环境（WSL2 Ubuntu 26.04）缺少 cmake/ninja/clang/Vulkan SDK，
> 尚未实际执行构建。详见 [BUILD_BASELINE.md](BUILD_BASELINE.md)。
