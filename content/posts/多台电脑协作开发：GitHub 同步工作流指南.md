---
title: "多台电脑协作开发：GitHub 同步工作流指南"
date: 2026-05-25
tags: ["GitHub", "Cloudflare", "工作流"]
description: "如何在多台电脑间同步代码，以及控制 Cloudflare Pages 自动部署的实用技巧。"
---

在用 GitHub + Cloudflare Pages 管理博客时，经常需要在不同电脑上切换开发。
本文整理了多设备协作的完整工作流，以及如何控制 Cloudflare 自动部署。

## 一、换了新电脑，如何拿到代码

每次在新电脑上第一次工作，需要把仓库克隆到本地：

```bash
git clone https://github.com/你的用户名/myblog.git
cd myblog
npm install
```

三行命令搞定——代码拉下来，依赖自动安装，直接可以开发。

`npm install` 会根据 `package.json` 重新下载所有依赖包，和原来完全一致，
所以 `node_modules/` 不需要上传到 GitHub，换电脑也不影响。

## 二、每次开始和结束工作的习惯

**开始工作前，先同步最新代码：**

```bash
git pull
```

把另一台电脑推上去的改动拉下来，保持两台电脑代码一致。

**工作结束后，推送到 GitHub：**

```bash
git add .
git commit -m "描述这次改了什么"
git push
```

核心习惯：**每次开始先 pull，每次结束后 push。**
养成这个习惯，多台电脑之间永远不会出现代码冲突。

## 三、如何控制 Cloudflare Pages 自动部署

默认情况下每次 push 都会触发 Cloudflare 构建，免费版每月 500 次。
频繁调试时容易浪费额度，有两种方式控制。

### 方法一：commit 信息加 [skip ci]

```bash
git commit -m "调整样式 [skip ci]"
```

Cloudflare 识别到这个关键词会自动跳过本次构建，灵活控制每次 push 是否部署。

### 方法二：关闭自动部署

进入 Cloudflare Pages 后台：

```
Settings → Builds & deployments → Branch control
```

把 Production branch 的自动部署关掉，之后所有 push 都不触发构建，
需要发布时手动去 Deployments → Create deployment 触发。

**推荐方式：** 两种方法结合——平时调试用 [skip ci] 跳过，
确认要上线的 commit 正常写，让自动部署触发。

## 四、没有部署时如何本地预览

### 方法一：hugo server（最常用）

```bash
hugo server
```

打开 `http://localhost:1313` 实时预览，改动文件后页面自动刷新，
不需要构建也不需要部署，是最快的开发方式。

### 方法二：Cloudflare 预览链接

每次 push 到 GitHub，Cloudflare Pages 会生成一个独立的预览链接：

```
https://[commit-hash].shaifx.pages.dev
```

不影响正式域名，专门用来预览测试。

## 五、完整工作流总结

**换新电脑？先运行：**

```bash
git clone https://github.com/你的用户名/myblog.git
cd myblog
npm install
```

**每天开始工作：**

```bash
git pull       # 同步最新代码
hugo server    # 本地预览
```

**改完代码，还在调试：**

```bash
git add . && git commit -m "xxx [skip ci]" && git push
```

**改完代码，准备上线：**

```bash
git add . && git commit -m "xxx" && git push
```

本地预览速度最快，改完立刻看到结果。
确认没问题再正式 push，既节省构建次数，也避免把半成品推到线上。