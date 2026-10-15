# STM32DevSkills

面向通用 STM32 MCU 开发的 Agent Skills 技能包。主要通过 `cube` 使用 STM32Cube 工具，结合工程已有的生成器、构建系统和原生工具，完成工程创建、固件开发及板端诊断。

第一版 `0.1.0`，中文为主，采用 MIT 许可。支持标准技能目录和 Codex 插件两种安装形式，均安装完整七技能包。

## 能力

| 技能 | 用途 |
| --- | --- |
| [stm32-env](skills/stm32-env/SKILL.md) | 发现和恢复工具、Bundle、Pack、编译器及平台环境。 |
| [stm32-project](skills/stm32-project/SKILL.md) | 创建、识别和整理工程，选择实际生成和维护方式。 |
| [stm32-configure](skills/stm32-configure/SKILL.md) | 配置引脚、时钟、外设、DMA、中断及生成器软件选项。 |
| [stm32-firmware](skills/stm32-firmware/SKILL.md) | 实现功能、移植驱动、维护裸机和 FreeRTOS 应用。 |
| [stm32-build](skills/stm32-build/SKILL.md) | 构建固件、定位编译链接错误、检查产物与资源。 |
| [stm32-flash](skills/stm32-flash/SKILL.md) | 下载、校验、复位和启动指定目标固件。 |
| [stm32-debug](skills/stm32-debug/SKILL.md) | 获取运行证据，定位启动、异常、外设与 FreeRTOS 问题。 |

技能按任务组合。工具可用时不必重复准备环境；纯应用改动可以直接开发和构建；运行问题则从证据获取和诊断开始。

包内技能可以相互引用。工具定位与依赖集中在 `stm32-env`，工程识别集中在 `stm32-project`，配置保存、重载及生成集中在 `stm32-configure`，构建方法集中在 `stm32-build`。其他技能按当前任务读取这些资料，完成子步骤后继续原任务，无需每次加载全部技能。

文件链接相对其所在文档解析，实际工具命令在用户工程目录运行。标准目录安装须保留七个技能的名称及同级关系；插件安装由完整插件携带这些目录。

## 工程范围

- 面向 Windows、Linux、macOS，按主机和工具实际支持处理安装、路径及设备访问差异。
- 覆盖传统 CubeMX `.ioc`、CubeMX2 `.ioc2` 及无生成器配置的工程；根据当前芯片和 SDK 发现能力。
- 支持 HAL1、HAL2、LL；第一版运行模型为裸机与 FreeRTOS，FreeRTOS 包含原生 API、CMSIS-RTOS v1/v2。
- 未指定体系的新工程默认 CMake + Ninja + GCC、裸机；已有工程沿用原有构建体系和 API。
- 具体芯片、板卡和工具版本用于选择操作路径，不以示例型号作为支持白名单。一个工具的主机或芯片限制仍需按其官方资料确认。

## 前置工具

需要能读取工程并执行本地命令的兼容 Agent Skills 客户端。Codex 插件需要本地执行环境。

安装或使用 [STM32CubeCLI](https://dev.st.com/stm32cube-docs/stm32cubecli/latest/en/index.html)，通过 `cube --version` 和 `cube --list` 检查当前能力。工具 Bundle、MCU SDK/Pack 按任务需要准备，项目指定的版本优先。

传统 CubeMX 工程需要对应的独立 CubeMX 及兼容 Java。已有 Keil/IAR 等工程需要原构建工具及其必要使用条件。烧录和调试需要匹配的目标、探针或 bootloader 连接方式；固件开发和构建可以先在无板卡环境完成。

插件安装本身只提供指导，不自动安装工具链、下载 MCU 软件包或修改工程配置。

不知道工具装在哪里时，可从[三平台工具定位](skills/stm32-env/references/tool-location.md)开始；传统 CubeMX 的查找、Java 选择和最小查询见[脚本入口](skills/stm32-env/references/cubemx-runtime.md)。

## 安装方式一：标准技能目录

将 `skills/` 中全部七个 `stm32-*` 目录复制到客户端的同一个技能发现目录，保留 `references`、`agents` 和许可证。只复制单个技能会缺少跨技能资料。

Codex 可以安装到用户级或工程级 `.agents/skills`。以下在本仓库根目录执行，先检查所有同名冲突，再复制完整包：

PowerShell：

```powershell
$skillsDir = Join-Path $HOME '.agents/skills'
$sourceSkills = @(Get-ChildItem -LiteralPath './skills' -Directory |
    Where-Object { $_.Name -like 'stm32-*' })
if ($sourceSkills.Count -ne 7) { throw '来源不完整，应包含七个 STM32 技能。' }
$conflicts = @($sourceSkills | Where-Object {
    Test-Path -LiteralPath (Join-Path $skillsDir $_.Name)
})
if ($conflicts.Count) { throw '存在同名技能，请比较并处理旧版本后整包安装。' }
New-Item -ItemType Directory -Path $skillsDir -Force | Out-Null
foreach ($sourceSkill in $sourceSkills) {
    Copy-Item -LiteralPath $sourceSkill.FullName -Destination $skillsDir -Recurse -ErrorAction Stop
}
```

Linux/macOS shell：

```bash
(
  set -eu
  skill_names='stm32-env stm32-project stm32-configure stm32-firmware stm32-build stm32-flash stm32-debug'
  skills_dir="$HOME/.agents/skills"
  for name in $skill_names; do
    if [ ! -f "./skills/$name/SKILL.md" ] || [ -e "$skills_dir/$name" ] || [ -L "$skills_dir/$name" ]; then
      printf '%s\n' "来源缺失或目标存在：$name，请先处理后整包安装。" >&2
      exit 1
    fi
  done
  mkdir -p "$skills_dir"
  for name in $skill_names; do
    cp -R "./skills/$name" "$skills_dir/"
  done
)
```

工程级安装将目标改为目标工程的 `.agents/skills`。其他兼容客户端按其发现规则导入完整七目录，并保留包内文件访问能力。重新加载客户端，确认七个技能均可见；升级也按完整包处理，避免混用不同版本的共享资料。

## 安装方式二：Codex 插件

仓库包含 portable `plugin.json`、Codex 兼容清单和本地 marketplace；插件与 marketplace 标识均为 `stm32-dev-skills`。

在本仓库根目录执行：

```text
codex plugin marketplace add .
codex plugin list
codex plugin add stm32-dev-skills@stm32-dev-skills
```

这些命令适用于提供对应子命令的 Codex 版本，先用 `codex plugin --help` 核对。若客户端通过界面安装，注册 marketplace 后在插件目录选择该来源并安装 STM32DevSkills。重新加载客户端，确认七个技能可见。

选择一种安装方式，避免同一份技能被多次发现。仓库移动后更新 marketplace 来源；更新技能后按客户端的 marketplace 刷新/升级及插件更新方式加载新内容。

插件没有 MCP 服务、账号或后台运行组件。`cube` 和其他开发工具仍是本机的原生工具。

## 使用示例

支持自动发现技能的客户端可直接描述任务；也可显式选择技能。Codex 插件入口带命名空间，例如 `stm32-dev-skills:stm32-build`；标准技能目录中的名称为 `stm32-build`。显式调用时选择客户端列表中的对应入口。

### 从零创建工程

> 使用 stm32-project，为我指定的 STM32 芯片创建裸机工程，采用 CMake + Ninja + GCC，再实现指定引脚的 GPIO 输出并完成构建。

典型组合：工程 → 配置 → 固件 → 构建；需要上板时继续下载与运行验证。准确芯片、引脚和真实板卡时钟由任务输入或查询确定。

### 在已有工程上开发

> 使用 stm32-firmware，在当前工程增加串口 DMA 接收，沿用已有 HAL 和构建体系；如果是 FreeRTOS 工程，把接收数据交给现有任务处理，并检查重新生成后的代码保留。

典型组合：识别工程 → 配置或固件修改 → 构建 → 所需板端验证。

### 板端诊断

> 使用 stm32-debug，定位当前固件启动后没有串口输出的问题，先确认实际运行镜像，再检查初始化返回值、时钟和中断证据，修复后复查。

典型组合：获取证据 → 定位 → 修复 → 构建、下载及同条件复查。

## 结果与维护原则

生成器可表达的设置优先维护配置模型；应用代码放在已有用户文件或可保留区域。命令、实例和参数依据当前版本的帮助与工程查询，参考资料按需加载。

构建通过、烧录完成和功能验证通过分别说明。无硬件时继续完成可执行的源码、配置和构建工作，指出待上板事项。包内提供操作和检查方法，不附带历史实验记录。

## 资料与许可

- [STM32CubeCLI 官方文档](https://dev.st.com/stm32cube-docs/stm32cubecli/latest/en/index.html)
- [STM32CubeMX CLI](https://dev.st.com/stm32cube-docs/stm32cubemx/6.18.1/en/docs/markup/CubeMX_CLI.html)
- [Agent Skills 与 Codex 使用方式](https://learn.chatgpt.com/docs/build-skills)
- [插件打包说明](https://developers.openai.com/plugins/build/plugins)

本仓库自有内容使用 [MIT 许可](LICENSE)。STM32Cube 工具、SDK、Pack 及第三方组件按其自身许可证使用，本仓库不包含这些软件的二进制或软件包。
