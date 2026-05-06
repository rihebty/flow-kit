# CONTEXT — 项目共享上下文

> 本文件**跨 change 长期累积**。每个 change 在 REQUIREMENT 阶段会向这里追加术语和决策。
> 目标：为 AI 提供项目级的「域语言 + 默认偏好」，省去重复解释。

---

## 项目概要

<3~5 句话讲清楚项目是什么、给谁、为什么存在。>

## 技术栈（团队级默认 / 已锁定）

> 这里写**全项目共用**的栈。每次 CHANGE 的 `DESIGN.md ## 0` 会读此处作为默认；如果某次 CHANGE 用了不同的栈（例如临时加个 Python 服务），那次 DESIGN.md ## 0 会显式覆盖。

- **语言/运行时**: <例如 TypeScript + Node 20>
- **前端框架**: <如有>
- **后端框架**: <如有>
- **数据库**: <如有>
- **测试**: <例如 vitest + playwright>
- **构建/部署**: <例如 Vite + Vercel>
- **栈卡片编号**（来自 `@flow-kit/reference/tech-stacks.md`）: <如 1 Next.js 全栈>

## 域语言（术语表）

| 术语 | 定义 |
|---|---|
| <例：物化级联> | <一个课程/小节被首次落地到文件系统时触发的连锁状态变化> |
| ……  | …… |

> 加新术语时只在右列写定义，不解释来历。

## 已锁决策

按时间倒序追加：

- `[YYYY-MM-DD]` <决策内容> — 来自 `@.specs/<change-id>/DESIGN.md`
- ……

## 默认偏好（AI 在缺省时按此决策）

- 命名风格：<例 camelCase 函数 + PascalCase 组件 + kebab-case 文件>
- 错误处理：<例 不抛异常给 UI，统一返回 Result 类型>
- 状态管理：<例 React Context 优先，Redux 仅在跨远距离共享时考虑>
- 测试策略：<例 行为驱动 > 实现细节；不允许用 spy 屏蔽真实 IO>
- 提交格式：<例 `<type>(<change-id>): <subject>`>

## 既有抽象索引（来自 I-intel-scan · 防 AI 重复实现 · B5 老项目护栏）

> intel-scan 自动 grep 出来的项目级抽象。每个 change 4-dev 1.4 步骤会查这里。

### HTTP 客户端

- **路径**：`<src/lib/api-client.ts>` 或 `未发现`
- **入口符号**：<例：apiClient / fetchJson / ApiClient class>
- **使用方式**：<例：import { apiClient } from '@/lib/api-client'; await apiClient.get('/users')>

### 数据库访问

- **模式**：<Repository / DAO / 直接 ORM / Prisma client / Active Record / 其他>
- **路径**：`<src/repos/* 或 src/db/*>`
- **示例**：`<src/repos/UserRepo.ts:1 含 findById / findByEmail / save>`

### 状态管理

- **库**：<zustand / redux / mobx / context-only / SWR / TanStack Query>
- **路径**：`<src/stores/* 或 src/state/*>`
- **示例**：`<src/stores/userStore.ts:1 通过 createStore 暴露 useUserStore>`

### 工具函数（utils / helpers）

| 工具类型 | 路径 | 入口符号 |
|---|---|---|
| 日期 | `<src/utils/date.ts>` 或 `未发现` | `formatDate / parseDate` |
| 字符串 | `<src/utils/string.ts>` 或 `未发现` | `slugify / truncate` |
| 校验 | `<src/utils/validation.ts>` 或 `未发现` | `validateEmail / validateUrl` |
| 存储 | `<src/utils/storage.ts>` 或 `未发现` | `safeStorage`（隐私模式兼容） |
| 错误 | `<src/utils/errors.ts>` 或 `未发现` | `AppError / NotFoundError` |

### 自定义 hooks（前端）

| 类别 | 路径 | 数量 |
|---|---|---|
| 数据 hooks（useFetch / useQuery） | `<src/hooks/data/*>` | <N> |
| UI hooks（useModal / useToast） | `<src/hooks/ui/*>` | <N> |
| 业务 hooks（useUser / useAuth） | `<src/hooks/*>` | <N> |

### 错误处理

- **前端**：`<src/components/ErrorBoundary.tsx>` 或 `未发现`
- **后端**：`<src/api/errorHandler.ts>` 或 `未发现`
- **通知**：`<toast 库：sonner / react-hot-toast / 自封装>`

### Schema / 迁移

- **工具**：<Prisma / Alembic / Knex / Flyway / Liquibase / Rails AR / 裸 SQL / 未引入>
- **路径**：`<prisma/schema.prisma 或 alembic/ 或 migrations/>`
- **建议**：<如未引入，标注「下次 schema 变更时建议引入 X」>

### 命名约定

- 文件命名：<例 kebab-case>
- 函数命名：<例 camelCase>
- 组件命名：<例 PascalCase>
- 测试文件：<例 `*.test.ts` 紧邻源码 / `__tests__/` 子目录>

### 禁动清单（AI 不许"顺手"碰）

> 这些是与新 change 通常无关、改坏会出事的高风险模块。每个 change 的 DESIGN 0.5.1 会复用这清单。

- `<src/services/payment-service.ts>`（支付，改动需走金融合规）
- `<src/components/Layout.tsx>`（全局布局，改动影响所有页面）
- `<src/api/admin/*>`（管理后台，权限敏感）
- `<其他...>`

**清理窗口专列**（来自 `M-health` 步骤 2.5 冗余巡检 · 下次清理窗口一起 remove）：

- `<lodash>` · 未用依赖 · 2026-04-30 health 报告标记 · 不要再引入
- `<moment>` · 未用依赖 · 项目已迁 date-fns · 不要再 import

### 技术债（来自 M-health · 给 AI 在 2-design / 4-dev 时参考，别再加同类债）

> 只记 🟡 Scheduled 和 🟡 🔴 未处理项。🔴 Critical 已通过 health-fix CHANGE 处理中，不在此列。

| 债项 | 来源 | 首次发现 | 影响 | 计划处理 |
|---|---|---|---|---|
| `src/api/auth.ts` 与 `src/legacy/auth.ts` 38 行重复 | 2026-04-30 health · 2.5 冗余 | 2026-04-30 | AI 改一处时可能忘同步 | 2026-Q3 抽公共函数 |
| `UserService` 既接受 Email 又接受 username · 职责模糊 | 2026-04-30 health · R6 | 2026-04-30 | 新增用法易误用 | 待 user-service 重构 |

---

## intel-scan 元数据

- **last_intel_scan**: `<YYYY-MM-DD>`
- **scanner**: `prompts/I-intel-scan.md`
- **下次重扫建议**: `<触发条件：架构重构 / 框架升级 / 主要新增模块 / > 90 天>`

---

> 此文件长度建议 ≤ 300 行（含 intel-scan 自动填的字段）；超出时把陈旧条目归档到 `.specs/archive/CONTEXT-history.md`。
