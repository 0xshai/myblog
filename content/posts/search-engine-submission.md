---
title: "把网站提交给搜索引擎"
date: 2026-06-24
description: "Google Search Console、Bing Webmaster Tools、百度搜索资源平台的完整提交流程，以及 sitemap 和 IndexNow 的基本原理。"
tags: ["SEO", "Google", "Bing", "百度", "建站"]
---

建好网站之后，搜索引擎不会自动发现你——需要主动告诉它你在哪里。这篇记录三个主要搜索引擎的提交流程，以及背后的基本原理。

---

## 原理：爬虫、sitemap、收录

搜索引擎靠**爬虫**（crawler）抓取网页内容，再建立索引，用户搜索时才能找到你。

爬虫发现页面的方式有两种：

- 从其他网站的外链顺藤摸瓜爬过来
- 你主动提交 **sitemap**，告诉它有哪些页面

**Sitemap** 是一个 XML 文件，列出网站所有页面的 URL，让爬虫不用猜。Astro 自动生成 `sitemap-index.xml`，里面指向 `sitemap-0.xml`，后者才是真正的页面列表，这是标准的两级结构。

你的 sitemap 地址通常是：

```
https://你的域名/sitemap-index.xml
```

提交 sitemap 之后不代表立刻收录，Google 一般几天到两周，Bing 稍快，百度最慢也最难预测。

---

## Google Search Console

地址：[search.google.com/search-console](https://search.google.com/search-console)

### 验证所有权

进入后选**网址前缀**，输入你的完整网址（含 `https://`），点继续。

推荐用 **HTML 文件**验证：

1. 下载验证文件（形如 `google6cf6f7204d0d5185.html`）
2. 放进项目的 `public/` 目录（或网站根目录）
3. 部署后访问 `https://你的域名/google6cf6f7204d0d5185.html` 确认能打开
4. 回到 Search Console 点验证

> 验证完成后不要删除这个文件，Google 会定期回来检查，删掉会导致验证失效。

### 提交 sitemap

验证通过后，左侧菜单找到**站点地图**，在输入框填入：

```
sitemap-index.xml
```

点提交。状态初始显示"无法抓取"是正常的，Google 还没来得及读取，等几小时后刷新会变成"成功"。

### 日常使用

- **覆盖率** — 查看哪些页面已收录，哪些被排除或有错误
- **搜索结果** — 用户用什么关键词找到你，点击率如何
- **网址检查** — 新文章发布后可以手动输入 URL 请求立即抓取，比等爬虫自己来快

---

## Bing Webmaster Tools

地址：[bing.com/webmasters](https://www.bing.com/webmasters)

### 从 Google 导入（推荐）

已经配置好 Google Search Console 的话，Bing 支持一键导入，省去重新验证的步骤：

1. 登录 Bing Webmaster Tools（支持微软账号或 Google 账号）
2. 点击**从 Google Search Console 导入**
3. 授权 Google 账号，选择要导入的站点
4. 站点和 sitemap 自动同步过来，无需额外操作

### 单独添加

如果不想关联 Google 账号，也可以手动添加：

1. 点击**添加网站**，输入你的网址
2. 选 HTML 文件或 meta 标签验证（同 Google 流程）
3. 验证通过后在**站点地图**页面提交 `sitemap-index.xml`

> Bing 的数据会同步给 DuckDuckGo 和 Yahoo，提交一次相当于覆盖了好几个搜索引擎。

---

## 百度搜索资源平台

地址：[ziyuan.baidu.com](https://ziyuan.baidu.com)

> **注意：** 百度爬虫无法访问 `pages.dev`、`github.io` 等境外托管域名。如果你的网站部署在这些平台上，提交了也没有实际效果。绑定自定义域名并确保国内可访问后再操作。

### 流程

1. 登录后点击**用户中心 → 站点管理 → 添加网站**
2. 输入域名，选择站点类型
3. 验证方式选 HTML 文件，下载后放进网站根目录部署
4. 验证通过后进入**数据引入 → sitemap**，提交你的 sitemap 地址

百度还提供**主动推送**功能，可以通过 API 实时推送新页面 URL，收录速度比等爬虫快，但需要额外配置。

---

## IndexNow（进阶）

IndexNow 是微软发起的协议，支持的引擎包括 Bing、Yandex 等。原理是：你发布新内容后主动调用一个 API 通知搜索引擎，引擎很快来抓取，不用等爬虫按自己的节奏来。

只需要生成一个 API Key，提交到任意一个支持 IndexNow 的引擎，它会自动把通知转发给其他引擎。

Astro 目前没有官方插件，可以通过 Cloudflare Workers 或 GitHub Actions 在每次部署后自动调用 IndexNow API 实现自动化推送。Google 目前不支持 IndexNow，仍需通过 Search Console 单独处理。

---

## 小结

| 搜索引擎 | 适合场景 | 优先级 |
|---------|---------|--------|
| Google Search Console | 所有网站 | ★★★ |
| Bing Webmaster Tools | 所有网站，可从 Google 一键导入 | ★★★ |
| 百度搜索资源平台 | 绑定自定义域名、国内可访问的网站 | ★★（视情况） |
| IndexNow | 更新频繁、希望快速收录的网站 | ★（进阶可选） |
