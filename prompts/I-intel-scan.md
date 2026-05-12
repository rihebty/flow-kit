# I-intel-scan · 入场扫描（横向命令）

> 在已存在的项目（brownfield）里第一次用 flow-kit 时，**先跑这个**。
> 自动扫描代码库 → 自动填 `CONTEXT.md` → 后续所有 change 都受益。
> 不是周期命令，跑过一次就行；项目结构有大变化（重构 / 框架升级 / 新增模块）时再跑。

## 触发方式

- 用户说「**扫描代码 / scan / intel / 入场扫描 / 给项目体检**」
- 进入 0-change 时检测 `CONTEXT.md` 不存在或字段缺失（见 GO.md 入场检测段）
- `STATE.md` 没有 `last_intel_scan` 字段或距上次 > 90 天

## 输入

- 项目根目录（grep / find / read 各种探针）
- 不需要 `.specs/` 任何文件（这是入场前命令）

## 输出

- `templates/CONTEXT.md` 填好的版本，统一写到 `.specs/CONTEXT.md`
- `STATE.md` 的 `last_intel_scan: <YYYY-MM-DD>` 字段更新

## 步骤

### 0. 既有文档探测（必跑 · 在任何扫描前）

**目的**：很多项目已经有 AI 可读的开发约定文档。不要直接覆盖或忽略，先问用户。

#### 0.1 探测下面这些标准文档

按下面顺序 `ls` / `find` 探测项目根目录与常见路径：

| 文档 | 路径 | 来自哪个生态 |
|---|---|---|
| `CONTEXT.md` | 仓库根 / `.specs/` | flow-kit 自己 |
| `AGENTS.md` | 仓库根 | OpenAI Codex / 标准 agents 协议 |
| `CLAUDE.md` | 仓库根 / `.claude/` | Anthropic Claude Code |
| `.cursor/rules/*.md` | `.cursor/` | Cursor IDE |
| `.windsurf/rules/*.md` | `.windsurf/` | Windsurf IDE |
| `.github/copilot-instructions.md` | `.github/` | GitHub Copilot |
| `.clinerules` | 仓库根 | Cline |
| `ARCHITECTURE.md` | 仓库根 / `docs/` | 项目自定义 |
| `CONTRIBUTING.md` | 仓库根 | 项目自定义 |
| `README.md` | 仓库根 | 通用（最低保底）|

#### 0.2 三种分支

##### 分支 A · 找到了至少一个标准 AI 上下文文档（CONTEXT / AGENTS / CLAUDE / Cursor / Windsurf / Copilot / Cline）

**反问用户**：

```
🔍 检测到本项目已有以下 AI 上下文文档：
  - AGENTS.md（仓库根，<size>）
  - CLAUDE.md（仓库根，<size>）
  - .cursor/rules/architecture.md（<size>）

flow-kit 用 CONTEXT.md 作为单一源。请选择：
  1. 综合 + 扫描（推荐）：我读所有现有文档 + 跑入场扫描，合并生成 CONTEXT.md
  2. 以现有文档为准：选定其中一个作为基础（告诉我哪个）→ 我提取关键信息到 CONTEXT.md，保留双向链接
  3. 忽略现有文档，重新扫描生成
  4. 不生成 `.specs/CONTEXT.md`，4-dev / 2-design 改用 <你指定的文档>（如 AGENTS.md）作为 AI 遵守依据

请选 1/2/3/4。无人工确认前我不动手。
```

各选项后续动作：

- **选 1（综合）**：read 每份文档（带 R1.9 token 预算控制） → 跑步骤 1（项目元信息）→ 把现有文档的关键决策**整合**到 CONTEXT.md（在段落末尾标 `来自 AGENTS.md:42`）→ 在 CONTEXT.md 顶部加「源文档」段列出所有引用
- **选 2（以现有为准）**：read 用户指定的文档 → 把它的关键内容映射到 CONTEXT.md 的对应段（`既有抽象索引` / `命名约定` / `禁动清单` 等），跑**精简扫描**（只补现有文档没说的部分）
- **选 3（忽略重扫）**：直接跑步骤 1，不读现有文档（**警告用户**：可能与现有约定冲突）
- **选 4（不生成 `.specs/CONTEXT.md`）**：在 STATE.md 写 `ai_context_doc: <用户指定路径>`，**所有后续阶段**（GO.md / 2-design / 4-dev）改读这个文档代替 `.specs/CONTEXT.md`。**跳过本 prompt 剩余步骤**

##### 分支 B · 找到非标准但项目级文档（README / ARCHITECTURE / CONTRIBUTING）

这些不是 AI 专用文档但常含有用信息。**反问用户**：

```
 未发现标准 AI 上下文文档（CONTEXT/AGENTS/CLAUDE/...）。

但发现这些项目级文档：
  - README.md（<size>）
  - docs/ARCHITECTURE.md（<size>）
  - CONTRIBUTING.md（<size>）

请选择：
  1. 把它们当作扫描的补充输入，生成完整 CONTEXT.md（推荐）
  2. 跳过这些，仅按代码扫描
  3. 用户指定其中一个作为开发遵守依据（告诉我哪个）

请选 1/2/3。
```

##### 分支 C · 一个文档都没有

**反问用户**（**必须**确认才能开始扫描）：

```
 项目里未发现任何 AI 上下文文档（CONTEXT/AGENTS/CLAUDE/Cursor/Windsurf/Copilot/Cline/README/ARCHITECTURE/CONTRIBUTING）。

flow-kit 需要一份 CONTEXT.md 给 AI 用。请选择：
  1. 现在跑入场扫描，纯从代码生成 CONTEXT.md（~15-30k tokens，仅首次）
  2. 我手动指定项目里某个文档作为开发遵守依据：<请回复路径>
  3. 跳过 intel-scan，直接进 0-change（不推荐 · AI 会"盲飞"，老项目护栏 B1-B5 全失效）

请选 1/2/3。无人工确认前我不开始扫描（避免无意义消耗 token）。
```

#### 0.3 把决策写进 SUMMARY

无论选哪个分支，都要在最终扫描总结里贴出：

```
## 既有文档探测（步骤 0）
- 探测命中：<列出找到的文档>
- 用户选择：<选项编号 + 简述>
- 决策：<生成 CONTEXT.md / 以现有文档为准 / 跳过 intel-scan>
- 引用源（如有）：<列出 source 文档路径 + 重点段位置>
```

### 1. 探测项目元信息（grep / read）

按下面顺序探测，**记录每条的发现**（找不到也要记"未发现"）：

#### 1.1 包管理与运行时

| 信号文件 | 提取信息 |
|---|---|
| `package.json` | name / engines / 主要依赖 / scripts |
| `pyproject.toml` / `requirements.txt` / `Pipfile` | Python 版本 / 主要依赖 |
| `Cargo.toml` | Rust edition / 依赖 |
| `go.mod` | Go 版本 / 主要依赖 |
| `pom.xml` / `build.gradle` | Java 版本 / Spring / Maven |
| `composer.json` | PHP 版本 / Laravel / Symfony |
| `Gemfile` | Ruby / Rails 版本 |

#### 1.2 框架检测（前端 + 后端）

```
grep -l "from ['\"]react" src/        → React
grep -l "from ['\"]vue" src/          → Vue
grep -l "import.*svelte" src/         → Svelte
ls -la nuxt.config.* next.config.*    → Next/Nuxt
grep -l "@nestjs" src/                → NestJS
grep -l "fastapi" .                   → FastAPI
grep -l "django" .                    → Django
grep -l "spring-boot" .               → Spring Boot
grep -l "express" src/                → Express
```

#### 1.3 关键约定

```
# 命名约定
ls src/ | head -20                    → 看目录命名是 kebab / camel / snake
find src -type f -name "*.ts" | head  → 看文件命名

# import 风格
grep "^import" src/**/*.ts | head -20 → 看 alias 用法（@/ vs ../）

# 测试框架
ls jest.config.* vitest.config.* pytest.ini  → 测试框架
find . -path '*/__tests__/*' -o -name '*.test.*' -o -name '*.spec.*' | head

# Lint / Format
ls .eslintrc.* .prettierrc.* .ruff.toml .pylintrc → 代码规范

# CI
ls .github/workflows/* .gitlab-ci.* azure-pipelines.* → CI 平台
```

#### 1.4 既有抽象层（关键 · 防重复实现）

grep 出每一类公共工具的实际位置：

| 抽象类型 | grep 命令 |
|---|---|
| HTTP client | `grep -rn "axios\|fetch\|httpClient\|apiClient\|http.Get" src/` |
| 数据库访问 | `grep -rn "Repository\|DAO\|prisma\|sequelize\|sqlalchemy\|@Entity" src/` |
| 状态管理 | `grep -rn "createStore\|useStore\|atom\|@Injectable" src/` |
| 工具函数 | `find src -path '*utils*' -o -path '*helpers*' -o -path '*shared*' -o -path '*common*' \| head` |
| 自定义 hooks（前端）| `find src -name 'use*.ts*' \| head` |
| 中间件 | `find src -path '*middleware*' \| head` |
| 错误处理 | `grep -rn "class.*Error\|errorHandler\|ErrorBoundary" src/` |

#### 1.5 数据库 schema

```
ls prisma/schema.prisma alembic.ini knexfile.* db/migrate/ → schema 工具
ls migrations/                                              → 迁移目录
```

#### 1.6 基础设施

```
ls Dockerfile docker-compose.* k8s/ helm/  → 容器化
grep -l "DATABASE_URL\|REDIS_URL" .env*    → 服务依赖
```

### 2. 生成 CONTEXT.md

用 `templates/CONTEXT.md` 模板，填入第 1 步的发现。每个字段都要有具体证据（文件路径 + 行号）。

**禁止**填空话："使用了一些工具" / "标准结构"——这些都要换成具体的「`src/utils/date.ts:12`」级证据。

### 3. 写 STATE.md

```markdown
last_intel_scan: 2026-05-02
context_file: .specs/CONTEXT.md
detected_stack: <主要技术栈一句话总结>
```

### 4. 给用户一份扫描总结

```markdown
## 入场扫描完成

**项目类型**：<前端 / 后端 / 全栈>
**主要技术栈**：<Next.js 14 + Prisma + Postgres>
**关键发现**：
  - ✅ HTTP 客户端：`src/lib/api-client.ts:1`（统一封装 axios）
  - ✅ 工具函数：`src/utils/`（date / string / number 已有）
  - ✅ 自定义 hooks：`src/hooks/`（22 个）
  - ⚠️ Schema 工具：未检测到（裸 SQL 或还没用迁移工具）
  - ⚠️ 测试覆盖：仅 `auth/` 目录有测试，其他模块裸奔

**对后续 change 的影响**：
  - 4-dev 阶段必须沿用既有 ApiClient（不要直接 fetch）
  - 涉及 schema 改动 → 建议引入 Prisma 或 Knex（4-dev 1.7 会拦）
  - 新增模块的 hooks 命名要参考现有 22 个的风格

`.specs/CONTEXT.md` 已生成。后续运行 0-change 时 AI 会自动加载。
```

## 自检

- [ ] **步骤 0 既有文档探测做了**：找到的标准 AI 上下文文档已列出，用户已选定分支（A/B/C），决策已写入扫描总结
- [ ] **未经用户同意没自动开始扫描**（分支 C 必须等用户回复 1/2/3）
- [ ] 用户选 4「不生成 `.specs/CONTEXT.md`」时已写 `STATE.md` 的 `ai_context_doc` 字段并跳过剩余步骤
- [ ] 1.1~1.6 各段都有 grep / read 输出（不靠猜）
- [ ] CONTEXT.md 的每个字段都有文件路径 / 行号引用
- [ ] CONTEXT.md 顶部「源文档」段列出引用的 AGENTS / CLAUDE / 等（如适用）
- [ ] STATE.md 的 `last_intel_scan` 已更新
- [ ] 扫描总结贴出关键发现 + 对后续 change 的影响

## 触发下一步

- 跑完 → 用户继续原本意图（如 `@flow-kit/GO.md` + 一句新需求）
- AI 此时会自动读 `.specs/CONTEXT.md`（或步骤 0 选 4 指定的替代文档）→ 后续阶段都基于它
