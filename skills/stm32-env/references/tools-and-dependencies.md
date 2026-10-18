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

## 环境缺失与替代边界

先按工具定位流程区分未安装、路径未发现和启动失败。确认缺失项确为当前任务所需后，说明其用途、受影响的步骤及建议安装或恢复的组件；安装来源和方法依据官方说明。用户已有明确要求时遵循其要求，否则按下表处理。

| 缺失项 | 默认处理 | 可接受的替代 |
| --- | --- | --- |
| CubeMX / CubeMX2 | 需要生成或重新生成工程时，建议安装对应生成器，不自行改成手写工程。 | 同一生成器的其他可用入口；用户明确选择无生成器方案。 |
| SDK、HAL、CMSIS、设备 Pack | 补齐工程要求的版本，不删减功能或编造接口绕过依赖。 | 满足版本要求的工程自带依赖、本地包或离线包。 |
| 编译器 | 建议安装或恢复工程要求的工具链，不自行转换编译器体系。 | 满足版本和目标架构约束的已有安装，包括其他工具附带的编译器。 |
| CMake、Ninja 等构建工具 | 优先补齐，保持工程既定构建方式。 | 兼容的独立安装，或工程已支持的其他构建入口。 |
| `cube` 命令入口 | 判断任务是否依赖其工具解析、环境和包管理能力；需要时建议补齐。 | 底层工具与依赖齐全、版本和环境满足工程约束时，直接调用原生工具。 |
| 烧录、调试工具 | 无可用工具满足任务时，建议安装。 | 已有工具经核实支持当前芯片、探针和所需操作，且不扩大擦除、复位等副作用。 |
| 搜索、文件查看、串口终端等辅助工具 | 可直接采用具备所需能力的现有工具。 | 系统命令、脚本或其他已有工具，保持结果和必要证据完整。 |
| 探针、板卡或必要驱动 | 说明需连接的硬件或需安装、修复的驱动，继续完成不依赖硬件的工作。 | 已有且受支持、能明确对应任务目标的设备；静态分析不能代替板端验证。 |

替代前确认能力和工程约束，使用后说明替代方式及验证结果。会改变生成、维护或构建方式的方案应先提出，由用户明确选择；不能因缺少工具自行采用。

只暂停依赖缺失环境的步骤，继续可独立完成的工作。例如已有源码仅需修改应用逻辑和构建、不涉及重新生成时，无需为该任务安装 CubeMX。尚未完成的生成、构建、烧录或验证应明确说明，不能宣称已完成。

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

## 传统 CubeMX 固件包的命令行安装

传统 CubeMX 的 MCU 固件包使用其自身的 `swmgr` 管理，不套用 `cube pack install`。先从 `.ioc` 的 `ProjectManager.FirmwarePackage` 或新工程所选目标确定系列和版本，按[工具与 SDK 定位](tool-location.md)检查实际仓库，优先复用满足要求的本地包，不为补齐依赖自动升级已有工程。

按[交互会话入口](cubemx-runtime.md#固件包管理的交互入口)启动 CubeMX。以下命令在 `MX>` 会话内逐条执行，不是 shell 命令；尖括号内容需替换，先通过 `help` 和 `swmgr` 核对当前版本支持的语法。

### 在线下载与安装

```text
help
swmgr
login <myST邮箱> <密码> <remember_me>
swmgr refresh
swmgr install stm32cube_<系列>_<版本> ask
```

`swmgr` 不带参数用于显示帮助，可能同时返回用法错误；这不代表安装失败。包标识采用当前软件包目录中的名称，例如官方示例 `stm32cube_f1_1.8.0`，不是安装后的目录名 `STM32Cube_FW_F1_V1.8.0`；示例版本不作为默认值。

在线下载前确认命令行认证成功，不假定 GUI 登录状态能被该会话复用。`remember_me` 按当前帮助和用户选择填写。需要输入凭据时让用户在本机交互界面完成，不要求在聊天中提供密码，也不把实际凭据写入脚本、命令记录或日志；无法安全交接交互时，改由用户在 CubeMX 包管理器完成登录和安装。

许可证参数按当前帮助选择；已核对的帮助列出 `deny|ask`，不能自行添加 `accept`、`yes` 等静默接受参数。`ask` 可能需要额外交互或弹窗，等待实际提示再处理，不预先发送固定回答，也不将拒绝许可证当作成功安装。需要用户操作时说明具体提示和待完成步骤。

### 本地固件包安装

当前帮助支持 `full_filepath` 时，可在同一会话中安装官方本地 ZIP：

```text
swmgr install "<固件ZIP绝对路径>" ask
```

若命令行安装或交互不可用，使用 `Help > Manage embedded software packages > From Local…` 导入。补丁包需要对应基础包，不能将补丁 ZIP 当作完整固件库。在线下载失败时检查认证、代理、证书和包版本，不切换生成器或手写库绕过。

安装后检查完成信息、目标版本是否被 CubeMX 识别，以及仓库中驱动和必要组件是否完整；不能仅凭目录存在或进程退出码判定成功。确认工程仍使用预期版本后，返回原任务继续生成和构建，最后用 `exit` 结束本次会话。安装命令帮助可用不等于下载、许可证交互和安装全流程已经验证。

依据：[CubeMX CLI](https://dev.st.com/stm32cube-docs/stm32cubemx/6.18.1/en/docs/markup/CubeMX_CLI.html)、[命令行认证说明](https://dev.st.com/stm32cube-docs/stm32cubemx/6.18.1/en/docs/markup/CubeMX_RelaseNotes/sections/3_1_command_line_authentication_required.html)、[固件包安装与补丁说明](https://dev.st.com/stm32cube-docs/stm32cubemx/6.18.1/en/docs/markup/CubeMX_UserManual/chapters/03_3_installing_and_running_stm32cubemx.html)及当前 `swmgr` 帮助。

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
