---
title: "ShareX 完全指南：截图、录屏与自动上传 Cloudflare R2"
date: 2026-06-23
description: "深度挖掘 ShareX 20.2 的所有核心功能，从截图模式、标注编辑、工作流自动化，到配置 Cloudflare R2 作为私有图床。"
tags: ["开源工具", "Windows", "截图", "Cloudflare"]
---

ShareX 是 Windows 上最强大的开源截图工具，没有之一。它免费、GPLv3 开源、持续活跃开发，功能密度远超任何付费竞品。但也正因为功能太多，很多人装上之后只用了截图，其他 90% 的能力从未触碰。

本文的目标是把 ShareX 20.2 的所有核心功能完整跑一遍。

> ShareX 仅支持 Windows，没有 macOS 或 Linux 版本。

---

## 安装

从 [getsharex.com](https://getsharex.com) 下载安装版或便携版。两者功能完全一致，便携版不写注册表，适合放在 U 盘或不想污染系统的场景。

---

## 截图模式

ShareX 提供 7 种截图模式，覆盖所有使用场景。

### Region（区域截图）

最常用的模式。按下快捷键后进入十字准星选区界面，支持像素级精确拖拽。进入选区界面后还可以切换为其他模式（窗口、全屏等），无需退出重来。

选区时按住空格键可以移动已经画好的选区，`W` 键切换放大镜，滚轮缩放放大镜倍率。

### Region (Freehand)（自由形状）

手绘任意形状截取。适合需要圈出不规则区域的场景，比如标记图片中某个具体位置。

### Fullscreen（全屏）

截取所有显示器的完整画面。多显示器用户会得到一张拼合图。

### Active Window / Active Monitor

仅截取当前焦点窗口或当前鼠标所在的显示器，不包含其他内容。

### Scrolling Capture（滚动截图）

长截图功能。ShareX 会自动滚动页面并拼合成一张完整图片。对现代复杂网页（尤其是含有 sticky header 或懒加载内容的页面）效果存在一定限制，简单页面和文档类内容表现良好。

### Auto Capture（自动截图）

按设定时间间隔自动截图。适合制作教程序列图、监控某个界面变化等场景。可以自定义截图区域和时间间隔。

---

## 录屏与 GIF

ShareX 使用 FFmpeg 处理视频录制，首次使用时会提示自动下载 FFmpeg。

### 屏幕录制

支持录制指定区域、活动窗口或全屏，输出为 MP4。可以同时录制系统音频和麦克风输入。进入 **Task Settings → Screen Recorder → Screen recording options** 可以调整编码器（默认 x264）、帧率、码率等参数。

### GIF 录制

适合录制短时操作演示。ShareX 直接输出 GIF，不需要额外转换步骤。帧率和色彩质量可以在录制设置里调整。GIF 文件体积通常比 MP4 大很多，超过 5 秒的内容建议用 MP4 代替。

---

## 图片编辑器与标注

ShareX 20 引入了基于 Avalonia UI 的新版编辑器，内置 18 种标注工具。

主要工具：

- **矩形 / 椭圆**：画框圈出重点区域
- **箭头**：支持经典样式和现代样式，可弯曲调整方向
- **文字**：添加说明文字，支持自定义字体和颜色
- **高亮**：半透明色块，不遮挡原始内容
- **马赛克 / 模糊**：对隐私信息打码，马赛克比高斯模糊更彻底
- **步骤编号**：自动递增的圆形数字标签，制作教程图非常好用
- **放大镜（Spotlight）**：20.2 新增，圈出并放大局部区域
- **贴纸**：内置 emoji 贴纸，支持旋转缩放

**背景美化（Background Beautifier）** 可以给截图添加渐变背景、阴影、圆角，适合制作展示用的截图，完全本地处理。

---

## After Capture Tasks：截图后做什么

这是 ShareX 最核心的自动化能力。截图完成后，ShareX 会按顺序执行你勾选的一系列任务。在主界面顶部菜单 **After capture tasks** 可以看到完整列表：

| 任务 | 说明 |
|---|---|
| Open in image editor | 打开标注编辑器 |
| Save image to file | 保存到本地 |
| Copy image to clipboard | 复制图片到剪贴板 |
| Upload image to host | 上传到图床 |
| Delete file locally | 上传后删除本地文件 |
| OCR image | 提取图片中的文字 |

典型工作流配置：勾选 **Open in image editor + Save image to file + Upload image to host + Copy URL to clipboard**，这样截图后标注完成就自动上传，链接已经在剪贴板里等你粘贴。

---

## After Upload Tasks：上传后做什么

上传完成后同样可以执行自动化任务：

- **Copy URL to clipboard**：上传完成后自动复制链接，最常用
- **Open URL**：在浏览器里打开上传结果
- **Shorten URL**：使用短链服务压缩 URL
- **Show QR code**：生成二维码方便手机扫描

---

## 快捷键配置

ShareX 的所有操作都可以绑定快捷键，进入 **Hotkey settings** 编辑。

| 操作 | 默认快捷键 |
|---|---|
| 区域截图 | `Print Screen` |
| 全屏截图 | `Ctrl + Print Screen` |
| 活动窗口截图 | `Alt + Print Screen` |
| 屏幕录制 | `Shift + Print Screen` |

每个快捷键可以单独配置截图模式、上传目标和 After Capture Tasks，互相独立。比如一个快捷键截图后保存本地，另一个截图后直接上传并复制链接。

> 如果 `Print Screen` 键被 Windows 11 截图工具占用，可以在 ShareX 里换成任意组合键，比如 `Ctrl + Shift + S`。

---

## 工具箱

主界面菜单 **Tools** 隐藏着一批独立工具，很多人不知道它们存在。

**OCR（文字识别）**：从屏幕任意区域提取文字。基于 Windows 内置 OCR 引擎，识别结果直接复制到剪贴板。

**颜色拾取器（Color Picker）**：取色器，支持 HEX、RGB、HSB 等多种格式输出，悬停在屏幕任意像素上即可读取颜色值。

**标尺（Ruler）**：屏幕像素标尺，测量元素尺寸和间距。

**哈希校验（Hash Checker）**：输入文件和期望哈希值，支持 MD5、SHA256 等算法验证文件完整性。

**图片合并（Image Combiner）**：将多张图片水平或垂直拼合为一张。

**二维码生成（QR Code）**：输入文字或 URL 生成二维码，也可以从屏幕扫描二维码内容。

**视频转换（Video Converter）**：基于 FFmpeg 的视频格式转换工具，不需要单独安装其他软件。

**屏幕固定（Pin to Screen）**：将截图以浮动窗口的形式钉在屏幕上，方便对照参考。比如对照设计稿写代码时可以把设计稿截图钉在角落，非常实用。

**DNS 更改器（DNS Changer）**：快速切换 DNS 服务器，内置 Google、Cloudflare、OpenDNS 等常用 DNS 预设。

**显示器测试（Monitor Test）**：显示纯色测试画面，用于检查显示器坏点和色彩均匀性。

---

## 配置 Cloudflare R2 作为图床

相比第三方图床，R2 的优势是：数据在你自己的 Cloudflare 账户里，免费额度充足（10GB 存储 + 每月 100 万次读取），无出站流量费，不会突然跑路。

### 第一步：创建 R2 存储桶

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com)
2. 左侧菜单进入 **Storage & Databases → R2**
3. 点击 **Create bucket**，填写存储桶名称（例如 `sharex-images`），选择地区后确认创建

### 第二步：绑定自定义域名（可选但推荐）

在存储桶设置里进入 **Settings → Custom Domains**，添加你在 Cloudflare 上托管的域名的子域（例如 `img.yourdomain.com`）。这样图片链接会是 `https://img.yourdomain.com/xxx.png` 而不是默认的 R2 公共 URL。

> 域名必须托管在 Cloudflare（NS 指向 Cloudflare），才能直接绑定为 R2 自定义域。

### 第三步：生成 R2 API Token

1. 在 R2 Overview 页面点击 **Manage API Tokens → Create API Token**
2. 权限选择 **Object Read & Write**，范围选择 **Apply to specific bucket** 并选中你刚创建的存储桶
3. 点击 **Create API Token**
4. 记录下 **Access Key ID** 和 **Secret Access Key**（Secret Key 只显示一次，务必保存）
5. 同时记录页面下方的 **Account ID**

### 第四步：在 ShareX 中配置

1. 打开 ShareX，进入 **Destinations → Destination Settings**
2. 在左侧列表找到 **Amazon S3** 并选中
3. 填写以下信息：

| 字段 | 填写内容 |
|---|---|
| Access Key ID | 第三步获取的 Access Key ID |
| Secret Access Key | 第三步获取的 Secret Access Key |
| Endpoint | `<Account_ID>.r2.cloudflarestorage.com` |
| Region | `auto` |
| Bucket name | 你的存储桶名称 |
| Upload path | 可选，例如 `screenshots/{yyyy}/{MM}/` 按年月归档 |
| Use custom domain | 勾选，填入你绑定的自定义域名 |

4. 进入右侧 **Advanced** 标签，**取消勾选** Set public-read ACL on file

> R2 不支持 ACL，勾选 Set public-read ACL on file 会导致上传失败，务必取消。

### 第五步：设置为默认上传目标

回到 ShareX 主界面，进入 **Destinations → Image Uploader**，选择 **File Uploader → Amazon S3**。对 File Uploader 做同样设置。

截一张图触发上传，等待完成后链接自动复制到剪贴板，用浏览器打开验证即可。

---

## 文件名规则

进入 **Task Settings → File naming → Name pattern** 自定义文件名格式。

常用变量：

| 变量 | 说明 |
|---|---|
| `%y` | 年（4 位） |
| `%mo` | 月（2 位） |
| `%d` | 日（2 位） |
| `%h` | 小时 |
| `%mi` | 分钟 |
| `%s` | 秒 |
| `%ra` | 随机字母数字 |

示例：`%y%mo%d_%h%mi%s_%ra` 生成类似 `20260623_143022_k7x` 的文件名，时间戳 + 随机后缀，确保唯一且有序。

---

## 设置备份与迁移

ShareX 的所有配置保存在 `%AppData%\ShareX` 文件夹中。迁移到新电脑时直接复制整个文件夹即可，包含所有快捷键、上传器配置和个人设置。

---

## 总结

ShareX 的功能体系可以用三层来理解：

1. **截图层**：7 种截图模式 + 完整标注编辑器，覆盖所有截图需求
2. **自动化层**：After Capture / After Upload Tasks 构成可编程工作流，一次截图触发一串操作
3. **工具层**：OCR、颜色拾取、哈希校验、屏幕固定等独立工具，很多场景下比单独安装专用软件更方便

配合 Cloudflare R2，截图到获得可用链接的整个过程可以压缩到 2 秒以内，且数据完全在自己手里。

- GitHub：[ShareX/ShareX](https://github.com/ShareX/ShareX)
- 官网：[getsharex.com](https://getsharex.com)
- 协议：GPLv3
