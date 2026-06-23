---
title: "MarkItDown：微软开源的万能文档转 Markdown 工具"
date: 2025-06-23
description: "将 PDF、Word、PPT、Excel、图片、音频批量转成 Markdown，专为 LLM / RAG 场景设计。"
tags: ["工具", "Python", "AI", "Markdown"]
ShowToc: true
TocOpen: true
---

[MarkItDown](https://github.com/microsoft/markitdown) 是微软开源的 Python 工具，用于将 PDF、Word、PPT、Excel、图片、音频、HTML 等多种文件格式**转换为 Markdown 文本**，非常适合为 LLM / RAG 知识库预处理文档。

---

## 前置条件

- **Python 3.10 及以上版本**
- 建议使用虚拟环境避免依赖冲突：

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate
```

---

## 安装

**完整功能（推荐）**

```bash
pip install 'markitdown[all]'
```

包含 OCR、音频转写、所有文档格式支持。

**基础版**

```bash
pip install markitdown
```

仅核心功能，体积最小。

**按需安装指定格式**

```bash
pip install 'markitdown[pdf,docx,pptx,xlsx]'
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

需要启用 OCR 功能，安装完整版（`markitdown[all]`），或配置 Tesseract OCR 引擎。

### 转换大文件时速度慢？

对于超大 PDF，可以尝试先拆分再逐个转换。使用 Python API 时，可以配合异步处理提高吞吐量。
