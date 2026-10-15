# Programmer 下载、校验与启动

## 先发现目标和固件

工具入口未知时按[工具定位](../../stm32-env/references/tool-location.md)查 cube 命令或独立 `STM32_Programmer_CLI`；先验证帮助，之后再枚举探针。固件和链接布局不明确时读取[构建与产物](../../stm32-build/SKILL.md)，不要仅凭扩展名选文件。

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

| 现象 | 下一步 |
| --- | --- |
| 命令不存在 / 无法启动 | 返回[环境定位](../../stm32-env/references/tool-location.md)，不把工具错误解释为目标故障。 |
| 枚举不到探针或接口 | 按[连接检查](connections.md)确认 USB/串口、驱动、权限和占用。 |
| 多个探针 | 取得与用户目标对应的序列号，显式选择；无法确认时请求目标标识。 |
| 探针可见但目标连接失败 | 检查供电、SWD/复位接线、占用；再依据帮助选择支持的降频或 under-reset，不直接全片擦除。 |
| 校验失败 | 检查地址、芯片、存储保护、loader 和供电，保留失败范围，不重复写入同一未知条件。 |
| 下载校验通过但应用异常 | 检查启动地址、boot 配置及复位结果；需要定位时采用[调试](../../stm32-debug/SKILL.md)。 |

只在任务确实涉及时修改 option bytes、RDP、OTP、密钥和生命周期状态；这些操作可能擦除数据或不可逆。先核对对应芯片文档、现有状态和用户要求，不把解除保护作为普通连接重试步骤。

依据：[STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html)、当前 `programmer --help` 及目标参考手册。
