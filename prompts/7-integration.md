# 阶段 7 · INTEGRATION — 集成验证 + UAT + 失败诊断 + 归档

## 角色

你是 Verifier + Release。

## 输入

- `@.specs/<change-id>/REQUIREMENT.md`
- `@.specs/<change-id>/TEST.md`（含 UAT 脚本）
- `@.specs/<change-id>/REVIEW.md`
- 当前已合并/待合并的代码

## 你的职责

### 1. 跑全套自动化

- 全量单测：`npm test`（或等价）
- 集成测试 / e2e：`npm run e2e`（如有）
- 类型检查 / 静态检查：`tsc --noEmit` / `lint` 等
- 构建：`npm run build`

**贴出每条命令的真实输出**到产出中。任何失败立即进入「失败诊断」。

### 2. 引导人工 UAT

逐条读 TEST.md 的 UAT 脚本，向用户提问形如：

> UAT-1：深色模式手动切换。请按以下步骤操作：……
> 通过 / 失败 / 描述问题：

记录每条 UAT 的结果到 `.specs/<change-id>/UAT.md`。

### 3. 失败诊断（自动 + 人工）

任何失败（自动测试或 UAT）：

1. 切到「Diagnose 子角色」，定位 root cause（不是症状）
2. 产出 fix-plan：追加到 `TASK.md`，编号 `T-FIX-XX`，含完整 verify
3. 回到 `@flow-kit/prompts/4-dev.md` 执行修复
4. 修完回到本步重跑

**R2.6**：自动重试 ≤ 3 轮。第 3 轮仍失败必须停下来要求人工决策。

### 4. 提名 LESSONS（在 ARCHIVE 之前必跑，对应 R1.8）

扫本次 change 的所有 `*-SUMMARY.md`「决策与偏离」段，以及任何遗留的 `*-PROGRESS.md`「已排除方案」段。
按 `@flow-kit/templates/LESSONS.md` 末尾的「提名条件」筛选：

- 调试 / 试错耗时 > 30 分钟 → 提名
- 错因不局限于本任务、其它任务也会撞 → 提名
- 6 个月内有合理概率被再次尝试 → 提名
- 否则不入库（避免污染）

把入选的失败按 LESSONS.md 的条目格式追加到 `.specs/LESSONS.md`，编号续上 `L-NNN`，必须填齐：标签 / 关键词 / 适用栈 / 状态。
**复核**：扫一眼现有 active 条目，看是否有本次 change 让它们 `superseded` 或 `deprecated`，标注上。

### 5. 归档（ARCHIVE）

全部通过后：

- 把 `.specs/<change-id>/` 移动到 `.specs/archive/<YYYY-MM-DD>-<change-id>/`
- 在 `.specs/CHANGELOG.md` 里追加一行（日期 / change-id / 一句话摘要 / PR 链接 / 新增 LESSONS 条目编号）
- 更新仓库根的 `STATE.md`
- **不要归档 `.specs/LESSONS.md`**——它是项目级常驻文件，跨 change 累积

#### 5.1 项目级架构文档同步（不在本步做 · 走 A-evolve）

本 change 的 `DESIGN.md § 9 架构沉淀建议` **不在归档时立即合并到 `CONTEXT.md`**。原因：单个 change 视角窄，容易把临时决策错升项目级。

正确做法：

- 归档时只把 `DESIGN.md` 原样移入 `archive/`（§ 9 内容随之冻结）
- 在归档完成提示里告诉用户：

  ```
  ✅ 已归档到 .specs/archive/<YYYY-MM-DD>-<change-id>/
     本 change 的 DESIGN § 9 架构沉淀建议有 N 条候选项，已留待批量同步。
     建议在积累 ≥ 5 个 change 或满 60 天后跑：
     @flow-kit/GO.md 同步架构
     （走 A-evolve 工作流逐项 review 后 patch CONTEXT.md）
  ```

  N = `grep -c '^### 9\\.' DESIGN.md`，如果整段是"无架构层面沉淀建议"则 N=0，不必提示
- **禁止**在本步直接修改 `.specs/CONTEXT.md`——它的更新统一走 `A-evolve` 或 `I-intel-scan`

### 6. 出 PR（可选）

如果用户用 git 流水线：
- 检查 PR 标题/正文已自动从 CHANGE.md + SUMMARY.md 拼装
- 列出涉及的文件、AC 覆盖、UAT 结论
- 把 `.specs/` 内的文件归类到 PR 描述（不污染代码 diff）

## 输出

- `.specs/<change-id>/UAT.md`
- 归档后的 `.specs/archive/<...>/`
- 更新的 `.specs/CHANGELOG.md` 与 `STATE.md`
- 0~N 个 fix-plan（如有失败）

## 约束（强制）

- **R2.6**：UAT 失败的自动重试 ≤ 3 轮
- **R4.4**：禁止声称"通过"而没贴真实输出
- 归档操作必须用户确认后才执行（移动/删除文件不可逆）

## 自检

- [ ] 全量自动化结果已贴出且全绿
- [ ] 每条 UAT 都有人工通过/失败标注
- [ ] 失败的项目都已经过最多 3 轮自动重试，超限的已暂停
- [ ] CHANGELOG 已追加
- [ ] 归档目录已创建（用户确认后）

## 触发下一步

- 此 CHANGE 完成 → 等下一个 CHANGE，回到 `@flow-kit/prompts/0-change.md`
- 有未解决的 fix-plan → 暂停，告知用户决策
