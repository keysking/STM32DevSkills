# 传统 CubeMX 工程路径

适用于 `.ioc` 和传统 STM32CubeMX。还未建立脚本会话时先读取[传统 CubeMX 入口](../../stm32-env/references/cubemx-runtime.md)，SDK 定位读取[工具与 SDK 定位](../../stm32-env/references/tool-location.md)。

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

## 已有工程与重新生成

沿用已存在的工程，不重复执行创建模板。加载、参数发现、保存重载、用户代码保留和重新生成统一使用[传统 CubeMX 配置](../../stm32-configure/references/cubemx.md)。

## 构建交接

首次生成后查找实际源码、CMakeLists/presets 和工具链文件，按[构建技能](../../stm32-build/SKILL.md)验证；其他工具链工程保持已有选择。创建完成需要配置文件、目标和源码确实存在，日志没有未解决的生成错误，任务要求的构建有对应产物。

依据：[CubeMX CLI](https://dev.st.com/stm32cube-docs/stm32cubemx/6.18.1/en/docs/markup/CubeMX_CLI.html)、目标版本运行时帮助。
