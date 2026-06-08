---
title: "只改一个文件，README 自动同步——用 GitHub Actions 告别手动维护"
date: 2026-06-08
tags: ["GitHub", "自动化", "Python"]
---

维护开源项目或工具清单时，经常会遇到一个问题：数据文件改了，README 忘记同步。时间一长，两边对不上，要么手动逐条复制，要么干脆摆烂。

这篇文章用一个真实场景——书签工具清单——来演示怎么用 GitHub Actions + Python 脚本，实现「只改 `bookmarks.yaml`，README 自动更新」的工作流。

## 思路

整个方案分两部分：

1. **Python 脚本**：读取 `bookmarks.yaml`，按固定格式生成 `README.md`
2. **GitHub Actions**：监听 `bookmarks.yaml` 的变动，自动触发脚本并提交结果

改动 `bookmarks.yaml` 之后，不管是本地 push 还是直接在 GitHub 网页上编辑，都会自动触发，不需要手动运行任何命令。

## 数据文件结构

`bookmarks.yaml` 的结构很简单，每个分类包含图标、名称和工具列表：

```yaml
- category: 编程工具
  icon: 🛠️
  items:
    - name: Zed
      url: https://zed.dev/
      desc: 轻量极速的现代代码编辑器
      opensource: true
    - name: WezTerm
      url: https://wezfurlong.org/wezterm
      desc: 高性能跨平台终端模拟器
      opensource: true
```

`opensource: true` 是可选字段，没有这个字段默认视为非开源。

## Python 脚本

新建 `gen_readme.py`，放在仓库根目录：

```python
#!/usr/bin/env python3
import yaml

HEADER = """# 🛠️ tools

> 精选隐私友好、开源优先的实用工具清单。
> A curated list of privacy-friendly, open-source tools I actually use.

🔓 = 开源 / Open Source

---

"""

FOOTER = """
---

欢迎提 Issue 或 PR 推荐新工具。
"""

def generate_readme(yaml_path="bookmarks.yaml", output_path="README.md"):
    with open(yaml_path, "r", encoding="utf-8") as f:
        data = yaml.safe_load(f)

    lines = [HEADER]

    for category in data:
        icon = category.get("icon", "")
        name = category.get("category", "")
        items = category.get("items", [])

        lines.append(f"## {icon} {name}\n")
        lines.append("| 工具 | 描述 | 开源 |")
        lines.append("|------|------|------|")

        for item in items:
            tool_name = item.get("name", "")
            url = item.get("url", "")
            desc = item.get("desc", "")
            opensource = "🔓" if item.get("opensource") else ""
            lines.append(f"| [{tool_name}]({url}) | {desc} | {opensource} |")

        lines.append("")

    lines.append(FOOTER.strip())

    with open(output_path, "w", encoding="utf-8") as f:
        f.write("\n".join(lines))

    print(f"✅ README.md 已生成，共 {len(data)} 个分类")

if __name__ == "__main__":
    generate_readme()
```

几个关键点：

- `yaml.safe_load()` 而不是 `yaml.load()`——safe 版本不会执行 YAML 中的任意 Python 对象，更安全
- `category.get("icon", "")` 用 `get` 加默认值，字段缺失时不会报错
- `"🔓" if item.get("opensource") else ""`——`get` 找不到字段返回 `None`，条件判断直接当 `False` 处理，所以不需要额外判断

## GitHub Actions 配置

在仓库里新建 `.github/workflows/update-readme.yml`。

**GitHub 网页上怎么新建带目录的文件：**

GitHub 不支持直接新建空文件夹，但可以在新建文件时顺带创建目录。在仓库主页点 **Add file → Create new file**，文件名输入框里直接输入完整路径：

```
.github/workflows/update-readme.yml
```

输入 `/` 的时候，GitHub 会自动把前面的部分折叠成目录层级，最终效果就是在 `.github/workflows/` 下创建了 `update-readme.yml`。粘贴内容后点右上角 **Commit changes** 保存。

内容如下：

```yaml
name: Update README

on:
  push:
    paths:
      - bookmarks.yaml

jobs:
  generate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.x"

      - name: Install dependencies
        run: pip install pyyaml

      - name: Generate README
        run: python gen_readme.py

      - name: Commit and push
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add README.md
          git diff --staged --quiet || git commit -m "docs: auto-update README from bookmarks.yaml"
          git push
```

逐段说明：

**触发条件**
```yaml
on:
  push:
    paths:
      - bookmarks.yaml
```
`paths` 过滤器让 Actions 只在 `bookmarks.yaml` 有变动时才触发，改其他文件不会白跑一次。

**运行环境**
```yaml
runs-on: ubuntu-latest
```
在 GitHub 提供的 Ubuntu 虚拟机上运行，免费额度对个人项目完全够用。

**checkout**
```yaml
- uses: actions/checkout@v4
```
把仓库代码拉到虚拟机里，后续步骤才能读取文件。`@v4` 是版本号，锁定版本避免上游更新导致行为变化。

**提交逻辑**
```bash
git diff --staged --quiet || git commit -m "..."
```
这一行是关键：`git diff --staged --quiet` 检查暂存区有没有变化，没变化退出码为 0（静默成功）；`||` 表示前面命令失败（有变化）才执行后面的 commit。这样避免了「README 没变但强行提交空 commit」的情况。

## 仓库结构

最终仓库文件结构：

```
tools/
├── bookmarks.yaml          # 只需要维护这一个
├── gen_readme.py           # 生成脚本
├── README.md               # 自动生成，不要手动改
└── .github/
    └── workflows/
        └── update-readme.yml
```

`README.md` 不再手动维护，所有改动都通过 `bookmarks.yaml` 进行。

## 必要设置：开启写入权限

配置完之后第一次触发，可能会遇到这个报错：

```
remote: Permission to xxx/tools.git denied to github-actions[bot].
fatal: unable to access '...': The requested URL returned error: 403
```

原因是 GitHub Actions 默认只有只读权限，bot 无法往仓库写入。去仓库设置里开一下：

**Settings → Actions → General → Workflow permissions → Read and write permissions**

保存后重新提交一次 `bookmarks.yaml` 触发即可。

## 效果

推送 `bookmarks.yaml` 后，去仓库的 Actions 标签页可以看到工作流运行状态。成功后会多出一条 bot 提交的 commit，`README.md` 已经同步更新。

这套方案的核心思路——**单一数据源，自动派生其他格式**——可以推广到很多类似场景：用 JSON 维护数据、自动生成文档站；用 CSV 维护变更日志、自动更新 CHANGELOG 等等。
