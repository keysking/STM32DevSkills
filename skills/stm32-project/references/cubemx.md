# 传统 CubeMX 工程路径

适用于 `.ioc` 和传统 STM32CubeMX。先发现安装、运行版本、MCU 数据库和固件仓库；`cube mx` 通常是 MX2 入口，不能据名称将 `.ioc` 交给它。

## 创建基础工程

使用独立 CubeMX 的交互或脚本模式。脚本一行一条命令，参数以当前会话帮助为准。以下是替换目标和路径后的流程模板：

```text
load <MCU数据库标识>
project name <工程名>
project path <新工程目录>
project toolchain CMake
project compiler GCC
config save <配置绝对路径.ioc>
project generate
exit
```

准确 CPN 与数据库加载标识可能不同，加载后回读实际选中的设备与封装。固件仓库不在默认位置时查询 `project setCustomFWPath` 的帮助并指定匹配 SDK。不要把某个型号的固件路径写成通用默认值。

Windows 可通过合适的 Java 执行 `-jar <CubeMX.exe> -q <脚本>`；Linux/macOS 使用安装提供的入口及 `-q`。JVM 参数按对应安装配置处理。可用的 `cube java` 也可以作为运行时。

## 已有工程与重新生成

使用 `config load <配置.ioc>` 加载。发现实际可用命令后修改配置，回读、`config save`，再重新加载检查。硬件配置细节由当前上下文帮助发现，不能只使用启动时的帮助列表。

生成前检查目标输出目录、文件拷贝模式、用户代码保留设置和已存在的源码。保存 `.ioc` 与生成 C 文件是不同步骤；两者都应检查。比较生成差异，确认用户区及独立用户文件仍被保留。

## 构建交接

查找生成的 CMakeLists、presets 和工具链文件。使用它们实际声明的配置，不猜测工具链文件名。检查 GCC/Ninja 是否可用并构建相应目标。其他工具链工程保持已有选择。

过程日志可能含失败提示而进程仍返回 0；检查配置回读、生成诊断、文件及构建结果。生成器不能表达的修正应说明维护位置和下次生成的处理方式。

CMake 导出出现重复绝对路径或缺失 syscalls/sysmem 等文件时，核对 `.ioc` 的 `ProjectManager.MainLocation`、实际源码目录及模板生成路径。在存在该问题的版本中，使用符合当前 CMake 模板的相对位置（例如 `Core/Src`）作为最小配置补充，重新加载、生成并构建验证。不要把这个示例路径强加到已有正常工程。工程目录和配置文件使用绝对路径可避免某些版本的相对路径保存错误；生成器仍可能在指定父目录下创建工程名子目录，应从实际文件确认。

依据：[CubeMX CLI](https://dev.st.com/stm32cube-docs/stm32cubemx/6.18.1/en/docs/markup/CubeMX_CLI.html)、目标版本运行时帮助。
