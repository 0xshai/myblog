---
title: "为博客添加评论区：Giscus"
date: 2026-05-23T10:00:00+08:00
description: "基于 GitHub Discussions 的免费评论系统，数据在自己手里，零成本接入 Hugo PaperMod。"
tags: ["Hugo", "建站", "Giscus", "评论"]
ShowToc: true
TocOpen: false
---

> 基于 GitHub Discussions 的免费评论系统，数据在自己手里，零成本接入 Hugo PaperMod。

---

## 为什么选 Giscus

给静态博客加评论的方案不少，常见的有 Disqus、Valine、Waline、Giscus 等。

Giscus 的优势：

- **免费**：完全免费，无广告
- **数据自托管**：评论存在你自己的 GitHub 仓库 Discussions 里，不依赖第三方服务器
- **无需单独注册**：访客用 GitHub 账号登录即可留言
- **隐私友好**：不追踪用户，不植入 Cookie
- **PaperMod 原生支持**：无需改模板，配置几行 toml 即可

唯一限制：访客需要有 GitHub 账号才能评论，适合技术类博客。

---

## 一、开启仓库 Discussions

打开你的博客仓库 `https://github.com/你的用户名/myblog`，进入：

**Settings** → 向下找到 **Features** → 勾选 **Discussions**

保存后仓库顶部导航栏会出现 **Discussions** 标签。

---

## 二、安装 Giscus App

访问 👉 [https://github.com/apps/giscus](https://github.com/apps/giscus)

点 **Install** → 选择 **Only select repositories** → 选你的博客仓库 → 确认安装。

---

## 三、获取配置参数

访问 👉 [https://giscus.app/zh-CN](https://giscus.app/zh-CN)

按如下填写：

| 选项 | 填写内容 |
| --- | --- |
| 仓库 | `你的用户名/myblog` |
| Discussion 分类 | `Announcements` |
| 页面 ↔ Discussion 映射 | `pathname` |
| 主题 | `preferred_color_scheme`（跟随系统深浅色） |
| 语言 | `zh-CN` |

填完后页面底部会自动生成一段 `<script>` 代码，其中有两个关键值：

```
data-repo-id="R_xxxxxxxxxx"
data-category-id="DIC_xxxxxxxxxx"
```

**把这两个值复制记下来**，下一步要用。

---

## 四、修改 hugo.toml

打开博客根目录的 `hugo.toml`，在 `[params]` 里加入以下内容：

```toml
comments = true

[params.giscus]
repo = "你的用户名/myblog"
repoId = "R_xxxxxxxxxx"          # 替换为你的 data-repo-id
category = "Announcements"
categoryId = "DIC_xxxxxxxxxx"    # 替换为你的 data-category-id
mapping = "pathname"
strict = "0"
reactionsEnabled = "1"
emitMetadata = "0"
inputPosition = "bottom"
theme = "preferred_color_scheme"
lang = "zh-CN"
```

完整的 `[params]` 区域示例：

```toml
[params]
homeInfoParams = {Title = "0xSHAI 👋", Content = "探索 Web 与系统的边界"}
defaultTheme = "auto"
ShowReadingTime = true
ShowPostNavLinks = true
ShowBreadCrumbs = true
ShowCodeCopyButtons = true
ShowToc = true
TocOpen = false
comments = true                  # 新增这一行

[params.giscus]                  # 新增这整个块
repo = "你的用户名/myblog"
repoId = "R_xxxxxxxxxx"
category = "Announcements"
categoryId = "DIC_xxxxxxxxxx"
mapping = "pathname"
strict = "0"
reactionsEnabled = "1"
emitMetadata = "0"
inputPosition = "bottom"
theme = "preferred_color_scheme"
lang = "zh-CN"
```

---

## 五、推送部署

```bash
cd D:\shai\blog
git add .
git commit -m "add giscus comments"
git push
```

Cloudflare Pages 检测到推送后会自动重新构建，几分钟内生效。

---

## 验证

部署完成后打开任意一篇文章，滚动到底部，应该能看到评论区加载出来。

首次有人评论时，GitHub 仓库的 Discussions 里会自动创建对应的条目，后续所有评论都会存在那里，在仓库 **Discussions** 标签下可以直接管理。

---

## 注意事项

- 本地 `hugo server` 预览时评论区会正常显示，但登录功能需要真实域名才能完成 OAuth 授权
- `repoId` 和 `categoryId` 必须从 [giscus.app](https://giscus.app/zh-CN) 生成，不能手动填写
- 如果评论区不出现，检查 `comments = true` 是否写在 `[params]` 内部，而不是其他位置
