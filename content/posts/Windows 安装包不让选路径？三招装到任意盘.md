---
title: "Windows 安装包不让选路径？三招装到任意盘"
date: 2026-05-28
tags: ["Windows", "工具", "Scoop"]
---

有些 Windows 安装包，打开直接就装了，连个路径选择都没有，默认往 C 盘塞。
下面三种方法，从简单到彻底，总有一个适合你。

以 ABDownloadManager 为例，其他类似的安装包思路完全一样。

## 方法一：命令行 `/D` 参数

很多安装包底层用的是 NSIS 打包工具，支持用命令行参数指定安装路径。

在安装包所在文件夹，按住 `Shift` 右键空白处，选**在此处打开 PowerShell 窗口**（或终端窗口），然后执行：

```
.\ABDownloadManager_1.8.8_windows_x64.exe /D=D:\ABDownloadManager
```

注意事项：
- `/D=` 后面**不要加引号**，即使路径有空格也不加
- `/D=...` 必须放在命令的最后
- 路径必须是绝对路径

**如果安装包文件名本身有空格**，直接执行会报错：

```
.\Anytype: The term '.\Anytype' is not recognized...
```

这是因为 PowerShell 会把空格当作参数分隔符，把文件名截断了。解决方法是用引号包住文件名，并在前面加调用运算符 `&`：

```powershell
& ".\Anytype Setup 0.55.4.exe" /D=D:\anytype
```

- `&` 是 PowerShell 的调用运算符，专门用来执行路径中含空格的程序
- 引号只包文件名，`/D=...` 参数放在引号外面，保持原样

如果安装包正常弹出安装向导，安装完成后就会出现在 D 盘指定目录。

> 这个方法不是百分百有效——部分安装包会忽略 `/D` 参数，强制装到 C 盘。遇到这种情况，用下面的方法。

## 方法二：安装后迁移

先默认装完，再把程序整体搬到 D 盘。

**步骤：**

1. 按默认流程安装完成
2. 确保程序已完全退出（任务管理器确认没有相关进程）
3. 把安装目录整个文件夹**剪切**到 D 盘目标位置
4. 右键桌面快捷方式 → 属性 → 修改**目标**路径指向新位置

大部分非系统级程序直接搬就能正常运行，不需要重装。

**可选：更新注册表**

如果你在意"程序和功能"里的卸载路径显示正确，可以顺手改一下：

按 `Win + R` 输入 `regedit`，找到：

```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\
```

在里面找到对应程序的项，修改 `InstallLocation` 的值为新路径即可。

## 方法三：用 Scoop 安装（一劳永逸）

如果你经常装软件，Scoop 是更根本的解决办法。

Scoop 是 Windows 上的命令行包管理器，所有软件统一装在一个目录，这个目录可以放在任意盘。

**Scoop 适合装什么：**

| 类型 | 举例 |
|---|---|
| 开发工具 | git、node、python、go、ffmpeg、7zip |
| 命令行工具 | curl、wget、jq、fzf、ripgrep |
| 终端美化 | oh-my-posh、starship |
| GUI 软件 | ABDownloadManager、Obsidian、VSCode、Lazygit |
| 编程字体 | JetBrainsMono、CascadiaCode（需加 nerd-fonts bucket） |

不适合装：Office、Adobe 全家桶、微信这类大型商业或国产软件，Scoop 里基本没有收录。

**先把 Scoop 装到 D 盘：**

在 PowerShell 里执行（顺序不能乱）：

```powershell
$env:SCOOP='D:\Scoop'
[Environment]::SetEnvironmentVariable('SCOOP', $env:SCOOP, 'User')
irm get.scoop.sh | iex
```

**然后装 ABDownloadManager：**

```powershell
scoop bucket add extras
scoop install extras/ab-download-manager
```

装完就在 `D:\Scoop\apps\ab-download-manager\` 下，干净，好管理。

**Scoop 常用命令：**

```powershell
scoop install <软件名>     # 安装
scoop uninstall <软件名>   # 卸载
scoop update <软件名>      # 更新某个软件
scoop update *             # 更新所有软件
scoop search <关键词>      # 搜索
scoop list                 # 查看已安装
```

卸载彻底，不写注册表，不需要管理员权限，更新一条命令搞定。
如果你有多个软件想统一管理，值得配一下。

## 三种方法对比

| 方法 | 适合场景 | 成功率 |
|---|---|---|
| 命令行 `/D` 参数 | 快速搞定单个软件 | 约 70%，看安装包是否支持 |
| 安装后迁移 | 通用兜底方案 | 高，非系统级程序基本都行 |
| Scoop | 长期管理多个软件 | 100%，完全掌控路径 |
