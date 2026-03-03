---
name: openclaw-remote-deploy
description: 远程协助朋友在 Windows/Linux/Mac 安装并跑通 OpenClaw 的实战流程，包含常见报错与避坑清单。支持 Telegram / Discord / 飞书（Feishu）渠道全流程接入。
read_when:
  - 用户要你远程指导部署 OpenClaw
  - 用户要做新手保姆级安装支持
  - 用户遇到安装报错，需要快速排错
  - 用户要接入飞书机器人渠道
allowed-tools: Bash, Browser
---

# OpenClaw 远程部署技能（中文实战版）

## 目标

- 30-60 分钟内，让新手完成安装并跑通第一个可用 Agent
- 保证可复现：每一步都有检查点和回滚方式
- 减少常见失败：网络、权限、账号登录、模型配置

## 部署前预检（必须先做）

### 1) 环境矩阵

- 操作系统：Windows 10/11、Ubuntu 22.04+、macOS
- 网络：全程可访问 GitHub、npm、OpenClaw、模型服务
- 账号：ClawHub 已注册并可登录
- 聊天渠道：至少准备 1 个（Telegram/Discord/Feishu/WhatsApp）
- 模型 API：至少 1 个可用 key（Gemini/OpenAI/Minimax 等）

### 2) 避坑原则

- Linux 不要用 root 直接长期运行，创建普通用户部署
- 遇到报错先看日志，不要盲删配置
- 任何"安装成功"都要过健康检查，不以命令返回码为准

## 标准安装流程

### 步骤 A：安装 OpenClaw

```bash
curl -fsSL openclaw.ai/install.sh | bash
```

检查点：

- `openclaw --version` 有输出
- `openclaw status` 能正常返回

### 步骤 B：登录与基础配置

```bash
openclaw login
openclaw status
```

检查点：

- 显示已登录
- 网关可启动，模型配置为空或待配置均可

### 步骤 C：安装核心技能

```bash
clawhub login
clawhub search telegram
clawhub search discord
clawhub search feishu
```

按需安装后检查：

- 技能目录存在
- `openclaw status` 不报技能加载错误

### 步骤 D：配置模型

最小可用策略：只接 1 个稳定模型，先跑通再扩展。

检查点：

- 使用一次简单问答测试（如"你好，回复 ok"）
- 响应延迟和错误率在可接受范围

### 步骤 E：打通一个聊天渠道

优先 Telegram（新手最常见），或参考下方飞书专项接入指南。

检查点：

- 机器人能收消息
- 在会话里能正常回复
- 群聊中不会刷屏（只在需要时发言）

---

## 飞书（Feishu）专项接入指南

> 飞书支持**长连接（WebSocket）模式**，不需要公网服务器/内网穿透，本地部署即可直接接入。

### 第一步：创建飞书开放平台应用

1. 打开 [飞书开放平台](https://open.feishu.cn)，登录后点击「开发者后台」
2. 点击「创建企业自建应用」，填写应用名称和描述
3. 在左侧「应用能力」→「添加应用能力」，选择并添加**机器人**

### 第二步：获取 App ID 和 App Secret

在应用管理页，左侧导航「凭证与基础信息」，复制保存：

- **App ID**（形如 `cli_xxxxxxxxxx`）
- **App Secret**（点击眼睛图标或复制按钮）

### 第三步：开通必要权限

在「权限管理」中搜索并开通以下权限（均选应用身份）：

| 权限名 | 权限标识 | 说明 |
|--------|----------|------|
| 获取与发送单聊消息 | `im:message` | 收发私聊 |
| 发送消息 | `im:message:send_as_bot` | 群聊发消息 |
| 获取与更新群组信息 | `im:chat` | 群组管理 |

> 权限开通后需创建新版本并发布才能在正式环境生效。

### 第四步：配置事件订阅（长连接模式）

1. 在左侧导航点击「事件与回调」→「事件配置」
2. 点击「订阅方式」旁边的编辑按钮，选择**「使用长连接接收事件（WebSocket 模式）」**
   - 此模式**不需要**填写服务器 URL，OpenClaw 主动连接飞书
   - 若提示"未建立长连接"，说明 OpenClaw 侧还没配好，先完成第五步再回来保存
3. 点击「添加事件」，搜索并添加以下事件：
   - `im.message.receive_v1`（接收消息，机器人收到私聊/群聊消息时触发）
   - `im.chat.member.bot.added_v1`（机器人加入群聊时触发，可选）

### 第五步：在 OpenClaw 中配置飞书凭证

```bash
# 交互式添加渠道（推荐新手）
openclaw channels add
# 选择 Feishu，依次填入 App ID 和 App Secret

# 或者直接命令行配置
openclaw config set -- channels.feishu.appId "你的AppID"
openclaw config set -- channels.feishu.appSecret "你的AppSecret"
```

配置完成后启动/重启网关：

```bash
openclaw gateway
# 或重启
openclaw gateway restart
```

查看日志确认飞书连接成功：

```bash
openclaw logs --follow
```

看到 `feishu ws connected` 或 `feishu provider ready` 即为成功。

### 第六步：发布飞书应用版本

> **此步骤非常关键，跳过会导致功能不生效**

1. 在飞书开放平台左侧「版本管理与发布」
2. 点击「创建版本」，填写版本号和说明
3. 提交审核（企业自建应用通常自动审核通过）
4. 发布成功后，飞书客户端会收到"应用已发布"通知

检查点：

- 应用状态显示「已启用」
- 「当前修改均已发布」提示出现

### 第七步：验证配对（群聊场景）

将机器人添加到群聊：群设置 → 群机器人 → 添加机器人 → 搜索应用名。

私聊机器人发送任意消息，若机器人回复了配对码，在 OpenClaw WebUI 执行：

```bash
openclaw pairing approve feishu <配对码>
```

### 飞书接入检查清单

- [ ] 飞书开放平台应用已创建，机器人能力已添加
- [ ] App ID 和 App Secret 已复制到 OpenClaw
- [ ] `im:message` 等权限已开通
- [ ] 事件订阅已切换为**长连接模式**
- [ ] `im.message.receive_v1` 事件已添加
- [ ] OpenClaw 日志中看到 `feishu ws connected`
- [ ] 飞书应用已发布最新版本
- [ ] 私聊机器人能正常回复

---

## 高价值经验（来自高互动中文教程共性）

- 教程要"截图+步骤+检查点"，纯命令列表新手会卡住
- ClawHub 登录是关键，不登录会触发限速或安装失败
- 网络问题占多数，尤其是依赖下载和登录回调阶段
- 权限要提前讲清：不给权限，很多自动化功能必然失败
- 新手部署不要同时接多个渠道，先单通道稳定后再扩容
- **飞书长连接模式无需公网 IP**，是最适合个人/内网部署的选项

## 常见故障与处理

### 1) 安装脚本超时/中断

- 先确认网络和代理
- 重试安装前，保留错误输出
- 换一个更稳定节点再执行

### 2) 技能装不上或加载失败

- 先确认 `clawhub login` 成功
- 再看技能依赖是否满足
- 单个技能逐个安装，避免一次装太多

### 3) Linux 权限问题

- 避免 root 下混用普通用户目录
- 修正目录权限后重启 OpenClaw

### 4) 模型可用但无回复

- 检查模型 key 是否过期
- 检查渠道回调是否配置完整
- 用最小 prompt 做链路测试，先排除复杂逻辑

### 5) 飞书机器人收不到消息

- 检查事件订阅是否选择了**长连接模式**（不是 HTTP 回调模式）
- 检查 `im.message.receive_v1` 事件是否已添加并保存
- 检查 OpenClaw 网关是否在运行：`openclaw status`
- 查看日志：`openclaw logs --follow`，确认有 `feishu ws connected`
- 检查应用是否已在飞书平台**发布最新版本**
- 检查 App ID 和 App Secret 是否正确（无多余空格）

### 6) 飞书事件页面提示"长连接未建立"

- 正常现象：需要 OpenClaw 网关先跑起来并配好凭证，飞书才能检测到连接
- 操作顺序：先在 OpenClaw 配好 App ID/Secret 并启动网关，再回飞书开放平台保存事件配置

### 7) 飞书应用状态正常但机器人不响应

- 确认应用「已启用」且当前为最新发布版本
- 在「权限管理」确认 `im:message` 权限「已开通」（不是"待审核"）
- 尝试将机器人从群里移除再重新添加

## 远程协助话术模板（可直接复制）

### 开场

"我会按 5 步带你跑通：安装、登录、技能、模型、渠道。每步都先过检查点，不盲跳。"

### 卡住时

"先别重装，把这一步完整报错贴我。我先定位是网络、权限还是配置问题，再给你最小修复动作。"

### 收尾

"现在你已经能稳定收发消息了。下一步再做双渠道、自动化任务、定时提醒，不要一口气全开。"

## 验收标准（Done 定义）

- OpenClaw 已安装并可查询状态
- ClawHub 已登录且至少 1 个技能可用
- 至少 1 个模型可正常响应
- 至少 1 个聊天渠道已收发成功（飞书 / Telegram / Discord 均可）
- 用户知道常见报错如何自查

## 进阶扩展

- 加一个"部署采集表"：记录 OS、网络、报错、修复动作
- 给不同系统做 3 份 SOP：Windows/Linux/macOS
- 沉淀 FAQ：把高频报错做成固定排障树
- 飞书进阶：配置多租户、企业审批流、飞书卡片消息格式
