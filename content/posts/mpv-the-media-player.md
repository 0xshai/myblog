---
title: "mpv：一个播放器的极简哲学"
date: 2026-06-13
tags: ["工具", "开源", "视频", "Windows"]
description: "从安装到调教，系统介绍 mpv 这个被开发者、摄影师、影迷奉为圭臬的命令行播放器。"
---

大多数人选择播放器的逻辑是：哪个界面好看用哪个。mpv 反其道而行之——它几乎没有界面，但在这个领域沉淀了十几年，成为画质、格式兼容、可扩展性三者同时在线的播放器里，公认最难被取代的一个。

## 从哪里来

mpv 的历史要从 2000 年说起。那年，匈牙利程序员 Árpád Gereöffy 发布了 MPlayer，那是 Linux 世界里第一个真正能打的开源播放器，啥格式都能播，但代码越积越乱。2010 年，一批开发者受不了了，fork 出 mplayer2，清理了一批积重难返的旧代码。

2012 年，德国开发者 Vincent Lang（网名 wm4）觉得 mplayer2 也走偏了，再次 fork，这次改名 mpv。他的目标只有一个：**把播放器本身做到极致，其他交给用户自己配**。

2013 年 8 月 7 日，mpv 0.1.0 正式发布。

此后十二年，mpv 由社区持续维护，截至 2025 年 12 月最新稳定版是 0.41.0。代码托管在 GitHub，GPLv2+ 授权，用 C 语言写成，底层依赖 FFmpeg 解码。它跑在 Linux、macOS、Windows、BSD，甚至 Android（mpv-android）。全平台一套逻辑，配置文件通用。

在播放器领域，mpv 的地位有点像 Vim 在编辑器里的地位：不适合所有人，但适合它的人很难换掉它。

---

## 为什么值得用

**格式支持。** 底层是 FFmpeg，理论上 FFmpeg 能解的它都能播。MKV、AV1、HEVC、Dolby Vision、HDR10+，不需要装额外解码器。

**画质渲染。** mpv 的视频渲染管线是专门为高质量播放设计的，支持色彩管理、HDR tone mapping、高精度抖动、帧插值。同一个文件，mpv 和 VLC 的画面肉眼可辨。

**可扩展。** Lua 脚本体系成熟，社区有几百个脚本，从字幕管理到着色器切换都有现成的。

**轻量。** 启动速度快，内存占用低，没有后台服务。

---

## 安装

已经用 Scoop 的话，一行搞定：

```powershell
scoop install mpv
```

装完在 `%APPDATA%\mpv\` 下建立配置目录，mpv 会自动识别。

---

## 配置文件

mpv 的所有行为都通过配置文件控制。核心文件是 `%APPDATA%\mpv\mpv.conf`，不存在就新建。

下面是一份适合日常使用的基础配置，每行都有注释：

```ini
# ── 画质 ────────────────────────────────────────────────
profile=high-quality          # 官方高质量预设（旧版叫 gpu-hq）
vo=gpu-next                   # 新一代 GPU 渲染后端，推荐开启
video-sync=display-resync     # 视频帧率与显示器同步，减少撕裂
interpolation=yes             # 帧插值，运动更流畅（需要 GPU 有余量）
tscale=oversample             # 插值算法，oversample 适合大多数内容

# ── 硬件解码 ────────────────────────────────────────────
hwdec=auto-safe               # 自动硬解，safe 模式避免兼容性问题

# ── 音频 ────────────────────────────────────────────────
volume=100
volume-max=150                # 允许超过 100% 音量
audio-normalize-downmix=yes   # 多声道降混时自动归一化

# ── 字幕 ────────────────────────────────────────────────
sub-auto=fuzzy                # 自动加载同目录下文件名相近的字幕
sub-font=Noto Sans CJK SC     # 中文字幕字体（按实际安装字体修改）
sub-font-size=44
sub-color=1.0/1.0/1.0         # 白色字幕
sub-border-color=0.0/0.0/0.0  # 黑色描边
sub-border-size=3
sub-shadow-offset=1

# ── 截图 ────────────────────────────────────────────────
screenshot-format=png
screenshot-high-bit-depth=yes
screenshot-directory=~/Pictures/mpv-screenshots

# ── 窗口行为 ─────────────────────────────────────────────
keep-open=yes                 # 播放结束后不关闭窗口
save-position-on-quit=yes     # 退出时记住播放位置，下次继续
autofit=90%x90%               # 窗口最大占屏幕 90%
```

保存后重启 mpv 立即生效。

---

## 快捷键

mpv 默认绑定了一批快捷键，高频的如下：

| 键 | 功能 |
|---|---|
| `空格` | 播放 / 暂停 |
| `←` / `→` | 后退 / 前进 5 秒 |
| `Shift+←` / `Shift+→` | 后退 / 前进 1 秒 |
| `↑` / `↓` | 音量 +2 / -2 |
| `f` | 全屏切换 |
| `m` | 静音 |
| `s` | 截图（含字幕） |
| `S` | 截图（不含字幕） |
| `[` / `]` | 播放速度 -0.1 / +0.1 |
| `{` / `}` | 播放速度减半 / 加倍 |
| `` ` `` 或 `反引号` | 显示播放信息 |
| `q` | 退出 |
| `Q` | 退出并保存进度 |

自定义快捷键放在 `%APPDATA%\mpv\input.conf`：

```ini
# 示例：用 Ctrl+左右快进快退 60 秒
Ctrl+RIGHT seek 60
Ctrl+LEFT seek -60

# 用 Alt+s 截图不含字幕
Alt+s screenshot video
```

---

## 加入右键菜单

Scoop 安装的 mpv 不会自动注册文件关联。用以下方法解决：

**方法一：用 mpv-install-for-scoop 脚本**

```powershell
# 先安装 sudo（需要管理员权限）
scoop install sudo

# 进入 mpv 持久化目录
cd ~\scoop\persist\mpv

# 下载注册脚本
git clone https://github.com/koyokr/mpv-install-for-scoop mpv-install
cd mpv-install

# 以管理员权限运行
sudo .\mpv-install.bat
```

之后在「设置 → 应用 → 默认应用」里把视频播放器改成 mpv 即可。

**方法二：手动注册表**

如果只想在右键菜单里多一个「用 mpv 打开」而不改默认应用，可以手动写注册表。新建一个 `.reg` 文件：

```reg
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\*\shell\Open with mpv]
@="用 mpv 打开"
"Icon"="C:\\Users\\你的用户名\\scoop\\apps\\mpv\\current\\mpv.exe"

[HKEY_CLASSES_ROOT\*\shell\Open with mpv\command]
@="\"C:\\Users\\你的用户名\\scoop\\apps\\mpv\\current\\mpv.exe\" \"%1\""
```

路径改成实际的 mpv.exe 位置，双击导入注册表即可。

---

## 换皮肤：OSC 替换

mpv 默认的 OSC（屏幕控制条）是简洁但略显朴素的风格。社区提供了几个成熟的替换方案：

### uosc — 功能最全

[uosc](https://github.com/tomasklaen/uosc) 是目前用户最多的 OSC 替代，提供现代化的进度条、音量滑块、字幕轨道切换、播放列表面板等，本身就是一套完整的 UI 框架。

安装步骤：

```powershell
# 下载 uosc.lua 放入 scripts 目录
# 目录位置：%APPDATA%\mpv\scripts\

# 同时需要关闭内置 OSC，在 mpv.conf 加入：
# osc=no
```

然后在 `mpv.conf` 里加：

```ini
osc=no
border=no   # 可选，无边框窗口更现代
```

### ModernZ — 更简洁

[ModernZ](https://github.com/Samillion/ModernZ) 是 ModernX 的增强 fork，界面干净，支持多主题、多布局，还有图片浏览模式。安装同样是把 `.lua` 文件放进 `scripts/` 目录。

脚本统一放在 `%APPDATA%\mpv\scripts\` 目录下，mpv 启动时自动加载。

---

## 着色器：画质增强

这是 mpv 最有意思的部分。GLSL 着色器可以在 GPU 上实时处理画面，效果从轻微锐化到 AI 超分都有。

着色器文件放在 `%APPDATA%\mpv\shaders\` 目录下。

### Anime4K — 动画超分

[Anime4K](https://github.com/bloc97/Anime4K) 是专门针对动画内容的实时超分着色器，能让低分辨率动画在 1080p 甚至 4K 屏幕上显得更锐利。

下载 Anime4K 的 GLSL 包，解压到 `%APPDATA%\mpv\shaders\`。

然后在 `input.conf` 里绑定快捷键切换：

```ini
CTRL+1 no-osd change-list glsl-shaders set "~~/shaders/Anime4K_Clamp_Highlights.glsl:~~/shaders/Anime4K_Restore_CNN_M.glsl:~~/shaders/Anime4K_Upscale_CNN_x2_M.glsl:~~/shaders/Anime4K_AutoDownscalePre_x2.glsl:~~/shaders/Anime4K_AutoDownscalePre_x4.glsl:~~/shaders/Anime4K_Upscale_CNN_x2_S.glsl"; show-text "Anime4K: Mode A (Fast)"
CTRL+0 no-osd change-list glsl-shaders clr ""; show-text "Shaders cleared"
```

`Ctrl+1` 开启，`Ctrl+0` 关闭。

### ArtCNN — 更适合真实内容

相比 Anime4K 专注动画，[ArtCNN](https://github.com/Artoriuz/ArtCNN) 是一个轻量 CNN 超分着色器，对真实内容（电影、纪录片）效果更好，GPU 消耗也更低。

```ini
# 在 mpv.conf 中固定启用
glsl-shaders="~~/shaders/ArtCNN_C4F32.glsl"
```

> **注意**：着色器会消耗 GPU 资源，播放 4K 内容时建议先确认帧率正常，否则得不偿失。可以按反引号键查看实时帧率。

---

## 字幕样式调整

mpv 区分两种字幕：ASS/SSA 格式（自带样式信息）和 SRT 等纯文本格式（由播放器渲染样式）。

**对纯文本字幕（SRT），直接在 `mpv.conf` 控制：**

```ini
sub-font=Noto Sans CJK SC
sub-font-size=44
sub-color=1.0/1.0/1.0/1.0      # RGBA，白色不透明
sub-border-color=0.0/0.0/0.0/1.0
sub-border-size=3
sub-shadow-color=0.0/0.0/0.0/0.5
sub-shadow-offset=2
sub-margin-y=36                 # 离底部的距离
sub-ass-force-margins=yes       # 防止字幕超出视频边界
```

**对 ASS 字幕，如果想强制覆盖原有样式：**

```ini
sub-ass-override=force          # 强制用上面的样式覆盖 ASS 内置样式
```

不加这行的话 ASS 字幕会保持原有样式（通常是动画的特效字幕），加了之后全部统一成你设置的样式。按需选择。

---

## 播放网络视频

mpv 内置支持 yt-dlp，可以直接播放 YouTube 等平台的视频：

```powershell
# 先安装 yt-dlp
scoop install yt-dlp

# 然后直接传 URL 给 mpv
mpv https://www.youtube.com/watch?v=xxxxxx
```

在 `mpv.conf` 里指定默认画质：

```ini
ytdl-format=bestvideo[height<=1080]+bestaudio/best[height<=1080]
```

---

## 播放列表与批量播放

```powershell
# 播放整个目录
mpv /path/to/folder/

# 播放目录并随机排序
mpv --shuffle /path/to/folder/

# 循环播放
mpv --loop-playlist=inf video.mp4
```

在播放中：`<` 和 `>` 切换上一个 / 下一个文件。

---

## 几个实用小技巧

**A-B 循环**：按 `l` 设置 A 点，再按 `l` 设置 B 点，mpv 会在这个片段内循环。再按一次取消。学外语、反复研究某个镜头都好用。

**截图后处理**：截图默认是 PNG，体积较大。如果想要 JPEG：

```ini
screenshot-format=jpg
screenshot-jpeg-quality=90
```

**查看文件信息**：播放时按 `` ` ``（反引号），会显示编码、分辨率、帧率、音频轨道等完整信息，和 MediaInfo 能看到的差不多。

**控制台**：按 `` ~ `` 打开内置控制台，可以实时输入命令，比如 `set volume 80`、`seek 30` 等，调试配置很方便。

---

## 目录结构总览

配置完成后，`%APPDATA%\mpv\` 大致是这个结构：

```
mpv/
├── mpv.conf          # 主配置文件
├── input.conf        # 快捷键绑定
├── scripts/          # Lua 脚本（uosc、ModernZ 等放这里）
│   └── uosc.lua
├── shaders/          # GLSL 着色器
│   ├── Anime4K_*.glsl
│   └── ArtCNN_*.glsl
└── fonts/            # 自定义字体（字幕用）
```

---

## 延伸阅读

- [mpv 官方文档](https://mpv.io/manual/stable/) — 最全，但信息量大，建议按需查
- [awesome-mpv](https://github.com/stax76/awesome-mpv) — 社区整理的脚本、着色器、工具合集
- [thewiki.moe/tutorials/mpv](https://thewiki.moe/tutorials/mpv/) — 针对动画内容的详细配置指南

---

mpv 的学习曲线比大多数播放器陡，但一旦配好，很难再想换别的。它的设计哲学是：**播放器只负责播放，其余的交给你**。这和很多工具的思路一样——少做但做好，留足空间给用户自己决定。
