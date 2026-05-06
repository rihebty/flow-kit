---
name: <项目名>
description: <一句话美学定位，例：温暖纸面编辑式 sanctuary，单一品牌色 / brutalist 黑白网格 / 玩具糖果色>

# 所有颜色用 OKLCH。Hex 仅在已有遗留约束时允许，必须解释。
colors:
  brand: "oklch(% chroma hue)"
  brand-deep: "oklch(...)"            # hover / active 状态
  bg: "oklch(...)"                     # 主背景（不要纯白）
  surface: "oklch(...)"                # 卡片 / 提升表面
  text-primary: "oklch(...)"           # 主文字（不要纯黑）
  text-secondary: "oklch(...)"
  text-tertiary: "oklch(...)"
  border: "oklch(...)"                 # hairline 1px

typography:
  display:
    fontFamily: "<具体字体>, <fallback>, serif/sans-serif"
    fontSize: "clamp(2.5rem, 7vw, 4.5rem)"
    fontWeight: <100~900>
    lineHeight: 1
    italic: <true|false>
  headline:
    fontFamily: "..."
    fontSize: "clamp(...)"
    ...
  title: { ... }
  body:
    fontFamily: "..."
    fontSize: "1rem"
    fontWeight: 400
    lineHeight: 1.6              # 1.6 是 load-bearing
  body-lead: { ... }
  supporting: { ... }
  label:
    fontWeight: 500
    textTransform: uppercase
    letterSpacing: "0.05em"
  micro-label: { ... }
  mono: { ... }

spacing:
  # 非线性 scale，故意跳过 4px 这种均匀步长
  xs: "8px"
  sm: "16px"
  md: "24px"
  lg: "32px"
  xl: "48px"
  "2xl": "80px"
  "3xl": "120px"

rounded:
  none: "0"
  sm: "4px"
  md: "8px"
  lg: "12px"
  xl: "16px"

motion:
  ease-out: "cubic-bezier(0.16, 1, 0.3, 1)"     # expo-out 是默认
  ease-out-quint: "cubic-bezier(0.22, 1, 0.36, 1)"
  duration-fast: "150ms"             # 颜色 / 不透明
  duration-base: "300ms"             # transform
  duration-slow: "600ms"             # 编排式入场

shadow:
  hover-lift: "0 4px 24px -4px rgba(0,0,0,0.12), 0 1px 3px rgba(0,0,0,0.06)"
  card-lifted: "0 20px 40px rgba(0,0,0,0.08)"
  accent-glow: "0 20px 60px var(--color-brand-veil)"   # 仅"刻意强调时刻"
---

# UI Design: <项目名>

## 0. 视觉语汇对齐（仅 brownfield 填 · greenfield 删掉此段）

> 来自 2a 步骤 1.5。新增的元素目标是与原有 UI 在视觉上"无法区分"。

### 0.1 观察报告（代码为源）

- **Token 源**：<文件路径与定义位置>
- **主色实际比例**：<占 X%；在哪些元素上用（button / link / accent）>
- **中性色**：<背景 / 文字 / 边框 oklch(...) 含 chroma 的位置>
- **hover / focus 反馈**：<颜色变深 / transform / 阴影 / opacity，统一一两个档位>
- **动效语言**：<cubic-bezier / duration 档位 / CSS vs JS>
- **elevation 层级**：<几级阴影 / 一致性>
- **卡片密度 / rounded**：<内边距 / 圆角档 / 稀密>
- **图标库**：<lucide / heroicons / phosphor / 自绘； stroke-width>
- **文案调性**：<工程向 / 营销向 / 中性；按钮文案长什么样>

### 0.2 用户校准结论

- <用户确认：看对了 ✓ / 指出了误读……>
- <本次确定沿用 / 部分破 / 特例远离>

### 0.3 应用策略

- 沿用：<哪些维度照搬，比如字体 / 主色>
- 延伸：<哪些维度需要在原基础上扩，比如新页面需要额外的 layout token>
- 打破：<哪些维度刻意远离，比如 promo 页的渐变背景 — 仅在用户明说"本次想远离"时>

## 1. 美学北极星

> 一段话写清这个产品的视觉北极星。例：
> "Editorial Sanctuary —— 温暖纸面 + 单一磁红 + 平面静态。读起来像设计杂志，不像 SaaS 落地页。"

### v0 确认摸路

来自 2a 步骤 3。记录你在写完整版前已经用 v0 同用户确认的要点：

- **已确认的假设**（用户明说 go 的部分）：
  - <例：hero 有背景图，本次用 16:9 占位符，后续找用户要真图>
  - <例：不要 secondary button>
  - <例：默认不开 dark mode>
- **用户指出的偏差**（如有）：
  - <例：hero 不应该整屏，保留 nav 可见 → 已改>

## 2. 4 个决策问题（开工前回答）

- **目的**：<这个界面解决什么问题？给谁用？核心动作？>
- **调性**：<从清单里选 1 个：极简 / 极繁 / 编辑式 / 工业 / 复古未来 / 有机 / 奢华 / 玩具 / 粗野 / ...>
  - **理由**：<为什么选这个？>
- **约束**：<技术栈 / 性能 / 无障碍 / 品牌>
- **差异化**：<让人记住的那一件事>

## 3. 颜色系统

### Primary

- **<主色名>** `oklch(...)`：用于 <什么场景>。**绝不用于** <避免场景>

### Neutral

- **<bg 名>** `oklch(...)`：主背景。<为什么这个色温>
- **<text 名>** `oklch(...)`：主文字
- ...

### 命名规则

- **The One Voice Rule**：本项目主色只有 1 个，绝不引入第二个 hue
- **The Tinted Neutral Rule**：所有中性色含 chroma <值>，不用纯灰
- ...

## 4. 字体系统

- **Display**：<字体名>。**为什么不用 Inter/Roboto**：<理由>
- **Body**：<字体名>。**为什么这个**：<理由>
- **Mono / Label**：<字体名>

### 层次表

| 角色 | 字体 | 字号 | 字重 | 行高 | 备注 |
|---|---|---|---|---|---|
| Display | ... | clamp(...) | ... | ... | hero 标题 |
| Headline | ... | ... | ... | ... | section 标题 |
| Body | ... | 1rem | 400 | 1.6 | 段落 |
| ... | | | | | |

## 5. 间距 & 圆角 & 动效

按 frontmatter 中的 token 落实。**禁止**在 token 之外引入新数值。

## 6. 关键组件规约

### Button (Primary)

- **形状**：<sharp/rounded? radius 多少?>
- **背景**：<bg token>
- **文字**：<text token + typography token>
- **内边距**：<spacing tokens>
- **at rest**：<是否有阴影 / 边框>
- **hover**：<具体变化：颜色 / transform / 阴影 / 时长 / 缓动>
- **focus**：<focus ring 怎么显示>

### Button (Secondary)

> 慎用。如果没有 secondary 是更好的选择，写"本项目无 secondary button，次级动作用 inline link 表达"。

### Input / Field

- **at rest**：<边框 / 背景 / 内边距>
- **focus**：<边框色变化 / 是否有 backdrop glow>
- **error**：<怎么表达错误状态，颜色或图标>

### Card / Container

- **at rest**：平面 / 是否有阴影 / hairline 边框
- **hover**：响应是什么
- **嵌套规则**：是否允许 card 套 card（默认禁止）

### Navigation

- 高度、字体、间距、hover/active 表达

### Typography Hierarchy

参考第 4 节层次表。

## 7. Do's and Don'ts（本项目特定）

### Do

- ...
- ...

### Don't（除了通用 anti-patterns，本项目额外禁止）

- ...
- ...

## 8. 占位符策略（反伪造 · 对应 R8.9）

本项目所有缺素材场景按下表执行：

| 缺的东西 | 本项目有什么？ | 缺时用什么占位 | 禁什么 |
|---|---|---|---|
| 图标 | <展开：lucide-react / 其他 / 无> | `[icon]` 方块 或 `▢` 几何形 | — emoji (🚀⚡✨) / AI 粗糙 SVG |
| 头像 | <是否有头像组件> | 首字母圆 + 品牌色 | AI 生人脸 / 网抓图 |
| 图片 | <是否有 Image 组件 / CDN> | aspect-ratio 卡片 标 `16:9 image` | stock photo / AI 生成图 |
| 数据 | — | **反问用户要真数据** | 编活跃用户数 / 好评率 |
| logo | <项目有没有品牌 logo> | 品牌名文字标签 + 几何形 | AI 自绘额外图形标 |
| 客户推荐 | — | 明标 `TESTIMONIAL PLACEHOLDER` | 编用户名 + 照片 + 评论 |
| 活动计数 / KPI | <是否有真数据源> | 明标 `[awaits real metric]` | 编数字 |

红线emoji 仅在本品牌原就用 emoji（如 Notion / 早期 Linear）时才允许，本项目默认 ❌。

## 9. 反 AI-slop 自检结果

逐条对照 `@flow-kit/reference/ui-anti-patterns.md` 的"强制禁忌"段：

- [ ] 字体类禁忌：未命中
- [ ] 颜色类禁忌：未命中
- [ ] 阴影类禁忌：未命中
- [ ] 边框类禁忌：未命中
- [ ] 动效类禁忌：未命中
- [ ] 布局类禁忌：未命中
- [ ] 文案类禁忌：未命中
- [ ] 组件类禁忌：未命中

如有命中，列出条目 + 解释为什么本项目接受这个例外：

- ...

## 10. 触发任务

下一步进入 `3-task` 阶段时，把以下作为**第一批 UI 任务**：

- T-UI-01：把本文件 frontmatter 的 design tokens 物化为 CSS variables（或 Tailwind config）
- T-UI-02：实现 typography 层次（global stylesheet）
- T-UI-03：实现 Button (Primary) 组件，匹配本规约
- T-UI-04：实现 Input 组件
- T-UI-05：实现 Card 组件
- ...
