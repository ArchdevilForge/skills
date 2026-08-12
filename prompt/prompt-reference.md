# Prompt 编写指南

Prompt 是 agent 工作的**唯一接口**。写 prompt 不是「措辞优化」，是**规格说明**——把模糊意图变成可执行、可验证的契约。

---

## 黄金法则

> 把 prompt 拿给一个不了解任务的同事看。他们如果看不懂，模型也看不懂。

把模型当作**聪明但零背景的新员工**：

- 它不知道你的项目惯例，除非你写进 `<context>`
- 它不知道你的优先级，除非你写进 `<task>` 和 `<rules>`
- 它不知道什么叫「做好」，除非你写进 `<verify>`

**精确 > 模糊。可验证 > 主观。**

---

## 10 个组成部分

按推荐顺序排列。不是每条都要，但缺了哪条，就在审计清单里标 ❌。

| # | 部分 | 作用 | 写法要点 |
|---|------|------|----------|
| 1 | **人设/角色** | 激活正确的知识域与判断标准 | 一句话 + 受众：`向资深后端解释` vs `向新人解释` |
| 2 | **语气** | 控制冗长度、正式度、语言 | `简洁、中文、不用敬语`；需要时说「像技术博客，不要列表堆砌」 |
| 3 | **背景数据** | 项目事实，减少幻觉 | 技术栈、目录结构、相关文件路径、已有决策 |
| 4 | **任务与规则** | 做什么、不做什么 | 任务用动词开头；规则分 MUST / MUST NOT |
| 5 | **示例（few-shot）** | 锁定输出格式与边界 | 3-5 个；覆盖 happy path + 1-2 个边缘情况 |
| 6 | **对话历史** | 多轮时传递状态 | 摘要前文结论，不要贴全文 |
| 7 | **即时任务** | 本轮具体动作 | 与长期目标分开：「本轮只改 auth.py」 |
| 8 | **分步思考** | 复杂推理时提高正确率 | 要求先分析再结论；推理放 `<thinking>` 可隐藏 |
| 9 | **输出格式** | 结构化交付 | JSON schema、Markdown 模板、字段必填/可选 |
| 10 | **预填回复** | 跳过废话开头 | 预填 `## 发现` 或 `{` 让输出直接进入正题 |

### 优先级（资源有限时）

1. `<task>` + `<rules>` + `<verify>` — 没有这三条，其他都是装饰
2. `<context>` — 项目相关任务必写
3. `<examples>` — 格式敏感或易漂移时必写
4. `<role>` — 一行即可，别写散文
5. 其余按需

---

## XML 标签结构（推荐默认格式）

模型对 XML 标签注意力更好。标签名语义化，内容简短。

```xml
<role>你是本仓库资深 Python 后端工程师</role>

<context>
- Python 3.12, FastAPI, SQLAlchemy
- 测试：uv run pytest
- 相关文件：src/auth/service.py, tests/test_auth.py
</context>

<task>
修复 test_login_expired_token 失败。只改 auth 模块，不动其他文件。
</task>

<rules>
- MUST：保留现有公开 API 签名
- MUST NOT：引入新依赖
- MUST NOT：删除或跳过测试
- 不确定时直接说「需确认」，不要猜
</rules>

<output_format>
1. 根因（1-2 句）
2. 改动文件列表
3. 每个文件的改动说明
</output_format>

<verify>
完成当且仅当：uv run pytest tests/test_auth.py 全部通过
</verify>
```

### 常用标签

| 标签 | 放什么 |
|------|--------|
| `<role>` | 身份 + 专业深度 + 受众 |
| `<context>` | 项目事实、路径、命令、约束环境 |
| `<task>` | 本轮要完成的一件事 |
| `<rules>` | MUST / MUST NOT，附原因 |
| `<examples>` | 输入输出对 |
| `<input>` | 本轮原始材料（diff、日志、代码） |
| `<output_format>` | 交付结构和模板 |
| `<verify>` | 完成判定条件 |
| `<thinking>` | 要求模型把推理放这里（可选隐藏） |

---

## 核心技巧

### 1. 解释 why，不只列 what

| ❌ 弱 | ✅ 强 |
|------|------|
| 不要用省略号 | 回复会被 TTS 朗读，省略号无法发音，所以禁用 |
| 保持简洁 | 输出给移动端，超 500 字截断，所以每段 ≤3 句 |
| 遵循项目风格 | 见 src/foo.py 的命名方式，新代码与之对齐 |

### 2. 负向约束往往比正向更有效

模型默认「尽量帮忙」。要说清楚**禁止**什么：

```
MUST NOT：
- 不要重构无关代码
- 不要在没有运行测试的情况下声称完成
- 不要修改 package-lock.json
```

### 3. Few-shot：覆盖边界，不只 happy path

```xml
<examples>
<example name="happy">
输入：用户说「加个登录接口」
输出：在 src/auth/routes.py 新增 POST /login，调用现有 AuthService.login
</example>
<example name="edge_missing_spec">
输入：用户说「优化一下」
输出：先问：优化什么指标？延迟、可读性、还是包体积？不要直接动手。
</example>
<example name="edge_ambiguous_scope">
输入：「修所有测试」
输出：先跑 pytest 列出失败列表，按模块逐个修，不要一次改 20 个文件
</example>
</examples>
```

规则：**3 个示例 > 30 行抽象规则**，当输出格式或行为容易漂移时。

### 4. 分步思考：复杂任务才用

简单任务加强制思考 = 浪费 token + 更啰嗦。

适合用的场景：多文件权衡、安全审计、架构决策、数学推导。

```
先分析问题，把推理放在 <thinking> 里。
<thinking> 内容不要出现在最终交付给用户的部分。
```

### 5. Prompt Chaining：复杂任务拆解

一个 prompt 做一件事。上一步输出是下一步输入：

```
Prompt 1（探索）：列出 auth 模块所有失败测试及可能根因，不写代码
Prompt 2（实现）：根据 <findings> 只修 test_login_expired_token
Prompt 3（验证）：跑 pytest，失败则回到 Prompt 2
```

**每环各自的 `<task>` 和 `<verify>` 必须独立可测。**

### 6. 允许说不知道

```
如果信息不足以做出判断，直接说「我不知道」或「需确认：…」。
不要为了显得能干而编造文件路径、API 行为或测试结果。
```

### 7. 把验证写进 prompt

没有 `<verify>` 的 prompt 没有终点：

```xml
<verify>
- [ ] pytest tests/ 全绿
- [ ] 无新增 lint 错误（ruff check src/）
- [ ] 未修改 git 未跟踪的 .env 文件
</verify>
```

验证必须是**第三方可观测的**——命令退出码、文件存在、输出匹配——不是「我觉得对了」。

---

## 场景模板

### 编码任务

```xml
<role>你是本仓库开发者，改动最小化，风格与现有代码一致</role>
<context>
技术栈、测试命令、必读文件路径
</context>
<task>一句话描述要交付的功能或修复</task>
<rules>
- MUST：改完跑测试
- MUST NOT：扩大 scope、加未请求依赖
</rules>
<verify>具体测试命令</verify>
```

### Code Review

```xml
<role>资深审查员，优先安全与正确性，其次可维护性</role>
<context>语言、框架、测试命令</context>
<task>审查下方 diff，只报可行动问题</task>
<rules>
- MUST：每条附 文件:行号
- MUST NOT：报纯风格偏好（除非项目有明确 linter 规则）
- 严重程度：Critical / Suggestion / Nit
</rules>
<input>
...diff...
</input>
<output_format>
## Critical
- [文件:行] 问题 — 建议
## Suggestion
...
</output_format>
```

### 给另一个 Agent 的指令（meta-prompt）

```xml
<task>把以下用户需求改写为可执行的指令</task>
<rules>
改写后的 prompt 必须包含：
1. 确切的最终状态（可观察）
2. 验证命令
3. 不能动的边界
</rules>
<input>用户的原始需求</input>
```

### 写 Skill（SKILL.md）

```xml
<task>为「部署到 staging」写一份操作 SOP</task>
<rules>
- frontmatter 含 name + description（第三人称 + 触发词）
- 正文 < 500 行，细节外置 reference.md
- 写 Workflow，不只写百科
- 每条步骤可执行，带具体命令
</rules>
<output_format>
---
name: ...
description: ...
---
# Title
## Workflow
...
</output_format>
```

---

## 审计清单

改写 prompt 前，逐项检查：

### 必要性（缺了必 ❌）

- [ ] `<task>` 能用一句话说清本轮交付物
- [ ] `<rules>` 有至少 1 条 MUST NOT
- [ ] `<verify>` 可被第三方检查（命令/文件/格式）
- [ ] 无互相矛盾的规则

### 清晰度（模糊标 ⚠️）

- [ ] 无「优化一下」「改好看点」「适当处理」类不可测词
- [ ] 文件路径、命令、版本号具体
- [ ] scope 有边界（改哪些、不改哪些）
- [ ] 输出格式有模板或 schema

### 格式敏感（需要时）

- [ ] 有 `<examples>`（≥2 个，含边界情况）
- [ ] JSON 输出附字段说明和必填项

---

## 反模式（见到就改）

| 反模式 | 为什么烂 | 修法 |
|--------|----------|------|
| 「你是世界上最厉害的 XX」 | 空洞角色，不激活具体能力 | 换成领域 + 受众 + 优先级 |
| 30 行背景，3 行任务 | 注意力被稀释 | 背景压缩到 5-10 行，任务前置 |
| 只有「要做什么」没有「不要做什么」 | scope 膨胀 | 加 MUST NOT |
| 「完成后告诉我」 | 不可验证 | 换成命令 + 退出码 |
| 规则堆叠无优先级 | 冲突时模型随机选 | 标 MUST > SHOULD > 可选 |
| 每轮贴全文对话 | token 爆炸 | 只摘要结论和未决项 |
| 一个 prompt 干 5 件事 | 失败难定位 | prompt chaining |
| 示例只有 happy path | 边界行为漂移 | 加拒绝/追问/降级示例 |
| 「尽量」「适当」「合理」 | 无法审计 | 换成数字、命令、具体文件名 |

---

## 质量评分（交付前自评）

| 分数 | 标准 |
|------|------|
| **A** | 零背景同事能执行；verify 可自动跑；规则无矛盾 |
| **B** | 任务清晰，verify 有但不够具体（「测试通过」没写命令） |
| **C** | 有意图但 scope/格式模糊，需追问才能动手 |
| **D** | 只有一句「帮我做 X」，等同没有 prompt |

**目标：交付的 prompt 至少 B，最好 A。**
