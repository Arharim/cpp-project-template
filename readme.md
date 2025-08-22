# C++ Project Template

A minimal yet practical C++20 project template using CMake with cross-platform support, CMake Presets, optional warnings-as-errors, sanitizers, LTO, clang-tidy integration, and CTest. Suitable for starting new projects quickly while keeping tooling sane.

## Features
- C++20, no compiler-specific extensions (CMAKE_CXX_EXTENSIONS=OFF)
- CMake 3.20+ with CMakePresets.json for consistent builds
- Works with Ninja/GCC/Clang on Linux/macOS and MSVC on Windows
- Exported compile_commands.json for IDEs and clangd
- Optional: warnings-as-errors, Address/UBSan in Debug (Clang/GCC), Release LTO/IPO
- Optional: clang-tidy integration (reads .clang-tidy)
- CTest scaffold: auto-enables tests if tests/CMakeLists.txt exists
- Install target to standard bin dir (defaults to _install via presets)

## Requirements
- CMake >= 3.20
- A C++20 compiler
  - Linux/macOS: GCC or Clang
  - Windows: Visual Studio 2022 (MSVC)
- Ninja recommended (for Ninja presets). Alternatively, use your generator of choice.
- Optional developer tools: clangd, clang-tidy, clang-format

## Layout
- inc/               Public headers (exposed to consumers)
- src/               Source files (and private headers if any)
- tests/             Optional tests (only built if present)
- CMakeLists.txt     Project build configuration
- CMakePresets.json  Reproducible configure/build/test presets
- .clang-format      Code style (applies to all C/C++ sources)
- .clang-tidy        Static analysis rules
- .clangd            Language server configuration
- .gitignore

The default target name is MyProject.

## Quick start (Presets)
Presets place build artifacts into build/<presetName> and default installation prefix to _install.

List presets:
- cmake --list-presets

Linux/macOS (Ninja + default toolchain):
- Configure: cmake --preset ninja-debug
- Build:     cmake --build --preset ninja-debug
- Run:       ./build/ninja-debug/MyProject
- Test:      ctest --preset ninja-debug
- Install:   cmake --install build/ninja-debug

GCC or Clang explicitly:
- Debug GCC:     cmake --preset gcc-debug && cmake --build --preset gcc-debug
- Release GCC:   cmake --preset gcc-release && cmake --build --preset gcc-release
- Debug Clang:   cmake --preset clang-debug && cmake --build --preset clang-debug
- Release Clang: cmake --preset clang-release && cmake --build --preset clang-release

Windows (Visual Studio 2022):
- Configure: cmake --preset msvc-debug
- Build:     cmake --build --preset msvc-debug
- Run:       .\\build\\msvc-debug\\Debug\\MyProject.exe (or use Visual Studio)
- Test:      ctest --preset msvc-debug
- Install:   cmake --install build/msvc-debug

Note: MSVC is a multi-config generator; the preset specifies the configuration (Debug/Release) for build and test.

## Manual usage (no presets)
- Configure: cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Debug -DENABLE_WARNINGS=ON
- Build:     cmake --build build -j
- Test:      ctest --test-dir build --output-on-failure
- Install:   cmake --install build --prefix _install

If you don’t use Ninja, replace -G Ninja with your generator (e.g., "Unix Makefiles").

## Options
All options can be toggled at configure time with -D<OPTION>=ON/OFF.
- ENABLE_WARNINGS (default ON)
  - Enables -Wall -Wextra -Wpedantic (or /W4 on MSVC)
- ENABLE_WARNINGS_AS_ERRORS (default OFF)
  - Treats warnings as errors (-Werror or /WX)
- ENABLE_SANITIZERS (default OFF; Debug-only, non-MSVC)
  - Adds -fsanitize=address,undefined for Clang/GCC in Debug
- ENABLE_LTO (default OFF)
  - Enables IPO/LTO for Release if supported
- ENABLE_CLANG_TIDY (default OFF)
  - Runs clang-tidy during the build if clang-tidy is found

## Development tooling
- compile_commands.json
  - Generated in the build directory (CMAKE_EXPORT_COMPILE_COMMANDS=ON)
  - Some tools expect it in the source root; you can symlink it if needed:
    - ln -sf build/ninja-debug/compile_commands.json compile_commands.json
- Formatting: use the provided .clang-format file (run clang-format in your editor or via scripts/CI)
- Static analysis: .clang-tidy is configured; enable it by setting ENABLE_CLANG_TIDY=ON or run clang-tidy manually
- clangd: .clangd present for LSP configuration

## Tests
CTest is enabled when BUILD_TESTING is ON (default via include(CTest)). If tests/CMakeLists.txt exists, it will be added automatically. Typical structure:
- tests/
  - CMakeLists.txt
  - your test sources

Run tests with:
- ctest --preset <your-configure-preset>
- or: ctest --test-dir build/<dir> --output-on-failure

## Install
The default install prefix in presets is _install at the repo root. To install:
- cmake --install build/<presetName>
The binary will be placed in _install/bin.

To override the prefix without presets:
- cmake --install build --prefix /path/to/prefix

## Example output
The default program prints:
- Sup, C++!

## Customization
- Change the project name in CMakeLists.txt: project(MyProject VERSION 1.0 LANGUAGES CXX)
- Add new headers to inc/ and sources to src/ (they’re discovered via GLOB_RECURSE in src/)
- For libraries or more complex layouts, replace GLOB with explicit target sources and add subdirectories

## Troubleshooting
- compile_commands.json not found by your tools: point them to build/<presetName> or create a symlink at the repo root
- Build fails due to warnings: set -DENABLE_WARNINGS_AS_ERRORS=OFF
- Sanitizers on MSVC: not supported in this template (ignored when using MSVC)
- LTO not supported: CMake will print a warning and continue without LTO

## License
This project is licensed under the MIT License. See [LICENSE.md](LICENSE.md) for details.
