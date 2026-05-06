# Frontend Engineer Rules — 前端任务实现硬规则

> 4-dev 阶段处理任何涉及 HTML / CSS / JavaScript / React 的任务时**必读**此文件。
> 提炼自 garden-skills `web-design-engineer`（基于 Anthropic Claude Design 系统提示词），针对 flow-kit 上下文做了简化。
>
> 阅读优先级：**硬规则（必看）> 交付清单（提交前过一遍）> 具体模式（按需查）**。

---

## 0. 适用判定

触发条件（任一命中就必读）：

- 本任务 `files` 含 `.html` / `.jsx` / `.tsx` / `.css` / `.scss` / `.vue` / `.svelte`
- 本任务 `action` 含：UI / 组件 / 页面 / 动画 / 幻灯片 / 仪表盘 / 原型 / 可视化 / 前端

不触发（后端 / CLI / lib / 纯脚本 / 纯文档）→ 不必读。

---

## 1. React + Babel 三条硬规则（**不可协商**）

这三条单独拿出来都像小事，实际上每一条都是 AI 写出 bug 然后回头 debug 两小时的高频事故源。

### 1.1 禁止用 `const styles = {...}` 作全局变量名

**问题**：多个 `<script type="text/babel">` 文件里都声明 `const styles` → 在浏览器端编译后它们会在同一作用域下互相**静默覆盖**，导致某些组件样式"莫名其妙失效"。

**正确**：用组件名做命名空间：

```jsx
const terminalStyles = { container: { ... }, line: { ... } };
const headerStyles = { wrap: { ... } };
```

或直接用内联 `style={{...}}`。**绝对不要**用 `styles` 做变量名。

### 1.2 跨文件组件必须 `Object.assign(window, {...})`

**问题**：独立的 `<script type="text/babel">` 块**作用域不共享**。你在 `Header.jsx` 里定义的 `<Header />`，在 `App.jsx` 里直接用会报 `Header is not defined`。

**正确**：组件定义文件尾部显式挂 `window`：

```jsx
function Terminal() { /* ... */ }
function Line() { /* ... */ }

Object.assign(window, { Terminal, Line });
```

### 1.3 禁用 `scrollIntoView`

**问题**：iframe 嵌入的预览环境（flow-kit 的大部分调试场景 + 在线预览工具都算）里，`scrollIntoView` 会触发外层 frame 的滚动，造成视觉错乱甚至把用户卷出预览区。

**正确**：用 `element.scrollTop = ...` 或 `window.scrollTo({ top: ..., behavior: 'smooth' })`。

### 1.4 React CDN 版本必须固定

```html
<script src="https://unpkg.com/react@18.3.1/umd/react.development.js"
        integrity="sha384-hD6/rw4ppMLGNu3tX5cjIb+uRZ7UkRJ6BPkLpg4hAu/6onKUg4lLsHAs9EBPT82L"
        crossorigin="anonymous"></script>
<script src="https://unpkg.com/react-dom@18.3.1/umd/react-dom.development.js"
        integrity="sha384-u6aeetuaXnQ38mYT8rp6sbXaQe3NL9t+IBXmnYxwkUI2Hw4bsp2Wvmx4yRQF1uAm"
        crossorigin="anonymous"></script>
<script src="https://unpkg.com/@babel/standalone@7.29.0/babel.min.js"
        integrity="sha384-m08KidiNqLdpJqLq95G/LEi8Qvjl/xUYll3QILypMoQ65QorJ9Lvtp2RXYGBFj1y"
        crossorigin="anonymous"></script>
```

**禁止**：在 React CDN `<script>` 标签上加 `type="module"`（会破坏 Babel 转译管线）。

**导入顺序固定**：React → ReactDOM → Babel → 你的组件文件（每个都是 `<script type="text/babel" src="...">`）。

---

## 2. CSS 硬规则

### 2.1 颜色一律来自 UI-DESIGN.md

- 颜色必须 OKLCH（遗留 hex 必须在 UI-DESIGN.md 里显式声明例外）
- 组件代码里**禁止**硬编码颜色字面值（`#fff` / `rgb(...)` / 原生色名）
- 所有颜色通过 CSS custom properties 读取：`var(--color-brand)` / `var(--color-text-primary)`

### 2.2 布局优先级

- **CSS Grid + Flexbox** 做主布局（不用 `float` / `position: absolute` 做布局）
- **`text-wrap: pretty`** 处理段落折行
- **`clamp(min, fluid, max)`** 做流体字号
- **`@container` queries** 做组件级响应（比媒体查询更适合组件复用）

### 2.3 主题适配

- `@media (prefers-color-scheme)` 处理暗色（除非 UI-DESIGN.md 明说不做）
- `@media (prefers-reduced-motion)` 处理动效降级（任何大于 200ms 的动画都要在此 disable）

### 2.4 字体硬禁

**默认禁止直接引用**（不管 UI-DESIGN.md 里有没有）：

- Inter
- Roboto
- Arial
- Helvetica
- system-ui（仅作 fallback 最后一项可以）
- Fraunces

理由：这些字体构成典型 AI-slop 视觉指纹。UI-DESIGN.md 的 2a 阶段已经选定别的字体，4-dev 阶段只需要按 UI-DESIGN.md 写，不要额外引入。

---

## 3. 文件管理规则

### 3.1 文件名

用描述性文件名：`LandingPage.html` / `DashboardPrototype.html` / `OrderCheckoutFlow.tsx`。不要 `index2.html` / `test.tsx`。

### 3.2 单文件 >1000 行要拆

拆法：

```html
<!-- main.html -->
<script type="text/babel" src="components/Header.jsx"></script>
<script type="text/babel" src="components/OrderList.jsx"></script>
<script type="text/babel" src="App.jsx"></script>
```

每个子文件尾部 `Object.assign(window, { ... })` 暴露组件（见 1.2）。

### 3.3 重大改版保留旧版

UI 类变更如果是"重做一版"，保留旧版到 `v2` / `v3`：

- `MyDesign.html` → `MyDesign-v2.html`（旧版改名保留）
- 新版直接叫 `MyDesign.html`

理由：UI 设计来回对比很常见，git 历史不够直观。

### 3.4 多变体不开多文件

多变体优先**单文件 + Tweaks 面板开关**，而不是 `MyDesign-red.html` / `MyDesign-blue.html`（见第 5 节）。

### 3.5 资产本地化

用户给的图片 / 视频 / 字体资产 → **复制到项目本地**再引用，**不要**直接 hotlink 用户的原 URL（对方改了或删了你就挂）。

---

## 4. 动画分层（从轻到重）

**从最轻的方案开始，验证不行再往上升级**。不要一上来就引入重库。

### 4.1 第一档：CSS transitions / animations

80% 的微交互（button press / card hover / fade-in entry / state toggle）用 CSS 就能搞定。

```css
.btn { transition: background 150ms cubic-bezier(0.16, 1, 0.3, 1); }
.btn:hover { background: var(--color-brand-deep); }
```

### 4.2 第二档：React state + setTimeout / requestAnimationFrame

简单逐帧或事件驱动动画：

```jsx
const [offset, setOffset] = useState(0);
useEffect(() => {
  const id = requestAnimationFrame(function tick(t) {
    setOffset((t / 10) % 100);
    requestAnimationFrame(tick);
  });
  return () => cancelAnimationFrame(id);
}, []);
```

### 4.3 第三档：自定义 `useTime` + easing + interpolate

时间线驱动、多段编排、带 scrubber 的视频级演示。模板见外部 `web-design-engineer/references/advanced-patterns.md` 的 Animation Timeline 段。

### 4.4 最后才考虑 Popmotion

```html
<script src="https://unpkg.com/popmotion@11.0.5/dist/popmotion.min.js"></script>
```

只有上面三档都搞不定才用。

### 4.5 避免引入的重库

默认**不要**引入：

- Framer Motion（与 Babel inline 模式不兼容，且体积大）
- GSAP（版本兼容问题 + 商用授权问题）
- Lottie（运行时开销 + 资产格式耦合）

除非：UI-DESIGN.md 明确要求，或场景确实非此不可（比如已有 Lottie 资产）。

### 4.6 动画必备

- **播放/暂停按钮 + 进度条（scrubber）**——给有时长的动画
- **统一 easing 库**——一个项目复用同一组 easing 函数，保持运动语言一致
- **不加标题页**——视频类产物直接进入主内容，别浪费前几秒

---

## 5. Tweaks 面板（多变体 / 参数实时调整）

### 5.1 何时用

- 多变体探索（红 / 蓝 / 绿）
- 主题切换（light / dark / auto）
- 密度切换（sparse / dense）
- 动效开关 / 内容开关

### 5.2 设计规则

- 右下角浮动面板
- 标题统一叫 **"Tweaks"**
- **关闭后完全不可见**（演示时看起来像终稿）
- 多变体用 dropdown / toggle 不用多文件
- 即使用户没要求，默认加 1-2 个创意 tweak（让用户看到可能性）

实现模板见 `web-design-engineer/references/advanced-patterns.md` 的 Tweaks Panel 段。

---

## 6. 输出类型指引（按任务类型挑一段读）

### 6.1 Interactive Prototypes（可点击原型）

- **不要加标题页 / 封面**——原型要直接居中或撑满视口，让用户立即看到产品
- 用设备外框（iPhone / Android / browser frame）增强真实感
- 实现关键交互路径让用户能点
- 至少 3 个变体，用 Tweaks 切
- 状态覆盖完整：default / hover / active / focus / disabled / loading / empty / error

### 6.2 HTML Slide Decks / 演示

- 固定画布 1920×1080（16:9），用 `transform: scale()` 自适应视口
- 居中 + letterbox 黑边，prev / next 按钮**放在缩放容器外面**（保证小屏可用）
- 键盘导航：← → 切页，Space = 下一页
- `localStorage` 保存当前页号（刷新不丢页）
- **幻灯片编号 1-indexed**：`01 Title` / `02 Agenda`，"第 5 页"对应 `05`——不要用 0-indexed 造成 off-by-one
- 每张 slide 加 `data-screen-label` 属性便于引用
- 不要塞太多文字——视觉为主，文字为辅；全套 deck 最多 1-2 个背景色

### 6.3 Data Visualization Dashboards

- Chart.js（简单）/ D3.js（复杂定制）—— CDN 加载
- 响应式图表容器（`ResizeObserver`）
- 提供暗 / 亮模式切换
- **聚焦 data-ink ratio**：去除无意义网格线 / 3D 效果 / 阴影；让数据说话
- 颜色必须**携带语义**（涨 / 跌 / 分类 / 时间），不是装饰

### 6.4 Animation / Video Demos

- 按 4.1 → 4.4 分层选方案
- 提供播放 / 暂停按钮 + 进度条
- 定义统一 easing 库
- 不加"标题页"

### 6.5 纯视觉对比 vs 完整流程

- **纯视觉比较**（按钮颜色 / 字体 / 卡片样式）→ 用 design canvas 并排展示
- **交互流程 / 多选项**→ 构建完整可点击原型 + Tweaks 暴露选项

---

## 7. 变体探索心态

**提供多变体 = 穷举可能性让用户混搭，不是给"最优解"。**

5 维探索至少覆盖：

1. **布局**：分栏 / 卡片网格 / 列表 / 时间线
2. **视觉**：配色 / 字体 / 纹理 / 层次
3. **交互**：动效 / 反馈 / 导航
4. **创意**：打破常规的隐喻 / 新型 UX / 强视觉概念

**策略**：前几个变体保守（在 design system 内）→ 后几个逐步推边界。让用户看到完整光谱：从"保守实用"到"激进冒险"。

---

## 8. 追求惊艳的具体手段

UI-DESIGN.md 的"美学北极星"是方向，下面是具体技术手段：

- 玩比例和留白创造视觉节奏
- 大字号对比（h1 和 body 的 4-6× 比例是正常的）
- 色块 / 纹理 / 层叠 / 混合模式创造纵深
- 尝试非常规布局 / 新颖交互比喻 / 细腻 hover 状态
- CSS 动画 + 过渡做精致的微交互（button press / card hover / 入场动画）
- `backdrop-filter` / `mix-blend-mode` / SVG filters / `mask` 创造记忆点

CSS / HTML / JS / SVG 能做的远超大多数人想象。

---

## 9. 合理尺度

| 场景 | 最小尺寸 |
|---|---|
| 1920×1080 演示 | 文字 ≥ 24px（建议更大） |
| 移动端 mockup | 可点击目标 ≥ 44px |
| 打印文档 | ≥ 12pt |
| Web 正文 | 从 16-18px 起 |

---

## 10. 交付前清单（**提交 PR / 合并前逐项过**）

- [ ] 浏览器 console **无错误 / 无警告**（至少 info 级以下）
- [ ] 在**目标设备 / 视口**渲染正确（响应式 Web → mobile / tablet / desktop；移动原型 → 目标设备；定尺幻灯 / 视频 → 缩放容器不失真）
- [ ] **交互组件**（button / link / input / card 等）按需包含状态：hover / focus / active / disabled / loading；按场景加 empty / error
- [ ] 文字无溢出 / 截断；已用 `text-wrap: pretty`
- [ ] **所有颜色来自 UI-DESIGN.md**（无新 hue 被悄悄引入）
- [ ] 没用 `scrollIntoView`
- [ ] React 任务无 `const styles = {...}`；跨文件组件用 `Object.assign(window, {...})` 暴露
- [ ] 没有 AI-slop（紫粉渐变 / emoji 凑图标 / 左彩边卡片 / Inter/Roboto）
- [ ] 无凑字数内容 / 无编造数据（见 UI-DESIGN.md 第 8 节占位符策略）
- [ ] 语义化命名 / 结构干净 / 后续易改
- [ ] 视觉质量达到 Dribbble / Behance 展示级别

---

## 11. 和用户协作

- **早展示半成品**：v0 占位符 + 假设清单 > 花 3 倍时间的 v1 终稿——用户能早纠偏
- 用**设计语言**解释决策（"我把间距收紧以造出工具感"），不是技术语言（"我把 padding 从 24 改到 16"）
- 用户反馈含糊时**主动追问**——不要自己脑补
- 多给变体 / 创意选项，让用户看到可能性的边界
- 总结时**只说重要的 caveat 和 next step**，不复述做了什么（代码本身就是证据）

---

## 触发下一步

- 任务完成 → 回到 `@flow-kit/prompts/4-dev.md` 步骤 4 self-review
- 需要更多代码模板 → 看 `garden-skills/skills/web-design-engineer/references/advanced-patterns.md`（幻灯片引擎 / 设备外框 / Tweaks 面板 / 动画时间线 / 设计画布 / 暗模式 / 可视化）
