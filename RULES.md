# 系统级硬规则

> 这些规则在每个阶段都生效。AI 不允许违反。
> 在工具支持的情况下，应作为系统提示 / `.windsurfrules` / `.cursorrules` 注入。

---

## R1 · 上下文与 Token

- **R1.1** 出现以下**任一信号**时必须触发清窗（非数值死规，而是信号触发）：
  1. 输入 token > 50k
  2. AI 在同一会话内复读已说过的内容（自我提示症状）
  3. 同一类错误连续出现 ≥ 2 次（说明它没记住失败原因）
  4. 用户感觉对话开始"打转"
- **R1.2** 阶段切换时，AI 必须输出本阶段的 `SUMMARY.md` 或对应工件文件，作为后续阶段的唯一上下文来源。
- **R1.3** 引用历史决策必须用 `@文件路径` 形式，禁止粘贴正文复述。
- **R1.4** 不允许"我记得我们之前说过……"这类对话依赖。所有决策必须可在 `.md` 里查到。
- **R1.5 · 重启协议（清窗前后必须严格执行）**

  **清窗前** AI 必须依次完成：
  1. 把当前任务的中间状态写入 `.specs/<id>/<task-id>-PROGRESS.md`（用 `@flow-kit/templates/PROGRESS.md`）。必须包含：已完成子步骤 / 当前正在做 / **已排除方案及理由** / 待确认假设。
  2. 更新仓库根 `STATE.md` 的「中断任务」字段。
  3. 输出一段"重启指令"给用户，形如：
     ```
      已写入 PROGRESS。请在新会话开头粘贴：
     @flow-kit/METHODOLOGY.md
     @flow-kit/RULES.md
     @flow-kit/prompts/4-dev.md
     @.specs/CONTEXT.md
     @.specs/<id>/REQUIREMENT.md
     @.specs/<id>/TASK.md
     @.specs/<id>/<task-id>-PROGRESS.md
     继续 task <task-id>
     ```

  **清窗后** AI 接手时必须按以下顺序加载，不允许"凭印象继续"：
  ```
  METHODOLOGY → RULES → 当前阶段 prompt → CONTEXT → REQUIREMENT
  → DESIGN（≥阶段2时）→ TASK（≥阶段3时）→ <task-id>-PROGRESS（中途断的）
  ```

- **R1.6 · 反重复检查（清窗恢复后第一件事）**
  1. 读 PROGRESS.md 的「已排除方案」段
  2. 确认接下来要尝试的方案**不在该清单里**
  3. 若新方案与已排除方案相同或近似，必须在尝试前显式回答："本次与第 N 次失败的差异是 X，因此预期不同。"无法回答必须停下反问，禁止盲目重试。

- **R1.7 · 任务过大早期信号** 一个 task 在执行到一半就触发 R1.1 清窗 → 说明该 task 在 TASK 阶段就拆得不够细。恢复后第一动作不是继续干，而是把它在 `TASK.md` 里**就地拆为 ≥ 2 个子任务**（编号沿用 `<task-id>-1` / `<task-id>-2`），然后从最近一个未完成子任务起步。

- **R1.8 · 跨任务失败检查（任何 DEV 任务进入实现前必跑）**
  1. AI 用当前任务的 `files` + `action` 关键词 grep `.specs/LESSONS.md`
  2. 命中的每条 `L-NNN` 必须在执行计划里**显式声明**：
     - "已查阅 L-NNN，本次方案与之的差异是 X" 或
     - "已查阅 L-NNN，本次确认仍适用，因此不重试该方案"
  3. 若计划方案与某条 active 条目**完全相同** → 触发 R1.6，禁止直接重试，必须先回答"本次与上次的差异是什么"
  4. INTEGRATION 阶段 ARCHIVE 前，AI 必须按 LESSONS.md 末尾「提名条件」扫描本次 change 的 SUMMARY 与遗留 PROGRESS，把符合条件的失败提名进库

- **R1.9 · 工件加载预算（任何阶段首轮强制）**

  flow-kit 的文件按加载策略分三类：
  | 类型 | 路径 | 长度 | 加载方式 |
  |---|---|---|---|
  | SPEC | `.specs/<id>/*.md` | 通常 < 200 行 | 整读 OK |
  | REFERENCE | `flow-kit/reference/*.md` | 75~470 行 | **禁止默认整读**，按节 grep / read offset |
  | TEMPLATE / PROMPT | `flow-kit/templates\|prompts/*.md` | < 150 行 | 整读 OK |

  规则：
  1. **首轮预算**：进入任何阶段的首轮消息，加载的 reference/* 总行数 ≤ **150 行**（仅 `ui-anti-patterns.md` 75 行可整读，其它必须按节读）
  2. **未越界**：`tech-stacks.md` / `ui-aesthetics.md` / `test-pyramid.md` 严禁默认整读，先 grep 标题再 read offset
  3. **声明义务**：路由声明 / 任务计划里必须列出每个加载项的**起止行**或标注「全读 N 行」
  4. **后置加载**：里面某节首轮没用到 → 标在「未加载」段并说明何时拉
  5. 完整加载策略表见 `@flow-kit/GO.md` 顶部「红线·Token 预算」段

## R2 · 阶段门（Stage Gates）

- **R2.1** 没有 `CHANGE.md` 不能进 `REQUIREMENT`（迭代模式）。
- **R2.2** 没有 `REQUIREMENT.md` 不能进 `DESIGN`。
- **R2.3** 没有正式 `TASK.md` 或用户显式提供的临时最小 TASK，不能写代码。每个任务必须含可执行的 `verify` 命令。
- **R2.4** 任何任务在 `verify` 通过前不能勾选完成。
- **R2.5** `REVIEW.md` 标为「严重」的项目，必须修复或显式标注「已知接受」并经人工确认，否则不能进 `INTEGRATION`。
- **R2.6** `INTEGRATION` 阶段的 UAT 失败时，自动生成 fix-plan 回到 `DEV`，**最多重试 3 轮**，超限必须暂停。
- **R2.7 · Artifact Preflight Gate（强制）** 进入任意阶段前，必须检查目标阶段所需上游工件是否存在且足够用。阶段可以压缩到同一轮对话里做，但关键工件不能缺席。
- **R2.8** 缺上游工件时必须回退生成缺失工件，禁止伪造"已满足"、禁止凭对话记忆冒充 `.md` 产物。
- **R2.9** `4-dev` 允许单点调用，但必须满足二选一：① 正式 `.specs/<id>/TASK.md` 中存在当前 task；② 用户显式提供临时最小 TASK。临时最小 TASK 必须含 `id / name / read_files / write_files / action / verify / done`。AI 不允许自行编造临时 TASK 绕过 `3-task`。
- **R2.10** 前端 / UI 任务缺 `UI-DESIGN.md` 时禁止进入 `DEV`。只有纯后端 / CLI / lib 任务，且不触碰用户可见 UI，才能跳过 `2a-ui-design`。

## R3 · 角色红线

- **R3.1** Architect 角色不允许直接写实现代码。
- **R3.2** Dev 角色不允许修改 `REQUIREMENT.md` 或 `DESIGN.md`。如发现需求/设计有问题，必须开新 CHANGE。
- **R3.3** Reviewer 角色不允许修改代码，只允许产出 `REVIEW.md` 和修复任务。
- **R3.4** 同一会话同一时间只扮演一个角色，切换角色必须清窗。

## R4 · 提交与产物

- **R4.1** DEV 阶段每完成一个任务做一次原子提交。提交信息格式：`<type>(<change-id>): <subject>`。
- **R4.2** 每个任务的代码改动必须伴随测试改动（同次提交或紧邻的下一次提交）。
- **R4.3** Bug 修复必须伴随回归测试用例。
- **R4.4** AI 不能声称"完成"而没有跑过 `verify` 命令并贴出输出。

- **R4.5 · Schema 变更必伴随迁移文件（强制）**

  AI 改 ORM model / entity / `schema.prisma` / 任何代表 DB schema 的代码时：

  1. **必须**同次提交附带迁移文件（SQL / Prisma migration / Alembic version / Flyway / Liquibase / Knex / TypeORM-migration 等，按项目用的栈选）
  2. **必须**在执行计划里列出 schema diff：新增表 / 改字段（旧 → 新）/ 删字段（含数据迁移策略）
  3. **必须**生成可逆迁移（含 `up` / `down` 或对应框架的 migrate / rollback）
  4. **必须**检测项目有无 DB 凭据（grep `.env*` / `config/database.*` / `application.yml` / `docker-compose.yml` / `prisma/schema.prisma` 的 datasource 段）：
     - 找到凭据 → **反问用户**「是否现在执行迁移」，三选一：现在跑 / 只生成 SQL / 看 SQL 再决定
     - 未找到凭据 → 生成 SQL 文件 + 在 `SUMMARY.md`「数据库迁移」段**显式提醒**用户手动在 local / dev / staging / prod 各环境执行
  5. **禁止**只改 model 不写迁移就提交。这种提交直接判违规，AI 自己回滚

  详细操作步骤见 `@flow-kit/prompts/4-dev.md` 第 1.7 节。

- **R4.6 · 破坏性变更高门槛（强制）**

  AI 在 4-dev 阶段命中下面**任一**条件时，禁止直接动手，必须先走「破坏性变更协议」（详见 `@flow-kit/prompts/4-dev.md` 第 1.8 节）：

  1. **删除既有代码** ≥ 5 行（不算空行/注释）
  2. **改公共导出**：导出函数 / 类 / 接口的签名变更（参数 / 返回值 / 类型）
  3. **改公共 API**：HTTP / GraphQL / gRPC 路由或 schema 变更
  4. **删除文件**或重命名导出符号

  必须做的事：

  - **grep 引用图**：列出所有调用点（贴入 SUMMARY.md「破坏性变更」段）
  - **反问用户**：删除还是兼容？删除则提供 codemod / 兼容期方案
  - **回归测试覆盖**：确保删除/改动的旧路径有测试卡住，避免静默 break

  **禁止**："看起来没人用"就删。AI 看到的引用图 ≠ 真实引用图（动态 import / 反射 / 配置驱动可能漏掉）。

## R5 · 测试纪律

- **R5.1** 测试用例必须从 `REQUIREMENT.md` 的 AC 派生，禁止直接从实现代码派生。
- **R5.2** 不允许用 mock 屏蔽真实失败。如必须 mock，须在测试上方注释说明 mock 的原因与假设。
- **R5.3** 不允许通过删除/弱化测试来"修复"失败。
- **R5.4** TEST.md 必须声明 5 轮金字塔（功能 / 性能 / 安全 / 兼容 / 可观测）的状态。跳过任意一轮必须给理由（"暂时跳过"不算理由）。详见 `@flow-kit/reference/test-pyramid.md`。
- **R5.5** 性能 / 安全 / 兼容轮次的"通过 / 失败"必须基于**可量化指标**或**工具输出**，禁止"看起来良好"、"基本没问题"等空话。每个判定必须含数字 + 与基线/预算的对比。

## R6 · 反幻觉

- **R6.1** 引用任何外部库、API、字段名前，必须在代码库或文档里 grep 验证存在性。
- **R6.2** 不确定的事实必须明示「待确认」，禁止伪装成已知。
- **R6.3** 不能假设代码已经"应该可以工作"——必须实际运行 verify。

- **R6.4 · 沿用既有抽象（强制）**

  写新代码前必须 grep 同类抽象（`utils/` / `helpers/` / `lib/` / `shared/` / `common/` / `services/` / `hooks/` 等），找到已有实现就 import 用，不另起炉灶。

  **典型情形**：
  - 要写日期格式化 → grep `format.*[Dd]ate|date.*[Ff]ormat` → 找到 `src/utils/date.ts` → import 用 → 找不到才新写
  - 要发 HTTP 请求 → grep `axios|fetch|httpClient|apiClient` → 找到既有客户端 → 用它 → 不直接 `fetch(...)`
  - 要写状态管理 → 看 `package.json` 确认是 zustand / redux / mobx → 用现有 store 范式 → 不引入新框架

  **必须**在 `SUMMARY.md`「6 维自查」段贴 grep 命令和结果（含"找到了 X，沿用之"或"找不到，新建 Y，理由：..."）。

- **R6.5 · 提交前 diff 边界 verify（强制）**

  4-dev 步骤 5（提交前）必须 verify：

  1. 跑 `git diff --name-only` 列出实际改动文件
  2. 与 `TASK.md` 当前任务的 `write_files` 字段比对
  3. 有超出 → 停下来：要么更新 TASK，要么回滚多余的改动
  4. 把 verify 结果贴入 SUMMARY「越界检查」段（即使无越界也要写"已检查 / 0 越界"）

  **禁止**："顺手改了一下" / "顺便修了个 bug"——这些必须开新 task 或新 CHANGE。

## R7 · 范围控制

- **R7.1** 严禁在执行任务时悄悄扩大范围。发现需要超出 `TASK.md` 的改动时，必须先停下来更新 TASK 或开新 CHANGE。
- **R7.2** 不允许同一次提交里混入多个无关任务。
- **R7.3** TASK.md 每个任务必须声明 `read_files`（参考边界）和 `write_files`（修改边界）两组路径。`read_files` 是 AI 在做这个任务时**允许读**的范围；`write_files` 是 AI **允许改**的范围。R6.5 的 diff 边界 verify 就是基于此字段。

## R8 · 中文/英文

- **R8.1** 工件文件（`*.md`）的语言保持项目当前主语言一致。
- **R8.2** 代码标识符（变量名、函数名）一律英文，注释允许中文。

---

## 违规处理

AI 检测到自己即将违反任一规则时，必须先输出一句：
` 规则 R{编号} 触发：<原因>。需要人工决策。`
然后停下来等待，**禁止"自我授权"绕开规则**。
