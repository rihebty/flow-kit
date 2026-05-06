# LESSONS — 跨任务失败知识库

> 项目级常驻文件，位置：`.specs/LESSONS.md`
> **每个新任务开工前必扫**（R1.8）；任务完成后按本文件末尾的「提名条件」决定是否新增。

---

## 标签索引

- `arch` 架构决策类
- `lib` 第三方库选型 / 陷阱
- `tool` 构建 / 测试 / 工具链
- `data` 数据建模 / 迁移
- `perf` 性能
- `a11y` 可访问性
- `sec` 安全
- `ux` 交互 / 视觉
- `ops` 部署 / 运维
- `proc` 流程 / 协作

---

## 使用方式

### 写代码前（DEV 阶段，R1.8）

1. AI 用任务的 `files` + `action` 关键词 grep 本文件
2. 命中的条目必须在执行计划里显式声明：
   - 「已查阅 L-NNN，本次方案与之的差异是 X」 或
   - 「已查阅 L-NNN，本次确认仍适用，因此不重试该方案」
3. 若计划方案与某条完全相同 → 触发 R1.6，不允许直接重试

### 任务完成后（INTEGRATION ARCHIVE 步骤）

AI 扫本次 change 的所有 `*-SUMMARY.md` 与遗留 `*-PROGRESS.md`，按下面「提名条件」筛选并追加新条目。

---

## 条目格式（复制此模板新增）

```markdown
### L-NNN · [tag1, tag2] 标题（一行内说清，便于目录扫读）

- **首发**: <change-id> · <task-id> · <YYYY-MM-DD>
- **上次复核**: <YYYY-MM-DD>
- **适用栈**: <例 React 18+ / Node 20 / SQLite 当前 schema>
- **状态**: active / superseded-by:L-MMM / deprecated:<原因>
- **关键词**: <空格分隔，方便 grep；用任务里会出现的词>

**问题场景**
<一段话描述什么情况下会想用被否决的方案>

**当时尝试的方案**
<具体写法，必要时贴片段>

**为什么不行**
<具体失败原因，越可量化越好；含可复现的错误信号 / 度量数字 / 链接>

**当前推荐做法**
<指向 SUMMARY / ADR / DESIGN 链接，或一句话替代方案>

**何时可重新评估**
<例：升级到 React 19 之后；该 lib 修复 #123 之后；引入 SSR 后>
```

---

## 提名条件（满足任一即建议入库）

- 调试 / 试错总耗时 > 30 分钟
- 错因不局限于本任务的微细节，**其它任务也会撞上**
- 未来 6 个月内有合理概率被再次尝试（含其他成员、其他 AI 会话）
- 否决理由不写在 ADR 里就会丢失（架构层面已写 ADR 的不必再放这里）

**反例**（这些不要进 LESSONS）：
- 一次性的拼写错误、笔误
- 项目独有的业务规则（应进 `CONTEXT.md` 已锁决策）
- 已经在 ADR 中详细说明的架构权衡（指过去即可）

---

## 复核与剪枝（每个 ARCHIVE 步骤顺手做）

- **栈变更检查**：当 `package.json` / `Dockerfile` / DB schema 有大版本变化时，扫描所有 active 条目，把"适用栈"已不匹配的标 `deprecated:<栈版本>`
- **superseded 标记**：发现新条目对老条目形成更优替代时，给老条目加 `superseded-by:L-MMM`，不删除（保留历史）
- **季度剪枝**：deprecated > 6 个月的条目移到 `.specs/archive/LESSONS-archive.md`

---

## 条目区（实际项目里把下面的示例替换成你的真实条目）

<!-- 示例条目，新项目可保留作模板，第一次提名时删除 -->

### L-001 · [arch, ux] 主题切换不要用 useEffect 同步 localStorage

- **首发**: add-dark-mode · T03 · 2026-04-30
- **上次复核**: 2026-04-30
- **适用栈**: React 18+，纯 SPA（无 SSR）
- **状态**: active
- **关键词**: theme dark-mode localstorage useEffect FOUC flash hydration

**问题场景**
实现深色模式时，希望在用户上次设置上恢复主题。直觉做法是用 `useEffect` 在 mount 后从 localStorage 读取并 setState。

**当时尝试的方案**
```tsx
useEffect(() => {
  const saved = localStorage.getItem('theme');
  if (saved) setTheme(saved);
}, []);
```

**为什么不行**
首次渲染会先以默认主题 paint 一帧（约 80–150ms），然后才切到 saved，造成肉眼可见的 FOUC（Flash Of Unstyled Content）。Lighthouse Performance 也会因此扣分。

**当前推荐做法**
- 把读 localStorage 和写 `<html data-theme>` 这一步**前置到首次渲染前**：用一段同步内联 script 注入 `<head>`，或用 `useLayoutEffect` 在 SPA 入口组件里执行
- 详见 `.specs/archive/2026-04-30-add-dark-mode/SUMMARY.md` 的"决策与偏离"段

**何时可重新评估**
- 引入 SSR 后（届时方案改为基于 cookie，而非 localStorage）
- React 引入官方主题 API 后

---

<!-- 后续条目按 L-002, L-003 ... 编号追加 -->
