# CubeMX2 配置

使用当前 `cube mx` 帮助确认子命令。实际调用传入 `.ioc2` 路径；存在多个软件工程时指定 `--software-project`。路径、实例、参数路径和取值由当前项目发现。

## 操作映射

| 对象 | 发现与查询 | 修改方法 |
| --- | --- | --- |
| 引脚 | `pinout all-pins`、`pin`、`assigned-pins`。 | `assign-signal`、`unassign-signal`、标签及分组命令。 |
| 外设 | `peripherals list`、`view`、`description`。 | `enable`/`disable`，再处理组件参数。 |
| 参数 | `sw-config list-configurations`、`list-parameters`、`get-parameter-possible-values`、`get-parameter-value`。 | `set-parameter-value` 或当前版本的批量设置命令。 |
| 时钟 | `clock get-clocks`、`get-clock-items`、`list-possible-values`。 | `set-value` 等模型接口，再查询 `get-clock-status`。 |
| DMA/IRQ | 当前 `dma`、`nvic`、`exti` 命令组帮助和资源查询。 | 先发现资源关联、合法选项，再设置并回读。 |
| 软件组件 | `composer`、`middleware` 的帮助、可用组件及工程实例。 | 选择匹配 SDK 的组件，然后配置对应实例。 |

例如已发现目标信号和资源后：

```text
cube mx pinout assign-signal --project-path "<工程.ioc2>" --pin "<引脚>" --signal "<信号>"
cube mx sw-config set-parameter-value --project-path "<工程.ioc2>" --resource-name "<资源>" --parameter-path "<查询到的路径>" --value "<合法值>"
cube mx project save --project-path "<工程.ioc2>"
```

每条 leaf 命令先看 `--help`，不要把其他命令的 flag 拼到当前命令中。参数数组、对象或布尔值依帮助规定编码，并注意 PowerShell 与 POSIX shell 引号差异；需要多个参数时优先参数数组传参。

分配 GPIO 信号后，继续查询该引脚的软件参数，明确设置输入/输出、电气类型和上下拉并回读。生成器可能规范化标签为宏名，检查生成后的真实名称及引用，而不是按输入标签的大小写直接拼接标识符。

## 组件与时钟一致性

FreeRTOS 及其 API 适配层需要当前 SDK 实际提供。确认软件工程绑定资源、时基、内核、heap 实现和异常处理链。依赖解析通常由模型自动触发；不要手动调用标注为内部或不建议手动使用的 resolver。

时钟求解可能修改关联节点，使用 solver 后回读受影响频率、约束和错误。不能把节点 ID、最大频率或外部晶振值当成跨系列常量。

## 持久化与生成

保存后关闭当前任务的工程，并通过查询重新加载、回读。保留当前 GUI 的未保存编辑；不要用旧会话覆盖磁盘上新设置。`.ioc2` 不按纯 JSON 改写。

查询 `ide-project show-config` 与 `generate --help`，使用工程实际输出格式和软件目标。生成时保留用户文件策略，检查返回 `status`、diagnostics 和文件差异，再构建。`--diff keep_user` 是冲突处理策略，不能代替代码差异检查。

已有导出目录包含手写内容时，优先导出到新临时目录，再按文件归属合并生成变化。某些导出策略中 `--force --diff keep_user` 仍可能覆盖该目录内的用户修改；保留原工程、用户区和构建输入，合并后检查配置生效和用户行为。

导出报缺少工具链时先查询 `ide-project build-target list`，选择包含软件工程名的完整目标；不要只设置格式。缺少 `outputPath` 时显式传入 `--destination`。多个软件工程通过 `--software-project` 明确选择。

依据：[MX2 官方文档](https://dev.st.com/stm32cube-docs/stm32cubemx2)、当前 leaf 命令帮助及项目查询。
