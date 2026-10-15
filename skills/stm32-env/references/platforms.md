# 平台差异

同一开发流程共用，工具位置和设备访问按当前主机处理。先确认 Windows/Linux/macOS 以及 x86_64/Arm，安装对应架构的工具；不要把其他主机的 Bundle 路径复制过来。

| 项目 | Windows | Linux | macOS |
| --- | --- | --- | --- |
| 查找工具 | 按[工具定位](tool-location.md)查 PATH、安装记录及软件目录。 | 同页查 PATH、启动文件和限定安装目录。 | 同页查 PATH、应用包和实际启动程序。 |
| 路径与调用 | 路径加引号；调用字符串中的程序使用 `&`。 | 路径加引号，检查执行权限。 | 检查 `.app` 内实际启动程序、架构和系统允许的执行权限。 |
| CubeMX | 按[脚本入口](cubemx-runtime.md)选择 Java 和启动参数。 | 同页使用安装提供的入口。 | 同页确认应用包内命令行入口。 |
| USB/串口 | 检查设备管理器、ST-Link 驱动及 COM 号。 | 检查 USB、udev 规则和设备组权限；使用系统规定的串口设备。 | 检查 USB 可见性、工具支持及 `/dev/cu.*` 等实际设备。 |

## 命令传参

文档中的 `text` 命令块表示参数模板，替换尖括号字段后执行。含空格的路径作为单独参数；不要用字符串拼接后交给另一个 shell 解释。

PowerShell 调用绝对路径时使用 `&`，多个参数采用数组：

```powershell
$toolProgram = '<已确认的程序绝对路径>'
$toolArgs = @('<参数1>', '<参数2>')
& $toolProgram @toolArgs
```

POSIX shell 使用 `"$tool_program" "$arg1" "$arg2"`。不要用 `eval` 重新解释参数。组件 ID、路径或参数值含 `$` 时使用合适的单引号或变量，避免 shell 提前展开；JSON/数组参数按 leaf 帮助规定编码。

## 路径与环境恢复

- Bundle/Pack root 和程序查找采用[工具定位](tool-location.md)，不硬编码用户名或系统盘。
- 分清 shell 环境与工具子进程导出的环境。工具链路径变化后重新运行版本检查及必要的 CMake 配置。
- USB 访问失败先辨认驱动、权限、接线或进程占用。管理员/root 权限不是连接失败的通用解决方案。
- 文档覆盖平台不等于本次任务已在全部平台运行。只陈述实际观察结果。

依据：[CubeCLI 安装文档入口](https://dev.st.com/stm32cube-docs/stm32cubecli/latest/en/index.html)、[CubeMX 命令行模式](https://dev.st.com/stm32cube-docs/stm32cubemx/6.18.1/en/docs/markup/CubeMX_CLI.html)。
