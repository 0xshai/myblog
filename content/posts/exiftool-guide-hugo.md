---
title: "ExifTool 完整使用指南"
date: 2026-06-28
description: "读取、修改、批量重命名、按时间归档照片元数据的完整命令参考"
tags: ["工具", "照片", "命令行"]
---

ExifTool 是由 Phil Harvey 开发的开源命令行工具，能读取和写入照片、视频几乎所有格式的元数据（EXIF、GPS、IPTC、XMP 等）。最新版本为 **13.59**（2026年5月）。

## 安装

**Windows**

从 [exiftool.org](https://exiftool.org/) 下载 Windows 可执行文件 `.zip`，解压后将 `exiftool(-k).exe` 改名为 `exiftool.exe`，放到 `C:\tools\` 或任意加入了 PATH 的目录即可。

也可以用 Scoop 安装：

```powershell
scoop install exiftool
```

**验证安装**

```powershell
exiftool -ver
```

---

## 一、查看元数据

### 查看单张照片的全部信息

```powershell
exiftool photo.jpg
```

输出会列出几十个字段，包括相机型号、拍摄时间、快门速度、光圈、ISO、GPS 等。

### 只查看指定字段

```powershell
exiftool -DateTimeOriginal photo.jpg
exiftool -Make -Model -LensModel photo.jpg
exiftool -GPSLatitude -GPSLongitude photo.jpg
```

### 批量查看一个文件夹

```powershell
exiftool D:\Photos\
```

### 只显示文件名和拍摄时间（表格式输出）

```powershell
exiftool -T -FileName -DateTimeOriginal D:\Photos\
```

`-T` 参数让输出按制表符分隔，方便复制到表格。

### 导出为 CSV

```powershell
exiftool -csv -FileName -DateTimeOriginal -Make -Model D:\Photos\ > output.csv
```

### 递归扫描子文件夹

```powershell
exiftool -r D:\Photos\
```

> **提示**：查看 RAW 文件（`.ARW`、`.RW2`、`.CR3` 等）同样适用，ExifTool 支持几乎所有相机品牌的原始格式。

---

## 二、修改元数据

### 修改拍摄时间

```powershell
exiftool -DateTimeOriginal="2024:06:01 12:00:00" photo.jpg
```

时间格式固定为 `YYYY:MM:DD HH:MM:SS`。

### 批量修改一个文件夹内所有照片的时间（偏移）

比如照片时间全部偏差了 1 小时：

```powershell
exiftool "-DateTimeOriginal+=0:0:0 1:0:0" D:\Photos\
```

### 添加/修改 GPS 坐标

```powershell
exiftool -GPSLatitude=39.9042 -GPSLatitudeRef=N -GPSLongitude=116.4074 -GPSLongitudeRef=E photo.jpg
```

### 修改版权和作者信息

```powershell
exiftool -Copyright="© 2024 Shai" -Artist="Shai" photo.jpg
```

### 批量修改所有 JPG 的作者

```powershell
exiftool -ext jpg -Artist="Shai" D:\Photos\
```

> **注意**：ExifTool 默认会保留原始文件的备份（`photo.jpg_original`）。确认修改无误后可以用 `-overwrite_original` 跳过备份：
>
> ```powershell
> exiftool -overwrite_original -Artist="Shai" photo.jpg
> ```

---

## 三、清除元数据（去除隐私信息）

### 清除所有元数据

```powershell
exiftool -all= photo.jpg
```

### 只清除 GPS 信息（保留其他数据）

```powershell
exiftool -GPS:all= photo.jpg
```

### 批量清除一个文件夹的 GPS

```powershell
exiftool -r -GPS:all= D:\Photos\
```

> **提示**：发朋友圈或上传到社交平台前，可以用这个命令批量清除 GPS，防止位置泄露。

---

## 四、批量重命名

### 按拍摄时间重命名

将文件名改为 `20240601_120000.jpg` 格式：

```powershell
exiftool "-FileName<DateTimeOriginal" -d "%Y%m%d_%H%M%S.%%e" D:\Photos\
```

参数说明：
- `%Y` 年、`%m` 月、`%d` 日、`%H` 时、`%M` 分、`%S` 秒
- `%%e` 保留原始扩展名

### 加上相机型号前缀

```powershell
exiftool "-FileName<${Model}_${DateTimeOriginal}" -d "%Y%m%d_%H%M%S.%%e" D:\Photos\
```

### 预览重命名结果（不实际执行）

```powershell
exiftool "-FileName<DateTimeOriginal" -d "%Y%m%d_%H%M%S.%%e" -n D:\Photos\
```

> **提示**：加上 `-n` 参数可以预览操作结果而不真正执行，确认无误再去掉 `-n` 正式运行。

---

## 五、按时间自动归类到文件夹

这是整理散乱照片最实用的功能。

### 按年/月归类（推荐）

将照片整理到 `D:\Sorted\2024\06\` 这样的目录结构：

```powershell
exiftool "-Directory<DateTimeOriginal" -d "D:/Sorted/%Y/%m" D:\Photos\
```

### 按年归类（一级目录）

```powershell
exiftool "-Directory<DateTimeOriginal" -d "D:/Sorted/%Y" D:\Photos\
```

### 同时重命名 + 归类

```powershell
exiftool "-FileName<DateTimeOriginal" -d "%Y%m%d_%H%M%S.%%e" "-Directory<DateTimeOriginal" -d "D:/Sorted/%Y/%m" -r D:\Photos\
```

### 递归处理子文件夹

加上 `-r` 参数：

```powershell
exiftool -r "-Directory<DateTimeOriginal" -d "D:/Sorted/%Y/%m" D:\Photos\
```

> **注意**：没有 EXIF 时间的照片（截图、微信图片等）不会被移动。可以加上 `-if "$DateTimeOriginal"` 只处理有时间的文件：
>
> ```powershell
> exiftool -r -if "$DateTimeOriginal" "-Directory<DateTimeOriginal" -d "D:/Sorted/%Y/%m" D:\Photos\
> ```

---

## 六、复制元数据

### 从一张照片复制元数据到另一张

```powershell
exiftool -TagsFromFile source.jpg target.jpg
```

### 只复制时间信息

```powershell
exiftool -TagsFromFile source.jpg -DateTimeOriginal target.jpg
```

---

## 七、常用参数速查

| 参数 | 说明 |
|------|------|
| `-r` | 递归处理子文件夹 |
| `-ext jpg` | 只处理指定扩展名 |
| `-n` | 预览模式，不实际执行 |
| `-overwrite_original` | 不保留备份文件 |
| `-q` | 静默模式，减少输出 |
| `-csv` | 输出为 CSV 格式 |
| `-T` | 制表符分隔输出 |
| `-if "条件"` | 条件过滤 |
| `-all=` | 清除所有元数据 |

---

## 八、常见问题

**Q：照片没有 EXIF，时间是错的怎么办？**

先手动修正一张，确认格式正确，再批量偏移其他照片的时间。

**Q：处理后文件变大了？**

JPEG 写入元数据时 ExifTool 会重新整理文件结构，属于正常现象，图像质量不受影响。

**Q：如何撤销操作？**

ExifTool 默认保留 `_original` 备份文件。恢复方法：

```powershell
exiftool -restore_original photo.jpg
```

批量恢复：

```powershell
exiftool -restore_original D:\Photos\
```

---

## 延伸阅读

- [ExifTool 官网](https://exiftool.org/)
- [ExifTool 支持的文件格式列表](https://exiftool.org/#supported)
- [ExifTool Tag Names 完整字段列表](https://exiftool.org/TagNames/)
- [ExifToolGUI](https://github.com/FrankBijnen/ExifToolGui)（图形界面版本，不想用命令行可以试试）
