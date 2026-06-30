---
title: "n8n 自托管：搭建你的工作流自动化中心"
date: 2026-06-30
draft: false
description: "用 Docker 部署开源工作流自动化平台 n8n，并实战一个 RSS 推送到 Telegram 的自动化流程"
tags: ["自托管", "自动化", "Docker"]
---

如果你用过 IFTTT、Zapier 这类自动化工具，但又对把数据交给第三方服务感到不安，n8n 是最值得考虑的开源替代。它支持可视化拖拽搭建工作流，400+ 服务集成，而且可以完全自托管，数据不出自己的服务器。

> **提示**：n8n 采用 fair-code 协议，核心功能开源免费，自托管不收任何费用。官方也提供付费的 n8n Cloud 托管版，本文不涉及，只讲自部署。

## 准备工作

你需要：

- 一台能跑 Docker 的机器（VPS、NAS、或者本地 WSL 都行）
- Docker 和 Docker Compose

如果你还没有 Docker 环境，可以直接在 Linux/WSL 下安装 Docker Engine，或者用 Rancher Desktop 这类带 GUI 的方案。

## 部署 n8n

最简单的方式是用 Docker Compose，新建一个目录，写入 `docker-compose.yml`：

```yaml
version: '3'

services:
  n8n:
    image: n8nio/n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - GENERIC_TIMEZONE=Asia/Shanghai
      - TZ=Asia/Shanghai
      - N8N_SECURE_COOKIE=false
    volumes:
      - ./n8n_data:/home/node/.n8n
```

> **注意**：`N8N_SECURE_COOKIE=false` 是因为没有配置 HTTPS 时，n8n 默认的安全 Cookie 策略会导致登录失败。如果你后续用 Nginx/Caddy 反代加了 HTTPS，可以去掉这行。

启动：

```bash
docker compose up -d
```

打开浏览器访问 `http://你的服务器IP:5678`，首次访问会要求创建管理员账号，邮箱密码自己设，不需要联网验证。

数据全部存在 `./n8n_data` 目录里，包括你的工作流、凭证、执行记录，备份这个目录就等于备份了整个 n8n。

## 跑通第一个工作流：RSS 推送到 Telegram

熟悉界面最快的方式是直接做一个有用的东西。这里搭建一个"监控 RSS 源，有新文章就推送到 Telegram"的自动化。

### 第一步：创建 Telegram Bot

打开 Telegram，搜索 `@BotFather`，发送 `/newbot`，按提示设置 Bot 名字，完成后会拿到一个 Token，形如：

```
123456789:ABCdefGHIjklMNOpqrSTUvwxYZ
```

记下来，之后要填进 n8n。

然后找到你刚创建的 Bot，随便发一条消息（比如"hi"），这一步是为了让 Bot 能识别到你的 Chat ID。

接着访问这个链接（把 Token 换成你自己的）：

```
https://api.telegram.org/bot<你的Token>/getUpdates
```

返回的 JSON 里找 `chat.id` 字段，记下来。

### 第二步：在 n8n 里搭建工作流

1. 登录 n8n，点击 **New Workflow**

2. 添加第一个节点，搜索 **RSS Feed Trigger**，配置你想监控的 RSS 地址（比如你自己博客的 `/index.xml`，或者任何你关注的站点）

3. 设置触发间隔，比如每 30 分钟检查一次

4. 添加第二个节点，搜索 **Telegram**，选择 **Send Message** 操作

5. 在 Telegram 节点里创建新凭证（Credential），填入第一步的 Bot Token

6. **Chat ID** 填第一步拿到的 ID

7. **Text** 字段用表达式引用 RSS 节点的输出，比如：

```
新文章：{{ $json.title }}
{{ $json.link }}
```

8. 连接两个节点，点击右上角 **Active** 开关启用工作流

完成后，RSS 源一旦有新内容，n8n 会自动检测并推送到你的 Telegram。

> n8n 的表达式系统用 `{{ }}` 引用上游节点数据，类似 Excel 公式，鼠标悬停在字段上会有自动补全提示，不需要死记语法。

## 进阶：文件变动通知

如果你在用 Syncthing 之类的自托管同步工具，也可以用 n8n 搭一个"指定目录有新文件就通知"的流程：

1. 用 **Local File Trigger** 节点（需要装社区节点）监控目录
2. 连接 **Telegram** 或 **Webhook** 节点做通知

n8n 的优势就在这里：几乎任何"A 发生时做 B"的场景，都能用节点拼出来，不需要写代码。

## 几个实用提示

- **凭证管理**：n8n 会加密保存所有 API Key、Token，存在数据目录里，不会明文暴露在工作流 JSON 里（除非你导出时选择包含凭证）
- **执行记录**：左侧菜单的 Executions 可以看到每次运行的详细日志，调试报错很方便
- **社区节点**：官方集成不够用时，可以在设置里启用社区节点（Community Nodes），生态相当丰富
- **备份**：直接打包 `n8n_data` 目录即可，迁移到新服务器解压启动就能用

## 写在最后

n8n 比 IFTTT/Zapier 多花的这点部署成本，换来的是数据主权和无限的工作流数量限制（官方云服务通常按执行次数收费）。如果你已经有自托管的习惯，把 n8n 加入你的服务清单是很自然的一步。
