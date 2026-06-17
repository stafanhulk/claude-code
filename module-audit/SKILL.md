---
name: module-audit
version: 2.0.0
description: "模块开发指南生成：深度阅读指定目录或模块的现有代码，提炼出编码约定、架构模式、扩展入口，生成一份「如何在该模块上继续开发」的指导文档，供人类或 AI 后续编码参考。高频场景优先使用 Shortcuts：+gen <路径>（生成指导文档）、+update <路径>（更新已有文档）、+preview <路径>（仅在对话中预览，不写文件）。"
metadata:
  requires:
    bins: ["find", "grep"]
triggers:
  - 生成开发指南
  - 模块指导文档
  - 怎么在这个模块上继续开发
  - 分析模块规范
  - module-audit
---

# Module Audit

**CRITICAL — 执行前 MUST 先确认目标路径存在，路径不存在时输出错误并终止，禁止继续。**
**CRITICAL — 分析阶段 MUST 逐文件 Read 核心源码，禁止仅凭文件列表或文件名推断内容。**
**CRITICAL — 生成的指导文档只描述"现有代码中真实存在的模式"，禁止编造未见过的约定或假设未来方向。**
**CRITICAL — 文档中所有代码示例 MUST 来自目标模块的真实代码片段，不得凭空捏造示例。**

## 调用方式

```
/module-audit +gen <路径>               # 完整分析 + 生成指导文档（写入文件）
/module-audit +update <路径>            # 重新扫描并更新已有的指导文档
/module-audit +preview <路径>           # 仅在对话中预览文档内容，不写文件
/module-audit +gen <路径> --out <文件>  # 指定输出路径（默认见下方）
```

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `<路径>` | 目标目录或单文件，支持相对/绝对路径 | 必填 |
| `--out <文件>` | 指定文档输出路径 | `{模块根}/.module-guide.md` |
| `--depth <n>` | 扫描子目录深度（0 = 仅当前层） | 无限制 |

## Shortcuts

| Shortcut | 说明 |
|----------|------|
| `+gen <路径>` | 完整流程：扫描 → 分析 → 生成文档并写入文件 |
| `+update <路径>` | 重新扫描，对比旧文档，仅更新已变化的章节 |
| `+preview <路径>` | 分析后仅在对话中展示文档草稿，不落盘 |

## 执行流程

### Step 1：路径校验与文件收集

```bash
# 校验
test -d "<path>" || test -f "<path>" || { echo "ERROR: 路径不存在"; exit 1; }

# 收集源码（排除构建产物和依赖）
find "<path>" -type f \( -name "*.ts" -o -name "*.tsx" -o -name "*.js" -o -name "*.jsx" \
  -o -name "*.py" -o -name "*.go" -o -name "*.java" -o -name "*.kt" -o -name "*.swift" \) \
  ! -path "*/node_modules/*" ! -path "*/.git/*" ! -path "*/dist/*" ! -path "*/build/*"
```

输出：文件总数 + 检测到的主语言。文件数 > 80 时，询问用户是否缩小范围或继续。

### Step 2：深度代码阅读

**CRITICAL — 按以下优先级逐文件 Read，理解现有代码后再生成文档：**

1. **入口文件**（`index.ts`、`__init__.py`、`mod.rs` 等）
2. **类型定义文件**（`types.ts`、`interfaces/`、`models/`）
3. **核心逻辑文件**（业务逻辑最集中的文件）
4. **工具/公共函数文件**（`utils/`、`helpers/`、`shared/`）
5. **测试文件**（理解预期行为和边界条件）

阅读过程中只提炼五类信息（详见 [`references/extraction-guide.md`](references/extraction-guide.md)）：
- **核心约束**：必须被保留的不变量，破坏会导致功能错误或隐式假设失效
- **设计意图**：从代码结构推断出的模块组织思路，帮助新代码保持风格一致
- **扩展路径**：新增某类功能的具体步骤，基于现有真实例子推导，注明参考位置
- **风险点**：看起来不合理但有意为之的代码，标注现象、原因、改动后果，防止误修
- **文件联动**：确认路径搜索命令可用，不枚举具体联动对

### Step 3：生成指导文档

**CRITICAL — 全文严格不超过 100 行。超出时压缩表达，不截断内容。**
**CRITICAL — 文档只包含五个章节，不增加其他章节。**

按 [`references/doc-template.md`](references/doc-template.md) 的结构生成文档，固定五个章节：

1. **核心约束** — 必须遵守的不变量，每条附 `文件:行号` 依据
2. **设计意图** — 模块组织思路，说明新代码应遵循的分工
3. **扩展路径** — 新增功能的具体步骤，注明真实参考位置
4. **风险点** — 看似有问题但有意为之的代码，三要素：现象 / 原因 / 风险
5. **文件联动** — 路径搜索命令（填入实际根路径和扩展名）+ 找到文件后的行动指引

### Step 4：写入文件

默认写入 `{模块根}/.module-guide.md`。

```
[完成] 指导文档已生成：{输出路径}
       模块：{路径}  语言：{lang}  文件数：{N}
```

若已存在旧文档，先提示用户选择：覆盖 / 追加更新 / 仅预览。

## 专项规则文档（按需读取）

| 场景 | 参考文档 |
|------|---------|
| Step 2 提炼什么、怎么提炼 | [`references/extraction-guide.md`](references/extraction-guide.md) |
| 文档结构与各章节写法 | [`references/doc-template.md`](references/doc-template.md) |
