---
name: stm32-configure
license: MIT
description: 配置 STM32 的引脚、时钟、外设、DMA、中断及生成器中的 FreeRTOS 选项，完成查询、修改、保存和重新生成。用于 CubeMX/CubeMX2 或手写初始化工程的硬件配置调整。
---

# STM32 硬件与生成器配置

1. 明确目标外设行为、板卡接线和允许修改的范围。核对芯片封装、真实时钟源及工程维护方式，不根据开发板示例假定用户硬件。
2. 选择[传统 CubeMX](references/cubemx.md)、[CubeMX2](references/cubemx2.md)或现有手写初始化路径。发现当前目标的有效引脚、资源、模式、参数和约束，再执行设置。
3. 按[配置检查](references/validation.md)核对时钟、引脚、DMA、中断和相关软件组件。生成器中的 FreeRTOS 启用与配置按目标 SDK 的实际组件处理。
4. 回读设置并检查错误状态，保存、重新加载，再次确认。生成器能表达的配置优先写入配置模型；手写工程按现有方式更新初始化。
5. 重新生成后检查差异、用户代码保留和构建结果。不要把生成命令的退出码当成全部成功依据。

详细参数和实例名依据当前工程查询。不要跨系列套用时钟节点、复用功能、DMA 通道或 NVIC 数值。`.ioc2` 是复合工程文件，不能按纯 JSON 直接改写。

## 完成条件

- 配置的实际值与需求一致，保存重载后仍有效。
- 生成代码反映配置，构建通过；生成器限制及必要手写补充有清晰维护位置。
- 缺少板卡条件时说明待验证的接线、时钟源或信号，继续完成可进行的配置与构建。

任务逻辑、驱动封装和 FreeRTOS 任务设计属于固件开发；本技能负责它们所需的硬件与生成器设置。

## 与其他技能协作

- 目标、SDK 或生成器未识别时读取[工程识别](../stm32-project/references/identify-and-maintain.md)；只有确需创建基础工程时进入[工程管理](../stm32-project/SKILL.md)。
- 无法定位或启动生成器时读取[工具定位](../stm32-env/references/tool-location.md)，传统 CubeMX 继续读取[脚本入口](../stm32-env/references/cubemx-runtime.md)。
- 配置需配套应用代码时继续[固件](../stm32-firmware/SKILL.md)；生成后按[构建](../stm32-build/SKILL.md)验证，不在本技能重复维护构建方法。
