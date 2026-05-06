# Forge runtime adapter（可选）

flow-kit 默认仍然是纯 markdown：不需要 CLI、不需要安装运行时，也不绑定任何外部工具。Forge 是一个可选 runtime adapter，适合那些希望在 Claude Code 里防止 AI 跳阶段、漏产物、漏测试或漏 review 的项目。

## 它解决什么

flow-kit 定义“应该怎么做”：阶段、prompt、template、产物和 review 规则。

Forge 检查“实际有没有做到”：通过 Claude Code hooks、routing log、session state、health check 和 smoke test，在运行时检查当前阶段是否允许写文件、是否缺关键产物、是否缺测试或 review 证据。

## 安装关系

Forge 不应该成为 flow-kit 的依赖。

- 没装 Forge：flow-kit 行为不变，继续按 `@flow-kit/GO.md` 和 `prompts/*.md` 使用。
- 装了 Forge：AI 可以把 flow-kit 的阶段、change-id、task-id 和风险信息写入 Forge state/routing，供 runtime guard 使用。
- Forge 缺失或不可用：必须 fail-open 到纯 markdown 流程，不应阻断 flow-kit。

## 检测方式

在项目根目录里检测以下文件：

```text
flow-kit/GO.md
flow-kit/prompts/4-dev.md
flow-kit/templates/TASK.md
.claude/hooks/forge-pretool-guard.ps1
.claude/hooks/forge-session-audit.ps1
```

只有前 3 个存在时，说明项目使用 flow-kit；同时存在 Forge hooks 时，说明可以启用 Forge runtime adapter。

## 阶段映射

| flow-kit 阶段 | Forge 用途 |
|---|---|
| `0-change` | 记录新 change、风险、是否需要完整流程 |
| `1-requirement` | 标记需求产物生成中，禁止直接跳到实现 |
| `2-design` / `2a-ui-design` | 记录设计产物、架构/UI 风险 |
| `3-task` | 记录 task 拆分完成，作为 `4-dev` 的前置证据 |
| `4-dev` | 进入写文件阶段，检查 task-id、routing started、测试计划 |
| `5-test` | 记录测试证据 |
| `6-review` | 记录 review/self-review 证据 |
| `7-integration` | 记录集成、归档和 lessons |
| `M-health` | 可映射到 Forge health/smoke 的只读检查 |

## 建议写入的 routing 字段

Forge adapter 不要求 flow-kit 改变产物格式。推荐只写一条轻量 routing event：

```json
{
  "adapter": "flow-kit",
  "change_id": "<change-id>",
  "stage": "4-dev",
  "task_id": "T01",
  "group_status": "started",
  "risk": "normal",
  "todo_ref": "TodoWrite:<id>",
  "test_ref": "unit:<path-or-command>",
  "review_ref": "self-review:<summary>"
}
```

这些字段只给 Forge runtime gate 使用，不替代 `.specs/<change-id>/*.md` 工件。

## 使用建议

只有在这些场景建议接入 Forge：

- 团队项目或长期维护项目。
- 高风险变更：鉴权、支付、数据库、schema、公共契约、发布流程。
- 发现 AI 经常跳过 `3-task`、`5-test`、`6-review`。
- 希望 Claude Code 写文件前有 fail-close 门禁。

小改动、一次性脚本、纯讨论和非 Claude Code 环境，继续用纯 markdown flow-kit 即可。
