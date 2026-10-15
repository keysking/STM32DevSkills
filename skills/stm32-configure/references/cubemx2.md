# CubeMX2 配置

使用当前 `cube mx` 帮助确认子命令。实际调用传入 `.ioc2` 路径；存在多个软件工程时指定 `--software-project`。路径、实例、参数路径和取值由当前项目发现。

工具或命令不可用时读取[工具定位](../../stm32-env/references/tool-location.md)和[命令与依赖](../../stm32-env/references/tools-and-dependencies.md)。尚未创建工程时采用[MX2 工程创建](../../stm32-project/references/cubemx2.md)。下列步骤维护已存在的工程。

## 先确认工程与后端

1. 定位当前任务的 `.ioc2`，使用绝对路径。查询 `cube mx project get-sw-project-list --help`，再对该路径执行 `get-sw-project-list`，取得软件工程名。
2. 多软件工程选择与目标内核、安全域及固件对应的一项；需要 `--software-project` 的命令显式传入该名称，不使用猜测的默认工程。
3. CLI 可能启动无界面后端或连接现有 host/port。帮助查询不需要工程；实际操作确认连接的是预期后端，复用已建立会话，保留用户 GUI 的未保存修改。
4. 第一次查询必须返回所需工程数据，不能只看外层退出码。出现 `status: error` 或 diagnostics 时先解决路径、版本、后端或 Pack 依赖问题。

## 操作映射

| 对象 | 发现与查询 | 修改方法 |
| --- | --- | --- |
| 引脚 | `pinout all-pins`、`pin`、`assigned-pins`。 | `assign-signal`、`unassign-signal`、标签及分组命令。 |
| 外设 | `peripherals list`、`view`、`description`。 | `enable`/`disable`，再处理组件参数。 |
| 参数 | `sw-config list-configurations`、`list-parameters`、`get-parameter-possible-values`、`get-parameter-value`。 | `set-parameter-value` 或当前版本的批量设置命令。 |
| 时钟 | `clock get-clocks`、`get-clock-items`、`list-possible-values`。 | `set-value` 等模型接口，再查询 `get-clock-status`。 |
| DMA/IRQ | 当前 `dma`、`nvic`、`exti` 命令组帮助和资源查询。 | 先发现资源关联、合法选项，再设置并回读。 |
| 软件组件 | `composer`、`middleware` 的帮助、可用组件及工程实例。 | 选择匹配 SDK 的组件，然后配置对应实例。 |

## 从查询到修改的示例

下例展示组件参数的完整顺序。先逐条查询对应 `--help`，替换工程和软件工程；配置列表用于取得资源名，参数列表用于取得参数路径，不预先填写其他芯片的实例：

```text
cube mx project get-sw-project-list --project-path "<工程.ioc2>"
cube mx sw-config list-configurations --project-path "<工程.ioc2>" --software-project "<软件工程名>"
cube mx sw-config list-parameters --project-path "<工程.ioc2>" --software-project "<软件工程名>" --resource-name "<列表中的资源>"
cube mx sw-config get-parameter-value --project-path "<工程.ioc2>" --software-project "<软件工程名>" --resource-name "<资源>" --parameter-path "<返回的参数路径>"
cube mx sw-config get-parameter-possible-values --project-path "<工程.ioc2>" --software-project "<软件工程名>" --resource-name "<资源>" --parameter-path "<返回的参数路径>"
cube mx sw-config set-parameter-value --project-path "<工程.ioc2>" --software-project "<软件工程名>" --resource-name "<资源>" --parameter-path "<返回的参数路径>" --value "<合法值>"
cube mx sw-config get-parameter-value --project-path "<工程.ioc2>" --software-project "<软件工程名>" --resource-name "<资源>" --parameter-path "<同一参数路径>"
```

资源未出现时先检查外设启用、软件组件是否加入及绑定到该软件工程。参数隐藏时检查其上游使能/模式，使用当前 `list-available-parameters` 等接口查询可见参数；返回数值范围或自由输入说明时依据该约束取值。查询不到合法值不意味着可以随意设置。`--use-internal-value` 等选项会改变值的表示，查询和设置时保持一致。

列表可能按完整组件 ID 分组，实例中含 `name`、`instanceId` 和 `resource`。`--resource-name` 取实际 `resource` 字段，不能误用实例显示名或 UUID；无硬件绑定且仅一个实例的组件，也可能允许用组件 ID 的 csub 短名，按当前 leaf 帮助确认。参数列表的 `data` 可能直接是路径字符串数组，选取返回的完整路径作为 `--parameter-path`，不要添加外层 `data` 前缀或自行改名。枚举结果常在 `data.oneOf` 中给出 `const` / `title`，范围参数则检查类型和边界；依帮助选择内部值或显示值表示，设置后比较同一表示下的回读。

引脚先用 `pinout all-pins`、`pin --pin` 等查询当前封装及合法信号。已经发现目标信号后：

```text
cube mx pinout assign-signal --project-path "<工程.ioc2>" --pin "<引脚>" --signal "<信号>"
cube mx pinout pin --project-path "<工程.ioc2>" --pin "<引脚>"
```

每条 leaf 命令先看 `--help`，不要把其他命令的 flag 拼到当前命令中。参数数组、对象或布尔值依帮助规定编码；shell 传参见[平台差异](../../stm32-env/references/platforms.md)。

分配 GPIO 信号后，继续查询该引脚的软件参数，明确设置输入/输出、电气类型和上下拉并回读。生成器可能规范化标签为宏名，检查生成后的真实名称及引用，而不是按输入标签的大小写直接拼接标识符。

## 组件与时钟一致性

FreeRTOS 及其 API 适配层需要当前 SDK 实际提供。确认软件工程绑定资源、时基、内核、heap 实现和异常处理链。依赖解析通常由模型自动触发；不要手动调用标注为内部或不建议手动使用的 resolver。

时钟求解可能修改关联节点，使用 solver 后回读受影响频率、约束和错误。不能把节点 ID、最大频率或外部晶振值当成跨系列常量。

## 保存、重载和关闭

先回读目标参数，再执行 `cube mx project save --project-path "<工程.ioc2>"`。为验证磁盘持久化，查询 `project close --help`，关闭本次任务拥有的工程，再通过参数查询按同一路径重新加载并回读。

`.ioc2` 是含多个 store 的复合文本，不按纯 JSON 改写。保留 GUI 的未保存编辑，不用旧会话覆盖新设置；无法安全关闭共享会话时说明尚未完成独立重载验证，保留已完成回读结果。

生成可能改变导出选项等模型状态，结束前保存需要保留的变化。关闭提示未保存时先核对并保存，不用 `--discard` 掩盖丢失设置。

## 选择目标并导出

```text
cube mx ide-project show-config --project-path "<工程.ioc2>"
cube mx ide-project build-target list --project-path "<工程.ioc2>"
cube mx ide-project list-format --help
cube mx ide-project set-target --help
cube mx ide-project generate --help
```

构建目标形态为 `[projectName][.buildType][+targetType]`。列表返回以点开头的后缀时，结合已查询的软件工程名组成完整目标；不要只传后缀。缺少所需类型时查 `build-target add --help`，创建后重新列出目标。

新建且未指定构建体系时采用 CMake + GCC；已有工程沿用原格式。以下只用于已确认支持这些选项的 CMake/GCC 导出；输出为新的空目录：

```text
cube mx ide-project set-target --project-path "<工程.ioc2>" --format CMake --target "<完整构建目标>"
cube mx ide-project generate --project-path "<工程.ioc2>" --software-project "<软件工程名>" --build-target "<完整构建目标>" --destination "<新的输出目录>" --format CMake --source include-packs-from-local --diff keep_user
```

`include-packs-from-local` 用于所需包已安装的场景，其他 source 策略依据帮助及工程需求选择。`--diff keep_user` 是冲突处理策略；某些版本/导出策略下，`--force --diff keep_user` 仍会替换输出目录内手写的 `main.c`。

维护含用户修改的工程时，优先导出到新临时目录，以原工程为基准比较生成配置、初始化和构建输入，按文件归属合并必要变化。用户区按标记合并，独立用户文件保留原实现；不把临时目录整体覆盖原工程。仅对可丢弃目录，或已验证保留规则并保存用户内容的场景使用 force。

导出目录可能含 `user_modifiable` 等用户目录和只读版本基线文件。按映射与归属维护，不强行覆盖 SDK 基线；SDK 变化需另行核对依赖。

导出后检查状态、diagnostics 和实际源码路径，按[配置检查](validation.md)核对变化，并按[构建](../../stm32-build/SKILL.md)验证所选目标。CMake 格式本身不能证明软件工程、工具链和全部目标均已正确选择。

## 失败分支

| 现象 | 下一步 |
| --- | --- |
| 找不到工程或无法连接后端 | 核对绝对路径、host/port 及错误日志，确认已有会话；不盲目启动多个后端。 |
| 资源/参数不存在或为空 | 核对软件工程、组件启用与绑定、参数上游条件和工具版本；重新执行对应 list 查询。 |
| 保存后恢复旧值 | 核对实际文件、GUI/CLI 会话和约束求解，保存后独立重载同一参数。 |
| `No toolchain defined` | 重新列出构建目标，选择含软件工程名的完整目标；不能只设置格式。 |
| 缺 `outputPath` / 输出目录已存在 | 显式传 `--destination` 选择新目录，再审查合并；不为绕过提示直接覆盖用户工程。 |
| 模型正确但生成/构建未生效 | 检查导出目录、文件差异、真正参与构建的源文件和目标。 |

依据：[MX2 官方文档](https://dev.st.com/stm32cube-docs/stm32cubemx2)、当前 leaf 命令帮助及项目查询。
