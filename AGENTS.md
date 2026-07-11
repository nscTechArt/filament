# Filament AI Coding & Agent Guidelines

## Build & Verify

- **Desktop debug build** (with install): `./build.sh -ip desktop debug`
- **Desktop release build**: `./build.sh -ip desktop release`
- **Full clean**: `./build.sh -C`
- **Single target**: `./build.sh debug gltf_viewer`
- Build output goes to `out/cmake-debug/` or `out/cmake-release/`.

## Include Headers

Never sort `#include` directives by hand. After any C++ change, run:
```
./tools/reorganize-headers/run.py <file_or_directory>
```
This tool parses CMakeLists.txt to topologically order proprietary-library includes and respects preprocessor-guarded blocks. See [skills/cpp_header_inclusion/SKILL.md](skills/cpp_header_inclusion/SKILL.md).

## Verification Pipeline

Before concluding any C++ task, execute in order:
1. **Format includes** with `./tools/reorganize-headers/run.py` on affected files.
2. **Compile desktop debug** with `./build.sh -ip desktop debug`. Must be warning-free.
3. **Run core tests** from `build/common/test_list.txt` (e.g., `./out/cmake-debug/libs/utils/test_utils`).

See [skills/verification_protocols/SKILL.md](skills/verification_protocols/SKILL.md).

## Tests & Benchmarks

- Tests use Google Test. Filter: `--gtest_filter=Suite.TestName`
- Benchmarks use Google Benchmark. Filter: `--benchmark_filter=name`
- Key test binaries:
  - `./out/cmake-debug/libs/utils/test_utils`
  - `./out/cmake-debug/filament/test/test_filament`
  - `./out/cmake-debug/libs/math/test_math`
- Full test list: `build/common/test_list.txt`

## Code Style

- 4-space indent, 100-column limit, `{` at end of line
- Headers: `.h`, sources: `.cpp`, inlined: `.inc`
- Public headers live under `include/<lib>/`, private under `src/`
- `#include < >` for public/exported headers, `#include " "` for private
- No spaces in filenames; filenames are case-insensitive
- See full rules in [CODE_STYLE.md](CODE_STYLE.md).

## Header Discipline

- Every `.h` must compile independently (self-containment). Include what you use.
- Preprocessor guards must not be moved/reordered by hand — the reorganize tool handles them.
- Thread-safety annotations (`UTILS_GUARDED_BY`, etc.) are required on shared state.

## Language Bindings

When public C++ APIs change (`filament/include/filament/`):
- **Options structs** (`Options.h`): run `cd tools/beamsplitter && go run .` to auto-update Java/JS bindings and JSON serialization.
- **Other API changes**: manually update Java (`android/filament-android/src/main/java/...`) and JNI (`android/filament-android/src/main/cpp/...`), plus JS/TS (`web/filament-js/`).
- See [skills/bindings_synchronization/SKILL.md](skills/bindings_synchronization/SKILL.md).

## CI & Repo Constraints

- `.github/` workflows are restricted to maintainers. External PRs touching them are auto-closed.
- `docs/` is generated from `docs_src/`. Edit source in `docs_src/`, not `docs/`.
- Presubmit runs only desktop macOS and Linux builds (not Android/iOS/Web).

## Platform-Specific

- **Windows**: Only Visual Studio 2019+ native tools. No MSYS2 support. Open `TNT.sln` from CMake output.
- **Android**: Requires `ANDROID_HOME`, NDK 29, Java 17. Build: `./build.sh -p android -q arm64-v8a release`.
- **Web**: Requires `EMSDK` env var. Emscripten 5.0.4. Build: `./build.sh -p webgl release`.
- **iOS**: Build: `./build.sh -p ios debug`. XCFrameworks created automatically.

## Commit Conventions

- Commit title ≤ 50 chars, body wrapped at 72 chars.
- Use full path for file references in commit messages.

## Study Directory

- `study/` is a separate research workspace (focused on code reading, architecture analysis, and documentation).
- `study/AGENTS.md` defines research-discipline rules (no mass source edits in Phase 1, always cite `file:line`, distinguish `[事实]` vs `[推断]`, separate doc commits from experiment commits). These rules apply when working under `study/`.
