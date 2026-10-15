# CMake、Ninja 与工具链

## 使用工程实际入口

先读 CMakeLists、CMakePresets/CMakeUserPresets、工具链文件和构建说明。查出所需目标及配置，确认工具：

```text
cube cmake --version
cube ninja --version
cube cmake --list-presets
```

有 configure/build preset 时分别使用真实名称：

```text
cube cmake --preset "<配置preset>"
cube cmake --build --preset "<构建preset>" --parallel
```

configure preset 与 build preset 名称不一定相同。没有 build preset 时，使用 configure preset 对应的 binaryDir 进行 `--build`。没有 preset 时依据工程的工具链文件配置：

```text
cube cmake -S "<源码目录>" -B "<构建目录>" -G Ninja -DCMAKE_TOOLCHAIN_FILE="<真实工具链文件>" -DCMAKE_BUILD_TYPE=Debug
cube cmake --build "<构建目录>" --parallel
```

为新工程选择 CMake + Ninja + GCC 时检查生成器已经提供的工具链和目标设置，避免覆盖正确的交叉编译配置。已有多配置生成器需用其实际 `--config`，而不是强行依赖 `CMAKE_BUILD_TYPE`。

## 编译器解析

1. 尚无可靠编译器路径时按[工具定位](../../stm32-env/references/tool-location.md)找到工程要求的 GCC/Clang；不要仅依赖 shell 中碰巧找到的版本。
2. 检查工具链文件如何取编译器路径，配置前向当前工具子进程提供必要 PATH 或指定的工具链变量。
3. 读取 CMakeCache 中的 `CMAKE_C_COMPILER`、`CMAKE_ASM_COMPILER`、generator 和 toolchain，检查实际编译命令。

`cube` 的工具解析不代表 shell 中所有编译器都可直接调用。编译器 Bundle 可能只导出路径；不要使用未注册的 `cube arm-none-eabi-gcc`。需要 `size`、`objcopy` 时使用所选 GNU 工具链的真实程序。

例如工具链文件通过 PATH 查找 GCC，可只对本次配置/构建提供 bin 路径。先用定位到的编译器查询版本和目标：

```powershell
$compilerBin = '<已选GNU工具链bin绝对路径>'
& (Join-Path $compilerBin 'arm-none-eabi-gcc.exe') --version
& (Join-Path $compilerBin 'arm-none-eabi-gcc.exe') -dumpmachine
$savedTaskPath = $env:PATH
try {
    $env:PATH = $compilerBin + [IO.Path]::PathSeparator + $savedTaskPath
    & cube cmake --preset '<工程真实配置preset>'
    if ($LASTEXITCODE -ne 0) { throw 'CMake 配置失败，先处理诊断。' }
    & cube cmake --build '<该preset的binaryDir绝对路径>' --parallel
    if ($LASTEXITCODE -ne 0) { throw '构建失败，检查首个有效错误。' }
} finally {
    $env:PATH = $savedTaskPath
}
```

POSIX shell 对每次调用传入环境：

```bash
compiler_bin="<已选GNU工具链bin绝对路径>"
"$compiler_bin/arm-none-eabi-gcc" --version
"$compiler_bin/arm-none-eabi-gcc" -dumpmachine
PATH="$compiler_bin:$PATH" cube cmake --preset "<真实配置preset>" &&
PATH="$compiler_bin:$PATH" cube cmake --build "<该preset的binaryDir>" --parallel
```

工具链文件若使用 `GCC_TOOLCHAIN_ROOT` / `TOOLCHAIN_ROOT` 或绝对编译器变量，读取其拼接逻辑后提供正确值；不同工程可能要求安装根目录或 bin 目录，不能两者混用。Ninja 无法解析时同样采用已定位的程序路径或 `CMAKE_MAKE_PROGRAM`。未提供 preset 的工程使用上文 `-S/-B` 模板。

切换编译器、目标或 generator 后使用新的构建目录，或仅清理已确认的对应缓存。不要删除整个工程或其他目标的产物。带空格的路径和含 `-D` 的参数作为单独参数传递。

## 常见问题

- 找不到 Ninja：按[工具定位](../../stm32-env/references/tool-location.md)检查程序与 CMake 进程 PATH，必要时使用已发现的 `CMAKE_MAKE_PROGRAM`。
- CMake 版本或 policy 不兼容：先沿用工程约束的工具版本，检查真实最低版本；不盲目改生成文件或关闭诊断。
- 构建旧固件：核对源码/构建目录、目标、配置、文件时间及实际编译输入。
- `.settings` 约束不生效：检查运行目录和父目录，重新查看 `cube --resolve` 的实际结果。

依据：[CMake 手册](https://cmake.org/cmake/help/latest/manual/cmake.1.html)、[Presets](https://cmake.org/cmake/help/latest/manual/cmake-presets.7.html)、当前工程生成文件和工具帮助。
