# 传统 CubeMX 配置

## 建立正确会话

发现可运行的 CubeMX、Java、数据库和固件仓库。以脚本模式加载 `.ioc`，或加载准确 MCU 数据库标识。加载并完成模块初始化后再查询 `help`、对应命令组和参数可用值；DMA、NVIC、时钟节点可能随上下文出现。

通过 `-q <脚本>` 执行一行一命令的脚本；Windows 使用配套 Java，或已确认兼容的 `cube java`；Linux/macOS 使用安装提供的入口。脚本路径及配置路径明确传参，JVM 参数依安装启动配置确定。

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

## DMA 与中断

查询当前 `dma` 与 `NVIC` 帮助。先发现外设请求和可分配资源，再创建 DMA 请求并配置方向、模式、宽度、地址递增和优先级；检查实际分配的控制器/通道/流及句柄关联。

常见会话形式为 `dma available_periph_requests <实例>`、`dma add <请求>`、`dma set_param <请求> <参数> <值>` 和对应 `get_param`。请求可添加不等于通道分配、NVIC 和 ISR 链路已经完整。

NVIC 按真实 IRQ 名称启用并设置优先级，使用 `get_enable_state`、`get_priority` 等回读。使用 FreeRTOS 的 ISR 还必须满足内核端口的系统调用优先级约束。

## 时钟模型

查询 `clock` 命令组及当前节点、候选值和索引含义。算术节点与选择器可能分别使用倍率/分频值和输入索引，不能跨芯片复制数值。

使用 `clock set <节点> <值>` 等实际时钟树接口，修改后回读计算频率和时钟错误状态。单独 `set ip parameters RCC …` 的参数回读变化不能证明时钟树已经重算。核对系统、总线、外设、定时器和 USB 等受影响的频率。

## 保存、重载和生成

回读后 `config save <配置.ioc>`，在重新加载会话中再次检查。随后 `project generate`，审查初始化代码、用户区和构建。`KO`、unsupported、空结果和插件错误均需解释；退出码 0 或孤立的 `OK` 不足以确认成功。

必要时直接编辑 `.ioc` 只能作为针对已知字段的补充：最小修改，保留版本和其他内容，交给目标版本 CubeMX 加载、检查并生成验证。

依据：[CubeMX CLI](https://dev.st.com/stm32cube-docs/stm32cubemx/6.18.1/en/docs/markup/CubeMX_CLI.html)、目标会话运行时帮助。
