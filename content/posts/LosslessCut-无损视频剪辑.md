---
title: "40k Stars 开源工具 LosslessCut：把 FFmpeg 的能力装进 GUI"
date: 2025-06-17
tags: ["开源工具", "视频处理", "FFmpeg", "Windows"]
draft: false
---

剪视频，不用重新编码。

---

用 Premiere 或剪映剪掉几段素材，等待导出的时间总让人抓狂——几分钟的片段，渲染半小时，画质还损失一代。这叫 generation loss，每次重新编码都会叠加损耗。其实大多数情况下你根本不需要重新编码，只是想掐头去尾、提取一段、拼几个片段而已。

**[LosslessCut](https://github.com/mifi/lossless-cut)** 就是为这个场景生的。它调用捆绑的 FFmpeg，用 stream copy 模式直接复制数据流，不解码不重编，几秒钟处理几 GB 的文件，零质量损失。GitHub 上 40.9k+ Stars，挪威开发者 Mikael Finstad 独立维护，被 Hacker News、Wikipedia、YouTube 大 V 多次推荐。定位介于 Avidemux 和专业 NLE 之间——比前者强大，比后者轻快。

> "The swiss army knife of lossless video/audio editing"

GitHub 版本永久免费开源，这是项目的核心理念。

## 关于作者

Mikael Finstad，挪威独立开发者，网站 [mifi.no](https://mifi.no/)，GitHub [@mifi](https://github.com/mifi)，2k+ followers。全职 freelance，技术栈以 JavaScript 为主：React、React Native、Node.js、Electron。他曾担任 SwimClips.com 的 CTO，2009 年成立了自己的软件公司 Yankee Software，主要以承接开发外包为生。

除了 LosslessCut（2016 年首发，持续维护至今），他还做了不少值得一提的开源项目：

**[Editly](https://github.com/mifi/editly)**（5.3k Stars）：和 LosslessCut 定位互补，后者负责无损剪切，前者负责"用代码剪视频"——基于 Node.js + FFmpeg 的声明式视频编辑工具，写一个 JSON5 配置文件就能自动合成带转场、字幕、背景音乐的视频。支持 GL shader 转场效果、Canvas/Fabric.js 自定义层、Ken Burns 效果，也可以直接当 CLI 或 npm 库用。适合批量生成视频或自动化视频制作流程。

**[SimpleInstaBot](https://mifi.github.io/SimpleInstaBot)**：Instagram 自动化工具，现在基本因平台限制而式微。

**[reactive-video](https://github.com/mifi/reactive-video)**：用 React 写视频，每帧渲染一个 React 组件，走浏览器无头渲染后用 FFmpeg 合成，是 Editly 的 React 版替代方案。

**[instauto](https://github.com/mifi/instauto)**：Instagram 自动化 API 库。

**其他小工具**：`stacktracify`（混淆 stack trace 反混淆）、`ical-expander`（iCal 日历展开工具）、`hls-vod`（HLS 流媒体服务器）、`cognito-backup` / `dynamodump`（AWS 数据备份）等，基本都是他自己用得上然后开源出来的实用工具。

他在 GitHub README 里写的一句话很能概括他的创作动机：

> "I create free and open source software to share with the world because I believe software should be available to everyone."

平时爱好是旅行和徒步，Instagram 上常发挪威风光照，个人网站上还做了个叫 [miffy.no](https://miffy.no/) 的小游戏。

---

## 功能全览

所有操作都优先尝试 stream copy，能无损就绝不重编。

### 基础剪切（Lossless Cutting & Trimming）

在 Timeline 上用 `I` / `O` 设入出点，创建 Segments，导出时只保留选定部分。典型场景是从几 GB 的 GoPro / 无人机 / 相机文件里快速挖出精彩片段，丢弃无用素材。

**关于切割精度**：切割起点会对齐到最近的前一个关键帧（GOP 对齐），不是精确到帧的切割。这是 H.264/H.265 编码结构决定的，不是 bug。需要帧级精确可以开启实验性的 Smart Cut，它会在切点附近做小范围 re-encode 来提高精度。

### 反转模式与片段重排

点击左下角"婴儿图标"启用高级视图，出现 Yin Yang ☯️ 按钮。点击后从"保留选中片段"变为"跳过选中片段"——专门用来切广告、删掉中间某段。

Segments 面板支持拖拽重新排序，导出时按新顺序合并或分别输出。

### 无损合并/拼接

将多个 codec 参数相同的文件（比如同一台相机连续拍的分段文件）无损拼接成一个。底层是 FFmpeg concat demuxer，选 Merge cuts 导出模式即可。

### 多轨道编辑（最强大的功能之一）

这是 LosslessCut 区别于简单切割工具的核心能力：

- 从多个文件组合任意轨道：把外部音乐音频 + 字幕轨道合并进视频
- 移除不需要的轨道（多余语言音轨、附件等）
- 仅替换部分轨道：保留视频无损，只 re-encode 音频
- 提取所有轨道：把视频、音频、字幕、附件分离到独立文件
- Tracks 面板查看每条轨道的 codec、语言、disposition 等技术信息
- 编辑文件级和轨道级 metadata（标题、作者、语言、GPS 等）
- 支持同时播放多个音频轨道

实用场景：多语言音频切换、替换背景音乐、分离/合并字幕、从视频提取纯音频剪辑。

### 格式 Remux（无损容器转换）

把 H.264/H.265 的 MKV 转成 MP4 / MOV，codec 数据一字节不动，只换容器外壳，秒完成。iPhone 不认 MKV 但能播 MP4 的问题，这样解决最合适。

### 快照与帧导出

- 全分辨率快照：当前时间点导出 JPEG / PNG（低/高质量可选）
- 范围帧导出：每 N 帧、每秒、按场景变化、按最佳缩略图导出图像序列
- 支持只从选定 Segment 范围导出
- 文件名可包含原始时间戳
- 高级用法：从 keyframes 创建 segments → 转为 markers → 批量提取为图像

### Timeline 高级交互

- 缩略图 + 音频波形可视化
- 关键帧跳转，精确在 I-frame 附近定位
- 时间码偏移（per-file timecode offset），支持从文件自动加载
- 旋转/方向元数据修改：修复手机横竖屏录制方向错误，不 re-encode，只改 flag
- 黑场 / 静音 / 场景变化检测：FFmpeg filter 扫描后自动生成 Segments
- 智能分割：按固定时长、文件大小（X MB）、N 个片段数、甚至随机化来切分 Timeline

### Segments 管理

- **Segments**：有 start/end 的时间片段，导出时保留
- **Markers**：无 end time 的点标记，不参与导出
- 每个 Segment 可添加 label 和多个 tags，按 tag 创建输出文件夹结构
- 右侧面板列表显示，右键菜单可编辑、提取帧，拖拽排序影响合并顺序

### 项目持久化与格式互导

自动保存 per-project 数据到 `.llc` 文件（JSON5 格式），退出后可恢复。支持多种格式导入导出 Segments：

- MP4 / MKV 内置章节标记（chapters）
- 纯文本、CSV / TSV（EDL 格式：start, end, label）
- YouTube Chapters
- CUE
- XML（DaVinci Resolve, Final Cut Pro）

内置 MKV / MP4 章节编辑器，支持查看内嵌字幕。

### JS 表达式语言（高级功能）

可以用 JavaScript 表达式查询、筛选、修改 Segments：

```javascript
segment.label === 'highlight' && segment.duration < 5
```

配合 `selectSegmentsByExpr`、`edit segments by expression` 等操作，实现按时长/标签过滤后批量处理。结合热键绑定可以做到"一次配置，重复用于多个文件"。

### 其他值得一提的功能

- 速度调整：加速/减速视频或音频（支持改变 FPS）
- DJI GPS 轨迹地图显示（Leaflet，UI 内展示）
- HTTP 无损下载：支持 HLS `.m3u8` 等流媒体直接下载并切割
- 极速移除所有非关键帧：适合 timelapse 处理
- Undo/Redo + FFmpeg 最后命令日志查看（可复制到终端修改后重跑）
- Basic CLI & HTTP API：支持命令行和远程 HTTP 控制
- Batch list：批量打开多文件逐个编辑导出

**明确不支持的操作**（需要 re-encode）：水印/文字叠加、模糊打码、颜色分级、转场、音量调整、GIF 生成、烧字幕、resize/stretch。

---

## 安装

### Windows 直接下载（推荐）

去 [GitHub Releases](https://github.com/mifi/lossless-cut/releases/latest) 下载 Windows x64 的 `.7z` 压缩包，解压即用，不需要安装程序。注意 v3.50.0 之后不再支持 Win7/8/8.1。

配置文件、快捷键、日志、缓存存在 `%APPDATA%\LosslessCut`。卸载时删掉 app 文件夹 + 这个目录就干净了。

想要自动更新可以从 **Microsoft Store** 付费购买（签名版，功能相同，稍滞后 GitHub）。

### 其他平台

| 平台 | 免费下载 | 付费（自动更新） |
|------|----------|------------------|
| macOS | Intel / Apple Silicon DMG（注意 PKG 不工作） | Mac App Store（沙盒化，稍滞后） |
| Linux | x64 tar.bz2 / AppImage / arm64 / armv7l（树莓派） | Flathub / Snap Store |

Nightly builds（实验性）：`https://mifi.no/llc/nightly/`

### 系统要求

- Windows 10+，macOS 12+，现代 Linux
- 任何能跑 Electron 的机器都行，预览用 Chromium 硬件解码
- 处理大文件建议 SSD，项目本身很轻量，瓶颈在磁盘和 FFmpeg

### 从源码构建

```bash
# 克隆仓库
git clone https://github.com/mifi/lossless-cut.git
cd lossless-cut

# 安装依赖
yarn install

# 下载对应平台的 FFmpeg（必须，项目捆绑自定义构建）
yarn download-ffmpeg-win32-x64    # Windows
yarn download-ffmpeg-darwin-arm64 # macOS Apple Silicon
yarn download-ffmpeg-linux-x64    # Linux

# 开发运行
yarn dev

# 打包
yarn pack-win    # Windows 7z
yarn pack-mac    # macOS DMG
yarn pack-linux  # Linux AppImage 等
```

项目用 TypeScript 严格模式 + ESLint + Vitest，有贡献意向看 `CONTRIBUTING.md`，支持多语言翻译。

---

## 基本操作流程

1. **打开文件**：拖拽或 `Ctrl + O`，支持批量拖入 Batch list
2. **导航预览**：`Space` 播放/暂停；方向键、`,` `.` 或滚轮 seek；Timeline 支持缩放和关键帧跳转
3. **创建 Segments**：
   - `I` 设入点，`O` 设出点
   - `+` 新增 Segment，`B` 在当前位置分割，`Backspace` 删除
   - `Shift` 拖拽可移动/调整 Segment 大小
4. **启用高级视图**：点左下角婴儿图标，展开 Tracks、Yin Yang、Export options 等按钮
5. **轨道管理**：Tracks 面板选择/添加/移除轨道，可从其他文件引入
6. **自动检测**：Tools → 场景/静音/黑场检测，或 Divide timeline into segments
7. **导出设置**：
   - Export mode：Separate files（默认）或 Merge cuts
   - 输出文件名模板支持变量：`${SEG_NUM_INT}`、`${EXPORT_COUNT}` 等
8. **执行导出**：`E` 键，查看 FFmpeg 命令日志，保存 `.llc` 项目

### 快捷键速查

| 操作 | 快捷键 |
|------|--------|
| 设入点 | `I` |
| 设出点 | `O` |
| 新增 Segment | `+` |
| 在当前位置分割 | `B` |
| 删除 Segment | `Backspace` |
| 播放/暂停 | `Space` |
| 导出 | `E` |
| 查看/编辑全部快捷键 | `Shift + /` |

---

## 实战场景

**去广告/删中间段**：用 Yin Yang 反转模式，标出要删的部分，导出剩余内容。

**导出 YouTube Chapters**：Merge cuts → Create chapters from merged segments → File → Export project → YouTube Chapters。

**仅重编音频保留视频**：提取视频/音频轨道 → HandBrake 处理音频 → LosslessCut 打开视频 + include 新音频轨道 → 导出。

**多文件剪辑后合并**：Batch list 配合自定义输出文件名模板（带序号）→ 分别导出 → 拖入排序后 Merge。

**提取 Keyframes 为图像序列**：Tools → Create segments from keyframes → 转为 markers → Extract frames as images。

**重复性自动化**：`toggleStripAll` + Filter tracks 表达式 + `selectSegmentsByExpr` + 热键绑定，一键准备 + 导出。

LosslessCut 不是完整 NLE，多文件编辑需分步进行。但通过 CSV/EDL/XML 可以和 DaVinci / FCP 协同工作。

---

## 技术原理

### 架构

```
┌─────────────────────────────────────────────────────────┐
│                  UI Layer                               │
│  React + Radix UI + 自定义 Timeline                    │
│  HTML5 Video 预览（Chromium 硬件解码）+ Leaflet 地图  │
├─────────────────────────────────────────────────────────┤
│              Electron Main Process                      │
│              Node.js / TypeScript                       │
│  项目状态管理（JSON5 .llc）、全局快捷键、CLI/HTTP API │
├─────────────────────────────────────────────────────────┤
│                  Core Media Engine                      │
│         Bundled FFmpeg（各平台自定义构建）             │
│  stream copy、concat demuxer、-map 多轨道              │
│  blackdetect / silencedetect / scene change filters    │
├─────────────────────────────────────────────────────────┤
│              Output & Import/Export                    │
│  Separate files / Merged、Chapters、CSV/EDL/XML        │
└─────────────────────────────────────────────────────────┘
```

技术栈：Electron + Vite + React + TypeScript。Frontend 用 @radix-ui/themes + DND Kit（拖拽）+ @tanstack/react-virtual（虚拟列表）+ Leaflet（GPS 地图）。打包用 electron-vite + electron-builder。

项目不依赖系统 FFmpeg，而是为每个平台捆绑自定义构建（存于 `ffmpeg/<platform>/`），通过 `child_process.spawn` 调用，保证跨平台一致性。

### Lossless 的本质

1. **Stream Copy（`-c copy`）**：直接复制 codec 数据流，不解码不重编。这是"几秒完成 + 零质量损失"的根本原因。

2. **关键帧对齐**：视频编码的 GOP 结构决定了切割点只能对齐到 I-frame，这就是"start time rounded to previous keyframe"的来源。

3. **Smart Cut**：在 cut 点附近做有限 re-encode 来提高精度，可通过命令日志观察具体实现。

4. **多轨道**：大量使用 `-map` 参数精确选择输入流，支持多输入文件组合。

5. **合并（Concat）**：生成临时 concat demuxer 文件列表，然后：
   ```
   ffmpeg -f concat -safe 0 -i list.txt -c copy output.mp4
   ```

6. **检测功能**：调用 FFmpeg filters（`blackdetect`、`silencedetect`、`select` for scene change），解析 stdout/stderr 生成 Segments。

7. **元数据/旋转修改**：通过 `-metadata`、`-disposition`、`-rotation` 等选项在 remux 时修改，不触碰 codec 数据。

每次导出后可以查看完整的 FFmpeg 命令日志，复制到终端修改后重跑——学 FFmpeg 的好资料。

### 数据模型

`.llc` 文件是 JSON5 格式，存储 segments 数组（含 start/end/label/tags）、tracks 配置、metadata、timecode offset 等。Segments 数据结构支持 JS 表达式直接访问属性，这是 JS 表达式语言功能的基础。

### 已知限制

- 部分容器/codec 的精确切割困难（依赖 keyframe 位置）
- 某些轨道类型无法切割
- App Store 版本有沙盒限制（VOB 文件、写权限提示）
- 不支持原地覆盖输入文件（设计上禁止，防止数据丢失）
- 处理超大文件时 FFmpeg 可能使用临时文件，建议预留充足磁盘空间

---

## 总结

- GitHub：[mifi/lossless-cut](https://github.com/mifi/lossless-cut)
- 协议：GPL-2.0
- 跨平台：Windows / macOS / Linux

LosslessCut 没有华而不实的 UI，就是把 FFmpeg 的强大能力通过键盘驱动的工作流交给普通用户。日常粗剪大文件、多音轨混剪、EDL 工作流对接 DaVinci，还是单纯想学 FFmpeg 命令，它都是目前 GitHub 上最成熟的无损视频处理方案。

配合 HandBrake 处理需要 re-encode 的部分，基本覆盖日常视频处理的所有场景。
