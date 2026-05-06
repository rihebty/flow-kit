# Regression Demo · brooks-lint 两条路径输出一致性

> **目的**：验证 4-dev / 5-test / 6-review 的「装了 brooks-lint」路径 A 与「内置回退」路径 B 都能产出 **4 要素 + R/T 编号** 的统一格式，让 flow-kit 下游（fix 任务追加、TASK.md 编号延续）无缝接收。
>
> **怎么用**：给任意一个 AI 跑 `## 任务` 段，拿它的输出对照 `## 验收清单`。两条路径都过清单 = 体系融合 OK。

---

## Fixture · 故意有问题的代码

`example/user_service.py`（取自 brooks-lint README 改造）：

```python
class UserService:
    def update_profile(self, user_id, name, email, avatar_url):
        user = self.db.query(f"SELECT * FROM users WHERE id = {user_id}")
        user['email'] = email
        user['name'] = name
        user['avatar_url'] = avatar_url
        self.db.execute(f"UPDATE users SET ... WHERE id = {user_id}")

        if user['email'] != email:
            self.smtp.send(
                to=email,
                subject="Email changed",
                body=f"Your email was changed for user {user_id}"
            )

        points = user['login_count'] * 10 + 500
        self.db.execute(f"UPDATE loyalty SET points={points} WHERE user_id={user_id}")

        self.cache.delete(f"user:{user_id}")
        self.cache.delete(f"profile:{user_id}")
        self.cache.delete(f"avatar:{user_id}")
```

明显缺陷：
- SQL 注入（f-string 拼 `user_id`）
- 静默 bug（`user['email'] = email` 后再比较 `user['email'] != email`，永远 False）
- 4 个不相关业务责任挤在一个方法（profile / 邮件通知 / 积分 / 缓存）
- 魔法数字（10 / 500）

---

## 任务

> 你是 4-dev 阶段的 AI。本任务只做**步骤 4 · 提交前 self-review**。
>
> 拿上面的 fixture 当成你刚改完的 diff，按 4-dev.md 步骤 4 的 6 维诊断输出报告。

---

## 路径 A · 装了 brooks-lint 的预期输出

调用 `/brooks-review` 后，应得到至少 4 条结构化发现。每条**必须**含 4 要素 + R 编号：

```markdown
### 🔴 R6 · Domain Model Distortion：邮件通知静默失效（dead code）
**Symptom**：`example/user_service.py:11` 比较 `user['email'] != email`，但第 4 行已先把 `email` 写入 `user['email']`，分支永远 False。
**Source**：McConnell · Code Complete · Ch. 17 Unusual Control Structures（控制结构反模式）
**Consequence**：用户改邮箱后**永不**收到通知。系统看似工作，实际违反业务规则——是数据完整性级 bug。
**Remedy**：第 3 行加 `old_email = user['email']`，第 11 行改为 `if old_email != email`。

### 🔴 R2 · Change Propagation：单方法承担 4 个不相关责任
**Symptom**：`example/user_service.py:2-23` 一个方法体内同时做：profile 字段更新（4 行）/ 邮件通知（5 行）/ 积分计算（2 行）/ 缓存失效（3 行）。
**Source**：Fowler · Refactoring · Divergent Change；Hunt & Thomas · Pragmatic Programmer · Orthogonality
**Consequence**：改积分公式有概率破坏邮件通知；任何一处修改都要回归 4 个不相关业务领域。
**Remedy**：抽 `NotificationService` / `LoyaltyService` / `UserCacheInvalidator`，`update_profile` 仅做编排。

### 🔴 R4 · Accidental Complexity：SQL 注入（f-string 拼接）
**Symptom**：`example/user_service.py:3 / 7 / 19` 用 f-string 拼接 `user_id` 进 SQL，无参数化。
**Source**：OWASP A03（注入）；Pragmatic Programmer · Tip 35 (Don't trust input)
**Consequence**：恶意 user_id 可读取/篡改任意用户数据。
**Remedy**：改参数化查询：`self.db.query("SELECT * FROM users WHERE id = ?", [user_id])`，三处全改。

### 🟡 R6 · Domain Model Distortion：魔法数字（10 / 500）
**Symptom**：`example/user_service.py:18` `points = user['login_count'] * 10 + 500` 含两个未命名常量。
**Source**：Fowler · Refactoring · Magic Number
**Consequence**：业务规则不可见——读代码不知道 10 是 "每次登录 10 分"，500 是 "新人补贴 500"，未来改规则要翻代码。
**Remedy**：抽 `POINTS_PER_LOGIN = 10` / `SIGNUP_BONUS = 500`，或更进一步抽 `LoyaltyPolicy.calculate(user)`。
```

**严重度统计**：🔴 3 / 🟡 1 / 🟢 0 → 必修后再提交。

---

## 路径 B · 未装 brooks-lint 的预期输出（内置回退）

AI 自己按 R1~R6 过 diff，发现的问题数会**少一些**（按 brooks-lint benchmark：约 16% vs 100%），但**格式必须一致**：

```markdown
### 🔴 R6 · Domain Model Distortion：邮件比较条件永远 False
**Symptom**：`example/user_service.py:11` 的条件比较前已被覆盖。
**Source**：Code Complete · 控制结构反模式
**Consequence**：邮件通知不会发送。
**Remedy**：先 capture `old_email`，再比较。

### 🔴 R2 · Change Propagation：方法承担太多责任
**Symptom**：`example/user_service.py:2-23` 单方法 4 类业务逻辑混合。
**Source**：Refactoring · Divergent Change
**Consequence**：任意改动都有跨域回归风险。
**Remedy**：抽 NotificationService / LoyaltyService / CacheInvalidator。
```

**关键差异**（合理）：
- 路径 B 可能漏掉 SQL 注入（不是 6 维核心）
- 路径 B 可能漏掉魔法数字（Minor 项）
- 路径 B 的 Source 引用更"概括"（"Code Complete · 控制结构反模式"vs"McConnell · Code Complete · Ch. 17"）

**关键一致**（必须）：
- 都用 `🔴/🟡/🟢 R<x> · <name>：<结论>` 标题格式
- 都含 Symptom / Source / Consequence / Remedy 4 段
- 都指向 `<file>:<line>`
- 都按严重度分级

---

## 验收清单（两条路径都过 = 体系 OK）

### 格式一致性

- [ ] 标题用 `🔴/🟡/🟢 R<x> · <名字>：<一句话>` 格式
- [ ] 4 要素齐全（Symptom / Source / Consequence / Remedy）
- [ ] 至少一处 `<file>:<line>` 引用
- [ ] Source 含至少一本书 / 一节标题（不是"最佳实践"）
- [ ] 严重度统计有数字（🔴 X / 🟡 Y / 🟢 Z）

### 下游可接收

- [ ] 🔴 项可以被复制到 `TASK.md` 末尾作为 `T-FIX-NN` 任务（路径 A 的 Remedy 段足够明确，AI 能直接执行）
- [ ] 🟡 项可以被记入 SUMMARY.md「已知接受 + 理由」段
- [ ] 🟢 项可以被记入 SUMMARY.md「已知小问题」段

### 路径差异（合理范围）

- [ ] 路径 B 漏的项**全是非典型 6 维问题**（如 SQL 注入更属于 OWASP / 安全测试，不是衰退风险）
- [ ] 路径 B 漏的项**全是 Major 或 Minor**（不能漏 Critical）
- [ ] 路径 A 多出来的项的 Source 引用**精确到节标题**（如 "Ch. 17"）

---

## 怎么实际跑这个 demo

### 你是 flow-kit 维护者验证集成

```bash
# 1. 装 brooks-lint
/plugin install brooks-lint@brooks-lint-marketplace

# 2. 把这个 fixture 喂给 4-dev 阶段
@flow-kit/prompts/4-dev.md
@flow-kit/regression-demos/brooks-lint-paths.md

请按 4-dev 步骤 4 跑这个 demo 的 self-review，对照验收清单。

# 3. 卸 brooks-lint，重新跑（验证路径 B）
/plugin uninstall brooks-lint
（同上请求）

# 4. 比对两份输出，过验收清单
```

### 你是用户想试用 flow-kit + brooks-lint

```
@flow-kit/GO.md
帮我用这个 demo（@flow-kit/regression-demos/brooks-lint-paths.md）走一遇 4-dev 的 self-review
```

GO.md 路由命中 `执行 T<NN>` / `跑` 类触发词 → `prompts/4-dev.md` → AI 进入步骤 4，按你装没装 brooks-lint 自动选路径 A/B。

---

## 期望结果

| 路径 | 预期发现数 | 严重度分布 | 下游可接收 |
|---|---|---|---|
| A · 装了 brooks-lint | ≥ 4 条 | 🔴 ≥ 3 / 🟡 ≥ 1 | ✅ 直接生成 fix 任务 |
| B · 内置回退 | ≥ 2 条 | 🔴 ≥ 2 | ✅ 部分生成 fix 任务，需人工补 |

**只要两条路径都过「格式一致性」+「下游可接收」段，融合就成功了**。「路径差异」段的 gap 是 brooks-lint 本身的价值（带书本引用 + 系统化覆盖），不是 flow-kit 集成的问题。
