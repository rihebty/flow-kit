# GO — flow-kit 统一入口（每个 IDE 都用这一个）

> **用户使用方式**：`@flow-kit/GO.md` + 一句话意图
> AI 看到 `@GO.md` 就按本文件路由，自动决定阶段、自动生成 ID、自动按需加载工件，**不要等用户提供 ID 或路径**。

---

## 红线·Token 预算（走任何阶段前必读）

flow-kit 的文件分两类，**加载策略不同**：

| 类型 | 路径 | 长度 | 加载方式 |
|---|---|---|---|
| **SPEC**（项目产物）| `.specs/<id>/*.md` | 通常 < 200 行 | 整读 OK |
| **REFERENCE**（查阅型）| `flow-kit/reference/*.md` | 75~470 行 | **禁止默认整读**，只 grep / read offset 需要的那一节 |
| **TEMPLATE / PROMPT** | `flow-kit/templates|prompts/*.md` | < 150 行 | 整读 OK |

**违规示例**（AI 常犯）：

- ❌ 进入 2-design 阶段后直接 `read_file flow-kit/reference/tech-stacks.md` 整读 467 行
- ✅ 正确：`grep_search` 查 `适用矩阵` 或 `read_file offset=??? limit=80` 只读所需那节

- ❌ 进入 2a-ui-design 后同时读 `ui-aesthetics.md` 与 `ui-anti-patterns.md` 两全文
- ✅ 正确：`ui-anti-patterns.md` 75 行可整读；`ui-aesthetics.md` 只读「调性」一节

**Token 预算**：进入任何阶段的首轮消息，加载的 reference 总行数 ≤ **150 行**。超过 → 拆到后面按需拉。

---

## Token 预算估计 · 进阶段前必跑

> 上面是单阶段加载预算。本段是**整链路预算**——让用户提前知道价格，并选挡位。

### 典型 token 成本表（按一个中等规模 change，前端项目，5 个 task）

| 阶段 | 完整模式 | 极简模式（跳 2a / 第四轮 / 跨模型）| 单点调用（仅跑选定阶段）|
|---|---|---|---|
| 0 + 1 + 2 + 2a + 3（规划链） | ~42k - 62k | ~30k - 45k | 按需，只跑你需要的那个 |
| 4 × 5 task（实施） | ~125k - 300k | ~125k - 300k | 单 task ~25k - 60k |
| 5（测试） | ~30k - 80k | ~20k - 50k | 单独 ~30k |
| 6（review） | ~25k - 50k | ~15k - 25k | 单独 ~25k |
| 7（集成归档） | ~20k - 40k | ~15k - 25k | 单独 ~20k |
| **总计** | **~250k - 530k** | **~205k - 445k** | **选什么跑什么** |

### 用户首轮路由后，AI 必须输出预算估算

在路由声明（见第四步）后追加一段：

```
✅ Token 预算估计：
   - 本次 change 规模：<small / medium / large>（依据：预估代码行数 / 任务数 / 是否前端）
   - 默认模式预估：~XXk - YYk tokens
   - 已选挡位：完整 / 极简 / 单点
✅ 是否继续？或换挡位？
   1. 完整（推荐 500+ 行 / 团队项目 / 长期维护）
   2. 极简（推荐 100~500 行 · 跳 2a / 跳第四轮 / 跳跨模型，省 ~20%）
   3. 单点（你只想跑某一阶段，告诉我哪一个）
   4. 不走 flow-kit（< 50 行代码 / bugfix 直接修，别走闭环）
```

### 何时**不必**跑这段

- 用户已显式指定模式（如「快速加个字段」/「极简模式跑」）
- 当前是恢复中断任务（直接走 R1.5 重启协议，不重新估）
- 用户跑横向命令（L-restyle / M-health）— 这些有自己的预算

### 真实成本影响因子（让估算更准）

把下面这些因子套进估算：

| 因子 | 影响 |
|---|---|
| 前端项目 | +20%（多 2a-ui-design 阶段 + UI 第三轮）|
| 涉及 schema 变更 | +5~10%（多 1.7 段 + 5-test 4.2 验证）|
| brooks-lint 已装 | +10%（多 4 个命令调用）|
| 跨模型 spot-check 触发 | +30%（双模型走同样 review）|
| task 数 < 3 | -30%（建议走单点调用不走闭环）|
| task 数 > 10 | +50%（建议拆 milestone）|

### 用户视角的取舍

| 你的诉求 | 选 |
|---|---|
| 想要全套产物（CHANGE / REQUIREMENT / DESIGN / TASK / SUMMARY × N / TEST / REVIEW） | 完整 |
| 想要核心产物但能少则少（DESIGN / TASK / SUMMARY × N / REVIEW） | 极简 |
| 只想跑某一阶段（如只 review / 只 design / 只 体检）| 单点 · 见 README 决策表 |
| 代码 < 50 行 · 一次性修补 · hackathon | 不走 flow-kit，走 7 个原生 skill 更划算 |

---

## 第一步 · 读取项目状态（必须，跳过即违反）

1. 尝试读 `STATE.md`（仓库根）。不存在 → 视为新项目，跳过
2. 关注字段：`活跃 Change` / `当前阶段` / `当前 Task` / `中断任务`
3. 如果存在 `中断任务` 非空 → **优先级最高**，直接走"恢复中断任务"分支（见下表）

## 第二步 · 解析用户意图，路由到阶段

按以下表格匹配用户输入（**取最先命中那一条**）：

| 用户输入特征 | 路由到 | 备注 |
|---|---|---|
| `继续` / `接着上次` / `恢复` / `resume` | `prompts/4-dev.md` 的「入场恢复」段 | 加载 `STATE.md` 中断任务对应的 PROGRESS |
| `执行 T<NN>` / `跑 T<NN>` / `do T<NN>` | `prompts/4-dev.md` | task-id 从用户输入提取 |
| `审查` / `review` / `检查代码` / `code review` | `prompts/6-review.md` | |
| `测试` / `写测试` / `UAT` / `test` | `prompts/5-test.md` | |
| `上线` / `集成` / `验收` / `ship` / `归档` | `prompts/7-integration.md` | |
| `拆任务` / `plan tasks` / `分解` | `prompts/3-task.md` | |
| `设计` + `<已有需求>` / `架构` / `design` | `prompts/2-design.md` | 当 REQUIREMENT 已存在时；后端项目 |
| `选技术` / `选栈` / `选框架` / `tech stack` / `用什么开发` / `迁移评估` | `prompts/2-design.md` 步骤 0 | 只需技术栈选型时入口 |
| `UI` / `视觉` / `美学` / `theme` / `design system` / `design tokens` | `prompts/2a-ui-design.md` | 前端项目，用户可见 UI |
| `换调性` / `改风格` / `换风格` / `redesign` / `restyle` / `重做视觉` / `换皮` | `prompts/L-restyle.md` | 已有项目换视觉，保留功能 |
| `健康检查` / `health` / `体检` / `技术债扫描` / `巡检` / `brooks-health` / `brooks-sweep` / `brooks-audit` / `brooks-debt` / `扫冗余` / `找死代码` / `找重复` / `清未用导出` / `清未用依赖` / `dedupe` / `dead code` | `prompts/M-health.md` | 代码库周期性巡检 + 冗余扫描（步骤 2.5），不属任何 change |
| `扫描代码` / `scan` / `intel` / `入场扫描` / `给项目体检` / `老项目首次访问` | `prompts/I-intel-scan.md` | 生成 / 更新 `CONTEXT.md`，brownfield 项目首次使用必跑 |
| `同步架构` / `沉淀架构` / `evolve` / `架构演进` / `同步 CONTEXT` / `整理沉淀` | `prompts/A-evolve.md` | 扫近期归档 change 的 DESIGN § 9，批量 review + patch CONTEXT.md / ARCHITECTURE.md（不属任何 change）|
| `建立架构` / `架构梳理` / `重构架构` / `architect` / `重审 ADR` / `画架构图` | `prompts/A-architect.md` | 首次 / 重构时建立 `ARCHITECTURE.md`。项目级 ADR / 模块图 / 跨模块契约（不属任何 change）|
| `需求` / `spec` / `requirement` | `prompts/1-requirement.md` | |
| 任何**新事物描述**（"做 / 想 / 加 / 实现 / 设计 + X"，且当前无活跃 change） | `prompts/0-change.md` | **自动生成 change-id**，不要问用户要 |
| 模糊不清 | 反问用户：「你想做的是新需求 / 继续上次 / 别的吗？」 | 一句话定位 |

> **架构级变更的二次拦截**：路由表只看关键词，但 0-change / 2-design 进去后会**用判断**做二次检测——如果用户描述涉及「拆模块 / 换数据库 / 换鉴权方案 / 改公共契约 / 容量边界 / 跨服务编排」这类项目级变更，会停下来反问"是否先跑 A-architect"。判定细则见 `@flow-kit/prompts/0-change.md` 步骤 0.4 / `@flow-kit/prompts/2-design.md` 步骤 0₋。
>
> **AI 的额外职责**：路由到 0-change / 2-design 时，路由声明的「第一动作」一栏应明确写"按 0.4 / 0₋ 先做架构级预检"，避免遗漏。

## 第三步 · 老项目入场检测（brownfield 必跑）

**触发**：进入 0-change 之前必跑。

### 3.1 探测 AI 上下文文档

按下面顺序 `ls` / `find` 探测：

| 文档 | 路径 | 来自哪个生态 |
|---|---|---|
| `CONTEXT.md` | 仓库根 / `.specs/` | flow-kit 自己 |
| `AGENTS.md` | 仓库根 | OpenAI Codex / 标准 agents 协议 |
| `CLAUDE.md` | 仓库根 / `.claude/` | Anthropic Claude Code |
| `.cursor/rules/*.md` | `.cursor/` | Cursor IDE |
| `.windsurf/rules/*.md` | `.windsurf/` | Windsurf IDE |
| `.github/copilot-instructions.md` | `.github/` | GitHub Copilot |
| `.clinerules` | 仓库根 | Cline |

读 `STATE.md` 的 `ai_context_doc`（用户上次指定的替代文档）和 `last_intel_scan` 字段。

### 3.2 判决

#### 情况 A · `STATE.md` 已设 `ai_context_doc`

用户上次明确指定了某文档为 AI 遵守依据。**直接读它**，不再询问。

后续阶段（2-design / 4-dev）改读 `<ai_context_doc>` 替代 CONTEXT.md。

#### 情况 B · 已存在 CONTEXT.md 且 `last_intel_scan` 在 90 天内

**直接读 CONTEXT.md**，跳过本步。无需打扰用户。

#### 情况 C · 已存在 CONTEXT.md 但超过 90 天

读 CONTEXT.md，**提醒用户**："上次扫描已 X 天，可重跑 intel-scan"，但**不强制**。

#### 情况 D · 未发现 CONTEXT.md，但有其他 AI 上下文文档（AGENTS / CLAUDE / Cursor / 等）

**反问用户**（**必须**等待回复才能继续）：

```
🔍 检测到本项目已有以下 AI 上下文文档：
  - <列出找到的，含路径与文件大小>

flow-kit 默认用 CONTEXT.md 作为单一源。请选择：
  1. 跑 intel-scan，综合现有文档 + 代码扫描，生成 CONTEXT.md（推荐）
  2. 以现有文档为准（告诉我哪个），跳过 intel-scan
  3. 跳过 intel-scan + 不读现有文档（不推荐 · AI 会盲飞）

请选 1/2/3。无人工确认前我不进 0-change（避免 AI 在不知项目约定下开始动手）。
```

- **选 1**：进 `prompts/I-intel-scan.md` 走分支 A
- **选 2**：在 `STATE.md` 写 `ai_context_doc: <用户指定路径>`，回到原意图（直接进 0-change）
- **选 3**：在 `STATE.md` 写 `ai_context_doc: none / skip`，回到原意图（带警告）

#### 情况 E · 未发现任何 AI 上下文文档（CONTEXT / AGENTS / CLAUDE / Cursor / Windsurf / Copilot / Cline 全无）

**反问用户**（**必须**等待回复才能继续）：

```
🔍 项目里未发现任何 AI 上下文文档。

flow-kit 后续阶段需要项目上下文给 AI 用。请选择：
  1. 现在跑入场扫描，自动生成 CONTEXT.md（~15-30k tokens · 仅首次 · 推荐）
  2. 我手动指定项目里某个文档作为开发遵守依据：<请回复路径>
  3. 跳过 intel-scan，直接进 0-change（不推荐 · AI 会"盲飞"，老项目护栏 B1-B5 全失效）

请选 1/2/3。无人工确认前我不开始扫描或进入 0-change（避免无意义消耗 token）。
```

- **选 1**：进 `prompts/I-intel-scan.md` 走分支 C
- **选 2**：在 `STATE.md` 写 `ai_context_doc: <路径>`，回到原意图
- **选 3**：在 `STATE.md` 写 `ai_context_doc: none`，回到原意图（带警告）

#### 情况 F · 是刚创新项目（无 `package.json` / `pyproject.toml` / 等代码上下文）

跳过本步。这是 greenfield 项目，CONTEXT.md 会在 0-change / 1-requirement / 2-design 过程中逐步沉淀。

### 3.3 为什么这步重要

跳过会导致：
- AI 不知项目架构 → 写出不合项目风格的代码
- AI 重复实现已有抽象 → 费 token 且产生重复代码
- 4-dev 1.7 schema 任务 / 1.8 破坏性变更检测都会不准
- 用户明明有 CLAUDE.md 写好的约定 → AI 完全不读 → 抱怨"为什么不按我说的来"

## 第四步 · 自动准备（不打扰用户）

进入对应阶段前，AI 必须自行完成：

- **新 CHANGE**：按 `prompts/0-change.md` 的步骤 0 自动生成 `change-id`（kebab-case，2~4 词），并在第一条回复里显式声明
- **目录不存在**：自行 `mkdir -p .specs/<id>/`，不要让用户先建
- **规则加载**：若 IDE 未注入全局规则，读 `@flow-kit/RULES.md`（精简版 `@flow-kit/SYSTEM.md` 也行）
- **检测外部扩展**（阶段 4/5/6/M 需要）：进入阶段前检查是否装了以下并在路由声明里表明走「外部路径」还是「内置回退」：
  - [`brooks-lint`](https://github.com/hyhmrright/brooks-lint)：4-dev self-review / 5-test 测试质量 / 6-review 代码质量 / M-health 巡检都会优先用
  - [`ui-ux-pro-max`](https://uupm.cc) / [`impeccable`](https://impeccable.style)：2a-ui-design / 4-dev UI 任务会优先用

### 加载工件（严格区分 必读 / 按需）

**语义约定**：
- `⚡︎ 全读` ：进阶段首轮必须 read_file 整个文件（只出现在 SPEC / TEMPLATE 上）
- `⚡︎ 查表` ：只 grep 指定节 或 read offset/limit，**禁止默认整读**（reference/* 都是这个）
- `⚡︎ 按需` ：首轮不读，里面某个决定点需要时才 grep / 读

| 阶段 | 全读（SPEC） | 查表（REFERENCE，只读指定节） | 按需 |
|---|---|---|---|
| 0 / 1 | —（新建）| `flow-kit/reference/ui-aesthetics.md` 只查「给 AI 在 0-change 阶段展示用的标准模板」一节（仅前端项目）| — |
| 2 | `<id>/CHANGE.md` + `<id>/REQUIREMENT.md` + `.specs/CONTEXT.md` + `.specs/ARCHITECTURE.md`（如存在 · brownfield 强烈推荐 · 重点读 § 2/§ 3/§ 4）| `flow-kit/reference/tech-stacks.md` 只查「适用矩阵」+ 过滤出的 5~6 张卡片 | ADR 阶段某项要深谈时再读 |
| 2a | `<id>/CHANGE.md` + `<id>/REQUIREMENT.md` + `<id>/DESIGN.md` `## 0` 段 + `.specs/CONTEXT.md` + `flow-kit/reference/ui-anti-patterns.md`（仅 75 行可全读）| `flow-kit/reference/ui-aesthetics.md` 查「5 维度」+ 「给 AI 的模板」 | uipro / impeccable 查询（装了才调）|
| 3 | `<id>/REQUIREMENT.md` + `<id>/DESIGN.md` + `<id>/UI-DESIGN.md`（前端项目）+ `.specs/CONTEXT.md` | — | 任务模板查询 |
| 4 | `<id>/TASK.md`（只读当前 task 块）+ `<id>/DESIGN.md` `## 0` 段 + `<id>/UI-DESIGN.md`（UI 任务）+ `.specs/CONTEXT.md` + `.specs/LESSONS.md` | `flow-kit/reference/ui-anti-patterns.md`（UI 任务 · 75 行可全读）| — |
| 5 | `<id>/REQUIREMENT.md` + `<id>/DESIGN.md` `## 0` 段 + `<id>/TASK.md` + 各 `*-SUMMARY.md` | `flow-kit/reference/test-pyramid.md` 只查「适用矩阵」+ 需要的那几轮详情 | — |
| 6 | `<id>/REQUIREMENT.md` + `<id>/DESIGN.md` + `<id>/TASK.md` + `<id>/TEST.md` + `git diff` | `flow-kit/reference/ui-anti-patterns.md`（前端项目第三轮 · 75 行可全读）| — |
| 7 | `.specs/<id>/` 全部产物 + `.specs/LESSONS.md` | — | — |
| **M** (health) | `.specs/CONTEXT.md` + `.specs/LESSONS.md` + 最近 1 份 `.specs/health/*.md`（如有，做对比基线）| — | 抽样 5 个最近改动频繁的 src/ 模块 + 5 个测试文件 + 最近 30 天 git log |
| **A** (evolve) | `.specs/STATE.md` + `.specs/CONTEXT.md` + `.specs/ARCHITECTURE.md`（如存在）+ 范围内每个 `.specs/archive/<change>/DESIGN.md` 的 § 9 段（仅 § 9，非整份 DESIGN）| — | 仅扫 `last_evolve_at` 之后归档的 change，禁止越界读 § 9 以外的 DESIGN 内容 |
| **A** (architect) | `.specs/CONTEXT.md` + `.specs/ARCHITECTURE.md`（如存在）+ `.specs/CHANGELOG.md` + `flow-kit/templates/ARCHITECTURE.md`（模板）| — | `src/` 顶层结构 + `package.json` / 依赖文件 + 抽样几份 `.specs/archive/*/DESIGN.md` |

### 查 reference 某一节的实际动作示例

```
# ± 查「适用矩阵」那一节的起始行
grep_search Query="适用矩阵" SearchPath="flow-kit/reference/tech-stacks.md"
# 取到 line 380 左右为起始，再：
read_file path="flow-kit/reference/tech-stacks.md" offset=380 limit=60
```

不要「为了保险」一上来就整读。不仅费 token，还让你在后面的推理中被无关节况干扰。

## 第五步 · 显式声明执行计划（必须）

进入实际工作前，AI 必须输出一段**路由声明**，**每个加载项必须标明起止行或「全读」**：

```
✅ 路由：<阶段，例如 0-change>
✅ Change-ID：<id>（已自动生成 / 已恢复活跃 change：<existing-id>）
✅ 已加载：
   - <file1>（全读，N 行）
   - <file2>（全读，N 行）
   - <reference-file>（仅查「某节」，line X-Y）
✅ 未加载：<本阶段不需但后面可能用到的 reference，明说何时才拉>
✅ 第一动作：<具体下一步，例如 "按 0-change 流程反问澄清，先问 3 个问题"）
```

示例（2-design 阶段首轮）：

```
✅ 路由：2-design
✅ Change-ID：companion-platform
✅ 已加载：
   - .specs/companion-platform/CHANGE.md（全读，52 行）
   - .specs/companion-platform/REQUIREMENT.md（全读，98 行）
   - .specs/CONTEXT.md（全读，41 行）
   - flow-kit/reference/tech-stacks.md（仅查「适用矩阵」，line 380-405，提取出 5 张候选卡）
✅ 未加载：ui-aesthetics.md / test-pyramid.md / ui-anti-patterns.md（后面阶段才需）
✅ 第一动作：按 2-design 步骤 0，列 5 张技术栈卡片 + 推荐 + 排除，等用户选定后才出 ADR。
```

用户看到这段后可以一句话纠偏（"换 id"、"我想要的是别的阶段"、"你加载太多了"），AI 必须接受。

## 第六步 · 执行对应阶段 prompt

加载 `prompts/<n>-*.md` 的内容并按其指令推进。所有规则（`RULES.md` / `SYSTEM.md`）继续生效。

---

## 极少数情况：用户根本没说他想做什么

例：用户只发了 `@flow-kit/GO.md`，正文空。

→ AI 必须主动反问，给出 3 个最可能的选项让用户选：
1. 我有个新想法想做
2. 继续上次的工作（如果 STATE 有活跃 change，主动列出）
3. 我要审查/测试某段已有代码

不要瞎猜路由。

---

## 自检（产出路由声明前）

- [ ] 已读 STATE.md（如果存在）
- [ ] 已按表格匹配意图，没有跳过
- [ ] 新 CHANGE 已自动生成 ID 并展示
- [ ] **Token 预算**：本轮加载的 reference/* 总行数 ≤ 150（全读仅 ui-anti-patterns 75 行 · 其他均查节）
- [ ] **未越界**：没有读上表「查表」或「按需」列中的文件为全文
- [ ] 路由声明含「已加载 / 未加载 / 起止行」三要素
- [ ] 没有要求用户提供 ID / 路径 / 阶段名（这些 AI 自己决定）
