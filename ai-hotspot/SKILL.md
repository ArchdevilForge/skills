---
name: ai-hotspot
description: >-
  Find AI and tech hotspots with high viral potential on both Xiaohongshu and X/Twitter.
  Prioritize fresh model releases, open-source drops, tool acquisitions, pricing shocks,
  and practical agent updates that are easy to rewrite into short notes or threads.
  Use when user asks for 热点, 可发内容, 爆款选题, AI新闻, 今日可搬, or daily content ideas
  for 小红书 and 推特.
metadata:
  version: 1.0.0
---

# 双平台爆款热点雷达（小红书 + X）

找出当下有爆款潜力的 AI/科技热点，输出**可搬运的选题清单**（含双平台角度），不写全文。核心要求：新鲜、有硬数字、有反差、用户容易加个人角度改写成自己的内容。

## When to Activate

- 用户说「找热点」「今日可发」「有什么值得搬」「爆款选题」「AI新闻推荐」
- 用户要小红书 + 推特双平台的每日/实时内容选题
- 与 x-viral skill 配合：ai-hotspot 出选题，x-viral 出成文

## Workflow

### 1. 搜源（按顺序）

1. **AIHOT**（首选）：`https://aihot.virxact.com/`（精选 + 今日热点 + 日报）；有 `/hot`、`/daily` 端点就探测，抓不到就用页面
2. **实时搜索**：exa web search「AI latest news [日期]」「open source LLM release」「模型发布 开源」等
3. **补充源**：Hugging Face / GitHub trending（AI 仓库）、中文科技媒体（IT之家、量子位、36氪）出国内角度、X 上 AI 大 V 关键词（Grok、OpenAI、Anthropic、DeepSeek、Qwen、GLM、Cursor）

### 2. 打分（按权重排序）

| 标准 | 权重 |
|------|------|
| 新鲜度：48-72h 内首发 | 最高 |
| 硬数字/首创：「首个开源」「提升 50%」「价格减半」「家用显卡可跑」「被收购」 | 高 |
| 情绪钩子：惊讶、FOMO、国产自豪、省钱、权力转移 | 高 |
| 实操价值：用户今天/下周就能试 | 中 |
| 双平台适配：能同时写成小红书笔记 + X 短帖 | 中 |
| 低竞争窗口：刚发布还没被刷屏 | 加分 |

### 3. 过滤（MUST NOT 入选）

- 纯观点/模糊趋势，没有硬料
- 超过 5-7 天的旧闻（除非还在持续加热）
- 难以个人化改写的内容
- 法律/合规风险高的内容（内幕、未证实传闻）
- 用户只要文字热点时，排除纯视频演示类

### 4. 输出（固定模板）

```
【今日可发热点 · 双平台爆款潜力】

1. [热点名称] — 热度/新鲜度
   核心卖点: ...
   小红书角度: 标题钩子建议 + 为什么能爆
   X/推特角度: 一句话核心 claim + 数据
   建议形式: 笔记 / 单帖 / 短Thread
   为什么值得发: ...

2. ...
```

- 最多 5-8 条，潜力最高的在前；可附 1-2 条「次优」
- 只推**能用当前搜索结果验证**的；标注事件的大致发布时间/热度峰值
- 有早期互动信号（已有人在发、数据在涨）的优先标注

## 平台信号

**小红书**：标题带数字/对比/疑问/「国产/开源/家用」；短段落 + 列表 + 个人试用感（「我试了下」）；触发收藏/分享（实用工具、省钱、榜单跃升）

**X/推特**：开头强 claim + 具体指标；附原始来源链接；能引发 quote 转发和开发者讨论；戏剧/权力转移角度（收购、价格战、开源反超）

## 每日模式

用户要长期管线时：每天早上一遍 AIHOT → 按上述标准打分 → 标注可组合的选题（如两个竞品模型同日发布 = 对比帖）

## 工具

- exa_web_search_exa：搜最新 AI 新闻、具体模型名、开源/发布
- exa_web_fetch_exa：抓 AIHOT 页面 / 原始来源
- 没有 X 实时搜索 API —— X 端热度靠 AIHOT 和搜索结果推断，不要假装有实时 X 数据

## 爆款元素速查

- 纯新闻 = 转发新闻没流量。爆款 = 热点钩子 + 实用价值（教程/避坑/合集/测评/攻略）
- 双平台配方：推特 = 利益叙事（赚钱/亏钱引站队），小红书 = 学习资产（可收藏的干货）
- 标题：数字 / 对比 / 疑问 / 「国产·开源·家用」词 / 情绪词；小红书标题权重最高的位置是前 8 字
- 细节见 x-viral skill 的 references/strategy.md（标签策略、爆款公式）

## Verify

- [ ] 每条都有可验证的来源（链接/数字/发布时间）
- [ ] 每条都给了双平台角度 + 建议形式
- [ ] 无超过 7 天的旧闻、无纯观点、无高风险内容
- [ ] 未写全文（除非用户明确要求）

## 示例触发

- 「帮我找今天能发的热点」
- 「有什么 AI 新闻在小红书和推特都有望爆」
- 「每日热点推荐，只要可搬的」
- 「GLM / Qwen / Cursor 相关有没有新的可发点」
