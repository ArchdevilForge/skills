# UI Contract（产品 UI 一致性契约）

（从主文件分流，按需加载）

适用场景：**dashboard、管理后台、数据表格、多步产品 UI**。营销页/作品集/Redesign 走主文件 anti-slop 规则，两套规则不混用。这一层解决的是 AI Coding 最大的前端痛点：**多页面应用 UI 越写越散**。

## 15. UI CONTRACT (Product UI Consistency Enforcement)

### 15.A 问题根源：组件库 ≠ Design System

shadcn/ui 给你 `Button` / `Card` / `Dialog` / `Input` / `Table`，但它没有告诉 AI：

- 页面标题应该多大？页面左右 padding 是多少？
- Card 默认 padding 多大？表格工具栏长什么样？空状态怎么画？
- 详情页 Header 怎么组织？Filter 怎么排？
- 什么时候用 Dialog、什么时候用 Sheet？
- 一页最多几个 Primary Button？

于是 AI 每写一个页面都在重新"设计"一次：Users 页 Header 32px / Orders 页 28px / Products 页 30px。单独看都没问题，项目整体像 10 个设计师各写了一页。

根源有五层：

1. **组件库 ≠ Design System** —— 没有 UI Contract，AI 只能临场发挥。
2. **AI 是局部最优，不是全局最优** —— 每个页面在独立 context 下生成，看不到"正确"长什么样。
3. **Tailwind 自由度放大** —— `p-4/p-5/p-6`、`rounded-md/lg/xl`、`gap-3/4/5`。业务页面不该拥有无限 CSS 表达能力。
4. **缺页面级组件** —— 只有 `components/ui/*` 原子组件，没有 patterns 层。
5. **缺视觉 review** —— lint / typecheck / test 抓不到"这个按钮怎么比其他页面大、圆角为什么突然变 16px"。

### 15.B 核心原则：Agent 组合 Design System，而不是创造

普通 feature 的流程：

```
已有组件 + 已有 pattern + 已有 token → 组合页面
```

而不是：

```
需求 → AI 自由发挥设计
```

只有用户明确说 redesign 才允许改变设计语言。核心思想与 LLM Coding 天然契合：**减少 Agent 的 decision space —— 可决定的东西越少，整体越稳定**。

### 15.C UI Contract 最小形态：一个文件

`.trellis/spec/frontend/ui-system.md`（或项目内 `docs/ui-system.md`），几百行即可。模板：

```markdown
# UI System

## Foundations

Page max width:
- dashboard: max-w-7xl
- settings/forms: max-w-4xl

Page spacing:
- desktop: px-6 py-6
- mobile: px-4 py-4

Vertical rhythm:
- page sections: gap-6
- card content: gap-4
- compact controls: gap-2

Radius:
- controls: rounded-md
- cards: rounded-lg
- never rounded-xl/2xl unless explicitly required

## Typography

Page title:   text-2xl font-semibold tracking-tight
Section title: text-lg font-semibold
Description:  text-sm text-muted-foreground

## Pages

Every standard page uses:

<PageContainer>
  <PageHeader />
  <PageContent />
</PageContainer>

Do not manually recreate PageHeader.

## Cards

Use <Card>. Default: rounded-lg, border, shadow-sm, p-6.
Do not manually create card-like divs.

## Tables

All management tables use <DataTable> + <FilterBar> + <Pagination>.
Never build page-specific table toolbars.

## Actions

One primary action maximum per section.
Primary: <Button> / Secondary: <Button variant="outline"> / Danger: <Button variant="destructive">

## Prohibited

Do not introduce: arbitrary hex colors, arbitrary spacing values,
arbitrary border radius, duplicate primitive components, page-local button styles.
```

### 15.D 三层组件结构

```
components/
├── ui/        # primitive  —— 按钮长什么样
├── patterns/  # ★ 一致性关键层 —— 页面长什么样
└── features/  # domain    —— 业务是什么
```

patterns/ 至少包含：`PageContainer`、`PageHeader`、`PageActions`、`Section`、`DataTable`、`FilterBar`、`EmptyState`、`StatCard`、`FormSection`、`DetailSection`。

大多数 AI 项目只有 `ui + features`，**缺的正是中间这一层**，而它是 UI consistency 的关键。例如页面头不要手写：

```tsx
<div className="flex items-center justify-between">
  <div>
    <h1 className="text-3xl font-bold">Users</h1>
    <p className="text-muted-foreground">Manage users</p>
  </div>
  <Button>Add user</Button>
</div>
```

而是：

```tsx
<PageHeader title="Users" description="Manage users" actions={<Button>Add user</Button>} />
```

AI 根本没有机会把 Header 写歪。这是解决问题最有效的一步。

### 15.E Token 收紧

禁止任意值：`mt-[13px] bg-[#171717] rounded-[11px]`。只允许从有限 token 中选择：

```css
--radius-control / --radius-card
--spacing-page / --spacing-section / --spacing-control
--color-background / --color-surface / --color-border / --color-muted / --color-primary
```

Tailwind 层面自定义语义化 utility（`page-padding`、`section-gap`、`card-padding`），把"自由 CSS"变成"有限设计语言"。

### 15.F Agent 工作流（mandatory）

每次创建新 UI 前，先搜索再写：

```
1. Inspect existing components/patterns.
2. Find the closest existing page.
3. Reuse existing layout and interaction patterns.
4. Do not introduce new visual patterns unless necessary.
5. If a reusable pattern is missing, create it under
   components/patterns/ instead of implementing it locally.
```

完成后加视觉 review —— 对产品 UI 它与 lint/typecheck/test 同等重要：

```
Implement → lint → typecheck → test → browser screenshot → visual review
（与项目已有页面逐项对比：字号/圆角/padding/背景/宽度）→ finish
```

实现中产生可复用 pattern → 沉淀回 `components/patterns/` + 更新 `ui-system.md`（实现中产生知识 → 沉淀回 Spec）。
