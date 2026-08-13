# UX System（产品交互与页面模式契约）

（从主文件分流，按需加载）

与 [ui-contract.md](ui-contract.md) 分层：ui-contract 管**长什么样**，本文件管**怎么交互、页面怎么组织、什么情况下怎么做**。**Design System 防止它写丑，UX System 防止它写蠢。**

视觉上没问题的页面可能用起来很蠢：

- 搜索和筛选藏得太深、高频操作要点 3 次
- 删除按钮放在主操作旁边、编辑一个字段却弹全屏 Modal
- 表格一行塞 8 个按钮、保存后没有反馈
- 空状态只有"暂无数据"、操作完成后不知道下一步去哪
- 设置页所有内容堆成一个大表单、移动端照搬桌面布局

这些不是 `Button` / `Card` / spacing token 能解决的。

## 16. UX SYSTEM (Interaction & Page Pattern Contract)

### 16.A 四层模型

```text
Design System      长什么样
↓
Interaction Patterns  怎么交互
↓
Page Patterns      页面怎么组织
↓
Product Heuristics   什么情况下应该怎么做
```

真正喂给 AI 的是后三层。

### 16.B Interaction Patterns：什么时候用什么

不要只定义 Dialog / Sheet / Tabs 长什么样，要定义**什么时候用**。

**Editing 决策树：**

```text
简单字段（1 个）    → inline edit
3~6 个字段        → Sheet
复杂对象 / 独立流程  → dedicated page
```

禁止为编辑一个 name 字段打开完整 Dialog。

**Destructive 决策树：**

```text
可恢复    → 直接执行 + Undo Toast
不可恢复  → Confirmation Dialog
高风险    → 用户输入对象名称确认
```

**摩擦匹配风险（Nielsen H5）**：确认框只留给真正破坏性操作，过度使用会被用户无视。反馈强度匹配操作重要性：轻操作轻微反馈，破坏性操作显著确认。

### 16.C Page Patterns：有限页面原型

后台系统通常就这几种 archetype。AI 必须先识别并套用已有原型，**不允许从零设计页面结构**：

```text
List Page / Detail Page / Create-Edit Page / Settings Page
Dashboard / Wizard / Search-Browse Page
```

**List Page 模板：**

```text
PageHeader    title + description + primary action
Toolbar       search + filters + secondary actions
Content       DataTable / List
Footer        pagination
```

**Detail Page 模板：**

```text
DetailHeader   breadcrumb + title + status + actions
Summary
Main content   primary information / secondary information
Danger Zone
```

### 16.D Product Heuristics（If X → Prefer Y / Avoid Z）

LLM 擅长「当前场景 → 匹配 heuristic → 做决策」，不擅长「凭空像资深产品设计师一样判断」。写成规则表，每条附 why：

| If | Then | Why |
|---|---|---|
| 操作高频 | 保持可见 | 减少点击成本 |
| 操作低频 | 放进 overflow menu | 不占主界面 |
| 操作破坏性 | 与主操作视觉分离 | 防误触 |
| 用户需对比条目 | 用 table/list 而非 card | 卡片弱化对比 |
| 用户需扫描 | 避免过多 card | 信息密度失控 |
| 任务 < 5 个简单字段 | 不建多步 wizard | 流程大于内容 |
| 页面有一个明显主任务 | 该操作视觉主导 | 首屏引导 |
| 数据加载中 | 保留布局 + skeleton | 系统状态可见（Nielsen H1） |
| 无数据 | 解释原因 + 给下一步操作 | 空状态是引导不是终点 |
| 操作完成 | 必须给反馈 | H1：状态可见 |
| 用户可能出错 | 预防优于恢复（约束输入、默认值） | H5 |
| 出错后 | 说明原因 + 给出解法，不用错误码 | H9 |
| 需要记忆 | 信息展示在需要处，不靠回忆 | H6 |
| 页面间 | 内外标准统一，偏离惯例需有收益 | H4 / Jakob's Law |
| 移动端 | 重新设计布局，不是机械缩小 | 触控目标、单列流 |

### 16.E 正反例（contrastive examples）

默会知识最难用纯文字传递。AI 对正反例的理解远强于长篇规范。

```text
Bad:  用户列表一行放 Edit / View / Reset Password / Disable / Delete
Good: 主行操作 View；其余进 ··· overflow
      ├ Edit
      ├ Reset password
      ├ Disable
      └ Delete
```

```text
Bad:  点击"修改用户名"→ 打开包含 12 个字段的大 Modal
Good: 点击用户名 → inline edit → Enter 保存 → Esc 取消
```

```text
Bad:  Reset password 做成主按钮（账户管理页高频 AI 错误）
Good: Reset password 是次级操作，永不作为主 CTA
```

### 16.F UX Plan：实现前先回答 5 个问题

很多 AI 前端问题本质上是：**写 JSX 前没有建立用户任务模型**。实现前必须回答：

1. 用户来到这个页面的主要目标是什么？
2. 页面最重要的 primary action 是什么？
3. 用户最常执行的 3 个操作是什么？
4. 哪些信息应该首屏看到？
5. 哪些操作应该隐藏、延后或减少打扰？

### 16.G UX Review：独立审查，不改代码

实现 agent 审自己不可靠，需要独立的 ux-review 步骤。输入：需求 + 页面截图 + 交互流程 + UX rules。输出按 8 维度 + 严重级：

```text
Hierarchy / Discoverability / Efficiency / Feedback
Error prevention / Consistency / Cognitive load / Responsive
```

```text
P1   Delete 和 Save 视觉权重相同，容易误操作。
P1   用户完成筛选后没有明显 Clear Filters。
P2   编辑单个字段使用 Modal，交互成本过高。
P2   空状态没有下一步 CTA。
```

**Screenshot 必须**：按钮太多、层级不清晰、首屏密度过高、两个 Card 权重相同、过滤器占过多空间——这些从源码看不出来。流程：

```text
frontend task → run app → capture screenshot
→ review screenshot（current vs reference page：布局/密度/交互入口/视觉层级偏差）
→ interaction check
```

### 16.H UX Lint Checklist（每次完成逐条过）

像 ESLint，只不过 lint 的是产品体验：

- [ ] Primary action 是否唯一且明确？
- [ ] 高频操作是否直接可见？
- [ ] 低频操作是否过度占用页面？
- [ ] 危险操作是否与主操作分离？
- [ ] 操作后是否有 feedback？
- [ ] loading / empty / error / success 状态是否齐全？
- [ ] 表单是否有不必要字段？
- [ ] 简单任务是否用了 Modal / Wizard？
- [ ] 用户是否需要来回跳转完成简单任务？
- [ ] 是否保持已有信息架构（未引入新 interaction pattern）？
- [ ] 移动端是否只是机械缩小桌面布局？

### 16.I Reference Implementation（性价比最高的一招）

与其写"List Page 应该……"，不如直接说：

```text
UsersPage      是标准 List Page
BillingPage    是标准 Settings Page
OrderDetailPage 是标准 Detail Page
```

新任务（如 Products 页）→ 先搜 closest reference implementation → `ProductsPage ≈ UsersPage` → 按已有页面结构实现。**默会知识很多已经存在于你觉得好用的页面里：不要全部转成语言，部分直接转成 reference。**

Pattern Registry（每个 pattern 很短，几十行）：

```text
Pattern: List Page

Use when:
- user browses many entities
- user needs search/filter/sort

Structure:
- PageHeader
- Toolbar
- Content
- Pagination

Primary action:
- Create X

Row actions:
- one visible action max
- rare actions in overflow

Avoid:
- card grid for highly comparable tabular data
- more than one dominant CTA
- filters hidden behind modal on desktop

Reference:
src/pages/users
```

### 16.J 知识沉淀循环（不要一次写 100 页）

不需要一开始把脑子里的经验全部 dump。像知识管理一样沉淀：

```text
AI 做错 → 你指出 → 判断是否重复性错误 → 是 → 沉淀为 heuristic / pattern / example
```

例：AI 第一次把 Reset password 做成主按钮 → 纠正；第二次又犯 → 沉淀：

```text
Account management:
Reset password is a secondary action.
Never present it as the primary CTA.
```

配合 spec 结构（Trellis 风格）：

```text
.trellis/spec/frontend/
├── design-system.md  长什么样
├── interaction.md    怎么交互
├── page-patterns.md  页面怎么组织
├── ux-heuristics.md  什么情况下怎么做
└── references.md     哪个页面是标准答案
```

frontend task 全流程：

```text
需求 → 识别 page pattern → 找 reference implementation
→ UX plan（5 问）→ Implement → Visual Check → UX Review
→ Finish → 新可复用知识回写 spec
```

这就是 **AI Frontend Playbook**：Patterns + Heuristics + References + Anti-patterns，四类东西，控制在千行内。
