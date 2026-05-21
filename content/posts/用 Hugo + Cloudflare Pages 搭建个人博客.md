+++
date = '2026-05-21T10:39:29+08:00'
draft = false
title = '用 Hugo + Cloudflare Pages 搭建个人博客'
tags = ['Hugo', '建站', 'Cloudflare']
+++

> 零成本、全静态、自动部署。用 Hugo 搭建个人博客并托管到 Cloudflare Pages 的完整流程。

## 准备工作

安装以下工具，并注册好账号：

- [Git](https://git-scm.com/downloads)
- [Hugo](https://gohugo.io/installation/)（Windows 用 `winget install Hugo.Hugo.Extended`）
- [GitHub](https://github.com) 账号
- [Cloudflare](https://cloudflare.com) 账号

---

## 一、创建站点

```bash
hugo new site myblog
cd myblog
git init
```

## 二、安装 PaperMod 主题

```bash
git submodule add https://github.com/adityatelange/hugo-PaperMod themes/PaperMod
```

## 三、配置 hugo.toml

用以下内容替换根目录的 `hugo.toml`：

```toml
baseURL = "/"
locale = "zh-CN"
title = "我的博客"
theme = "PaperMod"

[params]
  homeInfoParams = {Title = "你好 👋", Content = "欢迎来到我的博客"}
  defaultTheme = "auto"
  ShowReadingTime = true
  ShowPostNavLinks = true
  ShowBreadCrumbs = true
  ShowCodeCopyButtons = true
  ShowToc = true

[menu]
  [[menu.main]]
    name = "文章"
    url = "/posts/"
    weight = 1
  [[menu.main]]
    name = "标签"
    url = "/tags/"
    weight = 2
  [[menu.main]]
    name = "归档"
    url = "/archives/"
    weight = 3

[outputs]
  home = ["HTML", "RSS", "JSON"]
```

## 四、创建归档页

```bash
hugo new content archives.md
```

打开文件，内容替换为：

```markdown
---
title: "归档"
layout: "archives"
url: "/archives/"
summary: "archives"
---
```

## 五、写文章

```bash
hugo new content posts/文章标题.md
```

打开生成的文件，将 `draft: true` 改为 `draft: false`，然后在下方写正文。

本地预览：

```bash
hugo server
```

浏览器打开 `http://localhost:1313` 查看效果。

---

## 六、推送到 GitHub

在 GitHub 新建仓库，然后在本地运行：

```bash
git remote add origin https://github.com/你的用户名/仓库名.git
git add .
git commit -m "初始化博客"
git push -u origin master
```

> **网络问题？** 如果连不上 GitHub，先配置本地代理（端口替换为你实际使用的）：
> ```bash
> git config --global http.proxy http://127.0.0.1:7890
> git config --global https.proxy http://127.0.0.1:7890
> ```

---

## 七、部署到 Cloudflare Pages

1. 登录 Cloudflare → **Workers & Pages** → **Create** → **Pages**
2. 连接 GitHub，选择你的仓库
3. 填写构建配置：

   | 字段 | 值 |
   |---|---|
   | Framework preset | `Hugo` |
   | Build command | `hugo` |
   | Build output directory | `public` |
   | 环境变量 `HUGO_VERSION` | `0.147.0` |

4. 点击 **Save and Deploy**

部署完成后会获得一个 `*.pages.dev` 免费域名。此后每次 `git push`，网站自动更新。

---

## 后续维护

每次写完新文章，只需三条命令：

```bash
git add .
git commit -m "新文章：文章标题"
git push
```
