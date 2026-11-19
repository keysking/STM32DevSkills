# STM32DevSkills

STM32 开发入口技能。默认全局安装一次，Agent 根据用户型号、当前工程或 ST-Link 判断芯片系列，在当前工程安装相应系列技能，然后继续原来的开发任务。

第一版提供一个 [`stm32-dev`](skills/stm32-dev/SKILL.md) 技能及参考资料，由 Agent 调用原生命令；不附带辅助脚本、工具二进制或 SDK。内容中文，MIT 许可，面向具备本地文件读取与命令执行能力的 Agent Skills 客户端。

## 使用流程

1. 用户指定型号 → 当前工程配置 → ST-Link 探测 → 无法确定时询问。
2. F103 等 F1 型号选择 [STM32F1DevSkills](https://github.com/keysking/STM32F1DevSkills)，C532 等 C5 型号选择 [STM32C5DevSkills](https://github.com/keysking/STM32C5DevSkills)。暂未提供其他系列包。
3. 中文用户优先 Gitee，其他语言默认 GitHub；首选不可访问时尝试另一允许的来源。
4. 系列包默认完整安装到工程级技能目录。已有完整包直接复用，不自动更新；已有残缺目录先比较，不覆盖本地内容。
5. 读取当前任务需要的系列技能继续执行。具体型号、封装及接线信息按任务需要进一步确认。

例如：“使用 F103 新建工程”“给这个工程增加串口”“不知道板子型号，先识别再开发”。用户或工程已明确目标时无需连接硬件。ST-Link 识别只读取器件信息，不执行烧录、擦除或复位恢复。

## 安装入口：标准技能目录

从 [STM32DevSkills 仓库](https://github.com/keysking/STM32DevSkills)取得完整内容后，在仓库根目录执行。默认安装到用户级 `.agents/skills`；使用其他客户端时按其实际规则替换目标目录。下面只安装入口；F1、C5 等系列包由入口安装到各自工程。

PowerShell：

```powershell
$sourceSkill = Join-Path (Get-Location) 'skills/stm32-dev'
$skillsDir = Join-Path $HOME '.agents/skills'
if (-not (Test-Path -LiteralPath (Join-Path $sourceSkill 'SKILL.md') -PathType Leaf)) { throw '请在完整入口仓库根目录执行。' }
if (Get-Item -LiteralPath (Join-Path $skillsDir 'stm32-dev') -Force -ErrorAction SilentlyContinue) { throw '入口已存在；先检查现有版本，不覆盖。' }
New-Item -ItemType Directory -Path $skillsDir -Force -ErrorAction Stop | Out-Null
Copy-Item -LiteralPath $sourceSkill -Destination $skillsDir -Recurse -Force -ErrorAction Stop
```

Bash：

```bash
(
  set -eu
  source_skill='./skills/stm32-dev'
  skills_dir="$HOME/.agents/skills"
  test -f "$source_skill/SKILL.md" || { echo '请在完整入口仓库根目录执行' >&2; exit 1; }
  if [ -e "$skills_dir/stm32-dev" ] || [ -L "$skills_dir/stm32-dev" ]; then
    echo '入口已存在；先检查现有版本，不覆盖' >&2; exit 1
  fi
  mkdir -p "$skills_dir"
  cp -R "$source_skill" "$skills_dir/"
)
```

完整保留 `references`、`agents` 和 `LICENSE`。按客户端规则刷新或开始新会话，核对发现 `stm32-dev`。已安装系列包时可以直接使用系列技能。

## 安装入口：Codex 插件

与标准技能目录安装二选一，避免重复发现。插件标识 `stm32-dev-skills`，显示名 `STM32DevSkills`，版本 `0.1.0`；包含相同的一个入口技能，要求本地执行环境。

在本仓库根目录，用提供相应命令的 Codex CLI 注册本地 marketplace 并安装：

```text
codex plugin --help
codex plugin marketplace add .
codex plugin add stm32-dev-skills@stm32-dev-skills
codex plugin list
```

核对实际插件状态和发现的入口；调用名可能带 `stm32-dev-skills:` 命名空间。此方式仅安装全局入口，系列包仍使用工程级目录。这是本地 marketplace 安装，不代表已上架公共插件目录。

## 参考资料

- [工程型号识别](skills/stm32-dev/references/identify.md)：读取实际选型，处理多工程及冲突。
- [工具发现与 ST-Link](skills/stm32-dev/references/probe.md)：三平台定位、原生 Programmer 备用路径和识别边界。
- [下载与工程级安装](skills/stm32-dev/references/install.md)：来源回退、完整包复制、已有包复用及继续任务。

只识别已明确型号和安装技能时无需 Cube。只有需要硬件探测时才查找 Cube/CubeProgrammer；工具或连接不可用时询问型号。具体开发能力与工具链范围以选定系列包为准。
