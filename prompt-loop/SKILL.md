---
name: prompt-loop
description: >-
  诊断并改写 prompt，或设计 Cursor agent 循环（/goal、/loop、hooks、sub-agents）。
  用户问如何写 prompt、优化指令、设计 agent 自动化、loop engineering、验证器循环时触发。
disable-model-invocation: true
---

# Prompt & Loop Engineering

**Prompt 优先。** 循环只是让好 prompt 反复执行并自带验证；prompt 写烂了，loop 只会更快地产出垃圾。

## 触发时先诊断

| 用户意图 | Workflow | 交付物 |
|----------|----------|--------|
| 写/改/审计 prompt | [Prompt Workflow](#prompt-workflow) | 改写版 prompt + 逐条改动理由 + 验证标准 |
| 设计 agent 循环/自动化 | [Loop Workflow](#loop-workflow) | loop spec（原语、验证器、上限、状态、hook） |
| 概念科普 | 读参考文件后简要回答 | 要点 + 链接 |

**参考文件（按需读取，prompt 为重点）：**

- **[prompt-reference.md](prompt-reference.md)** — Prompt 编写完整指南（10 要素、模板、审计、反模式）
- **[loop-patterns.md](loop-patterns.md)** — Loop 六要素、模式、失败模式、Cursor 原语
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

## Loop Workflow

Loop 的前提是 **prompt 里已嵌入验证面**。先确认 prompt 合格，再设计循环。

### Step 1：判断是否需要 loop

| 一次 prompt 够 | 需要 loop |
|----------------|-----------|
| 步骤可预测、一次会话能完成 | 需要试错、修改、重跑验证 |
| 你全程在场 | 你不在时也要继续 |
| 可逆、本地、低成本 | 共享文件、CI、多步 pipeline |

### Step 2：选 Cursor 原语

| 需求 | 用这个 | 详见 |
|------|--------|------|
| 做到验证通过才停 | `/goal` + `cursor-goal` | `goal/SKILL.md` |
| 周期检查/监控 | `/loop` | `loop/SKILL.md` |
| 人不在时定时跑 | 系统 cron + headless `agent`，或 Cloud Agent | `cursor-cli/SKILL.md` |
| 拦截危险操作 / 阻止过早结束 | hooks | `create-hook/SKILL.md` |

**没有 `/schedule` slash command。** 离线定时用 cron/webhook，不是 Cursor 内置命令。

### Step 3：设计六要素

执行者、验证者（必须独立）、状态文件、迭代上限、skill/hook、隔离（worktree 如需并行）。

详见 **[loop-patterns.md](loop-patterns.md)**。

### Step 4：交付 loop spec

```markdown
## Loop Spec: <名称>

**目标（可衡量）：** ...
**执行 prompt：** （附完整 XML prompt）
**验证命令：** `npm test` / `pytest` / ...
**原语：** /goal | /loop | cron
**验证者：** 独立 shell / subagent / hook（禁止执行者自评）
**状态文件：** AGENTS.md / .goal/
**上限：** N 轮；同 root cause 失败 2 次 → 升级人类
**Hook（可选）：** stop → 检查验证命令退出码
```

---

## 快速交叉引用

| 主题 | 去哪里 |
|------|--------|
| Prompt 完整指南 | [prompt-reference.md](prompt-reference.md) |
| Loop 模式与反模式 | [loop-patterns.md](loop-patterns.md) |
| 改写示例 | [examples.md](examples.md) |
| `/goal` 实操 | `goal/SKILL.md` |
| `/loop` 实操 | `loop/SKILL.md` |
| Hook 编写 | `create-hook/SKILL.md` |
| Skill 编写 | `create-skill/SKILL.md` |

## 今晚三步

1. `.cursor/rules/` 加一条：改 `src/` 后跑测试，失败修到绿
2. 把高频任务写成 skill（`.cursor/skills/<name>/SKILL.md`），loop 调 skill 而非贴长 prompt
3. 需要「没过验证不让停」→ `/create-hook` 写 `stop` hook
