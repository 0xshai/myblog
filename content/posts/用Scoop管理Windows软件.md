---
title: "用 Scoop 管理 Windows 软件，从此告别手动安装"
date: 2026-06-01
tags: ["Windows", "工具", "开发效率"]
description: "Scoop 是 Windows 上的命令行包管理器，一条命令安装软件、指定路径、一键更新，换电脑也能秒速还原所有工具。"
---

# 用 Scoop 管理 Windows 软件，从此告别手动安装

每次装新电脑，你是不是也要经历这套流程：打开浏览器，搜索软件名，找到官网，下载安装包，下一步下一步完成，然后发现装到 C 盘了……

Scoop 可以终结这一切。

---

## Scoop 是什么

Scoop 是 Windows 上的命令行包管理器，类似 macOS 上的 Homebrew。你只需要一条命令，它就能帮你下载、安装、更新、卸载软件，而且全部装在你指定的目录里，不动注册表，不需要管理员权限。

装软件就是这样：

```powershell
scoop install git
```

更新所有软件就是这样：

```powershell
scoop update *
```

---

## 安装 Scoop

打开 PowerShell（不需要管理员），执行：

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
irm get.scoop.sh | iex
```

默认会装在 `C:\Users\你的用户名\scoop`。如果想装到其他盘，在安装前先设置：

```powershell
$env:SCOOP='D:\scoop'
[Environment]::SetEnvironmentVariable('SCOOP', $env:SCOOP, 'User')
irm get.scoop.sh | iex
```

设置一次，之后所有 Scoop 管理的软件都在 D 盘，一劳永逸。

---

## Bucket 是什么

Scoop 把软件分装在不同的"桶"（bucket）里。默认只有 `main`，收录命令行工具。日常最常用的是 `extras`，收录了大量 GUI 应用：

```powershell
scoop bucket add extras
```

其他常用 bucket：

| Bucket | 内容 |
|--------|------|
| `main` | 命令行工具，默认已添加 |
| `extras` | GUI 应用，最常用 |
| `versions` | 软件的历史版本 |
| `nerd-fonts` | 编程字体，比如 JetBrains Mono |

添加完 extras 之后，你能装的软件范围就覆盖了绝大多数日常需求。

---

## 常用命令速查

```powershell
# 搜索软件
scoop search mailspring

# 安装软件
scoop install extras/mailspring

# 更新单个软件
scoop update mailspring

# 更新所有软件
scoop update *

# 卸载软件
scoop uninstall mailspring

# 查看已安装的软件
scoop list

# 查看软件安装信息
scoop info git
```

---

## 哪些软件可以用 Scoop 装

开发类工具几乎全覆盖：

```powershell
scoop install git python nodejs ffmpeg yt-dlp hugo
```

常用 GUI 软件也有：

```powershell
scoop install extras/zen-browser
scoop install extras/mailspring
scoop install extras/vscode
scoop install extras/obsidian
scoop install extras/vlc
```

字体也可以：

```powershell
scoop bucket add nerd-fonts
scoop install nerd-fonts/JetBrainsMono-NF
```

---

## 最大的隐藏好处：换电脑

Scoop 可以导出你所有已安装软件的列表：

```powershell
scoop export > scoopfile.json
```

新电脑装好 Scoop 之后，一条命令全部还原：

```powershell
scoop import scoopfile.json
```

把 `scoopfile.json` 放进云盘或 Git 仓库，重装系统再也不用痛苦地回忆"我之前装了什么"。

---

## 和手动安装的对比

| | 手动安装 | Scoop |
|--|---------|-------|
| 安装路径 | 由安装包决定，常常是 C 盘 | 完全由你控制 |
| 更新 | 打开软件等提示，或去官网重下 | `scoop update *` |
| 卸载 | 控制面板，残留文件难清理 | `scoop uninstall`，干净彻底 |
| 需要管理员权限 | 大多数需要 | 不需要 |
| 换电脑迁移 | 手动一个个重装 | 导出列表，一键还原 |
| 注册表 | 会写入 | 基本不动 |

---

## 小技巧

**不知道软件叫什么名字？** 用 `scoop search` 模糊搜索，或者去 [scoop.sh](https://scoop.sh) 直接在网页上搜。

**已经手动装了一堆软件怎么办？** 不用强行迁移。等某个软件下次需要更新时，卸掉旧的，用 Scoop 重装。命令行工具可以现在就迁，GUI 软件等自然更新周期到了再换。

**软件装在哪里？** 每个软件都在 `D:\scoop\apps\软件名\` 下，版本目录清晰，一目了然。

---

## 总结

Scoop 没有学习成本，装完就能用，带来的收益却是长期的：软件统一管理、路径可控、更新方便、换机无痛。

如果你用 Windows 做开发或者重度使用开源工具，Scoop 是目前最值得养成的使用习惯之一。
