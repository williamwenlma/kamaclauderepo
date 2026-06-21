# Day 1 问题日志：KamaClaude 项目环境搭建与故障排查

## 概述
今天在配置 KamaClaude 项目开发环境时，遇到了一系列从基础命令使用到环境变量配置、代码兼容性问题。本文档记录了完整的问题现象、排查过程和最终解决方案。

---

## 问题 1：Windows 环境下使用 Linux 命令报错

### 现象
在 Windows CMD 中执行 `ls` 命令查看目录文件时出现报错：
```
'ls' 不是内部或外部命令，也不是可运行的程序或批处理文件。
```

### 原因分析
- `ls` 是 Linux/Unix 系统的命令
- Windows 命令提示符（CMD）默认不支持 Linux 命令
- Windows 对应的目录查看命令是 `dir`

### 解决方案
1. **使用 Windows 原生命令**：
   ```cmd
   dir
   ```

2. **切换到 PowerShell**：
   ```cmd
   powershell
   # 然后在 PowerShell 中可以使用 ls（它是 Get-ChildItem 的别名）
   ```

3. **安装 Git Bash 并在 VSCode 中使用**（最终采用）：
   - 下载安装 Git for Windows
   - 在 VSCode 中打开终端（`` Ctrl + ` ``）
   - 点击终端下拉菜单选择 "Git Bash"
   - 后续所有命令均在 VSCode 的 Git Bash 终端中执行

---

## 问题 2：安装 uv 时 `sh` 命令找不到

### 现象
在 Windows CMD 中执行安装命令时出现报错：
```cmd
curl -LsSf https://astral.sh/uv/install.sh | sh
'sh' 不是内部或外部命令，也不是可运行的程序或批处理文件。
```

### 原因分析
- `sh` 是 Unix shell 解释器，CMD 中不包含
- Windows 环境下需要使用专用的安装方式

### 解决方案
1. **使用 PowerShell 安装**：
   ```powershell
   powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
   ```

2. **在 VSCode 的 Git Bash 终端中安装**（最终采用）：
   - 安装 Git for Windows 后，Git Bash 自带 `sh` 解释器
   - 在 VSCode 中打开 Git Bash 终端
   - 执行原安装命令即可正常安装

---

## 问题 3：uv 安装后无法使用 `uv self update`

### 现象
执行 `uv self update` 时出现报错：
```
error: Self-update is only available for uv binaries installed via the standalone installation scripts.
```

### 原因分析
- `uv` 可能是通过 pip 或其他包管理器安装的
- 独立安装脚本安装的版本才支持 `self update`

### 解决方案
- 不需要强制更新，当前版本 `0.11.23` 已是最新
- 如需更新，使用原安装方式重新安装

---

## 问题 4：VSCode 中打开 Git Bash 终端

### 背景
在解决完问题 1 和问题 2 后，为了提高开发效率，决定统一在 VSCode 中使用 Git Bash 作为终端环境。

### 解决方案
1. **直接打开**：
   - 快捷键 `` Ctrl + ` ``
   - 点击终端下拉菜单选择 "Git Bash"

2. **设置为默认终端**：
   - 打开设置（`Ctrl + ,`）
   - 搜索 `terminal default profile`
   - 选择 "Git Bash"

3. **优势**：
   - 支持 Linux 命令（`ls`、`sh` 等）
   - 集成在 VSCode 中，无需切换窗口
   - 后续所有操作均在统一环境中进行

---

## 问题 5：Vim 编辑器使用问题

### 现象
- 进入 Vim 后不知道如何退出
- 输入 `:q!` 没有反应

### 原因分析
- 处于插入模式（`-- INSERT --`），输入的 `:q!` 被当作文本内容写入文件
- 需要先退出插入模式才能执行命令

### 解决方案
**Vim 基本操作流程**：
1. **进入插入模式**：按 `i`、`a`、`o` 等键
2. **退出插入模式**：按 `Esc` 键
3. **保存并退出**：`:wq` + `Enter`
4. **不保存强制退出**：`:q!` + `Enter`

**替代方案**（推荐新手）：
```bash
code .env   # 使用 VSCode 编辑
notepad .env  # 使用记事本编辑
```

---

## 问题 6：ANTHROPIC_API_KEY 环境变量未设置

### 现象描述
在 Git Bash 中通过 `vim .env` 编辑并设置了 API Key，但运行 `uv run kama-core` 时仍触发以下报错：
```text
level=INFO ts=2026-06-21T16:10:33 source=kama_claude.core.app msg="permission manager: timeout_s=60.0 persistent=0 entries"
ANTHROPIC_API_KEY not set
```

### 原因分析
1. **未加载到全局环境**：`.env` 只是一个纯文本文件，直接运行 `uv run` 时，当前终端窗口（Git Bash）并没有把文件内的变量读取到系统的临时环境变量中。
2. **配置文件被注释（主要原因）**：经检查 `.env` 文件发现，对应的变量行前带有 `#` 号（例如：`# ANTHROPIC_API_KEY=...`）。在 `.env` 规范中，`#` 代表注释，导致该行被解析器直接忽略。
3. **未替换占位符**：变量值仍为模板自带的中文提示"我的api key"，而非实际申请的官方密钥。
4. **`uv run --env-file` 加载失败**：执行 `uv run --env-file .env kama-core` 仍然报错，进一步排查发现根本原因仍是 `.env` 文件中的 API Key 被注释，导致 `--env-file` 参数加载后变量依然为空。

### 解决方案
1. **取消注释并修改**：使用编辑器打开 `.env`，去掉 `ANTHROPIC_API_KEY` 前面的 `#` 号，并将等号后方替换为以 `sk-ant-` 开头的真实密钥（**注意：等号两边不能有空格**）。

   ```bash
   # ── LLM（S1 阶段启用）────────────────────────────────────────
   ANTHROPIC_API_KEY=sk-ant-xxxxxx...
   KAMA_LLM_DEFAULT_MODEL=deepseek-v4-flash
   KAMA_MAX_STEPS=20
   ```

2. **验证修改**：
   ```bash
   cat .env | grep ANTHROPIC_API_KEY
   # 应输出：ANTHROPIC_API_KEY=sk-ant-xxxxxx...
   ```

3. **（可选）终端临时生效**：若想在当前终端快速测试，可直接在 Git Bash 中使用 `export` 注入变量：
   ```bash
   export ANTHROPIC_API_KEY="sk-ant-xxxxxx..."
   uv run kama-core
   ```

4. **使用正确的启动命令**：
   ```bash
   uv run --env-file .env kama-core
   ```

---

## 问题 7：Windows 异步信号机制不兼容（NotImplementedError）

### 现象描述
解决环境变量问题后重新运行 `uv run kama-core`，程序成功读取配置并开始监听 `127.0.0.1:7437`，但随即抛出 `NotImplementedError` 异常崩溃：

```text
File "D:\KamaClaudeProject\KamaClaude\src\kama_claude\core\app.py", line 278, in run
loop.add_signal_handler(signal.SIGINT, shutdown.set)
File "C:\Users\13728\AppData\Roaming\uv\python\cpython-3.12-windows-x86_64-none\Lib\asyncio\events.py", line 582, in add_signal_handler
raise NotImplementedError
NotImplementedError
```

### 原因分析
- 报错根源位于项目源码 `src/kama_claude/core/app.py` 的第 278 行。
- 代码中使用了 `loop.add_signal_handler()` 来监听系统中断信号（如 `Ctrl+C` 的 `SIGINT`），用于优雅退出。
- **系统差异**：该方法依赖类 Unix 系统的底层信号机制。Windows 系统的 Python asyncio 事件循环（ProactorEventLoop）没有实现此方法，直接调用必定触发 `NotImplementedError`。

### 解决方案（计划采用：WSL 2 环境）
由于该项目存在明显的 Linux/macOS 平台侧重，为避免后续遇到更多 Windows 兼容性"暗坑"，计划采用 **本机安装 Linux 系统 + VS Code WSL 终端** 的方式搭建开发环境：

1. **安装 WSL 2**（以管理员身份运行 PowerShell）：
   ```powershell
   wsl --install -d Ubuntu
   ```

2. **在 VS Code 中连接 WSL**：
   - 安装 Remote - WSL 插件
   - 点击左下角绿色图标 → "Connect to WSL"
   - 在 VS Code 的 WSL 终端中执行后续操作

3. **在 WSL 内重构环境**：
   - 在 Linux 终端中安装 Linux 版的 uv 包管理器：
     ```bash
     curl -LsSf https://astral.sh/uv/install.sh | sh
     ```
   - 克隆项目代码到 Linux 家目录（避免跨文件系统读写导致性能下降）

4. **配置并运行**：
   - 依照问题 6 的标准配置好 WSL 环境下的 `.env` 文件
   - 在 VS Code 的 WSL 终端中运行：
     ```bash
     uv run kama-core
     ```
   - **预期结果**：信号监听在 Linux 原生环境下完美兼容，Daemon 守护进程顺利启动，Windows 本机亦可通过 `127.0.0.1:7437` 正常跨架构通信

---

## 核心问题总结

| 问题 | 根本原因 | 解决方案 |
|------|---------|---------|
| `ls` 命令找不到 | Windows CMD 不支持 Linux 命令 | 安装 Git Bash，在 VSCode 中使用 |
| `sh` 命令找不到 | Windows CMD 不支持 sh | 使用 Git Bash 终端安装 |
| `uv self update` 报错 | 非独立安装方式 | 忽略，当前版本已最新 |
| Vim 无法退出 | 处于插入模式 | 按 `Esc` 退出插入模式，再输入命令 |
| `ANTHROPIC_API_KEY not set` | `.env` 中 API Key 被注释 | 删除 `#` 注释符，使用 `--env-file` 加载 |
| `uv run --env-file` 不生效 | `.env` 中 API Key 被注释 | 本质同上一问题，解除注释即可 |
| `NotImplementedError` | Windows 不支持 `add_signal_handler` | 计划迁移到 WSL 2 环境运行 |

---

## 建议的最佳实践

1. **使用 VSCode + Git Bash 作为开发环境**：
   - 统一终端环境，避免 Windows CMD 的兼容性问题
   - 支持 Linux 命令，便于后续开发

2. **环境变量管理**：
   - 使用 `.env` 文件 + `--env-file` 参数
   - 确保变量前没有 `#` 注释符
   - 等号两边不能有空格

3. **跨平台开发**：
   - 对于明显偏向 Linux 的项目，优先考虑 WSL 2 环境
   - 避免在 Windows 上花费过多时间解决兼容性问题

4. **编辑工具**：
   - 新手建议使用 VSCode 而不是 Vim 编辑配置文件
   - 善用 VSCode 的图形化界面提高效率

5. **日志记录**：
   - 遇到错误时保留完整日志，便于排查
   - 记录每一步的解决过程，形成知识积累

---

## 当前进度与下一步计划

### 已完成
✅ 在 VSCode 中配置 Git Bash 终端  
✅ 安装 uv 包管理器  
✅ 配置 `.env` 文件，解决 API Key 加载问题  
✅ 定位 `NotImplementedError` 的根本原因  

### 待执行
⏳ 在本机安装 Linux 系统（WSL 2）  
⏳ 在 VSCode 中打开 WSL 终端  
⏳ 在 WSL 环境中重建项目并运行  
⏳ 验证服务在 Linux 环境下正常持续运行  

---

