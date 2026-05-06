# TASK: <change 标题>

- **Change ID**: <id>
- **关联**: `@.specs/<id>/REQUIREMENT.md`、`@.specs/<id>/DESIGN.md`

---

## 波次划分

```
Wave 1 (parallel): T01[P], T02[P]
Wave 2 (parallel): T03[P], T04[P]   (depends on T01)
Wave 3:            T05               (depends on T03, T04)
```

> 同 wave = 可并行；跨 wave = 必须顺序执行。

---

## 任务清单

```xml
<task id="T01" parallel="true" status="pending">
  <name><一句话任务名></name>
  <read_files>
    <参考边界 · 允许 read 的文件，支持 glob>
    <例：src/theme/*>
    <例：src/lib/api-client.ts>
  </read_files>
  <write_files>
    <修改边界 · 严格控制，超出会被 R6.5 提交前 verify 拦住>
    <例：src/theme/ThemeContext.tsx>
    <例：src/theme/__tests__/ThemeContext.test.tsx>
  </write_files>
  <action>
    <做什么。写意图，不写代码。可以包含设计要点引用：见 D1。
     如沿用既有抽象，必须显式说明：「沿用 src/utils/storage.ts 的 safeStorage」>
  </action>
  <verify>
    <一条可执行验证命令，例如 npm test -- theme.test.ts>
  </verify>
  <done>
    <完成判定。一句话，对应 AC 的某个子项，例如 AC-1>
  </done>
  <depends_on></depends_on>
</task>

<task id="T02" parallel="true" status="pending">
  <name>……</name>
  <read_files>……</read_files>
  <write_files>……</write_files>
  <action>……</action>
  <verify>……</verify>
  <done>……</done>
  <depends_on></depends_on>
</task>

<!-- 其他任务继续追加 -->
```

> **注意**：`read_files` 和 `write_files` 是 R7.3 强约束。`write_files` 必须**严格在 DESIGN `## 0.5.1` 「触碰模块 + 新增模块」范围内**，且**不能包含「禁动清单」**。否则 4-dev 步骤 5 提交前 verify 会 fail。

---

## 状态字段说明

- `status="pending"` — 未开始
- `status="in_progress"` — 进行中（同时只允许一个非 [P] 任务为此状态）
- `status="done"` — 已完成（verify 通过）
- `status="blocked"` — 阻塞（必须在文件末尾「阻塞日志」记录）

---

## 阻塞日志

| 任务 | 阻塞原因 | 待人工决策项 | 时间 |
|---|---|---|---|
|  |  |  |  |

---

## Fix 任务（来自 REVIEW / INTEGRATION）

> 此区域由 review/integration 阶段自动追加，编号 `T-FIX-XX`。

```xml
<!-- 占位 -->
```
