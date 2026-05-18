# claude-skill

我的 Claude Code skill 集合，用于扩展 Claude Code 的能力。

## 安装

把需要的 skill 目录复制到 Claude Code 的 skills 目录，Claude Code 会自动加载：

```bash
cp -r <skill-name> ~/.claude/skills/
```

---

## Skills

### [code-review](./code-review/)

对指定 Git 提交范围进行 Code Review，生成 HTML 报告。

**怎么用：** 在对话里输入 `/code-review <起始commit>`

**报告内容：**
- 代码逻辑审查，风险分级 🔴🟡🟢
- 影响范围分析（模块、接口、数据库、配置等）
- 测试用例建议（P0/P1/P2 优先级）

报告保存到 `SDD/<分支名>/CodeReview_Report_<分支名>.html`。

---

### [doc-to-md](./doc-to-md/)

把 `.docx` 或 `.pdf` 文件转成 Markdown，自动识别标题层级、表格、代码块、图片。

**怎么用：** 直接告诉 Claude "帮我把这个文件转成 md"，或者自己运行脚本：

```bash
python doc-to-md/scripts/convert.py 你的文件.docx
python doc-to-md/scripts/convert.py 你的文件.pdf
```

不指定输出路径时，自动在同目录生成同名 `.md` 文件。

**转换效果：**

| 原文档 | 识别结果 |
|---|---|
| Heading 样式 / 加粗大字 | `#` `##` `###` 标题 |
| 数字编号段落（`2.` / `2.3`） | `##` / `###` 标题 |
| 表格 | Markdown 表格 |
| 代码字体（Courier/Mono 等） | 代码块 |
| 图片 | 提取到 `文件名_assets/`，生成引用 |

**依赖：**
- `python-docx`、`pypdf`：首次运行自动安装
- `pymupdf`（可选，推荐）：PDF 解析质量更好，需手动安装：`pip install pymupdf`

> PDF 不安装 `pymupdf` 也能用，但表格和代码块识别质量会下降。

---

## License

MIT
