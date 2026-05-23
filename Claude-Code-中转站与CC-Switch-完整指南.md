# Claude Code 第三方中转站与 CC Switch 完整指南（小白避坑版）

> 本文档是 [Claude Code 在 Windows 上从零开始的完整配置与使用指南](./Claude-Code-Windows-从零开始完整指南.md) 的深度专题，专门解决以下问题：
>
> - 我没法直接付钱给 Anthropic（国内信用卡不支持），怎么用 Claude Code？
> - 我装过 CC Switch、配过 `ANTHROPIC_AUTH_TOKEN` 这种环境变量，**为什么切了账号没生效**？
> - 怎么在"官方订阅 / 官方 API Key / 多个中转站"之间干净地切换？
> - 中转站靠谱吗？有哪些坑？数据安全怎么办？
>
> 写于 2026-05-23。

---

## 目录

- [零、5 秒看懂你现在的状态](#零5-秒看懂你现在的状态)
- [一、中转站到底是什么？为什么国内用户离不开？](#一中转站到底是什么为什么国内用户离不开)
- [二、中转站的三种类型与辨别方法](#二中转站的三种类型与辨别方法)
- [三、安全警告：必读](#三安全警告必读)
- [四、纯环境变量路线（最简单，单账号场景）](#四纯环境变量路线最简单单账号场景)
- [五、CC Switch 路线（推荐，多账号场景）](#五cc-switch-路线推荐多账号场景)
- [六、自检：你机器上当前的认证状态](#六自检你机器上当前的认证状态)
- [七、切换账号 / 中转站的标准流程](#七切换账号--中转站的标准流程)
- [八、最容易踩的"切了没生效"坑](#八最容易踩的切了没生效坑)
- [九、中转站常见报错与解法](#九中转站常见报错与解法)
- [十、怎么彻底回到 Anthropic 官方](#十怎么彻底回到-anthropic-官方)
- [十一、隐私与合规说明](#十一隐私与合规说明)
- [十二、官方 vs 中转站决策表](#十二官方-vs-中转站决策表)
- [十三、信息来源与官方链接](#十三信息来源与官方链接)

---

## 零、5 秒看懂你现在的状态

打开 PowerShell，复制粘贴这三行回车：

```powershell
echo "ANTHROPIC_API_KEY:    $env:ANTHROPIC_API_KEY"
echo "ANTHROPIC_AUTH_TOKEN: $env:ANTHROPIC_AUTH_TOKEN"
echo "ANTHROPIC_BASE_URL:   $env:ANTHROPIC_BASE_URL"
```

对照下表自检：

| 输出情况 | 你实际在用什么 | 切账号时要看 |
|---|---|---|
| 三个都空 | Anthropic 官方（OAuth 订阅） | [第七章 路线 A 或 B-3](#七切换账号--中转站的标准流程) |
| 只有 `ANTHROPIC_API_KEY` 有值 | Anthropic 官方 API Key | [第七章 B-2](#路线-b完全不用-cc-switch直接改环境变量) |
| `ANTHROPIC_AUTH_TOKEN` + `ANTHROPIC_BASE_URL` 有值 | **第三方中转站** | [第七章 B-1](#路线-b完全不用-cc-switch直接改环境变量) |
| 三个变量乱七八糟都有值 | 历史遗留，混乱状态 | 先按 [第八章](#八最容易踩的切了没生效坑) 清干净再说 |

启动 `claude`，输 `/status` 看 Endpoint 字段——这是**终极判断标准**：
- `https://api.anthropic.com` → 官方
- 其他任何地址 / IP → 中转站

---

## 一、中转站到底是什么？为什么国内用户离不开？

**中转站**（也叫"**API 代理**"、"**国内站**"、"**镜像站**"、"**Claude 代理**"）是**第三方**运营的服务：把你的请求转发到 Anthropic（或 OpenAI）的官方 API，然后给国内用户提供更友好的访问方式。

### 1.1 为什么它存在

Anthropic 官方付费有两个硬门槛卡死了 90% 的国内用户：

1. **支付**：Anthropic 不接受中国大陆的银行卡 / 支付宝 / 微信。要付钱只能：
   - 借海外朋友的卡
   - 买 Apple 礼品卡曲线充值（仅对 ChatGPT，对 Claude 行不通）
   - **海外虚拟卡服务**（WildCard 等，约 $10 开卡费 + $2/月维护费 + 3-5% 充值溢价）
2. **网络**：Anthropic 的 API 域名（`api.anthropic.com`、`claude.ai`、`console.anthropic.com`）国内直连**不通**——必须有稳定的国际网络

中转站把这两个门槛同时拆掉：
- ✅ 国内信用卡 / 支付宝 / 微信付款
- ✅ 免翻墙直连
- ✅ 人民币计价
- ✅ 通常打包成"包月套餐"或"包量套餐"，比按 token 便宜

### 1.2 中转站和 Claude Code 怎么对接

Claude Code 通过两个环境变量接入中转站：

```
ANTHROPIC_BASE_URL  = 中转站的地址（如 https://api.xxx.com 或 http://1.2.3.4:8080）
ANTHROPIC_AUTH_TOKEN = 中转站给你签发的 token（形如 sk-xxxxxxxx...）
```

> **关键概念：** Claude Code 把请求按 Anthropic 协议构造，但发到 `ANTHROPIC_BASE_URL`，认证用 `ANTHROPIC_AUTH_TOKEN`。中转站收到后**自己**用它们和 Anthropic 官方 API 之间的关系发请求，**等于在中间做了一层翻译/计费**。

### 1.3 关键命名坑：三个变量别搞混

| 变量名 | 含义 | 何时用 |
|---|---|---|
| `ANTHROPIC_API_KEY` | Anthropic **官方** API Key | 来自 `console.anthropic.com` 的 `sk-ant-api03-...`；**只对 `api.anthropic.com`** 有效 |
| `ANTHROPIC_AUTH_TOKEN` | 通用 Bearer 令牌 | **给中转站用**；形如 `sk-...`，长得像但不是 Anthropic 的官方 key |
| `ANTHROPIC_BASE_URL` | API 端点地址 | 改这个就是改去哪个服务器；空 = 用默认 `https://api.anthropic.com` |

**两类 `sk-...` token 的视觉区别**：
- 官方 API Key：`sk-ant-api03-xxxxxxxxxxxxxxxx-xxxxxx`（有 `-ant-api03-` 中段）
- 中转站 token：`sk-xxxxxxxxxxxxxxxx`（没有 `-ant-` 中段）

> 拿到一个 `sk-` 开头的 Key 时，**先看有没有 `-ant-api03-`**，再判断该塞进哪个变量。这是新手最高频的坑——把中转站 token 塞进 `ANTHROPIC_API_KEY` 然后 Claude Code 报莫名其妙的错。

---

## 二、中转站的三种类型与辨别方法

不是所有中转站都一样。按**后端实现**分三类，能力差距巨大：

| 类型 | 后端实际是什么 | MCP / Caching / Skills | 价格 | 给小白的判断 |
|---|---|---|---|---|
| **A. 直连转发型** | 真的转发到 `api.anthropic.com` | **基本都能用** | 较贵，按 Anthropic 原价 + 10-30% 手续费 | 优先选这种 |
| **B. 协议兼容型** | 后端是 Qwen / DeepSeek / GLM 等开源/国产模型，伪装成 Anthropic 协议 | **大部分不能用** | **超便宜**（¥10/百万 token 见过） | 别用，体验完全不是 Claude |
| **C. 混合型** | 同时支持官方 Claude 和自研模型，按你的选择路由 | 选官方模型时能用 | 中等 | 可以，但容易选错模型变成 B |

### 2.1 怎么辨别（3 个小动作）

1. **问它自己是谁**：在 Claude Code 里输入：
   ```
   What model are you? Reply with the exact model name and version.
   ```
   - 直连转发型：准确回答 `Claude Sonnet 4.6` / `Claude Opus 4.7` 之类
   - 协议兼容型：露馅，回答 "I'm Qwen / GLM / DeepSeek / GPT" 之类
   - 协议兼容型有的会被"调教过"答得像 Claude，但接下来跑几个真任务就破功（代码风格、解题能力、中文表达都不一样）

2. **看价格**：
   - Anthropic 官方 Sonnet 4.6 是 **$3 输入 / $15 输出** 每百万 token
   - 任何中转站报价**低于 $1 / 百万 token**（约 ¥7）的，**100%** 是协议兼容型
   - 直连转发型一般报价是官方的 110%-140%

3. **测高级特性**：
   - 让 Claude Code 用 MCP：`/mcp` 看能不能调；
   - 试 1M context（Opus 4.7）：让它读一个 50 万 token 的长文档；
   - 跑 Subagents：`/agents` 试调一个
   - 上面任何一个挂，说明不是直连转发型

---

## 三、安全警告：必读

### 3.1 CC Switch 假冒站点

**官方 CC Switch 完全免费开源**：

- GitHub：`https://github.com/farion1231/cc-switch`
- 官网：`https://ccswitch.io`

**任何向你索要付款、充值、登录凭据的"CC Switch" 100% 假冒**——已经有用户在假冒站点损失财产。

✅ 只从 GitHub Releases 或 ccswitch.io 下载
❌ 不要相信任何中文翻译版、修改版、付费版

### 3.2 中转站的固有风险

| 风险 | 说明 |
|---|---|
| **代码 / Prompt 泄漏** | 你的所有对话经过中转站老板的服务器。技术上他们能记录、检索、卖给第三方。涉密代码、API Key、`.env` 内容粘进会话 = **泄露** |
| **运营方跑路** | 充了几百块的包年套餐，运营方一夜消失。投诉无门，没有任何法律保障 |
| **服务质量不稳定** | 中转站经常因 Anthropic 临时封禁 IP / 限速 / 改 API 而抽风。可能你正在赶 ddl 它挂了 |
| **ToS 风险（间接）** | Anthropic 不允许把 OAuth 凭据用于第三方代理（2026-02 更新的服务条款）。账号有被风控的可能 |

### 3.3 给小白的 5 条安全底线

1. **涉密 / 商业 / 大厂内网代码**：永远走官方账号或公司提供的 Bedrock/Vertex，**绝不**碰中转站
2. **个人学习 / 写文档 / 玩具项目**：用中转站没问题
3. **永远不要**在 Claude Code 会话里粘贴：
   - `.env` 文件
   - SSH 私钥
   - 生产环境 API Key
   - 数据库密码
4. **挑中转站**：用了 3 个月以上、有口碑的；新站再便宜也别第一时间充半年
5. **凭据隔离**：给中转站用的 token 永远只放环境变量，**绝不**写进项目代码或 `.gitignore` 外的文件

---

## 四、纯环境变量路线（最简单，单账号场景）

如果你**只在一个中转站**上跑（不需要频繁切换），最简单的方式就是手动设两个环境变量。

### 4.1 步骤

1. **从中转站后台拿到两个值**：
   - Base URL（如 `https://api.xxx.com` 或 `http://1.2.3.4:8080`）
   - Token（形如 `sk-...`）

2. **在 PowerShell 里设置**：
   ```powershell
   setx ANTHROPIC_BASE_URL "https://api.xxx.com"
   setx ANTHROPIC_AUTH_TOKEN "sk-xxxx你的token"
   ```

3. **关掉所有终端窗口**（PowerShell / cmd / Git Bash / Windows Terminal **全关**），重开一个。

4. **验证**：
   ```powershell
   echo $env:ANTHROPIC_BASE_URL
   echo $env:ANTHROPIC_AUTH_TOKEN
   ```
   两行都应输出非空值。

5. **启动**：
   ```powershell
   claude
   ```

   > **千万别走 `/login`**——它会拉你去 claude.ai 走 OAuth，跟你的中转站设置冲突。中转站不需要登录，环境变量设好直接用就行。

6. **验证**：交互里输 `/status`，Endpoint 应该是你的中转站地址。

### 4.2 删除 / 换中转站

```powershell
# 换地址
setx ANTHROPIC_BASE_URL "https://新地址"
setx ANTHROPIC_AUTH_TOKEN "sk-新token"

# 删除（彻底回到官方）
[Environment]::SetEnvironmentVariable("ANTHROPIC_BASE_URL",   $null, "User")
[Environment]::SetEnvironmentVariable("ANTHROPIC_AUTH_TOKEN", $null, "User")
```

> **删除变量必须用 `SetEnvironmentVariable(..., $null, "User")`**，**不能**用 `setx ANTHROPIC_BASE_URL ""`——后者在某些 Windows 版本只是设成空串，Claude Code 仍会读到空串然后报错。

---

## 五、CC Switch 路线（推荐，多账号场景）

如果你有**多个账号**——比如同时有 Anthropic 官方 + 中转站 A + 中转站 B——手动 `setx` 来回切是噩梦。**CC Switch** 把切换变成"GUI 里点一下"。

### 5.1 CC Switch 是什么

跨平台桌面应用，由开发者 farion1231 维护，2026 年 5 月已 75K+ stars。它能：

- 用 GUI 管理 **Claude Code / Codex / Gemini CLI / OpenCode** 等多个 AI CLI 工具的认证
- 内置 **50+ 个 Provider 预设**（含 AWS Bedrock、NVIDIA NIM、常见中转站）
- 切换时**自动改写** `~/.claude/settings.json`
- 系统托盘一键切换
- 本地代理 + 自动 failover（一个 Provider 挂了切到另一个）

### 5.2 安装 CC Switch

1. 浏览器打开 **官方** GitHub Releases：
   ```
   https://github.com/farion1231/cc-switch/releases
   ```

2. 下最新版的 **Windows installer**（一般叫 `CC-Switch-x.x.x-Setup.exe` 或 `cc-switch_x.x.x_x64-setup.nsis.exe`）

3. 双击安装，全程默认下一步。装完会自动启动；不启动就从开始菜单找 CC Switch。

### 5.3 添加你的第一个 Provider

1. CC Switch 主界面左侧选 **Claude Code** 标签

2. 点 **Add Provider** / **添加供应商**

3. 按你的目标账号类型填：

   **A. Anthropic 官方订阅（Pro / Max）：**
   - Name：`claude-official` 或随便起
   - Preset：选 `Anthropic Official` / `claude-official`
   - Base URL：留空 或 `https://api.anthropic.com`
   - Token：**留空**（走 OAuth 登录）
   - 保存

   **B. Anthropic 官方 API Key：**
   - Preset：选 `Anthropic Official`
   - Base URL：`https://api.anthropic.com`
   - Token：你的 `sk-ant-api03-...`
   - 保存

   **C. 第三方中转站：**
   - Preset：先看下拉里有没有匹配的（如"智增增"、"API易"、"OneAPI" 等知名站）。有就选，**自动填好 Base URL**；没有就选 `Custom`
   - Base URL：中转站给的地址
   - Token：中转站给的 token
   - 保存

4. 重复 Step 3 加你所有的账号。

### 5.4 在 Provider 间切换

平时使用：

- **从系统托盘**点 CC Switch 图标 → 选 Provider → 切换，瞬间完成
- 或打开主界面 → 选目标 Provider → 点 **Activate** / **启用**

每次切换，CC Switch 会**自动改写** `~/.claude/settings.json`。下次 `claude` 启动就用新的。

### 5.5 用 CC Switch 时必须清掉环境变量

**这是最关键也最容易被忽视的一点。**

Claude Code 启动时的认证优先级是：

```
环境变量 ( ANTHROPIC_API_KEY / ANTHROPIC_AUTH_TOKEN + ANTHROPIC_BASE_URL )
   ↓ 优先级最高
~/.claude/settings.json （CC Switch 写入的）
   ↓
~/.claude/.credentials.json （OAuth 缓存）
```

**所以**：只要环境变量没清，**CC Switch 切了 100 次都没用**——Claude Code 永远先读环境变量，CC Switch GUI 显示和实际行为永远对不上。**这是 90% 的"CC Switch 不生效"问题的根因。**

清环境变量的命令（**PowerShell**，逐条跑）：

```powershell
[Environment]::SetEnvironmentVariable("ANTHROPIC_API_KEY",    $null, "User")
[Environment]::SetEnvironmentVariable("ANTHROPIC_AUTH_TOKEN", $null, "User")
[Environment]::SetEnvironmentVariable("ANTHROPIC_BASE_URL",   $null, "User")
```

如果以前是用图形界面设的，也可以走图形界面删：`Win + R` → `sysdm.cpl` → 高级 → 环境变量 → **用户变量里**把这三个找到删掉。**别忘了同时检查"系统变量"那一栏**，里面有同名变量会压制用户变量。

清完之后**关掉所有终端**（包括 Windows Terminal 整个进程）再重开。

---

## 六、自检：你机器上当前的认证状态

切账号之前，**先弄清楚现在用的是什么**。

### 6.1 第一步：看环境变量

```powershell
echo "API_KEY:    $env:ANTHROPIC_API_KEY"
echo "AUTH_TOKEN: $env:ANTHROPIC_AUTH_TOKEN"
echo "BASE_URL:   $env:ANTHROPIC_BASE_URL"
```

### 6.2 第二步：看 Claude Code 自报家门

```powershell
claude
```

进去后输：

```
/status
```

会显示当前 endpoint、模型、账号信息。

### 6.3 三种典型状态对照

| 环境变量状态 | `/status` Endpoint | 实际在用什么 |
|---|---|---|
| 三个全空 | `https://api.anthropic.com` | Claude.ai 官方 OAuth |
| 只 `ANTHROPIC_API_KEY` 有值 | `https://api.anthropic.com` | Anthropic 官方 API Key |
| `ANTHROPIC_AUTH_TOKEN` + `ANTHROPIC_BASE_URL` 都有值 | 中转站 URL | 第三方中转站（**正常用法**） |
| 三个都有值 + `BASE_URL` 不是 anthropic.com | 中转站 URL | 中转站（但变量乱，建议清理） |
| CC Switch GUI 显示一个 Provider，但 `/status` 显示另一个 | 看 `/status` 为准 | **环境变量在压制 CC Switch**，按 [5.5](#55-用-cc-switch-时必须清掉环境变量) 清环境变量 |

---

## 七、切换账号 / 中转站的标准流程

**两条路任选一条，不要混着用。**

### 路线 A：CC Switch 接管，环境变量退场（推荐）

适合：已经装了 CC Switch、有多个账号想随时切换。

**步骤：**

1. 在 CC Switch 里**确认目标 Provider 已配好**（看 [5.3](#53-添加你的第一个-provider)）
2. **清空所有 Anthropic 环境变量**（按 [5.5](#55-用-cc-switch-时必须清掉环境变量)）：
   ```powershell
   [Environment]::SetEnvironmentVariable("ANTHROPIC_API_KEY",    $null, "User")
   [Environment]::SetEnvironmentVariable("ANTHROPIC_AUTH_TOKEN", $null, "User")
   [Environment]::SetEnvironmentVariable("ANTHROPIC_BASE_URL",   $null, "User")
   ```
3. **关掉所有终端**，重开
4. **验证环境变量已清**：
   ```powershell
   echo "K:$env:ANTHROPIC_API_KEY  T:$env:ANTHROPIC_AUTH_TOKEN  U:$env:ANTHROPIC_BASE_URL"
   ```
   应该看到 `K:  T:  U:`（冒号后面全是空）
5. **CC Switch GUI 里 Activate** 目标 Provider
6. `claude` 启动 → `/status` 验证 Endpoint 是不是预期的

### 路线 B：完全不用 CC Switch，直接改环境变量

适合：只有一两个账号、嫌 GUI 麻烦、想极简管理。

按目标场景：

**B-1：切到另一个中转站：**
```powershell
setx ANTHROPIC_BASE_URL "https://新中转站地址"
setx ANTHROPIC_AUTH_TOKEN "sk-新TOKEN"

# 如果以前用过 ANTHROPIC_API_KEY，删掉（保留它会优先生效）
[Environment]::SetEnvironmentVariable("ANTHROPIC_API_KEY", $null, "User")
```

**B-2：切到 Anthropic 官方 API Key：**
```powershell
[Environment]::SetEnvironmentVariable("ANTHROPIC_BASE_URL",   $null, "User")
[Environment]::SetEnvironmentVariable("ANTHROPIC_AUTH_TOKEN", $null, "User")
setx ANTHROPIC_API_KEY "sk-ant-api03-你的KEY"
```

**B-3：切到 Claude.ai 订阅（OAuth）：**
```powershell
[Environment]::SetEnvironmentVariable("ANTHROPIC_BASE_URL",   $null, "User")
[Environment]::SetEnvironmentVariable("ANTHROPIC_AUTH_TOKEN", $null, "User")
[Environment]::SetEnvironmentVariable("ANTHROPIC_API_KEY",    $null, "User")
```

然后下一步登录走 `claude /login`，浏览器走 OAuth。

**所有路线 B 切换完都要：**

```
关闭所有终端 → 重开 → 启动 claude → 输入 /status 验证
```

---

## 八、最容易踩的"切了没生效"坑

按概率从高到低：

| 现象 | 真正原因 | 解法 |
|---|---|---|
| `setx` 跑完新开 PowerShell 还读到旧值 | **Windows Terminal 复用父进程**——多个 tab 共享同一个 conhost.exe；关闭单个 tab 不够，要关**整个进程** | 任务管理器（Ctrl+Shift+Esc）→ 找到 `WindowsTerminal.exe` 进程 → 结束任务 → 再开 |
| 用户变量删了，但还是读到旧值 | **系统变量**里有同名变量在压制 | `Win + R` → `sysdm.cpl` → 高级 → 环境变量 → **检查下半部分"系统变量"** → 删掉同名条目 |
| 环境变量全清了，CC Switch 也切了，`/status` 还是旧账号 | OAuth 凭据缓存在作怪 | 删 `~/.claude/.credentials.json`：<br>`Remove-Item -Force $HOME\.claude\.credentials.json`<br>然后凭据管理器（`control /name Microsoft.CredentialManager`）删 `anthropic` 条目 |
| CC Switch 显示 Activate 成功，Claude Code 启动报 401 | settings.json 写入失败（权限问题 / 杀软拦截 / 文件被占用） | 记事本手动打开 `~/.claude/settings.json` 看内容；不对就管理员身份开 CC Switch 重 Activate |
| 中转站切完用一会儿报 `model not found` | 中转站不支持你当前选的模型（比如 `opus-4-7` 或 `claude-sonnet-4-6`） | 在 Claude Code 里 `/model sonnet` 切回基础 sonnet；或换支持该模型的中转站 |
| 走 OAuth 但浏览器一直转圈 | 中国大陆很多 IP 段被 Anthropic 限制；或 DNS 污染 | 改用 API Key 或中转站；或把 DNS 改成 `1.1.1.1` / `8.8.8.8` 试试 |
| `setx` 命令本身报错 / 没回显 SUCCESS | PowerShell 权限不足 / 路径有中文 / 杀软拦截 | 用管理员身份开 PowerShell；命令外层加双引号包好；临时退杀软 |
| 切完 token 但所有请求 403 | 中转站后台你的 IP 被风控 | 联系中转站客服解封；或换 IP / 节点 |

---

## 九、中转站常见报错与解法

| 报错 | 含义 | 解法 |
|---|---|---|
| `401 Unauthorized` | Token 错 / 过期 / 被吊销 | 去中转站后台确认 token 状态；重新生成；重新 `setx` |
| `402 Payment Required` / `Insufficient balance` | 中转站账户没钱了 | 充值 |
| `403 Forbidden` | IP 被风控、套餐到期、模型权限不够 | 看错误详情；联系中转站客服 |
| `404` 模型不存在 | 当前模型不在中转站支持列表 | 切到 sonnet/haiku：`/model sonnet` |
| `429 Too Many Requests` | 限速 | 等 1-5 分钟；升级中转站套餐 |
| `500/502/503/504` | 中转站后端挂了 | 等几分钟；切换到备用中转站；联系客服 |
| `connection refused` / `ETIMEDOUT` | 中转站地址不通 | 浏览器试中转站后台能否访问；检查本机网络；尝试别的节点 |
| 返回乱码 / 不像 Claude 的回复 | 协议兼容型中转站，后端是非 Claude 模型 | 换直连转发型中转站 |
| MCP 不工作 / Skills 报错 | 中转站没实现这些 Anthropic 扩展 API | 这些高级功能换官方账号用 |
| Prompt Caching 不生效 / 没省钱 | 中转站不支持 / 没启用 caching | 这部分省钱去官方 |
| 1M context（Opus 4.7）报 context too long | 中转站不支持长上下文 | 换支持的中转站；或换官方 |

---

## 十、怎么彻底回到 Anthropic 官方

不想再用中转站了，想干干净净回到 Anthropic 官方？按顺序跑：

```powershell
# 1. 清所有 Anthropic 相关环境变量
[Environment]::SetEnvironmentVariable("ANTHROPIC_API_KEY",    $null, "User")
[Environment]::SetEnvironmentVariable("ANTHROPIC_AUTH_TOKEN", $null, "User")
[Environment]::SetEnvironmentVariable("ANTHROPIC_BASE_URL",   $null, "User")

# 2. 清 Claude Code 的 OAuth 凭据缓存
Remove-Item -Force $HOME\.claude\.credentials.json -ErrorAction SilentlyContinue

# 3. （可选）如果之前用 CC Switch 写过 settings.json，重置它
Remove-Item -Force $HOME\.claude\settings.json -ErrorAction SilentlyContinue

# 4. （可选）卸载 CC Switch：从「设置 → 应用」找到 CC Switch 卸载
Remove-Item -Recurse -Force $HOME\.cc-switch -ErrorAction SilentlyContinue
```

去 **凭据管理器**（`control /name Microsoft.CredentialManager`）→ Windows 凭据 → 删 `anthropic` / `claude-code` / `claude.ai` 字样的条目。

**关掉所有终端，重开一个 PowerShell：**

```powershell
claude /login
```

浏览器走 OAuth 重登 Anthropic 官方账号。`/status` 验证 Endpoint 是 `https://api.anthropic.com` 即成功。

---

## 十一、隐私与合规说明

### 11.1 你的代码 / Prompt 数据流向

| 路径 | 数据经过哪里 | 隐私级别 |
|---|---|---|
| **Anthropic 官方订阅 / API Key** | 你的电脑 → Anthropic 服务器 | **最高**——Anthropic 明确承诺**不**用 API/Console 数据训练模型 |
| **直连转发型中转站** | 你 → 中转站服务器 → Anthropic 服务器 | **中**——中转站老板能看到你所有数据；Anthropic 也能 |
| **协议兼容型中转站** | 你 → 中转站服务器（不去 Anthropic） | **低**——只信中转站老板的良心 |
| **Bedrock / Vertex** | 你 → AWS / GCP（你自己的云账户）→ Anthropic 模型 | **最高 + 合规**——数据在你的云上 |
| **本地 LLM（如 Ollama）** | 你 → 你的电脑（不出网） | **绝对**——完全不出本机 |

### 11.2 Anthropic 服务条款（ToS）重点

2026 年 2 月更新：

- **Free / Pro / Max 的 OAuth 凭据**：仅允许在 Claude Code 和 claude.ai 使用。**任何**第三方代理、自动化工具、转发服务都**违反 ToS**——包括 CC Switch 把订阅 OAuth 复用到别的 CLI 工具的功能。
- **Anthropic API Key**：可以在任何工具里使用（这是正经付费方式）
- **违规处置**：Anthropic 可能临时限速 → 暂停账号 → 终止订阅

### 11.3 给中文用户的合规建议

1. **个人学习 / 小项目**：用中转站没问题（运营方的合规风险由他们自己承担）
2. **工作 / 商业**：走 Anthropic 官方 API Key，或公司提供的 Bedrock / Vertex
3. **千万别**把 OAuth 凭据导出来塞到 CC Switch 的"Token 字段"里复用——这是 ToS 明确禁止的

---

## 十二、官方 vs 中转站决策表

| 你的情况 | 推荐 |
|---|---|
| 学生 / 个人 / 没有海外支付手段 | **直连转发型中转站**（挑口碑好的，永远只放个人项目） |
| 涉密代码 / 商业项目 / 大厂内网 | **必须**官方（订阅 / API Key / Bedrock-Vertex），不能碰中转站 |
| 有美卡 / 海外卡 / 留学生 | 直接 **Anthropic 官方 Pro 或 Max**（$20-100/月），最省心 |
| 完全偶尔用（一周 1-2 次） | **官方 API Key 按量付费**（充 $5 能用半年） |
| 同时管多个账号 / 多个工具 | 装 **CC Switch** 集中管理 |
| 数据合规要求极高（医疗、金融） | **Bedrock** 或 **Vertex AI**，配合公司云账户 |
| 想完全离线 / 无外网 | 本地 LLM（Ollama 等）+ `ANTHROPIC_BASE_URL=http://localhost:...`，但模型能力会断崖式下降 |

---

## 十三、信息来源与官方链接

### 官方文档
- Claude Code 主页：`https://code.claude.com/`
- Claude Code 认证文档：`https://code.claude.com/docs/en/authentication`
- Claude Code 配置文档：`https://code.claude.com/docs/en/settings`
- Anthropic Console：`https://console.anthropic.com/`
- Anthropic 隐私政策：`https://www.anthropic.com/legal/privacy`
- Anthropic 商业服务条款：`https://www.anthropic.com/legal/commercial-terms`

### CC Switch
- 项目主页（**唯一可信源**）：`https://github.com/farion1231/cc-switch`
- 官网：`https://ccswitch.io`
- 最新发布：`https://github.com/farion1231/cc-switch/releases`

### 本仓库相关文档
- [Claude Code 在 Windows 上从零开始的完整配置与使用指南](./Claude-Code-Windows-从零开始完整指南.md)
- [Codex CLI 在 Windows 上从零开始的完整配置与使用指南](./Codex-CLI-Windows-从零开始完整指南.md)
- [Codex CLI 个人 & 团体的费用估算与选档指南](./Codex-CLI-费用与选档指南.md)

---

> **免责声明：** 本文档列出的中转站只是泛指类型，**不为任何具体中转站背书**——选择中转站的风险由用户自负。所有命令在 2026-05-23 的 Claude Code 2.x 和 CC Switch v3.15 上验证过；版本演进可能引入变化，以官方文档为准。
