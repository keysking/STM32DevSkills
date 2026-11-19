# 环境发现与 ST-Link 探测

仅在用户和[当前工程](identify.md)都不足以确定系列时使用。入口自行完成工具发现，无需先安装系列技能。ST-Link 结果只用于选择系列包，不用于擅自确定具体型号；系列已知而准确料号缺失时，转入工程或用户信息确认。此处不建立工程、不下载 SDK，也不准备整套编译环境。

## 找到 Cube

先查当前进程 PATH，再查 VS Code STM32CubeIDE Core 扩展和常见安装位置。用户提供的工具目录优先；候选路径不代表必然已安装。若有多个版本，沿用工程指定且能工作的版本。

Windows PowerShell：

```powershell
Get-Command cube -All -ErrorAction SilentlyContinue
$extensionRoots = @(
    (Join-Path $HOME '.vscode/extensions'),
    (Join-Path $HOME '.vscode-insiders/extensions')
)
foreach ($extensionRoot in $extensionRoots) {
    Get-ChildItem -LiteralPath $extensionRoot -Directory -Filter 'stmicroelectronics.stm32cube-ide-core-*' -ErrorAction SilentlyContinue |
        ForEach-Object {
            Get-ChildItem -LiteralPath (Join-Path $_.FullName 'resources/binaries') -Filter cube.exe -File -Recurse -ErrorAction SilentlyContinue
        }
}
Get-ChildItem -LiteralPath (Join-Path $HOME '.local/stm32cube/bin') -Filter 'cube*' -File -ErrorAction SilentlyContinue
```

Linux/macOS Bash：

```bash
command -v cube
for extension_root in "$HOME/.vscode/extensions" "$HOME/.vscode-insiders/extensions"; do
  for core_dir in "$extension_root"/stmicroelectronics.stm32cube-ide-core-*; do
    if [ -d "$core_dir/resources/binaries" ]; then
      find "$core_dir/resources/binaries" -type f -name cube
    fi
  done
done
for cube_candidate in "$HOME/.local/stm32cube/bin"/cube*; do
  if [ -f "$cube_candidate" ]; then printf '%s\n' "$cube_candidate"; fi
done
```

自定义 VS Code 扩展目录、远程开发环境按实际配置查询。找到程序后用已确认的绝对路径调用，不为探测修改系统 PATH。下面路径是变量示意，替换后运行：

```powershell
$cubeExe = 'C:/actual/path/cube.exe'
& $cubeExe --version
& $cubeExe --list
& $cubeExe --json --resolve programmer
& $cubeExe programmer --help
```

```bash
cube_exe='/actual/path/cube'
"$cube_exe" --version
"$cube_exe" --list
"$cube_exe" --json --resolve programmer
"$cube_exe" programmer --help
```

检查每条命令的退出码与输出。`--resolve programmer` 的 `command` 给出实际命令链，`bundle` 表明选中版本，`exported` 是其运行环境；优先通过 Cube 转发，保留所需环境。帮助成功只说明工具可启动，不说明已连接芯片。

## Cube 不可用时找原生 Programmer

Cube 缺失或没有 `programmer` 入口时，可直接使用已安装的 `STM32_Programmer_CLI`。先查 PATH，再查用户提供路径、系统安装记录及下列有限候选根目录；不要递归搜索整个系统盘。

| 平台 | 常见位置，按实际安装核对 |
| --- | --- |
| Windows | `%ProgramFiles%/STMicroelectronics/STM32Cube/STM32CubeProgrammer/bin`；`%LOCALAPPDATA%/stm32cube/bundles/programmer`；已安装 CubeCLT / CubeIDE 的 Programmer 目录。 |
| Linux | `$HOME/STMicroelectronics/STM32Cube/STM32CubeProgrammer/bin`；`$HOME/.local/share/stm32cube/bundles/programmer`；已知的 `/opt` 下 ST 安装目录。 |
| macOS | `/Applications/STMicroelectronics/STM32Cube/STM32CubeProgrammer` 安装树；用户安装目录或已知的 Cube Bundle 根目录。 |

Windows 用 `Get-Command STM32_Programmer_CLI -All`，Linux/macOS 用 `command -v STM32_Programmer_CLI`。在选定候选根目录内按文件名查找，运行实际程序的 `--help`。后文 `cube programmer` 可以替换为此程序的完整路径；不能把它套在另一个 `programmer` 子命令外。

两者都找不到或无法启动时，说明缺失的识别工具并询问芯片型号。若用户提供型号即可继续安装系列包，无需为路由安装完整开发环境。

## 枚举与读取目标

按本机叶子帮助核对 `-l`、`sn` 和 `mode` 参数。执行命令时设置有限超时，例如 30 秒；超时结束本次尝试，保留输出，不循环重连。

先只枚举探针，不连接目标：

```text
cube programmer -l st-link-only
```

无探针时询问型号或连接条件。多个探针时使用用户指定的序列号；无法确定时列出候选供选择，不按枚举顺序取第一个。探针名称、序列号或 `Board` 字段不是当前 MCU 的可靠型号依据。

仅有一个明确目标或用户已选定探针时，将下面序列号替换为枚举结果，通过 SWD 热连接读取器件信息：

```text
cube programmer -c port=SWD sn=<实际序列号> mode=HOTPLUG
```

`HOTPLUG` 用于避免正常连接模式的停机/复位行为，仍需尊重用户对硬件连接的限制。此次识别不追加写入、擦除、复位、解锁或选项字节操作，也不自动改为 `UR` / `NORMAL` 来恢复连接。连接失败、探针占用、保护阻止读取或超时，报告原因线索并询问型号。

## 解读结果

读取连接日志的 `Device name`、`Device ID`、Flash 信息及错误信息。退出成功且返回明确系列时，记录原始候选组并回到[入口](../SKILL.md)选择包；准确型号是否已知单独记录。

- `STM32F103…` 或 `STM32F1…` 可以选择 F1；`STM32C532…` 或 `STM32C5…` 可以选择 C5。入口仅用这些字段确认系列，不据此声明准确型号。
- 工具返回多个候选但都属于同一系列，可选择该系列包；未经用户或工程信息确认的具体型号仍标为未确定。不能取第一个候选，也不能结合 Flash 容量、修订号或 CPU 类型擅自补齐型号、封装和容量后缀。
- 仅有未知的数字 Device ID 时，查当前 Programmer 随附的器件数据库或 ST 官方资料确认映射；不能凭记忆填入型号。无法唯一确定系列时询问用户。
- 只有探针枚举成功、CPU 内核名或电压值，不能宣称已识别 MCU。

下面是 STM32CubeProgrammer 2.23.0 经 ST-Link 热连接返回的两组实际字段，说明同一个器件 ID 可对应多个具体型号；不将这两项作为全部器件的识别表：

| Device ID | Device name | NVM size | 可选系列包 | 仍不能确定 |
| --- | --- | --- | --- | --- |
| `0x410` | `STM32F101/F102/F103 Medium-density` | `64 KBytes` | F1 | 不能唯一确定 F103，更不能确定完整料号。 |
| `0x44F` | `STM32C53x/542` | `256 KBytes` | C5 | 不能唯一确定 C531、C532 或 C542，更不能确定完整料号。 |

达到选包所需的系列精度后即可安装并读取对应技能。**新建工程必须在创建目标相关配置或生成工程前确认准确芯片型号**；其他任务在后续确实需要时确认。用户未提供、工程中也没有准确料号时，按[型号确认规则](identify.md)询问用户并等待，不继续探测猜测具体型号。

命令依据：[STM32CubeCLI](https://dev.st.com/stm32cube-docs/stm32cubecli/latest/en/docs/markup/STM32CubeCLI_Command_Reference.html)、[STM32CubeProgrammer CLI](https://dev.st.com/stm32cube-docs/prog/2.23.0/en/docs/markup/CubeProg_Command_Lines.html)。实际可用参数以本机帮助为准。
