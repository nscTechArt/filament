# study/ Agent 研究规则

本文件为 `study/` 目录下的 Coding Agent 研究规则，**不覆盖**仓库根目录 `AGENTS.md`。
如与根目录规则冲突，以根目录 `AGENTS.md` 与 `skills/` 为准。

## 适用范围

本规则适用于在 `study/` 目录下进行 Filament 研究分析的所有 Agent。

## 研究纪律

1. **默认只研究当前任务指定的模块。** 不要一次性分析整个仓库。
2. **每次分析必须引用文件路径与符号**（类名/函数名，优先带行号）。
   - 正确示例：`FRenderer::render` 定义于 `filament/src/Renderer.cpp:NN`
   - 错误示例：`Renderer 有个 render 方法`（无路径无行号）
3. **所有结论必须基于实际源码**，不要仅凭类名或目录名猜测。
4. **不允许在未确认调用链前下结论。** 未确认的步骤必须标记：
   - `Status: Unverified`
   - `Status: Partially verified`
   - 并列出下一步需要跟踪的符号。
5. **明确区分事实与推断：**
   - `[事实]`：源码中已确认
   - `[推断]`：基于源码的架构推断，需说明依据
6. **完成分析后更新对应研究文档**，保持文档与源码同步。

## 修改纪律

1. **第一阶段不直接大规模修改或删除 Filament 源码。**
2. 如需源码实验（加日志、插桩、验证假设），必须放在**独立实验 commit**中。
3. **文档修改和源码实验不要混在同一个 commit。**
4. 不要删除 Android、iOS、Metal、WebGL、WebGPU 或其他暂时不研究的平台代码。
5. 不要修改许可证、版权声明和第三方依赖声明。
6. 不要修改根目录 `AGENTS.md` 与 `skills/` 目录。
7. 不要修改 Git 历史（不 rebase、reset、force push、删除分支）。

## 与根目录 AGENTS.md 的关系

- 根目录 `AGENTS.md` 定义仓库级 AI 编码指令与技能引用。
- 本文件仅补充研究流程纪律，不重复也不覆盖根目录规则。
- 冲突时以根目录 `AGENTS.md` 为准。
- 根目录引用的关键技能（Agent 在修改 C++ 时仍需遵守）：
  - `skills/cpp_header_inclusion/SKILL.md`
  - `skills/header_self_containment/SKILL.md`
  - `skills/preprocessor_guard_hygiene/SKILL.md`
  - `skills/cpp_static_thread_safety/SKILL.md`
  - `skills/verification_protocols/SKILL.md`
  - `skills/filament_build_clean/SKILL.md`
  - `skills/bindings_synchronization/SKILL.md`

## 文档更新约定

- 研究文档位于 `study/` 下，按模块分目录。
- 每次完成一个模块的分析后，更新对应文档。
- 新增结论时标注来源（文件:行号）与状态（事实/推断/待验证）。
- 不为填满文档而编造结论；未确认的部分留空并标记状态。
