# CubeMX2 工程路径

适用于当前 `cube mx` 提供的 MX2 命令及 `.ioc2` 工程。先确认目标设备在已选 SDK/Pack 和生成器版本中可用，不能根据另一个系列的示例推断支持。

## 创建和软件工程

```text
cube mx --help
cube mx project create-from-mcu --help
cube mx project create-from-mcu --cpn "<准确CPN>" --project-location "<新目录>" --project-name "<工程名>"
cube mx project get-sw-project-list --project-path "<工程.ioc2>"
```

板卡工程使用 `project create-from-board --help` 发现参数。设备、板卡、Pack 名称与版本通过当前 finder/pack-manager 或 SDK 元数据查询。创建可能下载所需依赖；先复用已有合法本地包，检查实际返回状态和文件。

如果还没有所需软件工程，使用 `project create-software --help`，再创建对应的软件工程。已有软件工程时不要重复创建。多核、安全域或多个软件工程需明确选择 `--software-project`。

## 保存与生成

```text
cube mx project save --project-path "<工程.ioc2>"
cube mx ide-project show-config --project-path "<工程.ioc2>"
cube mx ide-project build-target list --project-path "<工程.ioc2>"
cube mx ide-project list-format --help
cube mx ide-project generate --help
cube mx ide-project set-target --project-path "<工程.ioc2>" --format CMake --target "<完整构建目标>"
cube mx ide-project generate --project-path "<工程.ioc2>" --software-project "<软件工程名>" --build-target "<完整构建目标>" --destination "<输出目录>" --format CMake --source include-packs-from-local --diff keep_user
```

`include-packs-from-local` 适合所需包已经安装的情况；其他 source 策略依据可用选项和项目需求选择。`keep_user` 是模型的冲突处理策略，不能据此认定导出目录内的手写修改会保留。

已有输出目录可能要求 `--force` 才能再次导出；部分版本/导出策略下，即使同时指定 `--diff keep_user`，输出目录内修改过的 `main.c` 仍可能被替换。

维护含用户修改的工程时，优先导出到新的临时目录：保持原工程为合并基准，比较生成配置、初始化代码和构建输入，按文件归属合并必要变化，保留用户实现和调用链，然后构建。生成文件内有用户区时按标记逐段合并；不把临时目录直接覆盖整个工程。仅对可丢弃的导出目录，或已验证保留规则且已保存用户内容的场景使用 force。

导出工程可能包含 `user_modifiable` 等用户目录和只读的版本基线文件。依据工程映射和文件归属维护，避免把 SDK 基线当成普通生成源码强行覆盖；只更新本次配置实际影响的内容，SDK 变化另行核对依赖。

构建目标表示为 `[projectName][.buildType][+targetType]`。列表返回以点开头的后缀时，结合已查询的软件工程名组成完整目标；显式选择软件工程、目标和输出目录，避免空 projectName 导致无法确定工具链。没有所需构建类型时，查询 `build-target add --help` 后创建，再重新列出目标。

输出目录、目标、编译器和构建配置依据 `show-config`、生成状态和实际生成文件确定。生成 CMake 格式不会自动替用户选择正确的软件工程或全部目标。

## 重新加载与生命周期

`.ioc2` 是包含多个 store 的复合文本。优先使用项目 CLI 保存，不通过纯 JSON 序列化重写。需要检查磁盘持久化时，保存后关闭这个工程，再用查询命令按路径重新加载，回读目标值；只关闭当前任务拥有的工程。

CLI 可能启动无界面后端或连接现有 host/port。帮助查询不需要工程；实际操作必须确认项目路径和连接的后端。避免不必要地启动多个后端，也不要结束用户已有的 GUI 会话。

生成可能改变导出选项等模型状态，结束操作前保存需要保留的变化。关闭提示未保存时先核对变化并保存，不用 `--discard` 掩盖丢失设置的问题。

返回值若包含 `status: error` 或生成诊断，按失败处理，即使外层进程退出码为 0。确认 `.ioc2`、生成目录、实际目标与构建产物。

依据：[MX2 官方文档入口](https://dev.st.com/stm32cube-docs/stm32cubemx2)、当前 `cube mx … --help`。不同版本以实际帮助和工程查询为准。
