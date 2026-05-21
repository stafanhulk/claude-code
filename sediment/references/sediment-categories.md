# Sediment 分类与写入模板

本文件定义 5 个分类的判定规则、文件骨架、条目模板，以及 `+init` / `+save` 的执行细节。

---

## 一、分类判定决策树

收到一条内容时，按以下顺序判断（命中即停止）：

1. 描述的是**架构选择 / 模块边界 / "为什么这么设计"**（与具体代码风格无关）？→ `how-it-works`
2. 描述的是**一次具体故障 + 解决过程**或**调试经验**？→ `how-to-debug`
3. 描述的是**怎么操作**（部署 / 本地起服务 / CI / 调试命令 / 特殊环境变量）？→ `how-to-run`
4. 描述的是**「必须这样写 / 禁止那样写」**或命名 / 目录 / 提交规范？→ `how-to-write`
5. 描述的是**业务术语 / 状态机 / 外部系统语义**？→ `what-it-means`
6. 都不符合 → 反问用户哪类，或拒绝沉淀（理由：太宽泛，不够"非显而易见"）。

## 二、文件骨架（首次创建时使用）

### `.ai-sediment/INDEX.md`

INDEX.md 同时承担两个职责：① 通过 `CLAUDE.md @.ai-sediment/INDEX.md` 进入 system prompt，向 Claude 解释**何时该读哪个分类文件**；② 维护各分类条目索引便于人类浏览。

```markdown
# 项目沉淀（个人）

> 本文件是项目个人沉淀的入口，记录 AI 编码时需要遵守的项目特有约定、踩过的坑、操作流程等。
> Claude 在编码前 MUST 先判断本次任务涉及哪些方面，按需 Read 对应的分类文件。

## 编码前的强制读取（按任务类型）

| 任务类型 | MUST 先读 |
|----------|-----------|
| 新增代码 / 修改业务逻辑 | `how-to-write.md`（约定与规范）+ `what-it-means.md`（业务术语） |
| 排查 bug / 调试 | `how-to-debug.md`（坑与调试经验） |
| 部署 / 跑测试 / 起本地服务 | `how-to-run.md`（操作流程） |
| 重构 / 设计新模块 | `how-it-works.md`（架构决策）+ `how-to-write.md` |

## 分类文件说明

### `how-it-works.md` — 架构与设计决策
记录"为什么这么设计、模块边界、技术选型权衡"。改架构、引入新依赖、或质疑某个设计前，先读这里看历史决策。

### `how-to-write.md` — 约定与规范
记录命名、目录结构、提交规范、必须用什么 / 禁止用什么。**编辑任何代码前必读**，避免违反项目约定。

### `how-to-run.md` — 操作流程
记录部署、本地起服务、CI、调试命令、特殊环境变量。**跑命令前必读**，避免用错命令或缺参数。

### `how-to-debug.md` — 踩过的坑与调试经验
记录症状 + 根因 + 修法 + 避免方式。**遇到 bug 或异常行为时必读**，先看是不是已知坑。

### `what-it-means.md` — 业务领域知识
记录术语、状态机、外部系统语义。**接触业务字段或业务流程时必读**，避免误解语义。

## 条目索引

### 架构与设计（how-it-works）

<!-- entries:how-it-works -->

### 约定与规范（how-to-write）

<!-- entries:how-to-write -->

### 操作流程（how-to-run）

<!-- entries:how-to-run -->

### 踩过的坑与调试（how-to-debug）

<!-- entries:how-to-debug -->

### 业务领域（what-it-means）

<!-- entries:what-it-means -->
```

> **设计要点**：不使用 `@-import` 串联各分类文件（避免 system prompt 膨胀），而是通过明确的"任务类型 → MUST 先读"映射，引导 Claude 按需 Read 具体文件。配合全局 `Read(...)` 权限白名单，按需加载不会触发授权弹窗。
>
> **容量上限**：单个分类文件**建议不超过 100 条**。超过后 Claude 阅读时遵守率会下降（倾向只看前几条），此时应优先运行 `+review --stale` 归档低分 / 久未命中的条目。如果合理条目就是超过 100，考虑按子主题拆分（例如 `how-to-write-naming.md` / `how-to-write-state.md`）。

### 各分类文件首部

```markdown
# {分类中文名}

> 本文件由 sediment skill 维护，写入前请通过 `/sediment +save` 或 `/sediment +extract`。
```

## 三、条目模板（严格遵守）

### how-it-works.md 模板（架构与设计决策）

```markdown
## {决策标题}

**决策**：{选了什么 / 怎么设计的}
**为什么**：{背后的考量、约束、权衡}
**取舍**：{放弃了什么 / 为什么不选其他方案}
**影响范围**：{哪些模块依赖这个决策}

<!-- added: YYYY-MM-DD | score: 1 | last_hit: YYYY-MM-DD | hits: 1 -->
```

**示例**：

```markdown
## 前端状态拆分为 server-state 和 ui-state 两层

**决策**：服务端数据用 React Query 管理，UI 临时状态用 useState/Zustand。
**为什么**：服务端数据有缓存、重试、失效语义，与 UI 状态生命周期完全不同。
**取舍**：放弃了"全部用 Redux"的单层方案，因为会让 server-state 失去自动重新拉取能力。
**影响范围**：`src/queries/**` 全部走 React Query；组件内 useState 仅用于临时 UI 状态。

<!-- added: 2026-05-08 | score: 3 | last_hit: 2026-05-20 | hits: 5 -->
```

### how-to-write.md 模板（约定 + 规范）

```markdown
## {规则标题}

**规则**：{命名规则 / Do / 必须用什么}
**反例**：{Don't / 禁止用什么}
**为什么**：{原因}
**适用范围**：{哪些目录/文件生效}

<!-- added: YYYY-MM-DD | score: 1 | last_hit: YYYY-MM-DD | hits: 1 -->
```

**示例**：

```markdown
## 金额字段必须使用 decimal.js

**规则**：金额相关字段统一用 `Decimal`，序列化时用 `.toFixed(2)`。
**反例**：禁止用原生 `number` 做金额加减乘除（浮点误差）。
**为什么**：曾出现 0.1 + 0.2 累加偏差导致对账失败。
**适用范围**：`src/services/**`、`src/api/**` 中所有 price/amount 字段。

<!-- added: YYYY-MM-DD | score: 1 | last_hit: YYYY-MM-DD | hits: 1 -->
```

### how-to-run.md 模板（操作流程）

```markdown
## {操作场景}

**场景**：{什么时候需要做这个操作}
**步骤**：
1. ...
2. ...
3. ...

**注意事项**：{易错点 / 必需的环境变量 / 前置依赖}

<!-- added: YYYY-MM-DD | score: 1 | last_hit: YYYY-MM-DD | hits: 1 -->
```

**示例**：

```markdown
## 本地起服务前必须先 mock 飞书认证

**场景**：本地启动后端时，飞书 OAuth 在内网无法回调。
**步骤**：
1. 复制 `.env.example` 为 `.env.local`
2. 设置 `MOCK_FEISHU=true` 跳过飞书登录
3. `pnpm dev` 起服务
4. 访问 `http://localhost:3000`，会自动以 mock 用户登录

**注意事项**：`MOCK_FEISHU=true` 仅本地用，提 PR 前确认 `.env.local` 没被提交。

<!-- added: YYYY-MM-DD | score: 1 | last_hit: YYYY-MM-DD | hits: 1 -->
```

### how-to-debug.md 模板（坑 + 调试）

```markdown
## {坑的简短描述}

**症状**：{用户/系统观察到的现象}
**根因**：{为什么会发生}
**修法**：{当时怎么修的，附关键文件路径或 commit}
**避免**：{以后怎么写就不会再踩}

<!-- added: YYYY-MM-DD | score: 1 | last_hit: YYYY-MM-DD | hits: 1 -->
```

**示例**：

```markdown
## Pinia store 在 SSR 下被多个请求串数据

**症状**：用户 A 偶发看到用户 B 的姓名。
**根因**：Nuxt SSR 模式下 Pinia store 是进程级单例，被并发请求共享。
**修法**：所有 store 改为 `useXxxStore()` 在 setup 内调用，禁止模块级解构。详见 commit `a1b2c3d`。
**避免**：新建 store 时检查是否在模块顶层用了 `const { x } = useXxxStore()`。

<!-- added: YYYY-MM-DD | score: 1 | last_hit: YYYY-MM-DD | hits: 1 -->
```

### what-it-means.md 模板（业务领域）

```markdown
## {术语/概念}

**定义**：{这个词在本项目里特指什么}
**关键属性 / 状态**：{如有状态机，列出状态与流转条件}
**与外部系统的对应**：{如对接第三方，说明字段映射或语义差异}

<!-- added: YYYY-MM-DD | score: 1 | last_hit: YYYY-MM-DD | hits: 1 -->
```

**示例**：

```markdown
## "已结算"订单（settled）

**定义**：订单已完成支付且已对账，进入财务可结算状态。注意：与"已支付（paid）"不同——paid 只表示用户付钱，settled 才会触发分账。
**关键属性 / 状态**：状态机 `pending → paid → settled → refundable`，refundable 仅 7 天有效。
**与外部系统的对应**：飞书审批系统中的 `settled` 对应字段 `final_status=2`（不是 1）。

<!-- added: YYYY-MM-DD | score: 1 | last_hit: YYYY-MM-DD | hits: 1 -->
```

## 四、`+init` 执行步骤

1. 检查 `{项目根}/.ai-sediment/` 是否存在，不存在则 `mkdir -p`。
2. 检查并创建以下文件 / 目录（已存在则跳过，不覆盖）：
   - `.ai-sediment/INDEX.md`（用本文件第二节"骨架"模板）
   - `.ai-sediment/how-it-works.md`
   - `.ai-sediment/how-to-write.md`
   - `.ai-sediment/how-to-run.md`
   - `.ai-sediment/how-to-debug.md`
   - `.ai-sediment/what-it-means.md`
   - `.ai-sediment/_archived/`（软归档目录，**不被 CLAUDE.md 加载**）：
     - `_archived/INDEX.md`（归档索引，结构同主 INDEX.md 但仅列归档条目）
     - `_archived/how-it-works.md` / `how-to-write.md` / `how-to-run.md` / `how-to-debug.md` / `what-it-means.md`（5 个空骨架文件，归档时 append 条目）
3. 检查 `{项目根}/.gitignore`：
   - 文件不存在 → 创建并写入 `.ai-sediment/`。
   - 文件存在但无 `.ai-sediment/` 行 → 默认**在新行追加**该规则（必须确保前一行末尾已有换行符，禁止拼接在已有规则的行尾），同时告知用户"已添加，若不需要可手动删除"。用户在调用 `+init` 时若明确说"不加 gitignore"，则跳过此步。
   - 已有 → 跳过。
4. 检查 `CLAUDE.md`：
   - 不存在 → 询问用户是否创建，确认后写入最小骨架并加入引用行。
   - 存在但无 `@.ai-sediment/INDEX.md` → **明确告知用户**将在文件顶部追加引用行，征得同意后再写。
   - 已有引用 → 跳过。
5. 输出：

   ```
   [初始化] .ai-sediment/ 已就绪
   [gitignore] 已添加 .ai-sediment/ 排除规则
   [引用] CLAUDE.md 已加载 @.ai-sediment/INDEX.md
   ```

## 五、`+save` 执行步骤

1. 解析参数：`<分类> <内容>`。分类必须是 `how-it-works | how-to-write | how-to-run | how-to-debug | what-it-means` 之一，否则报错。
2. **拆解自由文字到模板字段**：根据分类对应的模板（见第三节），把 `<内容>` 拆到各字段。**严格禁止**：
   - 禁止用 `TBD` / `待补充` / `（暂无）` / `?` / 空字符串等占位符填充字段。
   - 拆解后若**任一必填字段无法从用户输入中提取**（如 `pitfalls` 缺"根因"或"修法"），MUST 通过 AskUserQuestion 反问用户补全，禁止猜测或硬塞。
   - 反问示例：`你提到"金额要用 decimal.js"，但缺少"为什么"——是踩过浮点精度的坑，还是单纯团队约定？请补充原因，否则这条规则未来 AI 看不到 why 会执行不到位。`
3. 用 Read 读取目标分类文件全文。
4. 用 grep 在文件中搜索内容关键词（取实词 2-3 个）。
5. 决策：
   - **完全重复** → 不新增；将原条目元数据 `score+=1, hits+=1, last_hit=今天`；输出 `[已存在] {标题}（score: N → N+1）`。
   - **命中相似** → 展示原条目和新内容差异，请用户选择：
     - `合并`：把新信息补入原条目正文，元数据 `score+=1, hits+=1, last_hit=今天`
     - `覆盖`：旧条目**移到 `_archived/{分类}.md`** 并在末尾追加 `<!-- archived: YYYY-MM-DD | reason: superseded -->`；新内容作为新条目写入（初始 `score=1, hits=1, last_hit=今天`）
     - `新增`：原条目保持不变，新内容追加为独立条目（初始 `score=1, hits=1`）
     - `跳过`：什么都不做
   - **无命中** → 直接按对应模板生成条目（初始 `score=1, hits=1, last_hit=今天`），Edit 工具追加到文件末尾。
6. 同步追加 / 更新索引行到 `INDEX.md` 对应分类的 `<!-- entries:xxx -->` 标记下方。
7. 输出 `[沉淀] 已写入 {分类}：{标题}（score: N）`。

## 六、INDEX.md 索引行格式

```markdown
- [{条目标题}](./{分类文件}#{slug}) <!-- score: N | {YYYY-MM-DD} -->
```

`slug` 取条目标题转 kebab-case。例如标题 `金额字段必须使用 decimal.js` → slug `金额字段必须使用-decimal-js`（中文保留即可，Markdown 锚点支持）。

每次写入或评分变动时，MUST 同步更新此行的 `score` 字段，确保 `+list` 输出时分数与正文一致。

## 七、评分系统

### 7.1 元数据字段定义

每条条目末尾的 HTML 注释存四个值：

```html
<!-- added: 2026-05-08 | score: 3 | last_hit: 2026-05-20 | hits: 5 -->
```

| 字段 | 含义 | 变动时机 |
|------|------|----------|
| `added` | 首次写入日期 | 写入时设置，永不变 |
| `score` | 净分（强化 - 削弱） | 命中合并 / 完全重复 +1；`+revise` -2。"覆盖"不调分，旧条目直接归档 |
| `last_hit` | 最后一次被强化的日期 | `score` 增加时同步更新；`+review --stale` 据此判断"久未命中" |
| `hits` | 累计强化次数（只增不减） | 命中合并 / 完全重复时 +1；`+revise` 不动 |

### 7.2 评分阈值与含义

| 评分 | 状态 | 出现位置 / 行为 |
|------|------|-----------------|
| ≥ 5 | 核心规则 | `+list` 加 ⭐ 标记；`+revise` 删除前需双重确认 |
| 1 – 4 | 正常 | 不特殊处理 |
| 0 | 待观察 | `+review` 默认列入 |
| -1 | 可疑 | `+review` 高亮，建议确认 |
| ≤ -2 | 建议归档 | `+review` 默认询问"是否归档" |

### 7.3 `+review` 执行步骤

1. 解析参数：可选 `--stale`（仅列出 `last_hit` 超过 90 天的低分条目）。
2. Read 全部 5 个分类文件，解析每条的元数据。
3. 筛选目标：
   - 默认：`score ≤ 0` 的全部条目
   - `--stale`：`score ≤ 1` **且** `last_hit` 距今 > 90 天
4. 按分类分组展示，每条显示标题 / score / hits / last_hit / 正文摘要：

   ```
   [how-to-write] (score: -1, hits: 2, last_hit: 2026-02-15)
     金额字段必须使用 decimal.js
     摘要：金额相关字段统一用 Decimal...

   请选择：1=保留 / 2=归档 / 3=升级为核心(+5) / 4=略过
   ```

5. 用户逐条选择或批量选择（如"全部归档"）。
6. 执行：
   - **保留**：不变。
   - **归档**：从原文件移除该条目（含元数据），追加到 `.ai-sediment/_archived/{分类}.md` 文件末尾，元数据末尾追加 ` archived: {YYYY-MM-DD} | reason: low-score`（reason 视触发场景填，详见第八节）；从主 `INDEX.md` 移除索引行；在 `_archived/INDEX.md` 对应分类下追加索引行。
   - **升级为核心**：`score = max(score, 5)`，正文不变。
   - **略过**：跳过本次，下次 `+review` 仍会列出。
7. 输出汇总：

   ```
   [回顾] 共 N 条低分条目
   [归档] M 条已移至 _archived/
   [升级] K 条标记为核心规则
   [保留] L 条
   ```

### 7.4 `+revise` 执行步骤

显式标记某条规则已过期：

1. 解析参数：`<关键词>`（用于在所有分类中 grep 定位条目）。
2. 找到匹配条目：
   - 0 条 → 报错 `[未找到] {关键词}`。
   - 1 条 → 进入步骤 3。
   - 多条 → 列出全部，让用户选编号。
3. 显示该条目正文与当前评分，确认是否标记过期。
4. 用户确认后：
   - `score -= 2`
   - 若新 `score ≤ -2`，追问"是否立即归档？"，肯定则按 `+review` 的"归档"动作处理。
5. 输出：

   ```
   [修订] {分类} / {标题} 已扣分（score: N → N-2）
   [归档] 已移至 _archived/  ← 仅在用户选了"立即归档"时显示
   ```

### 7.5 严格禁止

- ❌ AI 在普通对话中"猜测"某条规则被命中或被违反，自行加减分。
- ❌ 评分降到阈值以下时**自动**删除或归档，必须经 `+review` / `+revise` 由用户确认。
- ❌ 硬删除（`rm`）任何条目；硬删除唯一入口是 `+purge`，且必须用户确认。
- ❌ 加载 `_archived/` 内容到 CLAUDE.md（归档目录不被自动加载，否则失去归档意义）。

## 八、归档管理

### 8.1 归档元数据

归档条目在原元数据基础上追加两个字段：

```html
<!-- added: 2026-03-15 | score: -2 | hits: 1 | last_hit: 2026-03-15
     archived: 2026-05-09 | reason: low-score -->
```

`reason` 取值（必须从下列枚举中选一个）：

| reason | 来源场景 |
|--------|---------|
| `superseded` | `+save` 时用户选"覆盖"（被新条目替代） |
| `low-score` | `+review` 中用户标"归档"（评分低 / stale） |
| `revised` | `+revise` 后用户选"立即归档"（显式标记过期） |
| `manual` | 用户通过其他方式手动操作 |

### 8.2 `_archived/INDEX.md` 骨架

```markdown
# 归档索引

> 本文件不被 CLAUDE.md 加载。仅供 `+archived` / `+restore` / `+purge` 使用。
> 索引行格式：`- [{标题}](./{分类}.md#{slug}) <!-- archived: YYYY-MM-DD | reason: ... -->`

## 架构与设计（how-it-works）

<!-- archived:how-it-works -->

## 约定与规范（how-to-write）

<!-- archived:how-to-write -->

## 操作流程（how-to-run）

<!-- archived:how-to-run -->

## 踩过的坑与调试（how-to-debug）

<!-- archived:how-to-debug -->

## 业务领域（what-it-means）

<!-- archived:what-it-means -->
```

### 8.3 `+archived` 执行步骤

查看归档条目，**不修改任何文件**。

1. 解析参数：可选 `<分类>`（5 选 1）、可选 `--reason <关键词>`（匹配 reason 枚举）。
2. Read `_archived/INDEX.md` 与各 `_archived/{分类}.md`。
3. 按筛选条件过滤，按"归档日期降序"排列展示：

   ```
   #1  [how-to-write] (archived: 2026-05-09 | reason: superseded)
       金额字段必须用 number（已被 decimal.js 规则覆盖）
       原始：added 2026-03-15 | score -1 | hits 1

   #2  [how-to-debug] (archived: 2026-04-20 | reason: low-score)
       某个临时调试 hack
       原始：added 2026-02-01 | score 0 | hits 0
   ```

4. 输出汇总：

   ```
   [归档] 共 N 条，最早 YYYY-MM-DD，最近 YYYY-MM-DD
   [提示] 用 +restore <关键词> 恢复；用 +purge 永久清理
   ```

### 8.4 `+restore` 执行步骤

把归档条目拉回 active。

1. 解析参数：`<关键词>`（在 `_archived/` 下全分类 grep）。
2. 匹配：
   - 0 条 → 报错 `[未找到] {关键词}`
   - 1 条 → 进入步骤 3
   - 多条 → 列出 `#N` 让用户选编号
3. 展示该条目正文与归档元数据，确认是否恢复。
4. 用户确认后：
   - 从 `_archived/{分类}.md` 移除该条目（含归档元数据）
   - 在主 `{分类}.md` 末尾追加该条目，元数据调整为：
     - 保留原始 `added`
     - `score = 1, hits = 1, last_hit = 今天`
     - 移除 `archived` / `reason` 字段
     - 追加 `restored_from_archive: YYYY-MM-DD`
   - 在主 `INDEX.md` 对应分类下追加索引行
   - 从 `_archived/INDEX.md` 移除索引行
5. 输出：

   ```
   [恢复] {分类} / {标题} 已从归档恢复（score: 1, hits: 1）
   [审计] 元数据保留 added: YYYY-MM-DD，新增 restored_from_archive: YYYY-MM-DD
   ```

### 8.5 `+purge` 执行步骤

永久删除归档条目，**唯一允许硬删除的入口**。

1. 解析参数：`--older-than <天数>`（默认 `180`）、`--reason <关键词>`（可选）。
2. Read `_archived/INDEX.md`，按 `archived` 日期与 reason 筛选目标条目。
3. 列出待删清单：

   ```
   即将永久删除 N 条归档条目（archived 早于 YYYY-MM-DD）：

   #1  [how-to-write]  金额字段必须用 number
       archived: 2025-08-12 | reason: superseded
   #2  [how-to-debug]  某个临时调试 hack
       archived: 2025-09-01 | reason: low-score
   ...

   请选择：
     - all / 全部确认       → 全部硬删除
     - #1,#3 / 1,3         → 仅删除指定序号
     - none / 取消         → 全部保留，终止操作
   ```

4. **MUST** 通过 AskUserQuestion 工具明确收集用户选择，禁止默认全删。
5. 确认后：从 `_archived/{分类}.md` 物理删除条目；从 `_archived/INDEX.md` 移除索引行。
6. 输出：

   ```
   [永久删除] M 条归档条目已清理
   [保留] N - M 条归档条目仍在 _archived/
   ```

### 8.6 严格禁止（归档管理专项）

- ❌ `+archived` 修改任何文件（仅查看）。
- ❌ `+restore` 跳过用户确认。
- ❌ `+purge` 在用户未明确选择时执行任何删除。
- ❌ 硬删除时不更新 `_archived/INDEX.md`，导致索引与文件失同步。
- ❌ 在 `+save` / `+review` / `+revise` 等流程中尝试硬删除，必须走 `+purge`。

## 九、查询与自检

### 9.1 `+search` 执行步骤

跨全部分类的全文搜索，**只读**，不修改任何文件。

1. 解析参数：`<关键词>`（支持空格分隔多关键词，AND 语义）。
2. 用 grep 在 5 个 active 分类文件中搜索关键词（默认不含 `_archived/`；如需含归档，提示用户用 `+archived` 或追加 `--include-archived` 参数）。
3. 命中条目按 `score` 降序、同分按 `last_hit` 降序排列：

   ```
   找到 N 条匹配「{关键词}」的条目：

   #1  [how-to-write] (score: 5 ⭐, hits: 8, last_hit: 2026-04-30)
       金额字段必须使用 decimal.js
       匹配行：**反例**：禁止用原生 `number` 做金额加减乘除（浮点误差）。
       位置：.ai-sediment/how-to-write.md:42

   #2  [how-to-debug] (score: 2, hits: 3, last_hit: 2026-03-15)
       金额浮点累加偏差导致对账失败
       匹配行：**症状**：日终对账偶发 0.01 元差异...
       位置：.ai-sediment/how-to-debug.md:18
   ```

4. 输出汇总：

   ```
   [搜索] 共 N 条命中
   [提示] 用 +list 查看全部条目；用 +archived 在归档中搜索
   ```

5. 0 命中时输出：

   ```
   [搜索] 未找到匹配「{关键词}」的条目
   [提示] 关键词可能需要换个说法，或该规则尚未沉淀（可用 /sediment +save 添加）
   ```

### 9.2 `+doctor` 执行步骤

健康自检：先只读扫描，输出问题清单，再对**可修复项**逐条询问用户是否修复，确认后才执行。

逐项检查并输出问题清单：

| 检查项 | 检查方式 | 问题表现 | 可自动修复 |
|--------|---------|---------|-----------|
| **CLAUDE.md 引用** | grep `@.ai-sediment/INDEX.md` 是否存在 | 缺失 → AI 编码时不会加载 INDEX | ✅ |
| **.gitignore 规则** | 检查 `.ai-sediment/` 是否在 .gitignore 中 | 缺失 → 个人沉淀会被提交到 git | ✅ |
| **INDEX 约束语** | 检查 INDEX.md 是否含"任务类型 → MUST 先读"段落 | 被误删 → 失去引导作用 | ✅ |
| **主 INDEX 与条目同步** | 解析每个分类文件的标题与 INDEX 索引行做 diff | 差异 → 条目数与索引不一致 | ⚠️ 需用户确认每条差异 |
| **归档 INDEX 与归档文件同步** | 同上对 `_archived/` | 差异 → 归档检索失效 | ⚠️ 需用户确认每条差异 |
| **元数据格式合法性** | 用正则匹配每条目的 HTML 注释，校验 4 个字段齐全 | 损坏 → 评分系统无法解析 | ❌ 需手动处理 |
| **元数据日期合法性** | 校验 `added` / `last_hit` / `archived` 是否为合法日期，`last_hit` 不应早于 `added` | 异常 → 可能是 AI 误填 | ❌ 需手动处理 |
| **score / hits 合法性** | `score` 应是整数，`hits ≥ 0`，`hits ≥ score` | 异常 → 元数据被人手改坏 | ❌ 需手动处理 |
| **分类文件超量** | 任一分类文件条目数 > 100 | 超量 → 提示运行 `+review --stale` | ❌ 需手动处理 |

**执行顺序：**

1. 扫描全部检查项，输出问题清单：

   ```
   [自检] 共发现 3 个问题：

   🔴 严重（影响 AI 编码加载）
     1. CLAUDE.md 缺少 @.ai-sediment/INDEX.md 引用               [可修复]

   🟡 警告（影响检索 / 评分）
     2. how-to-write.md 实际 23 条，但 INDEX.md 只有 21 条索引行  [需确认]
     3. how-to-debug.md 第 87 行条目元数据缺少 score 字段          [需手动]

   ✅ 通过（6 项）
   ```

2. 对每个标注 **[可修复]** 或 **[需确认]** 的问题，逐条询问用户：

   ```
   问题 1：CLAUDE.md 缺少 @.ai-sediment/INDEX.md 引用。是否修复？(y/n)
   ```

   - 用户回复 `y` → 执行修复并输出 `[已修复] ...`
   - 用户回复 `n` → 跳过，输出 `[已跳过] ...`

3. 标注 **[需手动]** 的问题只展示说明与建议操作，不发起询问。

**严格禁止**：未经用户确认直接修复任何问题。标注 [需手动] 的项目（元数据损坏、日期异常、评分异常等）一律不进入修复询问流程，因为这类问题可能源于用户的有意修改。
