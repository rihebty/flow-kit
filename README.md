# flow-kit

一套攒起来的 AI 编程流程文档。我自己平时写项目用的那套方法抽成可复用的 markdown 包。

里面的思路不是凭空想的，主要是把最近折腾过的几个开源项目——bmad、spec-kit、OpenSpec、GSD、claude-task-master、superpowers、gstack、skills——里面我觉得有用的部分挑出来，再按自己的工作流重排一遍。各自的品牌和 CLI 依赖都剥掉了，只留 markdown。

装一次，后面不跟任何上游版本绑定。

### 2026.5.13 合并提交：添加Artifact预检门
### 2026.5.6 下午15点，运行过程发现AI会跳过一些流程，请注意

## 为什么是纯 markdown

写多了项目会发现：一个工具只要"装"了，就一定会有版本问题、依赖冲突、哪天突然 breaking change。flow-kit 故意反过来——它不是工具，是一组文件。clone 到项目根目录就用，没有运行时。

- 不用 `npm install`、不用 `pip`、不用 CLI
- Windsurf / Claude Code / Cursor / Copilot / Codex / Gemini / Cline 都能跑，只要 AI IDE 支持 `@` 引用文件
- 所有规则都在文件里，想改哪条就改哪个 md

坏处是手动操作多了点，好处是不会某天被工具更新坑到。

---

## 安装（30 秒）

把整个 `flow-kit/` 文件夹复制到你的新项目根目录：

```text
your-project/
├── flow-kit/          ← 这里
│   ├── METHODOLOGY.md
│   ├── RULES.md
│   ├── prompts/
│   └── templates/
├── src/
└── ...
```

**就这样**。下面看怎么调用。

---

## 在不同 AI 工具里的调用方式

### 通用方式（任何 AI 都行）

会话开头让 AI 读 3 个文件作为基线：

```
请先读取并严格遵守：
@flow-kit/METHODOLOGY.md
@flow-kit/RULES.md
@flow-kit/prompts/0-change.md

然后我们开始一次 CHANGE 流程。我要做的是：<一句话需求>
```

之后每进一个阶段，再追加引用对应的 `prompts/<n>-*.md`。

### Windsurf

把 `prompts/*.md` 复制成 workflow：

```bash
# Windows PowerShell
mkdir -Force .windsurf\workflows
copy flow-kit\prompts\*.md .windsurf\workflows\
```

之后用 `/0-change`、`/1-requirement` 等斜杠命令直接调用。

> 也可以把 `RULES.md` 内容复制到 `.windsurfrules`，作为常驻系统规则。

### Claude Code

把每个 prompt 包成 skill：

```bash
mkdir -p .claude/skills/flow-change
cp flow-kit/prompts/0-change.md .claude/skills/flow-change/SKILL.md
# 其他阶段同理
```

或者最简单——直接在会话里 `@flow-kit/...` 引用。

### Cursor

把 `RULES.md` 复制为 `.cursorrules`，prompts 用 `@` 引用即可。

### Copilot / Codex / Gemini / Cline / 其他

都支持 `@` 引用文件，直接按"通用方式"那段操作。

---

## 降低输入摩擦（每次 @ 三行太烦怎么办）

### 终极极简（每个 IDE 都能用，1 个 @，不写 ID）

**任何 IDE、任何 AI、任何会话**，都只需要：

```
@flow-kit/GO.md

设计陪诊网站
```

或：

```
@flow-kit/GO.md

继续
```

或：

```
@flow-kit/GO.md

执行 T03
```

`GO.md` 是**统一入口**：
- 自动判断你想做的是哪个阶段（新需求 / 继续 / 执行任务 / 审查 / 测试 / 集成）
- 自动生成 `change-id`（你不用想名字）
- 自动加载该阶段需要的所有工件 `.md`
- 自动主动反问澄清

**对每个 IDE 都生效**——`@文件` 是所有 AI 工具的最低公分母。

> 配合下面的方式 B（全局注入 `SYSTEM.md`），AI 在路由的同时也守规则。这是最干净的组合。

> 如果你的 IDE 支持斜杠命令（Windsurf / Continue），还可以再把 `GO.md` 复制为 `/go` 命令，连那 1 个 `@` 都省掉：
> ```powershell
> mkdir -Force .windsurf\workflows
> copy flow-kit\GO.md .windsurf\workflows\go.md
> copy flow-kit\prompts\*.md .windsurf\workflows\
> ```
> 之后输入 `/go 设计陪诊网站` 即可。

---

下面三种是更细粒度的传统方式，如果你想直接控制每一步、不走自动路由可看：

### 方式 A · 斜杠命令（零 @，Windsurf 用户首选）

```powershell
mkdir -Force .windsurf\workflows
copy flow-kit\prompts\*.md .windsurf\workflows\
```

之后会话里输入：

```
/0-change

我要做的是：设计陪诊网站。change-id: init-platform。
```

完全不用 `@`。`/1-requirement`、`/2-design` ... 一路下来都这样。

### 方式 B · 全局规则注入（每次少 1~2 个 @）

把 `flow-kit/SYSTEM.md` 的内容**复制一次**到 IDE 全局规则文件，永久注入：

| IDE | 复制到哪 |
|---|---|
| Windsurf | 项目根 `.windsurfrules` 或全局 rules |
| Cursor | 项目根 `.cursorrules` |
| Claude Code | 项目根 `CLAUDE.md` 或 `~/.claude/CLAUDE.md`（全局） |
| Aider | `.aider.conf.yml` 加 `read: [flow-kit/SYSTEM.md]` |
| Cline / Continue | 各自规则文件 |

注入后每次会话只需：

```
@flow-kit/prompts/0-change.md

我要做的是：设计陪诊网站。change-id: init-platform。
```

只剩 **1 个 @**。

> `SYSTEM.md` 是 `METHODOLOGY` + `RULES` 的精简合并版，专门为永久注入设计。完整版在 `METHODOLOGY.md`，需要时再 `@` 即可。

### 方式 C · 复制粘贴开场白小抄

不想配置任何东西、就想 ctrl+c ctrl+v？把下面这些保存到你的笔记软件，每次直接拷：

**0-change**：
```
@flow-kit/prompts/0-change.md
我要做的是：<一句话>。change-id: <id>。先反问澄清。
```

**1-requirement**：
```
@flow-kit/prompts/1-requirement.md @.specs/<id>/CHANGE.md
按 1-requirement 流程出 REQUIREMENT.md 和 .specs/CONTEXT.md。
```

**2-design**：
```
@flow-kit/prompts/2-design.md @.specs/<id>/REQUIREMENT.md @.specs/CONTEXT.md
按 2-design 流程出 DESIGN.md。
```

**3-task**：
```
@flow-kit/prompts/3-task.md @.specs/<id>/REQUIREMENT.md @.specs/<id>/DESIGN.md @.specs/CONTEXT.md
按 3-task 流程出 TASK.md。
```

**4-dev**（每任务一次）：
```
@flow-kit/prompts/4-dev.md @.specs/<id>/TASK.md @.specs/CONTEXT.md @.specs/LESSONS.md
执行 task <T01>。
```

**5-test / 6-review / 7-integration** 类似，引用对应 prompt + 已有产物即可。

> 配合**方式 B**，可以把每条开头的 `@flow-kit/prompts/...` 之前的部分省掉。

### 三种方式的取舍

| 方式 | 一次性配置成本 | 每次摩擦 | 适合谁 |
|---|---|---|---|
| A 斜杠命令 | 一条 copy 命令 | 零 @ | Windsurf 用户、能配 workflow 的工具 |
| B 全局注入 | 一次性复制 SYSTEM.md | 1 个 @ | 任何 IDE，最通用 |
| C 复制粘贴 | 0 | 1~3 个 @ | 想保持灵活、跨多个 AI 工具切换 |

> 推荐：**A + B 一起上**——配了斜杠命令仍然把 SYSTEM.md 注入全局规则，斜杠命令不工作时回退到 `@` 也少一个文件。

---

## 怎么用——两条路径

### 路径 A：从 0 开发新项目

按顺序跑这 8 个 prompt（中间允许多轮对话）：

```
0-change      → 1-requirement → 2-design → 3-task
   → 4-dev (逐任务循环)
   → 5-test → 6-review → 7-integration
```

每个阶段产出一个 `.md` 工件，存到 `./.specs/<change-id>/`。

### 路径 B：给已有项目加功能（CHANGE 驱动）

最少路径不是跳过产物，而是把需求 / 设计压缩成轻量工件后继续：

```
0-change
  → requirement/design preflight（缺 REQUIREMENT.md 或 DESIGN.md 就补轻量版）
  → 3-task
  → 4-dev
  → 5-test
  → 6-review
  → 7-integration
```

`1-requirement` 与 `2-design` 可以很短，但不能缺。`3-task` 需要 `REQUIREMENT.md` 和 `DESIGN.md` 来确定验收、技术栈和 `write_files` 边界。

### MVP 模式（当天跑通）

用 4 个轻量产物跑通：

```
1-requirement (含 CONTEXT)  →  DESIGN-lite  →  3-task  →  4-dev (含 self-verify)
```

产物：`REQUIREMENT.md` + `DESIGN.md`（可很短，但必须含技术栈 / 边界）+ `TASK.md` + `SUMMARY.md`。
跑顺了再升级到完整版。

---

## 文件作用速查

> 「能否单独用」的判定标准：**抛开整套流程，只把这一个文件丢给 AI 或当模板用，是否仍能产出有价值的结果**。

### 核心文档（3 个）

| 文件 | 作用 | 能否单独用 | 单独用场景 |
|---|---|---|---|
| `METHODOLOGY.md` | 方法论骨架（流程图 + 阶段定义 + 文件体系 + 7 个核心机制） | ✅ 可单独读 | 想理解整套思路；新成员入门；做技术分享 |
| `RULES.md` | 系统级硬规则（R1 token、R2 阶段门、R3 角色红线、R4 提交、R5 测试、R6 反幻觉、R7 范围、R8 语言） | ✅ 可单独注入 | 直接复制进 `.cursorrules` / `.windsurfrules` / 系统提示，给任何 AI 用都立刻提升纪律 |
| `README.md`（本文件） | 安装方式 + 调用方式 + 升级路径 + 设计原则 | ✅ 可单独读 | 装到新项目时先看；判断要不要采用 |

### 阶段 prompts（9 个，按顺序）

| 文件 | 作用 | 能否单独用 | 单独用场景 |
|---|---|---|---|
| `prompts/0-change.md` | 把模糊想法**反问澄清**成变更提案（自动生成 change-id）| ✅ 完全独立 | 任何"我想做点什么但讲不清"的时刻；产品经理整理需求；自己想清一件事 |
| `prompts/1-requirement.md` | 把变更提案变成可执行需求 + 验收准则 + 域语言 | ✅ 完全独立 | 写需求文档；把口头需求变成 PRD；提取项目术语表 |
| `prompts/2-design.md` | 把需求变成技术设计 + ADR + 风险 | ✅ 完全独立 | 架构评审；技术选型对比；写设计文档 |
| `prompts/2a-ui-design.md` | UI 美学方向决策 + design tokens + 反 AI-slop 自检（仅前端项目）| ✅ 完全独立 | 任何前端项目开工前；redesign；从设计稿提取 design system |
| `prompts/3-task.md` | 把设计拆成可并行的原子任务 + verify | ✅ 完全独立 | 工作分解；冲刺规划；把一个大功能拆给团队 |
| `prompts/4-dev.md` | 在 fresh context 中执行**单个任务**（含 TDD + 提交 + 中途断点恢复 + UI anti-pattern 扫描）| ⚠️ **半独立** | 严格依赖 `TASK.md` 中的 `verify` 字段。单点调用时可以由用户显式提供一份"临时最小 TASK"，但它必须包含 `id / name / read_files / write_files / action / verify / done`，AI 不允许自己编造来绕过 `3-task` |
| `prompts/5-test.md` | 从 AC 派生测试矩阵 + UAT 脚本 | ✅ 完全独立 | 给已有功能补测试；写 QA 脚本；做覆盖率审计 |
| `prompts/6-review.md` | 双 / 三轮审查（spec 合规 + 代码质量 + UI 视觉 + 跨模型 spot-check）| ✅ 完全独立 | 给一段 diff 让 AI 评审；merge 前自查；做安全审计；UI 视觉 audit |
| `prompts/7-integration.md` | UAT 引导 + 失败诊断 + LESSONS 提名 + 归档 | ⚠️ **半独立** | UAT 引导部分可单跑；归档/提名部分依赖 `.specs/<id>/` 已有产物 |

### 项目级文档三层（跨 change 常驻）

`.specs/` 下有三层项目级文档，职责不重叠：

| 文件 | 职责 | 维护者 | 加载频率 | 大小 |
|---|---|---|---|---|
| `.specs/CONTEXT.md` | **rules 层**。技术栈版本 / 命名约定 / 既有抽象索引 / 禁动清单 / code-style。**AI 实施时读** | `I-intel-scan` 首创 + `A-evolve` 增量 | 每个 change 的 2-design / 3-task / 4-dev 都加载 | 50-200 行 |
| `.specs/ARCHITECTURE.md` | **structure 层**。项目模块图 / 依赖规则 / ADR 列表 / 跨模块契约 / 扩展点 / 容量边界。**人读 + 2-design 读** | `A-architect` 首创/重构 + `A-evolve` 增量 append ADR | 仅 brownfield 项目的 2-design 阶段加载 | 200-600 行 |
| `.specs/<change-id>/DESIGN.md` | **change 层**。本次 change 的技术栈选定 / 架构对齐 / ADR / 风险 / § 9 架构沉淀建议。**归档后冻结** | `2-design` 写，`A-evolve` 只读 § 9 | 仅本 change 的后续阶段加载 | 100-400 行 |

**三者如何联动**：

```
change-001 完成 → DESIGN.md § 9 填「新增抽象」、「项目级决策」
             ↓×N个 change
A-evolve 跑 → 扫各 change 的 § 9 → 用户逐项 review → patch CONTEXT.md + ARCHITECTURE.md
             ↓
CONTEXT 从 50 行 增到 200+ / ADR 冲突 ≥ 5 → 建议跑 A-architect 重审
```

**三层从哪里来：**

- 受 [BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD) 的 `project-context.md`（rules）+ `architecture.md`（structure）分层启发
- 但 BMAD 靠 `bmad-correct-course` 在 sprint 中修订，flow-kit 靠 `A-evolve` 在多个 change 后批量同步 —— 后者更贴合 change-driven 哲学

### 横向命令（L / M / I / A / X 系列 · lateral / maintenance / intel / architecture / explore）

不在主流程里，按需调用。前缀区分作用：**`L-`** 生命周期；**`M-`** 维护巡检；**`I-`** 项目情报；**`A-`** 架构演进；**`X-`** 探索思考。

| 文件 | 作用 | 能否单独用 | 单独用场景 |
|---|---|---|---|
| `prompts/X-explore.md` | **探索模式（思考伙伴）**：不写代码、不写方案，只做探索 / 可视化 / 对比 / Bug 排查。探索结束后建议可能的下一步（继续深挖 / 开 change / 回到当前阶段等），不强制跳转 | ✅ 完全独立 | 模糊想法需要理清；技术方案对比；代码库调研；Bug 症状不明需排查；需求探索 |
| `prompts/L-restyle.md` | **一键换调性**：保留功能不变，只换视觉。自动识别现有调性 → 引导选新调性 → 生成 UI-DESIGN v2 + 任务波次 + 风险通告 | ✅ 完全独立 | 已有产品视觉陈旧；品牌换新；做暗色版 / 高端版并行设计系统 |
| `prompts/M-health.md` | **代码库周期性巡检**：brooks-lint 6+6 维衰退诊断 / 架构图 / 技术债优先级 + **步骤 2.5 冗余扫描**（jscpd / knip / vulture / staticcheck · 字面重复块 / 死代码 / 未用导出 / 未用依赖） | ✅ 完全独立 | 月度 / 季度体检；里程碑前估价；接手陌生项目首周；单跑"扫冗余 / 找死代码 / 清未用导出" |
| `prompts/I-intel-scan.md` | **老项目入场扫描**：检测 AI 上下文文档 + 扫代码生成 / 更新 `.specs/CONTEXT.md` | ✅ 完全独立 | brownfield 项目首次使用 flow-kit 必跑；老项目架构偏移后重扫 |
| `prompts/A-architect.md` | **项目级架构梳理**：建立 / 重构 `ARCHITECTURE.md`，含模块图 + 依赖规则 + ADR 列表 + 跨模块契约 + 容量边界 | ✅ 完全独立 | 项目里程碑后架构梳理；接手陌生项目后结构化架构理解；ADR 定期重审 |
| `prompts/A-evolve.md` | **架构增量同步**：扫近期归档 change 的 DESIGN § 9，逐项 review 后 patch CONTEXT.md / ARCHITECTURE.md | ✅ 完全独立 | 每月 / 每季 批量同步一次；里程碑发布后凝固架构决策 |

> 未来还可能新增 `L-prune-lessons` / `L-archive-old` / `L-merge-changes` 等，思路一致：**横向、不属于线性流程、按需调用**。

### 工件模板（11 个）

| 文件 | 作用 | 能否单独用 | 单独用场景 |
|---|---|---|---|
| `templates/CHANGE.md` | 一次变更的提案 | ✅ 独立 | 任何项目里写「变更申请」「功能提案」 |
| `templates/REQUIREMENT.md` | 需求 + 验收准则（Given/When/Then）+ v1·v2·out | ✅ 独立 | 任何项目的需求文档模板，与流程无关 |
| `templates/CONTEXT.md` | 项目级共享上下文 · **rules 层**（术语表 + 已锁决策 + 默认偏好 + 既有抽象索引 + 禁动清单） | ✅ 独立，**强烈推荐** | 任何项目都该有一份；放一份就能让所有 AI 助手输出更稳更短 |
| `templates/ARCHITECTURE.md` | 项目级架构文档 · **structure 层**（模块图 + 依赖规则 + ADR 列表 + 跨模块契约 + 扩展点 + 容量边界） | ✅ 独立，**中大型项目推荐** | 任何需要跨模块架构记录的项目；ADR 中心化管理；与 `prompts/A-architect.md` + `A-evolve.md` 配套使用 |
| `templates/DESIGN.md` | change 级技术设计 + ADR + 风险 + § 9 架构沉淀建议（供 A-evolve 后续同步） | ✅ 独立 | 任何架构 / 技术方案文档 |
| `templates/UI-DESIGN.md` | UI 美学方向 + design tokens（OKLCH / 字体 / 间距 / 动效）+ 反 AI-slop 自检 | ✅ 独立，**强烈推荐前端项目用** | 任何前端项目的视觉规约；提取已有项目的 design system；redesign 起点 |
| `templates/TASK.md` | 任务清单（XML + 波次 + verify + done） | ✅ 独立 | 任何工作分解；可直接当 todo 看板用 |
| `templates/TEST.md` | 测试矩阵 + UAT 脚本 + 覆盖率回顾 | ✅ 独立 | 任何测试计划 / QA 文档 |
| `templates/REVIEW.md` | 双轮审查报告（spec 合规 + 代码质量 + 跨模型分歧） | ✅ 独立 | 任何 code review 报告模板 |
| `templates/SUMMARY.md` | 任务级完成报告（做了什么 + verify 输出 + 决策偏离） | ✅ 独立 | 写日报 / 完成回执 / 任务汇报 |
| `templates/PROGRESS.md` | **临时**——任务中途清窗的快照（已排除方案是核心） | ⚠️ **半独立** | 强绑定 R1.5 重启协议；脱离 4-dev 用价值不大 |
| `templates/LESSONS.md` | **项目级常驻**——跨任务失败知识库 | ✅ 独立，**强烈推荐** | 任何项目都该有；不需要跑流程，只要每次踩坑后追加一条就值 |
| `templates/STATE.md` | 跨会话项目状态（活跃 change / 中断任务 / 决策日志） | ⚠️ **半独立** | 只在用了多阶段流程后才有用；纯单文件项目不必要 |

### 参考资料（4 个）

| 文件 | 作用 | 能否单独用 | 单独用场景 |
|---|---|---|---|
| `reference/tech-stacks.md` | 技术栈卡片—— 8 个主流前后端组合 + 适用矩阵 + 选型决策模板 | ✅ 独立 | 任何项目开工前的技术选型；给团队讲「为什么选这个栈」；评估现有项目要不要迁移 |
| `reference/ui-aesthetics.md` | UI 美学决策框架（融合 Anthropic frontend-design + impeccable）—— 4 个问题 + 5 维度 + 9 张调性卡片 | ✅ 独立 | 任何前端项目开工前的"该选什么调性"思考；从设计稿反推 design system |
| `reference/ui-anti-patterns.md` | 反 AI-slop 清单（grep 用）—— 字体 / 颜色 / 阴影 / 边框 / 动效 / 布局 / 文案 / 组件 8 类禁忌 | ✅ 独立，**强烈推荐前端项目用** | code review checklist；前端 PR self-review；`grep` 一次少踩 80% AI-slop 坑 |
| `reference/test-pyramid.md` | 5 轮测试金字塔（功能 / 性能 / 安全 / 兼容 / 可观测）的工具 / 标准 / 反模式 / 适用矩阵 | ✅ 独立 | 给任何项目立测试规约；写 QA checklist；做发布前自查 |

### 可选外部扩展（按需安装，flow-kit 自动检测并优先使用）

| 扩展 | 维度 | 角色 | flow-kit 何处优先调用 | 何时受益 |
|---|---|---|---|---|
| [`brooks-lint`](https://github.com/hyhmrright/brooks-lint) | **代码质量** | 12 本经典工程书籍驱动的 6 维代码 + 6 维测试衰退诊断；带书页引用 / 严重度 / 4 要素结构化输出 | `4-dev` self-review · `5-test` 测试质量自检 · `6-review` 代码质量轮 · `M-health` 巡检 | 团队想要带书本引用的 review 报告；想周期性体检代码库；想有 Pain × Spread 优先级的技术债清单 |
| `jscpd` + `knip` / `ts-prune` / `depcheck`（TS/JS）· `vulture` / `deptry`（Python）· `staticcheck` / `deadcode`（Go）· `cargo udeps` / `clippy`（Rust）| **字面冗余 + 死代码** | 跨语言字面重复块检测 + 未用导出 / 未用依赖 / 死代码扫描 | `M-health` 步骤 2.5（字面级冗余巡检） | 想找出重复代码块抽公共函数；清理未用的导出 / 依赖 / 死分支；接手老项目做"大扫除" |
| [`ui-ux-pro-max`](https://uupm.cc) | **UI 广度** | 67 styles / 161 palettes / 57 fonts / 25 charts / 99 UX rules / 15+ stacks 的查询数据库 | `2a-ui-design` 字体 / 颜色 / 图表选择 | 想从更大候选池里挑（不只 9 调性）；想要栈级（React/Vue/Next）建议；做数据可视化想选合适图表 |
| [`impeccable`](https://impeccable.style) | **UI 深度** | 23 个细粒度 UI 命令 + 自动检测 CLI | `2a-ui-design` design tokens · `4-dev` UI 任务自查 · `6-review` 第三轮视觉审 | 想把 design system 抠到组件级；想在 review 阶段做更严格的视觉 audit |

flow-kit 与四类扩展的关系：

- **flow-kit 编排流程**：定义在哪个阶段做哪类决策（0-change 选调性 / 2-design 选栈 / 2a 深化视觉 / 4-dev 落地 / 5-test 测试 / 6-review 审 / M-health 巡检）
- **brooks-lint 加深「代码质量」一面**：把 review / dev self-check / test 质量从"凭感觉"提升为带书页引用的结构化诊断（概念级：R3 知识重复等 6 维衰退）
- **jscpd / knip / vulture / staticcheck 加深「字面冗余」一面**：brooks-lint R3 覆盖不到的字面级重复代码块 / 死代码 / 未用导出 / 未用依赖，由它们补齐
- **uipro 加宽「UI 决策」一面**：从更大数据库里查调性 / 调色板 / 字体 / 图表
- **impeccable 加深「UI 落地」一面**：组件级细抠 / 视觉 audit / 修复建议

**都没装 flow-kit 也能跑**——内置 `reference/*` 作为基线，6 维代码诊断 / 6 维测试诊断 / 9 调性 / 反 AI-slop 清单 / 冗余 grep fallback 都有内置回退。装上后**质量明显提升**（按 brooks-lint benchmark：带书本引用的代码 review 发现率 100% vs 不带 16%；字面冗余扫描 jscpd 的 recall 远高于 AI grep fallback）。

---

### 可选运行时门禁：Forge

flow-kit 默认仍然是纯 markdown、无运行时。如果你在 Claude Code 里经常遇到 AI 跳过阶段、漏写产物、漏测试或漏 review，可以额外接入 Forge 作为可选 runtime adapter。

Forge 的角色不是替代 flow-kit，而是读取 flow-kit 的阶段 / change-id / task-id，并通过 Claude Code hooks、routing log、health check 和 smoke test 做运行时门禁。没装 Forge 时，flow-kit 行为完全不变。

详细说明见：[`reference/runtime-adapters/forge.md`](reference/runtime-adapters/forge.md)。

---

## 「我能不能只用 X」决策指南

按你的具体诉求选择：

| 你想要的 | 推荐组合 | 不需要 |
|---|---|---|
| 只想让 AI 更靠谱、更少幻觉 | `RULES.md` 注入到系统提示 | 其他都不要 |
| 只想理顺一个想法 | `prompts/0-change.md` + `templates/CHANGE.md` | 其他都不要 |
| 只想探索 / 调研 / 对比 / 排查 bug（不写代码）| `prompts/X-explore.md` | 其他都不要 |
| 只想写好一份需求 | `prompts/1-requirement.md` + `templates/REQUIREMENT.md` + `templates/CONTEXT.md` | 其他都不要 |
| 只想做技术设计 | `prompts/2-design.md` + `templates/DESIGN.md` + `reference/tech-stacks.md` | 其他都不要 |
| 只想选个技术栈 / 评估迁移 | `reference/tech-stacks.md` | 其他都不要 |
| 只想拆好一组任务 | `prompts/3-task.md` + `templates/TASK.md` | 其他都不要 |
| 只想做 code review | `prompts/6-review.md` + `templates/REVIEW.md`（装 [`brooks-lint`](https://github.com/hyhmrright/brooks-lint) 出书本引用 review）| 其他都不要 |
| 只想做 UI 视觉 audit | `prompts/6-review.md` 第 3 轮 + `reference/ui-anti-patterns.md` | 其他都不要 |
| 只想做代码库周期性体检 / 评估技术债 | `prompts/M-health.md`（装 [`brooks-lint`](https://github.com/hyhmrright/brooks-lint) 自动 4 维仪表板 + 还债优先级）| 其他都不要 |
| 只想扫冗余 / 找死代码 / 清未用导出 / 清未用依赖 | `prompts/M-health.md` 步骤 2.5（装 `jscpd` + `knip`/`vulture`/`staticcheck` 任一）| 其他都不要 |
| 只想立测试规范 / 做发布前自查 | `prompts/5-test.md` + `templates/TEST.md` + `reference/test-pyramid.md` | 其他都不要 |
| 只想给前端项目立 design system | `prompts/2a-ui-design.md` + `templates/UI-DESIGN.md` + `reference/ui-aesthetics.md` | 其他都不要 |
| 只想沉淀失败教训 | `templates/LESSONS.md`（手动维护即可） | 其他都不要 |
| 给 AI 一个项目级"小抄" | `templates/CONTEXT.md` 填好 | 其他都不要 |
| 想跑完整闭环 | 全套 | — |

> **核心一条**：除了 `PROGRESS.md` 和 `STATE.md` 这种和流程绑定的临时/状态文件，其它每一个 prompt 与每一个 template 都设计成可独立使用，不要求你必须全套上车。

---

## Token 成本表（让你提前知道价格）

> flow-kit 的设计是 **token 多花、单窗压力小、产物有外溢价值**。下表帮你按场景选挡位。
> 完整估算逻辑见 `@flow-kit/GO.md` 的「Token 预算估计」段。

### 单点调用 · 你只想跑某一阶段

| 单独跑 | 典型 token | 加载 | 产出 |
|---|---|---|---|
| `prompts/0-change.md` | **~6k - 8k** | CHANGE 模板 | 一份 CHANGE.md |
| `prompts/1-requirement.md` | **~6k - 9k** | REQUIREMENT 模板 + CONTEXT | 一份 REQUIREMENT.md（AC 完备）|
| `prompts/2-design.md`（含查 tech-stacks 适用矩阵） | **~10k - 15k** | DESIGN 模板 + tech-stacks 1 节 | 一份 DESIGN.md（含技术栈卡片）|
| `prompts/2a-ui-design.md` | **~12k - 18k** | UI-DESIGN 模板 + ui-aesthetics 2 节 + ui-anti-patterns 全文 | 一份 UI-DESIGN.md（含调性 + token）|
| `prompts/3-task.md` | **~8k - 12k** | TASK 模板 | 一份 TASK.md（N 个任务）|
| `prompts/4-dev.md` × **单个 task** | **~25k - 60k** | 4-dev prompt + LESSONS + 当前 task 块 | 实现代码 + 测试 + SUMMARY |
| `prompts/5-test.md` | **~30k - 80k** | TEST 模板 + test-pyramid 适用矩阵 | TEST.md（5 轮 + 真跑测试输出）|
| `prompts/6-review.md` | **~25k - 50k** | REVIEW 模板 + git diff + ui-anti-patterns | REVIEW.md（三轮 + 第四轮可选）|
| `prompts/7-integration.md` | **~20k - 40k** | 所有产物 | UAT + LESSONS 提名 |
| `prompts/M-health.md`（横向） | **~15k - 30k** | CONTEXT + LESSONS + 上次 health 报告 | `.specs/health/<date>-HEALTH.md` |
| `prompts/L-restyle.md`（横向） | **~25k - 50k** | UI-DESIGN + ui-aesthetics + ui-anti-patterns | 新 UI-DESIGN.md + theme migration |
| `prompts/X-explore.md`（横向） | **~5k - 30k**（按对话深度）| 无固定加载，按需探索代码 | 无强制产出（可选探索笔记）|

### 完整闭环 · 一个 change 走完 0~7

| change 规模 | 典型 task 数 | 完整模式 | 极简模式 |
|---|---|---|---|
| **小**（< 100 行 / 加字段 / 修 bug） | 1-2 | ~80k - 150k | ~50k - 100k（推荐：直接走单点）|
| **中**（100-500 行 / PR 级 feature） | 3-5 | ~150k - 300k | ~120k - 240k |
| **中-大**（500-1500 行 / 中型 feature） | 5-10 | **~250k - 530k** | **~205k - 445k** |
| **大**（1500+ 行 / milestone） | 10+ | ~500k - 1M | ~400k - 800k（建议拆 milestone）|

### 横向对比 · 真实场景的取舍

| 路线 | 总 token | 单窗压力 | 产物 | 适合 |
|---|---|---|---|---|
| **不走任何工具，原生 LLM** | ~30k - 80k | 高 | 代码 + 一段对话 | 一次性脚本 / 改 < 30 行 |
| **7 个原生 skill 单会话**（brainstorm / writing-plans / TDD 等）| ~75k - 200k | **高**（一窗装下所有） | 代码 + 测试 + review 文字 | 个人项目 / 50-300 行 / hackathon |
| **flow-kit 极简模式** | ~205k - 445k | 低（每 task fresh ctx）| REQUIREMENT / DESIGN / TASK / SUMMARY × N / TEST / REVIEW + LESSONS（UI 项目另含 UI-DESIGN） | 中等 feature / 个人长期项目 |
| **flow-kit 完整模式** | ~250k - 530k | 低 | 8 个 .md + LESSONS + history | 团队项目 / PR 必走 / 长期维护 |
| **flow-kit + brooks-lint 完整** | ~280k - 600k | 低 | 同上 + 书本引用 review | 严肃工程 / 跨人交接 / 高质量门 |

### 怎么选

1. **改 < 30 行 / 一次性 / 简单 bugfix** → 跳 flow-kit，让 AI 直接改
2. **改 30-100 行 / 个人项目** → 走 7 个原生 skill（brainstorm + TDD + review 三件套就够）
3. **改 100-500 行 / 想要可追溯产物** → flow-kit 极简模式，或 flow-kit 单点调 6-review
4. **改 500+ 行 / 团队 / 长期** → flow-kit 完整 + brooks-lint，**多花的 token 沉淀为团队资产**

### 为什么 flow-kit token 多

主要花在三处（按贡献排序）：

- **40%** · 每个 task 一个 fresh context，重新加载 prompt + spec（这是 R1.4 的核心设计——降单窗压力，避免 50k+ 的窗内崩盘 / "打转"）
- **25%** · 写 8 个 .md 工件（CHANGE / REQUIREMENT / DESIGN / UI-DESIGN / TASK / SUMMARY × N / TEST / REVIEW）
- **20%** · brooks-lint / brooks-test / brooks-audit / brooks-debt 的 4 个命令调用（装了才有）

**剩下 15% 是 prompt 文件本身长**——已经被 R1.9 工件加载预算 + reference 按节读 + R1.3 引用历史用 `@路径` 等机制压到底了。

---

## 老项目（brownfield）安全护栏总览

> flow-kit 的两个高频担忧：**AI 不按既有架构开发** + **AI 乱删乱改不相关代码**。下表是每条护栏拦的具体事故。

| 编号 | 护栏 | 文件 | 拦的事故 |
|---|---|---|---|
| **B1** | 入场扫描自动生成 CONTEXT.md | `prompts/I-intel-scan.md` + `GO.md` 第三步 | AI 不知道项目栈 / 命名风格 / 既有抽象 → 写出格格不入的代码 |
| **B2** | DESIGN 0.5「既有架构对齐」段 | `prompts/2-design.md` 步骤 0.5 + DESIGN 模板 | 引入与项目格格不入的新模式（如老项目用 Repository，AI 写出直接调 ORM 的 Service） |
| **B3** | TASK 的 `read_files` + `write_files` 强约束 + R6.5 提交前 diff 边界 verify | `prompts/3-task.md` + `prompts/4-dev.md` 步骤 5 + RULES R7.3 | "顺手改了别的文件" → 提交前自动检测越界并要求回滚 / 扩范围 |
| **B4** | 1.8 破坏性变更高门槛协议 | `prompts/4-dev.md` 1.8 + RULES R4.6 | 删错 5+ 行代码 / 改坏公共接口 → 强制 grep 引用图 + 反问用户 + 回归测试覆盖 |
| **B5** | 1.4 沿用既有抽象 grep | `prompts/4-dev.md` 1.4 + RULES R6.4 | 重复实现已有抽象（项目里已有 formatDate，AI 又写一个）→ 写代码前必须 grep |

### 老项目首次使用流程

```
@flow-kit/GO.md
帮我加一个 X 功能
   ↓
GO.md 第三步检测到没有 CONTEXT.md
   ↓
反问："建议先扫描项目（~15-30k tokens，仅首次）"
   ↓ 用户：1（现在跑）
prompts/I-intel-scan.md → 生成 CONTEXT.md（含既有抽象索引 / 禁动清单 / 命名约定）
   ↓
回到原意图 → 0-change → 1-requirement → 2-design
   ↓ DESIGN 0.5 步骤
列出本次 change 触碰的既有模块 + 沿用决策 + 禁动清单（从 CONTEXT 复用）
   ↓ 3-task
每个 task 声明 read_files / write_files（write_files 不许包含 DESIGN 禁动清单）
   ↓ 4-dev 每个 task
1.4 沿用既有抽象 grep（防重复）
1.7 schema 任务额外检查（防忘生迁移）
1.8 破坏性变更门槛（防删错）
TDD → verify
5 提交前 diff 边界 verify（防越界）
   ↓ 5-test / 6-review / 7-integration
```

### 跳过这套护栏会出什么事

| 场景 | 不走护栏 | 走完护栏 |
|---|---|---|
| 老项目用 Repository 模式，AI 加新功能 | 写 Service 直接调 ORM | DESIGN 0.5 强制对齐 → 沿用 Repository |
| 项目已有 `formatDate`，AI 要做日期展示 | 自己写新 `formatDate2` | 1.4 grep → 找到 → import 用 |
| 看到一个"没人用"的函数 | 顺手删了（其实是反射调用） | 1.8 协议 → grep 全库 → 反问用户 → 不轻删 |
| 给公共 hook 加必填参数 | 改了，12 处调用全炸 | 1.8 协议 → grep 引用图 → 反问选兼容期 |
| 改某个文件时旁边文件有 bug | 顺手修了，污染 PR | 5 边界 verify → 检测越界 → 要求开新 task |
| 项目用 zustand，AI 加新状态 | 引入 redux | 1.4 grep package.json → 用 zustand |

### 老项目场景的 token 成本

| 项目 | 首次 | 之后每个 change |
|---|---|---|
| 5 万行老项目首次跑 flow-kit | +15-30k（intel-scan）= ~265-560k | ~250-530k（与 greenfield 同） |
| > 10 万行老项目 | +30-50k（intel-scan）= ~280-580k | ~250-530k |

**结论**：老项目场景下 intel-scan 是**一次性投资**，后续每个 change 用 CONTEXT.md 都受益。

---

## 升级与定制

- **要更轻量**：删掉 `prompts/2-design.md` 和 `prompts/6-review.md`，对应阶段并入相邻步骤
- **要更严格**：在 `RULES.md` 里追加项目专属规则（如 commit 格式、覆盖率门槛）
- **要换语言/换术语**：批量替换 prompts 与 templates 里的关键词，方法论本身保留
- **要加新阶段**：在 `prompts/` 里加 `8-<name>.md`，并在 `METHODOLOGY.md` 流程图里插入

---

## 设计原则（说清楚，方便你判断要不要改）

1. **每个阶段一次 fresh context**：阶段切换时清窗，靠 `.md` 工件传递状态，不靠对话堆叠
2. **每个产物可被一个文件承载**：没有隐式状态，没有"AI 记得"
3. **任务必带 verify**：`TASK.md` 每项含可执行的验证命令，否则不允许进入执行
4. **审查至少两轮**：spec 合规 + 代码质量；建议其中一轮跨模型
5. **失败是输入而不是终点**：UAT 失败自动产 fix-plan，回到 `4-dev`，最多 3 轮
6. **artifact-first，不是 phase-gated**：阶段可压缩、可回炉，但 `.md` 不能缺席。它不是 phase-gated，而是 artifact-gated

---

## 许可

随便用。MIT。
