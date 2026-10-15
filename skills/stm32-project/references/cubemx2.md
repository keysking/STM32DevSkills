# CubeMX2 工程路径

适用于当前 `cube mx` 提供的 MX2 命令及 `.ioc2` 工程。先确认目标设备在已选 SDK/Pack 和生成器版本中可用，不能根据另一个系列的示例推断支持。

cube 或 mx 不可用时先读取[工具定位](../../stm32-env/references/tool-location.md)及[命令与依赖发现](../../stm32-env/references/tools-and-dependencies.md)。已有 `.ioc2` 无需再次创建，直接采用[配置与生成流程](../../stm32-configure/references/cubemx2.md)。

## 创建和软件工程

```text
cube mx --help
cube mx project create-from-mcu --help
cube mx project create-from-mcu --cpn "<准确CPN>" --project-location "<新目录>" --project-name "<工程名>"
cube mx project get-sw-project-list --project-path "<工程.ioc2>"
```

不知道准确设备标识时，依次查询 `cube mx finder --help`、`cube mx finder device --help`，再使用当前提供的 list/info 及其 leaf 帮助查询目标 CPN/PPN。板卡工程使用 `project create-from-board --help` 发现参数，板卡标识从当前帮助支持的查询或 SDK 元数据取得。Pack 发现见[依赖](../../stm32-env/references/tools-and-dependencies.md)。创建可能下载所需依赖；先复用已有合法本地包，检查实际返回状态和文件。

如果还没有所需软件工程，使用 `project create-software --help`，再创建对应的软件工程。已有软件工程时不要重复创建。多核、安全域或多个软件工程需明确选择 `--software-project`。

## 首次导出与后续维护

创建完成后，按[MX2 配置、保存和导出](../../stm32-configure/references/cubemx2.md)选择完整软件目标、输出格式和新目录，首次导出后按[构建](../../stm32-build/SKILL.md)验证。参数查询、生命周期、已有工程重新生成和用户代码合并均集中维护在该配置资料中。

创建完成需要实际 `.ioc2` 路径、查询得到的软件工程和目标、成功生成的源码及任务要求的构建产物。创建返回错误、没有软件工程或设备依赖缺失时先解决对应问题，不继续盲目导出。

依据：[MX2 官方文档入口](https://dev.st.com/stm32cube-docs/stm32cubemx2)、当前 `cube mx … --help`。不同版本以实际帮助和工程查询为准。
