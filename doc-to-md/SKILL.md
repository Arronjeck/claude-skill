---
name: doc-to-md
description: 将 .docx 或 .pdf 文件转换为可读的 Markdown 文件。自动处理标题层级（通过样式名、字号、加粗推断）、表格、行内格式，以及中文编码问题。Use when user wants to convert a Word or PDF document to Markdown, mentions "转换成md"、"转成md"、"docx转md"、"pdf转md"、"doc-to-md" or asks to make a document readable.
---

# doc-to-md

将 `.docx` 或 `.pdf` 文件转换为结构清晰的 Markdown 文件。

## 快速使用

```bash
python "C:/Users/chenhj10/.claude/skills/doc-to-md/scripts/convert.py" "<input.docx|input.pdf>" ["<output.md>"]
```

不指定输出路径时，自动在同目录生成同名 `.md` 文件。

## 转换规则

### DOCX

| 原文档元素 | 转换结果 |
|---|---|
| Heading 1/2/3/4 样式 | `#` / `##` / `###` / `####` |
| 全段加粗 + 字号 > 正文 1.2x | `#` 标题 |
| 全段加粗 + 字号 > 正文 1.05x | `##` 标题 |
| 全段加粗（无字号信息） | `###` 标题 |
| 数字编号段落（`2. xxx`） | `##` 标题 |
| 数字编号段落（`2.3xxx`） | `###` 标题 |
| 多列表格 | Markdown 表格 |
| 单列单行表格 | 代码块 ` ``` ` |
| 行内加粗/斜体 run | `**text**` / `*text*` |

### PDF

优先使用 `pymupdf`（需安装）通过字号和字体推断标题层级；
fallback 到 `pypdf`（已内置），仅能识别数字编号标题，表格和代码块质量下降。

| 原文档元素 | 转换结果（pymupdf） |
|---|---|
| 数字编号行（`2. xxx`） | `##` 标题 |
| 数字编号行（`2.3 xxx`） | `###` 标题 |
| 字号 ≥ 正文 1.8x | `#` 标题（文档大标题） |
| SourceCodePro/Courier/Mono 字体 | 代码块 ` ``` ` |
| 字号 = 正文字号的 block | Markdown 表格行 |

## 依赖

- `python-docx`：DOCX 解析（首次运行自动安装）
- `pypdf`：PDF fallback（首次运行自动安装）
- `pymupdf`（可选）：PDF 高质量解析，支持字号/字体推断；需手动安装：`pip install pymupdf`

## 脚本能力边界

脚本负责**确定性的结构提取**，Claude 负责**语义判断和润色**。

### 脚本保证的

- 按字号/字体识别标题层级
- 按 `SourceCodePro`/`Courier`/`Mono` 字体识别代码块
- 按 block 字号和位置重建 Markdown 表格
- 跨行单元格（x 偏右续行）合并到上一行最后一列
- 跨页连续表格合并为一张表
- DOCX 图片提取到 `_assets/` 并生成引用

### 脚本无法保证的（交给 Claude 后处理）

- 单元格内枚举值（如 `list=...`、`pause=...` 各占一行）是否应拆为列表
- 表格最后一行混入了页脚/链接文字
- 多列 PDF 布局（左右分栏）导致的行顺序错乱
- 图片缺少语义描述（alt text 为文件名）
- 章节标题识别错误（文档不使用数字编号或标准字号）

## PDF 已知问题与处理方式（pymupdf）

| 问题 | 原因 | 处理方式 |
|---|---|---|
| 代码块被误判为标题 | 代码字体字号与标题相近 | 检测 SourceCodePro/Courier/Mono 字体，合并连续 block 为代码块 |
| 单元格内容跨行（续行 block） | PDF 长文本溢出到下一行独立成 block | 检测 block x0 偏右 >50pt，视为续行，追加到上一行最后一列 |
| 跨页表格被拆成两张 | 按页处理时页边界截断表格 | 所有页 block 拍平后统一处理，连续表格 block 自动合并 |
| `参数：`、`返回值：` 被误判为标题 | 字号与章节标题相同 | 只有数字编号行（`2.x`）或字号 ≥ 1.8x 才识别为标题 |

## 注意事项

- 图片提取到 `<文档名>_assets/`，重名时自动追加 `(1)`、`(2)` 后缀
- PDF 图片暂不支持提取
- 转换后建议用 Claude 检查：表格内枚举值、页脚混入、图片 alt text
