# UI 美学决策框架（精简版）

> 融合自 Anthropic 原版 `frontend-design` skill + 社区 `impeccable` 项目精华。
> **本文件是 flow-kit 内置的"够用"基线**。如果装了下面任一外部 skill，优先用它们的更精细数据。

## 外部扩展（按可用优先级使用）

| Skill | 提供 | 何时压过本文件 |
|---|---|---|
| [`ui-ux-pro-max`](https://uupm.cc) | 67 UI styles + 161 color palettes + 57 font pairings + 99 UX guidelines + 25 chart types + 15+ stack guidelines | 装了 uipro 时，**调性卡片源 / 调色板 / 字体配对 / 图表选择**全部走它 |
| [`impeccable`](https://impeccable.style) | 23 个细粒度 UI 命令 + 自动检测 CLI | 装了 impeccable 时，**视觉决策深化 / anti-pattern 检测**走它 |

**调用方式**：

```bash
# ui-ux-pro-max（Python CLI）
python3 src/ui-ux-pro-max/scripts/search.py "<query>" --domain {style|product|color|typography|chart|landing|ux}
python3 src/ui-ux-pro-max/scripts/search.py "<query>" --stack {react|nextjs|vue|...}

# impeccable（npm CLI）
npx impeccable detect <file>
npx impeccable suggest <component>
```

如何检测装没装：检查项目根 / 用户全局是否有 `src/ui-ux-pro-max/` 或 `node_modules/impeccable/`。

**降级路径**：两者都没装 → 用本文件的 9 调性卡片（够用，但选项更少）。

---

## 决策的 4 个问题（开工前必须想清楚）

每个 UI 项目开始前，**显式回答**这四题，缺一不可：

### 1. 目的（Purpose）

- 这个界面解决什么问题？
- 用户在什么场景下打开它？
- 用户期待的核心动作是什么（看 / 选 / 填 / 决策）？

### 2. 调性（Tone）

**选一个明确的调性方向**，不允许"现代风"这种空话。下面是 9 种主流方向 + 卡片式说明。**0-change 阶段在识别为前端项目后必须把这一节展示给用户选**。

> **关键**：粗野和精致都能赢，关键是"明确选一个并执行到位"，不是"温和的中性"。

---

#### 1️⃣ 编辑式（Editorial）

- **关键词**：杂志感 / 衬线显示字 / 大留白 / 长文阅读 / 暖纸色
- **参考产品**：A List Apart、It's Nice That、Apple Newsroom、Pitchfork
- **适合**：内容驱动 / 品牌权威 / 长文 / 行业声音 / 医疗-人文 / 文化产品
- **不适合**：高频操作的工具 / dashboard / 数据密集型

#### 2️⃣ 极简（Minimal）

- **关键词**：单色 / 几何字 / 无装饰 / 留白 / 净中性
- **参考产品**：Linear、Vercel、Stripe（产品端）、Notion 静态页
- **适合**：技术工具 / B2B SaaS / 开发者产品 / 高频专注使用
- **不适合**：需要情感连接的消费场景 / 儿童 / 创意

#### 3️⃣ 工业（Industrial）

- **关键词**：等宽字 / 暴露网格 / 单调 / 冷色 / 数据密度
- **参考产品**：Bloomberg Terminal、Grafana、Datadog、Linear（命令面板）
- **适合**：监控 / 金融 / 工程师工具 / 数据驱动产品
- **不适合**：面向大众 / 情感场景 / 营销页

#### 4️⃣ 玩具（Playful）

- **关键词**：圆角 / 高饱和 / 弹动（克制使用）/ 插画 / 渐变可控
- **参考产品**：Duolingo、Notion（早期）、Headspace（活力页）
- **适合**：教育 / 儿童 / 协作 / 习惯养成 / 轻量消费
- **不适合**：严肃 / 专业 / 金融 / 医疗诊断

#### 5️⃣ 奢华（Luxury）

- **关键词**：衬线 / 金属质感 / 深色 + 金 / 大留白 / 慢节奏
- **参考产品**：Bottega Veneta、Aesop、Loro Piana、Apple Vision Pro 落地页
- **适合**：高端品牌 / 奢侈品 / 私人服务 / 高客单价
- **不适合**：大众消费 / 工具 / 性价比定位

#### 6️⃣ 有机（Organic）

- **关键词**：流动形 / 暖色 / 不规则 / 自然意象 / 柔和过渡
- **参考产品**：Headspace、Calm、Apple Health、Oatly
- **适合**：健康 / 冥想 / 医疗-关怀 / 食品 / 母婴 / 老年友好
- **不适合**：商业仪表盘 / 数据密集 / 严格效率

#### 7️⃣ 复古未来（Retro-Futurism）

- **关键词**：80s/90s / 像素 / 霓虹 / 方块 / CRT 噪点
- **参考产品**：Beeple、Cyberpunk 2077 落地页、Vercel Conf
- **适合**：游戏 / 创意工具 / Web3 / 反差营销
- **不适合**：严肃业务 / 工具产品 / 老年用户

#### 8️⃣ 粗野（Brutalist）

- **关键词**：单一字（通常等宽）/ 黑白 / 网格暴露 / 反美学 / 大块色
- **参考产品**：Bloomberg.com（部分）、Balenciaga 早期、独立设计师作品集
- **适合**：设计圈 / 反主流 / 艺术 / 文化机构 / 时尚先锋
- **不适合**：大众用户 / 转化漏斗 / 老年用户 / 工具型

#### 9️⃣ 极繁（Maximalist）

- **关键词**：信息密集 / 多层叠加 / 装饰主义 / 多字体 / 多色彩
- **参考产品**：Eye Magazine、It's Nice That（活动页）、独立电商
- **适合**：内容广场 / 节日主题 / 文化展览 / 品牌庆典
- **不适合**：日常工具 / 长期阅读 / 性能敏感

---

#### 给 AI 在 0-change 阶段展示用的标准模板

收到前端项目需求后，AI **必须**展示如下格式（不是引用，是实际渲染）：

```
🎨 视觉调性预选（前端项目必选）

我为你列出 9 种主流方向，请选一个数字回复（也可描述自己的想法）：

1️⃣ 编辑式 — 杂志感 / 长文阅读 / 暖纸色（参考：A List Apart、Apple Newsroom）
2️⃣ 极简 — 单色 / 几何字 / 留白（参考：Linear、Vercel、Stripe）
3️⃣ 工业 — 等宽字 / 数据密度（参考：Bloomberg、Grafana）
4️⃣ 玩具 — 圆角 / 高饱和（参考：Duolingo、Notion 早期）
5️⃣ 奢华 — 衬线 / 深色 + 金（参考：Aesop、Bottega Veneta）
6️⃣ 有机 — 流动形 / 暖色 / 自然（参考：Headspace、Calm、Apple Health）
7️⃣ 复古未来 — 80s/90s / 霓虹（参考：Cyberpunk、Vercel Conf）
8️⃣ 粗野 — 黑白 / 反美学（参考：Balenciaga、设计师作品集）
9️⃣ 极繁 — 信息密集 / 装饰主义（参考：Eye Magazine）

📌 我对你这个「<项目名>」的初步推荐：
   - 首选：<编号><名称> — 因为 <一句话理由，结合项目场景>
   - 备选：<编号><名称> — 因为 <理由>

回复数字即可（如"6"），或描述你想要的感觉（如"接近 Stripe 但更暖一些"）。
```

**推荐生成规则**（AI 不允许跳过）：

- 必须给 1 首选 + 1 备选，不允许"任选都行"
- 推荐理由必须**结合项目的具体业务**，不允许"看起来不错"
- 必须显式排除明显不合适的（例：金融工具不推荐"玩具"和"复古未来"）
- 用户已有强偏好（"参考 Notion"）→ 直接锁定对应方向，不再列卡片

### 3. 约束（Constraints）

- **技术栈**：什么框架？SSR / SPA？是否要支持 WebView？
- **性能**：首屏 LCP 目标？冷启动预算？
- **无障碍**：WCAG 等级要求？
- **品牌限定**：已有 logo / 色板 / 字体？哪些不可改？

### 4. 差异化（Differentiation）

- 这个界面让人记住的"那一件事"是什么？
- 三天后用户描述这个产品时会用到的形容词是什么？
- 如果只能保留一个视觉决策，是哪个？

---

## 五大美学维度（实施层）

完成"4 个问题"后，按下面 5 维度做具体决策。

### 1. 字体（Typography）

- **选一对**：display 字体 + body 字体。display 要有性格，body 要中性
- **绝对避开** AI slop 默认：Inter / Roboto / Arial / system-ui（除非品牌强制）
- **OpenType 特性**：开 `font-feature-settings`（合字、上下文替换、小型大写）
- **行高**：body **1.6**（这是 load-bearing 的可读性参数）
- **行长**：body 限制 65–75ch（用 `max-width`）
- **流体头部**：headings 用 `clamp()`；body 不用流体（会让行长漂移）

**字体探索方向**（避免和别人撞脸）：
- 衬线展示：Cormorant、PT Serif Display、Tiempos、EB Garamond
- 中性正文：Instrument Sans、IBM Plex Sans、Söhne 类、Inter Display（不是 Inter）
- 等宽：JetBrains Mono、IBM Plex Mono、Berkeley Mono

### 2. 颜色（Color）

- **用 OKLCH**，不用 hex（OKLCH 在亮度感知上线性，便于做无障碍）
- **倾斜中性色**：纯灰会读起来"机器感"。给中性色一点点品牌色 hue（chroma 0.005~0.02 即可）
- **一个声音原则**：主色调只有 1 个，靠它的稀缺性出彩。需要第二个强调点时**用粗细或字号**，不要加第二个 hue
- **暗色不等于黑**：避免 #000 / #fff，用 oklch(10% ...) / oklch(98% ...) 替代
- **对比验证**：用真实的对比检查工具，不是肉眼

### 3. 动效（Motion）

- **缓动曲线**：用 expo-out（`cubic-bezier(0.16, 1, 0.3, 1)`）作默认
- **禁止 bounce / elastic**：现实中物体平滑减速，bounce 是 2014 年的过时审美
- **时长档位**：颜色 / 不透明 150ms；transform 300–400ms；编排式入场 600–1200ms
- **只动 transform 和 opacity**：不要 animate `width` / `height` / `padding`（重排成本高）
- **prefers-reduced-motion**：必须支持
- **集中爆发优于分散**：一次精心编排的入场（stagger reveal）比满屏微交互更打动人

### 4. 空间布局（Spatial）

- **不对称比对称好**：editorial-style 的非对称二栏比标准三栏更难忘
- **打破网格**：给一个元素故意越界（overlap / 大留白）
- **间距比例**：用类似 8/16/24/32/48/80/120 的非线性 scale，不要 4/8/12/16/20 这种均匀步长
- **密度二选一**：要么大留白（editorial），要么受控密度（dashboard），不要中间态

### 5. 背景与质感（Atmosphere）

- 别只用纯色背景。考虑：
  - 渐变网格（gradient mesh）
  - 噪点纹理（noise overlay）
  - 几何图案 / 等高线
  - 分层透明度
  - 戏剧性阴影
  - 自定义光标
  - 颗粒感叠加

---

## 阴影词汇（控制使用）

- **平静默认**：组件 at rest 不带阴影。质感由排版和留白承担
- **状态响应**：阴影只在 hover / focus / 主动抬起时出现
- **低 alpha**：每个阴影最强 blur 的 alpha ≤ 0.15。高了就是 2014 Material Design 的味道
- **着色阴影限定**：只有"刻意的强调时刻"用主色调染阴影，绝不为装饰染

---

## 触发任务的检查清单

UI 任务进入实现前，AI 必须能回答以下问题，否则**停下来反问**：

- [ ] 调性是哪一个？（说出名字，不许"现代")
- [ ] display 字体和 body 字体是？（说出名字，不许"系统字")
- [ ] 主色 hue 是？（OKLCH 写出来）
- [ ] 间距 scale 是？（8 个数值列出来）
- [ ] 一个会让人记住的视觉决策是？（一句话讲清）
- [ ] 已扫描过 `ui-anti-patterns.md` 并确认未命中？

---

## 装了 impeccable 的话怎么办

如果项目里有 `.claude/skills/impeccable/` 或类似目录，**优先调用** impeccable 的命令：

- `/impeccable shape <area>` — 替代本文档的"4 个问题 + 5 维度"环节
- `/impeccable critique <area>` — UX 评审
- `/impeccable audit <area>` — 技术质量
- `/impeccable polish <area>` — 出货前打磨
- `/impeccable harden <area>` — 错误处理 / 边界 / i18n

flow-kit 与 impeccable 互补：flow-kit 决定**什么时候做 UI 决策**，impeccable 决定**做得多深**。
