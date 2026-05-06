# UI 反模式清单（grep 用）

> 来源：Anthropic `frontend-design` skill + impeccable 项目的 anti-pattern 库。
> 用法：每个 UI 任务进入实现前 grep 此文件；提交前再扫一遍。

---

## 强制禁忌（命中即必须改）

### 字体类

- ❌ **Inter / Roboto / Arial / Helvetica / system-ui** 作主字体（除非品牌强制）
- ❌ Space Grotesk 用作 display（已被用烂）
- ❌ Display 和 body 是同一个字体的不同字重（缺乏对比）

### 颜色类

- ❌ **纯黑 `#000` / 纯白 `#fff`**（永远用倾斜中性色）
- ❌ **紫色渐变 on 白底**（最强 AI 标志）
- ❌ **霓虹青 on 黑底**（AI 工具营销页常见）
- ❌ **彩色背景上配灰色文字**（对比度不够 + 工程师配色）
- ❌ **两个或更多强调色**（一个声音原则）
- ❌ **Gradient text**（`background-clip: text` + 渐变）已经 cliché

### 阴影类

- ❌ 静态卡片带 drop shadow（at rest 应该是平的）
- ❌ Shadow alpha > 0.15（2014 Material 味）
- ❌ 装饰性彩色阴影（着色阴影只给"刻意强调时刻"）
- ❌ Inset shadow + glow 同时用（油腻）

### 边框类

- ❌ **`border-left` 或 `border-right` > 1px 作彩色侧条**（卡片/列表/alert 的最强 AI dashboard 标志）
- ❌ 渐变边框（gradient border）作装饰
- ❌ 玻璃拟态（glassmorphism）— 模糊半透明卡片 + 玻璃边

### 动效类

- ❌ **Bounce / elastic 缓动**（2014 之后已过时）
- ❌ animate `width` / `height` / `padding` / `margin`（性能差，应只动 transform/opacity）
- ❌ 散落的微交互（不如一次编排好的入场动画）
- ❌ 不支持 `prefers-reduced-motion`
- ❌ 滚动劫持（scroll hijacking）覆盖原生滚动

### 布局类

- ❌ **卡片嵌套卡片**（hierarchy flatten 优先）
- ❌ **统一大小卡片网格 + 图标 + 标题 + 文本 + 重复 N 次**（SaaS template）
- ❌ **Hero 大数字 + 小标签 + 副指标 + 渐变装饰**（SaaS hero-metric cliché）
- ❌ 统一 4/8/12/16/20 间距（用非线性 scale 替代）
- ❌ 默认 dark mode（除非产品就是夜间使用，且有理由）

### 文案类

- ❌ "Boost your productivity" / "Unleash your potential" 类空话
- ❌ "Maybe consider..." / "It might be helpful..."（hedging）
- ❌ Lorem ipsum 出现在最终交付里
- ❌ 按钮写 "Submit" / "Click here" / "Learn more"（应具体动词）

### 组件类

- ❌ 圆角矩形 + 通用 drop shadow 的 button（任何 AI 都会做这个）
- ❌ 在 form 里用 placeholder 替代 label（无障碍灾难）
- ❌ Tooltip 在移动端依赖 hover
- ❌ 模态框无 escape 键关闭
- ❌ Skeleton loader 用纯灰条（应该用品牌色调的中性）

---

## 判断模糊地带（视情况决定）

| 模式 | 何时可用 | 何时禁止 |
|---|---|---|
| 默认字体 (Inter 等) | 行政内部工具、有强对比设计支撑 | 任何 marketing / brand 页 |
| 紫色 | 项目本来就是紫色品牌 | 通用应用为了"高级感"加紫色渐变 |
| Drop shadow | hover/focus 状态响应 | 静止卡片装饰 |
| 第二个强调色 | 语义需要（成功 / 警告 / 危险） | 视觉装饰 |
| Dark mode | 阅读 / 视频 / 编程工具 | 编辑式 / marketing |
| Glassmorphism | 极少数情况下作"玻璃 HUD"特效 | 任何 marketing 页 |

---

## 自检脚本（可选）

如果项目装了 [impeccable CLI](https://impeccable.style)：

```bash
npx impeccable detect src/                   # 扫描目录
npx impeccable detect index.html             # 扫描 HTML
npx impeccable detect --fast --json .        # 快速扫描，JSON 输出
```

输出会标记 24 类常见 anti-pattern。

如果没装：手动按上面"强制禁忌"清单 grep。

---

## 给 AI 的扫描指令（被 4-dev / 6-review 引用）

UI 任务进入实现前，AI 必须：

1. 用任务的关键词（颜色 / 字体 / 卡片 / 按钮 / 动画 / 布局 / 边框 / 阴影）grep 本文件相关章节
2. 把命中的禁忌**逐条声明**："已知 X 是禁忌，本任务不涉及 / 本任务采用 Y 替代"
3. 实现完成后再扫一遍，作为 self-review

UI review 阶段必须：

1. 把 diff 与本清单逐条比对
2. 命中即标 🔴 Critical，追加 fix 任务
