---
name: stm32-env
license: MIT
description: 准备或修复 STM32 开发环境，发现 cube 与原生工具、恢复项目 Bundle 和 Pack 依赖、定位编译器及版本冲突。用于缺少工具、SDK 或探针驱动的任务。
---

# STM32 环境与依赖

## 执行流程

1. 确认操作系统、主机架构、工作目录和当前任务。检查已有工程的 `.settings`、SDK、生成器及构建配置，保留其工具版本约束。
2. 工具路径未知或无法调用时，按[工具定位](references/tool-location.md)依次检查工程声明、PATH、安装信息和限定目录；传统 CubeMX 继续读取[脚本入口](references/cubemx-runtime.md)。先验证入口，再进行工程操作。
3. 按[工具与依赖](references/tools-and-dependencies.md)发现当前命令、Bundle 和 Pack，补齐任务必需依赖；按[平台差异](references/platforms.md)处理参数、驱动及权限。
4. 对选定的工具执行版本或帮助查询；编译器额外确认实际可执行文件及目标架构。环境变量变化后重新查询解析结果。

优先通过 `cube` 使用工具，已有工程依赖的独立 CubeMX、Keil、IAR 或其他原生工具也可沿用。依据当前帮助发现命令，不能根据其他机器的列表推断本机能力。

## 完成条件

- 能说明当前任务使用的工具路径、版本、必要 SDK/Pack 和项目版本约束。
- 工具确实可调用，依赖错误已解决或明确定位到缺少的组件、网络访问或凭据。
- 不以环境检查代替工程构建；任务需要构建时继续完成相应验证。

只准备用户任务需要的环境，不主动升级工程工具链或下载所有软件包。输出结论及必要后续操作，不回显完整环境变量或凭据。

## 与其他技能协作

- 工程目标或版本约束不明时读取[工程识别](../stm32-project/references/identify-and-maintain.md)，复用已确认信息。
- 工具可用后返回原任务；需要配置、开发或构建时分别继续[配置](../stm32-configure/SKILL.md)、[固件](../stm32-firmware/SKILL.md)或[构建](../stm32-build/SKILL.md)，不在环境准备阶段启动无关操作。
