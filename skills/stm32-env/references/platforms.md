# 平台差异

同一开发流程共用，工具位置和设备访问按当前主机处理。先确认 Windows/Linux/macOS 以及 x86_64/Arm，安装对应架构的工具；不要把其他主机的 Bundle 路径复制过来。

| 项目 | Windows | Linux | macOS |
| --- | --- | --- | --- |
| 查找工具 | `Get-Command cube`；必要时 `where.exe cube`。 | `command -v cube`。 | `command -v cube`。 |
| 路径与调用 | 路径加引号；调用字符串中的程序使用 `&`。 | 路径加引号，检查执行权限。 | 检查 `.app` 内实际启动程序、架构和系统允许的执行权限。 |
| CubeMX | 发现独立安装目录、可用 Java 和启动参数。 | 发现安装程序及依赖库，使用安装提供的入口。 | 发现应用包内入口，不假定目录大小写或可执行文件位置。 |
| USB/串口 | 检查设备管理器、ST-Link 驱动及 COM 号。 | 检查 USB、udev 规则和设备组权限；使用系统规定的串口设备。 | 检查 USB 可见性、工具支持及 `/dev/cu.*` 等实际设备。 |

## 命令传参

文档中的 `text` 命令块表示参数模板，替换尖括号字段后执行。含空格的路径作为单独参数；不要用字符串拼接后交给另一个 shell 解释。

PowerShell 传多个 Java/JVM 参数时采用数组，避免 `-D` 或 `--add-opens` 参数被拆开：

```powershell
$mxArgs = @('java', '--add-opens=java.desktop/java.awt=ALL-UNNAMED', '--add-exports=java.desktop/sun.awt=ALL-UNNAMED', '-Djavax.net.ssl.trustStoreType=WINDOWS-ROOT', '-jar', '<CubeMX.exe绝对路径>', '-q', '<脚本绝对路径>')
& cube @mxArgs
```

这些 JVM 选项取决于 CubeMX/Java 版本；先检查安装中的启动配置或官方启动说明。若当前 `cube java` 不满足要求，使用 CubeMX 配套的 Java。Linux/macOS 优先使用安装提供的 CubeMX 启动入口及 `-q` 脚本模式。

## 路径与环境恢复

- 用 `cube --get-current-value` 的当前帮助、解析结果和工程元数据发现 Bundle/Pack root；不要硬编码某个用户名或系统盘。
- 分清 shell 环境与工具子进程导出的环境。工具链路径变化后重新运行版本检查及必要的 CMake 配置。
- USB 访问失败先辨认驱动、权限、接线或进程占用。管理员/root 权限不是连接失败的通用解决方案。
- 文档覆盖平台不等于本次任务已在全部平台运行。只陈述实际观察结果。

依据：[CubeCLI 安装文档入口](https://dev.st.com/stm32cube-docs/stm32cubecli/latest/en/index.html)、[CubeMX 命令行模式](https://dev.st.com/stm32cube-docs/stm32cubemx/6.18.1/en/docs/markup/CubeMX_CLI.html)。
