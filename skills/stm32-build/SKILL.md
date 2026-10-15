---
name: stm32-build
license: MIT
description: 构建 STM32 固件、排查编译链接错误并检查 ELF、映射文件和资源占用。用于 CMake/Ninja/GCC 及已有 Makefile、Keil、IAR 工程的构建与产物检查。
---

# STM32 构建与检查

1. 识别真实构建入口、目标、编译器、工具链文件和 Debug/Release 配置，保留工程已指定的体系。不要按 `.ioc` 或 `.ioc2` 推断编译器。
2. CMake 工程读取[CMake 与工具链](references/cmake.md)；其他体系读取[已有构建体系](references/existing-builds.md)。确认 `cube` 解析的工具与工程约束一致。
3. 运行相应配置及构建命令，保留完整诊断供定位。错误处理和产物检查见[诊断与产物](references/diagnostics.md)。从第一处有意义的错误分析，避免用批量清理或关闭警告掩盖问题。
4. 检查本次生成的 ELF、BIN/HEX（若任务需要）、链接布局和资源占用，确认产物属于正确目标及配置。

新建且未指定体系的工程默认 CMake + Ninja + GCC。GCC Bundle 可能只导出编译器路径，并未注册 `cube arm-none-eabi-gcc` 命令；查询实际工具链，不猜测 wrapper 子命令。

## 完成条件

所需目标构建成功，产物与配置匹配；相关链接或资源问题已解释；无法构建时准确指出缺少的工具、组件或输入，不编造成功结果。

构建成功不代表已下载或功能正常。用户只要求构建时，以构建及相关产物检查为完成范围。

工程入口或目标不明确时读取[工程识别](../stm32-project/references/identify-and-maintain.md)；工具找不到、版本冲突或依赖缺失时读取[环境](../stm32-env/SKILL.md)。编译错误若来自配置或应用实现，分别读取[配置](../stm32-configure/SKILL.md)或[固件](../stm32-firmware/SKILL.md)的相关部分，修正后返回本流程构建，不循环重复环境检查。
