# 从用户与当前工程识别系列

此阶段为选择技能包收集目标信息，不修改工程或调用生成器。分别记录已确定的系列、准确型号是否已知，以及用户或工程中的依据；不要将一次关键词命中直接当成当前构建目标。

## 用户输入

忽略型号字母大小写，允许省略 `STM32` 前缀。`F103`、`F105`、`F107` 属于 F1；`C532` 属于 C5。`F1`、`C5` 系列名也足够选择包。保留用户给出的完整型号，后续不以工具的通用器件名称替换它。

`F103`、`C532` 等简称足够选包，但新建工程仍需准确料号及其封装、容量。不能自动补成文档示例中的 `STM32F103C8T6` 或 `STM32C532CCT6`。

仅有 `STM32`、`Cortex-M3`、开发板昵称或无依据的型号碎片时，继续查找工程。不要把 GD32F103 等其他厂商器件当成 STM32F103。

用户在当前工程要求修改现有固件且明确型号与配置冲突时，询问是否需要换目标；不要自动迁移工程。多个目标工程按用户指出的路径和实际构建预设选择，仍有歧义时询问。

## 找到目标配置

先定位用户指定工程；没有指定时检查当前工作目录。Git 根目录只是定位线索，一个仓库可能包含多个固件工程。新建工程使用用户指定的目标目录；目录未定且无法从任务判断时先询问安装落点。

在候选工程根目录，使用 `rg` 限定文件类型和排除范围：

```bash
rg --files --hidden -g '*.ioc' -g '*.ioc2' -g '*.csolution.yml' -g '*.cproject.yml' -g 'project.cube' -g 'hw-composition.store.json' -g 'CMakeLists.txt' -g 'CMakePresets.json' -g '*toolchain*.cmake' -g '*.uvprojx' -g '*.cproject' -g '*.ewp' -g '!**/.git/**' -g '!**/.agents/**' -g '!**/.codex/**' -g '!**/.claude/**' -g '!**/build/**' -g '!**/Drivers/**' -g '!**/Middlewares/**' -g '!**/Examples/**' -g '!**/.work/**'
```

该 `rg` 命令也可直接在 PowerShell 运行。找到候选后只读取相关配置；没有 `rg` 时使用文件浏览工具，或在已定位的目录用 `Get-ChildItem` / `find` 查找相同文件。不要递归扫描整块磁盘。

| 文件 | 依据及限制 |
| --- | --- |
| 传统 CubeMX `.ioc` | `Mcu.CPN` 为具体料号，`Mcu.Family` 为系列，`Mcu.Name` 可能覆盖多个容量变体；优先保留具体料号。 |
| CubeMX2 `.ioc2` | 包含多份文件内容的容器，不能当作单个 JSON。定位当前硬件组成中的 `mcuUID`、solution 的 `device` 等实际目标字段并读上下文。 |
| `.csolution.yml` / `.cproject.yml` | 当前选中 target/build type 的 `device`，不是其他 target 的默认值。 |
| `project.cube` / `.settings/hw-composition.store.json` | 沿当前工程引用的硬件文件查 `mcuUID` 等选型字段。 |
| CMake、预设、Keil/IAR 等配置 | 当前使用的 device 定义、启动文件和链接脚本互相印证。能识别系列不意味着系列包支持该构建体系。 |

已确定文件路径后，例如：

```powershell
$targetConfig = 'D:/firmware/Control/Control.ioc' # 替换为已定位的文件
Select-String -LiteralPath $targetConfig -Pattern '^Mcu\.(CPN|Family|Name)=|mcuUID|device\s*[:=]|STM32[A-Z][0-9]'
```

```bash
target_config='/work/Control/Control.ioc' # 替换为已定位的文件
rg -n -i 'Mcu\.(CPN|Family|Name)=|mcuUID|device[[:space:]]*[:=]|STM32[A-Z][0-9]' "$target_config"
```

头文件、README、SDK 库和历史编译日志仅作辅助证据。`-mcpu=cortex-m3` 无法区分具体 STM32 系列；芯片家族目录名称、缓存里的旧型号也不能单独决定目标。

## 结果与下一步

- 系列已明确：回到[入口](../SKILL.md)选择系列，无需 ST-Link。准确型号尚缺时保留“未确定”，无需为安装技能立即追问。
- 工程配置缺失或证据不足以确定系列：继续[ST-Link 探测](probe.md)。系列已知但具体料号未知时，不通过重复探测补齐料号。
- 多个活动目标未指定或配置互相冲突：先请用户选择，不通过随机连接一块板消除工程歧义。
- 已识别但系列无对应包：说明暂未支持，不再探测硬件寻找另一个可用系列。

## 后续需要准确型号时

新建工程在创建目标相关配置或生成工程前，必须确定准确芯片型号；其他任务在操作确实依赖准确型号时再确认。

优先使用用户明确提供的完整料号，或本次任务工程中明确的选型字段。`Mcu.Family`、覆盖多个容量的 `Mcu.Name`、ST-Link 的候选器件组均不足以替代具体料号。用户未提供且当前工程也没有时，询问用户提供完整型号（例如芯片丝印上的料号），得到答案后再继续相关步骤。保留已经安装的系列技能和原任务上下文，不默认选定候选组中的某一个型号。
