---
name: stm32-debug
license: MIT
description: 诊断 STM32 启动失败、HardFault、中断或 DMA/外设异常，以及 FreeRTOS 任务卡死等运行问题，使用 GDB、ST-Link 和板端证据定位并验证修复。
---

# STM32 调试与诊断

1. 明确预期、实际现象、复现条件和最近修改。检查目标与实际运行固件是否一致，区分环境、构建、下载和运行问题。
2. 按[GDB 与证据获取](references/gdb.md)连接目标，使用与固件匹配的 ELF。复位、暂停和断点会改变运行状态，选择符合当前问题的观察方式。
3. 依据证据读取相关资料：
   - [启动与异常](references/startup-and-faults.md)：复位、初始化失败、HardFault、内存和栈。
   - [外设、中断与 DMA](references/peripherals.md)：时钟、引脚、总线、DMA、中断和通信。
   - [FreeRTOS 诊断](references/freertos.md)：任务状态、栈、堆、同步、优先级及 ISR API。
4. 形成可验证的原因假设，只修改与原因相关的配置或代码；随后构建、按任务重新下载并使用同一复现条件复查。

先采集必要证据，避免第一步重置或重新生成工程而丢失现场。编译错误主要按构建流程解决；运行故障的定位可以返回配置或固件修改流程。

## 完成条件

说明观察证据、原因及其确定程度、修改和复查结果。区分已证明的原因与待验证假设；无硬件时完成源码、配置和产物分析，明确需要补充的运行证据。

不把一个芯片的故障寄存器、时钟结构或某个历史项目参数作为所有 STM32 的通用事实。

## 与其他技能协作

- 工程目标不明时读取[工程识别](../stm32-project/references/identify-and-maintain.md)；工具缺失时读取[工具定位](../stm32-env/references/tool-location.md)。
- 证据指向配置或应用问题时，按需读取[配置](../stm32-configure/SKILL.md)或[固件](../stm32-firmware/SKILL.md)，只修改与原因相关的内容。
- 修复后按[构建](../stm32-build/SKILL.md)取得新产物；需要写入目标时按[烧录](../stm32-flash/SKILL.md)执行。复用已确认的工具、目标和连接参数，再回到本流程按原条件复查。
