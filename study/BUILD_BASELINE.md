# 构建基线

> 本文档记录 Filament `v1.73.0` 在当前环境的构建状态。
> 区分 **[已验证]**（实际执行确认）与 **[待验证]**（未执行，标记为建议方案）。

## 仓库基线

| 项目 | 值 | 来源 |
|------|----|----|
| 分支 | `study/v1.73.0` | `git branch --show-current` |
| Commit | `d006c55032a8ff0f0efc8321b76016684e34c5e8` | `git log -1` |
| Tag | `v1.73.0`（精确匹配） | `git describe --tags --exact-match` |
| 工作区 | 有预存修改（非本次研究产生） | `git status --porcelain` 显示 `third_party/`、`tools/`、`web/` 下大量 ` M` 修改，均为本次研究开始前已存在的改动。本次研究唯一新增为 `?? study/`（未跟踪）。 |
| origin | `https://github.com/nscTechArt/filament.git` | `git remote -v` |
| upstream | `https://github.com/google/filament.git` | `git remote -v` |

## 操作系统与工具链

| 项目 | 值 | 状态 |
|------|----|----|
| OS | WSL2 Ubuntu 26.04 LTS（Linux 6.18.33.2-microsoft-standard-WSL2 x86_64） | [已验证] |
| CMake | **缺失** | [已验证] |
| Ninja | **缺失** | [已验证] |
| clang | **缺失** | [已验证] |
| gcc | **缺失** | [已验证] |
| MSVC (cl) | **缺失**（WSL 内） | [已验证] |
| Python | 3.14.4 | [已验证] |
| VULKAN_SDK | **未设置** | [已验证] |
| NASM | **缺失** | [已验证] |

> **结论：当前环境无法构建。** 缺少 CMake、Ninja、C++ 编译器与 Vulkan SDK。
> 用户选择不安装工具链，因此构建部分标记为 **[待验证]**。

## 构建方式（源码确认）

### 仓库封装脚本 `build.sh`

来源：`skills/filament_build_clean/SKILL.md`、`BUILDING.md:43-63`

```bash
# Debug 构建（开发调试，增量）
./build.sh debug

# Release 构建（性能分析，增量）
./build.sh release

# 安装到 out/debug/ 或 out/release/
./build.sh -i debug

# 增量 + 安装
./build.sh -ip desktop debug

# 全量清理
./build.sh -C
```

### 手动 CMake + Ninja（Linux）

来源：`BUILDING.md:119-134`

```bash
mkdir out/cmake-release
cd out/cmake-release
CC=/usr/bin/clang CXX=/usr/bin/clang++ CXXFLAGS=-stdlib=libc++ \
  cmake -G Ninja -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_INSTALL_PREFIX=../release/filament ../..
ninja
```

### 手动 CMake（Windows / Visual Studio）

来源：`BUILDING.md:210-232`

```bat
mkdir out
cd out
cmake ..
:: 打开 TNT.sln，或命令行构建指定 target：
cmake --build . --target hellotriangle --config Release
```

> Windows 注意：MSYS2 不支持（`CMakeLists.txt:14-19` 会 fatal error）；
> 需 Visual Studio 2019+，使用 `x64 Native Tools Command Prompt`。

## CMake 选项（源码确认）

来源：`CMakeLists.txt`，行号已标注。

### Backend 选项

| 选项 | 默认值 | 效果 | 来源 |
|------|--------|------|------|
| `FILAMENT_SUPPORTS_OPENGL` | `ON` | 包含 OpenGL/ES backend | `CMakeLists.txt:584` |
| `FILAMENT_SUPPORTS_VULKAN` | Linux/macOS: `ON`；Windows/WASM/iOS: `OFF` | 包含 Vulkan backend | `CMakeLists.txt:600-604` |
| `FILAMENT_SUPPORTS_METAL` | Apple 非 WASM: `ON`；其他: `OFF` | 包含 Metal backend | `CMakeLists.txt:618-622` |
| `FILAMENT_SUPPORTS_WEBGPU` | 未在根 CMakeLists 定义（默认 OFF） | 包含 WebGPU backend | 仅 `android/*/CMakeLists.txt:8` |

**Vulkan 默认值逻辑**（`CMakeLists.txt:600-604`）：

```cmake
if (WIN32 OR WASM OR IOS)
    option(FILAMENT_SUPPORTS_VULKAN "Include the Vulkan backend" OFF)
else()
    option(FILAMENT_SUPPORTS_VULKAN "Include the Vulkan backend" ON)
endif()
```

> **[事实]** 在 Linux 桌面（本研究环境），Vulkan backend **默认 ON**，无需显式启用。
> **[事实]** 在 Windows 桌面，Vulkan backend **默认 OFF**，需 `-DFILAMENT_SUPPORTS_VULKAN=ON` 显式启用。
> **[事实]** OpenGL backend 在所有平台**默认 ON**。

### Sample / 测试选项

| 选项 | 默认值 | 效果 | 来源 |
|------|--------|------|------|
| `FILAMENT_SKIP_SAMPLES` | `OFF` | 跳过 sample 构建 | `CMakeLists.txt:62` |
| `FILAMENT_BUILD_TESTING` | `ON` | 构建测试 | `CMakeLists.txt:98` |
| `FILAMENT_ENABLE_FGVIEWER` | `OFF` | FrameGraph viewer | `CMakeLists.txt:86` |
| `FILAMENT_ENABLE_MATDBG` | Debug+Host: `ON`；其他: `OFF` | 材质调试器 | `CMakeLists.txt:636-639` |

### 其他关键选项

| 选项 | 默认值 | 效果 | 来源 |
|------|--------|------|------|
| `FILAMENT_SUPPORTS_XCB` | `ON` | Linux XCB 窗口 | `CMakeLists.txt:64` |
| `FILAMENT_SUPPORTS_XLIB` | `ON` | Linux XLIB 窗口 | `CMakeLists.txt:66` |
| `FILAMENT_SUPPORTS_EGL_ON_LINUX` | `OFF` | Linux 用 EGL | `CMakeLists.txt:68` |
| `FILAMENT_SKIP_SDL2` | `OFF` | 跳过 SDL2 依赖 | `CMakeLists.txt:72` |
| `FILAMENT_ENABLE_RTTI` | `OFF` | C++ RTTI | `CMakeLists.txt:59` |
| `FILAMENT_ENABLE_EXCEPTIONS` | `ON`（iOS OFF） | C++ 异常 | `CMakeLists.txt:57` |

## 最小调试入口 Sample

| 属性 | 值 | 来源 |
|------|----|----|
| CMake target | `hellotriangle` | `samples/CMakeLists.txt:276` |
| 源码路径 | `samples/hellotriangle.cpp` | — |
| 链接库 | `sample-resources`、`filamentapp`、`samples-common` | `samples/CMakeLists.txt:254` |
| 构建命令 | `cmake --build <build-dir> --target hellotriangle` | — |

> **[事实]** `hellotriangle` 是最简单的 sample，通过 `add_demo(hellotriangle)` 注册
> （`samples/CMakeLists.txt:276`，`add_demo` 定义于 `samples/CMakeLists.txt:251-263`）。
> 它创建 Engine + Renderer + View + Scene，适合作为最小调试入口。

### 其他可选 sample

| target | 说明 | 来源 |
|--------|------|------|
| `gltf_viewer` | GLTF 查看器，功能更全 | `samples/CMakeLists.txt:269` |

## Backend 运行时切换

来源：`samples/common/arguments.cpp`、`filamentapp/FilamentApp.cpp`、`filamentapp/PlatformHelper.cpp`

### 命令行参数

```bash
./hellotriangle -a opengl      # 强制 OpenGL
./hellotriangle --api vulkan   # 强制 Vulkan
./hellotriangle                # 使用平台默认
```

**[事实]** 参数解析：`samples/hellotriangle.cpp:95-117` 调用 `handleCommandLineArguments()`，
经 `getopt_long` 解析 `--api` / `-a`，调用 `samples::parseArgumentsForBackend()`。

**[事实]** 字符串映射（`samples/common/arguments.cpp`）：

| 字符串 | Backend 枚举 | 来源 |
|--------|-------------|------|
| `"opengl"` | `Backend::OPENGL` | `arguments.cpp:60` |
| `"vulkan"` | `Backend::VULKAN` | `arguments.cpp:62` |
| `"metal"` | `Backend::METAL` | `arguments.cpp:58` |
| `"webgpu"` | `Backend::WEBGPU` | `arguments.cpp:64` |
| 未指定 | `Backend::DEFAULT`（值 0） | `arguments.cpp:96` |

### 默认 Backend 解析

**[事实]** `Backend::DEFAULT` 的解析链：

1. `FilamentApp::get().run()` 调用 `filament::app::resolveBackend()`（`FilamentApp.cpp:114`）
2. `resolveBackend()` 定义于 `PlatformHelper.cpp:36-51`，按平台选择：
   - Apple → `METAL`
   - `__EMSCRIPTEN__` 或 `__ANDROID__` → `OPENGL`
   - `FILAMENT_DRIVER_SUPPORTS_VULKAN` 定义 → `VULKAN`
   - `FILAMENT_SUPPORTS_WEBGPU` 定义 → `WEBGPU`（fallback）
3. 解析结果传给 `Engine::Builder().backend(backend)`（`FilamentApp.cpp:141`）

**[事实]** Linux 桌面（Vulkan 编译启用时）默认 Backend = **VULKAN**。
**[事实]** Windows 桌面（Vulkan 默认 OFF）默认 Backend = **OPENGL**，除非显式 `-DFILAMENT_SUPPORTS_VULKAN=ON`。

## CMake Target 名称

| Target | 类型 | 说明 | 定义位置 |
|--------|------|------|----------|
| `filament` | STATIC | 主 Filament 库 | `filament/CMakeLists.txt:4` |
| `backend` | STATIC | Backend 抽象（含各 Driver） | `filament/backend/CMakeLists.txt:4` |
| `filamentapp` | STATIC | Sample 框架（SDL2） | `libs/filamentapp/CMakeLists.txt:8` |
| `samples-common` | STATIC | Sample 共享参数解析 | `samples/CMakeLists.txt:226` |
| `sample-resources` | STATIC | 编译后的材质资源 | `samples/CMakeLists.txt:105` |
| `hellotriangle` | EXECUTABLE | 最小 sample | `samples/CMakeLists.txt:276` |
| `gltf_viewer` | EXECUTABLE | GLTF 查看器 | `samples/CMakeLists.txt:269` |

> **[事实]** Vulkan backend **不是独立 target**，当 `FILAMENT_SUPPORTS_VULKAN=ON` 时
> 编译进 `backend` 静态库（`filament/backend/CMakeLists.txt:118-282`）。
> OpenGL backend 同理（`filament/backend/CMakeLists.txt:70-110`）。

## 建议构建方案（待验证）

> 以下命令**未实际执行**，标记为 **[待验证]**。需先安装工具链。

### 前置依赖（Linux / WSL2）

来源：`BUILDING.md:100-111`

```bash
# CMake 3.22.1+、clang 17+、ninja 1.10+
sudo apt-get install cmake ninja-build clang-17 \
  libc++-17-dev libc++abi-17-dev \
  libglu1-mesa-dev libxi-dev libxcomposite-dev libxxf86vm-dev
```

> Vulkan SDK：Linux 下 Vulkan headers 通常由 Mesa 提供；若需 validation layer，
> 需安装 Vulkan SDK 或 `libvulkan-dev`。当前 `VULKAN_SDK` 未设置。

### Debug 构建（最小 sample）

```bash
# 方式 A：仓库脚本（推荐，封装了编译器与 libc++ 设置）
./build.sh -ip desktop debug

# 方式 B：手动 CMake + Ninja
mkdir out/cmake-debug && cd out/cmake-debug
CC=/usr/bin/clang CXX=/usr/bin/clang++ CXXFLAGS=-stdlib=libc++ \
  cmake -G Ninja -DCMAKE_BUILD_TYPE=Debug \
        -DCMAKE_INSTALL_PREFIX=../debug/filament ../..
ninja hellotriangle
```

### 只构建指定 target

```bash
# 在 build 目录内
ninja hellotriangle          # 只构建最小 sample
# 或
cmake --build . --target hellotriangle --config Debug
```

### 裁剪构建范围（建议方案）

为加快研究构建速度，可显式关闭无关 backend 与测试：

```bash
cmake -G Ninja \
  -DCMAKE_BUILD_TYPE=Debug \
  -DFILAMENT_SUPPORTS_VULKAN=ON \
  -DFILAMENT_SUPPORTS_OPENGL=ON \
  -DFILAMENT_SUPPORTS_METAL=OFF \
  -DFILAMENT_BUILD_TESTING=OFF \
  -DFILAMENT_ENABLE_FGVIEWER=OFF \
  -DFILAMENT_ENABLE_MATDBG=OFF \
  ../..
```

> **[待验证]** 上述裁剪配置未实际执行。`FILAMENT_SUPPORTS_WEBGPU` 在桌面默认不定义，
> 无需显式关闭。Metal 在非 Apple 平台默认 OFF。

### 运行 sample 并切换 Backend

```bash
# Vulkan（Linux 桌面默认）
./out/debug/filament/bin/hellotriangle -a vulkan

# OpenGL（对照）
./out/debug/filament/bin/hellotriangle -a opengl

# 默认（Linux = Vulkan）
./out/debug/filament/bin/hellotriangle
```

> **[待验证]** 实际安装路径以 `build.sh -i` 的 `out/debug/filament/` 为准，
> 未实际执行确认。

## 构建结果

| 项目 | 状态 |
|------|------|
| Debug 构建 | **未执行**（缺工具链） |
| `hellotriangle` target | **未执行** |
| Vulkan 运行验证 | **未执行** |
| OpenGL 运行验证 | **未执行** |

## 已知问题

1. **工具链缺失**：WSL2 环境无 cmake/ninja/clang/gcc，无法构建。
   - 最小修复：`sudo apt-get install cmake ninja-build clang-17 libc++-17-dev libc++abi-17-dev libglu1-mesa-dev libxi-dev libxcomposite-dev libxxf86vm-dev`
   - 可选：安装 `libvulkan-dev` 或 Vulkan SDK 以获得 validation layer。
2. **VULKAN_SDK 未设置**：Linux 桌面构建 Vulkan backend 不强制需要 SDK
   （Vulkan headers 由 Mesa 提供），但运行时需要 Vulkan loader（`libvulkan1`）
   与可用的 Vulkan ICD（如 Mesa RADV/NVIDIA 驱动）。WSL2 下 Vulkan 需
   Vulkan-on-DX12 转译（如 DXC/dzn ICD），可用性待确认。
3. **WSL2 图形**：sample 需要 SDL2 窗口系统。WSL2 支持 WSLg（X11/Wayland 转发），
   但需确认 SDL2 能在 WSLg 下正常创建窗口。

## 下一步（构建验证）

当工具链就绪后，按以下顺序验证：

1. 安装依赖 → `cmake --version` / `ninja --version` / `clang-17 --version` 确认。
2. `./build.sh -ip desktop debug` 或手动 CMake 配置 + `ninja hellotriangle`。
3. 运行 `./out/debug/filament/bin/hellotriangle -a vulkan` 确认 Vulkan。
4. 运行 `./out/debug/filament/bin/hellotriangle -a opengl` 确认 OpenGL。
5. 将真实结果回填至本文档"构建结果"与"运行验证"小节，并改为 **[已验证]**。
