# 技术栈预选（完整工程模板卡片）

> `2-design.md` 步骤 0 在做 ADR / 架构图前，必须把本清单展示给用户选定一个**完整工程模板**。
> 每张卡片不只是"框架名"，是一套**可直接落地**的工程组合：后端（框架 + ORM + 缓存 + 鉴权 + 对象存储 + 队列）+ 前端（框架 + 构建 + UI 库 + 状态 + HTTP + 图表 + 表格）+ DevOps（部署 + 监控）。

---

## 10 张完整工程模板

### 1️⃣ Next.js 全栈（国际 SaaS / 内容站）

**定位**：一体化 React 全栈，SEO + Edge 性能首选。

| 层 | 技术选型 |
|---|---|
| **前端框架** | Next.js 14+ (App Router) · React 18 · TypeScript 5 |
| **构建** | Next.js 内置（Turbopack）|
| **样式** | Tailwind CSS v4 |
| **UI 组件库** | shadcn/ui · Radix UI primitives |
| **图标** | Lucide React |
| **状态管理** | Zustand（客户端）+ React Server State |
| **服务端状态 / 数据获取** | TanStack Query · Server Components |
| **HTTP** | fetch（内置 cache）+ tRPC（可选）|
| **表格** | TanStack Table |
| **图表** | Recharts · Tremor（dashboard 专用）|
| **表单** | React Hook Form + Zod |
| **后端** | Next.js Route Handlers · Server Actions |
| **ORM** | Drizzle（首选 · type-safe）/ Prisma |
| **数据库** | Postgres（Supabase / Neon / Vercel Postgres）|
| **缓存** | Vercel KV (Upstash Redis) · Edge Cache |
| **鉴权** | Auth.js (next-auth v5) / Clerk |
| **对象存储** | UploadThing · Cloudflare R2 / S3 |
| **队列** | Inngest · Trigger.dev |
| **邮件** | Resend |
| **监控** | Sentry · Vercel Analytics |
| **部署** | Vercel（首选）/ Cloudflare Pages |

- **关键词**：Edge / SSR / Server Actions / 一体化
- **适合**：B2B SaaS · 内容 / 营销站 · 全栈一人开发 · 国际化项目
- **不适合**：超高并发后端 · 复杂业务逻辑独立部署 · 团队无 React 经验
- **典型 RTT 起步**：1~2 周可上线 MVP

---

### 2️⃣ Vite + React + Hono / NestJS（前后端分离 SPA）

**定位**：前后端独立部署 / 独立扩展，团队前后分工明确。

| 层 | 技术选型 |
|---|---|
| **前端框架** | Vite 5 · React 18 · TypeScript |
| **路由** | React Router v6 / TanStack Router |
| **样式** | Tailwind CSS |
| **UI 组件库** | shadcn/ui · Radix |
| **状态管理** | Zustand · Jotai |
| **服务端状态** | TanStack Query · SWR |
| **HTTP** | Axios · ofetch |
| **表单** | React Hook Form + Zod |
| **图表** | Recharts · Apache ECharts |
| **表格** | TanStack Table · AG Grid（企业版）|
| **后端框架** | Hono（首选 · Edge 友好）/ NestJS（企业级）/ Express |
| **ORM** | Drizzle / Prisma / TypeORM |
| **数据库** | Postgres / MongoDB |
| **缓存** | Redis（ioredis）|
| **鉴权** | Lucia · Better Auth · JWT (jose) |
| **对象存储** | S3 / MinIO / Cloudflare R2 |
| **队列** | BullMQ（Redis 后端）|
| **API 文档** | OpenAPI (Hono 自动生成 / NestJS Swagger) |
| **部署** | 前端 Cloudflare Pages / Vercel；后端 Railway / Fly / Docker |

- **关键词**：前后分离 / 独立部署 / 多客户端共后端
- **适合**：复杂前端（重交互 / 实时）· 团队前后分工 · 移动 + Web 共后端 · 多个微服务
- **不适合**：SEO 关键的内容站 · 单人项目（运维成本高）
- **典型 RTT 起步**：2~3 周

---

### 3️⃣ Nuxt 3 全栈（Vue 全家桶）

**定位**：Vue 团队的 Next.js 等价物，思路一致。

| 层 | 技术选型 |
|---|---|
| **前端框架** | Nuxt 3 · Vue 3 · TypeScript |
| **样式** | UnoCSS / Tailwind |
| **UI 组件库** | Nuxt UI · Naive UI · Vuetify |
| **状态管理** | Pinia |
| **服务端状态** | TanStack Query Vue · `useFetch` / `useAsyncData` |
| **HTTP** | `$fetch` / ofetch |
| **图标** | Iconify · `@nuxt/icon` |
| **图表** | Apache ECharts (vue-echarts) · Chart.js |
| **表格** | Tanstack Table Vue · Element Plus Table |
| **表单** | VeeValidate + Zod |
| **后端** | Nitro server · Server Routes |
| **ORM** | Drizzle / Prisma |
| **数据库** | Postgres |
| **缓存** | Redis · Nitro Storage |
| **鉴权** | nuxt-auth-utils · Better Auth |
| **对象存储** | S3 / MinIO |
| **部署** | Vercel · Cloudflare · 自部署 Node |

- **关键词**：Vue 全家桶 / 文件路由 / 同 Next 思路
- **适合**：Vue 团队 · 中文社区项目 · 轻量 SaaS · 内容站
- **不适合**：需要重 React 生态的（图表 / 复杂表单库 / 富文本）
- **典型 RTT 起步**：1~2 周

---

### 4️⃣ SvelteKit（极简全栈）

**定位**：编译时框架，体积小、性能高。

| 层 | 技术选型 |
|---|---|
| **前端 + 后端** | SvelteKit · Svelte 5 · TypeScript |
| **样式** | Tailwind |
| **UI 组件库** | shadcn-svelte · Skeleton |
| **状态管理** | Svelte Stores（内置）|
| **服务端状态** | SvelteKit Load Functions |
| **图标** | Lucide Svelte |
| **图表** | LayerChart · uPlot |
| **表格** | Svelte Headless Table |
| **表单** | Superforms + Zod |
| **ORM** | Drizzle |
| **数据库** | Postgres / SQLite (Turso) |
| **缓存** | Redis |
| **鉴权** | Lucia |
| **对象存储** | S3 / Cloudflare R2 |
| **部署** | Vercel · Cloudflare · Netlify |

- **关键词**：极简 / 体积小 / 编译时框架
- **适合**：性能敏感（弱网 / 低端机）· 小团队 · 创意站 · 个人项目
- **不适合**：依赖庞大 npm 生态的（Svelte 库少）· 大型企业（生态保守）
- **典型 RTT 起步**：1~2 周

---

### 5️⃣ FastAPI + React（Python AI / 数据后端）

**定位**：Python 后端 + React 前端，AI / ML 集成首选。

| 层 | 技术选型 |
|---|---|
| **后端框架** | FastAPI 0.110+ · Pydantic v2 · Python 3.11+ |
| **ORM** | SQLAlchemy 2 · Alembic（迁移）|
| **数据库** | Postgres + pgvector（AI 场景）|
| **缓存** | Redis · aiocache |
| **鉴权** | fastapi-users · python-jose (JWT) · OAuth2 |
| **对象存储** | boto3 (S3) · MinIO Python SDK |
| **队列** | Celery + Redis · Arq · RQ · Dramatiq |
| **AI 集成** | OpenAI SDK · LangChain · LlamaIndex · Instructor |
| **API 文档** | FastAPI 自带（OpenAPI）+ ReDoc |
| **前端框架** | Vite + React 18 + TypeScript |
| **UI 组件库** | shadcn/ui · Radix |
| **状态 / 数据** | TanStack Query + Zustand |
| **HTTP** | Axios · ofetch |
| **图表** | Recharts · Plotly.js（适合科学可视化）|
| **表格** | TanStack Table · AG Grid |
| **部署** | Docker + Railway / Fly · Hugging Face Spaces（AI）|

- **关键词**：Python 友好 / OpenAPI / AI 优先
- **适合**：AI / ML 应用 · 数据科学 · 算法 / 爬虫 / 科学计算 · 需要 numpy / pandas
- **不适合**：纯前端项目 · 实时性极高（用 Node / Go 更优）· 团队无 Python
- **典型 RTT 起步**：2~3 周

---

### 6️⃣ Go + React（高性能后端）

**定位**：Go 后端 + React 前端，高 QPS / 低延迟首选。

| 层 | 技术选型 |
|---|---|
| **后端框架** | Go 1.22+ · Gin / Echo / Chi（首选 Chi · 轻量）|
| **ORM** | sqlc（首选 · 类型安全）/ GORM / Ent |
| **数据库** | Postgres + pgx |
| **缓存** | Redis (go-redis) · BadgerDB（嵌入式）|
| **鉴权** | golang-jwt · Casbin（RBAC）|
| **对象存储** | aws-sdk-go-v2 (S3 / MinIO) |
| **队列** | Asynq（Redis 后端）· Watermill |
| **API 文档** | swaggo + swagger UI |
| **观测** | OpenTelemetry · Prometheus · zap（日志）|
| **前端** | Vite / Next + React + TanStack Query + shadcn/ui |
| **图表** | Recharts · ECharts |
| **部署** | Docker → K8s · 单二进制部署 |

- **关键词**：并发 / 单二进制 / 低延迟 / 部署简单
- **适合**：高 QPS API · 金融 · 基础设施 · 实时系统 · 微服务
- **不适合**：MVP 快速迭代 · 业务频繁变化 · 团队无 Go 经验 · AI 集成（不如 Python 顺）
- **典型 RTT 起步**：3~4 周

---

### 7️⃣ Spring Boot + Vue 中后台（**国内企业经典**）

**定位**：国内企业管理系统 / 政务 / 内部系统 / 中后台首选。生态最成熟、招人最容易。

| 层 | 技术选型 |
|---|---|
| **后端框架** | Spring Boot 3 · Java 17+ · Spring Security |
| **ORM** | **MyBatis Plus**（首选）/ MyBatis / JPA |
| **数据库** | **MySQL 8** / Postgres + Druid 连接池 |
| **缓存** | **Redis** + Redisson（分布式锁 / 限流）|
| **鉴权** | **JWT**（jjwt / java-jwt）+ Spring Security · Sa-Token（国产更顺手）|
| **对象存储** | **MinIO**（自部署首选）/ 阿里云 OSS / 腾讯云 COS / S3 |
| **队列** | RocketMQ（首选大厂）/ RabbitMQ / Kafka |
| **任务调度** | Quartz · XXL-Job（分布式）|
| **API 文档** | **Knife4j**（首选）/ SpringDoc |
| **配置中心** | Nacos（首选）· Apollo |
| **服务治理** | Spring Cloud Alibaba · Sentinel（限流熔断）|
| **观测** | SkyWalking · ELK · Prometheus |
| **前端框架** | **Vue 3** + **Vite** + TypeScript |
| **UI 组件库** | **Element Plus**（首选中后台）/ Ant Design Vue / Naive UI |
| **状态管理** | **Pinia** |
| **HTTP** | **Axios** + 拦截器封装 |
| **图表** | **Apache ECharts**（vue-echarts）|
| **表格** | **vxe-table**（高性能 · 中后台首选）/ Element Plus Table |
| **路由** | Vue Router 4 |
| **工程模板** | vue-vben-admin · vue-element-plus-admin · ruoyi-vue-pro |
| **部署** | Docker + Nginx · Nacos + Sentinel · Kubernetes |

- **关键词**：企业级 / 中后台 / 中文生态完整 / 招人容易
- **适合**：**国内** B2B / 政务 / 金融 / 大型企业管理系统 / 已有 Java 团队 / 中后台 dashboard
- **不适合**：初创快速迭代（重量级）· 国际化 SaaS（生态偏内）· AI 集成（Python 更顺）
- **典型 RTT 起步**：3~6 周（用 ruoyi 等模板可压到 1~2 周）

---

### 8️⃣ NestJS + Next.js（TypeScript 全栈分离）

**定位**：TypeScript 全栈 + 企业级架构，前后端 type-safe 共享。

| 层 | 技术选型 |
|---|---|
| **后端框架** | NestJS 10 · TypeScript 5 |
| **ORM** | Prisma（首选）/ TypeORM / Drizzle |
| **数据库** | Postgres |
| **缓存** | Redis（@nestjs/cache-manager + ioredis）|
| **鉴权** | Passport + JWT (passport-jwt) · @nestjs/passport |
| **对象存储** | aws-sdk-v3 (S3 / R2) |
| **队列** | BullMQ（@nestjs/bullmq）|
| **API 文档** | @nestjs/swagger（自动 OpenAPI）|
| **gRPC / WebSocket** | NestJS 内置支持 |
| **观测** | OpenTelemetry · Pino 日志 |
| **前端** | Next.js + React + TanStack Query + shadcn/ui |
| **共享类型** | 单 monorepo + tRPC 或 OpenAPI 生成 client |
| **部署** | Docker + Railway / K8s + Vercel |

- **关键词**：TS 全栈 / 企业级 / 模块化
- **适合**：TS 团队 · 中型 SaaS · 需要 gRPC / WebSocket · 复杂业务领域建模
- **不适合**：纯静态 / 轻 SaaS（用 Next.js 全栈即可）· 团队无 OOP 经验
- **典型 RTT 起步**：2~3 周

---

### 9️⃣ Laravel + Inertia + Vue（PHP 全栈，托管便宜）

**定位**：PHP 团队 / 共享主机 / "battery included" MVP 神器。

| 层 | 技术选型 |
|---|---|
| **后端框架** | Laravel 11 · PHP 8.3 |
| **ORM** | Eloquent（内置）|
| **数据库** | MySQL · Postgres · SQLite |
| **缓存** | Redis · 内存 / 文件 |
| **鉴权** | Laravel Sanctum / Breeze · Fortify |
| **对象存储** | Laravel Filesystem (S3 / 本地)  |
| **队列** | Laravel Queues + Redis · Horizon（监控）|
| **任务调度** | Laravel Scheduler（cron 替代）|
| **前端** | Inertia.js + Vue 3 / React + Tailwind |
| **UI 组件库** | shadcn-vue / Naive UI |
| **状态** | Pinia / Inertia 共享 |
| **图表** | Chart.js · ApexCharts |
| **表格** | TanStack Table |
| **管理后台** | Filament（PHP 写后台 · 极快）|
| **部署** | Forge · Vapor（serverless）· shared hosting · Docker |

- **关键词**：开发速度极快 / 一站式 / 托管便宜
- **适合**：PHP 团队 · MVP 快速上线 · 内容站 · 已有 Laravel 项目
- **不适合**：高 QPS API · 实时 / 流式 · 大型团队微服务
- **典型 RTT 起步**：1~2 周

---

### 🔟 React Native / Flutter（跨平台移动）

**定位**：iOS + Android 一套代码。后端任选，常配 Hono / FastAPI / Spring Boot。

#### Option A · React Native（前端团队首选）

| 层 | 技术选型 |
|---|---|
| **框架** | React Native 0.74+ · Expo SDK 51+ · TypeScript |
| **路由** | Expo Router（文件路由）/ React Navigation |
| **样式** | NativeWind（Tailwind）/ Tamagui |
| **状态** | Zustand · Legend State |
| **服务端状态** | TanStack Query |
| **HTTP** | Axios · ofetch |
| **存储** | MMKV（高性能）/ AsyncStorage |
| **导航过渡** | Reanimated v3 · Gesture Handler |
| **OTA 更新** | Expo Updates |
| **后端服务** | Supabase / Firebase / 自建 |
| **构建** | EAS Build（云构建）|

#### Option B · Flutter（性能 / 视觉一致首选）

| 层 | 技术选型 |
|---|---|
| **框架** | Flutter 3.22+ · Dart 3.4+ |
| **状态** | Riverpod · BLoC |
| **路由** | go_router |
| **HTTP** | Dio |
| **本地存储** | Hive · Drift（SQLite ORM）|
| **后端** | Firebase · Supabase · 自建 |

- **关键词**：跨平台 / 一套代码 / 接近原生
- **适合**：双端 app · 团队前端背景（RN）/ 团队学新技术（Flutter）· 创业产品
- **不适合**：需要深度 iOS / Android 系统能力（用 SwiftUI + Kotlin）· 重图形 / 重音视频
- **典型 RTT 起步**：3~5 周（含 app store 审核）

---

## 给 AI 在 2-design 阶段展示用的标准模板

收到 REQUIREMENT.md 后，AI **必须**展示如下精简格式（每张卡只露 1 行后端 + 1 行前端 + 适合）；用户选定后再展开完整工程清单：

```
🛠️ 技术栈预选（设计阶段必选）

我从 10 个完整工程模板里挑了几个最匹配你的项目，请选一个数字（也可以描述自己的偏好）：

1️⃣ Next.js 全栈
   后端：Next.js Route Handlers + Drizzle + Postgres + Redis + Auth.js
   前端：Next.js + React + Tailwind + shadcn/ui + TanStack Query + Recharts
   适合：B2B SaaS / 国际项目 / SEO 重要

5️⃣ FastAPI + React（Python AI 后端）
   后端：FastAPI + SQLAlchemy + Postgres + Redis + Celery + JWT + boto3
   前端：Vite + React + shadcn/ui + TanStack Query + Recharts
   适合：AI / ML 应用 / 数据科学 / 算法

7️⃣ Spring Boot 中后台（国内企业经典）
   后端：Spring Boot 3 + MyBatis Plus + Redis + JWT + MinIO + MySQL + RocketMQ
   前端：Vue 3 + Vite + Element Plus + Pinia + Axios + ECharts + vxe-table
   适合：国内中后台 / 政务 / 内部系统

8️⃣ NestJS + Next.js (TypeScript 全栈)
   后端：NestJS + Prisma + Postgres + Redis + BullMQ + Passport JWT
   前端：Next.js + shadcn/ui + TanStack Query + tRPC
   适合：TS 团队 / 中型 SaaS

📌 我对你的「<项目名>」的初步推荐：
   - 首选：<编号><名称> — 因为 <一句话理由，结合 REQUIREMENT 的 AC + 非功能性需求 + 团队画像>
   - 备选：<编号><名称> — 因为 <理由>

❌ 不推荐：
   - <编号><名称> — 因为 <理由，结合项目特征>

回复数字（如 "7"）或描述偏好（如 "想用 Bun + Elysia"），选定后我才出完整工程清单和 ADR。
```

**推荐生成规则**（AI 不允许跳过）：

- 必须给 **1 首选 + 1 备选 + 1~2 不推荐项**，每项都有理由
- 推荐理由必须**结合 REQUIREMENT.md 的 AC + 非功能性需求 + CONTEXT.md 的团队偏好**（QPS / 团队语言栈 / 部署环境 / 数据规模 / 是否国内）
- 用户已说出偏好（"用 Spring Boot" / "我们是 Java 团队"）→ 跳过卡片直接锁定 + 告诉用户"锁定为 X，可调整"
- 国内项目优先考虑 7️⃣ 或 9️⃣（如 PHP 团队）
- 库 / SDK / CLI → 跳过本步，只选语言

**展示候选数**：列 4~6 张最匹配的（不是 10 张全列），过载会让用户难选。

---

## 锁定后的处理

1. 写入 `DESIGN.md` 的 `## 0. 技术栈选定` 段（结构见末尾）
2. 后续所有 ADR / 数据流 / 架构图都基于这个栈展开
3. 任务的 verify 命令 / 依赖管理 / 目录结构都按栈来
4. 测试工具链按栈选（JS → Vitest，Python → pytest，Java → JUnit + Mockito，Go → testing）
5. 如果有装 [`ui-ux-pro-max`](https://uupm.cc)：用 `--stack <stack>` 拉栈级 UI 建议（在 2a-ui-design 阶段）
6. **变栈** = 改 DESIGN.md 的 0 段并全文复审 → 视为开新 CHANGE 处理（R7.1）

### `## 0. 技术栈选定` 段结构（写进 DESIGN.md）

```markdown
## 0. 技术栈选定

**模板**：<编号 + 名称>（例：7️⃣ Spring Boot 中后台）

### 后端
- **框架**：Spring Boot 3.2 / Java 17
- **ORM**：MyBatis Plus 3.5
- **数据库**：MySQL 8.0 + Druid 连接池
- **缓存**：Redis 7 + Redisson（分布式锁）
- **鉴权**：Sa-Token + JWT
- **对象存储**：MinIO（自部署）
- **队列**：RocketMQ（订单异步派单）
- **任务调度**：XXL-Job
- **API 文档**：Knife4j
- **观测**：SkyWalking + ELK

### 前端
- **框架**：Vue 3 + Vite 5 + TypeScript 5
- **UI 库**：Element Plus 2.7
- **状态**：Pinia
- **HTTP**：Axios（含统一拦截 + 错误处理）
- **图表**：Apache ECharts (vue-echarts)
- **表格**：vxe-table（订单列表 + 派单看板）
- **工程模板基线**：vue-element-plus-admin

### DevOps
- **部署**：Docker + Nginx
- **配置**：Nacos
- **CI/CD**：GitHub Actions / Jenkins

### 关键依赖版本（锁定）
- spring-boot-starter-web 3.2.5
- mybatis-plus 3.5.5
- element-plus 2.7.0
- vxe-table 4.6.x

### 选型理由
<一句话，结合 AC + 非功能性需求 + 团队偏好>

### 明确排除
- <栈编号 + 名称>：<理由>
- <栈编号 + 名称>：<理由>
```

---

## 适用矩阵（按项目类型快速过滤）

| 项目类型 | 优先候选 | 不建议 |
|---|---|---|
| 国际 Web SaaS / 中后台 | 1️⃣ 2️⃣ 3️⃣ 8️⃣ | 7️⃣（生态偏内）9️⃣（PHP 招人难）|
| **国内**企业管理系统 / 政务 / 中后台 | **7️⃣** 9️⃣ | 1️⃣ 2️⃣（生态在外）|
| 内容站 / SEO 营销 | 1️⃣ 3️⃣ 4️⃣ 9️⃣ | 5️⃣ 6️⃣（缺 SSR）|
| AI / 数据 / 算法应用 | **5️⃣** | 4️⃣ 9️⃣（不是首选）|
| 高 QPS API / 金融 / 基础设施 | **6️⃣** 7️⃣ | 4️⃣（生态弱）|
| 移动 app（双端）| **🔟** + 任选后端 | 1️⃣（仅 Web）|
| 个人项目 / 创意站 / 工具站 | 4️⃣ 1️⃣ | 7️⃣（重）8️⃣（重）|
| MVP / 快速验证 | 1️⃣ 9️⃣（PHP）4️⃣ | 6️⃣ 7️⃣ 8️⃣（迭代慢）|
| 已有 Java 团队 | **7️⃣** 8️⃣ | — |
| 已有 Python 团队 | **5️⃣** | — |
| 已有 PHP 团队 | **9️⃣** | — |
| 库 / SDK / CLI | **跳过本步**——直接选语言（TypeScript / Rust / Go）| — |

---

## 组件可替换清单（每张卡片的"可换"零件）

每个工程模板里**多数零件都是可替换的**，下面列出常见替换对：

### UI 组件库（按调性 + 框架选）

| 框架 | 中后台首选 | 国际化 / 现代 | 极简 |
|---|---|---|---|
| Vue 3 | **Element Plus** · Ant Design Vue · TDesign Vue | Naive UI · PrimeVue | Vuetify · UnoCSS |
| React | Ant Design · TDesign React | **shadcn/ui** · Mantine · Radix | Tailwind 自建 |
| Svelte | shadcn-svelte · Skeleton | Carbon Svelte | Tailwind 自建 |

### 状态管理

| 框架 | 轻量首选 | 数据获取 | 大型应用 |
|---|---|---|---|
| React | **Zustand** · Jotai | TanStack Query · SWR | Redux Toolkit |
| Vue | **Pinia** | TanStack Query Vue | Pinia + 模块化 |

### ORM

| 语言 | Type-safe 首选 | 传统 ORM | SQL builder |
|---|---|---|---|
| TypeScript | **Drizzle** | Prisma · TypeORM | Kysely |
| Python | SQLAlchemy 2 | Tortoise ORM | — |
| Java | — | **MyBatis Plus** · JPA | jOOQ |
| Go | — | GORM · Ent | **sqlc**（首选）|
| Rust | SeaORM · Diesel | — | sqlx |

### 鉴权

| 场景 | 推荐 |
|---|---|
| Next.js 全栈 | Auth.js (next-auth v5) · Clerk · Lucia |
| Node 自建 | Lucia · Better Auth · Passport + JWT |
| Java | **Sa-Token**（国产 · 简单）/ Spring Security + JWT |
| Python | fastapi-users · python-jose |
| Go | golang-jwt + Casbin（RBAC）|
| 第三方托管 | Clerk · Auth0 · Supabase Auth · Firebase Auth |

### 对象存储

| 场景 | 推荐 |
|---|---|
| 国际 / 不在乎成本 | S3 · Cloudflare R2（无出口费）· UploadThing |
| **国内** / 自部署 | **MinIO** · 阿里云 OSS · 腾讯云 COS · 七牛云 |
| 边缘 / 静态资产 | Cloudflare R2 + Workers |

### 缓存 / 队列

| 用途 | 推荐 |
|---|---|
| 缓存 | **Redis**（首选 · 通用）· Memcached · Cloudflare KV |
| 队列（Node）| **BullMQ**（Redis）· Inngest · Trigger.dev |
| 队列（Java）| **RocketMQ**（首选大厂）· RabbitMQ · Kafka（流式）|
| 队列（Python）| **Celery** · Arq · RQ · Dramatiq |
| 队列（Go）| **Asynq**（Redis）· Watermill |

### 表格 / 图表

| 类型 | 推荐 |
|---|---|
| 中后台高密度表格 | **vxe-table**（Vue）· AG Grid（React 企业版）· TanStack Table |
| 普通业务表格 | TanStack Table · Element Plus Table · Ant Table |
| 图表 - 通用 | **ECharts**（Vue 首选 · vue-echarts）· Recharts（React）· Chart.js |
| 图表 - 数据 dashboard | Tremor · Apache Superset · Plotly |
| 图表 - 科学可视化 | Plotly.js · D3 |

---

## 与外部扩展的协作

| 扩展 | 在本步的作用 |
|---|---|
| `ui-ux-pro-max` | 后续 2a 阶段用 `--stack <stack>` 拿栈级 UI 模式（67 styles + 161 palettes 中适配该栈的部分）|
| `impeccable` | 后续 2a 阶段做 design system 时按栈裁剪 |

本步**不依赖任何外部扩展**——flow-kit 内置 10 张完整工程模板够用。
