# 连接方式、主机平台与运行观察

| 方式 | 应确认的条件 |
| --- | --- |
| SWD/JTAG | 目标供电与电平参考、共地、信号线、复位线（如需要）、探针序列号及目标 AP/内核。 |
| UART bootloader | 当前芯片支持的引脚/USART、启动条件、电平、端口、波特率与校验设置。 |
| USB DFU | 芯片 ROM bootloader 或应用 DFU 的能力、启动模式、USB 连接及正确设备标识。 |
| 其他接口/探针 | 当前芯片 ROM、Programmer 或原有工具链明确支持的协议和硬件。 |

使用 `cube programmer -l` 的具体接口选项发现端口。ROM bootloader 支持依据目标官方文档及实际接口，不按“STM32 都支持”推断。

## 模式与占用

`NORMAL`、`HOTPLUG`、`UR` 等模式的支持与影响取决于目标、探针和工具版本。HOTPLUG 适合尽量保留运行现场的连接；UR 可能需要连接 NRST。不能通过更换 mode 解决错误供电或接线。

调试器、IDE、Programmer 和 ST-Link server 可能争用探针。先识别占用者；需要共享时各工具必须支持并正确配置共享模式。不要结束用户已有会话，或把共享模式当作通用默认值。

## 平台处理

- Windows：从设备管理器确认 ST-Link/串口/DFU 设备与驱动，端口变化后重新枚举。
- Linux：检查 USB 设备和官方 udev/组权限，确认当前用户能访问；路径依据当前设备发现。
- macOS：确认当前工具支持主机架构、USB 设备可见及系统访问条件，串口使用实际枚举路径。

## 运行证据

确认串口波特率与实际外设时钟匹配，记录与本次动作对应的响应。无输出时检查是否初始化前失败、是否复位到了 bootloader，以及 UART/USB 输出依赖是否就绪。

只有下载需求时交付下载和校验结果；涉及功能验证时把上板观察和构建结果分别说明。硬件缺失时完成产物、地址和连接参数准备，指出待补充的设备信息。

依据：[Programmer 官方产品与文档](https://www.st.com/en/development-tools/stm32cubeprog.html)、[STM32 系统存储器 boot 模式 AN2606](https://www.st.com/resource/en/application_note/an2606-stm32-microcontroller-system-memory-boot-mode-stmicroelectronics.pdf)。
