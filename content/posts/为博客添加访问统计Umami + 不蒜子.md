+++
date = '2026-05-21T18:00:00+08:00'
draft = false
title = '为博客添加访问统计：Umami + 不蒜子'
tags = ['Hugo', '建站', 'Umami', '不蒜子', '统计']
+++

> 一套给自己看，一套给访客看。用 Umami + 不蒜子为博客添加完整的访问统计。

---

## 工具介绍

### Umami

[Umami](https://umami.is) 是一款开源的网站统计工具，定位是 Google Analytics 的隐私友好替代品。

- **开源**：代码完全公开，可自部署
- **隐私友好**：不使用 Cookie，不追踪个人信息，符合 GDPR
- **数据详细**：访客来源、设备类型、浏览器、停留时长、实时在线人数
- **免费版**：Umami Cloud 每月 10 万次页面浏览，个人博客完全够用
- **数据归属**：显示在后台，只有博主自己能看

适合用来了解博客的真实流量情况。

### 不蒜子

[不蒜子](https://busuanzi.ibruce.info) 是一个极简的页面计数器，由国内开发者维护。

- **接入极简**：两行代码搞定
- **免费无限制**：没有用量上限
- **双维度统计**：网站总访问量 + 单篇文章浏览量
- **数据公开**：数字直接显示在博客页面上，访客可见
- **缺点**：不开源，数据存在作者服务器，部分广告拦截插件（如 Brave）会屏蔽

适合在博客页面直接展示浏览数字。

---

## 两者的区别

| | Umami | 不蒜子 |
|---|---|---|
| 数据可见性 | 仅博主后台 | 页面公开显示 |
| 统计维度 | 详细（来源、设备等） | 仅浏览次数 |
| 隐私 | 极佳 | 一般 |
| 接入难度 | 中 | 极简 |
| 开源 | ✅ | ❌ |

两者互补，同时使用效果最好。

---

## 一、接入 Umami Cloud

### 1. 注册账号

打开 [umami.is](https://umami.is)，点 **Sign up** 注册，选择数据存储地区（推荐 United States）。

### 2. 添加网站

登录后进入 **Settings** → **Websites** → **Add website**：

- Name：填博客名称
- Domain：填你的域名（如 `shaifx.pages.dev`，不加 `https://` 和末尾 `/`）

### 3. 获取追踪代码

添加完成后点 **Edit** → **Tracking code**，复制类似这样的代码：

```html
<script defer src="https://cloud.umami.is/script.js" data-website-id="你的ID"></script>
```

### 4. 注入博客

在 `layouts/partials/` 目录下新建 `extend_head.html`（PaperMod 专用扩展钩子，不会覆盖主题文件）：

```html
<script defer src="https://cloud.umami.is/script.js" data-website-id="你的ID"></script>
```

部署后登录 Umami 后台即可查看实时数据。

---

## 二、接入不蒜子

### 1. 加载脚本

在 `layouts/partials/extend_head.html` 里追加一行：

```html
<script async src="//busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js"></script>
```

### 2. 显示网站总访问量

在 `layouts/partials/extend_footer.html` 里加入：

```html
<div style="text-align:center; padding: 10px; font-size: 0.85em; opacity: 0.6;">
  本站累计访问 <span id="busuanzi_value_site_pv"></span> 次 · 访客 <span id="busuanzi_value_site_uv"></span> 人
</div>
```

### 3. 显示单篇文章浏览量

先把主题模板复制到自己的目录（以 PaperMod 新版为例）：

```bash
cp themes\PaperMod\layouts\single.html layouts\_default\single.html
```

然后找到显示阅读时间的位置，加入：

```html
· 浏览 <span id="busuanzi_value_page_pv"></span> 次
```

---

## 三、推送部署

```bash
git add .
git commit -m "添加 Umami + 不蒜子统计"
git push
```

---

## 注意事项

- 本地 `hugo server` 预览时不蒜子数字显示异常，属正常现象，部署到线上才准确
- Brave 等浏览器内置拦截器会屏蔽不蒜子脚本，导致数字不显示，不影响其他普通访客
- Umami 的统计数据在后台查看，不蒜子的数字直接显示在页面上，两套数据不互通
