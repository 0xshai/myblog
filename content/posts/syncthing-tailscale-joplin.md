---
title: "自建私有云笔记：Syncthing + Tailscale + Joplin 全平台同步"
date: 2026-06-22T10:00:00+08:00
description: "完全免费、无云商、端到端可控的笔记同步方案实战记录。"
tags: ["自托管", "Syncthing", "Tailscale", "Joplin", "笔记"]
ShowToc: true
TocOpen: false
---

> 一套**完全免费、无云商、端到端可控**的笔记同步方案：
> **Joplin（笔记） + Syncthing（P2P 同步） + Tailscale（异地组网）**

---

## 一、为什么要这样搭？

最初的需求很简单：

- 不想把笔记丢到商业网盘
- 要跨平台：Windows + Android
- 手机断网也要能看、有网就能同步
- 不想折腾公网 IP、DDNS、端口映射

最终方案选型：

| 组件          | 作用                                |
| ------------- | ----------------------------------- |
| **Joplin**    | Markdown 笔记 + 附件管理            |
| **Syncthing** | 开源 P2P 文件同步（替代 Dropbox）   |
| **Tailscale** | 零配置虚拟内网，解决 NAT 和异地直连 |

---

## 二、整体架构

```
Joplin (Windows)
   ↓↑ 文件系统同步
Syncthing (Windows)
   ↕  Tailscale 虚拟内网直连
Syncthing (Android)
   ↓↑ 文件系统同步
Joplin (Android)
```

- **Syncthing**：负责把 Joplin 的同步目录在设备间同步
- **Tailscale**：让两台设备永远"在一个内网"
- **Joplin**：只管读写笔记，不关心网络细节

---

## 三、安装与基础配置

### Windows：为什么推荐 SyncTrayzor？

Syncthing 本体（`syncthing.exe`）是纯后台命令行程序，没有系统托盘、没有开机自启、没有图形外壳。Windows 上直接跑会有这些麻烦：

- 每次要手动启动或注册 Windows 服务
- 同步状态不可见（需自己开浏览器）
- 锁屏/注销可能中断进程

**SyncTrayzor** 是 Windows 社区维护的官方推荐前端封装：

| 功能                    | SyncTrayzor ✅      | 裸 syncthing.exe  |
| ----------------------- | ------------------- | ----------------- |
| 内嵌 Syncthing 内核     | ✅ 自动捆绑         | ✅ 需自己下       |
| 系统托盘图标 + 同步状态 | ✅                  | ❌                |
| 开机自启（用户级）      | ✅ 勾选即开         | ⚠️ 需手动配服务  |
| 一键升级 Syncthing      | ✅                  | ⚠️ 手动替换      |

> SyncTrayzor 不是 fork，只是把 Syncthing 包了一层 Windows GUI/托盘壳，协议和同步逻辑完全一样，设备间 100% 互通。

下载地址：https://github.com/canton7/SyncTrayzor/releases

下 **`SyncTrayzorSetup-x64.exe`** → 安装 → 自动打开 `http://127.0.0.1:8384`

### Android：Syncthing-Fork

- 安装 **Syncthing-Fork**（[F-Droid](https://f-droid.org) / [GitHub](https://github.com/Catfriend1/syncthing-android/releases)）
- 原版已停止维护，Fork 版持续更新
- 打开 App，记录 **Device ID**（也可展示为二维码）

### 配对设备

在电脑 SyncTrayzor 的 Web UI 中：

1. 右下角 **Add Remote Device**
2. 粘贴手机 **Device ID**
3. **Save** → 手机端弹出"新设备请求" → **Accept**

> **如果手机没弹窗**（常见于国产 ROM 通知被拦截）：手机 App → Folders → 右上角 ➕ 添加文件夹，**Folder ID 必须与电脑端完全一致**（区分大小写，如 `JoplinSync`），选择本地路径后保存，向电脑发送接受信号。

---

## 四、配置 Joplin 同步目录

### 创建 Syncthing 同步文件夹

电脑端 Syncthing Web UI：

1. **Add Folder**
2. **Folder Label**：`JoplinSync`
3. **Folder Path**：`C:\Users\你的用户名\Syncthing\JoplinSync`
4. **Sharing** 标签页 → 勾选手机设备 → **Save**
5. 手机端接受共享，选择本地路径：
   ```
   /storage/emulated/0/Syncthing/JoplinSync
   ```

### Joplin 设置为「文件系统」同步

Windows / Android 通用步骤：

1. **设置 → 同步（Synchronization）**
2. **同步目标**：选 **File system（文件系统）**
3. **同步目录**（填绝对路径）：
   - Windows：`C:\Users\你的用户名\Syncthing\JoplinSync`
   - Android：`/storage/emulated/0/Syncthing/JoplinSync`
4. 点 **同步（Synchronize）**

首次同步会生成大量小文件，这是正常现象——Joplin 的笔记和资源都会写成独立文件。

### 开启删除同步

> **注意**：默认情况下，Joplin 的"文件系统同步"不会主动传播删除操作。电脑删了笔记 → Joplin 只在本机数据库标记删除 → Syncthing 没有收到删除指令 → 手机端文件还在 → 手机 Joplin 重新读入后笔记仍存在。

解决方法：

1. 同步设置页 → 展开 **显示高级设置**
2. 勾选 **「同步时从同步目标中删除已删除的笔记/资源」**
3. **每台设备都要单独开启**

---

## 五、解决"连上了但很慢"的问题（Tailscale）

### 问题现象

Syncthing 日志中出现：

```
Detected NAT type: Symmetric NAT
Joined relay
```

说明两边网络是严格 NAT（对称型），Syncthing 打洞失败，被迫走官方中继（relay），带宽有限，速度慢、延迟高。

### Tailscale 是什么？

**Tailscale = 零配置 WireGuard 虚拟内网**，把你的所有设备拉进同一个虚拟局域网：

- 每台设备装 Tailscale → 登录同一账号 → 自动获得固定虚拟 IP（如 `100.x.x.x`）
- 不管设备在家 WiFi、公司网还是手机 4G/5G，都像插在同一台路由器上
- 流量端到端加密（WireGuard 协议），控制服务器只分发密钥，看不到你的数据
- 免费版个人日常使用完全够用

### 安装 Tailscale

**Windows**

下载官方 MSI：https://tailscale.com/download/windows

> **不要用 Scoop 安装**：Scoop 版是纯命令行，无 GUI，需手动管理启动。MSI 版含系统服务 + 托盘图标 + 开机自启，安装即用。

安装 → 托盘图标 → **Login** → 浏览器登录

**Android**

Play Store / F-Droid 安装 **Tailscale**，登录**同一个账号**。

安装完成后两台设备各自获得虚拟 IP（如 `100.82.x.x` 和 `100.75.x.x`）。

### 让 Syncthing 走 Tailscale 直连

在电脑 Syncthing Web UI：

1. 点手机设备卡片 → **Edit**
2. 展开 **Show Advanced**
3. 找到 **Addresses** 字段（默认是 `dynamic`）
4. 改为手机 Tailscale IP，例如：
   ```
   tcp://100.75.23.18:22000
   ```
5. **Save** → 等设备变绿

手机端同理（可选）：编辑电脑设备 → Addresses 改为电脑 Tailscale IP。

**成功标志：**

- 设备状态变绿 **Connected**
- 鼠标悬停设备 → 连接显示 `tcp://100.x.x.x:22000`
- 不再是 `relay://...` → 直连成功，速度接近局域网

> 不想手动填 IP 也可以不改 Addresses，Tailscale 连上后 Syncthing 有时会自己发现直连。手动填是强制走 Tailscale、跳过中继，更稳。

---

## 六、Android 保活

国产 ROM 容易杀后台，需要手动配置：

**系统设置（各品牌路径略有差异）：**

1. 设置 → 应用管理 → Syncthing-Fork → 电池 → **不允许优化 / 无限制**
2. 允许开机自动启动 + 允许后台运行
3. 多任务界面长按 Syncthing → **锁定🔒**

**Syncthing App 内设置：**

打开 Syncthing-Fork → Settings → **Run Conditions**：

- ✅ Run on WiFi
- ✅ Run when charging（可选）

---

## 七、最终效果

- 笔记跨平台实时同步（Windows ↔ Android）
- 手机断网可离线阅读，有网自动同步
- 不依赖任何商业云服务
- 数据全程在自己设备之间流动
- 异地 WiFi / 4G / 公司网络均可直连
- 没有月费、没有容量上限

---

## 八、常见问题

| 问题                   | 解决方案                                                                      |
| ---------------------- | ----------------------------------------------------------------------------- |
| 电脑删笔记手机还在     | 开启 Joplin 同步高级设置中的"删除同步"选项                                    |
| 手机不弹共享请求       | 手动添加文件夹，Folder ID 必须与电脑完全一致                                  |
| 同步速度慢             | 安装 Tailscale，让 Syncthing 走虚拟内网直连                                   |
| 手机经常断连           | 关闭电池优化 + 允许自启 + 锁定后台                                            |
| iOS 能用吗？           | ❌ iOS Joplin 不支持 File System 同步，只能用 Joplin Cloud / WebDAV / Dropbox |
| 数据安全吗？           | Syncthing TLS 加密 + Tailscale WireGuard 加密 + Joplin 可选端到端加密        |
| 首次同步特别慢         | 正常，Joplin 会生成大量小文件，Syncthing 初次索引需要时间                     |
