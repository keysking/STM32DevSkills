# 工具、版本与依赖

## 发现顺序

在目标工程目录运行，先查看工程的 `.settings/bundles.store.json`、`.settings/bundles-lock.store.json`、CMake presets 和 SDK 元数据。cube 不在 PATH 时先按[工具定位](tool-location.md)找到入口，再查询工具：

```text
cube --version
cube --list
cube --json bundle list
cube --json --resolve cmake
cube --tool-paths
cube cmake --version
cube ninja --version
```

wrapper 与 Bundle 版本分开判断。`--list` 取决于已加载的 Bundle；安装了编译器 Bundle 不代表存在同名 wrapper 命令。解析结果可能包含环境信息，只保留与当前诊断相关的程序及工具链路径。

工程版本约束优先于机器上恰好安装的最新版。父目录也可能影响 `.settings` 的发现；解析结果不符合预期时检查运行目录及父目录。锁文件可能含多个平台的解析结果，保留其他平台条目。

## 从帮助找到可执行命令

1. 用 `cube --list` 确认需要的命令是否注册；仅对存在的命令使用 `cube --json --resolve <命令>`，核对 Bundle 名、版本及调用入口。
2. 逐层查询，例如 `cube mx --help` → `cube mx sw-config --help` → `cube mx sw-config list-parameters --help`。执行前核对 leaf 帮助的必需参数、合法选项和示例，不混用上层或相邻命令的 flag。
3. 帮助只说明语法。实例名、软件工程、参数路径和合法值从目标工程的 list/view/get 等查询获得；保存与生成操作前，先做对应的只读查询确认上下文。
4. 返回结构化结果时检查 `status`、`error`、`diagnostics` 和实际数据。只有帮助文本、空列表或进程退出 0，不能证明目标配置已找到。
5. 查询为空先核对工程路径、软件工程、资源是否启用以及版本支持；得到合法参数后再修改，不能从另一芯片示例猜值。

传统 CubeMX 使用自己的脚本帮助，不使用上述 wrapper 子命令语法，入口见[脚本会话](cubemx-runtime.md)。

## Bundle 与 Pack

```text
cube bundle --help
cube bundle list-online
cube pack --help
cube pack list packs --help
cube pack list packs
```

- Bundle 提供执行工具，例如 CMake、Ninja、编译器、Programmer 或 MX2。
- Pack 提供设备、驱动、板卡、组件和模板。确认 vendor、name、version 及依赖，不把型号或组件名当成 Bundle ID。
- 传统 CubeMX 的 MCU 固件仓库、MX2 的 Pack 仓库和 Cube Bundle 仓库分别定位，不能互换。

补齐依赖前查询对应子命令帮助；按工程声明安装需要的版本。存在 `cube bundle install` 时使用其帮助给出的标识与版本语法；Pack 安装示例为 `cube pack install VENDOR.NAME.VERSION`，这些字段必须从索引或工程需求获得。离线工程优先复用已安装包或合法的本地包路径。

需要 CMSIS 扫描时先检查帮助。若扫描器未正确使用环境中的 Pack root，显式传入 `--cmsis-pack-root`。扫描输出为空或进程返回 0 均不能独立证明设备已找到。

## 编译器与辅助工具

GCC Bundle 可能通过 `PATH`、`TOOLCHAIN_ROOT`、`GCC_TOOLCHAIN_ROOT` 等声明工具。按[编译器定位](tool-location.md)取得真实程序；CMake 使用的编译器最终以 cache 和实际编译命令为准。

`cube --with <版本或范围> <命令>` 选择该命令所属 Bundle 的版本，不能用它凭空创建未注册的编译器命令。项目仍应通过实际工具链配置选择 GCC/Clang。

根据任务发现 `programmer`、GDB、ST-Link server、语言服务或 ELF 检查工具。探针升级、驱动安装和全局配置修改只在确有需要时执行。`--dry-run` 仅对明确支持它的操作有效。

## 失败判断

| 现象 | 优先核对 |
| --- | --- |
| cube 不存在 | 安装是否完成、当前 shell 的 PATH、可执行权限。 |
| 命令不存在 | Bundle 是否安装/加载、项目版本约束、当前平台架构支持。 |
| SDK 或设备找不到 | 仓库位置、Pack 索引、准确型号、版本及离线依赖。 |
| 下载失败 | 网络、代理、证书、官方服务及所需登录条件；不要输出凭据。 |
| 工具能启动却不能构建 | 实际编译器、构建缓存、启动/链接文件和 SDK 配套版本。 |

依据：[CubeCLI 总览](https://dev.st.com/stm32cube-docs/stm32cubecli/latest/en/index.html)、[命令参考](https://dev.st.com/stm32cube-docs/stm32cubecli/latest/en/docs/markup/STM32CubeCLI_Command_Reference.html)、当前工具帮助。
