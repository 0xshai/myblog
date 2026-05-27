---
title: "用 Hermes Agent 免费跑 AI：OpenRouter / Gemini / Groq 配置指南"
date: 2026-05-27
tags: ["AI", "工具", "开发效率"]
description: 从零安装 Hermes Agent，配置三种免费 API 方案，本地跑 AI Agent 不花一分钱。
---

Hermes Agent 是目前我用过最顺手的本地 Agent 之一，支持强大的工具调用和多模型切换。

最近把它装到了 Windows 上，踩了不少坑，整理成这篇教程，方便大家少走弯路。

## 一、安装

新开一个 PowerShell 窗口，指定安装目录，然后执行安装脚本：

```powershell
$env:HERMES_HOME = "D:\AI\Hermes"
iex (irm https://ghproxy.com/https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.ps1)
```

> 推荐装到 D 盘，空间大、好管理。安装脚本走 ghproxy 镜像，国内直连不卡。

安装过程会依次下载 Python 3.11、Node.js 22（便携版）、ripgrep、ffmpeg 以及 Hermes 本体，总下载量 500MB～800MB，网速正常的话 10～20 分钟完成。

安装完成后，重开一个 PowerShell 窗口，验证是否成功：

```powershell
hermes --version
```

---

## 二、启动

推荐用 TUI 模式，界面好看很多：

```powershell
hermes --tui
```

进入后会看到配置向导，按提示走一遍。

---

## 三、配置免费模型

免费模型有三个来源可以选，按优先级排：

### 方案一：OpenRouter

1. 输入 `/model` 回车
2. 选择 **OpenRouter**
3. 填入 API Key（去 https://openrouter.ai/keys 获取）
4. 模型名称从下面选一个

| 模型 | 适合场景 |
| --- | --- |
| `openrouter/free` | 自动选最优免费模型，日常首选 |
| `qwen/qwen3-coder:free` | 写代码 |
| `deepseek/deepseek-r1:free` | 推理、逻辑题 |

> 注意：OpenRouter 的免费模型变动比较频繁，有些标了 `:free` 实际用起来会报错或报 403（大陆地区限制）。如果选了跑不起来，直接换方案二。

### 方案二：Gemini API（推荐备选）

Google 官方免费额度，不需要绑卡，模型质量高，Gemini 2.0 Flash 速度快、能力强。

去这里申请 Key：https://aistudio.google.com/apikey

登录 Google 账号，点 **Create API key**，复制保存。

然后在 Hermes 里：

1. 输入 `/model` 回车
2. 选择 **Google** 或 **Gemini**
3. 填入 API Key
4. 模型填 `gemini-2.0-flash`

### 方案三：Groq

同样免费，跑的是 Llama、Mixtral 等开源模型，响应速度极快，注册即用。

去这里申请 Key：https://console.groq.com/keys

配置方式和上面一样，Provider 选 **Groq**，模型选 `llama-3.3-70b-versatile`。

---

## 四、常用命令

| 命令 | 作用 |
| --- | --- |
| `hermes --tui` | 启动 TUI 界面 |
| `/model` | 切换模型 |
| `hermes model` | 模型配置向导 |
| `hermes tools` | 管理工具 |
| `hermes setup` | 重新运行完整设置 |

---

## 五、几个建议

**工具记得开。** 进 `hermes tools`，把 `browser`、`code_execution`、`file` 打开，Agent 能力会强很多，不开等于浪费了一半功能。

**跑不起来先换方案。** OpenRouter 免费模型变动频繁，报错别死磕，直接换 Gemini 或 Groq，稳定多了。

**多试几个模型。** 写代码用 qwen3-coder 或 Groq 的 Llama，推理用 deepseek-r1，找到最顺手的那个。

---

目前 Hermes Agent 还属于早期版本，但工具调用和 Agent 能力已经很能打了，值得长期跟进。

你装好了吗？欢迎在评论区聊聊你用的哪个模型效果最好，或者遇到了什么问题，我会持续补充这篇文章。
