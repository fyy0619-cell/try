# Codex CLI 在 Windows 上从零开始的完整配置与使用指南

> 本文档面向**完全没用过命令行、没装过 Node.js、没碰过任何 AI 终端工具**的零基础读者。每一步都会给出**精确的命令**、**期望看到的输出**以及**遇到错误时该怎么办**。按顺序照着做即可。
>
> - 适用系统：Windows 10 / Windows 11（64 位）
> - 适用版本：Codex CLI 0.115 及以上（2026 年 5 月）
> - 默认模型：`gpt-5-codex`
> - 工程量预估：第一次完整跑下来约 **30–60 分钟**（取决于网速和是否已有 Node）

---

## 目录

- [一、先搞清楚 Codex CLI 是什么](#一先搞清楚-codex-cli-是什么)
- [二、动手前的硬性条件清单](#二动手前的硬性条件清单)
- [三、检查并安装 Node.js 22+](#三检查并安装-nodejs-22)
- [四、选定你的终端：PowerShell vs Git Bash](#四选定你的终端powershell-vs-git-bash)
- [五、安装 Codex CLI](#五安装-codex-cli)
- [六、首次登录（两种方式二选一）](#六首次登录两种方式二选一)
- [七、第一次跑 Codex（Hello World）](#七第一次跑-codexhello-world)
- [八、日常使用：模式、模型、常用命令](#八日常使用模式模型常用命令)
- [九、配置文件 `~/.codex/config.toml`](#九配置文件-codexconfigtoml)
- [十、常见问题与解法（FAQ）](#十常见问题与解法faq)
- [十一、卸载与彻底清理](#十一卸载与彻底清理)
- [十二、进阶资源](#十二进阶资源)

---

## 一、先搞清楚 Codex CLI 是什么

**Codex CLI** 是 OpenAI 推出的一个**跑在你自己电脑终端里**的轻量级编程 Agent。简单理解：

- 它**不是**网页版 ChatGPT，也**不是** OpenAI Playground。
- 它**是**一个命令行程序，启动后你可以用自然语言让它读你的代码、改你的文件、跑你的命令。
- 它运行在**沙盒**里——默认情况下，它只能动你当前目录里的文件，不能联网、不能改系统文件，每次写盘前还会问你一次。
- 收费模式：**包含在 ChatGPT Plus / Pro / Business / Edu / Enterprise 订阅里**（不另外收费）；也可以用**纯 OpenAI API Key** 按用量付费。

> **重要澄清：** npm 上有两个名字像但完全无关的包：
> - `@openai/codex` ← **这才是 OpenAI 官方的 Codex CLI**
> - `codex`（不带 `@openai/` 前缀）← 是一个 2012 年的无关老项目，装了没用
>
> 后文所有命令都用 `@openai/codex`，**别打错**。

---

## 二、动手前的硬性条件清单

逐项确认，缺一不可：

| # | 条件 | 怎么自查 |
|---|---|---|
| 1 | Windows 10 / 11 64 位 | 桌面右键「此电脑」→「属性」，查看"系统类型" |
| 2 | 管理员权限（至少能装软件） | 能不能装 QQ 微信？能就行 |
| 3 | 稳定的国际网络访问能力 | 能不能打开 `https://platform.openai.com/`？打不开请见 [FAQ Q9](#q9登录浏览器一直转圈无法回调到终端) |
| 4 | 一个 OpenAI 账号 | 没有的话去 `https://platform.openai.com/signup` 注册 |
| 5 | **二选一**：ChatGPT 付费订阅 **或** OpenAI API Key | 见[第六章](#六首次登录两种方式二选一) |
| 6 | 大约 1 GB 可用磁盘空间 | C 盘别接近爆满 |

> **关于网络：** Codex CLI 需要稳定访问 `api.openai.com`、`auth.openai.com`、`chatgpt.com`。国内直连这些域名通常**不通**，请确保你的网络环境（公司专线、合法代理等）能访问到 OpenAI 服务，否则后面所有步骤都会卡在登录。

---

## 三、检查并安装 Node.js 22+

Codex CLI 是用 Node.js 写的，**必须先装 Node.js 22 或更高版本**。

### 3.1 第一步：检查你是否已经装了 Node.js

按 `Win + R`，输入 `powershell`，回车，把下面这行**完整复制**进去再回车：

```powershell
node -v
```

可能出现的三种情况：

| 输出 | 含义 | 下一步 |
|---|---|---|
| `v22.x.x` 或 `v23.x.x`、`v24.x.x` 等 | 已经装好，且版本够 | 跳到 [3.4 验证 npm](#34-验证-npm) |
| `v18.x.x`、`v20.x.x` 等比 22 小的版本 | 装了但**版本太低** | 跟着 [3.2 用 nvm-windows](#32-推荐路线用-nvm-windows-管理-nodejs) 升级 |
| `node : 无法将"node"项识别...` 或 `'node' 不是内部或外部命令` | 还没装 | 任选 [3.2](#32-推荐路线用-nvm-windows-管理-nodejs) 或 [3.3](#33-备用路线直接装-nodejs-官方安装包) |

### 3.2 推荐路线：用 nvm-windows 管理 Node.js

**为什么推荐 nvm-windows：** 以后想升级、降级、装多版本只要一行命令；不污染系统 PATH；卸载干净。

**步骤：**

1. 浏览器打开 `https://github.com/coreybutler/nvm-windows/releases`
2. 找到最新版（不是 pre-release），下载 **`nvm-setup.exe`** 那个文件（约 2 MB）
3. 双击运行，**全程默认下一步**即可。安装路径默认是 `C:\Users\<你的用户名>\AppData\Roaming\nvm`，Node.js 软链接路径默认是 `C:\Program Files\nodejs`，**不要改**。
4. 安装完成后**关掉所有已打开的 PowerShell / cmd / Git Bash 窗口**，重新打开一个**新的** PowerShell，依次执行：

```powershell
nvm version
nvm install 22
nvm use 22
```

第一行应该输出 `1.1.x` 之类的版本号。第二行会下载 Node 22 的安装包并安装（这步可能要等 30 秒到几分钟，视网速）。第三行切换到 22。

5. 验证：

```powershell
node -v
npm -v
```

期望输出类似：

```
v22.11.0
10.9.0
```

> **常见坑 1：** `nvm use 22` 报 `exit status 1: 拒绝访问`。原因：PowerShell 不是管理员身份打开。解决：右键 PowerShell 图标 → **以管理员身份运行**，再执行 `nvm use 22`。
>
> **常见坑 2：** 装 nvm-windows 之前**已经装过**普通 Node.js。会导致 PATH 冲突。解决：先去「控制面板 → 程序和功能」卸载老的 Node.js，**重启电脑**，再装 nvm-windows。
>
> **常见坑 3：** `nvm install 22` 报 `Could not retrieve ...node.exe... 404`。原因：nvm 镜像被墙。解决：在 PowerShell 里设镜像源后重试：
> ```powershell
> nvm node_mirror https://npmmirror.com/mirrors/node/
> nvm npm_mirror https://npmmirror.com/mirrors/npm/
> nvm install 22
> ```

### 3.3 备用路线：直接装 Node.js 官方安装包

如果你不需要切换版本，也可以直接装：

1. 浏览器打开 `https://nodejs.org/`
2. 下载 **LTS** 那个按钮指向的 `.msi`（写着 `Recommended For Most Users`）。务必确认显示的是 **22.x.x** 或更高，否则去 `https://nodejs.org/en/download/releases/` 手动找 22+ 版本。
3. 双击运行，**勾上 `Add to PATH`**（默认已勾），全程下一步。
4. 安装完成后**关掉所有终端窗口重新开**，执行 `node -v` 验证。

### 3.4 验证 npm

```powershell
npm -v
```

应输出版本号（10 或更高）。如果报"npm 不是内部命令"——说明 PATH 没生效，**关掉终端重新开**，再试。还不行就重启电脑。

### 3.5 （强烈推荐）设置 npm 国内镜像加速

国内直连 `registry.npmjs.org` 很慢甚至超时。换成淘宝镜像：

```powershell
npm config set registry https://registry.npmmirror.com
npm config get registry
```

第二行应回显 `https://registry.npmmirror.com/`。

> **要回到官方源**就执行：`npm config set registry https://registry.npmjs.org`

---

## 四、选定你的终端：PowerShell vs Git Bash

| 终端 | 推荐度 | 说明 |
|---|---|---|
| **PowerShell**（系统自带） | **★★★★★** | Codex CLI 在 Windows 上的**原生最佳运行环境**，沙盒功能完整 |
| **Windows Terminal** | ★★★★★ | 等价于上面，只是外观更好；从 Microsoft Store 免费装 |
| Git Bash | ★★★ | 可以跑，但部分沙盒能力降级 |
| cmd（旧版命令提示符） | ★★ | 能跑，但提示渲染丑陋，不推荐 |
| WSL2（Linux 子系统） | ★★★★ | 用 Linux 沙盒，体验最稳。但是装 WSL 本身门槛偏高，本指南不展开 |

**本指南后续所有命令都以 PowerShell 为准。**

### 4.1 一次性修复：PowerShell 执行策略

PowerShell 默认禁止跑脚本，会导致 `codex` 启动时报 `无法加载文件 ... 在此系统上禁止运行脚本`。一劳永逸地修：

1. **以管理员身份**打开 PowerShell（开始菜单搜 PowerShell → 右键 → 以管理员身份运行）
2. 执行：

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

3. 看到提示输入 `Y` 回车。
4. 关闭管理员窗口，**用普通用户身份**重新开一个 PowerShell 继续下面的操作。

---

## 五、安装 Codex CLI

确认 `node -v` 是 22+ 之后，执行：

```powershell
npm install -g @openai/codex
```

期望看到类似：

```
added 1 package in 8s
```

> **绝对不要打成 `npm install -g codex`**——那是 2012 年的无关老包，装了没用。前缀 `@openai/` 必须有。

验证安装：

```powershell
codex --version
```

应输出类似 `codex-cli 0.115.0`。

### 5.1 安装阶段可能遇到的错误

| 错误信息 | 原因 | 解法 |
|---|---|---|
| `EACCES: permission denied` | 全局目录没写权限 | 用管理员身份开 PowerShell 重试 |
| `npm ERR! code ETIMEDOUT` / 卡住不动 | 网络问题 | 见 [3.5](#35-强烈推荐设置-npm-国内镜像加速) 配淘宝镜像 |
| `Unsupported engine ... required: node >=22` | Node 版本太低 | 回 [3.2](#32-推荐路线用-nvm-windows-管理-nodejs) 升级到 22 |
| `npm ERR! 404 Not Found - GET ...@openai/codex` | 包名打错或镜像不同步 | 确认是 `@openai/codex`；若用了淘宝镜像，临时切回官方源重装：<br>`npm install -g @openai/codex --registry=https://registry.npmjs.org` |
| `codex : 无法将"codex"项识别为...` | 全局 bin 不在 PATH | 执行 `npm config get prefix`，把输出路径加到系统 PATH，**重开终端**；nvm-windows 用户一般不会遇到这个 |

---

## 六、首次登录（两种方式二选一）

Codex CLI 必须登录后才能用。OpenAI 官方支持**两种登录方式**，**任选其一**：

| 方式 | 适合 | 计费 | 网络要求 |
|---|---|---|---|
| **A. 用 ChatGPT 账号登录（OAuth）** | 已经订阅了 ChatGPT Plus/Pro 等的用户 | **不另外扣钱**（用订阅额度） | 浏览器要能打开 `chatgpt.com` 完成 OAuth |
| **B. 用 OpenAI API Key 登录** | 没订阅 ChatGPT，或想精确按调用量付费 | **按 API tokens 付费**，需账户预存余额 | 终端能访问 `api.openai.com` 即可 |

> **没有 ChatGPT 订阅 ≠ 不能用。** 只是要走方式 B、自己掏 API 调用费。
>
> **ChatGPT 免费账户不行。** 走方式 A 必须是付费订阅。

### 6.1 方式 A：用 ChatGPT 账号登录

1. 在 PowerShell 里直接输入：

```powershell
codex
```

2. 第一次启动会出现一个交互菜单，用 ↑↓ 选择 **`Sign in with ChatGPT`**，回车。
3. 它会**自动打开你的默认浏览器**并跳到一个 OpenAI 登录页。如果没自动打开，终端会显示一个 URL，**把这个 URL 复制到浏览器手动打开**。
4. 浏览器里登录你订阅的那个 ChatGPT 账号、点授权。
5. 浏览器显示 "You can return to your terminal" 之类的字样后，回终端，已经登录成功。
6. 终端进入了 Codex 主界面（一个等你输入提示词的输入框）。先输入 `/exit` 回车退出，下一步我们再正式用。

> **如果浏览器一直转圈、回不到终端：** 见 [FAQ Q9](#q9登录浏览器一直转圈无法回调到终端)。

### 6.2 方式 B：用 OpenAI API Key 登录

#### Step 1：拿到 API Key

1. 浏览器打开 `https://platform.openai.com/api-keys`，登录。
2. 点 **`+ Create new secret key`**，给它起个名字（比如 `codex-cli`），**勾选 All 权限**，点 Create。
3. 弹出的 Key 形如 `sk-proj-...`，**只显示这一次！** 立刻**复制下来存到一个安全的地方**（比如本地的密码管理器）。
4. 顺便去 `https://platform.openai.com/settings/organization/billing/overview` 充点钱（至少 5 美金，否则调用会被 429）。

#### Step 2：把 Key 设进环境变量

在普通用户的 PowerShell 里执行（把 `sk-你的实际KEY` 替换成你刚才复制的那串）：

```powershell
setx OPENAI_API_KEY "sk-你的实际KEY"
```

`setx` 会把变量**永久**写入用户级环境变量。它**不会**对当前窗口生效，所以——

5. **关掉当前 PowerShell，重新开一个**。
6. 验证：

```powershell
echo $env:OPENAI_API_KEY
```

应该输出你刚才那串 `sk-...`。

#### Step 3：用 API Key 启动 Codex

```powershell
codex
```

启动菜单选 **`Use API key`** 或者**直接看到主界面**（Codex 检测到 `OPENAI_API_KEY` 环境变量后会自动用它）。

> **API Key 泄漏立刻去 `https://platform.openai.com/api-keys` 撤销那一条，再生成新的。**

---

## 七、第一次跑 Codex（Hello World）

### 7.1 准备一个测试目录

```powershell
mkdir D:\codex-test
cd D:\codex-test
```

> 路径**不要**包含中文、空格、特殊字符。`D:\桌面\我的项目` 会经常出怪事。

### 7.2 启动

```powershell
codex
```

终端进入 Codex 交互界面。你会看到一个输入框、底部一行状态栏（显示当前模型、当前沙盒模式、当前审批模式）。

### 7.3 输入第一个提示词

直接打字，回车提交：

```
帮我在当前目录创建一个名为 hello.py 的文件，里面写一段打印「Hello from Codex」的代码，然后运行它把输出告诉我。
```

Codex 会逐步：

1. **告诉你它打算做什么**（"我要创建 hello.py..."）
2. **请求批准写文件**——按 `y` + 回车（或直接 ↑/↓ 选 Approve 回车）批准。
3. **请求批准跑命令**（`python hello.py`）——同样按 `y` 批准。
4. **给你看输出**：`Hello from Codex`。

### 7.4 退出

输入：

```
/exit
```

或按 `Ctrl+C` 两次。

### 7.5 第一次跑可能踩的坑

| 现象 | 解释 | 解法 |
|---|---|---|
| 它请求批准跑 `python`，但你说没装 Python | 它跑了机器上还没有的程序 | 装 Python 或者改让它创建 `.txt` 文件 |
| 弹出 "Enable experimental sandbox" 询问 | Windows 沙盒首次启用 | 按 Enter 接受，**重启 codex** |
| 报 `windows sandbox: CreateProcessWithLogonW failed: 5` | 沙盒未完成 onboarding | 见 [FAQ Q3](#q3-windows-沙盒报-createprocesswithlogonw-failed-5) |
| 报 `PowerShell host failed ... 8009001d` | PowerShell 托管运行环境异常 | 见 [FAQ Q4](#q4-报-8009001d-powershell-host-错误) |

---

## 八、日常使用：模式、模型、常用命令

### 8.1 启动时的常用命令行参数

```powershell
codex                                      # 默认：交互模式，GPT-5-Codex，workspace-write 沙盒，按需审批
codex --model gpt-5-codex-mini              # 用更便宜更快的小模型
codex -m gpt-5                              # 用通用 GPT-5
codex --full-auto "把这个项目跑一下单元测试"   # 全自动模式：不再每条命令问你
codex --ask-for-approval never              # 永不询问审批（危险，仅信任仓库用）
codex -s read-only                          # 只读沙盒：它只能看不能改
codex --dangerously-bypass-approvals-and-sandbox "deploy"  # 关掉一切防护（仅 CI / 临时使用）
```

### 8.2 进入交互后的内置 Slash 命令

在 Codex 输入框里输入这些命令并回车：

| 命令 | 作用 |
|---|---|
| `/help` | 列出所有可用命令 |
| `/model` | 切换当前会话用的模型 |
| `/permissions` | 临时调整审批严格度（1=最严，3=完全放行） |
| `/clear` | 清空当前会话上下文（节省 token） |
| `/exit` | 退出 Codex |

### 8.3 沙盒模式速查

| 沙盒模式 | 它能做什么 | 它不能做什么 |
|---|---|---|
| `read-only` | 读当前目录所有文件 | 改文件、跑命令、联网 |
| `workspace-write`（默认） | 读 + 改当前目录文件、跑被批准的命令 | 改目录外的文件、默认不能联网 |
| `danger-full-access` | 全部 | 没限制（谨慎使用） |

---

## 九、配置文件 `~/.codex/config.toml`

第一次跑过 `codex` 之后，会自动在 `C:\Users\<你的用户名>\.codex\` 下创建配置目录。你可以手动新建一个 `config.toml`（如果还没有）：

```powershell
notepad $HOME\.codex\config.toml
```

`notepad` 会问"找不到文件，要不要新建"，点是。

### 9.1 一份适合大多数人的基础配置

把下面整段贴进去保存：

```toml
# 默认模型
model = "gpt-5-codex"
model_provider = "openai"

# 默认沙盒：可读写当前目录、命令默认要审批
sandbox_mode = "workspace-write"
approval_policy = "on-failure"

# 让模型多花一点思考时间
model_reasoning_effort = "high"

# 允许沙盒访问网络（用于装包、调 API）。如果你想严格离线，把 true 改为 false
[sandbox_workspace_write]
network_access = true
```

### 9.2 多 Profile（场景切换）

需要随时在「严格只读」「快速跑」「全自动」之间切换？把下面追加到 `config.toml`：

```toml
experimental_use_profile = true

[profiles.safe]
sandbox_mode = "read-only"
approval_policy = "on-failure"

[profiles.quick]
sandbox_mode = "workspace-write"
approval_policy = "never"
model_reasoning_effort = "low"

[profiles.yolo]
sandbox_mode = "danger-full-access"
approval_policy = "never"
```

启动时选：

```powershell
codex --profile safe
codex --profile quick
codex --profile yolo
```

---

## 十、常见问题与解法（FAQ）

### Q1：`codex` 报 `'codex' 不是内部或外部命令`

**原因：** Codex 装上了，但全局 npm bin 不在 PATH。

**解法（按顺序试）：**

1. 关掉所有终端窗口，**重新打开**一个 PowerShell，再试 `codex --version`。
2. 还不行：`npm config get prefix`，记下输出路径（比如 `C:\Users\xxx\AppData\Roaming\npm`），把它加到「环境变量 → 用户变量 → Path」里，确定，**重启电脑**。
3. 还不行：用 `npm root -g` 看全局模块目录，确认 `@openai/codex` 真的在那；不在就再 `npm install -g @openai/codex`。

### Q2：PowerShell 启动 codex 报 `因为在此系统上禁止运行脚本`

**原因：** 执行策略限制。

**解法：** 见 [4.1](#41-一次性修复powershell-执行策略)，以管理员开 PowerShell 跑：

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

### Q3：Windows 沙盒报 `CreateProcessWithLogonW failed: 5`

**解法：**

1. 编辑 `C:\Users\<你>\.codex\config.toml`，加：
   ```toml
   experimental_windows_sandbox = false
   ```
2. 重新跑 `codex`。
3. 会提示 "Enable experimental sandbox"，**接受**。
4. 退出 codex，把刚加的 `experimental_windows_sandbox = false` **删掉**或改成 `true`。
5. 再次启动 `codex`，沙盒应已正常。

### Q4：报 `8009001d` PowerShell host 错误

**现象：** `Loading managed Windows PowerShell failed with error 8009001d`，Codex 跑不了任何 shell 命令。

**原因：** 系统 PowerShell 托管运行时损坏，常见于公司域控环境。

**解法：**

1. 用普通终端跑 `powershell -Command "$PSVersionTable"`，确认本地 PowerShell 本身工作正常。
2. 在 `~/.codex/config.toml` 里**先试切换沙盒**：
   ```toml
   experimental_windows_sandbox = false
   ```
3. 如果还不行，最稳的退路是装 WSL2（`wsl --install`）然后在 WSL 里用 Codex。
4. 持续关注 `https://github.com/openai/codex/issues/13917` 的官方修复进度。

### Q5：沙盒模式下 PowerShell 报 `Cannot set property. Property setting is supported only on core types in this language mode`

**原因：** 沙盒里的 PowerShell 处于受限语言模式（Constrained Language Mode），某些命令需要的属性设置被禁了。

**解法：** 在 Codex 里输入 `/permissions`，调到 Level 3（Full Access）。**注意** Level 3 会绕过沙盒，仅在你完全信任仓库时用。

### Q6：装 codex 时报 `Unsupported engine`

**原因：** Node 版本低于 22。

**解法：** 回 [3.2](#32-推荐路线用-nvm-windows-管理-nodejs) 升级。

### Q7：登录提示 `Sign-in failed` 或浏览器跳转后报错

**可能原因：**

1. 你的 ChatGPT 账号是免费账户 → 升级到 Plus 或改用 API Key（[方式 B](#62-方式-bopenai-api-key-登录)）。
2. 浏览器登录的是别的账号 → 退出所有 OpenAI 账号，只留付费那个再试。
3. 公司代理在做 TLS 拦截 → 浏览器 OAuth 完不成，**改用 API Key**。

**强制清空登录态再登：**

```powershell
codex logout
codex
```

ChatGPT OAuth 凭据存在 **Windows 凭据管理器**（控制面板 → 用户账户 → 凭据管理器 → Windows 凭据），找含 `codex` 或 `openai` 的条目可以手动删掉。

### Q8：跑着跑着报 `rate_limit_exceeded` / 429

**原因：** 单位时间请求太多。

**解法：**

- 稍等 30 秒到几分钟自动恢复。
- 用 API Key 计费的：去 `https://platform.openai.com/settings/organization/limits` 看当前层级，必要时充钱升 Tier。
- 别在循环里调用 Codex。

### Q9：登录浏览器一直转圈、无法回调到终端

**典型原因：** 网络不通 `auth.openai.com` 或回调本地 `127.0.0.1:1455` 被防火墙拦。

**解法：**

1. 浏览器手动打开 `https://platform.openai.com/`，确认能正常加载（页面能登录），证明出口网络 OK。
2. 关闭 Windows Defender 防火墙、第三方杀软对终端的拦截，**重启 codex 重试**。
3. 走不通就改用 **API Key**（[方式 B](#62-方式-bopenai-api-key-登录)），不依赖浏览器回调。

### Q10：`context length exceeded`

**原因：** 单次对话累积太长。

**解法：** 在 Codex 里输入 `/clear` 清空上下文；或者关掉重开。需要长项目记忆的话，把关键背景写进项目根目录的 `AGENTS.md`，Codex 会自动读。

### Q11：能不能不联网用？

**部分可以。** 模型推理走 OpenAI 服务器，**这部分必须联网**。但沙盒里跑的命令本身可以纯本地。把 `sandbox_workspace_write.network_access` 设成 `false` 就能阻止 Codex 替你联网（比如跑 `pip install`）。

### Q12：和我熟悉的 ChatGPT 网页版有啥区别？

- ChatGPT 网页：**纯聊天**，不能动你本地文件。
- Codex CLI：**直接读改你本地代码、跑你本地命令**，有沙盒保护，更接近"AI 帮你写代码"的工作流。
- 订阅是**同一份**，互不冲突。

### Q13：可以同时用 Claude Code 和 Codex CLI 吗？

可以。两个工具互不干涉，安装路径、配置目录都不一样（Claude Code 在 `~/.claude/`，Codex 在 `~/.codex/`）。

### Q14：误操作让 Codex 删了文件能恢复吗？

- 如果你的工程在 git 仓库里：`git status` → `git restore <file>` 或 `git reset --hard HEAD`。
- 如果没在 git 里：Windows 回收站找一下；可能没进回收站（命令行 `rm` 不进回收站）。
- **教训：** 重要目录养成 `git init` 习惯，宁可多 commit 也别裸跑 `--full-auto`。

---

## 十一、卸载与彻底清理

如果你想完全移除 Codex CLI：

```powershell
npm uninstall -g @openai/codex
```

然后手动清理：

```powershell
Remove-Item -Recurse -Force $HOME\.codex
```

如果用了 ChatGPT 登录，去**凭据管理器**删掉含 `openai`/`codex` 的条目。

最后验证：

```powershell
codex --version
```

应报"未识别的命令"——卸载干净了。

---

## 十二、进阶资源

- 官方 CLI 主页：`https://developers.openai.com/codex/cli`
- 官方 Windows 专页：`https://developers.openai.com/codex/windows`
- 源码与 issue 跟踪：`https://github.com/openai/codex`
- npm 包页面：`https://www.npmjs.com/package/@openai/codex`
- 历史 Release 日志：`https://github.com/openai/codex/releases`

学有余力可以读这些：

- `AGENTS.md` 写法（怎么把项目规范喂给 Codex）
- `~/.codex/config.toml` 完整字段表（在源码 `codex-rs/core/src/config_types.rs`）
- 用 `codex exec` 把 Codex 当 SDK 嵌进自动化脚本

---

> 本文档随 Codex CLI 版本演进会过时。看到本指南某条命令报错时，先用 `codex --version` 对照官方 Release 日志确认版本差异，再回头核对本文是否需要更新。
>
> 写于 2026-05-23，对应 Codex CLI 0.115.x。
