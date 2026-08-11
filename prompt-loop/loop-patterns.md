# Loop Engineering 参考

> 你不再亲自给 agent 写 prompt。你设计一个系统，让系统去给 agent 写 prompt。
> — Boris Cherny

**前提：执行 loop 的 prompt 本身必须合格。** 先过 [prompt-reference.md 审计清单](prompt-reference.md#审计清单)，再读本文。

---

## Cursor 原语对照

| 需求 | 用什么 | 说明 |
|------|--------|------|
| 持续做到验证通过 | `/goal` + `cursor-goal` CLI | 见 `goal/SKILL.md` |
| 会话内周期执行 | `/loop` | 见 `loop/SKILL.md` |
| 人不在时定时跑 | 系统 cron / Cloud Agent / webhook + headless `agent` | 无 `/schedule` 命令 |
| 调项目知识 | `/skill-name` 或 skill 自动发现 | `.cursor/skills/<name>/SKILL.md` |
| 拦截/注入 | hooks | 见 `create-hook/SKILL.md` |

### `/goal` 关键设计

- 目标文本 = 起点 prompt + 完成标准
- 用 `cursor-goal checkpoint` 记录每轮，不依赖模型记忆
- 验证用独立命令（`--verify "npm test"`），不是 agent 自评
- 每轮结尾：`GOAL_STATUS: COMPLETE | CONTINUE | BLOCKED`

---

## 六要素

### 1. Automations（心跳）

什么触发下一轮？

- `/goal`：验证未通过 → 继续 checkpoint
- `/loop`：固定间隔或事件唤醒（`loop/SKILL.md`）
- cron：离线定时，适合日报、依赖审计

**优先调 skill，不要每轮贴 500 字 prompt。**

### 2. Worktrees（隔离）

多 agent 并行改同一 repo → 用 `git worktree` 各 checkout 一份。

### 3. Skills（项目知识）

把约定、构建步骤、验证命令写进 skill。Loop 调用 `/deploy` 而非重复贴规则。

### 4. Plugins / Connectors

MCP、GitHub、Linear 等——loop 的「手」伸向外部世界。

### 5. Sub-agents（制造者 ≠ 检查者）

| 角色 | 职责 |
|------|------|
| explorer | 只读搜索，快速建图 |
| builder | 写代码、改文件 |
| verifier | 跑测试、审查、对比 artifact |

**写代码的 agent 不应自己宣布完成。** 验证者可以是 shell 退出码、另一个 subagent、或 hook。

### 6. State / Memory

模型跨轮不记得。持久化到磁盘：

- `AGENTS.md` — 进度、失败、下一步
- `.goal/` — cursor-goal 状态
- Linear/Jira — 通过 MCP

---

## Hooks（闭环关键）

Cursor 事件名（来自 `create-hook/SKILL.md`）：

| 事件 | 场景 |
|------|------|
| `stop` | agent 想结束 → 检查测试是否通过，不过则注入失败日志并阻止 |
| `postToolUseFailure` | 工具失败 → 重试或注入诊断 |
| `preToolUse` | 拦截危险工具调用 |
| `afterFileEdit` | 改完自动 format/lint |
| `subagentStop` | 子 agent 完成 → 触发 pipeline 下一步 |
| `beforeShellExecution` | 拦截危险 shell 命令 |

**最高杠杆：`stop` hook + 验证命令退出码。**

---

## 四种模式

### 模式 1：构建-测试-修复（build-test-fix）

```
/goal
1. Builder 按 spec 实现
2. Verifier 跑测试 + lint（独立 shell）
3. 失败 → Builder 修 → Verifier 再验
4. 最多 N 轮
```

### 模式 2：验证器循环（verifier loop）

```
执行任务 → 独立 verifier 检查 → 通过则下一项
不通过 → 打回重做
同 root cause 连续 2 次失败 → 升级人类
```

### 模式 3：规划-实现-验证-修复

```
/goal
规划 → 实现 → 验证 → 修复
每轮状态写文件；第一轮全过立即停止
```

### 模式 4：Goal-meta（最高杠杆）

动手前先把用户需求重写为严格 goal：

```
- 确切最终状态
- 验证命令
- 不能动的边界
- 停止条件
确认后再执行。
```

---

## 写 Loop Prompt 的五条

（Loop 里的 prompt 同样遵循 [prompt-reference.md](prompt-reference.md)）

1. **描述结果，不是步骤** — 让 agent 自己找路径
2. **包含验证命令** — `npm test`、`pytest`、`tsc --noEmit`
3. **指向参考文件** — 现有模式、规范、skill
4. **可衡量目标** — `bundle < 200KB`、`覆盖率不降`
5. **给 artifact** — 错误日志、diff、CI 输出供迭代

### Loop Prompt 骨架

```
<task>达成 [可观察的最终状态]</task>
<verify>[独立命令]，退出码 0 才算完成</verify>
<rules>
- 每轮一个 bounded checkpoint
- 验证失败：分析 → 修复 → 再验证
- 最多 N 轮；同 root cause 失败 2 次 → BLOCKED
</rules>
```

### 写入项目规则（`.cursor/rules/`）

```markdown
## Loop Behavior

- 修改 src/ 后必须跑 `npm test`；失败则修到绿
- 有 TypeScript 错误不算完成：先 `tsc --noEmit`
- 完成标准：测试绿 + lint 净 + PR 描述已写
```

---

## 何时用 Loop vs 单次 Prompt

| 维度 | 单次 prompt | Loop |
|------|-------------|------|
| 任务形状 | 可预测、一轮够 | 需试错+重验证 |
| 时长 | 你全程在场 | 你不在也要跑 |
| 风险 | 可逆、本地 | 共享文件、CI、生产 |
| 成本 | token 一次性 | 重试复合膨胀 |
| 团队 | 个人快速答案 | 可复现自动化 |

---

## 两个常见失败模式

1. **无验证器的开放循环**
   - 症状：agent 删失败测试然后宣布完成
   - 解法：独立验证器（shell 退出码 / verifier subagent / stop hook）

2. **无限重试同一路**
   - 症状：换措辞重试同一死路
   - 解法：迭代上限 + 同 root cause 2 次 → BLOCKED → 升级人类

---

## 相关 Skill

- `goal/SKILL.md` — `/goal` 实操
- `loop/SKILL.md` — `/loop` 实操
- `create-hook/SKILL.md` — hook 编写
- `create-skill/SKILL.md` — 把项目知识封装成 skill
