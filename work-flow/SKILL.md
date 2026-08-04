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

> **可选依赖**：仅「飞书文档来源」需要独立工具 `lark-cli`（本 skill 不含）；用本地 md / md+图文件夹来源则零额外依赖。故不预先提示安装，只在实际用飞书来源且 `lark-cli` 不存在时才提示装（见 split-guide）。

以下为**跨阶段铁律**（各阶段的具体做法在对应 references，进入该阶段先 Read）：

**CRITICAL — 不许瞎猜/编造：遇到看不懂、反直觉、拿不准的业务或"为什么这么设计"，MUST 用 AskUserQuestion 问用户，禁止凭想象设计、编造接口或猜答案填充。**
**CRITICAL — 知识库是核心资产，映射固定为「镜像目标项目源码树」（`<项目>/src/auth/` ↔ `<项目>/knowledge/src/auth/KNOWLEDGE.md`，一模块一份）；只记读代码得不到的业务含义/缘由/雷区，禁止流水账，用方法名/符号名锚定而非行号（详见 knowledge-guide）。**
**CRITICAL — 用户对业务/设计缘由的解答 MUST 落知识库，别问完丢一边；开发改动影响已记录知识 MUST 提议更新对应 KNOWLEDGE.md（只提议不擅自写，用户确认后落）。**
**CRITICAL — 落盘先确认：写需求/方案/知识库时目标已存在，MUST 先让用户选 覆盖 / 更新 / 仅预览，禁止直接覆盖。**
**CRITICAL — 需求文档、技术方案属个人本地过程产物、不进 git：方案写在 `<项目>/docs/design/`，首次落盘前 MUST 确保项目 `.gitignore` 含 `docs/design/`；工作区根若是 git 仓库，确保含 `requirements/`。`+kb` 首次建库前 MUST 问用户个人(`CLAUDE.local.md`+gitignore) / 共享(`CLAUDE.md`)，授权后写加载约定块，已有则跳过。**
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

首次在某工作区用一次：定位工作区根 → 建 `requirements/` → 探测父目录下的候选项目（含前后端）→ 输出清单供确认。

```bash
# 探测候选项目（前端 package.json / 后端 pom.xml、go.mod、Cargo.toml 等）
find . -maxdepth 2 \( -name package.json -o -name pom.xml -o -name build.gradle -o -name go.mod -o -name Cargo.toml -o -name requirements.txt \) -not -path '*/node_modules/*' | sed 's|/[^/]*$||' | sort -u
```

**幂等**：已初始化过再跑，只补齐新增项目，不重复建目录、不全量重探测。

### 阶段 1：拆需求（+split）

**CRITICAL — 进入本阶段 MUST 先 Read [`references/split-guide.md`](references/split-guide.md)，抓取方式（本地 md / md+图文件夹 / 飞书）、拆分粒度、项目定位、需求模板全在其中。**

抓取来源 → 拆成一个个独立需求 → 定位涉及项目 → 逐份写 `requirements/<拆分日期>/<需求id>.md`。
**SKILL 独有**：若还没初始化（无候选项目/`requirements/`）→ 先就地 `+init` 再继续。

### 阶段 2：技术方案（+design）

**CRITICAL — 进入本阶段 MUST 先 Read [`references/design-guide.md`](references/design-guide.md) 与 [`references/knowledge-guide.md`](references/knowledge-guide.md)，逐项目调研/澄清/复杂度判断/方案结构/知识落盘判定全在其中。**

按需求 id 在 `requirements/*/<需求id>.md` 定位取 `projects` → 对每个涉及项目：调研代码 → 澄清 → 出方案 `<项目>/docs/design/<需求id>.md` → 知识落盘判定。
**SKILL 独有**：定位时找不到/重名多个则提示；首次在某项目落方案前确保 `.gitignore` 含 `docs/design/`；落盘后更新该项目 status 为 `designed`（见「多项目进度」）。

### 阶段 3：开发（+dev）

**CRITICAL — 进入本阶段 MUST 先 Read [`references/dev-guide.md`](references/dev-guide.md)，按方案开发/小步提交/持续更新知识库/条件化单测/人工验收全在其中。**

**入参路由（SKILL 独有，先判断怎么进）**：
- 有方案文档 → Read 方案，按方案开发。
- 有需求文档但无方案 → 可能是简单需求跳过了设计，确认后直接按需求文档开发。
- 像 id 但需求/方案都找不到 → 提示先 `/work-flow +design <id>`，不擅自开写。
- 自由文字无 id → 普通开发，把用户输入当开发需求做。

开发开始把该项目 status 置 `dev`、验收通过置 `done`（见「多项目进度」）；其余按 dev-guide 走。

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

脱离具体需求,主动为某项目/模块建或补知识库(新接手项目、想沉淀某块业务)。首次建库先问个人/共享配置并写加载约定 → 按模块逐个建(禁止整项目一次全建)、逐文件读码 + 问用户 → 逐段确认后落 KNOWLEDGE.md。完整流程见 knowledge-guide「六、主动构建」。

## 专项规则文档（按需读取）

| 场景 | 参考文档 |
|------|---------|
| 抓取文档、拆分粒度、需求文档模板与 frontmatter | [`references/split-guide.md`](references/split-guide.md) |
| 调研套路、复杂度判断、方案文档分支树结构 | [`references/design-guide.md`](references/design-guide.md) |
| 镜像源码树知识库的映射、KNOWLEDGE.md 结构、提问落盘规则 | [`references/knowledge-guide.md`](references/knowledge-guide.md) |
| 按方案开发、知识库过时内容定位与持续更新节奏 | [`references/dev-guide.md`](references/dev-guide.md) |
