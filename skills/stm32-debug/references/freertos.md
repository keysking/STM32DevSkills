# FreeRTOS 运行诊断

先确认内核、port、API 适配层和调试符号。原生 FreeRTOS、CMSIS-RTOS v1/v2 的句柄和超时单位不同，以实际实现解释状态。

## 按现象取证

| 现象 | 核对方向 |
| --- | --- |
| 调度器未运行 | 启动调用和返回、任务/对象创建结果、heap、port、向量实现和初始时钟。 |
| 某任务不执行 | 任务是否创建、ready/blocked/suspended 状态、等待对象、优先级与高优先级忙循环。 |
| 延时或超时异常 | tick 是否推进、参数单位、tick 频率、HAL 时基、屏蔽中断及低功耗处理。 |
| 中断后异常/assert | ISR API、调用优先级、priority grouping、错误的移位宏和调度切换请求。 |
| 随机 Fault | 任务栈、堆、缓冲区越界、释放后使用、异步局部缓冲区和 cache。 |
| 死锁/长期等待 | mutex/queue/semaphore 的创建、持有者、等待链、锁顺序、超时和遗失通知。 |

## 栈、堆与对象

查看任务栈区、有效边界和 high-water mark；其单位通常为栈元素，核对内核 API 和 CMSIS 适配层，不能直接当字节。确认 ISR 使用的栈也有足够空间。

检查 heap 实现、总空间、当前可用空间、最小剩余空间和分配失败 hook；这些 API 取决于配置和 heap 方案。动态与静态分配混用时查清对象的生命周期和归属。

`configCHECK_FOR_STACK_OVERFLOW`、断言及 hook 可辅助重现，但启用它们需要满足工程定义和内存条件。不要只扩大 heap/stack 而不定位越界或泄漏。

## 调度与同步

使用已安装调试适配器枚举任务；不支持任务感知时从当前任务句柄、TCB、等待对象和源码获取最小必要信息，结构布局以当前内核符号为准。

暂停时看到某任务运行不证明其他任务从未运行，可加入轻量计数/时间戳或 trace。检查高优先级任务是否阻塞/让出，软件 timer 回调是否长时间占用，以及调用 HAL 阻塞函数是否等待无法推进的时基。

复查 ISR 的 `FromISR` 或 CMSIS 允许子集、零超时限制及内核中断优先级约束。互斥量不用在 ISR；`volatile` 不能代替跨任务同步。

修复后在相同负载、通信及中断条件下观察任务推进、超时、栈/堆和错误计数。没有板端证据时，把源码判断和运行确认分开。

依据：[FreeRTOS Kernel](https://github.com/FreeRTOS/FreeRTOS-Kernel)、[CMSIS-RTOS2](https://arm-software.github.io/CMSIS_6/main/RTOS2/index.html)、当前 SDK 的适配层和内核配置。
