# 已有构建体系

## 发现构建入口

先阅读工程说明、CI 或构建脚本，确认真实工具和配置。文件扩展名用于定位入口，不保证工具已经安装或当前主机受其支持。

UV4、IarBuild 或 make 不在 PATH 时采用[工具定位](../../stm32-env/references/tool-location.md)，通过安装记录和选定软件目录取得绝对路径，不直接要求用户手动编译。目标名称从工程文件取得；PowerShell 的 XML 查询示例：

```powershell
[xml]$keilProject = Get-Content -LiteralPath '<工程.uvprojx>' -Raw
$keilProject.SelectNodes('//Target/TargetName') | ForEach-Object { $_.InnerText }
[xml]$iarProject = Get-Content -LiteralPath '<工程.ewp>' -Raw
$iarProject.SelectNodes('//configuration/name') | ForEach-Object { $_.InnerText }
```

只执行当前工程对应的一组。POSIX 或其他环境可用已安装 XML 工具查询相同节点，或通过 `rg -n -C 3 '<TargetName>|<configuration>|<name>' <工程文件>` 查看上下文；不能把所有 `name` 元素都当成构建配置。选择用户指定或与目标固件对应的一项，检查其芯片和输出位置。

| 体系 | 操作方法 |
| --- | --- |
| Makefile | 检查默认目标、工具链前缀、依赖、输出目录和并行兼容性，使用工程原有 `make` 入口。 |
| Keil MDK | 定位 UV4 和 `.uvprojx` 中真实 target，查询工具命令行。常见批构建形态为 `UV4.exe -b "<工程.uvprojx>" -t "<target>" -o "<日志>"`。 |
| IAR EWARM | 定位 IarBuild、`.ewp` 及真实 configuration，查询帮助。常见形态为 `IarBuild "<工程.ewp>" -build "<配置>"`。 |
| IDE/headless 或其他脚本 | 沿用已有官方 headless 或脚本入口，确认工作目录、目标、返回值和实际产物。 |

命令形式依已安装版本确认；许可证、主机支持及设备支持包也属于构建条件。不能因为技能覆盖三平台就假定 Keil/IAR 的每个版本都能在三平台执行。

## 修改与验证

保持原有编译器、启动文件、链接脚本、宏定义和输出布局。新增源文件和 include 必须加入实际目标；检查各 target/configuration 是否需要分别加入。

构建日志是定位依据，检查错误统计及产物更新，不只看 shell 是否成功启动 IDE。某些 GUI 工具可异步返回，需等待其真实构建完成。修复环境问题时不要顺便把已有项目迁移到 CMake。

命令行构建入口不可用时，明确缺少的工具、许可证或官方自动化能力，保留已经完成的源码与配置分析。若用户要求迁移，再单独核对原配置和迁移后的行为。

依据：[Keil µVision 命令行](https://www.keil.com/support/man/docs/uv4/uv4_commandline.htm)、[IAR 官方文档入口](https://www.iar.com/knowledge/learn)，以及当前工具帮助。
