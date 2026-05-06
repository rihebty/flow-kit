# TEST: <change 标题>

- **Change ID**: <id>
- **关联**: `@.specs/<id>/REQUIREMENT.md`、`@flow-kit/reference/test-pyramid.md`
- **项目类型**: <Web 前端 / 后端 API / 全栈 / 移动 / CLI / 库>

---

## 0. 本次测试范围声明（5 轮金字塔）

| 轮次 | 状态 | 范围 | 跳过理由 |
|---|---|---|---|
| 第 1 轮 · 功能 | ✅ 必跑 | 全部 AC | — |
| 第 2 轮 · 性能 | ✅ / ⚠️ / ❌ | <Lighthouse / load test / bundle> | <如跳过填理由> |
| 第 3 轮 · 安全 | ✅ / ⚠️ / ❌ | <依赖 / 秘钥 / SAST / OWASP> | — |
| 第 4 轮 · 兼容 | ✅ / ⚠️ / ❌ | <浏览器 / 数据迁移 / 跨版本> | — |
| 第 5 轮 · 可观测 | ✅ / ⚠️ / ❌ | <日志 / 指标 / 告警> | — |

---

## 第 1 轮 · 功能测试

### 1.1 测试矩阵（AC → 用例）

| AC | 类型 | 用例文件 / UAT | 状态 |
|---|---|---|---|
| AC-1 | unit | `src/theme/__tests__/context.test.ts` | ✅ |
| AC-2 | integration | `src/theme/__tests__/persist.test.ts` | ✅ |
| AC-3 | e2e | `e2e/theme.spec.ts` | 🟡 |
| AC-4 | manual | UAT-1 | — |

### 1.2 UAT 脚本

#### UAT-1 · <场景>

- **前置**: <例 清除 localStorage>
- **步骤**: 1. ... 2. ... 3. ...
- **期望**: - ... - ...
- **实际**: 通过 / 失败 / 描述
- **执行人 / 时间**:

### 1.3 覆盖率

```text
<贴出 --coverage 输出>
```

- 当前：<行覆盖 %>
- 门槛：<默认 80% / core 90%>
- 不达项原因：

### 1.4 边界 / 错误路径用例

- 空 / null / undefined：
- 极大 / 极小：
- Unicode / 特殊字符：
- 错误路径（异常 / 失败）：

### 1.5 测试质量自检（6 维测试衰退风险）

> 装了 [brooks-lint](https://github.com/hyhmrright/brooks-lint) → `/brooks-test` 输出原样贴入；未装 → AI 内置 T1~T6 快查。

**严重度统计**：

| 编号 | 测试衰退风险 | 命中文件数 | 严重度分布 |
|---|---|---|---|
| T1 | Test Obscurity 测试晦涩 | 0 | 🔴 0 / 🟡 0 / 🟢 0 |
| T2 | Test Brittleness 测试脆弱 | 0 | 🔴 0 / 🟡 0 / 🟢 0 |
| T3 | Test Duplication 测试重复 | 0 | 🔴 0 / 🟡 0 / 🟢 0 |
| T4 | Mock Abuse Mock 滥用 | 0 | 🔴 0 / 🟡 0 / 🟢 0 |
| T5 | Coverage Illusion 覆盖率幻觉 | 0 | 🔴 0 / 🟡 0 / 🟢 0 |
| T6 | Architecture Mismatch 架构错配 | 0 | 🔴 0 / 🟡 0 / 🟢 0 |

**详细发现**：

```markdown
<贴 brooks-test 输出 / 内置 T1~T6 快查结果，4 要素格式：>

### 🔴 T<x> · <名字>：<结论>
**Symptom**：<test-file:line>
**Source**：<书名 · 章节>
**Consequence**：<会怎样>
**Remedy**：<怎么改>
```

**处理**：
- 命中 ≥ 1 项 → 记下面「测试质量记事」段
- 命中 ≥ 3 项 → release 前必修

### 1.6 测试质量记事（backlog）

| 文件 | 维度 | 严重度 | 计划修复时间 |
|---|---|---|---|
|  |  |  |  |

---

## 第 2 轮 · 性能测试

> 跳过则整段标 N/A 并说明理由。

### 2.1 性能预算（来自 REQUIREMENT.md 非功能性需求）

```yaml
frontend:
  LCP_p75: < 2.5s
  CLS_p75: < 0.1
  main_bundle: < 200KB gzip
backend:
  api_p95:
    "/login": < 200ms
    "/checkout": < 500ms
  error_rate: < 0.1%
```

### 2.2 实测结果

| 指标 | 预算 | 实测 | 上版基线 | 判定 |
|---|---|---|---|---|
| LCP_p75 | < 2.5s | 1.8s | 1.7s | ✅ 达标 |
| 主包 gzip | < 200KB | 187KB | 174KB | ⚠️ 接近 |
| `/checkout` p95 | < 500ms | 612ms | 480ms | ❌ 退步 27% |

### 2.3 工具输出

```text
<贴 Lighthouse / k6 / bundle-analyzer 报告>
```

### 2.4 退步项处理

- ❌ `/checkout` 退步 → 已开 fix 任务 T-FIX-XX

---

## 第 3 轮 · 安全测试

### 3.1 依赖漏洞

```bash
$ npm audit --production
<输出>
```

- High / Critical：<数量>
- 处理：<修了哪些 / 接受哪些 + 理由>

### 3.2 秘钥扫描

```bash
$ trufflehog filesystem .
<输出>
```

- 命中：<0 / N>
- 已 rotate：<是 / 否 / N/A>

### 3.3 SAST

- 工具：<Semgrep / CodeQL / Bandit>
- High：<数量 + 处理>
- Medium：<处理记录>

### 3.4 OWASP Top 10

| 项 | 状态 | 备注 |
|---|---|---|
| A01 越权 | ✅ / ❌ / 🟡 | |
| A02 加密失败 | | |
| A03 注入 | | |
| A04 不安全设计 | | |
| A05 配置错误 | | |
| A06 漏洞组件 | | 见 3.1 |
| A07 鉴权 | | |
| A08 数据完整性 | | |
| A09 日志监控 | | 见第 5 轮 |
| A10 SSRF | | |

---

## 第 4 轮 · 兼容性测试

### 4.1 跨浏览器（Web）

| 浏览器 | 版本 | 桌面 | 移动 | 状态 |
|---|---|---|---|---|
| Chrome | 最新 -1 | ✅ | ✅ | |
| Firefox | 最新 -1 | ✅ | — | |
| Safari | 最新 -1 | ✅ | ✅（iOS）| |
| Edge | 最新 -1 | ✅ | — | |

### 4.2 视口

| 视口 | 状态 |
|---|---|
| 360 (mobile) | ✅ |
| 768 (tablet) | ✅ |
| 1024 (laptop) | ✅ |
| 1440 (desktop) | ✅ |

### 4.3 数据迁移（涉及 schema 变更必填）

- [ ] 生产数据快照预演通过
- [ ] 实测耗时：<X 分钟>
- [ ] 回滚脚本就位且测过
- [ ] 灰度 / 双写方案验证通过

### 4.4 跨版本

- [ ] 旧 schema 数据兼容
- [ ] API v1 client → v2 server
- [ ] 编码 / locale

---

## 第 5 轮 · 可观测性验证

### 5.1 日志

- [ ] 关键路径有 log（入口 / 出口 / 异常）
- [ ] 含 trace-id
- [ ] 结构化 JSON
- [ ] grep 验证不含 PII / 秘钥 / token
- [ ] 错误日志上下文充分

### 5.2 指标

- [ ] 业务 metric：<列出>
- [ ] RED 指标覆盖关键 endpoint
- [ ] USE 指标覆盖关键资源

### 5.3 链路追踪

- [ ] 跨服务 trace 串通
- [ ] 慢操作能被定位

### 5.4 告警 + 健康检查

- [ ] 关键失败有告警
- [ ] 每个告警有 runbook 链接
- [ ] `/health` 区分 liveness / readiness
- [ ] 无噪音告警（命中率 > 50%）

---

## 新增测试登记

| 用例文件 | 类型 | 覆盖 AC | 所属轮次 |
|---|---|---|---|
| `src/theme/__tests__/context.test.ts` | unit | AC-1 | 1 |
|  |  |  |  |

## 回归保护

本次变更可能影响的旧功能：

- ……

对应已有测试是否仍通过：✅ / ❌ / 详情链接
