# 测试金字塔（5 轮 · 工具 / 标准 / 清单）

> `5-test.md` 主 prompt 决定**做哪几轮**，本文件提供**每轮怎么做**的查询资料。
> 各轮按项目类型可裁剪：见末尾「适用矩阵」。

---

## 第 1 轮 · 功能测试（Functional）

**目标**：每条 AC 都有自动化覆盖或可执行的 UAT。

### 工具选择（按栈选）

| 栈 | 单测 | 集成 | E2E |
|---|---|---|---|
| Web 前端 | Vitest / Jest + Testing Library | MSW + Vitest | Playwright（首选）/ Cypress |
| Node 后端 | Vitest / Jest | supertest + 容器 DB | Playwright（API 模式）/ Postman |
| Python | pytest | pytest + testcontainers | pytest + httpx |
| Go | testing | testify + dockertest | godog（BDD）/ httptest |
| Rust | cargo test | testcontainers-rs | reqwest + tokio |

### 通过标准

- [ ] 每条 AC 至少 1 条测试或 UAT 项
- [ ] 关键路径行覆盖率 ≥ 项目门槛（默认 80%；core 模块 ≥ 90%）
- [ ] 错误路径有显式测试（不只是 happy path）
- [ ] 边界值（空 / 极大 / 极小 / Unicode / 负数 / null）至少 3 条
- [ ] 异步 / 并发场景有 race condition 测试

### 反模式（命中即不通过）

- ❌ 只测实现细节（如 mock 后断言 mock 被调用过——这测的是 mock 不是行为）
- ❌ 测试名称是 `test_function_works`（应该描述行为：`should reject negative quantity`）
- ❌ 一个测试断言 > 5 条（拆成多个）
- ❌ 共享状态的测试（一个测试依赖另一个的执行）
- ❌ Sleep / setTimeout 等待（用 fake timer 或 await 条件）

---

## 第 2 轮 · 性能测试（Performance）

**目标**：可量化、有基线、有预算的性能验证。

### 前端性能

| 工具 | 测什么 | 通过标准（默认）|
|---|---|---|
| Lighthouse CI | LCP / CLS / INP / TBT | LCP < 2.5s / CLS < 0.1 / INP < 200ms |
| WebPageTest | 真实网络下的 First Byte / Speed Index | Speed Index < 3s |
| Bundle Analyzer | 主包 / 路由分包大小 | 主包 < 200KB gzip / 路由块 < 50KB gzip |
| `perfume.js` / RUM | 真实用户性能 | p75 LCP < 2.5s |

### 后端 / API 性能

| 工具 | 测什么 | 通过标准（默认）|
|---|---|---|
| k6 / locust / wrk | 关键 API 在 N 倍业务 QPS 下的响应 | p95 < SLO；错误率 < 0.1% |
| autocannon（Node） | 简单压测 | 同上 |
| `EXPLAIN ANALYZE`（DB）| 慢查询审计 | 关键查询 < 100ms / 无 N+1 |

### 通用预算（必须事先约定）

写在 `REQUIREMENT.md` 的"非功能性需求"段：

```yaml
performance:
  frontend:
    LCP_p75: < 2.5s
    CLS_p75: < 0.1
    main_bundle: < 200KB gzip
  backend:
    api_p95:
      "/login": < 200ms
      "/checkout": < 500ms
    error_rate: < 0.1%
    qps_target: 1000
```

测试结果必须**逐项**和预算对比，不允许"性能良好"这种空话。

### 反模式

- ❌ 只测 happy load，不测 spike / sustained
- ❌ 在开发机跑性能测试（必须在接近生产的环境）
- ❌ 单次跑就下结论（至少 3 次取中位数）
- ❌ 没有基线对比（必须和上一版对比，不退步）

---

## 第 3 轮 · 安全测试（Security）

**目标**：依赖 / 代码 / 秘钥 / 输入 / 鉴权五道防线都过。

### 依赖漏洞扫描

| 栈 | 工具 | 通过标准 |
|---|---|---|
| Node | `npm audit --production` / Snyk | 无 high / critical |
| Python | `pip-audit` / `safety check` | 无 high / critical |
| Go | `govulncheck ./...` | 无已知漏洞 |
| Rust | `cargo audit` | 无 unyanked 漏洞 |
| 容器镜像 | Trivy / Grype | 无 high / critical |

### 静态扫描（SAST）

| 工具 | 适用 |
|---|---|
| Semgrep（首选）| 多语言 / 自定义规则 |
| GitHub CodeQL | GitHub 项目免费 |
| SonarQube | 大型项目 |
| Bandit（Python）| Python 专用 |
| ESLint security plugin | JS/TS |

通过标准：无 high 级发现；medium 级有处理记录（修 / 接受 / 误报）。

### 秘钥扫描

| 工具 | 范围 |
|---|---|
| trufflehog | 历史 commit + 当前文件 |
| git-secrets | 防止提交 |
| gitleaks | CI 集成 |

通过标准：0 命中。命中必须立即 rotate 对应密钥（不只是删 commit）。

### OWASP Top 10 检查清单（API / Web）

按当年版本对照（当前 2021 版）：

- [ ] A01 Broken Access Control — 越权测试（横向 / 纵向）
- [ ] A02 Cryptographic Failures — 敏感数据传输 / 存储加密
- [ ] A03 Injection — SQL / NoSQL / Command / LDAP 注入测试
- [ ] A04 Insecure Design — 设计阶段威胁建模
- [ ] A05 Security Misconfig — CORS / CSP / Cookie flag / Headers
- [ ] A06 Vulnerable Components — 见上面"依赖漏洞"
- [ ] A07 Auth & Session — 弱密码 / brute force / session fixation
- [ ] A08 Software & Data Integrity — CI/CD 完整性 / 反序列化
- [ ] A09 Logging & Monitoring — 见第 5 轮可观测性
- [ ] A10 SSRF — URL 输入校验 / 内网保护

每项给"已测 / 不适用 / 待补"三态，不允许跳过。

### 反模式

- ❌ "我们用了框架所以应该没问题"——必须测
- ❌ 只测自己写的代码，不测依赖
- ❌ 把扫描结果当噪音忽略——每条 high 都要处理记录
- ❌ 用生产数据做测试

---

## 第 4 轮 · 兼容性测试（Compatibility）

**目标**：在用户实际使用的环境里跑得通。

### 前端跨浏览器 / 跨设备

**最低基线**（默认）：

| 类别 | 必测 | 可选 |
|---|---|---|
| 桌面浏览器 | Chrome / Firefox / Safari / Edge（最新 2 个版本）| IE 11 / 旧版 Edge |
| 移动浏览器 | iOS Safari / Android Chrome | 其他 webview |
| 视口 | 360 / 768 / 1024 / 1440 | 320 / 1920 / 2560 |
| 操作系统 | macOS / Windows / iOS / Android | Linux 桌面 |

工具：

| 工具 | 用途 |
|---|---|
| Playwright（多 browser） | 自动化跨浏览器 e2e |
| BrowserStack / LambdaTest | 真机云测试 |
| Chrome DevTools Device Mode | 快速验证视口 |

### 后端跨版本兼容

- [ ] 数据库迁移：旧 schema 数据能否被新代码正确读写
- [ ] API 版本：v1 客户端访问 v2 服务端能否优雅降级
- [ ] 协议兼容：HTTP/1.1 / HTTP/2 / HTTP/3
- [ ] 编码兼容：UTF-8 / UTF-16 / 不同 locale

### 数据迁移测试（涉及 schema 变更必跑）

- [ ] 在生产数据快照上预演迁移脚本
- [ ] 迁移耗时实测（决定是否需要 maintenance window）
- [ ] 回滚脚本就位且测过
- [ ] 双写期 / 灰度方案有验证步骤

### 反模式

- ❌ 只在开发者的浏览器上测
- ❌ 移动端只在模拟器测，不上真机
- ❌ 数据迁移只在小数据量上测过
- ❌ 假设"用户都用最新版"

---

## 第 5 轮 · 可观测性验证（Observability）

**目标**：上线后能"看到"系统在做什么，故障能定位。

### 日志（Logs）

- [ ] 关键路径的入口 / 出口 / 异常都有日志
- [ ] 日志级别正确（DEBUG / INFO / WARN / ERROR / FATAL）
- [ ] 日志含 trace-id（可串到分布式追踪）
- [ ] **不含敏感信息**（密码 / token / 身份证 / 卡号 / 邮件正文等）
- [ ] 结构化日志（JSON）而非纯文本（便于查询）
- [ ] 错误日志含足够上下文（user-id / request-id / 关键参数）

### 指标（Metrics）

- [ ] 业务关键 metric 有打点（如订单完成率、登录成功率、关键转化）
- [ ] 系统级 metric 完备（CPU / 内存 / 连接池 / 队列深度）
- [ ] RED 指标（Rate / Errors / Duration）覆盖关键 endpoint
- [ ] USE 指标（Utilization / Saturation / Errors）覆盖关键资源

### 链路追踪（Traces）

- [ ] 跨服务调用 trace 串通
- [ ] 慢操作 / 失败操作能被 trace 定位
- [ ] 采样率合理（生产 1~10%，故障期可临时拉高）

### 告警（Alerts）

- [ ] 关键失败有告警（不仅监控面板）
- [ ] 告警分级（P0 立即响应 / P1 工作时段 / P2 周报）
- [ ] 无"狼来了"（噪音告警必须降级或清除）
- [ ] 每个告警有 runbook 链接（"收到这个告警怎么办"）

### 健康检查

- [ ] `/health` 端点存在且区分 liveness / readiness
- [ ] 关键依赖（DB / cache / 上游 API）的可用性反映在健康检查里

### 反模式

- ❌ 只有 stdout 日志（必须接到日志聚合系统）
- ❌ 告警直接发邮件（噪音 + 延迟，应进 PagerDuty / Opsgenie / IM webhook）
- ❌ "出问题再加日志"（应在 4-dev 阶段就埋好）
- ❌ 把 PII 打到日志（GDPR / 合规风险）

---

## 适用矩阵（按项目类型裁剪）

| 项目类型 | 第 1 轮 | 第 2 轮 | 第 3 轮 | 第 4 轮 | 第 5 轮 |
|---|---|---|---|---|---|
| 前端 SaaS | ✅ 必跑 | ✅ Lighthouse + bundle | ✅ 依赖 + 秘钥 + CSP/XSS | ✅ 跨浏览器 | ⚠️ 错误上报 + RUM |
| 后端 API | ✅ 必跑 | ✅ load + DB | ✅ 全套 | ⚠️ 数据迁移 | ✅ 必跑 |
| 全栈 web | ✅ 必跑 | ✅ 全 | ✅ 全套 | ✅ 跨浏览器 + 数据 | ✅ 必跑 |
| 移动 app | ✅ 必跑 | ✅ 启动 + 内存 | ✅ 依赖 + 秘钥 + 越权 | ✅ 多机型 + iOS/Android | ✅ Crash 率 + ANR |
| CLI 工具 | ✅ 必跑 | ⚠️ 启动时间 | ✅ 依赖 + 秘钥 | ✅ 跨 OS | ⚠️ 错误上报 |
| 库 / SDK | ✅ 必跑 | ⚠️ 关键 API benchmark | ✅ 依赖 + 接口 fuzz | ✅ 跨语言版本 | ❌ 不适用 |
| 内部工具 | ✅ 必跑 | ⚠️ 不必苛刻 | ⚠️ 依赖 + 越权 | ⚠️ 主流即可 | ⚠️ 基础日志 |

**图例**：✅ 必跑 / ⚠️ 推荐但可裁剪 / ❌ 不适用

`5-test.md` 在阶段开始时**必须**显式声明本次走哪几轮 + 跳过哪些 + 跳过理由。
