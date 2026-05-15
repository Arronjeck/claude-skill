---
name: code-review
description: 对指定 Git 提交范围进行全面 Code Review，输出结构化 HTML 报告（样式参考 CodeReview_Report_v9.0.28.html）
---

## 用户输入

```text
$ARGUMENTS
```

## 输入参数说明

用户必须提供**起始提交 hash**（即 `FROM_COMMIT`）。结束提交默认为 `HEAD`。

- 若 `$ARGUMENTS` 为空，**立即停止**并提示用户：
  ```
  请提供起始提交 hash，例如：/code-review ac86794208246d98281b0b43ff5e72050eefb307
  ```
- 若 `$ARGUMENTS` 非空，将其作为 `FROM_COMMIT`。

---

## 执行步骤（严格按顺序）

### 步骤 1：获取代码变更

运行以下命令，收集原始数据：

```bash
git log --oneline $FROM_COMMIT..HEAD
git diff $FROM_COMMIT..HEAD --stat
git diff $FROM_COMMIT..HEAD
```

同时获取当前分支名和今天日期（用于报告头部）：

```bash
git rev-parse --abbrev-ref HEAD
```

### 步骤 2：功能改动分析

- 列出所有涉及的提交（commit hash + message）
- 按功能模块分类改动（如：安装器 / 网络 / UI / 配置 / 公共库等）
- 识别改动类型：🆕 新增功能 / 🐛 缺陷修复 / ⚡ 性能优化 / ♻️ 重构 / ⚙️ 配置变更

### 步骤 3：代码逻辑审查

对每个关键文件检查：

- 明显的逻辑错误（空指针、越界、死锁等）
- 边界条件处理是否完善
- 异常处理是否完备
- 并发安全问题
- 资源泄漏风险
- 硬编码 / 魔法数字问题
- 安全漏洞（注入、越权等）

风险分级：🔴 高风险 / 🟡 中风险 / 🟢 低风险或正向优化

### 步骤 4：影响范围分析

| 维度 | 内容 |
|------|------|
| 模块影响 | 受影响的业务模块 |
| 接口影响 | 变更 / 新增的接口 |
| 数据库影响 | Schema 变更、索引、数据迁移 |
| 配置影响 | 配置文件变更、环境变量 |
| 依赖影响 | 第三方库变更、服务依赖 |
| 兼容性 | 是否向后兼容、破坏性变更 |

### 步骤 5：测试范围与用例

- 测试范围矩阵（单元测试 / 集成测试 / 回归测试）
- 需补充单测的文件列表
- 详细测试用例（每个用例含：ID、目标、前置条件、步骤、预期结果、优先级 P0/P1/P2）

### 步骤 6：自检

- [ ] 是否覆盖了所有修改文件？
- [ ] 风险评级是否合理？
- [ ] 测试用例是否可执行？
- [ ] 是否有遗漏的边界场景？

---

## 输出要求

完成分析后，将报告输出为 **HTML 文件**，保存到：

```
SDD/<当前分支名>/CodeReview_Report_<当前分支名>.html
```

例如当前分支为 `v9.0.29`，则保存为 `SDD/v9.0.29/CodeReview_Report_v9.0.29.html`。

若目录不存在，先创建目录。

### HTML 报告结构（严格参照 SDD/v9.0.28/CodeReview_Report_v9.0.28.html 的样式和布局）

HTML 报告必须包含以下完整 CSS 样式（直接复用参考文件中的 `<style>` 块，仅替换内容），以及以下各节：

**Header（顶部）**
- 标题：`📋 Code Review Report — <分支名>`
- meta 信息：审查范围（`FROM_COMMIT 短 hash → HEAD`）、分支、审查日期（今天）、技术栈（C++ / Win32 / MSVC）、审查深度

**统计卡片行**（`.stats-row`）
- 变更文件总数、有效提交数、🔴 高风险问题数、🟡 中风险问题数、🟢 低风险/正向数、测试用例数

**Section 1：提交列表（按功能模块分类）**
- 每个子模块一个 `sub-header` + 表格（Commit hash + 说明）

**Section 2：功能模块分类**
- `.module-grid` 卡片，每张卡片含模块名和涉及文件

**Section 3：代码逻辑审查**
- 3.1 高风险问题（`.risk-card.red`）：每项含文件路径、问题描述、diff 代码片段、修复建议
- 3.2 中等风险问题（`.risk-card.yellow`）：同上
- 3.3 低风险/正向优化（表格形式）

**Section 4：影响范围报告**
- `.impact-grid` 六格布局：模块影响、接口影响、数据库影响、配置影响、依赖影响、兼容性

**Section 5：测试范围与用例**
- 5.1 测试范围矩阵（表格）
- 5.2 需补充单测的文件列表
- 5.3 详细测试用例（`.tc-grid` 卡片，按 P0/P1/P2 着色）

**Section 6：自检清单**
- `.selfcheck` 列表，每项含图标、标题、说明

**Section 7：核心风险汇总**
- `.checklist` 列表，按 🔴/🟡/🟢 着色，供快速核对

**Footer**
- `Code Review Report · <分支名> · LeAppStore · <今天日期>`

### CSS 样式要求

直接使用以下完整 CSS（与参考文件完全一致）：

```css
:root {
  --red: #e53935; --red-bg: #fff5f5; --red-border: #ffcdd2;
  --yellow: #f9a825; --yellow-bg: #fffde7; --yellow-border: #fff176;
  --green: #43a047; --green-bg: #f1f8e9; --green-border: #c5e1a5;
  --blue: #1976d2; --blue-bg: #e3f2fd; --blue-border: #90caf9;
  --gray: #546e7a; --gray-bg: #f5f7fa; --gray-border: #cfd8dc;
  --card-shadow: 0 2px 8px rgba(0,0,0,0.10);
  --radius: 8px;
}
* { box-sizing: border-box; margin: 0; padding: 0; }
body { font-family: 'Segoe UI', 'Microsoft YaHei', sans-serif; background: #f0f2f5; color: #263238; font-size: 14px; line-height: 1.7; }
.header { background: linear-gradient(135deg, #1565c0 0%, #0d47a1 60%, #283593 100%); color: #fff; padding: 36px 48px 28px; }
.header h1 { font-size: 28px; font-weight: 700; letter-spacing: 1px; margin-bottom: 10px; }
.header .meta { display: flex; flex-wrap: wrap; gap: 24px; margin-top: 14px; }
.header .meta-item { display: flex; align-items: center; gap: 6px; font-size: 13px; opacity: .88; }
.header .meta-item span.label { opacity: .7; }
.header .badge { background: rgba(255,255,255,.18); border: 1px solid rgba(255,255,255,.3); border-radius: 20px; padding: 2px 12px; font-size: 12px; }
.stats-row { display: flex; gap: 16px; padding: 24px 48px 8px; flex-wrap: wrap; }
.stat-card { flex: 1; min-width: 130px; background: #fff; border-radius: var(--radius); box-shadow: var(--card-shadow); padding: 18px 20px; text-align: center; border-top: 4px solid; }
.stat-card.red  { border-color: var(--red); }
.stat-card.yellow { border-color: var(--yellow); }
.stat-card.green { border-color: var(--green); }
.stat-card.blue { border-color: var(--blue); }
.stat-card.gray { border-color: var(--gray); }
.stat-card .num { font-size: 32px; font-weight: 700; line-height: 1.2; }
.stat-card.red .num { color: var(--red); }
.stat-card.yellow .num { color: var(--yellow); }
.stat-card.green .num { color: var(--green); }
.stat-card.blue .num { color: var(--blue); }
.stat-card.gray .num { color: var(--gray); }
.stat-card .label { font-size: 12px; color: #78909c; margin-top: 4px; }
.container { max-width: 1200px; margin: 0 auto; padding: 8px 48px 48px; }
.section { background: #fff; border-radius: var(--radius); box-shadow: var(--card-shadow); margin-top: 24px; overflow: hidden; }
.section-header { display: flex; align-items: center; gap: 10px; padding: 16px 24px; border-bottom: 1px solid #eceff1; background: var(--gray-bg); }
.section-header h2 { font-size: 16px; font-weight: 600; color: #1a237e; }
.section-header .sec-icon { font-size: 18px; }
.section-body { padding: 20px 24px; }
table { width: 100%; border-collapse: collapse; font-size: 13px; }
th { background: #e8eaf6; color: #283593; font-weight: 600; padding: 9px 12px; text-align: left; border-bottom: 2px solid #c5cae9; }
td { padding: 8px 12px; border-bottom: 1px solid #f0f0f0; vertical-align: top; }
tr:hover td { background: #f9fbff; }
.commit-hash { font-family: 'Consolas', monospace; background: #eceff1; border-radius: 4px; padding: 1px 6px; font-size: 12px; color: #37474f; white-space: nowrap; }
.tag { display: inline-block; border-radius: 12px; padding: 2px 10px; font-size: 11px; font-weight: 600; margin: 1px; }
.tag-new  { background: #e8f5e9; color: #2e7d32; border: 1px solid #a5d6a7; }
.tag-fix  { background: #fff3e0; color: #e65100; border: 1px solid #ffcc80; }
.tag-refactor { background: #e3f2fd; color: #1565c0; border: 1px solid #90caf9; }
.tag-perf { background: #fce4ec; color: #880e4f; border: 1px solid #f48fb1; }
.risk-list { display: flex; flex-direction: column; gap: 16px; }
.risk-card { border-radius: var(--radius); border-left: 5px solid; padding: 16px 18px; }
.risk-card.red    { background: var(--red-bg); border-color: var(--red); }
.risk-card.yellow { background: var(--yellow-bg); border-color: var(--yellow); }
.risk-card.green  { background: var(--green-bg); border-color: var(--green); }
.risk-card .risk-title { font-weight: 700; font-size: 14px; display: flex; align-items: center; gap: 8px; margin-bottom: 6px; }
.risk-card .risk-file  { font-family: 'Consolas', monospace; font-size: 12px; color: #546e7a; background: rgba(0,0,0,.06); border-radius: 4px; padding: 1px 7px; }
.risk-card .risk-desc  { font-size: 13px; color: #37474f; margin: 6px 0; }
.risk-card .risk-fix   { font-size: 12px; margin-top: 8px; padding: 8px 12px; background: rgba(255,255,255,.7); border-radius: 6px; }
.risk-card .risk-fix::before { content: "💡 修复建议："; font-weight: 600; }
pre.code { background: #263238; color: #cfd8dc; border-radius: 6px; padding: 12px 16px; font-family: 'Consolas', monospace; font-size: 12px; overflow-x: auto; margin: 8px 0; line-height: 1.6; }
pre.code .add { color: #80cbc4; }
pre.code .del { color: #ef9a9a; }
pre.code .comment { color: #78909c; font-style: italic; }
.tc-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(480px, 1fr)); gap: 16px; }
.tc-card { border: 1px solid #e0e0e0; border-radius: var(--radius); overflow: hidden; }
.tc-card .tc-head { display: flex; justify-content: space-between; align-items: center; padding: 10px 16px; font-weight: 600; font-size: 13px; }
.tc-card.p0 .tc-head { background: #fbe9e7; color: #bf360c; border-bottom: 2px solid #ff8a65; }
.tc-card.p1 .tc-head { background: #fff8e1; color: #e65100; border-bottom: 2px solid #ffd54f; }
.tc-card.p2 .tc-head { background: #e8f5e9; color: #1b5e20; border-bottom: 2px solid #81c784; }
.tc-card .tc-body { padding: 12px 16px; font-size: 13px; }
.tc-card .tc-body dt { font-weight: 600; color: #546e7a; font-size: 11px; text-transform: uppercase; letter-spacing: .5px; margin-top: 8px; }
.tc-card .tc-body dd { margin-left: 0; color: #37474f; }
.tc-card .tc-body ol { padding-left: 18px; color: #37474f; }
.tc-priority { font-size: 11px; font-weight: 700; border-radius: 10px; padding: 2px 10px; }
.p0 .tc-priority { background: #c62828; color: #fff; }
.p1 .tc-priority { background: #f57c00; color: #fff; }
.p2 .tc-priority { background: #388e3c; color: #fff; }
.impact-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; }
.impact-item { background: var(--gray-bg); border-radius: 6px; padding: 12px 16px; border-left: 3px solid #90a4ae; }
.impact-item h4 { font-size: 12px; font-weight: 700; color: #455a64; text-transform: uppercase; letter-spacing: .5px; margin-bottom: 6px; }
.impact-item p { font-size: 13px; color: #37474f; }
.checklist { list-style: none; display: flex; flex-direction: column; gap: 6px; }
.checklist li { display: flex; align-items: baseline; gap: 10px; padding: 7px 12px; border-radius: 6px; font-size: 13px; }
.checklist li.red    { background: var(--red-bg); }
.checklist li.yellow { background: var(--yellow-bg); }
.checklist li.green  { background: var(--green-bg); }
.checklist li .cl-id { font-weight: 700; font-family: 'Consolas', monospace; min-width: 32px; }
.checklist li .cl-desc { flex: 1; }
.selfcheck { display: flex; flex-direction: column; gap: 8px; }
.selfcheck .sc-item { display: flex; align-items: flex-start; gap: 12px; padding: 10px 14px; background: var(--gray-bg); border-radius: 6px; font-size: 13px; }
.selfcheck .sc-item .sc-icon { font-size: 16px; flex-shrink: 0; }
.selfcheck .sc-item .sc-text strong { display: block; margin-bottom: 2px; color: #263238; }
.selfcheck .sc-item .sc-text span { color: #546e7a; }
.sub-header { font-size: 14px; font-weight: 700; color: #37474f; margin: 20px 0 10px; display: flex; align-items: center; gap: 8px; }
.sub-header::after { content: ''; flex: 1; height: 1px; background: #e0e0e0; }
.module-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(220px, 1fr)); gap: 10px; }
.module-card { border: 1px solid #e0e0e0; border-radius: 6px; padding: 12px 14px; }
.module-card h4 { font-size: 13px; font-weight: 700; margin-bottom: 6px; color: #1a237e; }
.module-card .module-files { font-family: 'Consolas', monospace; font-size: 11px; color: #546e7a; line-height: 1.8; }
.module-card .module-files span { display: block; }
footer { text-align: center; padding: 24px; color: #90a4ae; font-size: 12px; }
```

---

## 注意事项

1. 报告中所有 diff 代码片段使用 `<pre class="code">` 包裹，新增行用 `<span class="add">` 标记，删除行用 `<span class="del">` 标记，注释用 `<span class="comment">` 标记。
2. 若某类风险（如高风险）为 0 项，对应小节显示"本次变更未发现高风险问题 ✅"。
3. 统计卡片中的数字必须与报告内容实际一致。
4. 输出 HTML 文件后，告知用户文件保存路径。
