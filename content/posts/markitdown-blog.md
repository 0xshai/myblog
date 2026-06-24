---
title: "MarkItDown：微软开源的万能文档转 Markdown 工具"
date: 2026-06-24
description: "将 PDF、Word、PPT、Excel、图片、音频批量转成 Markdown，专为 LLM / RAG 场景设计。附 Windows 实战踩坑记录。"
tags: ["工具", "Python", "AI", "Markdown"]
ShowToc: true
TocOpen: true
---

[MarkItDown](https://github.com/microsoft/markitdown) 是微软开源的 Python 工具，用于将 PDF、Word、PPT、Excel、图片、音频、HTML 等多种文件格式**转换为 Markdown 文本**，非常适合为 LLM / RAG 知识库预处理文档。

---

## Windows 快速上手（三步完成）

> 以下流程在 `D:\V` 目录下操作，可替换为你自己的项目目录。

**第一步：新建 venv**

```powershell
cd D:\V
python -m venv .venv
```

**第二步：激活并安装**

```powershell
.\.venv\Scripts\Activate.ps1
python -m pip install -U pip
python -m pip install "markitdown[all]"
```

激活成功后提示符前会出现 `(.venv)`。

**第三步：验证安装**

```powershell
python -m markitdown --version
```

**以后每次使用**，记住这一套：

```powershell
cd D:\V
.\.venv\Scripts\Activate.ps1
python -m markitdown "中文 文件名.xlsx" -o "中文 文件名.md"
```

> 💡 用 `python -m markitdown` 而不是直接 `markitdown`——不依赖 PATH，不依赖 venv 是否激活，Windows 上最稳妥的写法。路径含中文或空格时务必加引号。

---

## 安装说明

需要 **Python 3.10 及以上版本**。

**完整功能（推荐）**

```bash
pip install "markitdown[all]"
```

包含 OCR、音频转写、所有文档格式支持。

**基础版**

```bash
pip install markitdown
```

仅核心功能，体积最小。

**按需安装指定格式**

```bash
pip install "markitdown[pdf,docx,pptx,xlsx]"
```

仅安装需要的格式依赖，按需组合。

---

## 命令行（CLI）使用

### 基础转换

```bash
# 输出到终端
markitdown 报告.docx

# 保存为 .md 文件
markitdown 报告.docx -o 报告.md

# 使用重定向
markitdown 报告.pdf > 报告.md
```

### 从 URL 或管道输入

```bash
# 从 URL 转换
markitdown https://example.com -o page.md

# 管道输入
cat data.xlsx | markitdown > data.md

# 从网络文件转换
curl -sL https://example.com/doc.pdf | markitdown > out.md
```

### 常见格式示例

| 格式 | 命令 |
|------|------|
| Word → Markdown | `markitdown 文档.docx -o 文档.md` |
| PPT → Markdown | `markitdown 幻灯片.pptx -o slides.md` |
| Excel → Markdown | `markitdown 表格.xlsx -o 表格.md` |
| 图片 OCR | `markitdown 截图.png -o 截图.md`（需安装完整版） |
| 音频转写 | `markitdown 录音.mp3 -o 录音.md` |
| YouTube 字幕 | `markitdown https://youtube.com/watch?v=xxx -o transcript.md` |

### 批量转换

Windows CMD：

```batch
for %i in (*.pdf) do markitdown "%i" -o "%~ni.md"
```

Linux / macOS：

```bash
for f in *.docx; do markitdown "$f" -o "${f%.docx}.md"; done
```

---

## Python API 调用

适合在代码中集成，例如构建 RAG 流程：

### 基础用法

```python
from markitdown import MarkItDown

md = MarkItDown(enable_plugins=False)
result = md.convert("报告.pdf")
print(result.text_content)
```

### 配合 LLM 识别图片内容

```python
from markitdown import MarkItDown
from openai import OpenAI

client = OpenAI()
md = MarkItDown(llm_client=client, llm_model="gpt-4o")
result = md.convert("图表.jpg")
print(result.text_content)  # 包含 AI 生成的图片描述
```

### 使用 Azure Document Intelligence

```python
md = MarkItDown(docintel_endpoint="https://xxx.cognitiveservices.azure.com/")
result = md.convert("test.pdf")
```

---

## 支持的文件格式

| 类别 | 格式 |
|------|------|
| 办公文档 | `.docx`、`.pptx`、`.xlsx`、`.xls`、`.pdf` |
| 网页 / 数据 | HTML、CSV、JSON、XML、EPUB |
| 媒体 | 图片（JPG / PNG，OCR）、音频（MP3 / WAV 转写） |
| 其他 | ZIP（自动解压遍历）、Outlook `.msg`、纯文本 |

---

## 图形界面（GUI）可选

MarkItDown 本身没有官方 GUI，但有以下第三方方案：

- **[markitdown-gui](https://github.com/imadreamerboy/markitdown-gui)**：基于 PySide6 + Fluent 风格界面，支持拖拽批量转换、Markdown 预览、OCR，跨平台，提供打包好的可执行文件，无需安装 Python。
- **[MarkItDown_GUI](https://github.com/RafalekS/MarkItDown_GUI)**：基于 PyQt6，内置 47 套配色主题，支持多转换引擎回退（MarkItDown / Pandoc / pymupdf4llm），需自行安装依赖运行。
- **[markitdown-api](https://github.com/dezoito/markitdown-api)**：基于 FastAPI 的轻量 REST API 服务，提供 Docker 镜像，浏览器或 curl 即可调用，适合服务器部署场景。

> ⚠️ 这些均为社区第三方项目，非微软官方出品。

---

## 常见问题

### 转换后中文乱码怎么办？

确保源文件本身编码正确。对于旧版 Office 文档，可尝试在转换时指定编码参数。

### 扫描版 PDF 无法提取文字？

安装完整版（`markitdown[all]`）后自动支持 OCR，或配置 Azure Document Intelligence 获得更高精度。

### 转换大文件时速度慢？

对于超大 PDF，可以尝试先拆分再逐个转换。使用 Python API 时，可以配合异步处理提高吞吐量。

### 已安装但提示找不到命令？（Windows）

原因通常是 Python 环境与 PATH 不匹配，排查步骤：

```powershell
where.exe python
python -m pip show markitdown
```

如果 `pip show` 找不到，说明当前 Python 没有安装 MarkItDown。直接用 `python -m markitdown` 可绕过所有 PATH 问题。

### 文件名含中文或空格时报错？（Windows）

在文件所在目录执行命令，并用引号包裹路径：

```powershell
cd D:\V
python -m markitdown "IPD 敦煌主题-20260528.xlsx" -o "IPD 敦煌主题-20260528.md"
```

出现 `puremagic.main.PureError: Not a regular file` 不是文件损坏，是路径错误或不在当前目录。
