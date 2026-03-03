---
name: openclaw-remote-deploy
description: 远程协助朋友在 Windows/Linux/Mac 安装并跑通 OpenClaw 的实战流程，包含各平台详细安装步骤、模型配置、Telegram/Discord/飞书渠道接入、常见报错排查清单。
read_when:
  - 用户要你远程指导部署 OpenClaw
  - 用户要做新手保姆级安装支持
  - 用户遇到安装报错，需要快速排错
  - 用户要接入飞书/Telegram/Discord 机器人渠道
  - 用户要配置 AI 模型（OpenAI/Gemini/DeepSeek/本地模型）
allowed-tools: Bash, Browser
---

# OpenClaw 远程部署技能（中文实战版）

## 目标

- 30-60 分钟内，让新手完成安装并跑通第一个可用 Agent
- 保证可复现：每一步都有检查点和回滚方式
- 减少常见失败：网络、权限、账号登录、模型配置

---

## 一、部署前预检（必须先做）

### 1) 环境矩阵

| 条件 | 要求 |
|------|------|
| 操作系统 | Windows 10/11、Ubuntu 22.04+、macOS 10.15+ |
| Node.js | **v22 或更高**（v18/v20 均不支持，会启动失败） |
| 网络 | 全程可访问 GitHub、npm、OpenClaw、模型服务 |
| 账号 | ClawHub 已注册并可登录 |
| 聊天渠道 | 至少准备 1 个（Telegram/Discord/Feishu/WhatsApp） |
| 模型 API | 至少 1 个可用 key（Gemini/OpenAI/DeepSeek 等） |

### 2) 避坑原则

- Linux **不要用 root** 直接长期运行，创建普通用户部署
- 遇到报错先看日志，不要盲删配置
- 任何"安装成功"都要过健康检查，不以命令返回码为准
- 新手只接 **1 个渠道 + 1 个模型**，先跑通再扩展

---

## 二、各平台安装步骤

### macOS 安装

```bash
# 方法一：官方脚本（推荐）
curl -fsSL https://openclaw.ai/install.sh | bash

# 方法二：npm 全局安装
npm install -g openclaw@latest

# 安装后初始化
openclaw onboard
```

> **Apple Silicon 用户注意**：如果报 `sharp` 编译失败，执行：
> ```bash
> export SHARP_IGNORE_GLOBAL_LIBVIPS=1
> npm install -g openclaw@latest
> ```

检查点：
- `openclaw --version` 有输出
- `openclaw gateway status` 正常返回

---

### Linux 安装

```bash
# 第一步：确保 Node.js >= 22（推荐用 nvm）
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc
nvm install 22
nvm use 22
node -v   # 确认 v22.x.x

# 第二步：安装 OpenClaw
curl -fsSL https://openclaw.ai/install.sh | bash
# 或
npm install -g openclaw@latest

# 第三步：初始化（--install-daemon 让网关开机自启）
openclaw onboard --install-daemon
```

> **权限报错（EACCES）解决**：不要用 sudo，改 npm 全局目录：
> ```bash
> mkdir ~/.npm-global
> npm config set prefix '~/.npm-global'
> echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
> source ~/.bashrc
> npm install -g openclaw@latest
> ```

检查点：
- `openclaw status` 显示已登录
- `systemctl status openclaw-gateway`（Linux systemd）运行中

---

### Windows 安装

**方法一：PowerShell 一键安装（推荐）**

```powershell
iwr -useb https://openclaw.ai/install.ps1 | iex
```

**方法二：CMD 安装**

```cmd
curl -fsSL https://openclaw.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

**方法三：WSL2 安装（更稳定，推荐有 Linux 基础的用户）**

```powershell
# 先安装 WSL2
wsl --install
# 进入 WSL 后按 Linux 步骤操作
```

> **Windows 注意事项**：
> - `gateway install` 失败提示 schtasks 不可用时，改用 `openclaw gateway start`
> - WSL2 中操作文件建议在 `~/projects` 等 Linux 原生目录，不要在 `/mnt/c/` 下
> - 若 npm 报编译错误，需安装：Python 3、Git、Visual Studio Build Tools、CMake

检查点：
- `openclaw --version` 输出版本号
- 浏览器访问 `http://127.0.0.1:18789/` 能打开控制台

---

## 三、登录与基础配置

```bash
openclaw login       # 登录 ClawHub 账号
openclaw status      # 查看整体状态
openclaw dashboard   # 打开网页控制台
```

检查点：
- 显示已登录
- 状态中无红色 ERROR

---

## 四、安装核心技能

```bash
clawhub login
clawhub search feishu       # 搜索飞书技能
clawhub search telegram
clawhub search discord
# 找到后安装
clawhub install <技能名>
```

检查点：
- 技能目录存在
- `openclaw status` 不报技能加载错误

---

## 五、模型配置

### 通用原则

- 只配 1 个稳定模型先跑通，再扩展
- 配置文件路径：
  - macOS/Linux：`~/.openclaw/openclaw.json`
  - Windows：`%USERPROFILE%\.openclaw\openclaw.json`

### OpenAI / GPT

```bash
openclaw config set -- models.providers[0].provider "openai"
openclaw config set -- models.providers[0].api_key "sk-xxx"
openclaw config set -- models.providers[0].model.id "gpt-4o"
```

或直接编辑 `openclaw.json`：

```json
{
  "models": {
    "providers": [
      {
        "provider": "openai",
        "api_key": "sk-xxx",
        "model": { "id": "gpt-4o", "name": "GPT-4o" }
      }
    ]
  }
}
```

### Google Gemini

```json
{
  "provider": "google",
  "base_url": "https://generativelanguage.googleapis.com/v1beta/openai",
  "api": "openai-completions",
  "api_key": "你的Gemini API Key",
  "model": { "id": "gemini-2.0-flash", "name": "Gemini 2.0 Flash" }
}
```

### DeepSeek（硅基流动）

```json
{
  "provider": "siliconflow",
  "base_url": "https://api.siliconflow.cn/v1",
  "api": "openai-completions",
  "api_key": "你的SiliconFlow API Key",
  "model": { "id": "deepseek-ai/DeepSeek-V3", "name": "DeepSeek-V3" }
}
```

### MiniMax（Anthropic 协议）

```json
{
  "provider": "minimax",
  "base_url": "https://api.minimaxi.com/anthropic",
  "api": "anthropic-messages",
  "api_key": "你的MiniMax API Key",
  "model": { "id": "MiniMax-M2.5", "name": "MiniMax M2.5" }
}
```

### Ollama 本地模型（完全私有，无需 API Key）

```bash
# 先安装 Ollama 并拉取模型
ollama pull llama3
```

```json
{
  "provider": "ollama",
  "base_url": "http://127.0.0.1:11434",
  "api": "openai-completions",
  "model": { "id": "llama3", "name": "Llama3 Local" }
}
```

检查点：

- 发送测试消息"你好，回复 ok"，机器人有响应
- 响应延迟在可接受范围（本地模型 5-30 秒正常）

---

## 六、渠道接入

### Telegram 接入

**第一步：创建 Bot**

1. Telegram 搜索 `@BotFather`，发送 `/newbot`
2. 按提示填写 Bot 名称和用户名
3. 保存返回的 **Bot Token**（形如 `123456:ABCdef...`）

**第二步：配置 OpenClaw**

```bash
openclaw config set channels.telegram.enabled true
openclaw config set channels.telegram.botToken '你的BOT_TOKEN'
openclaw config set channels.telegram.dmPolicy pairing
```

**第三步：配对用户**

向 Bot 发任意消息，Bot 回复配对码后执行：

```bash
openclaw pairing approve telegram 配对码
```

**群组注意事项**：
- Bot 默认开启隐私模式，只能看到 `/` 命令消息
- 如需 Bot 看全部群消息，将 Bot 设为群管理员
- 切换隐私模式后需将 Bot 移出再重新加入群组

---

### Discord 接入

**第一步：创建 Discord Bot**

1. 前往 [Discord 开发者门户](https://discord.com/developers)
2. 创建新应用 → 添加 Bot 用户 → 复制 Bot Token
3. 在 Privileged Gateway Intents 下启用：
   - ✅ **Message Content Intent**（必须）
   - ✅ Server Members Intent（推荐）

**第二步：配置 OpenClaw**

```bash
openclaw config set channels.discord.enabled true
openclaw config set channels.discord.botToken '你的BOT_TOKEN'
openclaw config set channels.discord.dmPolicy pairing
```

**DM 策略说明**：

| 策略 | 说明 |
|------|------|
| `pairing`（默认） | 未知用户收到配对码，批准后可聊天 |
| `allowlist` | 只有白名单用户 ID 可发消息 |
| `open` | 任何人都能收到回复（谨慎使用） |
| `disabled` | 完全关闭私信 |

---

### 飞书（Feishu）专项接入指南

> 飞书支持**长连接（WebSocket）模式**，不需要公网服务器/内网穿透，本地部署即可直接接入。

**第一步：创建飞书开放平台应用**

1. 打开 [飞书开放平台](https://open.feishu.cn)，登录后点击「开发者后台」
2. 点击「创建企业自建应用」，填写应用名称和描述
3. 在左侧「应用能力」→「添加应用能力」，选择并添加**机器人**

**第二步：获取 App ID 和 App Secret**

在应用管理页，左侧导航「凭证与基础信息」，复制保存：

- **App ID**（形如 `cli_xxxxxxxxxx`）
- **App Secret**（点击眼睛图标或复制按钮）

**第三步：开通必要权限**

在「权限管理」中搜索并开通以下权限（应用身份）：

| 权限名 | 权限标识 | 说明 |
|--------|----------|------|
| 获取与发送单聊消息 | `im:message` | 收发私聊 |
| 发送消息 | `im:message:send_as_bot` | 群聊发消息 |
| 获取与更新群组信息 | `im:chat` | 群组管理 |

**第四步：配置事件订阅（长连接模式）**

1. 左侧「事件与回调」→「事件配置」
2. 点击「订阅方式」编辑，选择**「使用长连接接收事件（WebSocket 模式）」**
   - 不需要填写服务器 URL
   - 若提示"未建立长连接"，先完成第五步再回来保存
3. 点击「添加事件」，添加：
   - `im.message.receive_v1`（接收消息，必须）
   - `im.chat.member.bot.added_v1`（加入群聊，可选）

**第五步：在 OpenClaw 中配置飞书凭证**

```bash
# 交互式添加渠道（推荐）
openclaw channels add
# 选择 Feishu，依次填入 App ID 和 App Secret

# 或命令行直接配置
openclaw config set -- channels.feishu.appId "你的AppID"
openclaw config set -- channels.feishu.appSecret "你的AppSecret"

# 重启网关
openclaw gateway restart

# 查看日志确认连接
openclaw logs --follow
# 看到 feishu ws connected 即成功
```

**第六步：发布飞书应用版本（关键！不发布不生效）**

1. 飞书开放平台左侧「版本管理与发布」
2. 点击「创建版本」→ 填写版本号 → 提交
3. 企业自建应用通常自动审核通过

**飞书接入检查清单**：

- [ ] 飞书应用已创建，机器人能力已添加
- [ ] App ID 和 App Secret 已填入 OpenClaw（无多余空格）
- [ ] `im:message` 等权限已开通
- [ ] 事件订阅已切换为**长连接模式**
- [ ] `im.message.receive_v1` 事件已添加
- [ ] OpenClaw 日志中出现 `feishu ws connected`
- [ ] 飞书应用已发布最新版本
- [ ] 私聊机器人能正常回复

---

## 七、安全警告

> 这是最容易被忽视、后果最严重的部分

- OpenClaw 属于实验性软件，**不要安装在含有敏感数据的设备上**
- **不要以 root 用户运行**
- **不要将 Gateway 端口（默认 18789）直接暴露在公网**，已知超 1300+ 实例裸奔，攻击者可执行任意命令
- 正确做法：`gateway.mode` 设为 `local`，只允许 `127.0.0.1`，公网访问用 Cloudflare Tunnel 或 Nginx 反向代理 + 严格认证
- 安装社区技能前先用 Cisco Skill Scanner 扫描，部分技能含恶意代码

---

## 八、高价值经验

- 教程要"截图+步骤+检查点"，纯命令列表新手会卡住
- ClawHub 登录是关键，不登录会触发限速或安装失败
- 网络问题占多数，尤其是依赖下载和登录回调阶段
- 权限要提前讲清：不给权限，很多自动化功能必然失败
- 新手部署不要同时接多个渠道，先单通道稳定后再扩容
- **飞书长连接模式无需公网 IP**，是最适合个人/内网部署的选项
- 国内服务器连 Telegram/Discord 需要代理；飞书是国内服务无需代理

---

## 九、常见故障排查

### 安装类

#### Node.js 版本不符（占安装失败约 60%）

```bash
node -v   # 必须 >= v22
# 用 nvm 切换
nvm install 22 && nvm use 22
```

#### npm 权限报错（EACCES）

```bash
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
npm install -g openclaw@latest
```

#### npm install failed / sharp 编译失败（Apple Silicon）

```bash
export SHARP_IGNORE_GLOBAL_LIBVIPS=1
npm cache clean --force
npm install -g openclaw@latest
```

#### 网络超时 / 镜像源问题（国内用户）

```bash
npm config set registry https://registry.npmmirror.com
npm install -g openclaw@latest
```

#### 通用清除重装流程

```bash
npm cache clean --force
npm uninstall -g openclaw
npm install -g openclaw@latest
openclaw doctor   # 自动诊断并修复
```

#### Windows gateway install 失败

```bash
# 不用 install，直接 start
openclaw gateway start
```

---

### 模型类

#### 模型可用但无回复

- 检查 API key 是否过期或额度耗尽
- 用最小 prompt 测试：发"ping"，看能否收到任何回复
- `openclaw logs --follow` 查看具体错误

#### 模型切换后不生效

- 重启网关：`openclaw gateway restart`
- 确认 `openclaw.json` 语法正确（JSON 不允许注释和尾逗号）

---

### 渠道类

#### Telegram Bot 收不到群消息

- 检查 Bot 隐私模式（默认只收 `/` 命令）
- 将 Bot 设为群管理员，或关闭隐私模式后移出再加入

#### Discord Bot 无法读取消息

- 确认在开发者门户开启了 **Message Content Intent**
- 重新邀请 Bot 到服务器

#### 飞书机器人收不到消息

- 检查事件订阅是否选了**长连接模式**（不是 HTTP 回调）
- 检查 `im.message.receive_v1` 是否已添加
- `openclaw logs --follow` 确认有 `feishu ws connected`
- 检查飞书应用是否已**发布最新版本**

#### 飞书"长连接未建立"提示

- 正常现象：OpenClaw 网关需先跑起来才能建立连接
- 操作顺序：先在 OpenClaw 配好凭证并启动网关 → 再回飞书保存事件配置

#### 飞书状态正常但不响应

- 确认应用「已启用」且已发布
- 「权限管理」确认 `im:message` 为「已开通」非「待审核」
- 将机器人从群里移除再重新添加

#### 国内服务器连 Telegram/Discord 超时

```bash
# 测试连通性
curl -s --connect-timeout 3 https://api.telegram.org/bot<TOKEN>/getMe
curl -s --connect-timeout 3 https://discord.com/api/v10/users/@me -H "Authorization: Bot <TOKEN>"
# 超时则需配置代理
```

---

## 十、远程协助话术模板

### 开场

"我会按 5 步带你跑通：安装、登录、技能、模型、渠道。每步都先过检查点，不盲跳。"

### 卡住时

"先别重装，把这一步完整报错贴我。我先定位是网络、权限还是配置问题，再给你最小修复动作。"

### 收尾

"现在你已经能稳定收发消息了。下一步再做双渠道、自动化任务、定时提醒，不要一口气全开。"

---

## 十一、验收标准（Done 定义）

- [ ] OpenClaw 已安装，`openclaw --version` 有输出
- [ ] ClawHub 已登录，至少 1 个技能可用
- [ ] 至少 1 个模型可正常响应（发消息有回复）
- [ ] 至少 1 个聊天渠道已收发成功（飞书/Telegram/Discord 均可）
- [ ] Gateway 已设为开机自启（daemon 模式）
- [ ] 用户知道如何看日志和常见报错如何自查
- [ ] 控制台 `http://127.0.0.1:18789/` 可以正常访问

---

## 十二、常用命令速查

```bash
openclaw                    # 启动 OpenClaw
openclaw onboard            # 初始化向导
openclaw dashboard          # 打开网页控制台
openclaw status             # 查看整体状态
openclaw config             # 查看/修改配置
openclaw logs --follow      # 实时日志
openclaw gateway start      # 启动后台网关
openclaw gateway stop       # 停止网关
openclaw gateway restart    # 重启网关
openclaw gateway status     # 查看网关状态
openclaw skills             # 管理技能
openclaw channels add       # 添加渠道
openclaw update             # 检查并更新
openclaw doctor             # 自动诊断并修复
openclaw pairing approve <platform> <code>  # 批准用户配对
```

---

## 十三、进阶扩展

- 多模型任务分配：Claude 处理复杂推理，Gemini Flash 处理快速查询，Gateway 自动路由
- 飞书进阶：配置多租户、企业审批流、飞书卡片消息格式
- 安全加固：Cloudflare Tunnel + 反向代理，隐藏 Gateway 端口
- 自动化：定时任务、跨渠道消息转发、Webhook 触发
- 给不同系统做 SOP：Windows/Linux/macOS 各一份
- 沉淀高频报错固定排障树
