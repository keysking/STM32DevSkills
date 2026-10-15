# 工程识别、创建与维护

## 独立判断工程的几个维度

| 维度 | 检查线索 |
| --- | --- |
| 目标 | 编译器 MCU 宏、启动文件、设备头文件、链接脚本、CPN/板卡元数据。 |
| 生成器 | `.ioc`、`.ioc2`、生成目录和注释；可能存在多个软件工程或历史配置。 |
| 驱动 | 实际 include 路径、HAL/LL 头文件、SDK 版本、组件依赖。 |
| 构建 | CMakeLists、presets、Makefile、`.uvprojx`、`.ewp`、工作区和构建说明。 |
| 运行模型 | 调度器启动、FreeRTOSConfig、`cmsis_os.h`/`cmsis_os2.h`、任务创建和中断实现。 |

使用 `rg --files` 和定向文本搜索，先读入口、初始化及构建目标。工具链、生成器和 HAL 代际不是一一对应关系。配置、源码与构建声明矛盾时查清真实目标，不任意选取一份。

## 无生成器工程

沿用现有 startup、SystemInit、向量表、链接布局和外设初始化。已有工程无需为了使用本技能而迁移到 CubeMX。检查新增源码的构建归属，以及 Debug/Release、安全域或多核目标是否各自需要修改。

确实需要空 CMake 工程时，先查询当前 creator：

```text
cube ide-project-create --help
cube ide-project-create --destination "<新目录>" --name "<工程名>" --target-device "<准确CPN>" --toolchain GCC
```

必要时使用查询到的 `--processor-id`、`--security-model` 或 `--target-board`。空工程不等于已完成外设初始化，也不自动证明所需 SDK 组件齐全。检查产物和实际编译入口，再补齐功能。

## 已有工程维护

先查看工作区状态和构建基线，保留无关改动。修改生成配置前确定用户代码保留方式；没有版本控制时可对将受影响的文件建立临时副本。用户要求移植时同时核对引脚、时钟、供电、外设实例、DMA、IRQ 和内存条件，不能只替换句柄名。

生成或创建后检查文件是否真实存在，目标宏、启动文件和链接容量是否匹配。依赖问题与应用代码问题分别处理。默认使用 CMake + Ninja + GCC 只适用于未指定体系的新工程。

依据：[CubeCLI 首个工程](https://dev.st.com/stm32cube-docs/stm32cubecli/latest/en/docs/markup/STM32CubeCLI_Getting_Started/STM32CubeCLI_First_Project.html)、当前 `ide-project-create --help`。
