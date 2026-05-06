# 阶段 6 · REVIEW — 三轮审查（spec 合规 + 代码质量 + UI）

## 角色

你是 Reviewer。**只产报告 + 修复任务，不直接改代码**（R3.3）。

## 输入

- `@.specs/<change-id>/REQUIREMENT.md`
- `@.specs/<change-id>/DESIGN.md`（如有）
- `@.specs/<change-id>/UI-DESIGN.md`（如是前端项目）
- `@.specs/<change-id>/TASK.md`
- `@.specs/<change-id>/TEST.md`
- 本次变更的 git diff（用户提供或 AI 通过工具获取）
- `@flow-kit/reference/ui-anti-patterns.md`（如是前端项目）

## 你的职责

使用 `@flow-kit/templates/REVIEW.md` 模板分三轮审查（后端 / lib 项目跳过第三轮）。

### 第一轮 · Spec 合规审查

逐条对照 `REQUIREMENT.md` 的 AC，看实现是否真做到。
检查项：

- [ ] 每条 AC 是否被实现
- [ ] 每条 AC 是否被测试覆盖（链接到 TEST.md）
- [ ] 是否引入了 `out of scope` 里明令排除的内容
- [ ] 是否新增了 REQUIREMENT.md 里没有的功能（范围蔓延）
- [ ] 是否触动了 DESIGN.md 之外的架构

### 第二轮 · 代码质量审查（书本驱动 6 维衰退风险）

#### 2.0 TEST.md 5 轮金字塔完整性（先查）

打开 `.specs/<id>/TEST.md`，检查"本次测试范围声明"段：

- [ ] 5 轮状态都明确（无未填）
- [ ] 跳过的轮次都有理由（不允许"暂时跳过"）
- [ ] 第 1 轮（功能）每条 AC 有覆盖
- [ ] 第 2 轮（性能）若必跑：实测 / 预算 / 上版基线三列齐全；退步项有处理
- [ ] 第 3 轮（安全）若必跑：依赖 / 秘钥 / SAST / OWASP 各有处理记录
- [ ] 第 4 轮（兼容）若必跑：跨浏览器矩阵 / 数据迁移 / 跨版本对应填齐
- [ ] 第 5 轮（可观测）若必跑：日志 / 指标 / 告警 / 健康检查清单逐项验证

任意一项不达 → 标 🔴 Critical，先回 5-test 阶段补完，再继续后续审查。

#### 2.1 代码质量诊断 · 6 维衰退风险

以 [brooks-lint](https://github.com/hyhmrright/brooks-lint) 提出的 6 个生产代码衰退风险为诊断维度（源于 12 本经典软件工程书籍：《重构》/ 《Clean Architecture》/ 《DDD》/ 《Pragmatic Programmer》/ 《Philosophy of Software Design》 等）：

| 编号 | 衰退风险 | 诊断问题 | 主要源头 |
|---|---|---|---|
| R1 | Cognitive Overload 认知过载 | 理解这段代码要多少心智？ | Code Complete / Refactoring / DDD / Philosophy of SD |
| R2 | Change Propagation 变更传播 | 改一点会坏多少不相干的地方？ | Refactoring / Clean Architecture / Pragmatic / SE@Google |
| R3 | Knowledge Duplication 知识重复 | 同一个决定是否被表达在多处？ | Pragmatic / Refactoring / DDD |
| R4 | Accidental Complexity 偶然复杂 | 代码是否比问题本身更复杂？ | Refactoring / Code Complete / Brooks / Philosophy of SD |
| R5 | Dependency Disorder 依赖混乱 | 依赖流是否一致方向（高层 → 低层）？ | Clean Architecture / Brooks / Pragmatic / SE@Google |
| R6 | Domain Model Distortion 领域扭曲 | 代码是否忠实反映业务领域？ | DDD / Refactoring |

> **R3 边界（重要）**：这里的"知识重复"是**概念级**——"同一个业务规则 / 常量 / 决策被表达在多处"。
> 字面级的重复代码块、未用导出 / 依赖、死代码等**不属于 R3 范畴**，交由 `@prompts/M-health.md` 步骤 2.5 的冗余扫描处理（jscpd / knip / vulture / staticcheck 等工具级扫描 · 全库级别 · 定期跑）。
> 6-review 只盯本次 diff 的概念层级；字面级冗余是"仓库级长期债"，跨 PR 才能看清，因此不放在 PR review 里。

##### 路径 A · 装了 brooks-lint（首选）

在 Claude Code / Gemini CLI / Codex CLI 里调用：

```
/brooks-review            # 基于 diff 的 PR 级诊断
```

或针对大型变更（架构调整、跨模块重构）补跑一次：

```
/brooks-audit            # 架构审计，产 Mermaid 依赖图、标出循环依赖
```

**输出必须包含**该工具要求的四要素（也是 flow-kit 下游认的格式）：

```
### 🔴/🟡/🟢 R<x> · <风险名>：<一句话结论>
**Symptom（症状）**：<在哪个文件:行号发现的具体问题>
**Source（源头）**：<哪本书哪一节提出这个原则，例如 Fowler · Refactoring · Divergent Change>
**Consequence（后果）**：<不修会怎么样，未来多久会爆>
**Remedy（修补）**：<具体怎么改，贴 before/after 代码或接口调整>
```

把 brooks-lint 输出原样贴入 `REVIEW.md` 的「代码质量审查 · 6 维衰退」段，不要改写丝毫。补充部分仅为指向 fix 任务。

##### 路径 B · 未装 brooks-lint（内置回退）

AI 自己逐个维度诊断 diff，输出上面同样的 4 要素格式，发现的每个问题都要：

- 标出 **R1~R6 编号**
- 指出具体 `<file>:<line>`
- 引用上表中至少一本书作为 Source（不要“根据最佳实践”这种空话）
- 按下方「严重度分级」段标 🔴 Critical / 🟡 Major / 🟢 Minor

内置路径下**成果质量明显低于 brooks-lint**（根据该工具 benchmark：带书本引用的发现率 100% vs 约 16%）。如果项目质量要求高建议装上 brooks-lint。

#### 2.2 架构依赖检查（大型 change 触发）

**触发条件**：本次变更满足**任一**项：
- 新增或重名了顶级模块 / package / 目录
- 危险 `import` 合并（业务代码 import 类似 `infrastructure/` / `db/` 这种低层包）
- DESIGN.md 表示引入了新中间件 / 新服务
- 跨 ≥ 5 个模块的重构

装了 brooks-lint → `/brooks-audit`，拿到 Mermaid 依赖图，贴入 REVIEW.md。重点核：
- 是否出现**循环依赖**（图中虚线反向箭头）
- 是否出现「业务层 → 低层」线路以外的反向依赖（如 `domain/` 依赖 `controller/`）
- 是否出现跨边界依赖（如 `frontend/` 直接 import `backend/` 实现）

未装 brooks-lint → AI 自己画个简化 Mermaid 依赖图，判同三点。

### 第三轮 · UI 视觉审查（仅前端项目）

**触发条件**：本次 change 含 `UI-DESIGN.md` 或 diff 涉及任何 UI 文件（`.css` / `.tsx` / `.vue` / `.html` / `.svelte` 等）。

#### 3.1 Design Tokens 一致性

- [ ] 实现里的颜色值是否全部来自 UI-DESIGN.md frontmatter（CSS variables / theme）？
- [ ] 是否有硬编码的 hex / 字号 / 间距数值？（命中即 🔴 Critical）
- [ ] 字体是否与 UI-DESIGN.md 声明一致？是否引入了 anti-pattern 字体（Inter / Roboto / Arial）？

#### 3.2 Anti-Pattern 扫描

逐项对照 `@flow-kit/reference/ui-anti-patterns.md` 的"强制禁忌"段：

- [ ] 字体类（无 AI slop 默认字体）
- [ ] 颜色类（无纯黑/纯白、无紫色渐变、无彩底灰字、无第二个强调色）
- [ ] 阴影类（at rest 平面、alpha ≤ 0.15）
- [ ] 边框类（无彩色侧条 > 1px、无玻璃拟态）
- [ ] 动效类（无 bounce/elastic、不动 layout 属性、支持 reduced-motion）
- [ ] 布局类（无卡片嵌套、无 SaaS hero-metric template）
- [ ] 文案类（无 hedging、无 lorem ipsum、按钮动词具体）
- [ ] 组件类（无 placeholder 充当 label、模态可 ESC 关闭）

每条命中**必须列出文件:行号**，标 🔴 Critical 并生成 fix 任务。

> 装了 [impeccable](https://impeccable.style) → 跑 `npx impeccable detect <changed-files>` 自动化扫描，把输出贴进 REVIEW.md。

#### 3.3 视觉北极星一致性

回到 UI-DESIGN.md 第 1 节"美学北极星"，问一个问题：
**"如果只看实现的截图，看得出来这个产品的调性是 <UI-DESIGN 声明的那个> 吗？"**

不能 → 标 🟡 Major，列出哪些视觉决策让调性失焦，建议怎么改。

#### 3.4 无障碍快检

- [ ] 颜色对比 ≥ WCAG 2.1 AA（用工具实测，不是肉眼）
- [ ] 所有交互元素键盘可达（Tab 顺序合理）
- [ ] 焦点环可见（不是默认蓝色 outline，而是与设计调性匹配的）
- [ ] `prefers-reduced-motion` 响应正确
- [ ] 表单 label 显式关联（不靠 placeholder）
- [ ] 图片 alt 文本（装饰图用 `alt=""`）

### 第四轮 · 补充审查（可选 · 按触发条件跳）

上面三轮是**必跑**。本轮两项都是**按触发条件补跑**，不命中条件可跳。

#### 4.1 技术债评估（适用于里程碑 / 季度大版本 / 重构项目）

**触发条件**：本次 change 是里程碑 / 季度大版本 / 重构项目，或 `.specs/CONTEXT.md` 「技术债」段多于 30 天未更新。

```
/brooks-debt              # 装了 brooks-lint 才能调
```

输出会给出：
- 各项债务的 **Pain × Spread 优先级**
- Critical / Scheduled / Monitored 的还债路线图

拿到输出后：
- 🔴 Critical · 本次必修 → 追加为 fix 任务
- 🟡 Scheduled · 近 1~3 个迭代 → 追加为 backlog，记入 `.specs/CONTEXT.md` 的「技术债」段
- 🟢 Monitored · 仅记录不处理 → LESSONS.md

未装 brooks-lint → 跳过本段（内置不提供回退，债评估需要书本包装才不会“凭感觉”）。

#### 4.2 跨模型 spot-check（强烈建议）

**触发条件**：以下任一项命中：
- 涉及安全/认证
- 涉及并发/分布式
- 单一函数 > 80 行
- 测试覆盖率有显著下降

拿另一个模型跑同样的三轮审查，两份报告的差异填入 `REVIEW.md` 末尾的「跨模型分歧」章节。重点看两份报告都指出的 🔴 项（差异越少越可信）与**仅一方指出的 🔴 项**（需人工裁判）。

### 严重度分级

每个发现项标注：
- 🔴 **Critical**：必须修复（数据损坏、安全漏洞、AC 未实现）
- 🟡 **Major**：建议修复（明显的设计问题、显著性能回归）
- 🟢 **Minor**：可选改进（命名、风格、小重构）

### 产出修复任务

对所有 Critical 和决定要修的 Major，**追加到 `TASK.md`** 末尾，编号延续（如 `T-FIX-01`），并触发回到 `4-dev`。

## 输出

- `.specs/<change-id>/REVIEW.md`
- 0~N 条新增 fix 任务追加到 `TASK.md`

## 约束（强制）

- **R3.3**：禁止直接修改代码
- **R2.5**：所有 Critical 必须修复或显式「已知接受」并经人工确认，否则禁止进 INTEGRATION
- 不允许笼统结论（"代码写得不错"），每条结论必须有具体行号或文件引用

## 自检

- [ ] 三轮主审查都做了（后端 / lib 项目只跳第三轮 UI，不能跳二轮）
- [ ] 二轮 · 6 维诊断输出含 4 要素 + 书本引用 + R1~R6 编号
- [ ] 装了 brooks-lint 优先用（`/brooks-review` + `/brooks-audit`），输出原样贴入报告
- [ ] 第四轮按触发条件判完（命中触发必跑，未命中可跳但要在 REVIEW.md 写明"X 未命中"）
- [ ] 每条发现都有严重度标签
- [ ] 每个 Critical 都已生成 fix 任务
- [ ] 报告里没有自己悄悄改过的代码

## 触发下一步

- 有 Critical 待修 → `@flow-kit/prompts/4-dev.md`（执行 fix 任务）
- 全部通过或人工接受 → `@flow-kit/prompts/7-integration.md`
