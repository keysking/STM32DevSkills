# FreeRTOS API、集成与任务设计

## 识别内核和 API

确认实际 FreeRTOS 版本、CPU port、`FreeRTOSConfig.h`、heap 实现和适配层源码。CMSIS-RTOS 是 API 层，不能仅凭 `cmsis_os*.h` 判断底层内核。

| 项目 | 原生 FreeRTOS | CMSIS-RTOS v1 | CMSIS-RTOS v2 |
| --- | --- | --- | --- |
| 常见头文件 | `FreeRTOS.h`、`task.h` 等。 | `cmsis_os.h`。 | `cmsis_os2.h`。 |
| 任务创建 | `xTaskCreate` / 静态创建 API。 | `osThreadDef`、`osThreadCreate`。 | `osThreadAttr_t`、`osThreadNew`。 |
| 延时 | `vTaskDelay`，参数为 tick；毫秒可用 `pdMS_TO_TICKS` 转换。 | `osDelay` 的 API 参数为毫秒，检查实现的换算。 | `osDelay` 的参数为内核 tick，结合 `osKernelGetTickFreq` 换算。 |
| 栈 | `xTaskCreate` 深度以 `StackType_t` 元素为单位。 | 规范描述为字节，但部分 ST 适配层直接传递 FreeRTOS 栈深度；必须检查实现。 | `osThreadAttr_t.stack_size` 为字节。 |
| 消息队列 | `xQueueCreate` 指定每个元素的字节数。 | `osMessagePut` 的消息接口为 32 位值，检查适配层限制。 | `osMessageQueueNew` 指定元素大小，put/get 传入数据地址。 |

沿用工程 API。不要把一个 API 的句柄强制转换后交给另一层，或把毫秒、tick、字节和栈元素数量直接互换。优先阅读当前 SDK 的适配层实现，而不是假定它支持接口中的每个可选特性。

部分 ST CMSIS v1 适配层的 `osThreadDef` 接收名称、函数和其他选项共五个参数，与常见四参数示例不同。检查宏定义和 `osThreadCreate` 实现，尤其确认 stacksize 是否在传给 `xTaskCreate` 前做了单位转换；不能只依赖头文件注释。

## 集成顺序

1. 选择与芯片、编译器和浮点设置匹配的内核 port；把内核、适配层及一个适当的 heap 实现加入真实构建目标。
2. 核对内核配置、静态/动态分配能力、总堆空间、任务栈和断言/失败 hook。
3. 按工程方式初始化内核、创建对象和任务、启动调度器；检查所有创建结果，任务函数不能直接返回。
4. 核对 SysTick/其他时基、SVC、PendSV 和实际向量实现，避免生成代码与 port 重复定义。HAL 时基与内核 tick 的共存方式依 SDK 处理。
5. 从简单的可观察任务和一次通信开始验证，再加入并发逻辑。生成器没有提供所需内核/API 组合时使用匹配的官方集成方式，并记录实际条件。

## 中断与同步

原生 FreeRTOS ISR 使用相应 `FromISR` API，并按 port 规则请求必要的调度切换。CMSIS API 只有文档允许的子集可用于 ISR，超时通常必须为 0；互斥量操作不用于 ISR。

使用 RTOS API 的中断需满足 `configMAX_SYSCALL_INTERRUPT_PRIORITY` 等端口约束。Cortex-M 数值越小通常表示抢占优先级越高；硬件移位值与 library 宏的未移位值要区分。按实际 `__NVIC_PRIO_BITS`、priority grouping 和 port 判断，不固定套用某个优先级数字。

互斥量用于资源互斥，队列/通知/信号量用于交接事件或数据；明确对象生命周期和等待超时。任务中使用会忙等的 HAL 延时或阻塞调用时考虑调度和超时依赖。软件 timer 回调及 ISR 保持短小，耗时处理交给任务。

## 检查

确认任务能调度、队列或通知能交接、超时单位正确，栈/堆有余量。检查 stack high-water mark 的单位、malloc/stack hook、优先级与互斥等待；实际验证到哪一步据实说明。

依据：[FreeRTOS Kernel 源码与头文件](https://github.com/FreeRTOS/FreeRTOS-Kernel)、[CMSIS-RTOS v1](https://arm-software.github.io/CMSIS_5/RTOS/html/index.html)、[CMSIS-RTOS2](https://arm-software.github.io/CMSIS_6/main/RTOS2/index.html)、[ST FreeRTOS 中间件](https://github.com/STMicroelectronics/stm32-mw-freertos)。
