# L-restyle — 一键换调性（保留功能，重做视觉）

> **L 前缀表示这是 lateral / lifecycle 命令**，不在主流程编号里。
> 用法：`@flow-kit/prompts/L-restyle.md` + 一句话目标
> 例："换成有机调性"、"想要 Headspace 那种感觉"、"做个奢华版"

---

## 适用场景

- ✅ 已有项目，视觉感到陈旧 / 同质化 / 像"AI slop"
- ✅ 已有项目，品牌方向调整需要新视觉语言
- ✅ 已有项目，需要并行的"暗色版" / "高端版" 设计系统

## 不适用场景

- ❌ 全新项目（用 `0-change` → `2a-ui-design`）
- ❌ 业务功能变化（视觉变 + 功能变 = 拆成两个独立 change）
- ❌ 单组件视觉微调（直接当普通 4-dev 任务做即可）

---

## 角色

UI Director + Refactor Specialist。**保留功能不变**，只换视觉。

## 输入

- 用户的目标（一句话）
- 现有项目代码（组件、CSS / theme 文件、字体加载）
- 项目里已有的 `UI-DESIGN.md`（如有，作为 v1 调性来源）
- `@flow-kit/reference/ui-aesthetics.md`（调性卡片）
- `@flow-kit/reference/ui-anti-patterns.md`
- 项目主 `REQUIREMENT.md`（确认要保留的 AC 不变）

---

## 你的职责

### 1. 自动生成 change-id

格式：
- `restyle-<old>-to-<new>` 例：`restyle-minimal-to-organic`
- `restyle-v<N>` 例：`restyle-v2`（不知道旧调性时用）

### 2. 识别现有调性（v1）

**优先**：读 `.specs/archive/.../UI-DESIGN.md` 或仓库里现存的 UI-DESIGN.md，提取调性字段
**降级**（无 UI-DESIGN）：从代码反向提取
- 扫 CSS variables / theme.ts 看主色饱和度、字体家族、圆角值
- 按以下信号推断：
  - 字体是 Inter/Roboto + 圆角 8-12px + 中性色 → 大概率是**极简**
  - 衬线字 + 暖背景 + 大留白 → **编辑式**
  - 高饱和 + 大圆角 + 弹动动画 → **玩具**
  - 等宽字 + 暴露网格 + 单调色 → **工业**
  - 黑白 + 等宽字 + 大块色 → **粗野**
- 推断完后告诉用户："我判断现有调性是 X，对吗？"

### 3. 调性切换确认

用 `ui-aesthetics.md` 的「调性」卡片让用户选新调性 v2。展示时**额外显示**：

```
  调性切换：v1（<现有>） → v2（候选）

候选 1：<新调性 1>
  - 切换代价：<具体变什么 / 不变什么>
  - 用户感知：<访问后的第一印象>

候选 2：<新调性 2>
  ...

❌ 不建议切到：<明显不合适的调性 + 理由>
```

如果用户已说出强偏好（"参考 Notion"）→ 锁定对应调性，不再列卡片。

### 4. 影响面扫描

**强制**输出一份"动 / 不动"清单：

| 类别 | 动 | 不动 |
|---|---|---|
| Design tokens（CSS vars / theme）| ✅ 全替换 | — |
| 字体加载（@font-face / link）| ✅ 可能换 | — |
| 组件**视觉**（颜色 / 间距 / 动效 class）| ✅ | — |
| 组件**接口**（props / 事件 / 默认值）| — | ❌ 绝不动 |
| 组件**结构**（DOM 树 / 嵌套层级）| — | ❌ 绝不动（除非视觉必需）|
| 业务逻辑（hooks / store / API call）| — | ❌ 绝不动 |
| 路由 / URL / 数据库 / API 接口 | — | ❌ 绝不动 |
| 单元测试 | — | ❌ 不动（除非测视觉细节）|
| Visual regression / snapshot 测试 | ✅ **基线必须重置**| — |
| Lighthouse / a11y 报告 | ✅ 必须重测 | — |

任何"动"列发现要触及"不动"列的内容 → 立即停下，按 R7.1 开新 CHANGE 处理。

### 5. 写新 UI-DESIGN.md（标 v2）

- 路径：`.specs/<change-id>/UI-DESIGN.md`
- 在 frontmatter 加：
  ```yaml
  version: 2
  supersedes: archive/<old-change-id>/UI-DESIGN.md
  restyle_from: <old-tone>
  restyle_to: <new-tone>
  ```
- 组件段：保留**组件接口和结构**与 v1 相同（用同样的 component 名 / 同样的 props）；只改视觉规约（颜色、字体、间距、动效、圆角）
- 必须列出"v1 → v2 视觉对照表"，每个组件写"原本是 X，现在是 Y"

### 6. 拆 restyle 任务（按"token 先于组件"）

```
Wave 1（并行，纯 token 替换）：
  T01[P]：替换 design tokens 文件（CSS variables / theme.ts）
  T02[P]：替换字体加载（@font-face / preconnect / link）
  T03[P]：替换图标库（如调性变化要求换 stroke 风格 → outline、duotone 等）

Wave 2（并行，组件视觉调整。每组件一个任务）：
  T04[P]：Button 视觉重做（保留 props，只改 className 和 style）
  T05[P]：Input 视觉重做
  T06[P]：Card 视觉重做
  T07[P]：Navigation 视觉重做
  ... 一个组件一个任务

Wave 3（串行，回归与基线）：
  T-N1：更新 visual regression / snapshot 基线
  T-N2：跑 Lighthouse + a11y，对比 v1 报告
  T-N3：跑全量单测 / e2e，确认无业务回归
```

每个组件任务的 `verify` 模板：

```bash
npm test -- <Component>.test          # 单测必须仍过
npm run e2e -- <Component> --updateSnapshot  # 视觉 snapshot 重置
npx pa11y <url-of-component>          # a11y 不能退步
npx impeccable detect <component-file> # 反 AI-slop（可选，如装了）
```

### 7. 显式风险声明（**必须输出，不允许跳过**）

```
⚠️ Restyle 风险通告（请用户确认后再开工）

1. 用户感知变化
   - 老用户首次访问 v2 会感到"产品变样了"
   - 建议同步发布更新公告 / 引导浮层

2. 测试影响
   - Visual regression / snapshot 测试将批量失败
   - 需要逐个更新基线（任务 T-N1 已含）
   - 单元测试理论上全过；如果挂了，可能改动越界（按 R7.1 处理）

3. 性能影响
   - 新字体若 weight / 文件更大 → LCP 可能退步
   - 新动效若复杂 → 主线程占用上升
   - T-N2 会复测，未达 v1 基线必须修

4. 无障碍影响
   - 新颜色对比度需用工具实测（不是肉眼）
   - prefers-reduced-motion 支持必须保留
   - 焦点环新样式必须可见
```

用户回复"确认"后再进入 task 阶段。

---

## 约束（强制）

- **R3.1 延伸**：本 prompt 不写完整组件代码，只产视觉规约 + 任务清单
- **R7.1**（关键）：发现要改业务逻辑 / 接口 / 路由 → **立即停下**，开新 CHANGE。Restyle 绝不允许成为业务变更的偷渡口
- **R2.4 延伸**：每个组件任务的 verify 必须含"a11y 不退步"和"单测不变"两项
- 不允许同次 restyle 跨多个调性（要么全切到 v2，不允许 50% 编辑式 + 50% 工业）
- 测试基线更新必须**逐个组件 review**，不允许批量 `--updateSnapshot` 后直接提交

## 自检（产出前）

- [ ] 现有调性已识别（用户确认）
- [ ] 新调性已选定，用户已知代价
- [ ] 影响面"动 / 不动"清单完整
- [ ] UI-DESIGN.md v2 已写，组件接口与 v1 完全一致
- [ ] 任务按"token → 组件 → 回归"三波次拆分
- [ ] 风险通告已输出且用户已确认
- [ ] 没有任何任务触及"不动"列内容

## 触发下一步

- 用户确认 → `@flow-kit/prompts/3-task.md`（按上述波次落实任务）
- → `@flow-kit/prompts/4-dev.md`（每任务 fresh context，UI 任务自动扫 anti-patterns）
- → `@flow-kit/prompts/6-review.md`（**第三轮 UI 视觉审查必跑**，含 v1 vs v2 对比段）
- → `@flow-kit/prompts/7-integration.md`（UAT 重点：老用户能否完成关键流程；视觉是否传达声明的调性）
