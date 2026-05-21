---
name: sediment
version: 1.0.0
description: "项目知识沉淀：把项目约定、编码规范、踩过的坑、领域知识持续沉淀到 .ai-sediment/ 目录，并通过 CLAUDE.md 自动加载，作为后续 AI 编码的向导。高频场景优先使用 Shortcuts：+save（直接写入指定分类）、+extract（从当前对话提取候选条目）、+init（首次初始化目录与 CLAUDE.md 引用）、+list（查看已沉淀索引）。"
metadata:
  requires:
    bins: ["git"]
triggers:
  - 沉淀
  - 记一下
  - 记到文档
  - 这个要记下来
  - 踩坑
  - sediment
---

# Sediment

**CRITICAL — 任何写入操作前，MUST 先确认项目根目录存在 `.ai-sediment/` 与 `CLAUDE.md` 中对 `@.ai-sediment/INDEX.md` 的引用。若任一不存在，必须先执行 `+init`，禁止跳过。**  
**CRITICAL — 写入新条目前，MUST 先用 Read 工具读取目标分类文件并 grep 关键词去重。发现相似条目时，默认走"合并/更新"，禁止盲目追加导致重复。**  
**CRITICAL — 严禁沉淀以下内容：通用编程知识（语法、标准库用法）、当前会话的临时调试信息、代码本身能直接体现的命名/格式约定、可在 git log/git blame 中找到的事实。沉淀只服务于"非显而易见、未来 AI 看代码看不出来"的知识。**  
**CRITICAL — `+extract`（对话提取）必须先把候选条目按分类列给用户确认，**用户明确回复后**才能写入。绝不允许 AI 自行判断后直接落盘。**  
**CRITICAL — 修改 `CLAUDE.md` 属于敏感操作，仅允许 `+init` 在用户授权后追加 `@.ai-sediment/INDEX.md` 引用行；其他场景一律不得改动 CLAUDE.md。**  
**CRITICAL — 评分系统是"半自动"的：分数调整 MUST 由用户的显式动作触发（命中合并 / 完全重复 +1、`+revise` -2、`+review` 中标记归档）。"覆盖"语义为旧条目归档至 `_archived/`、新条目从 score=1 起算，不进行 -1 操作。绝对禁止 AI 在对话中自行判断"这条规则被触发"或"被违反"而擅自调分；也绝对禁止依据评分自动删除任何条目，低分条目只能通过 `+review` 由用户确认后软归档。**  
**CRITICAL — 硬删除（真正从磁盘移除条目）仅允许通过 `+purge` 入口执行，且必须经用户逐条或批量确认；其他任何场景一律只能软归档到 `_archived/`，禁止直接 `rm` 或重写文件来抹除条目。**

## 调用方式

```
/sediment <自由内容>                    # 等价于 +extract，让 Skill 自动判断分类
/sediment +save <分类> <内容>           # 直接写入指定分类
/sediment +extract                      # 扫描当前会话，提取候选条目
/sediment +init                         # 初始化 .ai-sediment/ 目录与 CLAUDE.md 引用
/sediment +list                         # 列出已沉淀的全部条目索引
/sediment +review [--stale]             # 回顾低分 / 久未命中的条目，逐条选保留/归档/升级
/sediment +revise <关键词>              # 显式标记某条规则已过期（评分 -2）
/sediment +archived [<分类>] [--reason <关键词>]   # 查看 _archived/ 中的归档条目
/sediment +restore <关键词>             # 从 _archived/ 恢复条目到 active
/sediment +purge [--older-than <天数>]  # 永久删除归档条目（唯一硬删除入口，需逐条/批量确认）
/sediment +search <关键词>              # 跨全部分类的全文搜索，按 score 降序输出
/sediment +doctor                       # 健康自检：检查 INDEX 一致性、元数据格式、CLAUDE.md 引用、.gitignore 等
```

## 核心概念

### 目录结构

```
{项目根}/
├── CLAUDE.md                      ← 顶部追加 @.ai-sediment/INDEX.md
└── .ai-sediment/
    ├── INDEX.md                   ← 索引（被 CLAUDE.md 自动加载）
    ├── how-it-works.md            ← 架构 / 设计决策
    ├── how-to-write.md            ← 约定 + 规范
    ├── how-to-run.md              ← 操作流程
    ├── how-to-debug.md            ← 踩过的坑 + 调试
    ├── what-it-means.md           ← 领域术语 / 业务规则
    └── _archived/                 ← 软归档目录（不被 CLAUDE.md 加载）
        └── {分类}.md              ← `+review` 归档的条目去向
```

### 五类沉淀（按"问题-导向"组织）

| 分类 | 文件 | 适用场景 |
|------|------|---------|
| `how-it-works` | `how-it-works.md` | 架构 / 模块边界 / 为什么这么设计（最容易丢失的"为什么"） |
| `how-to-write` | `how-to-write.md` | 约定 + 规范（命名、目录、提交、代码风格、必须用什么 / 禁止用什么） |
| `how-to-run` | `how-to-run.md` | 操作流程（部署、本地起服务、CI、调试命令、特殊环境变量） |
| `how-to-debug` | `how-to-debug.md` | 踩过的坑 + 调试经验：症状 + 根因 + 修法 + 避免 |
| `what-it-means` | `what-it-means.md` | 领域术语 / 业务规则 / 状态机 / 外部系统语义 |

### 沉淀准入标准（决定要不要写）

✅ **该写**：非显而易见、读代码读不出来、踩过具体的坑、有反直觉的约束。  
❌ **不该写**：通用编程知识、显而易见的命名风格、当下任务的临时上下文、可直接 grep 代码得到的事实。

详细分类规则与条目格式见 [`references/sediment-categories.md`](references/sediment-categories.md)。

## Shortcuts

| Shortcut | 说明 |
|----------|------|
| [`+init`](references/sediment-categories.md) | 创建 `.ai-sediment/` 目录、生成各分类空文件、默认将 `.ai-sediment/` 加入 `.gitignore`（用户明确说"不加"时跳过）、在 `CLAUDE.md` 顶部追加 `@.ai-sediment/INDEX.md` 引用 |
| [`+save`](references/sediment-categories.md) | 直接将一条内容写入指定分类（含去重检查与评分） |
| [`+extract`](references/sediment-extract.md) | 扫描当前会话，提取候选条目，分类后请用户确认再落盘；命中已有条目时 +1 |
| `+list` | 读取并展示 `INDEX.md` 全部条目标题与评分 |
| [`+review`](references/sediment-categories.md) | 列出低分（≤ 0）或长期未命中的条目，逐条选 保留 / 归档 / 升级为核心 |
| [`+revise`](references/sediment-categories.md) | 显式标记某条规则已过期，评分 -2，触发归档询问 |
| [`+archived`](references/sediment-categories.md) | 查看 `_archived/` 中的归档条目，可按分类或归档原因筛选 |
| [`+restore`](references/sediment-categories.md) | 从 `_archived/` 恢复条目到 active 文件，元数据重置但保留原始 `added` |
| [`+purge`](references/sediment-categories.md) | 永久删除归档条目（默认 `--older-than 180`），唯一允许硬删除的入口，需用户逐条 / 批量确认 |
| [`+search`](references/sediment-categories.md) | 跨全部分类全文搜索，输出匹配条目（分类 / 标题 / score / 行号），按 score 降序排列 |
| [`+doctor`](references/sediment-categories.md) | 健康自检：检查 INDEX 一致性、元数据格式、CLAUDE.md / .gitignore 引用；发现问题后对可修复项逐条询问用户是否修复，确认后才执行；元数据损坏等复杂问题只给建议，不发起修复 |

## 执行流程

### Step 0：前置检查

任何 Shortcut 执行前，先检查：

```bash
test -d {项目根}/.ai-sediment && grep -q ".ai-sediment/INDEX.md" {项目根}/CLAUDE.md
```

任一不满足 → 输出提示并自动执行 `+init`，征得用户同意后继续。

### Step 1：选择 Shortcut 路由

根据用户输入分发：

| 用户输入 | 路由到 |
|----------|-------|
| `+init` 或前置检查未通过 | `+init` 流程 |
| `+save <分类> <内容>` | 直接写入流程（必须 Read 对应文件 + grep 去重） |
| `+extract` 或仅 `/sediment` 无内容 | 对话提取流程，MUST 先读 [`references/sediment-extract.md`](references/sediment-extract.md) |
| `/sediment <自由内容>`（带内容但未指定分类） | 等价 `+extract`，把"自由内容"作为唯一候选喂入提取流程 |
| `+list` | Read `INDEX.md` 直接输出 |

### Step 2：写入规则（所有写入路径共用）

1. **去重**：MUST 先 Read 目标文件，grep 关键词。命中相似条目时让用户从以下动作中选择：
   - **合并**（内容互补）→ 保留原条目 + 补入新信息，`score+=1, hits+=1, last_hit=今天`。
   - **覆盖**（旧的过期了）→ 旧条目移到 `_archived/`（带"被覆盖"标记），新条目独立写入，初始 `score=1, hits=1`。
   - **新增**（两者并存）→ 旧条目不变，新条目独立写入，初始 `score=1, hits=1`。
   - **跳过** → 什么都不做。
   - 完全重复（无歧义） → 直接 `score+=1, hits+=1`，不写入新条目。
2. **格式**：严格按 [`references/sediment-categories.md`](references/sediment-categories.md) 中各分类的模板写入。
3. **元数据**：每条结尾追加 `<!-- added: YYYY-MM-DD | score: N | last_hit: YYYY-MM-DD | hits: N -->`：
   - 新写入 / 覆盖后的新条目：`score=1, hits=1, last_hit=今天`
   - 命中合并 / 完全重复：`score+=1, hits+=1, last_hit=今天`
   - 日期一律从系统取，禁止瞎写。
4. **更新 INDEX**：写入新条目后，在 `INDEX.md` 对应分类下追加一行 `- [{标题}]({文件}#{anchor}) <!-- score: N -->`。

### Step 3：写入完成输出

```
[沉淀] 已写入 {分类}：{条目标题}
[索引] .ai-sediment/INDEX.md
```

如果是合并而非新增：

```
[合并] {分类} 已有相关条目「{原标题}」，已补充新信息
```

## 与全局 Auto Memory 的边界

两者**都属于个人本机**，区别在范围：

- **Sediment（本 Skill）**：项目级、本机可见、**默认 gitignore**、记录"个人对该项目"的沉淀。
- **Auto Memory（`~/.claude/projects/.../memory/`）**：用户级（跨项目）、本机可见、记录"个人跨项目"的偏好和习惯。

判断准则：**只对当前项目有效的认知 → sediment；跨所有项目都成立的偏好 → auto memory。**

**为什么不签入 git**：评分系统按个人使用频率累积，团队共享会让数据失真；不同人对"什么算约定"的理解不同，强制共享会引发争议；如果团队确实需要共识规则，应该写进 `CLAUDE.md` 或 `docs/` 这种正式文档而不是 sediment。
