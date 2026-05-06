# DESIGN: <change 标题>

- **Change ID**: <id>
- **关联**: `@.specs/<id>/REQUIREMENT.md`、`@.specs/CONTEXT.md`、`@flow-kit/reference/tech-stacks.md`
- **作者**: AI（Architect 角色）+ 人工 review

---

## 0. 技术栈选定

> 由 2-design 步骤 0 锁定。变栈视为开新 CHANGE（R7.1）。

- **选定**：<编号 + 名称>（例：1 Next.js 全栈）
- **前端**：<具体框架 + 版本>（例：Next.js 14.2 / React 18 / TypeScript 5）
- **后端**：<具体框架 + 版本>（例：Next.js API Routes / Server Actions）
- **数据库**：<选型 + 版本>（例：Postgres 16 via Supabase）
- **部署**：<平台>（例：Vercel + Supabase）
- **关键依赖**：<3~5 个核心>（例：Tailwind v4 / Drizzle ORM / next-auth）
- **理由**：<一句话，结合 AC + 非功能性需求>
- **明确排除**：<1~2 个不选的栈 + 理由>

---

## 0.5 既有架构对齐（brownfield 必填 · 来自 2-design 步骤 0.5 / B2 老项目护栏）

> 新创项目可整段写 N/A。

### 0.5.1 本次 change 触碰的既有模块

```
触碰模块（grep 出来的实际清单）：
- src/services/user-service.ts（既有）
- src/api/auth/*（既有路由组）
- src/components/Modal.tsx（既有 · 复用）

新增模块：
- src/features/notifications/*

禁动清单（与本次无关，AI 不许"顺手"碰）：
- src/services/payment-service.ts
- src/components/Layout.tsx
- src/api/admin/*
```

### 0.5.2 既有抽象沿用对照表

| 本次需要 | 既有有没有？路径 | 决定 |
|---|---|---|
| HTTP 客户端 | `src/lib/api-client.ts` | 沿用 |
| 状态管理 | zustand（package.json） | 用现有 store 范式 |
| 表单校验 | `src/utils/validation.ts` | 沿用 |
| 通知组件 | 没有 | 新建（理由：第一次有此需求）|

### 0.5.3 沿用模式 vs 引入新模式

```
- 数据访问：**沿用** Repository 模式（既有 src/repos/* 都是这风格）
- 错误处理：**沿用** ErrorBoundary + toast
- API 路由：**沿用** src/api/<domain>/route.ts 风格
- 通知存储：**引入新模式** → 理由：通知是新业务域，独立 service + repo（已批准）
```

---

## 1. 决策清单

| # | 决策 | 备选 | 选择理由 | 取舍代价 |
|---|---|---|---|---|
| D1 | <例：使用 React Context 管理主题> | Redux / Zustand | <为什么选 Context> | <跨远距离传递性能不优，本场景不存在> |
| D2 | …… | | | |

## 2. 数据流 / 架构图

```
<ASCII 框图或 Mermaid，例：>

  User ──click──> ThemeToggle
                      │
                      v
                ThemeContext (state)
                      │
                      ├──> document.documentElement (data-theme)
                      └──> localStorage
```

## 3. 关键状态机（如有）

<状态/事件/转移列表>

## 4. ADR 索引

凡 D1~Dn 中可逆性低的，单独写 ADR：
- `@.specs/adr/001-theme-storage.md`
- ……

## 5. 风险

| # | 风险 | 影响 | 概率 | 缓解 |
|---|---|---|---|---|
| R1 | <例：localStorage 在隐私模式不可用> | 主题不持久 | 中 | 降级到 sessionStorage + 友好提示 |
| R2 | …… | | | |

> 至少 3 条；含实现风险 / 上线风险 / 长期债务各一条。

## 6. 不在范围

- <列出"这次设计不解决但未来要解决"的事项，避免 TASK 阶段误以为遗漏>

---

## 9. 架构沉淀建议（本 change 完成后供 `A-evolve` 同步用 · 软约束）

> 这是**给未来的礼物**，不是必填。AI 写 DESIGN 时如果发现本 change 引入了"项目级有复用价值"的东西，列在这里。
> 没引入新东西就整段写 `本 change 无架构层面沉淀建议`。
> 后续用户跑 `@flow-kit/prompts/A-evolve.md` 会扫这段，逐项 review，批准的内容 patch 到 `.specs/CONTEXT.md`。

### 9.1 新增的可复用抽象（建议 append 到 CONTEXT 「既有抽象索引」段）

| 路径 | 能力 | 触发场景 | 复用建议 |
|---|---|---|---|
| `<例：src/lib/cache.ts>` | <例：通用 LRU 缓存封装> | <例：API 响应缓存> | <例：以后所有需要缓存的地方先用它> |

### 9.2 新增 / 改变的项目级技术决策（建议 append 到 CONTEXT「已锁技术决策」段 · 或将来升 ARCHITECTURE.md「ADR 列表」）

| 决策 | 取值 | 影响范围 | 推翻代价 |
|---|---|---|---|
| <例：缓存层选型> | <例：Redis（替代 in-memory）> | <例：所有需要缓存的服务> | <例：迁移成本高，至少 1 周> |

### 9.3 新增 / 修改的跨模块契约（API / Schema / 事件总线）（建议 append 到 CONTEXT「跨模块契约」段）

```
- <例：新增 GET/POST /api/cache/* 一组路由，认证沿用 session>
- <例：events.cache.invalidated 事件，payload: { keys: string[] }>
- <例：表 cache_entries 主键 (key, namespace)，TTL 字段必填>
```

### 9.4 新增 / 升级的依赖（建议 append 到 CONTEXT「技术栈」段）

| 包 | 版本 | 用途 | 是否替换既有 |
|---|---|---|---|
| <例：ioredis> | <例：5.4.0> | <例：Redis 客户端> | <例：是 → 替换原本的 node-redis> |

### 9.5 禁动清单变化（建议 patch CONTEXT「禁动清单」段）

```
- 新增禁动：<例：src/lib/cache.ts 不允许绕过直接 import ioredis>
- 解禁：<例：原 src/legacy/cache-old.ts 标记 deprecated 可拆，由本 change 替代>
```

---

> 本文件不包含完整代码实现。函数签名、伪代码、接口定义可以；函数体不行。
