# STATE — 跨会话项目状态

> 仓库根 `STATE.md`。AI 每次开始新会话先读这个，结束前更新。

---

## 当前位置

- **活跃 Change**: `<change-id>` 或 "无"
- **当前阶段**: CHANGE / REQUIREMENT / DESIGN / TASK / DEV / TEST / REVIEW / INTEGRATION / 无
- **当前 Task**: `<T03>` 或 "—"
- **中断任务**（来自 R1.5 清窗）: `<task-id>` 或 "无"
  - 若非空，下一会话必须先加载 `@.specs/<id>/<task-id>-PROGRESS.md`
- **会话开始建议**:
  - 若有「中断任务」：按 R1.5 清窗后加载顺序恢复
  - 若有活跃 change：先读 `@.specs/<id>/` 下相关产物
  - 若无：等用户给新 CHANGE

## 阻塞与待决策

| 项 | 类型 | 详情 | 待谁 | 自 |
|---|---|---|---|---|
|  | bug / 决策 / 范围 |  | 用户 |  |

## 决策日志（最近 10 条，倒序）

- `[2026-04-30]` <一句话决策> — `@.specs/<id>/DESIGN.md#D2`
- ……

## 已归档 Changes（最近 5 个，倒序）

| 日期 | Change | 摘要 | PR |
|---|---|---|---|
| 2026-04-30 | add-dark-mode | 主题切换支持 + 持久化 | #42 |

---

## 横向命令状态（GO.md / 各 prompt 会读这里决定要不要主动提示）

```yaml
# 老项目入场扫描 / AI 上下文文档替代
ai_context_doc: .specs/CONTEXT.md   # 或 AGENTS.md / CLAUDE.md / 用户指定路径 / none
last_intel_scan: 2026-04-15          # 上次跑 I-intel-scan 的日期；为空表示从未跑
                                     # GO.md 第三步会按 90 天阈值提醒重扫

# 架构演进
last_architect_at: 2026-03-01        # 上次跑 A-architect 的日期；首跑后填
last_evolve_at: 2026-04-20           # 上次跑 A-evolve 的日期；用于过滤扫描范围
last_evolve_promoted:                # 上次 A-evolve 已处理的 change-id 列表，避免重扫
  - add-cache-layer
  - add-bg-jobs
  - add-notifications

# 健康巡检
last_health_at: 2026-04-10           # 上次跑 M-health 的日期
last_health_score: 78                # 上次综合分（用于本次对比）
```

字段含义：

- 读这块只为**决定要不要在 GO.md 路由时主动提醒**（如"距上次 evolve 已 60 天，建议同步"）
- 不影响主流程，缺字段就当未跑过对应工作流
- 各横向工作流跑完会自己更新这里

---

## 健康检查（AI 每次会话开始前自检）

- [ ] `.specs/` 目录存在
- [ ] 当前活跃 change 的工件文件齐全（按当前阶段）
- [ ] 没有未归档但已合并的 change
- [ ] 没有 > 3 轮自动重试仍未解决的失败
