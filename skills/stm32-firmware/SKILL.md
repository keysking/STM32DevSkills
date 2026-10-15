---
name: stm32-firmware
license: MIT
description: 实现 STM32 应用功能、移植外设驱动或编写 FreeRTOS 任务，按实际 HAL1/HAL2/LL 和工程结构开发并验证代码。用于功能开发、驱动集成与固件维护。
---

# STM32 固件开发

1. 从当前工程确认目标、SDK、驱动头文件、初始化顺序、构建方式，以及裸机或 FreeRTOS 的实际 API。先阅读相关现有调用，再设计最小修改。
2. 按[驱动与代码维护](references/drivers-and-ownership.md)选择代码位置和接口，保留生成代码与用户代码边界。外设配置缺失时同时完成相应配置。
3. 裸机工程读取[裸机与异步外设](references/bare-metal.md)。FreeRTOS 工程读取[FreeRTOS API 与集成](references/freertos.md)，沿用原生 API、CMSIS-RTOS v1 或 v2。
4. 实现功能，处理返回值、超时、数据所有权、ISR 与主循环/任务交接。阻塞、IT、DMA 的选择依据实时性、缓冲区和工程已有接口。
5. 把新增源码加入真实构建目标，完成相关构建。按行为需要进行测试或板端验证，说明可观察的完成标准。

优先使用 SDK 已有驱动、BSP 和匹配目标的官方示例。代码中的类型、函数及宏以当前头文件为准，不从其他 HAL 代际拼接接口。不要为小型外设需求额外引入通用框架。

## 完成条件

代码接入实际初始化及调用链，构建产物包含修改；生成后用户代码仍可维护；异常路径和数据生命周期合理；清楚区分编译结果和实际功能验证。

第一版 RTOS 范围仅为 FreeRTOS。遇到其他 RTOS 时明确该部分范围，保留原工程，不主动转换内核。

## 与其他技能协作

- 不清楚工程目标或 API 层时读取[工程识别](../stm32-project/references/identify-and-maintain.md)；缺少工具或 SDK 时读取[环境](../stm32-env/SKILL.md)。
- 功能涉及引脚、时钟、DMA、中断或生成器选项时采用[配置](../stm32-configure/SKILL.md)，保持应用实现与生成设置一致。
- 修改后采用[构建](../stm32-build/SKILL.md)。任务包含板端验证时采用[烧录](../stm32-flash/SKILL.md)，出现运行异常时采用[调试](../stm32-debug/SKILL.md)；不因读取技能就扩大任务范围。
