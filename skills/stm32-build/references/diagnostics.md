# 构建诊断与固件产物

## 按错误所处阶段定位

| 阶段/症状 | 检查方向 |
| --- | --- |
| 配置失败 | 工具链、generator、SDK 路径、preset 继承、版本或工具变量。 |
| 头文件缺失 | 真实 include、组件是否安装、大小写、目标条件和生成文件是否存在。 |
| 宏/类型/函数不存在 | HAL 代际、模块启用、设备宏、预包含链及 SDK 配套版本。 |
| 未定义符号 | 源文件或库是否进入目标、条件编译、C/C++ 名字修饰和链接顺序。 |
| 重复符号 | ISR/回调/系统调用或 RTOS port 重复，重复加入源文件，错误的弱符号覆盖。 |
| 区域溢出 | 实际芯片容量、段布局、栈/堆、常量/缓冲区、链接脚本和内存银行。 |
| 固件行为未改变 | 实际编译目标、增量输入、输出文件与下载文件是否一致。 |

修复最早有意义的诊断后重建。不要用删除报错文件、伪造 HAL 宏或扩大链接容量掩盖错误。

## 产物检查

找到当前目标实际 ELF 和 map。确认设备、入口、向量表、段地址、Flash/RAM 布局和构建配置。多核、安全域及 bootloader 分区按各自链接布局判断。

所选 GNU 工具链提供相应程序时可用：

```text
arm-none-eabi-size -A "<固件.elf>"
arm-none-eabi-objdump -h "<固件.elf>"
arm-none-eabi-objcopy -O ihex "<固件.elf>" "<固件.hex>"
arm-none-eabi-objcopy -O binary "<固件.elf>" "<固件.bin>"
```

按[工具定位](../../stm32-env/references/tool-location.md)从所选工具链找到真实程序；不要假定这些名称都已在 shell PATH 或注册为 cube 命令。`cube starm-readobj --help` 也可发现 ELF 检查能力。

text/data/bss 汇总不能完整说明各存储区占用：初始化 data 同时涉及装载与运行地址，heap/stack 可能只是链接预留。多 RAM/外部存储布局应结合 map 和链接脚本逐区解释。

BIN 不携带目标地址，HEX/ELF 通常携带地址信息；转格式后检查地址跨度、输出大小和实际下载用途。保留与运行固件匹配的带符号 ELF，避免用另一配置的符号调试。

依据：[GNU Binutils](https://sourceware.org/binutils/docs/binutils/)、实际编译链接命令、当前链接脚本及 map。
