# Day2 Debug Log: KamaClaude 环境迁移、WSL2 安装与中转 API 配置连环踩坑实录



## 🧱 Phase 0：WSL2 底层环境与 Ubuntu 分发版修复

在新电脑重新搭建 Linux 开发环境时，由于底层虚拟化配置及系统迁移，首先在 WSL2 安装阶段遭遇了“开门黑”。



### 1. 现象与报错

* **报错一**：启动 WSL 或安装 Ubuntu 时提示 `WSL2 requires error: 0x80370102` 或 `The virtual machine mechanism is blocked...`。

* **现象二**：打开新电脑的 WSL 发现之前旧电脑的 Ubuntu 分发版“凭空消失”，终端提示找不到默认分发版。



### 2. 原因分析

* **硬件虚拟化未开启**：新电脑的 BIOS/UEFI 设置中，CPU 的虚拟化开关（Intel VT-x 或 AMD-V）默认处于关闭状态，导致 Windows 的 Hyper-V 虚拟化平台无法为 WSL2 提供底层支持。

* **分发版未注册/路径变更**：换电脑或重装系统后，虽然 D 盘的项目文件还在，但新系统的 WSL 注册表中并没有旧 Ubuntu 的信息，系统误以为是一个空白环境。



### 3. 解决方案

* **解法一（主板打底）**：重启电脑狂按 `F2` / `Del` 进入 BIOS，找到 `Advanced` -> `CPU Configuration`，将 `Intel Virtualization Technology` 或 `SVM Mode` 修改为 **Enabled**。保存重启进入 Windows 后，进入“启用或关闭 Windows 功能”，勾选“虚拟机平台”与“Windows 虚拟机监控程序平台”。

* **解法二（分发版重置与拉取）**：确保 WSL2 核心组件更新后，重新通过应用商店或命令行安装/导入 Ubuntu 发行版，并使用 `wsl --set-default Ubuntu` 锁定默认环境，使其重回正轨。



---



## 📌 Phase 1：KamaClaude 运行期故障排查

底层环境打通后，运行客户端任务 `uv run kama run` 遭遇秒挂（`failed`）。为定位真凶，我们将 `kama-core` 切换至前台运行，展开了连环排查。



---



## 🛑 故障一：大模型请求报 403 Forbidden

### 1. 现象与日志

前台启动 `kama-core` 后，执行任务时核心服务疯狂吐出以下报错：

```text

level=INFO ts=... source=httpx msg="HTTP Request: POST [https://api.anthropic.com/v1/messages](https://api.anthropic.com/v1/messages) "HTTP/1.1 403 Forbidden""

anthropic.PermissionDeniedError: Error code: 403 - {'error': {'type': 'forbidden', 'message': 'Request not allowed'}}



```



### 2. 原因分析



* **网络走偏**：由于底层使用的是 Anthropic 客户端组件，代码默认将请求发送到了官方的 `api.anthropic.com`。

* **IP 拦截**：在关闭本地 VPN 的情况下，请求带着国内 IP 直连官方服务器，被 Anthropic 防火墙无情拒之门外（403 拒绝服务）。



### 3. 解决方案



在项目根目录的 `.env` 文件中，为中转/免翻墙模型显式配置 **Base URL**。

打开 `.env` 补充或修改：



```ini

ANTHROPIC_API_KEY="your_api_key_here"

ANTHROPIC_BASE_URL="https://你的中转服务商域名/v1" 



```



---



## 🛑 故障二：VSCode 找不到 `.env` 文件 / 打开文件夹断开 WSL



### 1. 现象与误区



* 误区一：在 Linux 终端中直接输入 `ls` 找不到 `.env`。

* 误区二：在 VSCode 的 `Remote Explorer`（远程资源管理器）中试图找代码文件，导致视图空白。

* 误区三：点击左侧的 `Open Folder` 时如果选了本地 Windows 路径，会导致 VSCode 自动退出 WSL Ubuntu 环境。



### 2. 原因分析



* Linux 系统中以 `.` 开头的文件均为**隐藏文件**，默认的 `ls` 不会将其显示。

* `Remote Explorer` 用于管理远程连接，而非管理文件；在 WSL 环境中必须通过 Linux 的绝对路径打开项目。



### 3. 解决方案



* **查看隐藏文件**：终端改用 `ls -a` 即可看到 `.env`。

* **WSL 内部一键拉起 VSCode**：在已连上 WSL 的终端内切换到项目目录，直接运行：

```bash

code .



```





VSCode 会自动弹出专属新窗口，不仅完美保持在 WSL 环境中，左侧文件树也会全量展开（含隐藏文件）。



---



## 🛑 故障三：连环触发 `JSONDecodeError` 与 `ConnectionResetError`



### 1. 现象与日志



配置好 Base URL 重新启动 `kama-core` 后，只要一跑任务，核心服务就会瞬间刷屏爆出：



```text

json.decoder.JSONDecodeError: Expecting value: line 1 column 1 (char 0)

ConnectionResetError: Connection lost

BrokenPipeError: [Errno 32] Broken pipe



```



### 2. 原因分析



* **暗号对不上（环境变量隔离）**：Linux 的环境变量具有强窗口隔离性。我们在窗口 A 修改了 `.env` 并通过 `export` 加载了变量，但去窗口 B 运行客户端 `uv run kama run` 时，窗口 B 是个纯净环境，完全不知道 Base URL 是什么。

* **客户端发空包**：客户端因为缺少必要的参数串，给核心服务发了一段空字符（`char 0`）。服务端在尝试用 `json.loads(line)` 解析时直接崩溃并断开连接。



### 3. 解决方案



让客户端和服务端在启动时**同时吃进 `.env` 配置**。



* 服务端（窗口 A）：`export $(cat .env | xargs) && uv run kama-core`

* 客户端（窗口 B）：`export $(cat .env | xargs) && uv run kama run --goal "..."`




---



## 🛑 故障四：`.env` 中文注释引发的 `not a valid identifier` 惨案



### 1. 现象与日志



当在前面使用粗暴的快捷加载命令时，Linux 终端开始疯狂刷屏：



```text

bash: export: `#': not a valid identifier

bash: export: `环境配置模板': not a valid identifier

bash: export: `用法：cp': not a valid identifier



```



### 2. 原因分析



原先使用的 `cat .env | xargs` 会不分青红皂白地把文件里的所有空格和换行打碎塞给 `export`。当它遇到 `.env` 内部自带的**中文注释**（如 `# 环境配置模板`）时，会将中文词组和特殊符号也当成变量名去加载，直接触发 Linux 变量命名非法报错，进而导致**真正的 Key 和 Base URL 压根没加载成功**。



### 3. 解决方案



**更换更聪明的标准 Linux 过滤加载命令**，利用 `grep -v '^#'` 自动剔除所有以 `#` 开头的注释行：



```bash

export $(grep -v '^#' .env | xargs) && uv run kama-core



```



---

