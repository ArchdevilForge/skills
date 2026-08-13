# Visual QA（声称完成前的截图验收）

（从主文件分流，按需加载）

源码过 lint ≠ 页面过关。没截图、没过 checklist，不算完成。用户明确说跳过视觉验收时才跳过。

与 [ui-contract.md](ui-contract.md) / [ux-system.md](ux-system.md) 分层：那两份管规则；本文件管**怎么拍、怎么看、何时算过**。

## 完成标准

- 至少一张桌面截图（1440×900）已用 Read 看过
- 产品 UI 另加一张移动端（390×844）；营销页至少再加一张移动端或 dark mode
- 对照下面清单，P1 未修 → 未完成
- 截图文件路径写进交付说明

## Capture（按顺序试，命中即停）

先把 app 跑起来（项目已有 `dev` script 就用它）。URL 默认 `http://localhost:3000`，以实际为准。

1. 项目已有 Playwright / Cypress 截图或 e2e → 跑现成的
2. `npx --yes playwright screenshot <url> <out.png> --viewport-size=1440,900`
3. 当前会话已有浏览器 MCP / `js-reverse_take_screenshot` → 用它
4. 本机有 Chromium：`chromium --headless --screenshot=<out.png> --window-size=1440,900 <url>`
5. 以上都没有 → 列出 URL + 要拍的状态（默认 / loading / empty / error），请用户给图。**不要在没图时宣称视觉验收通过。**

产品 UI 还要拍：空状态、错误状态（有的话）。营销页还要拍：light 和 dark（做了双色就两张）。

然后 **Read 截图文件**。只看源码不算。

## 对照什么

**产品 UI**（dashboard / 后台 / 表格）：

- 与最近的参考页并排：字号、圆角、padding、背景、宽度是否同一套
- [ui-contract.md](ui-contract.md) 的 token / page pattern
- [ux-system.md](ux-system.md) 16.H checklist + 16.G 八维（Hierarchy / Discoverability / Efficiency / Feedback / Error prevention / Consistency / Cognitive load / Responsive）

**营销页 / 作品集 / Redesign**：

- [ai-tells.md](ai-tells.md) 禁用模式
- 主文件 anti-slop：hero 是否出视口、CTA 对比度、假截图、eyebrow 密度

输出格式（有问题才写，没有就一句「visual-qa pass」）：

```text
P1  <截图里看见的问题>
P2  <…>
```
