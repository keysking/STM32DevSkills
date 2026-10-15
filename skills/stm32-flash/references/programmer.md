# Programmer 下载、校验与启动

## 先发现目标和固件

```text
cube programmer --help
cube programmer -l st-link-only
```

使用当前版本提供的仅枚举探针选项；普通枚举可能连接目标。确认探针序列号、芯片标识、Flash 容量、连接方式及实际固件。多目标时明确选择，不能按列表顺序猜测。

ELF/HEX 通常携带装载地址；BIN 必须依据链接布局指定地址。核对 bootloader 偏移、向量表、多个镜像、安全域、内核和外部存储加载器。新生成的文件名或时间戳不能独立证明芯片匹配。

## 下载模板

确认当前帮助支持这些选项后，替换参数：

```text
cube programmer -c port=SWD sn="<探针序列号>" -d "<固件.elf>" -v -rst
cube programmer -c port=SWD sn="<探针序列号>" -d "<固件.bin>" <装载地址> -v -rst
```

下载、校验和复位按任务需要组合。连接频率、mode、reset 和 AP 选择由目标及帮助确定。已有工程使用独立 `STM32_Programmer_CLI` 时可沿用对应入口与参数。

内部 Flash 下载通常处理所需扇区，不默认添加全片擦除。外部 Flash、特殊安全产品或多个镜像使用匹配目标的官方加载和启动流程；不要把普通内部 Flash 示例强套到它们。

## 判断结果

- 检查实际连接设备、下载地址范围、文件、校验及结束诊断。退出码与日志一起判断。
- 下载完成后按任务复位或启动；CPU 已启动不等于应用完成初始化。
- 功能任务使用串口、GPIO/波形、协议响应或寄存器证据检查实际运行。必要时对比 ELF 段与目标内容，确认运行的确为本次固件。

## 失败与特殊操作

连接失败先排查供电、探针、接线、驱动/权限和占用，再尝试目标支持的降频或 under-reset。校验失败检查地址、存储保护、loader、供电及目标型号。

只在任务确实涉及时修改 option bytes、RDP、OTP、密钥和生命周期状态；这些操作可能擦除数据或不可逆。先核对对应芯片文档、现有状态和用户要求，不把解除保护作为普通连接重试步骤。

依据：[STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html)、当前 `programmer --help` 及目标参考手册。
