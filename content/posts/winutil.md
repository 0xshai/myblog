---
title: "WinUtil：Windows 一键优化工具"
date: 2026-06-24
description: "Chris Titus 出品的 Windows 系统优化工具，软件安装、系统调整、隐私清理一站搞定，一条命令启动，30 million+ 次运行验证。"
tags: ["Windows", "工具", "开源", "效率"]
categories: ["工具"]
showToc: true
TocOpen: false
draft: false
---

[WinUtil](https://christitustech.github.io/winutil/) 是 Chris Titus Tech 开发的开源 Windows 优化工具，6 年迭代、200+ 贡献者、3000 万次运行记录。无需安装，一条 PowerShell 命令直接启动，覆盖软件批量安装、系统减肥、隐私加固、更新策略等常见需求。

## 启动

以管理员身份打开 PowerShell 或 Windows Terminal，运行：

```powershell
irm christitus.com/win | iex
```

> **提示**：`christitus.com/win` 是官方短链，等价于从 GitHub 拉取最新脚本，每次运行都是最新版本。需开代理访问。

> **注意**：必须以**管理员身份**运行，否则大部分功能无法生效。右键开始菜单 → Windows Terminal（管理员）。
>
> PowerShell 7 有时会把脚本内容误判为 HTML 导致解析报错，换 `powershell.exe`（5.1）即可：
> ```powershell
> powershell.exe -Command "irm christitus.com/win | iex"
> ```

---

## Install（软件安装）

底层调用 **winget** 批量安装，省去逐个下载和点击安装的麻烦。

界面上按分类折叠展示（浏览器、开发工具、媒体、系统工具等），勾选需要的软件后点 **Install**。

- **Get Installed**：扫描系统已装软件，自动勾选对应条目，方便迁移新机
- 取消勾选后点 **Uninstall** 即可卸载

---

## Tweaks（系统调整）

这是 WinUtil 的核心，分四个区块。

### 快速预设

页面顶部有四个快速按钮：

| 预设 | 说明 |
|------|------|
| Standard | 推荐大多数用户，安全可逆的基础优化 |
| Minimal | 低影响的最小化调整，保守首选 |
| Advanced | 针对有经验用户的进阶组合，跳过还原点和清理任务以加快执行速度 |
| Clear | 清除所有已选项 |

点 **Get Installed Tweaks** 可检测系统当前已应用的 tweak 状态。

### Essential Tweaks（基础调整）

适合所有用户，安全且可逆，从这里开始。

| 项目 | 说明 |
|------|------|
| Activity History - Disable | 关闭活动历史记录，减少本地行为追踪 |
| BitLocker - Disable | 关闭 BitLocker 加密（个人机常见需求） |
| ConsumerFeatures - Disable | 关闭微软消费者推送功能（商店推荐等广告行为） |
| Delivery Optimization - Disable | 禁用 P2P 更新分发，不把你的带宽分享给其他设备 |
| Disk Cleanup - Run | 立即清理系统垃圾文件 |
| End Task With Right Click - Enable | 右键任务栏应用可直接结束任务，省去打开任务管理器 |
| File Explorer Automatic Folder Discovery - Disable | 关闭资源管理器自动检测文件夹类型，打开大文件夹更快 |
| Hibernation - Disable | 关闭休眠，释放等同内存大小的 C 盘空间（hiberfil.sys） |
| Location Tracking - Disable | 关闭系统级位置追踪 |
| Microsoft Store Recommended Search Results - Disable | 搜索栏不再弹出商店推荐结果 |
| Restore Point - Create | 执行 tweaks 前自动创建还原点（**建议始终勾选**） |
| Services - Set to Manual | 将非必要服务改为手动启动，减少后台进程 |
| Start Menu Previous Layout - Enable | 恢复 Windows 10 风格的开始菜单布局 |
| Telemetry - Disable | 关闭微软遥测数据收集 |
| Temporary Files - Remove | 清理 %temp% 等临时文件目录 |
| Unwanted Pre-Installed Apps - Remove | 卸载预装的臃肿软件（各种游戏、Candy Crush 等） |
| Widgets - Remove | 移除任务栏 Widgets 新闻推送 |
| Windows Platform Binary Table (WPBT) - Disable | 禁用 WPBT，阻止厂商在系统启动时强制注入程序 |

### Advanced Tweaks（进阶调整，谨慎使用）

> **注意**：这里的每一项都有明确适用场景，**不要全勾**。先理解每项的作用再决定是否启用。

| 项目 | 说明 |
|------|------|
| Adobe URL Block List - Enable | 屏蔽 Adobe 的追踪域名 |
| Background Apps - Disable | 禁止后台应用运行（可能影响部分应用通知） |
| Brave Browser - Debloat | 移除 Brave 内置的 VPN、Rewards 等推广组件 |
| Date & Time - Set Time to UTC | 系统时钟改用 UTC，双系统（Windows + Linux）用户必选，否则两个系统时间会相互覆盖 |
| Disable Reserved Storage | 释放微软为系统更新预留的约 7GB 磁盘空间 |
| File Explorer Home and Gallery - Disable | 移除资源管理器左侧的"主页"和"图库"入口 |
| Fullscreen Optimizations - Disable | 关闭全屏优化，部分老游戏有改善，新游戏影响不大 |
| IPv6 - Disable | 完全禁用 IPv6（仅在确认不需要时启用） |
| IPv6 - Set IPv4 as Preferred | 保留 IPv6 但优先使用 IPv4 |
| Microsoft Edge - Debloat | 移除 Edge 内置的侧边栏、购物助手等冗余功能 |
| Microsoft Edge - Remove | 彻底卸载 Edge（需要备用浏览器） |
| Microsoft OneDrive - Remove | 彻底卸载 OneDrive |
| O&O ShutUp10++ - Run | 启动 OO ShutUp10++，更精细的隐私开关 GUI |
| Razer Software Auto-Install - Disable | 禁止 Razer 硬件自动安装 Synapse（有 Razer 设备但不想要管理软件时） |
| Right-Click Menu Previous Layout - Enable | 恢复 Windows 10 的右键菜单（不需要点"显示更多选项"） |
| Storage Sense - Disable | 关闭 Storage Sense 自动清理（避免误删文件） |
| System Tray Notifications & Calendar - Disable | 禁用系统托盘通知和日历弹窗 |
| Teredo - Disable | 关闭 Teredo IPv6 隧道，改善部分网络情况下的延迟 |
| Visual Effects - Set to Best Performance | 关闭动画和视觉特效，老机或虚拟机提速明显 |
| Windows AI - Disable And Remove | 移除 Copilot、Recall 等 AI 组件 |
| Xbox & Gaming Components - Remove | 卸载 Xbox Game Bar 等游戏组件（不玩游戏时可移除） |

### Customize Preferences（外观与行为偏好）

小开关集合，按需勾选：

| 项目 | 说明 |
|------|------|
| Dark Theme | 系统深色模式 |
| Enable Long Paths | 启用长路径支持（开发环境常见需求） |
| File Explorer File Extensions | 资源管理器显示文件扩展名 |
| File Explorer Hidden Files | 显示隐藏文件 |
| Game Mode | 开/关 Windows 游戏模式 |
| Mouse Acceleration | 鼠标加速开关（游戏玩家通常关闭） |
| Num Lock on Startup | 开机自动开启 NumLock |
| Scrollbars Always Visible | 滚动条常驻显示 |
| Start Menu Bing Search | 开始菜单 Bing 联网搜索开关 |
| Start Menu Recommendations | 开始菜单推荐内容开关 |
| Sticky Keys | 粘滞键开关 |
| Taskbar Centered Icons | 任务栏图标居中/居左切换 |
| Taskbar Search Icon | 任务栏搜索图标显示方式 |

### DNS 设置

无需手动改网络适配器，直接在此切换 DNS，IPv4 和 IPv6 同时生效：

| 选项 | 说明 |
|------|------|
| Default | ISP 默认 DNS |
| DHCP | 自动获取 |
| Google | 8.8.8.8，稳定快速 |
| Cloudflare | 1.1.1.1，速度与隐私兼顾 |
| Cloudflare_Malware | 屏蔽恶意软件域名 |
| Cloudflare_Malware_Adult | 同时屏蔽恶意软件和成人内容 |
| Open_DNS | 可定制过滤规则 |
| Quad9 | 9.9.9.9，以安全为主，屏蔽已知恶意域名 |
| AdGuard_Ads_Trackers | 屏蔽广告和追踪器 |
| AdGuard_Ads_Trackers_Malware_Adult | 全面屏蔽广告、追踪、恶意软件和成人内容 |

### Performance Plans（性能计划）

- **Add Ultimate Performance**：启用终极性能电源计划，最小化延迟、最大化效率，适合台式机
- **Remove Ultimate Performance**：切回均衡模式（笔记本省电时使用）

### 撤销

- **Undo Selected Tweaks**：仅撤销已勾选的项目
- 还原点：执行前创建了还原点的话，进入系统属性 → 系统保护 → 系统还原，可完整回滚

---

## Config（功能配置）

面向有经验用户，提供组件开关和修复工具。

### 可选功能开关

| 功能 | 说明 |
|------|------|
| .NET Framework 2/3/4 | 按需启用指定版本 |
| Hyper-V | 启用 Windows 内置虚拟化 |
| Windows Sandbox | 启用沙盒隔离环境 |
| WSL | 启用 Windows Subsystem for Linux |
| NFS | 启用网络文件系统支持 |
| Registry Backup | 每天凌晨 12:30 自动备份注册表 |
| Legacy F8 Boot Recovery | 开/关 F8 启动恢复 |

### 修复工具

| 工具 | 说明 |
|------|------|
| Reset Network | 重置网络栈（`netsh int ip reset` + `netsh winsock reset`），网络玄学问题首选 |
| Reset Windows Update | 重新注册更新 DLL，重启更新服务 |
| System Corruption Scan | 运行 `sfc /scannow` 和 `DISM /RestoreHealth`，耗时较长，作为最后手段 |
| WinGet Repair | 修复 winget 安装失败问题 |
| AutoLogon | 配置开机自动登录 |
| NTP Server | 启用 NTP 时间同步 |

### 经典控制面板快捷入口

WinUtil 在此提供了 Windows 7 时代的经典控制面板直接入口，比现代设置 App 功能更完整：控制面板、设备管理器、声音设置、打印机、网络连接、电源选项等。

### PowerShell Profile

**CTT PowerShell Profile - Install**：安装 Chris Titus 的 PowerShell 主题，带自动补全和 oh-my-posh 风格提示，仅支持 PowerShell 7+。

---

## Updates（更新策略）

| 选项 | 说明 |
|------|------|
| Default | 恢复微软默认设置 |
| Security | 仅安装安全更新，跳过功能更新 |
| Disable | 完全禁用自动更新 |

**Windows Pro 用户推荐配置**：

- 功能更新延迟 1 年，避免成为新版本小白鼠
- 安全更新延迟 4 天（周二发布 → 周六安装），给微软时间撤回问题补丁

> **注意**：完全禁用更新会带来安全风险。有问题更新时临时禁用，问题修复后记得重新开启。

---

## Win11 Creator（定制 ISO）

将官方 Windows 11 ISO 重新打包为干净版本：移除 AI 组件、预装应用和遥测，保留完整的软件兼容性，输出 ISO 约 2.5–3.5 GB（原版 5–6 GB）。

使用流程：

1. 挂载并验证官方 ISO，选择版本（默认 Pro）
2. 执行定制（耗时 10–30 分钟）
3. 导出为新 ISO 或直接写入 U 盘
4. 可选：清理临时工作目录（约 10–15 GB）

> **提示**：WinUtil 配置可以导出为 JSON 文件，方便在多台设备上重复应用相同设置。

---

## 相关链接

- [官网文档](https://winutil.christitus.com/)
- [GitHub 仓库](https://github.com/ChrisTitusTech/winutil)
- [Essential Tweaks 详细说明](https://winutil.christitus.com/dev/tweaks/essential-tweaks/)
- [Advanced Tweaks 详细说明](https://winutil.christitus.com/dev/tweaks/z--advanced-tweaks---caution/)
