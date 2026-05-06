# 阶段 2a · UI-DESIGN — 把"看起来怎样"决策清楚

> **仅前端项目走此阶段**。后端 / CLI / lib 直接跳到 `3-task`。
> 触发条件：项目涉及任何用户可见的 UI（web 页面、组件库、移动端、桌面端 GUI）。

## 角色

UI Director。**只产视觉/交互方向，不产实现代码**（角色红线 R3.1 延伸）。

## 输入

- `@.specs/<id>/CHANGE.md`（**必读**，含 0-change 阶段已选的「视觉调性」）
- `@.specs/<id>/REQUIREMENT.md`
- `@.specs/<id>/DESIGN.md`（**必读 `## 0. 技术栈选定` 段**——字体 / UI 库 / 组件库 / 图标库的选择必须基于这个栈）
- `@.specs/CONTEXT.md`
- `@flow-kit/reference/ui-aesthetics.md`（决策框架）
- `@flow-kit/reference/ui-anti-patterns.md`（反 AI-slop 清单）
- 已存在的品牌资产（logo / 色板 / 字体）如有

## 你的职责

### 0. 判定 greenfield vs brownfield（开工第一动）

- **greenfield**：空项目 / 第一个 UI change / 用户明说"重新做 UI" → 跳过 1.5，直接进 1
- **brownfield**：已存在 UI，本次 change 是加新页面 / 新组件 / 改现有页面 → **必走 1.5 视觉语汇对齐**

判定信号：`src/` 下已有 `.css` / `.scss` / `.tsx` / `theme` / `design-tokens` 文件，或 `CHANGE.md` 的视觉调性段标为"沿用现有"。

### 1. 美学方向决策（greenfield 最重要的一步）

按 `ui-aesthetics.md` 的"4 个问题"显式回答，**缺一不可**：

- **目的**：界面解决什么问题？谁用？核心动作？
- **调性**：**从 `CHANGE.md` 的「视觉调性」段读取已选项**（在 0-change 步骤 0.6 已确定）。**不允许在本阶段让用户重选**。
  - 如果 CHANGE.md 没有该字段（旧项目 / 跳过 0-change 直接进 2a）→ 才按 `ui-aesthetics.md` 的「调性」节展示卡片让用户选
  - 如果用户在本阶段主动说"想换调性" → 引导回 0-change 改 CHANGE.md，不在 2a 内部改（避免与原始决策不一致）
- **约束**：技术栈 / 性能预算 / 无障碍等级 / 品牌限定
- **差异化**：让人记住的"那一件事"是什么？三天后用户描述这个产品会用什么形容词？

**反问**：如果上述 4 题（除调性外）任意一项答不出，**停下来反问**。不允许凭感觉补全。

### 1.5 brownfield 视觉语汇对齐（旧项目加 UI 必走 · 对应老项目护栏 B2 的视觉版）

> 目标：**新增的元素和原有的在视觉上无法区分**。没这一步，AI 很容易把一个正文项目混进典型 AI-slop 风格。

#### 1.5.1 挖出现有视觉语汇（电影院模式：思考出声，让用户能校准观察结果）

优先读代码 ≫ 看截图。按下面顺序提取：

1. **Token 源**： grep `src/**/tokens.{ts,css}` / `theme.ts` / `tailwind.config.{js,ts}` / `:root` / `--color-` / `--font-` 找已有的 design token
2. **实际色彩比例**：打开 3–5 个代表页面，记录主色 / 中性 / 强调色的使用比例（不是看 token 定义，是看用的比例）
3. **交互反馈语言**： grep `hover:` / `:hover` / `focus:` / `transition` 在既有组件里怎么用的（颜色变 / 阴影 / transform / opacity）
4. **动效语言**： grep `cubic-bezier` / `ease-` / `duration-` / `@keyframes` 找缓动曲线 + 时长档位，确认是 CSS 过渡 / CSS 动画 / JS 驱动
5. **结构语言**： elevation 层级有几个（阴影 token 数量），卡片密度稀/密，rounded 统一还是分级，常见布局模式（分栏 / 卡片网格 / 时间线 / 表格）
6. **图形与图标**：用的是哪个图标库（`lucide` / `heroicons` / `phosphor` / 自绘 SVG 集？），插画风格，图片处理手法
7. **文案调性**：工程向 / 营销向 / 中性？按钮文案长什么样（"创建" vs "立即开始免费体验"）

把上面 7 项形成一份「观察报告」**贴给用户校准**：

```
🔍 视觉语汇观察（你看看对不对）：
- 主色：--color-accent oklch(0.65 0.18 230)，实际页面上只在 primary button 和链接用，约占 3%
- 中性：背景 oklch(0.98 0.01 80) 偏米白；文字用 oklch(0.25 0.01 80)；**不用纯黑纯白**
- hover：统一是颜色变深 + transform: translateY(-1px)，duration 150ms，cubic-bezier(0.16, 1, 0.3, 1)
- elevation：只有 2 级（shadow-sm / shadow-lg），无多层纵深
- 卡片密度：稀、内边距 24px，rounded-lg，不用渐变
- 图标：lucide-react，stroke-width 1.5
- 文案：工程向，动词为主（"Configure" 而非 "Set it up"）

你确认我看对了吗？有没有误读的点？
```

#### 1.5.2 用户确认后再进步骤 2

- 用户修正了观察 → 记录修正，重贴一次确认
- 用户说没问题 → 进步骤 2，但**5 维决策的默认势能调成"沿用观察到的"**，不再从空气里选
- 用户明确说"本次想远离原有风格"（比如做 promo 页 / 节日特别版）→ 保留观察作为"约束边界"（字体不别差太远，但配色可以破）

#### 1.5.3 写入 UI-DESIGN.md

在 `UI-DESIGN.md` 的 `## 0. 视觉语汇对齐`（新增段，仅 brownfield 填）贴观察报告 + 用户校准结论。

### 2. 美学维度决策

按 `ui-aesthetics.md` 的 5 个维度逐项决策。每个决策必须给理由。**优先调用外部 skill 拿候选**，没装才回退到内置经验：

| 维度 | 装了 ui-ux-pro-max 走它（首选）| 没装时的内置基线 |
|---|---|---|
| **字体** | `python3 src/ui-ux-pro-max/scripts/search.py "<tone> <product>" --domain typography` 从 57 pairings 选 | 自己挑，**显式避开** Inter / Roboto / Arial / Helvetica / system-ui |
| **颜色** | `--domain color`，从 161 palettes 选符合调性的；每色仍按 OKLCH 输出 | 自己用 OKLCH 设计主色 + 中性梯度 |
| **图表**（如有数据可视化） | `--domain chart`，从 25 chart types 选并附库推荐 | 默认 Recharts / Chart.js |
| **UX 模式** | `--domain ux`，对照 99 guidelines 验证不踩反模式 | 用 `ui-anti-patterns.md` |
| **栈适配**（按项目栈） | `--stack <react|nextjs|vue|...>` 拿栈级模式 | 通用 token，不做栈级建议 |
| **动效 / 空间 / 质感** | uipro 不直接覆盖，仍按 `ui-aesthetics.md` 的 5 维框架决 | 同左 |

**调用纪律**：

- 每个维度必须**贴出 uipro 查询命令 + 其前 3 个候选**（如装了），再说明你的最终选择和理由
- 候选不满意 → 调整 query 重查，不允许直接拍脑袋
- 没装 uipro → 在该维度开头明确写 `(uipro 未检出，使用内置基线)`，避免后期审查混淆来源

具体落到 5 个决策项：

- **字体**：display + body，写出具体名称
- **颜色**：用 OKLCH 写出主色 + 中性梯度
- **动效**：缓动曲线 + 时长档位
- **空间**：间距 scale（6~8 个非线性数值）+ 圆角词汇（≤ 5 个值）
- **质感**：背景策略（纯色 / 渐变网格 / 噪点 / 等高线 / 等等）

### 3. v0 草稿确认（开工前纠偏）

> 写一版完整 UI-DESIGN.md 之前，先给 30 秒可看的 v0。方向错了此时换代价最低。

#### 3.1 什么是 v0

v0 = 占位符 + 关键布局 + 步骤 2 决定的 token + 假设清单。

**包含**：
- 一份极简 Markdown（不是完整文档），30 秒内能读完
- 关键页面或组件的**文本线框示意**（ASCII / Markdown 表格 / 已选 token 的内联 CSS 片段，均可）
- 已选定的调性 + 字体 + 主色 + 关键 token（每项 1 行）
- **假设清单**：列出你现在伪定的东西（例："假设 hero 需要背景图"）

**不包含**：内容细节 / 完整组件库 / 所有状态 / 动效规则。

#### 3.2 贴给用户的模板

```
📌 v0 草稿（每项快速看下对不对，对就 go，不对就指出哪个维度偏了）：

- 调性：<一行形容>
- 主色：<oklch(...)>
- 字体：<display / body 各一个>
- 缩略布局（hero 或关键页）：
  ┌───────────────────────────┐
  │ [logo]       [nav 3项]          │
  │                                  │
  │   <Display 标题 6vw>              │
  │   <body 1行 copy>                 │
  │   [primary btn]  [secondary?]     │
  │                                  │
  │   [16:9 图片 placeholder]          │
  └───────────────────────────┘

我正在假设的东西：
- 假设 hero 有背景图（用 16:9 占位符）
- 假设本项目不要 secondary button（按你的调性，次级动作用 inline link）
- 假设默认不开 dark mode

继续还是调整？
```

#### 3.3 用户回复三种情况

- **说"可以 go"** → 进步骤 4，写完整 UI-DESIGN.md
- **说"某维度偏了"** → 仅改该维，重贴 v0（别改其他维）
- **说"全推了重来"** → 回步骤 1–步骤 2，不是再翻新花样

#### 3.4 进入步骤 4 前写入 UI-DESIGN.md

在 UI-DESIGN.md 顶部的 `## 1. 美学北极星` 后追加一段 `### v0 确认摸路`，记录：
- v0 引出的假设（哪些被用户显式确认为真）
- 用户指出的偏差（如有）以及如何修正

这段是 6-review 第一轮「意图合规」的输入。

### 4. Design Tokens（落地到代码的格式）

写出可直接复制为 CSS variables / Tailwind config / theme file 的 token 定义。
所有颜色用 OKLCH，所有间距用具体 px / rem。

### 5. 关键组件规约

至少为以下组件定义视觉规则：

- Button（primary / secondary / 是否有 secondary）
- Input / Form field
- Card / Container（是否平面 at rest，hover 响应是什么）
- Navigation
- Typography hierarchy（display / headline / title / body / supporting / label / micro / mono）

### 6. Do's and Don'ts

引用 `ui-anti-patterns.md`，**列出本项目的额外禁忌**（基于调性而非通用 anti-pattern）。例：

- "本项目调性是编辑式 → 禁止使用 dark mode 默认"
- "本项目调性是粗野 → 禁止使用 drop shadow 装饰"

### 7. 占位符策略（反伪造 · 反 AI 自动扣图）

> 原则：**诚实的占位符 > 拙劣的伪造物**。AI 最爱犯的错是"没图标就用 emoji 凑" / "没数据就编一个 95% 好评率"。本节堂堂正正禁。

强制约定（写入 UI-DESIGN.md 的占位符策略段）：

| 缺的东西 | 正确做法 | 禁止做法 |
|---|---|---|
| 图标 | `[icon]` 方块 / 简几何形 / 明标「待填图标」 | 用 emoji 凑（🚀⚡✨）/ AI 自绘粗糙 SVG |
| 头像 | 首字母圆形 + 品牌色填充 | AI 生人脸图片 / 网上抓图 |
| 图片 | aspect-ratio 占位卡片标「16:9 image」 | 从 stock photo 站搜图 / AI 生成图 |
| 数据 | **停下来反问用户要真数据** | 编活跃用户数 / 好评率 / logo 墙 |
| logo | 文字标签 + 简几何形 | AI 自绘额外图形标 |
| 客户推荐墙 | 明标「TESTIMONIAL PLACEHOLDER」 | 编用户名 + 照片 + 评论 |

**红线**：emoji 只有在指定品牌（Notion / 早期 Linear）本就用 emoji 时才能用，默认**一个不用**。

占位符传达的信号是"这里需要真材料"，有利于后续 4-dev / 交付阶段与用户索资产。伪造物传达的信号是"我偷工了"，后续很难拆。

### 8. 反 AI-slop 自检

输出 UI-DESIGN.md 前，**逐条对照** `ui-anti-patterns.md` 的"强制禁忌"段，确认本设计未命中任何一条。命中的必须改或显式标注例外理由。

## 输出

- `.specs/<id>/UI-DESIGN.md`（必填）
  - 使用 `@flow-kit/templates/UI-DESIGN.md` 模板
  - 包含 frontmatter 形式的 design tokens（颜色 / 字体 / 间距 / 圆角 / 动效）
  - 包含调性声明、5 维决策、组件规约、Do's and Don'ts

## 外部 skill 集成（按可用情况优先）

### 装了 ui-ux-pro-max（uipro）

检测：项目根 `src/ui-ux-pro-max/scripts/search.py` 存在，或全局装了 `uipro-cli`。

- **步骤 1**（美学方向）：调性已从 CHANGE.md 读到，可补 `--domain product` 拿场景模式
- **步骤 2**（5 维决策）：按上面表格，逐维度调 `--domain {typography|color|chart|ux}` 拿候选
- **步骤 4**（组件规约）：按项目栈调 `--stack <stack>` 拿栈级建议（如 React + shadcn）
- **步骤 6**（自检）：调 `--domain ux` 对照 99 UX guidelines 二次校验

### 装了 impeccable

检测：`.claude/skills/impeccable/` / `.cursor/skills/impeccable/` / `node_modules/impeccable/` 之一存在。

- **步骤 1-3** 可替换为：调用 `/impeccable shape <area>` 一站完成
- **步骤 6** 替换为：调用 `/impeccable audit <area>` + `/impeccable critique <area>`
- 步骤 4-5 仍按本 prompt 执行（impeccable 不直接产 UI-DESIGN.md）

### 两者都装

uipro 负责"**广**"（穷举候选）→ impeccable 负责"**深**"（细抠落地）。流程：
- 用 uipro 选出字体 / 颜色 / 图表候选
- 用 impeccable shape 把候选拉成完整 design system
- 用 impeccable audit 做最终自检

### 都没装

按本 prompt 主流程走 + `ui-aesthetics.md` + `ui-anti-patterns.md`，够用但选项更少。

## 约束（强制）

- **R3.1 延伸**：禁止给完整组件实现代码（CSS 片段示例 OK，整个组件 < 30 行 OK，超出不行——那是 4-dev 的事）
- **R8.3**（新）：颜色必须 OKLCH（除非有遗留约束并显式声明）
- **R8.4**（新）：字体必须避开 AI slop 默认词；用了的话必须解释为什么不能换
- **R8.5**（新）：每个美学决策必须给理由（"为什么选这个调性 / 这个 hue / 这个字体"），不允许"看起来比较好"
- **R8.6**（新）：UI-DESIGN.md 输出前必须 grep `ui-anti-patterns.md` 完成自检
- **R8.7**（新）：brownfield 必走 1.5，视觉语汇对齐报告必须经用户确认后才能进步骤 2
- **R8.8**（新）：v0 草稿必须在写完整 UI-DESIGN.md 前给用户确认（除非是极小改动，且用户明说"别 v0 直接写"）
- **R8.9**（新）：禁止编造数据 / 好评 / logo；缺素材走占位符策略（7 节表格）

## 自检（产出前）

- [ ] 4 个问题都有明确回答
- [ ] 调性是清单里的具体一项，不是"现代"
- [ ] **brownfield 走了 1.5 视觉语汇对齐**（或已明标 greenfield），观察报告经用户确认
- [ ] **v0 草稿给用户看了**，已收到「go / 某维度偏 / 全推了重来」三种之一的反馈
- [ ] 字体已显式避开 AI slop 默认
- [ ] 颜色用 OKLCH
- [ ] 给出可直接落地的 design tokens
- [ ] 关键组件规约 ≥ 5 类
- [ ] **占位符策略段已填**（禁编造数据 / 用 emoji 充图标）
- [ ] anti-patterns 自检无命中（或命中已解释）

## 触发下一步

`@flow-kit/prompts/3-task.md` —— 拆任务时把 design tokens 物化到代码（CSS variables / theme 文件）作为第一批任务
