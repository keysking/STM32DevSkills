# 定位工具与 SDK

需要找到 cube、生成器、编译器、调试器或独立 IDE 时读取。先完成当前所需工具的定位，已有可用路径无需重复搜索。以下候选目录用于发现，不代表固定安装位置。

## 搜索顺序与选择依据

1. 在目标工程内检查说明、CI、构建脚本、presets、工具链文件和 `.settings`。已有工程版本约束优先；工程识别见[工程识别](../../stm32-project/references/identify-and-maintain.md)。
2. 查询 PATH 中的候选。使用全部结果辨认多个安装，不直接把第一项当作项目所需版本。
3. cube 可用时按[工具与依赖](tools-and-dependencies.md)解析对应命令和 Bundle；传统 CubeMX、Keil、IAR 等独立安装再查系统安装信息。
4. 检查安装记录指向的目录及下列候选根目录，仅在这些有限目录内搜索。自定义安装目录从工程声明、快捷方式或用户提供的位置取得，不扫描所有磁盘。
5. 检查候选的版本、入口类型、主机架构和项目要求。选定后用绝对路径或已确认的 cube 命令调用，记录必要路径及版本即可。

## Windows / PowerShell

### PATH 与安装记录

```powershell
Get-Command cube,STM32CubeMX,STM32_Programmer_CLI,arm-none-eabi-gcc,arm-none-eabi-gdb,UV4,IarBuild -All -ErrorAction SilentlyContinue |
    Select-Object Name,Source
where.exe cube
```

`where.exe` 查不到会返回非零状态，继续检查安装记录。Windows 软件不一定注册到 PATH：

```powershell
$uninstallKeys = @(
    'HKCU:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*',
    'HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*',
    'HKLM:\Software\WOW6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*'
)
Get-ItemProperty -Path $uninstallKeys -ErrorAction SilentlyContinue |
    Where-Object { $_.DisplayName -match 'STM32|CubeMX|Keil|IAR|Arm GNU' } |
    Select-Object DisplayName,DisplayVersion,InstallLocation,DisplayIcon
```

`DisplayIcon` 可能含引号、图标索引或参数，不能整段作为可执行文件路径。`InstallLocation` 为空时再检查候选根目录：

```powershell
$candidateRoots = @(
    (Join-Path $env:LOCALAPPDATA 'Programs'),
    (Join-Path $env:USERPROFILE '.local\stm32cube\bin'),
    $env:ProgramFiles,
    ${env:ProgramFiles(x86)},
    (Join-Path $env:SystemDrive 'ST'),
    (Join-Path $env:SystemDrive 'Keil_v5')
)
$candidateRoots | Where-Object { $_ -and (Test-Path -LiteralPath $_) } |
    ForEach-Object {
        Get-ChildItem -LiteralPath $_ -Directory -ErrorAction SilentlyContinue |
            Where-Object { $_.Name -match 'STM32|Cube|STMicro|Keil|IAR|Arm|GNU' } |
            Select-Object -ExpandProperty FullName
    }
```

这些根目录也可能直接包含程序，例如 cube 的 bin。先检查根目录中的目标文件，再检查列出的软件目录。将选中的目录替换到下例；不要对每个 Program Files 根目录无差别递归：

```powershell
$selectedRoot = '<已确认的软件安装目录>'
$programNames = @('cube.exe','STM32CubeMX.exe','STM32_Programmer_CLI.exe',
    'arm-none-eabi-gcc.exe','arm-none-eabi-gdb.exe','UV4.exe','IarBuild.exe','java.exe')
Get-ChildItem -LiteralPath $selectedRoot -Recurse -File -ErrorAction SilentlyContinue |
    Where-Object { $_.Name -in $programNames } |
    Select-Object -ExpandProperty FullName
```

若安装记录和候选目录均无结果，检查已有快捷方式的目标或用户指定目录。仍无结果时进入下文失败处理。

## Linux / POSIX shell

```bash
for tool in cube STM32CubeMX STM32_Programmer_CLI arm-none-eabi-gcc arm-none-eabi-gdb IarBuild; do
  command -v "$tool" || :
done
```

用 `type -a <工具名>` 查看 shell 支持的多个解析结果。检查工程声明、已有桌面启动文件的 `Exec=` 和包管理器安装记录；例如 Debian 系用 `dpkg-query -W`、RPM 系用 `rpm -qa` 后筛选 STM32/工具名，只查询当前系统存在的管理器。

候选位置为 `"$HOME/.local/bin"`、`"$HOME/.local/stm32cube/bin"`、`"$HOME/STMicroelectronics"`、`/opt/st`、`/opt/STMicroelectronics`、`/usr/local/bin`。先 `ls` 查看存在的目录，再在一个已选目录内搜索：

```bash
find "<已选安装目录>" -maxdepth 6 -type f \
  \( -name 'cube' -o -name 'STM32CubeMX' -o -name 'STM32CubeMX.exe' \
     -o -name 'STM32_Programmer_CLI' -o -name 'arm-none-eabi-gcc' \
     -o -name 'arm-none-eabi-gdb' -o -name 'IarBuild' \) -print
```

找不到时先查看该软件的实际子目录层级，再扩大这个安装目录内的深度。检查 `file "<程序>"`、`ls -l "<程序>"` 和启动脚本内容；依赖库错误按该安装的官方依赖处理。

## macOS / POSIX shell

先采用上面的 PATH 查询，再检查 `/Applications`、`"$HOME/Applications"`、`"$HOME/.local"`、`/usr/local/bin` 和 `/opt/homebrew/bin`。Homebrew 安装可通过 `brew list --versions` 和 `brew --prefix <已发现的软件包>` 定位。

找到 STM32 相关 `.app` 后：

```bash
app="<已发现的应用.app绝对路径>"
/usr/libexec/PlistBuddy -c 'Print :CFBundleExecutable' "$app/Contents/Info.plist"
find "$app/Contents" -type f \
  \( -name 'STM32CubeMX' -o -name 'STM32CubeMX.exe' -o -name '*.ini' \
     -o -name '*.jar' -o -name 'java' -o -name 'STM32_Programmer_CLI' \) -print
```

从 `Contents/MacOS` 启动器、启动配置和应用内资源确认命令行入口；`CFBundleExecutable` 指向启动器时继续读其配置。`open -a` 打开 GUI 不足以证明脚本操作可用。用 `uname -m` 和 `file "<程序>"` 核对架构，不自动安装转译环境或更改系统权限。

## 从 Bundle 定位编译器、调试器和下载工具

`cube --list` 列出可调用命令；对列表里实际存在的命令执行 `cube --json --resolve <命令>`。解析结果可能是一串“运行时 + 脚本 + 参数”，应通过 cube 调用，不能只取最后一项当成独立程序。

GCC 可能没有注册 cube 子命令。查询 `cube --json bundle list` 中的 `path`、`details.name`、`details.version`、`details.variables` 和 `details.commands`，仅筛选相关工具。PowerShell 示例：

```powershell
$installedBundles = cube --json bundle list | ConvertFrom-Json
$installedBundles |
    Where-Object { $_.details.name -match 'gnu|gcc|clang|gdb|programmer' } |
    ForEach-Object {
        [pscustomobject]@{
            Path = $_.path
            Name = $_.details.name
            Version = $_.details.version
            Variables = ($_.details.variables | ConvertTo-Json -Depth 8 -Compress)
        }
    }
```

确认工程约束对应的 Bundle 后，在其 `path` 内查找目标程序。`${CUBE_BUNDLE_ROOT}` 是元数据占位符，替换为该记录的 `path`；不是 shell 环境变量。`cube --tool-paths` 为空也不能证明编译器未安装，仍检查 Bundle 元数据。

直接调用已选 GCC 的 `--version`、`-dumpmachine`，目标应与工程架构匹配；从同一工具链 bin 找 `size`、`objcopy`、`objdump`。构建端采用这些路径的方法见[CMake 工具链接入](../../stm32-build/references/cmake.md)。GDB、Programmer 及 server 的只读能力验证分别使用已发现入口的 `--version`/`--help`，暂不连接目标。

独立 Keil 的 `UV4.exe`、IAR 的 `IarBuild` 从安装记录及选定软件目录查找。依据安装附带文档核对版本和命令行支持；不要给所有 GUI 程序机械添加 `--version`。构建目标查询见[已有构建体系](../../stm32-build/references/existing-builds.md)。

## SDK 与仓库

- 传统 CubeMX：搜索工程 `.ioc` 的 `ProjectManager.FirmwarePackage`、`ProjectManager.CustomerFirmwarePackage` 等实际字段，以及 include 路径和生成器设置。`"$HOME/STM32Cube/Repository"` 是候选，检查实际软件包目录及版本元数据；自定义位置以工程声明为准。
- MX2：用 `cube pack list packs --help` / `cube pack list packs` 查询 vendor、name、version 和路径。支持时用 `cube --get-current-value cmsis_pack_root` 查 Pack root，再检查 `.pdsc` 元数据。不要把整个 `.ioc2` 当成 JSON 解析。
- Bundle：支持时用 `cube --get-current-value cube_bundle_path` 查根目录。输出可能是 `key=value`，不要将整行作为文件路径，也不要把仓库根目录误认为某个工具的可执行入口。
- 实际头文件、启动文件和链接脚本必须来自当前工程使用的依赖。目录存在不代表设备/组件依赖完整；恢复方法见[工具与依赖](tools-and-dependencies.md)。

## 失败处理与定位完成条件

| 结果 | 下一步 |
| --- | --- |
| PATH 无结果，但找到安装目录 | 用绝对路径进行最小查询；需要时只向本次工具进程提供 PATH。 |
| 多个候选 | 按工程版本、生成器及架构筛选；仍无法区分时说明候选和影响，再请求选择。 |
| cube 有命令但解析失败 | 检查工程/父目录版本约束和 Bundle 完整性，不换用不相关最新版。 |
| 程序存在但启动失败 | 保存错误，检查启动配置、运行时、架构及依赖；传统 CubeMX 转[脚本入口](cubemx-runtime.md)。 |
| 所有有限位置均无结果 | 明确缺少哪个工具和已查位置，按[环境缺失与替代边界](tools-and-dependencies.md#环境缺失与替代边界)建议安装、恢复或采用等效工具；缺少自定义位置时询问位置，不反复全盘扫描。 |
| 主机不支持所需工具 | 保留已完成分析，说明需要的执行环境或任务允许的替代路径。 |

完成定位时应得到“工具种类、调用入口、版本、项目约束和最小查询结果”。之后返回原任务继续执行，不以列出安装目录替代构建、配置或下载。

依据：[CubeCLI 安装](https://dev.st.com/stm32cube-docs/stm32cubecli/latest/en/docs/markup/STM32CubeCLI_Getting_Started/STM32CubeCLI_Installation.html)、[CubeCLI 命令参考](https://dev.st.com/stm32cube-docs/stm32cubecli/latest/en/docs/markup/STM32CubeCLI_Command_Reference.html)、当前安装元数据和启动配置。
