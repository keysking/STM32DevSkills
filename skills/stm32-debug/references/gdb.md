# GDB、ST-Link 与运行证据

## 准备与连接

确认探针、目标、匹配当前固件的 ELF 和调试符号。工具路径或 wrapper 命令不明时先按[工具定位](../../stm32-env/references/tool-location.md)查 GDB、ST-Link server 和 Programmer；对已发现的入口执行：

```text
cube arm-none-eabi-gdb --version
cube stlink-gdbserver --help
cube programmer --help
```

常见 ST-Link server 启动形式如下，具体选项依据当前帮助：

```text
cube stlink-gdbserver -d -p <本地端口> -i "<探针序列号>"
cube arm-none-eabi-gdb "<固件.elf>"
```

需要保留当前运行现场时查询并选择 attach 模式；某些版本为 `-g`。创建服务时避免占用已有端口、探针或用户会话；结束本次会话时仅清理自己启动的服务。

探针枚举与连接参数复用[Programmer 的目标发现](../../stm32-flash/references/programmer.md)及[连接检查](../../stm32-flash/references/connections.md)，不把重新下载作为连接前提。先读 server 日志，确认实际监听端口和目标连接状态，再启动 GDB；只启动了进程不能证明调试链路可用。

在 GDB 中：

```gdb
target remote localhost:<本地端口>
monitor help
info registers
bt
info threads
info files
p/x <相关变量或寄存器表达式>
x/16wx <经确认可读取的地址>
```

首次连接、暂停、复位和断点都可能影响现场。先决定要观察运行状态还是重新复现。`load` 会写入固件，不把它作为只读 attach 的默认步骤。monitor 命令由 server 实现，不跨服务器套用。

## 连接失败时

| 现象 | 下一步 |
| --- | --- |
| server 报工具/依赖缺失 | 检查解析入口、Programmer 路径及配套版本，返回工具定位。 |
| server 报端口占用 | Windows 用 `Get-NetTCPConnection -LocalPort <端口>`；Linux/macOS 用已安装的 `ss` / `lsof` 查询占用，选择可用端口；不终止未知用户进程。 |
| server 无法连接目标 | 采用烧录技能的连接检查；确认参数及探针占用，不通过重复启动 server 解决。 |
| GDB 拒绝连接 / 超时 | 核对 server 是否就绪、实际 host/port 和日志；远程连接另查任务实际网络路径。 |
| 无符号或地址明显不匹配 | 核对 ELF、目标内核及实际运行镜像；需要重新构建/下载时分别采用[构建](../../stm32-build/SKILL.md)和[烧录](../../stm32-flash/SKILL.md)。 |

成功连接后应能解释目标状态并读取所需寄存器或符号信息；随后才进入具体故障分析。新证据没有变化时停止同参数重试。

## 符号、内存和产物

对比 ELF 的段/向量与实际目标，确保符号属于当前运行固件。优化会使变量被省略或源码步进跳跃；必要时使用适当的 Debug 配置重现，但先保留原故障条件。

读取内存和外设寄存器前确认地址、访问宽度及读副作用。多核、安全域和 bank remap 可能影响可见地址；不能固定假定所有芯片的 Flash 起点、VTOR 或 AP。

Programmer 的 `-coreReg`、内存读取等也可获取证据，实际命令依帮助；核心寄存器读取可能需要暂停。硬件断点和 watchpoint 数量有限，按当前核心能力选择。

## 跟踪与任务感知

SWV/SWO/ITM 需内核和板卡连线支持，CPU/trace 时钟参数与实际频率一致。RTOS 任务感知依 server、GDB 和适配器支持；先查询 `rtos-proxy` 或其他已安装工具的帮助，不能仅凭出现该命令就认定已能枚举所有任务。

依据：[GNU GDB](https://sourceware.org/gdb/current/onlinedocs/gdb.html/)、当前 GDB server/Programmer 帮助和目标 CMSIS 头文件。
