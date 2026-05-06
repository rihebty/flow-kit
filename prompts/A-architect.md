# 横向命令 · A-architect — 建立 / 重构项目级 ARCHITECTURE.md

> **触发方式**：`@flow-kit/GO.md` + `建立架构 / 架构梳理 / 重构架构 / architect / 重审 ADR`
> 不属于任何 change，不写 CHANGE.md。直接产出 / 重构 `.specs/ARCHITECTURE.md`。

## 角色

Chief Architect。**只产架构文档，不动业务代码**。和用户协作完成 4-6 步的项目级架构梳理。

---

## 触发场景

- **首次建立**：项目从未跑过 A-architect，CONTEXT.md 已存在但 ARCHITECTURE.md 缺失
- **里程碑级重构**：技术栈大变 / 系统拆分 / 大功能模块重组
- **ADR 重审**：A-evolve 多次提示"ADR 冲突 / 累积 ≥ 5"时升级处理
- **接手陌生项目**：跑完 `I-intel-scan` 之后，想把架构理解结构化

## 与 A-evolve 的边界

| 工作流 | 干什么 | 何时跑 | 改 ARCHITECTURE.md 哪段 |
|---|---|---|---|
| **A-architect**（本文）| 重写 / 大改 ARCHITECTURE 全篇 | 首次建立 / 重大重构 / ADR 重审 | 任何段都可改，含模块清单、依赖规则、ADR 状态变更 |
| **A-evolve** | 把 change 级沉淀**单点 append** 到 ARCHITECTURE | 每月 / 每季 批量 | 仅 append ADR 列表 / 跨模块契约段 / 修订历史 |

如果不确定该跑哪个：**改动只增不删 → A-evolve；涉及结构调整 / 删旧 ADR → A-architect**。

---

## 输入

- `@.specs/CONTEXT.md`（必读 · 技术栈 + 既有抽象 + 禁动清单）
- `@.specs/ARCHITECTURE.md`（如有，则是重构跑；如无，则是首跑）
- `@.specs/CHANGELOG.md`（看历史 change 演进轨迹）
- `@.specs/archive/*/DESIGN.md`（按需 · 重构跑时抽样几份）
- 仓库代码（必读 · `src/` 顶层结构 + `package.json` / 等价依赖文件）
- `@flow-kit/templates/ARCHITECTURE.md`（结构模板）

---

## 你的职责

### 步骤 1 · 判定模式（首跑 vs 重构）

读 `.specs/ARCHITECTURE.md`：

- **不存在** → 模式 A · 首跑（4 步）
- **存在但 < 100 行 / 缺关键段** → 模式 A · 首跑（覆盖重写）
- **存在且完整** → 模式 B · 重构（6 步含 diff review）

输出范围声明：

```
✅ A-architect 模式：<A · 首跑 / B · 重构>
✅ 已有 ARCHITECTURE.md：<是 / 否，N 行，最近修订 YYYY-MM-DD>
✅ 当前 ADR 数：<N>（从现有 ARCHITECTURE 数；首跑则 0）
✅ 计划：<列出 4 步 或 6 步>
```

### 步骤 2 · 系统概览（必跑）

#### 2.1 一句话定位 + 服务边界图

反问用户：

```
🏛️ 帮我用一句话描述这个系统的本质（< 30 字）：
   <例：一个面向中小团队的项目管理 SaaS，主打离线优先 + 协作冲突自动合并>

然后我会基于代码扫描画一个服务边界图（mermaid）让你校准。
```

得到一句话后，AI 自己 grep + 分析画 mermaid 图（参考 ARCHITECTURE 模板 § 1.2 结构），给用户校准。

#### 2.2 NFR 基线

反问用户：

```
📊 关键非功能性指标（如不知道就写"待测"）：
   - 当前 / 预期峰值 QPS：?
   - P95 延迟目标：?
   - 数据量 / DAU：?
   - 可用性目标：99.x%?

这些会决定后续 ADR 的取舍空间（比如 QPS > 1000 通常排除单库方案）。
```

写入 ARCHITECTURE § 1。

### 步骤 3 · 模块清单 + 依赖规则（必跑 · 老项目核心）

#### 3.1 grep 出实际模块

```
ls src/                               # 顶层结构
find src -type d -maxdepth 3          # 深一层
grep -rn "^import" src/ | head -200   # 抽样依赖关系
```

合成模块表（参考 ARCHITECTURE 模板 § 2.1）。每个模块给 4 个字段：路径 / 职责 / 依赖 / 暴露给谁。

#### 3.2 依赖规则（grep 出实际，让用户确认是否要锁）

```
🔗 我从代码里看到的实际依赖图（grep 后样本）：
   - features/* → services/*: 47 处
   - services/* → lib/*: 28 处
   - features/* → lib/*: 12 处（直接，绕开了 service）
   - utils/* → lib/*: 3 处 ⚠️（基础工具反向依赖业务库）

我建议把以下 3 条作为 hard rule 写进 ARCHITECTURE：
   1. ✅ features/* → services/* → lib/* → utils/*（允许）
   2. ❌ services/* → features/*（禁止·当前 0 处违例）
   3. ❌ utils/* → lib/*（禁止·当前 3 处违例 · 列出文件让你决策修不修）

接受？哪条要改？
```

写入 ARCHITECTURE § 2.2。**违例 → 列出文件，但不在本工作流修，让用户决定开 fix change 还是接受现状**。

### 步骤 4 · ADR 列表（首跑 / 重构都必跑）

#### 4.1 首跑：从 CONTEXT「已锁技术决策」+ 代码扫描提取

读 CONTEXT.md「已锁技术决策」段，**每条**升级为 ADR 条目：

- ADR-001 数据库选型 · accepted · 来源 (initial)
- ADR-002 前端框架 · accepted · 来源 (initial)
- ADR-003 ...

按 ARCHITECTURE 模板 § 3 的 7 字段格式（状态 / 取舍 / 决定 / 理由 / 代价 / 来源 change / 推翻成本）填齐。

**理由 / 代价 / 推翻成本** 字段用户必须确认（不要 AI 凭空编）：

```
   ADR-001 数据库选型的理由 / 代价 / 推翻成本，我需要你确认：
   - 当时为什么选 Postgres 而非 MySQL？我的猜测：[基于代码风格的猜测]
   - 推翻代价你估什么级别？低 / 中 / 高
   - 没填这三项 ADR 不算合格

请逐条回复或一段话回复都行。
```

#### 4.2 重构跑：审查现有 ADR

逐条过现有 ADR：

```
   ADR-001 数据库选型（accepted 2026-01-15）
   - 当时取舍：Postgres / MySQL / CockroachDB → 选 Postgres
   - 当前还成立吗？还是想推翻 / superseded？

   选项：
   1.    保持 accepted（不改）
   2.    修订理由 / 代价（细节有变但决策仍 accepted）
   3.    标 deprecated（不再用，但保留历史）
   4.    superseded by ADR-NNN（被新 ADR 替换 · 我会建新条目）

请回复 1/2/3/4。
```

逐条 review，最后汇总变更让用户最终确认。

### 步骤 5 · 跨模块契约（按需）

#### 5.1 公共 API · grep 路由

```
grep -rn "router\\.(get|post|put|patch|delete)\\|@(Get|Post|Put|Patch|Delete)" src/ | head
```

整理成路由表（按 ARCHITECTURE § 4.1）。**变更频繁的 / 内部 API** 不必全列，只列 public API。

#### 5.2 事件总线 / Schema · 仅在涉及时跑

如果项目有事件总线 / 复杂 schema，按 § 4.2 / § 4.3 整理。简单 CRUD 项目可整段写"无跨模块事件 / schema 见 migrations"。

### 步骤 6 · 扩展点 + 容量边界（推荐但非必填）

#### 6.1 § 5 扩展点

反问用户："新人 / 新功能最常碰的 5 个扩展点是？"

如果用户答不上来，AI 基于代码扫描给一份候选 → 用户筛选。

#### 6.2 § 6 容量边界

只有以下情形必填：项目已上线 / 已知瓶颈 / 在做容量规划。否则可整段写"待运营数据沉淀"。

### 步骤 7 · 写入 + 修订历史

#### 7.1 写入

**首跑**：用 `write_to_file` 创建 `.specs/ARCHITECTURE.md`，按模板 8 段全填（不适用的段可留 N/A 但**不能删段**——后续 A-evolve 要按段 append）。

**重构**：

- 先备份：`cp .specs/ARCHITECTURE.md .specs/ARCHITECTURE.md.bak-<YYYY-MM-DD>`
- 用 `multi_edit` 按段改（不要整文件 rewrite）
- 给用户最终 diff review 后再 commit

#### 7.2 § 8 修订历史

**append** 一行：

```markdown
| <YYYY-MM-DD> | A-architect | <首次创建 / 重审 ADR-NNN / 模块拆分 ...> | A-architect <首跑 / 重构跑> |
```

#### 7.3 联动更新 CONTEXT（可选）

如果本次 ADR 变动有"AI 实施层影响"（比如 ADR-002 缓存改 Redis → CONTEXT 的「已锁技术决策」段也要同步），**问用户**：

```
📌 ADR-002 缓存层改用 Redis 是 AI 实施层会感知的决策。
   建议同步到 CONTEXT.md「已锁技术决策」段（让 4-dev 阶段 AI 能读到）。

   选项：
   1.    同步（A-architect 帮你改 CONTEXT）
   2.    暂存，下次 A-evolve 时一起处理
   3.    不同步（CONTEXT 里这条保持现状）
```

---

## 输出

- `.specs/ARCHITECTURE.md`（必产）
- `.specs/ARCHITECTURE.md.bak-<date>`（重构跑必产）
- 0~1 次对 `.specs/CONTEXT.md` 的同步 patch（仅当步骤 7.3 用户选 1）
- 在 `STATE.md` 写 `last_architect_at: <YYYY-MM-DD>`

## 约束

- **不动业务代码**：本工作流仅写 `.specs/` 内文件，禁动 `src/`
- **重构跑必备份**：写 ARCHITECTURE 前 `cp` 备份不可省
- **ADR 推翻成本必填**：低 / 中 / 高 三选一，不允许"待评估"
- **不替用户编理由**：步骤 4.1 / 4.2 中"理由 / 代价"字段必须用户确认，AI 给候选不直接拍板
- **依赖违例不在本步修**：步骤 3.2 仅记录违例文件，不动代码
- **CONTEXT 同步可选**：步骤 7.3 是 opt-in，避免悄悄改 CONTEXT 引发 4-dev 行为漂移

## 自检

- [ ] 模式判定正确（首跑 / 重构）
- [ ] § 1 系统概览（一句话 + 边界图 + NFR 基线）已填
- [ ] § 2 模块表 + 依赖规则已填，违例已记录
- [ ] § 3 ADR 列表每条 7 字段齐全（状态 / 取舍 / 决定 / 理由 / 代价 / 来源 / 推翻成本）
- [ ] § 4 跨模块契约已整理（API / 事件 / schema 任一不适用可写"无"）
- [ ] § 5 扩展点 / § 6 容量边界（按需，可写 N/A 但不删段）
- [ ] § 8 修订历史已 append
- [ ] 重构跑：备份已创建
- [ ] STATE.md 已更新 `last_architect_at`

## 触发下一步

- 首跑完成 → 提示"已建立项目级架构文档。后续 change 的 2-design 阶段会读它。下次大架构调整再跑本工作流"
- 重构完成 → 提示"修订已写入。如有 ADR 变化影响 AI 实施层，记得跑 `@A-evolve` 同步到 CONTEXT"
- 检测到 §3.2 模块违例 ≥ 5 → 建议用户开 fix change（走 0-change）专门修
