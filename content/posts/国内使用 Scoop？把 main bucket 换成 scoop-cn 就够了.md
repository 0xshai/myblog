---
title: "国内使用 Scoop？把 main bucket 换成 scoop-cn 就够了"
date: 2026-06-05
tags: ["Windows", "Scoop", "开发工具", "效率"]
description: "scoop-cn 是专为国内用户维护的 Scoop 镜像库，合并了官方全部十个 bucket，并把下载地址替换为国内可访问的镜像。本文记录完整切换过程。"
---

在国内用 Scoop，最头疼的不是用法，是网络。官方 bucket 托管在 GitHub，`git clone` 慢、下载超时、更新失败，三件套缺一不可。

## scoop-cn 是什么

[scoop-cn](https://github.com/duzyn/scoop-cn) 是 GitHub 用户 [duzyn](https://github.com/duzyn) 维护的一个专为国内用户设计的 Scoop 镜像库，本文的思路也完全来自这个项目。它做了两件事：

- **下载地址替换**：把所有应用的下载链接替换为 [gh-proxy.com](https://gh-proxy.com/) 镜像，绕开直连 GitHub 的问题
- **多库合并**：把官方全部十个 bucket（`main`、`extras`、`versions`、`nirsoft`、`sysinternals`、`php`、`nerd-fonts`、`nonportable`、`java`、`games`）合并进一个库，用一个 bucket 覆盖全部官方软件

库本身每小时自动同步官方内容，基本保持同步。换完之后不需要再单独 `scoop bucket add extras` 之类，`scoop search` 直接全覆盖。

本文记录把已有 Scoop 切换到 scoop-cn 的完整步骤。

---

## 切换步骤

### 第一步：删除现有 main bucket

```powershell
scoop bucket rm main
```

删除 bucket 只是移除软件包的元数据索引，不会卸载任何已安装的程序。

如果上面命令报错，手动删除文件夹：

```powershell
Remove-Item "$env:USERPROFILE\scoop\buckets\main" -Recurse -Force -ErrorAction SilentlyContinue
```

### 第二步：添加 scoop-cn 作为 main

```powershell
scoop bucket add main https://gh-proxy.com/https://github.com/duzyn/scoop-cn.git
```

### 第三步：更新并验证

```powershell
scoop update
scoop bucket list
scoop status
```

`main` 指向 `https://github.com/duzyn/scoop-cn`，且输出 `Everything is ok!`，切换完成。

---

## 配置 aria2 多线程下载

换完 bucket 之后顺手把 aria2 也配上，下载速度会明显提升：

```powershell
scoop install aria2

scoop config aria2-enabled true
scoop config aria2-max-connection-per-server 16
scoop config aria2-split 16
scoop config aria2-min-split-size 1M
```

---

## gh-proxy 失效时的处理

gh-proxy.com 偶尔会不稳定。如果 `scoop update` 开始失败，用以下命令更换代理地址：

```powershell
# 更新 Scoop 自身的拉取地址
scoop config scoop_repo https://gh-proxy.com/https://github.com/ScoopInstaller/Scoop.git

# 更新 main bucket 的远程地址
git -C "$env:USERPROFILE\scoop\buckets\main" remote set-url origin https://gh-proxy.com/https://github.com/duzyn/scoop-cn.git
```

把 `https://gh-proxy.com` 换成当时可用的其他镜像即可。

---

## 附：安装目录建议放 D 盘

如果是全新安装 Scoop，建议提前把目录指向 D 盘，避免 C 盘长期膨胀：

```powershell
$env:SCOOP = 'D:\scoop'
[Environment]::SetEnvironmentVariable('SCOOP', $env:SCOOP, 'User')
```

设置好之后再运行 Scoop 安装脚本。
