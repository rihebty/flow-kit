# SUMMARY: <T01 - 任务名> 

- **Change ID**: <id>
- **Task ID**: <T01>
- **完成时间**: <YYYY-MM-DD HH:mm>
- **AI 角色**: Dev

---

## 做了什么（一段话）

<这次任务实际完成的事，避免照抄 TASK.md 的 action。重点：实际选择 / 偏离原计划的地方 / 任何遗留。>

## 改动文件

| 文件 | 性质 | 说明 |
|---|---|---|
| `src/theme/ThemeContext.tsx` | 新增 | 实现 ThemeProvider 与 useTheme |
| `src/theme/__tests__/context.test.ts` | 新增 | 单元测试 |
| `src/App.tsx` | 修改 | 包入 ThemeProvider |

## verify 输出（必填）

```text
<贴出真实跑过的 verify 命令输出。失败也贴。>
$ npm test -- theme/ThemeContext.test.tsx

 PASS  src/theme/__tests__/context.test.ts
  ThemeContext
    ✓ 默认主题来自 prefers-color-scheme (12 ms)
    ✓ 切换后写入 localStorage (4 ms)
    ✓ 重新挂载从 localStorage 恢复 (3 ms)

Tests: 3 passed, 3 total
```

## 6 维自查（生产代码改动必填 · 来自 4-dev 步骤 4）

> 装了 [brooks-lint](https://github.com/hyhmrright/brooks-lint) → `/brooks-review` 输出原样贴入；未装 → AI 内置 R1~R6 快查结果。

```markdown
<贴 brooks-review 输出，或内置 6 维快查结果，4 要素格式：
### 🔴/🟡/🟢 R<x> · <名字>：<结论>
**Symptom**：...
**Source**：...
**Consequence**：...
**Remedy**：...
>
```

### 已知接受 + 理由（🟡 Major 项不修的）

- <列出本次未修的 Major 项 + 理由 + 计划何时修>

### 已知小问题（🟢 Minor 项可省的）

- <列出本次不修的 Minor 项，记入即可>

## 数据库迁移（涉及 schema 变更必填 · 来自 4-dev 步骤 1.7 / R4.5）

> 没有 schema 变更可整段写 N/A。

### Schema Diff

```
- 新增表：notifications（id BIGSERIAL PK, user_id BIGINT FK, type VARCHAR(32), payload JSONB, read_at TIMESTAMPTZ NULL, created_at TIMESTAMPTZ DEFAULT now()）
- 新增索引：idx_notifications_user_id_unread ON notifications(user_id) WHERE read_at IS NULL
- 改字段：users.email_verified BOOLEAN NULL → NOT NULL DEFAULT FALSE（含 backfill：UPDATE users SET email_verified=FALSE WHERE email_verified IS NULL）
```

### 迁移文件

| 文件 | 框架 | 含 up | 含 down |
|---|---|---|---|
| `prisma/migrations/20260501120000_add_notifications/migration.sql` | Prisma | ✅ | ✅ |

### 执行状态

- [ ] **已检测 DB 凭据**：是 / 否（grep 结果：`<file:line>` 或「未找到」）
- [ ] **已反问用户**（找到凭据时必填）：用户选择 = 现在执行 / 只生成 SQL / 看 SQL 再决定
- [ ] **执行结果**（用户选"现在执行"时填）：
  ```
  $ npx prisma migrate dev --name add-notifications
  <贴真实输出>
  ```
- [ ] **结构验证**（用户选"现在执行"时填）：
  ```
  $ psql -c "\d notifications"
  <贴 \d 输出，确认表 / 字段 / 索引落地>
  ```

### 提醒（用户选"只生成 SQL" 或未检测到凭据时必填）

```
 数据库迁移文件已生成，但未在任何环境执行。
  - 文件：prisma/migrations/20260501120000_add_notifications/migration.sql
  - 执行命令：npx prisma migrate deploy
  - 验证命令：psql -c "\d notifications"
  - 环境清单（执行后逐项打勾）：
    - [ ] local
    - [ ] dev
    - [ ] staging
    - [ ] prod（生产请走 maintenance window，先 dry-run）
```

## 越界检查（必填 · 来自 4-dev 步骤 5 / R6.5 / B3 老项目护栏）

```
✅ 越界检查（R6.5）：
  - TASK write_files：<N> 项
  - 实际 diff 涉及：<M> 项
  - 越界：0 / 已撤销 / 已扩范围
```

如有越界，列出处理方案：

- [ ] 已撤销的越界文件：<list>
- [ ] 已扩范围（须人工同意）：<list> + 同意人 + 时间
- [ ] 拆成新 task / 新 CHANGE：<task / CHANGE id>

## 破坏性变更（涉及破坏性改动必填 · 来自 4-dev 步骤 1.8 / R4.6 / B4 老项目护栏）

> 不涉及破坏性变更可整段写 N/A。

### 判定命中

- [ ] 删除既有代码 ≥ 5 行
- [ ] 改公共导出（函数 / 类 / 接口签名）
- [ ] 改公共 API（HTTP / GraphQL / gRPC 路由 / schema）
- [ ] 删除文件 / 重命名导出符号

### 引用图（必填 · grep 结果）

```
被删/改的符号：<symbol> (<file:line>)
直接调用：
  - <file:line>
  - <file:line>
间接影响：
  - <chain>
```

### 反问用户结果

- 用户选择：现在执行 / 留兼容期 / 写 codemod / 不删了
- 同意人：<>
- 时间：<>

### 回归测试覆盖

- 旧路径测试位置：`<test-file>`
- 新路径测试位置：`<test-file>`（如改公共 API）
- 覆盖率证据：<coverage 输出>

## 决策与偏离（如有）

- <若实际实现与 TASK 中 action 有偏离，说明原因>
- <若引入了未在 DESIGN 中提到的依赖或模式，说明原因>

## 是否触发新工作

- [ ] 触发新 fix-plan（已追加到 TASK.md）
- [ ] 触发 CONTEXT.md 更新（已更新）
- [ ] 发现需求/设计问题，已暂停并提交给人工

## 完成判定

- TASK.md 中对应任务已勾选：是
- 提交 hash：`<commit-sha>`
