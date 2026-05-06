# REVIEW: <change 标题>

- **Change ID**: <id>
- **审查时间**: <YYYY-MM-DD HH:mm>
- **审查者**: AI（Reviewer 角色）+ 跨模型 spot-check（如有）
- **总体结论**: 通过 / 需修复 / 阻塞

---

## 第一轮 · Spec 合规审查

| 检查项 | 结果 | 证据 |
|---|---|---|
| 每条 AC 都已实现 | ✅ / ❌ | <文件:行 或 测试名> |
| 每条 AC 都有测试 | ✅ / ❌ | `@.specs/<id>/TEST.md#AC-N` |
| 未引入 `out of scope` 内容 | ✅ / ❌ | |
| 未范围蔓延（无 REQUIREMENT 外的功能） | ✅ / ❌ | |
| 未越过 DESIGN 边界 | ✅ / ❌ | |

**Spec 合规结论**: 通过 / 不通过（需修复后重审）

---

## 第二轮 · 代码质量审查（6 维衰退风险）

> 装了 [brooks-lint](https://github.com/hyhmrright/brooks-lint) → `/brooks-review` + `/brooks-audit` 输出原样贴入；未装 → AI 内置 R1~R6 诊断。

### 2.0 TEST.md 5 轮金字塔完整性

| 轮次 | 状态 | 缺漏 |
|---|---|---|
| 1 功能 | ✅ / ❌ | |
| 2 性能 | ✅ / ❌ / N/A | |
| 3 安全 | ✅ / ❌ / N/A | |
| 4 兼容 | ✅ / ❌ / N/A | |
| 5 可观测 | ✅ / ❌ / N/A | |

### 2.1 6 维诊断 · 严重度统计

| 编号 | 衰退风险 | 🔴 | 🟡 | 🟢 |
|---|---|---|---|---|
| R1 | Cognitive Overload 认知过载 | 0 | 0 | 0 |
| R2 | Change Propagation 变更传播 | 0 | 0 | 0 |
| R3 | Knowledge Duplication 知识重复 | 0 | 0 | 0 |
| R4 | Accidental Complexity 偶然复杂 | 0 | 0 | 0 |
| R5 | Dependency Disorder 依赖混乱 | 0 | 0 | 0 |
| R6 | Domain Model Distortion 领域扭曲 | 0 | 0 | 0 |

### 2.2 6 维诊断 · 详细发现（4 要素 · 来自 brooks-lint 输出 / 内置回退）

```markdown
<贴 brooks-review 输出原文 / 内置 6 维诊断结果，格式：>

### 🔴 R<x> · <名字>：<结论>
**Symptom**：<file:line 的具体问题>
**Source**：<书名 · 章节>
**Consequence**：<不修会怎样>
**Remedy**：<具体怎么改>
**生成 fix 任务**：T-FIX-NN
```

### 2.3 架构依赖图（大型 change · 来自 /brooks-audit）

```mermaid
<贴 brooks-audit 输出 / 内置简化图>
```

**循环依赖**：有 / 无（如有列出）
**反向依赖**：有 / 无（如有列出）

---

## 第三轮 · UI 视觉审查（仅前端 · 见 6-review.md 第 3 节）

（仅前端项目填）

---

## 第四轮 · 补充审查（按触发条件）

### 4.1 技术债评估（来自 /brooks-debt，仅里程碑 / 重构）

未触发 / 已跑（贴输出）

### 4.2 跨模型分歧

| 主审发现 | 跨模型发现 | 是否一致 | 处理 |
|---|---|---|---|
| F-1 | F-1' | 一致 | 采纳 |
| —  | F-X' | 仅跨模型提出 | 评估后纳入 / 拒绝（理由：…）|

---

## 总结

- Critical 项：<数量>，已全部生成 fix 任务
- Major 项：<数量>，已修：<>，已知接受：<>（需人工签字）
- Minor 项：<数量>，建议下次顺手优化

**下一步**: 进入 `4-dev` 修复 Critical / 进入 `7-integration`（取决于结论）
