# 传统 CubeMX 的定位与脚本入口

处理 `.ioc` 工程且尚未建立可用脚本会话时读取。程序搜索统一使用[工具定位](tool-location.md)，本页负责候选辨认、Java 和会话启动。`cube mx` 的实际后端由解析结果和帮助确认，不能因名称含 mx 就把传统 `.ioc` 交给它。

## 选择入口与 Java

1. 找到独立 STM32CubeMX 安装目录。检查启动程序、相邻启动配置（例如 `STM32CubeMX.l4j.ini`）、数据库目录及配套 `jre`。只有安装包、卸载器或快捷方式不能算找到可操作的程序。
2. 从工程 `.ioc` 的 `MxCube.Version`、`MxDb.Version` 等实际字段取得版本线索，与安装信息核对。迁移提示需检查差异，避免无依据升级工程。
3. Linux/macOS 优先使用安装提供的脚本/应用内命令行入口及 `-q`。Windows 的传统安装可能使用带 Java 归档的 `STM32CubeMX.exe`，可通过兼容 Java 的 `-jar` 启动；不能推广到普通 exe。
4. Java 优先选该安装配套运行时；在安装目录中查找 `jre/bin/java` 或 `java.exe`，执行 `-version`。没有配套运行时时，核对当前 CubeMX 要求，再检查 `cube java -version` 或其他已安装 Java。
5. 阅读启动配置并按它传递必要的 `-D`、`--add-opens`、`--add-exports` 等 JVM 参数。不要把配置文件整段拼进 shell，也不要直接套用其他版本的全部 JVM 参数。

## 建立不改工程的最小会话

先写一个临时文本脚本，每行一条命令，UTF-8 编码：

```text
help
exit
```

在 PowerShell 中通过已经定位的配套 Java 调用；两处路径均替换为真实绝对路径：

```powershell
$mxProgram = '<STM32CubeMX.exe绝对路径>'
$javaProgram = '<配套java.exe绝对路径>'
$scriptPath = '<临时脚本绝对路径>'
$jvmArgs = @() # 按当前安装的启动配置填入必要参数
$mxArgs = $jvmArgs + @('-jar', $mxProgram, '-q', $scriptPath)
& $javaProgram @mxArgs
```

若安装配置要求模块访问参数，数组中的形态例如：

```powershell
$jvmArgs = @(
    '--add-opens=java.desktop/java.awt=ALL-UNNAMED',
    '--add-exports=java.desktop/sun.awt=ALL-UNNAMED'
)
```

这是参数编码示例，实际列表来自该安装。已确认兼容的 cube Java 可替代直接 Java；调用形式为 `& cube java @mxArgs`。Windows 信任库参数仅在该安装要求时添加。

Linux/macOS 使用已发现的安装入口：

```bash
mx_program="<安装提供的命令行启动入口>"
script_path="<临时脚本绝对路径>"
"$mx_program" -q "$script_path"
```

如果实际入口是 Java 归档，则使用该安装确认的 Java、JVM 参数及 `-jar` 方式。启动器内容和官方说明用于确定工作目录、归档和运行时；不要把 `.app` 目录本身当成可执行程序。

确认日志出现 CubeMX 的脚本帮助，并正常执行到 `exit`。仅打开 GUI、进程启动或退出码为 0，都不足以证明脚本会话已建立。

## 加载工程后的命令发现

最小会话成功后，将脚本改为以下查询模板；此阶段不加入 `config save` 或 `project generate`：

```text
config load <配置.ioc绝对路径>
help
get
exit
```

`get` 用于发现当前查询子命令，再使用返回的语法查询目标或本次涉及的配置；设备信息也可从加载日志与配置字段交叉核对。日志应显示工程成功加载及相关查询接口。模板中的尖括号必须替换，CubeMX 脚本内的路径语法按当前 CLI 处理；涉及空格而无法可靠解析时先用临时副本验证，不能靠拆分字符串猜测。

命令组本身可能在输出 Usage 后返回 `KO`，表示尚未提供完整子命令。取得语法后再查询实际值，例如帮助中存在的 `get mcu name` 和 `get mcu package`；完整查询必须返回对应数据。

只有配置加载及模块初始化完成后，才继续查 DMA、NVIC、时钟等上下文命令。帮助层级及配置操作见[传统 CubeMX 配置](../../stm32-configure/references/cubemx.md)；创建工程见[传统 CubeMX 工程](../../stm32-project/references/cubemx.md)。

## 启动失败的分支

| 现象 | 处理 |
| --- | --- |
| `Unable to access jarfile` / 路径不存在 | 检查绝对路径、参数边界和实际归档位置。 |
| `Invalid or corrupt jarfile` | 确认该文件确实是支持 `-jar` 的 CubeMX 入口；改用安装提供的启动器，不对任意 exe 继续重试。 |
| Java 版本、类版本或模块访问错误 | 对照配套 Java 和启动配置修正运行时/参数，再做最小会话。 |
| 加载失败或没有目标信息 | 核对 `.ioc`、工具/数据库版本和固件依赖，停止修改或生成。 |
| 只有 GUI / 脚本未执行 | 核对 `-q`、脚本绝对路径、入口是否转发参数，以及是否调用到另一安装。 |
| 命令 `KO`、unsupported 或无值 | 查询当前上下文帮助，按当前语法调整；不能把孤立 `OK` 当作目标值。 |

修正一个明确条件后重新做最小查询；错误没有新证据时停止重复启动，说明缺失的入口、运行时或版本条件。只结束自己启动的会话，不终止用户现有 GUI。

依据：[传统 CubeMX CLI](https://dev.st.com/stm32cube-docs/stm32cubemx/6.18.1/en/docs/markup/CubeMX_CLI.html)、当前安装启动配置与会话帮助。
