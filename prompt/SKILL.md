---
name: prompt
description: >-
  诊断并改写 prompt。用户问如何写 prompt、优化指令、审计指令时触发。
disable-model-invocation: true
---

# Prompt Engineering

**Prompt 是 agent 工作的唯一接口。** 写 prompt 不是「措辞优化」，是**规格说明**——把模糊意图变成可执行、可验证的契约。

## 触发时先诊断

| 用户意图 | Workflow | 交付物 |
|----------|----------|--------|
| 写/改/审计 prompt | [Prompt Workflow](#prompt-workflow) | 改写版 prompt + 逐条改动理由 + 验证标准 |
| 概念科普 | 读参考文件后简要回答 | 要点 + 链接 |

**参考文件（按需读取）：**

- **[prompt-reference.md](prompt-reference.md)** — Prompt 编写完整指南（10 要素、模板、审计、反模式）
- **[examples.md](examples.md)** — 完整 before/after 示例

---

## Prompt Workflow

被触发写 prompt 时，**不要直接给答案**。先审计，再交付。

### Step 1：收集（缺什么就问什么，一次问完）

- 任务：本轮要产出什么？
- 受众：输出给谁看？（用户 / 另一个 agent / CI）
- 约束：不能做什么？必须遵守什么规范？
- 上下文：已有文件、命令、错误日志？
- 验证：怎样算「写好了」？

### Step 2：审计现有 prompt

用 [prompt-reference.md 的审计清单](prompt-reference.md#审计清单) 逐条打分。标出：

- ❌ 缺失项
- ⚠️ 模糊项（多义、不可测）
- 🔴 矛盾项（规则互斥）

### Step 3：改写并结构化

输出必须用 XML 标签分隔（见 prompt-reference.md）。结构：

```xml
<role>...</role>
<context>...</context>
<task>...</task>
<rules>...</rules>
<examples>...</examples>   <!-- 格式敏感时必填 -->
<output_format>...</output_format>
<verify>...</verify>       <!-- 怎样算完成 -->
```

### Step 4：附交付说明

每次交付 prompt 时，**同时给出**：

1. **改动表**：原句 → 新句 → 为什么
2. **验证标准**：1-3 条可观测的完成条件
3. **风险**：仍可能歧义的地方，需用户确认什么

### Prompt 黄金法则（速记）

> 拿给不了解任务的同事看——看不懂，模型也看不懂。

- **精确 > 模糊**：「修复 auth 模块测试」>「修一下」
- **解释 why**：规则附原因，模型遵守率更高
- **可验证**：每条关键要求都能被检查（命令、文件、格式）
- **负向约束**：明确「不要做什么」往往比堆「要做」更有效

详细技巧、模板、反模式 → **[prompt-reference.md](prompt-reference.md)**

---

## 快速交叉引用

| 主题 | 去哪里 |
|------|--------|
| Prompt 完整指南 | [prompt-reference.md](prompt-reference.md) |
| 改写示例 | [examples.md](examples.md) |
