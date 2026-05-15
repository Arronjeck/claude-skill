# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个 Claude Code skill 集合仓库，每个 skill 是一个独立目录，包含 `SKILL.md`（skill 定义文件）和可选的辅助脚本。

## Skill 结构

每个 skill 目录的标准结构：

```
<skill-name>/
  SKILL.md          # skill 定义，包含 YAML frontmatter 和执行指令
  scripts/          # 可选：辅助脚本（Python 等）
```

`SKILL.md` 的 frontmatter 格式：

```yaml
---
name: <skill-name>
description: <触发描述，包含触发关键词>
---
```

## 现有 Skills

- **code-review**：对 Git 提交范围进行 Code Review，输出结构化 HTML 报告（保存到 `SDD/<分支名>/CodeReview_Report_<分支名>.html`）
- **doc-to-md**：将 `.docx` / `.pdf` 转换为 Markdown，附带 `scripts/convert.py` 脚本

## doc-to-md 脚本

```bash
python doc-to-md/scripts/convert.py <input.docx|input.pdf> [output.md]
```

依赖自动安装（`python-docx`、`pypdf`）；`pymupdf` 需手动安装以获得更好的 PDF 解析质量。

## 安装 Skill 到 Claude Code

将 skill 目录复制到 `~/.claude/skills/<skill-name>/`，Claude Code 会自动识别。
