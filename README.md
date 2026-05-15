# claude-skill

我的 Claude Code skill 集合，用于扩展 Claude Code 的能力。

## Skills

### [code-review](./code-review/)

对指定 Git 提交范围进行全面 Code Review，输出结构化 HTML 报告。

**触发方式：** `/code-review <FROM_COMMIT>`

**功能：**
- 分析提交列表，按功能模块分类
- 代码逻辑审查（逻辑错误、边界条件、安全漏洞等），风险分级 🔴🟡🟢
- 影响范围分析（模块、接口、数据库、配置、依赖、兼容性）
- 生成详细测试用例（P0/P1/P2 优先级）
- 输出美观的 HTML 报告到 `SDD/<分支名>/CodeReview_Report_<分支名>.html`

---

### [doc-to-md](./doc-to-md/)

将 `.docx` 或 `.pdf` 文件转换为结构清晰的 Markdown 文件。

**触发方式：** 提及"转换成md"、"docx转md"、"pdf转md"等关键词

**功能：**
- DOCX：通过样式名、字号、加粗自动推断标题层级，支持表格、行内格式、图片提取
- PDF：优先使用 `pymupdf` 通过字号/字体推断标题，fallback 到 `pypdf`
- 依赖首次运行自动安装（`python-docx`、`pypdf`）

**直接使用脚本：**
```bash
python doc-to-md/scripts/convert.py <input.docx|input.pdf> [output.md]
```

---

## 安装

将需要的 skill 目录复制到 Claude Code 的 skills 目录：

```bash
cp -r <skill-name> ~/.claude/skills/
```

Claude Code 会自动识别并加载。

## License

MIT
