---
name: work-flow
version: 1.0.0
description: "需求驱动全流程编排：输入一份需求 md 或飞书文档链接，拆分成一个个独立需求文档（按前后端功能拆、对应不同项目，大功能可拆成多个功能点便于协作），再逐需求做技术方案设计（贴合项目现有架构），设计中问出的业务/隐藏约束沉淀进镜像源码树的知识库，最后按方案开发、开发中持续更新知识库。高频场景优先使用 Shortcuts：+split <文档>（抓取+拆需求）、+design <需求>（调研+方案）、+dev <需求>（按方案开发）、+init（初始化工作区）。"
metadata:
  requires:
    bins: ["find", "grep", "git"]
triggers:
  - work-flow
  - 拆需求
  - 需求拆分
  - 需求文档
  - 需求编排
---

# Work-Flow

自包含的需求全流程编排 skill：抓取文档 → 拆需求 → 逐需求技术方案 → 按方案开发。不依赖任何其他 skill。

**CRITICAL — 抓取文档前 MUST 先判定来源：本地 `.md` 用 Read；本地文件夹(md+图)用 Read 读 md + 逐张 Read 图片；飞书链接（含 feishu.cn / larksuite）用 `lark-cli docs +fetch --api-version v2 --doc <url>`（只拿文本、图会丢，缺图致描述不清 MUST 问用户补清或建议改喂 md+图文件夹）。判不准就问用户，禁止猜。**
**CRITICAL — 拆需求 MUST 一个需求一份独立 md，写入工作区 `requirements/<拆分日期>/<需求id>.md`（日期形如 `2026-07-21`，同一批拆分的需求归同一日期文件夹，做完可整个文件夹删除），frontmatter 声明 `projects`（涉及的项目名列表）。禁止把多个需求塞进一份文档。**
**CRITICAL — 项目定位固定为「同父目录自动发现」：工作区父目录下的子目录即候选项目。判断某需求涉及哪些项目 MUST 基于真实存在的目录，禁止编造项目名或路径。**
**CRITICAL — 生成方案前 MUST 先调研目标项目相关代码（按线索定位 + 逐文件 Read），优先复用现有函数/工具/模式；方案引用的现有代码 MUST 标 `文件:行号` 且确认真实存在，禁止凭想象设计或编造接口。**
**CRITICAL — 设计/开发中遇到看不懂、反直觉、拿不准的业务或"为什么这么设计"，MUST 用 AskUserQuestion 主动问用户，禁止猜一个答案填进去。用户的解答 MUST 落盘进知识库，附 `文件:行号`。**
**CRITICAL — 知识库映射固定为「镜像目标项目源码树」：`<项目>/src/auth/` ↔ `<项目>/knowledge/src/auth/KNOWLEDGE.md`，一个模块一份、与源码目录一一对应。禁止自创其他路径或把多个模块知识混进一份。**
**CRITICAL — 开发阶段改动若影响已记录的知识，MUST 提议更新对应 KNOWLEDGE.md（只提议不擅自落盘，用户确认后写）。禁止改完代码把知识丢一边。**
**CRITICAL — 写入需求文档/方案文档/知识库属落盘操作；目标文件已存在时 MUST 先提示用户选 覆盖 / 更新 / 仅预览，禁止直接覆盖。**
**CRITICAL — `+kb` 首次为项目建库时 MUST 先问用户「个人配置 or 共享配置」：个人 → 写 `<项目>/CLAUDE.local.md` 且确保 `.gitignore` 含它（不进 git）；共享 → 写 `<项目>/CLAUDE.md`（进 git 团队共享）。再往所选文件写知识库加载约定块（说明 KNOWLEDGE.md 在哪、怎么读、怎么更新）。改这两个文件属敏感操作，仅在用户授权后追加,已有约定块则跳过。**
**CRITICAL — 范围克制：只做当前需求，禁止顺手重构周边或替用户设计未来需求。开发结果由人工验收，skill 不自行判定验收通过。**

## 工作区约定

```
父目录/                          ← 工作区根（多个前端项目的公共父目录）
├── requirements/               ← 拆分后的需求文档集中管理（不复制进项目）
│   ├── 2026-07-21/             ← 按拆分日期分文件夹，整批做完可一键删除
│   │   ├── <需求id>.md         ← 一个需求一份，frontmatter 声明 projects
│   │   └── ...
│   └── ...
├── knowledge/_flows/<名称>.md   ← 工作区级：跨项目业务链路知识（involves 列涉及项目）
├── project1/                   ← 项目（前端/后端,自动发现）
│   ├── knowledge/src/.../KNOWLEDGE.md   ← 该项目镜像源码树的知识库
│   └── docs/design/<需求id>.md          ← 该项目内该需求的技术方案
├── project2/
└── ...
```

- **项目发现**：工作区父目录下的每个子目录（排除 `requirements/` 及隐藏目录）为候选项目,前端后端都算。含 `package.json`/`pom.xml`/`go.mod`/`Cargo.toml` 等工程标识的视为项目,并据此粗判前端/后端。
- **需求 ↔ 项目**：需求文档 `projects` 字段列出涉及的项目名；一个需求横跨多项目时，在每个项目各生成一份方案文档 `<项目>/docs/design/<需求id>.md`。
- **知识库**：每个项目独立维护 `knowledge/`，镜像自己的源码树，与文件一一对应。

## 调用方式

```
/work-flow <文档路径 | 文件夹 | 飞书链接>  # 等价 +split，从文档进入全流程
/work-flow +init                     # 初始化工作区：建 requirements/、探测子项目
/work-flow +split <文档 | 文件夹 | 链接>   # 抓取（md / md+图文件夹 / 飞书）+ 拆成一个个需求 md
/work-flow +design <需求id | 需求md> # 逐需求技术方案设计（调研代码 + 生成方案 + 补知识库）
/work-flow +dev <需求id | 需求md>    # 按方案开发 + 开发中持续更新知识库
/work-flow +kb <项目 | 模块路径>     # 主动构建知识库（读码+问用户，按模块落 KNOWLEDGE.md）
/work-flow +status                   # 列出所有需求及其阶段（拆分/设计/开发）
```

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `<文档路径 \| 文件夹 \| 飞书链接>` | 需求来源：本地 `.md` / 含 md+图的本地文件夹 / 飞书文档 URL | 必填 |
| `<需求id \| 需求md>` | 需求编号（在 `requirements/*/` 各日期文件夹里 glob 定位）或直接给文档路径 | 必填 |
| `<项目 \| 模块路径>` | `+kb` 的目标，项目名或项目内模块目录 | 必填 |

## Shortcuts

| Shortcut | 说明 |
|----------|------|
| `+init` | 建 `requirements/`、扫描父目录列出候选项目，供后续拆分标注 projects |
| `+split <文档>` | 抓取（本地 Read / 飞书 fetch）→ 拆成独立需求 → 每需求一份 md |
| `+design <需求>` | 对需求涉及的每个项目：调研代码 → 澄清 → 生成方案 → 问出的业务落知识库 |
| `+dev <需求>` | 读方案 → 按方案开发 → 改动影响已记录知识则提议更新 → 人工验收 |
| `+kb <项目 \| 模块路径>` | 主动构建知识库：读码 + 问用户 → 按模块生成镜像源码树 KNOWLEDGE.md |
| `+status` | 汇总 requirements/ 各需求的 projects 与阶段状态 |

## 执行流程

四个阶段串起来是完整链路，也可各自单独调用。**每阶段的详细规则在对应 references，进入该阶段 MUST 先 Read。**

### 阶段 0：初始化工作区（+init）

首次在某工作区开启流程时用一次：

1. **定位工作区根**：默认当前目录为父目录；含多个前端项目子目录。判不准就问用户。
2. **建 `requirements/`**（空目录 `.gitkeep` 占位）。
3. **探测候选项目**：列出父目录下的子目录（排除 `requirements/`、隐藏目录），据工程标识文件粗判前端/后端，作为后续拆分标注 `projects` 的可选值。
4. 输出候选项目清单（含前端/后端标注）供用户确认。

**重复执行幂等**：已初始化过（`requirements/` 已存在）再跑 `+init`，不重复建目录、不全量重探测——只对比当前子目录与上次，**补齐新增的项目**列出来，已知项目不动。

```bash
# 探测候选项目（前端 package.json / 后端 pom.xml、go.mod、Cargo.toml 等）
find . -maxdepth 2 \( -name package.json -o -name pom.xml -o -name build.gradle -o -name go.mod -o -name Cargo.toml -o -name requirements.txt \) -not -path '*/node_modules/*' | sed 's|/[^/]*$||' | sort -u
```

### 阶段 1：拆需求（+split）

**CRITICAL — 进入本阶段 MUST 先 Read [`references/split-guide.md`](references/split-guide.md)，按其中的抓取方式、拆分粒度、需求模板执行。**

1. **判源 + 抓取**：本地 `.md` 用 Read；本地文件夹(md+图) Read md + 逐张 Read 图片；飞书链接用 `lark-cli docs +fetch`（**只拿文本、图会丢**，缺图致描述不清则问用户补清，或建议导出成 md+图文件夹再喂）。
2. **拆分**：把文档内容拆成一个个独立、可单独设计与开发的需求。粒度规则见 split-guide。
3. **定位项目**：从 `+init` 已探测的候选项目里判断每个需求涉及哪些项目，填 `projects`。**若发现还没初始化（无候选项目/无 `requirements/`）→ 先就地执行 `+init` 初始化再继续。** 判不准就问。
4. **落盘**：每需求写 `requirements/<拆分日期>/<需求id>.md`（日期取当天，同批需求同一文件夹；模板见 split-guide）。已存在则提示 覆盖/更新/预览。

### 阶段 2：技术方案（+design）

**CRITICAL — 进入本阶段 MUST 先 Read [`references/design-guide.md`](references/design-guide.md) 与 [`references/knowledge-guide.md`](references/knowledge-guide.md)。**

1. **读需求**：按需求 id 在 `requirements/*/<需求id>.md` 定位（glob 跨日期文件夹找），Read 后取 `projects`。找不到或重名多个则提示用户。
2. **逐项目设计**：对每个涉及项目，进入其目录上下文：
   - 调研相关代码（定位 + 逐文件 Read），提炼现状/可复用/约束/影响范围，标 `文件:行号`。
   - 先加载该项目已有 `knowledge/` 作背景；就近优先、向上补充。
   - 逻辑链条有断点、业务不懂、"为什么这么设计"拿不准 → AskUserQuestion 问用户。
   - 复杂度判断：≥2 个关键决策点先列选型+取舍让用户拍板；简单则出单一推荐方案。
   - 生成方案写 `<项目>/docs/design/<需求id>.md`（分支树结构，模板见 design-guide）。
3. **知识落盘判定（每次必做,不可静默跳过）**：方案完成后显式判定本次有无值得沉淀的隐藏约束/业务缘由——有则按 knowledge-guide 提议落 KNOWLEDGE.md,无则明说原因跳过(详见 design-guide 五)。

### 阶段 3：开发（+dev）

**CRITICAL — 进入本阶段 MUST 先 Read [`references/dev-guide.md`](references/dev-guide.md)。**

0. **入参路由（先判断怎么进）**：
   - **有方案文档**（`<项目>/docs/design/<需求id>.md` 存在）→ Read 方案，按方案开发。
   - **入参是需求 id、有需求文档但无方案**（`requirements/*/<需求id>.md` 存在、design 不存在）→ 可能是简单需求跳过了设计，直接按需求文档开发（先跟用户确认"跳过设计直接开发"）。
   - **入参像 id 但需求文档、方案都找不到** → 提示用户「未找到该需求，先 `/work-flow +design <id>` 生成方案」，不擅自开写。
   - **入参是自由文字、无 id** → 按普通开发流程，把用户输入直接当开发需求做。
1. **置 dev 态**：开发正式开始时，把需求文档对应项目的 status 置 `dev`（见「多项目进度」）。无需求文档的自由输入跳过。
2. **按方案开发（小步快跑）**：贴合项目现有约定；复用调研阶段确认的现有能力。每写完一个完整功能/模块就主动提示用户提交 git（提示,用户确认才提,不自动）。
3. **持续更新知识库（收尾必做显式判定）**：开发完成后显式判定——改动是否让已记录知识与代码现状对不上（过时）、或读码问出新隐藏约束。有则**提议更新**对应 KNOWLEDGE.md（用户确认才写）,无则明说原因,不可静默跳过(详见 dev-guide 二)。
4. **单元测试（条件化）**：探测项目(前端/后端)有无单测机制,有才为改动的行为补单测、贴合现有写法;无则跳过,禁止为单需求硬塞框架。集成/e2e 不在此步(后续靠 playwright/chrome-devtools 单独拆)。
5. **人工验收**：实现完成后交人工验收，skill 不自行判定通过。人工确认通过后，把该项目 status 置 `done`。

### 多项目进度（status 记到项目粒度，仅本地）

**范围说明：需求文档与技术方案不进 git，属你本地的过程产物。故这里的 status 只反映「你自己」对手上需求的推进，不会同步给同事，`+status` 也只汇总你本地的进度——不是团队看板。** 用途是：一个你负责的需求常横跨前后端多个项目，你自己得记清哪个项目设计完了、哪个还在开发。

一个需求横跨多项目时，各项目进度可能不齐（p1 设计完、p2 还没动）。故 status **记到项目粒度**，写在 `projects` 字段里，每个项目名后带自己的状态：

```yaml
projects: [商城前端(designed), 订单后端(dev)]
```

- 单项目状态取值：`split`（已拆分未设计）/ `designed`（方案已出）/ `dev`（开发中）/ `done`（人工验收通过）。
- 各阶段只更新**当前所动项目**的状态：`+design` 完某项目 → 该项目置 `designed`；`+dev` 开始 → `dev`；验收过 → `done`。不碰其他项目的状态。
- 需求整体阶段 = 所有项目状态取**最落后**的那个（全 done 才算需求 done）。

### +status（汇总所有需求进度）

1. 遍历 `requirements/*/` 各日期文件夹下所有 `<需求id>.md`，读 frontmatter 的 `id`/`title`/`projects`。
2. 按日期文件夹分组，逐条列出需求及各项目状态；需求整体阶段取最落后项目状态。
3. 输出：

```
[进度] requirements/2026-07-21/
  - login-sso   单点登录     商城前端(done) 订单后端(dev)      → 整体:dev
  - order-export 订单导出    数据后台(designed)  依赖:login-sso → 整体:designed
[进度] requirements/2026-07-28/
  - ...
```

有 `depends_on` 的标出「依赖:<id>」，前置需求未 done 的额外提示"被阻塞"。只读不改，纯汇总。**只汇总你本地的需求文档，反映你自己的进度，不含同事的（需求/方案不进 git）。**

### 主动构建知识库（+kb，独立于需求）

**CRITICAL — 进入本命令 MUST 先 Read [`references/knowledge-guide.md`](references/knowledge-guide.md)，按镜像源码树映射与写作纪律执行。**

脱离具体需求,主动为某项目/模块建或补知识库。用于新接手项目、想沉淀某块业务。

0. **首次为该项目建库 → 写加载约定**：先问用户个人 or 共享配置——个人写 `CLAUDE.local.md`(+`.gitignore`)、共享写 `CLAUDE.md`(进 git)。所选文件无约定块时征得同意后追加(模板见 knowledge-guide 七),已有则跳过。
1. **定位目标**：入参是项目名 → 列该项目顶层模块清单让用户选先做哪些；入参是模块路径 → 直接对该模块。
2. **按模块逐个建**：一次一个模块,禁止整项目一次全建。对每个模块：
   - 逐文件 Read 核心源码（禁止只凭文件名/目录推断）。
   - 看不懂/反直觉/"为什么这么设计" → AskUserQuestion 问用户,答案是知识库最值钱的部分。
   - 整理成 KNOWLEDGE.md 草案,逐段与用户确认后写 `<项目>/knowledge/<模块路径>/KNOWLEDGE.md`。
   - 已存在 → 只改与代码现状对不上的章节,没变的保留。
3. **输出**：已建/更新的 KNOWLEDGE.md 路径清单 + 建议下一个模块。

## 专项规则文档（按需读取）

| 场景 | 参考文档 |
|------|---------|
| 抓取文档、拆分粒度、需求文档模板与 frontmatter | [`references/split-guide.md`](references/split-guide.md) |
| 调研套路、复杂度判断、方案文档分支树结构 | [`references/design-guide.md`](references/design-guide.md) |
| 镜像源码树知识库的映射、KNOWLEDGE.md 结构、提问落盘规则 | [`references/knowledge-guide.md`](references/knowledge-guide.md) |
| 按方案开发、知识库过时内容定位与持续更新节奏 | [`references/dev-guide.md`](references/dev-guide.md) |
