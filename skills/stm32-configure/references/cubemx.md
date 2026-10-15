# 传统 CubeMX 配置

## 建立正确会话

工具路径或会话尚未确认时读取[传统 CubeMX 脚本入口](../../stm32-env/references/cubemx-runtime.md)，依赖定位读取[工具与 SDK](../../stm32-env/references/tool-location.md)。以脚本模式加载 `.ioc`；加载并完成模块初始化后再查询 `help`、对应命令组和参数可用值。DMA、NVIC、时钟节点可能随上下文出现。

先执行只读发现脚本，再准备修改脚本。输入命令组本身（例如 `get`、`set`、`clock`）查询其当前帮助；继续使用帮助返回的子命令语法。启动时的 `help` 只列出命令组，不代表已给出 MCU 的参数和值。

有些版本在输出命令组的 `Usage` 后返回 `KO`，表示缺少子命令。这种探查用于取得语法，不能算配置查询成功；下一步使用帮助中的完整 get 命令取得实际值。执行完整修改命令后出现 `KO` 则按失败处理。

## 引脚、模式和参数

以下展示查询与设置关系，尖括号值必须来自当前 MCU/工程：

```text
config load <配置.ioc>
set mode <外设实例> <模式>
set pin <引脚> <信号>
get signal <引脚>
get mode active <外设实例>
get ip parameters <实例> <参数名>
possible_value ip parameters <实例> <参数名> <候选值>
set ip parameters <实例> <参数名> <合法值>
get ip parameters <实例> <参数名>
```

GPIO 标签等由 `set/get gpio parameters` 处理。不要假定参数查询支持正则或通配符；某条查询返回 `OK` 但没有值时继续寻找有效接口。

先从当前 `get` 帮助取得可查询的引脚/信号、IP 实例和模式列表接口，再从相应实例帮助取得参数名。`possible_value` 是候选合法性查询，不能凭名称假定它能枚举全部候选。需要的值应由当前数据库、帮助或对应实例参数描述取得。设置后使用同一对象的 get 回读，确认是目标值而非只有状态字符串。

例如当前帮助提供这些接口时，可用 `get mcu name`、`get mcu package`、`get mcu peripherals` 确认目标，`get pinout` 查询封装引脚，`get functions <引脚表达式>` 查询复用信号，`get mode available <实例>` 查询可选模式，再用 `get mode_param_list <实例> <已选模式>` 及 `get mode_param_possvalue <实例> <模式> <参数>` 取得模式参数及取值。命令中明确允许的表达式和仅接受准确参数名的查询应分别处理。

## DMA 与中断

查询当前 `dma` 与 `NVIC` 帮助。先发现外设请求和可分配资源，再创建 DMA 请求并配置方向、模式、宽度、地址递增和优先级；检查实际分配的控制器/通道/流及句柄关联。

常见会话形式为 `dma available_periph_requests <实例>`、`dma add <请求>`、`dma set_param <请求> <参数> <值>` 和对应 `get_param`。请求可添加不等于通道分配、NVIC 和 ISR 链路已经完整。

NVIC 按真实 IRQ 名称启用并设置优先级，使用 `get_enable_state`、`get_priority` 等回读。使用 FreeRTOS 的 ISR 还必须满足内核端口的系统调用优先级约束。

## 时钟模型

查询 `clock` 命令组及当前节点、候选值和索引含义。算术节点与选择器可能分别使用倍率/分频值和输入索引，不能跨芯片复制数值。

使用 `clock set <节点> <值>` 等实际时钟树接口，修改后回读计算频率和时钟错误状态。单独 `set ip parameters RCC …` 的参数回读变化不能证明时钟树已经重算。核对系统、总线、外设、定时器和 USB 等受影响的频率。

计算频率可从当前 RCC 参数接口回读，例如数据库存在 `SYSCLKFreq_VALUE` 和 `APB1Freq_Value` 时使用 `get ip parameters RCC SYSCLKFreq_VALUE` 和 `get ip parameters RCC APB1Freq_Value`。字段名及大小写以当前数据库为准；结合时钟树检查和生成诊断确认相关错误，不把配置频率当成板端实测频率。

## 保存、重载和生成

回读后 `config save <配置.ioc>`，在重新加载会话中再次检查。随后 `project generate`，审查初始化代码、用户区和构建。`KO`、unsupported、空结果和插件错误均需解释；退出码 0 或孤立的 `OK` 不足以确认成功。

生成前检查实际输出目录、文件拷贝模式和用户代码保留设置，保留已有修改；生成后按[配置检查](validation.md)对照模型与源码，再按[构建](../../stm32-build/SKILL.md)验证。工程及配置使用绝对路径；生成器可能在指定父目录下创建工程名子目录，应从实际文件确认输出。

| 现象 | 下一步 |
| --- | --- |
| 查询没有值 / 命令不存在 | 确认工程已加载及模块已初始化，读取当前命令组帮助；不继续设置未知参数。 |
| 保存重载后恢复旧值 | 核对保存文件、实际加载路径、求解约束及并行 GUI 会话。 |
| 生成失败 / 缺 SDK | 检查生成日志及固件仓库版本，按[依赖](../../stm32-env/references/tools-and-dependencies.md)补齐所需组件。 |
| 模型正确但源码没变 | 检查输出目录、生成选项及本次真正生成的文件，按配置检查追踪到实际构建输入。 |
| CMake 路径重复或缺 syscalls/sysmem | 核对 `ProjectManager.MainLocation`、实际源码目录及生成模板；仅在该问题存在时采用符合模板的相对位置，例如 `Core/Src`，重载生成后构建。 |

必要时直接编辑 `.ioc` 只能作为针对已知字段的补充：最小修改，保留版本和其他内容，交给目标版本 CubeMX 加载、检查并生成验证。

依据：[CubeMX CLI](https://dev.st.com/stm32cube-docs/stm32cubemx/6.18.1/en/docs/markup/CubeMX_CLI.html)、目标会话运行时帮助。
