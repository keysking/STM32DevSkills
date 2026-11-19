# 下载、工程级安装与任务衔接

已从[入口映射表](../SKILL.md)确定系列后使用。入口自身默认全局安装；本文件只安装对应系列包，不把所有系列装到全局。

## 安装落点与已有内容

确定用户本次任务的工程根目录。新建工程时可创建用户指定的目标目录，再安装技能；不要把下载临时目录或包含多个工程的上级目录当作目标工程。

按当前客户端的实际技能发现规则选择工程级目录。支持 `.agents/skills` 的客户端采用 `<工程>/.agents/skills/`；其他客户端使用其确认支持的工程级目录，不能假定所有客户端使用同一路径。下面命令以 `.agents/skills` 为例，必要时替换目的目录变量。

系列包包含 `env`、`project`、`configure`、`firmware`、`build`、`flash`、`debug` 七个同级技能，前缀为 `stm32f1` 或 `stm32c5`：

- 已有七个对应目录、非空 `SKILL.md`、`LICENSE`、`agents/openai.yaml` 和被引用的参考资料，且包内相对链接有效时，直接读取使用，不下载、不访问远端检查更新。完整不等于最新。
- 同名目录部分存在或内容残缺时，列出缺失项和本地修改；先比较再确定修复方式，不把新包直接覆盖到旧包。不删除其他系列或其他技能。
- 没有对应目录时，执行以下下载与安装。仅在用户明确要求更新时处理版本更新；更新前保留本地修改，并按完整包比较。

## 下载选定仓库

从入口表选取完整仓库 URL 并加 `.git`。中文先 Gitee，其他语言先 GitHub；用户明确限定来源时只用允许的来源。Git 不可用时可用客户端现有下载能力获取该仓库的完整归档，检查解压后的 `skills/` 结构；不要为技能安装引入包管理器或下载器依赖。

每次下载使用一个新的系统临时目录，执行超时可设为 120 秒。首选源网络失败时，使用另一个来源和新的临时目录重新执行，最多尝试两个允许的来源。两个都失败时报告原因并停止安装，不继续假装已经获得系列指导。已有授权或用户指定的本地完整副本也可作为来源。

PowerShell，以下 URL 是中文 F1 的例子；C5 使用入口表中的 C5 URL：

```powershell
$repoUrl = 'https://gitee.com/keysking/STM32F1DevSkills.git'
$downloadDir = Join-Path ([IO.Path]::GetTempPath()) ('stm32-skills-' + [guid]::NewGuid().ToString('N'))
$sourceRoot = Join-Path $downloadDir 'package'
New-Item -ItemType Directory -Path $downloadDir -ErrorAction Stop | Out-Null
git -c credential.interactive=never -c http.lowSpeedLimit=1 -c http.lowSpeedTime=20 clone --depth 1 -- $repoUrl $sourceRoot
if ($LASTEXITCODE -ne 0) { throw '下载失败；按来源规则选择备用源，并使用新的临时目录重试。' }
git -C $sourceRoot rev-parse HEAD
if ($LASTEXITCODE -ne 0) { throw '下载目录不是完整 Git 工作副本。' }
```

Bash：

```bash
repo_url='https://gitee.com/keysking/STM32F1DevSkills.git'
download_dir=$(mktemp -d "${TMPDIR:-/tmp}/stm32-skills.XXXXXXXX") || exit 1
source_root="$download_dir/package"
GIT_TERMINAL_PROMPT=0 git -c credential.interactive=never -c http.lowSpeedLimit=1 -c http.lowSpeedTime=20 clone --depth 1 -- "$repo_url" "$source_root" || exit 1
git -C "$source_root" rev-parse HEAD || exit 1
```

保留成功下载的实际路径与提交号，供后续复制和结果说明使用。来源 URL、临时目录、提交号不是技能安装目录。临时下载目录由本次操作创建；如需清理，先核对其绝对路径，不能清理用户工程或复用的本地副本。

## 检查并整包复制

先确认来源 `README.md` 的包身份与所选系列一致，并读取安装说明。七个技能的 `name` 与目录名应相同，包内引用应能在同一 `skills` 目录下解析。文件为空或参考链接缺失时，不安装不完整来源。

下面命令在复制前检查全部七个来源及全部同名冲突；只适用于初次安装。三个变量必须替换为已确定的来源、目标工程和前缀。复制完整目录，保留参考资料、元数据和许可证。复制中途失败会留下不完整安装，应先检查实际文件再恢复，不无条件重跑覆盖。

PowerShell：

```powershell
$sourceRoot = 'C:/actual/download/package'
$projectRoot = 'D:/firmware/Control'
$prefix = 'stm32f1' # C5 使用 stm32c5
if ($prefix -notin @('stm32f1', 'stm32c5')) { throw '未支持的系列前缀。' }
if (-not (Test-Path -LiteralPath $projectRoot -PathType Container)) { throw '目标工程目录不存在。' }
$skillsDir = Join-Path $projectRoot '.agents/skills'
$names = @('env', 'project', 'configure', 'firmware', 'build', 'flash', 'debug') | ForEach-Object { "$prefix-$_" }
$actualNames = @(Get-ChildItem -LiteralPath (Join-Path $sourceRoot 'skills') -Directory -ErrorAction Stop | Select-Object -ExpandProperty Name)
if (@(Compare-Object $names $actualNames).Count) { throw '来源技能集合与系列包不一致。' }
foreach ($name in $names) {
    $from = Join-Path $sourceRoot "skills/$name"
    foreach ($required in @('SKILL.md', 'LICENSE', 'agents/openai.yaml')) {
        $file = Get-Item -LiteralPath (Join-Path $from $required) -ErrorAction Stop
        if ($file.PSIsContainer -or $file.Length -eq 0) { throw "来源文件无效：$name/$required" }
    }
    if (-not (Test-Path -LiteralPath (Join-Path $from 'references') -PathType Container)) { throw "缺少参考资料：$name" }
    if (Get-Item -LiteralPath (Join-Path $skillsDir $name) -Force -ErrorAction SilentlyContinue) { throw "已存在 $name；先复用或比较，不覆盖。" }
}
New-Item -ItemType Directory -Path $skillsDir -Force -ErrorAction Stop | Out-Null
foreach ($name in $names) {
    Copy-Item -LiteralPath (Join-Path $sourceRoot "skills/$name") -Destination $skillsDir -Recurse -Force -ErrorAction Stop
}
foreach ($name in $names) {
    $from = (Resolve-Path -LiteralPath (Join-Path $sourceRoot "skills/$name")).Path
    foreach ($file in Get-ChildItem -LiteralPath $from -Recurse -File -Force) {
        $relative = $file.FullName.Substring($from.Length + 1)
        $installed = Join-Path (Join-Path $skillsDir $name) $relative
        if ((Get-FileHash -LiteralPath $file.FullName).Hash -ne (Get-FileHash -LiteralPath $installed -ErrorAction Stop).Hash) { throw "复制不完整：$installed" }
    }
}
Write-Output "已安装并核对七个 $prefix 技能：$skillsDir"
```

Bash（括号把错误退出限定在这段安装中）：

```bash
(
  set -eu
  source_root='/actual/download/package'
  project_root='/work/Control'
  prefix='stm32f1' # C5 使用 stm32c5
  case "$prefix" in stm32f1|stm32c5) ;; *) echo '未支持的系列前缀' >&2; exit 1 ;; esac
  test -d "$project_root" || { echo '目标工程目录不存在' >&2; exit 1; }
  skills_dir="$project_root/.agents/skills"
  suffixes=(env project configure firmware build flash debug)
  source_dirs=("$source_root"/skills/*)
  test "${#source_dirs[@]}" -eq 7 || { echo '来源技能集合不完整' >&2; exit 1; }
  for suffix in "${suffixes[@]}"; do
    name="$prefix-$suffix"
    from="$source_root/skills/$name"
    for required in SKILL.md LICENSE agents/openai.yaml; do
      test -f "$from/$required" && test -s "$from/$required" || { echo "来源文件无效：$name/$required" >&2; exit 1; }
    done
    test -d "$from/references" || { echo "缺少参考资料：$name" >&2; exit 1; }
    if [ -e "$skills_dir/$name" ] || [ -L "$skills_dir/$name" ]; then
      echo "已存在 $name；先复用或比较，不覆盖" >&2; exit 1
    fi
  done
  mkdir -p "$skills_dir"
  for suffix in "${suffixes[@]}"; do
    cp -R "$source_root/skills/$prefix-$suffix" "$skills_dir/"
  done
  for suffix in "${suffixes[@]}"; do
    diff -qr "$source_root/skills/$prefix-$suffix" "$skills_dir/$prefix-$suffix"
  done
  printf '已安装并核对七个 %s 技能：%s\n' "$prefix" "$skills_dir"
)
```

## 安装完成并继续

核对七个目录、实际入口和参考链接；逐文件比对只能证明复制一致，不能替代对来源资料完整性的检查。不复制仓库的 `.git`，不自动注册系列包为全局插件，也不修改项目的 Git 忽略规则。

简短说明识别依据、来源与安装目录，随后按[入口的任务映射](../SKILL.md)读取安装后的相应 `SKILL.md` 并继续。若仅有 ST-Link 候选器件组，应同时说明“系列已确定，准确型号未确定”，不将选包成功当成型号确认。

安装只是准备步骤，不能宣称原来的构建、烧录或开发任务已经完成。保留原任务的参数与限制；新建工程必须在创建目标相关配置或生成工程前确定准确芯片型号。用户未提供且工程中也没有时，询问用户并等待；其他任务在确实需要准确型号时再询问。不要为安装技能提前要求完整料号，也不要用探测结果或示例值代填。
