# Claude Code 在 Windows 上从零开始的完整配置与使用指南

> 本文档面向**完全没用过命令行、没装过任何 AI 终端工具**的零基础读者。每一步都会给出**精确的命令**、**期望看到的输出**以及**遇到错误时该怎么办**。按顺序照着做即可。
>
> - 适用系统：Windows 10 / Windows 11（64 位）
> - 适用版本：Claude Code 2.x（2026 年 5 月）
> - 默认模型：Claude Sonnet 4.6（可手动切换到 Opus 4.7 / Haiku 4.5）
> - 工程量预估：第一次完整跑下来约 **20–40 分钟**（比 Codex 简单，因为有原生安装器，不强依赖 Node.js）

---

## 目录

- [一、先搞清楚 Claude Code 是什么](#一先搞清楚-claude-code-是什么)
- [二、动手前的硬性条件清单](#二动手前的硬性条件清单)
- [三、安装 Claude Code](#三安装-claude-code)
- [四、选定终端 + 中文乱码修复（重要）](#四选定终端--中文乱码修复重要)
- [五、首次登录（三种方式）](#五首次登录三种方式)
- [六、切换账号 / 重新登录](#六切换账号--重新登录)
- [七、第一次跑 Claude Code（Hello World）](#七第一次跑-claude-codehello-world)
- [八、日常使用：模型、Slash 命令、模式](#八日常使用模型slash-命令模式)
- [九、配置文件 `~/.claude/settings.json`](#九配置文件-claudesettingsjson)
- [十、常见问题与解法（FAQ）](#十常见问题与解法faq)
- [十一、卸载与彻底清理](#十一卸载与彻底清理)
- [十二、进阶资源（CLAUDE.md / MCP / Hooks / Agents）](#十二进阶资源claudemd--mcp--hooks--agents)

---

## 一、先搞清楚 Claude Code 是什么

**Claude Code** 是 Anthropic 推出的、**跑在你电脑终端里**的编程 Agent。一句话定位：

- 它**不是**网页版 Claude（claude.ai），也**不是**桌面客户端。
- 它**是**一个命令行程序，启动后你用自然语言让它**直接读你的代码、改你的文件、跑你的命令**。
- 它有**严格的权限模型**——默认每次写盘 / 跑命令都问你一次，可以按级别调放行。
- 收费模式：**包含在 Claude Pro / Max / Team / Enterprise 订阅里**；也可以用 **Anthropic API Key** 按 token 付费；企业可以走 **Amazon Bedrock / Google Vertex AI**。

### 跟 OpenAI Codex CLI 的关系

两者**是同类工具的竞品**：

| 维度 | Claude Code | Codex CLI |
|---|---|---|
| 厂商 | Anthropic | OpenAI |
| 默认模型 | Sonnet 4.6 | GPT-5-Codex |
| 入门订阅 | Claude Pro $20/月 | ChatGPT Plus $20/月 |
| Windows 安装 | **官方原生安装器，一行命令** | 需要先装 Node.js 22+ 再 npm 全局装 |
| 杀手锏功能 | CLAUDE.md / Subagents / MCP / Hooks / Auto-memory | 沙盒模式细分、`--full-auto` 流畅 |

两个工具**互不冲突，完全可以同时装**。本指南只讲 Claude Code；Codex 的安装看仓库里的另一份指南。

---

## 二、动手前的硬性条件清单

| # | 条件 | 自查方法 |
|---|---|---|
| 1 | Windows 10 / 11（64 位） | 右键「此电脑」→「属性」看"系统类型" |
| 2 | 至少能装软件的权限 | 装得了微信就行 |
| 3 | 能稳定访问国际网络 | 浏览器能否打开 `https://claude.ai/` 和 `https://console.anthropic.com/` |
| 4 | 一个 Anthropic / Claude 账号 | 没有就去 `https://claude.ai/` 注册 |
| 5 | **三选一**：Claude 订阅（Pro / Max）/ Anthropic API Key / 企业云（Bedrock / Vertex） | 见 [第五章](#五首次登录三种方式) |
| 6 | 约 500 MB 磁盘空间 | C 盘别接近爆满 |

> **关于网络：** Claude Code 需要稳定访问 `claude.ai`、`console.anthropic.com`、`api.anthropic.com`。国内直连这些域名 **大概率不通**，请提前确认你的网络环境能访问 Anthropic，否则登录会卡在 OAuth 回调。

---

## 三、安装 Claude Code

### 3.1 推荐路线：用原生安装器（不需要 Node.js）

这是 **2026 年 Anthropic 官方推荐**的方式。一行 PowerShell 命令搞定，**自动更新**，**不依赖 Node**。

1. 按 `Win + R`，输入 `powershell`，回车，打开一个普通用户的 PowerShell 窗口。

2. 把下面这一整行**完整复制**进去再回车：

   ```powershell
   irm https://claude.ai/install.ps1 | iex
   ```

   - `irm` 是 `Invoke-RestMethod` 的别名，下载脚本
   - `iex` 是 `Invoke-Expression`，执行脚本
   - 整条命令的意思就是"下载 Anthropic 官方安装脚本并立即执行"

3. 看到类似下面的输出说明装完：

   ```
   Claude Code installed to C:\Users\<你>\AppData\Local\Claude\
   Add to PATH: done
   ```

4. **关掉当前 PowerShell，重新开一个新的**（让 PATH 生效），然后：

   ```powershell
   claude --version
   ```

   应输出类似 `2.x.x` 的版本号。

### 3.2 备用路线：用 npm 安装

适合已经装了 Node.js、或者偏好用 npm 管理工具的人。

**前置：** 必须有 Node.js 18+（推荐 20 LTS）。如果你没装过，跟仓库里 `Codex-CLI-Windows-从零开始完整指南.md` 的"第三章 检查并安装 Node.js"走一遍（注意 Codex 要求 22+，但 Claude Code 只要 18+，所以装 22 同时满足两个工具）。

**装：**

```powershell
npm install -g @anthropic-ai/claude-code
```

期望看到 `added 1 package`。验证：

```powershell
claude --version
```

### 3.3 安装阶段可能踩的坑

| 错误信息 | 原因 | 解法 |
|---|---|---|
| `irm: 无法解析...` | 网络不通 `claude.ai` | 确认浏览器能打开 `https://claude.ai/`；检查代理/防火墙 |
| `iex: 因为在此系统上禁止运行脚本` | PowerShell 执行策略限制 | 见 [4.1 修复执行策略](#41-修复-powershell-执行策略) |
| 装完 `claude` 仍 `不是内部或外部命令` | PATH 没刷新 | **关掉所有终端窗口重开**；还不行就**重启电脑** |
| `npm install -g` 报 `EACCES` | npm 全局目录无写权限 | 用管理员开 PowerShell 重试 |
| `npm` 路线装的 claude 启动报错 | Node 版本太低 | 升级到 18+，详细排错见 Codex 指南第三章 |

> **不要**两条路线都装。如果先用 npm 装过、再想换原生安装器，先 `npm uninstall -g @anthropic-ai/claude-code` 再跑 `irm ... | iex`。

---

## 四、选定终端 + 中文乱码修复（重要）

### 4.1 修复 PowerShell 执行策略

如果原生安装器报"禁止运行脚本"，或者后续 `claude` 启动报同类错——一劳永逸修：

1. **以管理员身份**打开 PowerShell（开始菜单搜 PowerShell → 右键 → 以管理员身份运行）。

2. 执行：

   ```powershell
   Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
   ```

3. 提示输入 `Y` 回车。

4. 关闭管理员窗口，用**普通用户**身份重开 PowerShell 继续后面的操作。

### 4.2 修复中文乱码（Chinese / CJK 字符显示成方块或问号）

PowerShell 默认编码是 OEM 代码页（中文系统是 GBK），Claude Code 输出的是 UTF-8。结果就是**中文字符全乱码**。这是中文 Windows 用户最常踩的坑。

**一次性修复：** 把下面两步都做完。

**步骤 1：把控制台代码页改成 UTF-8**

在普通用户 PowerShell 里执行：

```powershell
notepad $PROFILE
```

如果提示文件不存在，**点是**创建。Notepad 打开后**追加**这一行（不要覆盖你已有的内容）：

```powershell
chcp 65001 > $null
[Console]::OutputEncoding = [Text.Encoding]::UTF8
```

保存关闭。**关掉所有终端窗口重开**，以后每次 PowerShell 启动都会自动设 UTF-8。

**步骤 2：换一个支持中日韩字符的等宽字体**

PowerShell 窗口默认字体是 Consolas，**不含中文字形**。改：

1. PowerShell 窗口左上角图标 → 属性 → 字体
2. 把字体改成 `MS Gothic` / `NSimSun` / `Cascadia Mono`（任选）
3. 确定

> **更省心的替代：** 直接用 **Windows Terminal**（从 Microsoft Store 免费装）。它默认就是 UTF-8 + 支持中文的现代字体，**不用做上面任何修复**。强烈推荐。

### 4.3 终端选择推荐

| 终端 | 推荐度 | 说明 |
|---|---|---|
| **Windows Terminal** | ★★★★★ | 现代化，默认 UTF-8，零配置；强烈推荐 |
| PowerShell 5.1（系统自带） | ★★★★ | 能用，但需要按 4.2 修中文 |
| Git Bash | ★★★ | Claude Code 在 Windows 上默认会探测 Git Bash 用作 shell，**建议装上** |
| cmd（旧版命令提示符） | ★★ | 能跑，体验差 |
| WSL2 | ★★★★ | Linux 体验最纯，但门槛偏高，本指南不展开 |

> **强烈建议安装 Git for Windows**（自带 Git Bash）。Claude Code 跑命令时优先用 Bash；没有 Bash 会回退到 PowerShell，对部分命令有兼容性影响。装法：浏览器打开 `https://git-scm.com/download/win`，下载 64 位 Setup，全程默认下一步。

---

## 五、首次登录（三种方式）

Claude Code 必须登录后才能用。支持三种登录方式：

| 方式 | 适合 | 计费 |
|---|---|---|
| **A. Claude.ai 账号 OAuth 登录** | 已经订阅 Claude Pro / Max / Team 的用户 | **不另外扣钱**（用订阅额度） |
| **B. Anthropic API Key** | 想精确按用量付费，或没有订阅 | 按 token 收费 |
| **C. Bedrock / Vertex AI** | 企业用户，公司用 AWS 或 GCP 凭据 | 走云厂商账单 |

本节主要讲 A 和 B；C 是企业场景，配置文件示例放在第九章。

### 5.1 方式 A：Claude.ai 账号 OAuth 登录

1. 在新打开的 PowerShell 里执行：

   ```powershell
   claude
   ```

2. 第一次启动会出现交互菜单：
   - **第一步**：选 Text style（文字风格）——默认即可，按 Enter。
   - **第二步**：选认证方式——选 **`Claude account with subscription`**。

3. 它会**自动打开浏览器**跳到 Anthropic 的 OAuth 授权页。如果没自动开，终端会显示一个 URL，**复制到浏览器手动打开**。

4. 浏览器里登录你订阅的 Claude 账号（Pro / Max / Team），点授权。

5. 浏览器显示"You can return to your terminal"之类字样后回终端——已登录。

6. 终端进入 Claude Code 主界面。先输入 `/exit` 退出，下一步再正式用。

> **如果浏览器一直转圈、回不到终端：** 见 [FAQ Q5](#q5登录浏览器一直转圈无法回调到终端)。

### 5.2 方式 B：Anthropic API Key 登录

#### Step 1：拿到 API Key

1. 浏览器打开 `https://console.anthropic.com/`，登录。
2. 点 **Settings → API Keys → Create Key**，命名（比如 `claude-code`），创建。
3. 弹出的 Key 形如 `sk-ant-api03-...`——**只显示这一次！** 立即**复制到本地的密码管理器**保存。
4. 顺便去 **Plans & Billing** 充值（至少 $5，否则会 401）。

#### Step 2：设环境变量

在 PowerShell 里执行（把 `sk-ant-你的KEY` 替换成你刚才复制的那串）：

```powershell
setx ANTHROPIC_API_KEY "sk-ant-你的KEY"
```

`setx` 把变量**永久**写入用户级环境变量，但**不对当前窗口生效**。

5. **关掉当前 PowerShell，重新开一个**。
6. 验证：

   ```powershell
   echo $env:ANTHROPIC_API_KEY
   ```

   应输出你的 Key。

#### Step 3：启动

```powershell
claude
```

第一次进交互菜单选 `Use API key` / `Anthropic Console account`；之后启动会自动用环境变量里的 Key。

> **API Key 泄漏立即去 Console 撤销**，再生成新的。

### 5.3 方式 C：Bedrock / Vertex AI（企业用户）

需要在 `~/.claude/settings.json` 里配置 `apiProvider`、AWS / GCP 凭据。详见第九章，本节略。

---

## 六、切换账号 / 重新登录

如果你已经登录过 Claude Code，想换账号——比如换另一个 Claude 订阅账号、换 API Key、或者从订阅切换到 API Key——按本章来。

### 6.1 先确认当前登录方式

启动 Claude Code：

```powershell
claude
```

主界面输入：

```
/status
```

会显示当前认证类型（`Subscription` / `API key`）和账号信息。`/exit` 退出。

### 6.2 场景 A：从 Claude 订阅账号 A 换到账号 B

1. **退出当前登录**——在 Claude Code 交互界面里输入：
   ```
   /logout
   ```
   或者从命令行：
   ```powershell
   claude /logout
   ```

2. **清浏览器对 Anthropic 的登录态**——这是 90% 切账号失败的根因。OAuth 会复用浏览器里 `claude.ai` 的登录 cookie，没清就会**直接拿账号 A 重新授权**。三种清法任选：

   - **最省事**：用浏览器**无痕窗口**（`Ctrl+Shift+N`）完成下面的重登
   - **彻底清**：`Ctrl+Shift+Delete` → 全部时间 → 勾 Cookies → 清除
   - **精确清**：浏览器地址栏 `chrome://settings/cookies/detail?site=claude.ai` 和 `?site=anthropic.com`，逐个删

3. **用账号 B 重新登录**：
   ```powershell
   claude /login
   ```
   浏览器（或无痕窗口）跳出 OAuth 时**用账号 B**登录授权。

4. **验证**：再启动 `claude`，输入 `/status`，账号应是 B。

### 6.3 场景 B：换 API Key

1. **后台生成新 Key**：`https://console.anthropic.com/` → API Keys → Create。

2. **撤销旧 Key**（强烈建议）：同一页面找到旧 Key → Revoke。

3. **更新环境变量**：
   ```powershell
   setx ANTHROPIC_API_KEY "sk-ant-新KEY"
   ```

4. **关掉所有终端**，重开一个。

5. **验证**：
   ```powershell
   echo $env:ANTHROPIC_API_KEY
   ```
   应输出新 Key。

6. 启动 `claude` 验证。

### 6.4 场景 C：从订阅切换到 API Key

1. `/logout` 退出 OAuth 登录
2. 按 [6.3](#63-场景-b换-api-key) 设环境变量
3. 启动 `claude`，选 `Use API key`

### 6.5 场景 D：从 API Key 切回订阅

**关键：** 只要 `ANTHROPIC_API_KEY` 环境变量还在，Claude Code 启动会优先用它。第一步必须先删环境变量。

1. **删 `ANTHROPIC_API_KEY` 环境变量**（任选）：

   **PowerShell 命令**：
   ```powershell
   [Environment]::SetEnvironmentVariable("ANTHROPIC_API_KEY", $null, "User")
   ```

   > **不要用** `setx ANTHROPIC_API_KEY ""`——某些 Windows 版本只是把值设为空串而不是真删，Claude Code 仍会读到空串然后报错。

   **图形界面**：`Win + R` → `sysdm.cpl` → 高级 → 环境变量 → 用户变量里找到删掉

2. **关掉所有终端**，重开一个。

3. **验证已删**：
   ```powershell
   echo $env:ANTHROPIC_API_KEY
   ```
   应输出空行。

4. 启动 `claude`，选 `Claude account with subscription` 走 OAuth。

### 6.6 终极方案：手动彻底清理登录态

`/logout` 没用、想从零开始一个干净登录环境——逐条执行：

1. **删 Claude Code 凭据文件**：
   ```powershell
   Remove-Item -Force $HOME\.claude\.credentials.json -ErrorAction SilentlyContinue
   ```

2. **清 Windows 凭据管理器**：
   - 控制面板 → 用户账户 → 凭据管理器 → **Windows 凭据**
   - 找含 `anthropic` / `claude-code` 的条目 → 全删
   - **Web 凭据**也同样处理
   - 命令行快捷：`control /name Microsoft.CredentialManager`

3. **删 `ANTHROPIC_API_KEY`** 环境变量（[6.5 Step 1](#65-场景-d从-api-key-切回订阅)）。

4. **清浏览器 cookies**（[6.2 Step 2](#62-场景-a从-claude-订阅账号-a-换到账号-b)）。

5. **保留**这些（除非你想完全重置）：
   - `~/.claude/settings.json` — 你的配置
   - `~/.claude/CLAUDE.md` 和项目里的 `CLAUDE.md` — 自定义指令
   - `~/.claude/memory/` — 你和 Claude 的长期记忆

做完 1–4，再 `claude /login` 等同于全新机器第一次登录。

### 6.7 切换没生效？排查清单

| 现象 | 原因 | 解法 |
|---|---|---|
| `/status` 还显示账号 A | 浏览器 cookies 没清 | 用无痕窗口重登 |
| `echo $env:ANTHROPIC_API_KEY` 还是旧 Key | 终端是 `setx` 之前开的 | 关掉**所有**终端重开 |
| 想切到订阅，但 `/status` 仍显示 API Key | 环境变量没真删 | 走 [6.5 Step 1 PowerShell 命令](#65-场景-d从-api-key-切回订阅) |
| `/logout` 显示成功但下次启动还是自动登录 | 凭据管理器有缓存 | [6.6 Step 2](#66-终极方案手动彻底清理登录态) |

### 6.8 用了 CC Switch 或第三方中转站？

如果你属于以下任何一种情况：

- 装过 **CC Switch**（开源 GUI 账号切换器）
- 用过 `ANTHROPIC_AUTH_TOKEN` + `ANTHROPIC_BASE_URL` 接入过第三方"中转站"（国内代理 API）
- CC Switch 切了 Provider 但 `/status` 还是旧账号
- 想同时管理多个账号（官方 + 多个中转站）

**单独的专题文档**：[Claude Code 第三方中转站与 CC Switch 完整指南](./Claude-Code-中转站与CC-Switch-完整指南.md)

里面覆盖了：环境变量 vs CC Switch 的优先级冲突、中转站三种类型辨别、安装配置 CC Switch、切换标准流程、所有"切了没生效"坑、中转站常见报错、怎么彻底回归官方、隐私合规说明。

---

## 七、第一次跑 Claude Code（Hello World）

### 7.1 准备一个测试目录

```powershell
mkdir D:\claude-test
cd D:\claude-test
```

> 路径**不要**包含中文、空格、特殊字符。Codex/Claude 在中文路径下会偶发问题。

### 7.2 启动

```powershell
claude
```

进入 Claude Code 主界面。底部状态栏显示当前模型、目录、模式。

### 7.3 输入第一个提示词

直接打字回车：

```
帮我在当前目录创建一个 hello.py 文件，写一段打印「Hello from Claude Code」的代码，然后运行它把输出告诉我。
```

Claude Code 会：

1. **说明计划**——"我将创建 hello.py..."
2. **请求批准写文件**——按 `1` 接受一次（或 `2` 始终接受这类操作，会话内不再问）。
3. **请求批准跑命令** `python hello.py`——同样批准。
4. **给你输出**：`Hello from Claude Code`。

### 7.4 退出

```
/exit
```

或按 `Ctrl+C` 两次。

### 7.5 第一次可能踩的坑

| 现象 | 解释 | 解法 |
|---|---|---|
| 它请求批准跑 `python`，但你没装 Python | 跑了机器上没有的程序 | 装 Python，或让它创建 `.txt` 文件 |
| 输出中文是问号 / 方块 | PowerShell 编码问题 | 走 [4.2 修中文乱码](#42-修复中文乱码chinese--cjk-字符显示成方块或问号) |
| 报 `Bash not found` 之类警告 | 没装 Git for Windows | 装 Git for Windows；或忽略，回退到 PowerShell 也能用 |

---

## 八、日常使用：模型、Slash 命令、模式

### 8.1 启动参数

```powershell
claude                                  # 默认：Sonnet 4.6，交互模式
claude --model opus                     # 用 Opus 4.7（复杂任务）
claude --model haiku                    # 用 Haiku 4.5（快/省钱）
claude --print "解释这段代码"            # 非交互，print 模式，一次性输出
claude --resume                         # 恢复上一次会话
claude --continue                       # 继续最近的会话
```

### 8.2 交互内的 Slash 命令（常用）

进入 Claude Code 后输入：

| 命令 | 作用 |
|---|---|
| `/help` | 列出所有命令 |
| `/model` | 切换模型（opus / sonnet / haiku） |
| `/status` | 查看当前账号 / 模型 / 模式 |
| `/login` | 登录 |
| `/logout` | 退出登录 |
| `/clear` | 清空当前会话上下文 |
| `/compact` | 压缩当前会话（保留要点，节省 token） |
| `/init` | 在当前项目生成 `CLAUDE.md`（项目级提示） |
| `/memory` | 查看 / 编辑长期记忆 |
| `/agents` | 列出可用 subagents |
| `/mcp` | 列出 / 管理 MCP servers |
| `/hooks` | 配置 hooks |
| `/permissions` | 调整权限模式 |
| `/exit` | 退出 |

### 8.3 三种权限模式

| 模式 | 行为 | 适合 |
|---|---|---|
| **default**（默认） | 每次写文件 / 跑命令前问你 | 不熟悉的代码、生产仓库 |
| **acceptEdits** | 自动接受文件编辑，但跑命令仍问 | 重构、批量改文件 |
| **bypassPermissions** | 全自动，不问 | 完全信任的实验目录 |

切换：交互里按 `Shift + Tab`，或 `/permissions` 选。

### 8.4 进入 Plan 模式

按 `Shift + Tab` 两次进入 **Plan 模式**——Claude 只规划不动手，给你看完整方案后再让你批准动手。**推荐**给小白用：先看它的计划，确认无误再让它跑。

---

## 九、配置文件 `~/.claude/settings.json`

第一次跑过 `claude` 后，会在 `C:\Users\<你>\.claude\` 下自动建配置目录。

### 9.1 配置层级

Claude Code 的配置**有四层**，优先级从高到低：

1. **Managed settings**（企业级）：`C:\Program Files\ClaudeCode\managed-settings.json`
2. **Project local**：项目根目录 `.claude\settings.local.json`（个人覆盖，gitignored）
3. **Project**：项目根目录 `.claude\settings.json`（提交进 git，团队共享）
4. **User**：`%USERPROFILE%\.claude\settings.json`（你的全局默认）

### 9.2 基础配置示例（全局）

编辑全局配置：

```powershell
notepad $HOME\.claude\settings.json
```

文件不存在时 Notepad 会提示新建。贴入：

```json
{
  "model": "sonnet",
  "permissions": {
    "allow": [
      "Read",
      "Glob",
      "Grep"
    ],
    "deny": []
  },
  "env": {
    "EDITOR": "code"
  }
}
```

- `model` 设默认模型：`sonnet` / `opus` / `haiku`
- `permissions.allow` 列出**永远放行**的工具，省得反复点确认
- `env` 注入环境变量

### 9.3 Bedrock / Vertex 配置（企业）

`settings.json` 里：

```json
{
  "apiProvider": "bedrock",
  "bedrock": {
    "region": "us-east-1",
    "modelId": "anthropic.claude-sonnet-4-6-20250929-v1:0"
  }
}
```

具体配 AWS 凭据走 `aws configure`；Vertex 走 `gcloud auth application-default login`。本指南不展开。

### 9.4 项目级 `CLAUDE.md`

在你的项目根目录创建 `CLAUDE.md`，写项目规范、技术栈、约定、不要做的事——**每次** Claude Code 进入这个项目都会自动读。

最快的方式：在项目目录里跑 `claude`，输入 `/init`，它会扫描项目自动生成一份。

---

## 十、常见问题与解法（FAQ）

### Q1：`claude` 报 `'claude' 不是内部或外部命令`

**解法（按顺序）：**

1. **关掉所有终端**重开一个 PowerShell，再试。
2. 还不行：用原生安装器重装：`irm https://claude.ai/install.ps1 | iex`，安装完**重启电脑**。
3. npm 路线用户：`npm config get prefix`，把输出路径加到「环境变量 → 用户变量 → Path」。

### Q2：PowerShell 报 `因为在此系统上禁止运行脚本`

走 [4.1 修复 PowerShell 执行策略](#41-修复-powershell-执行策略)。

### Q3：中文输出乱码 / 方块 / 问号

走 [4.2 修复中文乱码](#42-修复中文乱码chinese--cjk-字符显示成方块或问号)。两步必须都做（编码 + 字体）。

### Q4：报 `401 Unauthorized` 或 OAuth 错

**可能原因：**

1. 订阅状态过期 → 去 `https://claude.ai/settings/billing` 检查
2. 凭据缓存损坏 → `/logout` 后再 `/login`
3. 同时设了 `ANTHROPIC_API_KEY` 但 Key 无效 → 检查环境变量；要走订阅就删环境变量（[6.5](#65-场景-d从-api-key-切回订阅)）

**核武器**：删 `~/.claude/.credentials.json` 后重登。

### Q5：登录浏览器一直转圈，无法回调到终端

**原因**：网络不通 `claude.ai` 或回调端口被拦。

**解法：**

1. 浏览器手动确认能打开 `https://claude.ai/`
2. 关闭杀软 / 防火墙对终端的拦截，重试
3. 走不通就改用 **API Key**（[5.2](#52-方式-banthropic-api-key-登录)），不依赖浏览器回调

### Q6：429 速率限制 / 用量耗尽

**原因：** 单位时间内请求太多、或订阅档位上限到了。

**解法：**

- 等 5 小时后重置（订阅是按 5 小时滚动窗口）
- `/compact` 或 `/clear` 减少上下文 token 占用
- 切到 Haiku（`/model haiku`）省 token
- 必要时升级订阅（Pro $20 → Max $100 → Max $200）

### Q7：跑 bash 命令时报错 / 行为奇怪

**原因：** 没装 Git for Windows，Claude 回退到 PowerShell，但提示词假设是 bash。

**解法：** 装 Git for Windows（自带 Git Bash），重启 PowerShell。

### Q8：路径含中文 / 空格导致命令失败

**最佳实践：** 把项目放在**不含中文、不含空格**的纯英文路径里，如 `D:\projects\my-app`。`D:\桌面\我的项目` 经常出怪事。

### Q9：context length exceeded / 上下文爆了

**解法：**

- `/compact` 压缩当前会话
- `/clear` 彻底清空重来
- 把项目长期信息写进 `CLAUDE.md`，Claude 启动时自动读，不占临时上下文

### Q10：删了文件 / 跑了删除命令后悔了

- 项目在 git 里：`git status` + `git restore <file>` 或 `git reset --hard HEAD`
- 不在 git 里：Windows 回收站碰运气；命令行 `rm` 不进回收站
- **教训：** 重要工作目录养成 `git init` 习惯，宁可多 commit 也别让 Agent 在裸目录跑 `bypassPermissions`

### Q11：能不能完全离线用？

**不能。** 模型推理在 Anthropic 服务器，必须联网。但 Claude Code 跑的本地命令可以**离线**——你可以禁止它联网，只让它操作本地文件。

### Q12：能和 Codex CLI 同时装吗？

**完全可以。** 两个工具配置目录不同：Claude Code 在 `~/.claude/`，Codex 在 `~/.codex/`。互不干扰。

### Q13：误操作让 Claude 删了文件能恢复吗？

参考 Q10。**别裸跑 `bypassPermissions`** 是关键。

### Q14：怎么减少 token 消耗 / 省钱

- 优先用 Sonnet 而不是 Opus
- 简单任务用 Haiku
- 长会话定期 `/compact`
- 用 `CLAUDE.md` 沉淀重复指令，别每次复述
- 用 Plan 模式让它先规划，避免无效尝试

### Q15：怎么判断我现在用的是 Anthropic 官方还是某个中转站？

启动 `claude`，输入 `/status`，看 **Endpoint** 字段：

- `https://api.anthropic.com` → 官方
- 其它任何地址 / IP（如 `https://api.xxx.com` 或 `http://1.2.3.4:8080`）→ 中转站

也可以在 PowerShell 里跑：

```powershell
echo $env:ANTHROPIC_BASE_URL
```

如果有值且不是 `api.anthropic.com`，就是中转站。深度排查见 [中转站与 CC Switch 完整指南 第六章](./Claude-Code-中转站与CC-Switch-完整指南.md#六自检你机器上当前的认证状态)。

### Q16：CC Switch 切换了 Provider 但 `/status` 还是旧账号？

99% 是**环境变量在压制 CC Switch**。Claude Code 启动时认证优先级是：

```
环境变量  >  ~/.claude/settings.json (CC Switch 写入)  >  OAuth 缓存
```

CC Switch 改的是 settings.json。只要 `ANTHROPIC_API_KEY` / `ANTHROPIC_AUTH_TOKEN` / `ANTHROPIC_BASE_URL` **任何一个**有值，就先生效，把 CC Switch 的设置盖掉。

清环境变量（PowerShell）：

```powershell
[Environment]::SetEnvironmentVariable("ANTHROPIC_API_KEY",    $null, "User")
[Environment]::SetEnvironmentVariable("ANTHROPIC_AUTH_TOKEN", $null, "User")
[Environment]::SetEnvironmentVariable("ANTHROPIC_BASE_URL",   $null, "User")
```

**关掉所有终端（包括整个 Windows Terminal 进程）重开**，再 CC Switch 里 Activate 目标 Provider。详见 [中转站指南 5.5](./Claude-Code-中转站与CC-Switch-完整指南.md#55-用-cc-switch-时必须清掉环境变量)。

### Q17：能让 Claude Code 用本地 LLM（Ollama 等）吗？

**技术上可以**——Ollama 等本地服务能模拟 Anthropic API 协议：

```powershell
setx ANTHROPIC_BASE_URL "http://localhost:11434/v1"
setx ANTHROPIC_AUTH_TOKEN "dummy"
```

然后 `claude --model llama3` 之类。

**但模型质量断崖式下降**：本地开源模型在 Tool use / Caching / Subagents / 长上下文方面跟 Claude 差距巨大，**Claude Code 大部分高级功能会瘫痪**。仅适合极偶尔的离线场景；正常工作流绝不推荐。

### Q18：Anthropic 会用我的代码训练模型吗？

| 渠道 | 是否用于训练 |
|---|---|
| Claude.ai 订阅（Pro / Max） | **默认不会**（可在设置里再次确认） |
| Anthropic API / Console | **明确承诺不会** |
| 第三方中转站 | **取决于中转站隐私政策**——很可能被记录用于二次销售或自家模型训练 |
| Bedrock / Vertex AI | **不会**（数据在你自己的云账户） |

涉密代码 / 商业 IP **绝对不要走中转站**。详细政策：`https://www.anthropic.com/legal/privacy`。

---

## 十一、卸载与彻底清理

### 11.1 原生安装器装的

```powershell
where.exe claude
```

记下输出路径（通常 `C:\Users\<你>\AppData\Local\Claude\claude.exe`），删掉整个目录：

```powershell
Remove-Item -Recurse -Force "$env:LOCALAPPDATA\Claude"
```

然后去「环境变量 → Path」里删掉含 `Claude` 的条目。

### 11.2 npm 装的

```powershell
npm uninstall -g @anthropic-ai/claude-code
```

> **nvm-windows 用户注意**：如果你切换过多个 Node 版本、每个版本都装过 claude——需要在**每个 Node 版本下**都 `npm uninstall` 一遍：
> ```powershell
> nvm use 18
> npm uninstall -g @anthropic-ai/claude-code
> nvm use 20
> npm uninstall -g @anthropic-ai/claude-code
> nvm use 22
> npm uninstall -g @anthropic-ai/claude-code
> ```

### 11.3 彻底清配置 / 凭据 / 历史

```powershell
Remove-Item -Recurse -Force $HOME\.claude -ErrorAction SilentlyContinue
Remove-Item -Force $HOME\.claude.json -ErrorAction SilentlyContinue
```

去 **凭据管理器** 删 `anthropic` / `claude-code` 条目。

### 11.4 验证

```powershell
claude --version
```

应报"未识别的命令"——卸干净了。

---

## 十二、进阶资源（CLAUDE.md / MCP / Hooks / Agents）

入门以后，下面这些是 Claude Code 的真正杀手锏功能：

### 12.1 CLAUDE.md（项目级提示）

在项目根目录建 `CLAUDE.md`，写：
- 项目技术栈、架构
- 代码风格约定
- 不要做的事
- 测试 / 构建命令

Claude Code 每次进入这个目录都会自动读。

**最快建：** 项目目录里跑 `claude`，输入 `/init`。

### 12.2 Subagents（子代理）

让 Claude Code 派遣**专门化的子代理**处理特定任务（代码评审、安全审计、文档生成）。

- 列出：`/agents`
- 自定义：在 `~/.claude/agents/` 或项目 `.claude/agents/` 下写 `.md` 文件

### 12.3 MCP Servers（模型上下文协议）

把外部工具（数据库、API、内部系统）接进 Claude Code，让它直接操作。

- 列出：`/mcp`
- 配置：`~/.claude/settings.json` 的 `mcpServers` 字段；或用 `claude mcp add ...` 命令

### 12.4 Hooks（事件钩子）

让 Claude Code 在特定事件（写文件前、命令跑完后、退出前）自动跑你指定的脚本。

- 配置：`settings.json` 的 `hooks` 字段
- 常见用途：自动 lint、自动 commit、自动通知

### 12.5 Skills（技能）

用 `/skills` 调用预装的技能（如 `init`, `review`, `security-review`）。

### 12.6 IDE 集成

- **VS Code**：装 `Claude Code` 扩展，自动同步会话
- **JetBrains**：装官方插件
- **Cursor**：原生支持

### 12.7 官方文档

- 主页：`https://code.claude.com/`
- Setup 文档：`https://code.claude.com/docs/en/setup`
- 认证：`https://code.claude.com/docs/en/authentication`
- 模型配置：`https://code.claude.com/docs/en/model-config`
- 设置：`https://code.claude.com/docs/en/settings`
- 故障排查：`https://code.claude.com/docs/en/troubleshooting`

---

> 本指南随 Claude Code 版本演进会过时。看到某条命令报错时，先用 `claude --version` 对照官方 changelog 确认版本差异。
>
> 写于 2026-05-23，对应 Claude Code 2.x。
