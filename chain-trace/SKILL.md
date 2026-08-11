---
name: chain-trace
description: Use when investigating a crypto token, contract, or wallet across ETH, Base, BSC, or Solana for suspicious holder clusters, funding paths, LP or permission risks, and related website or X/Twitter evidence using only public no-key sources (DefiLlama, Blockscout, GeckoTerminal, GoPlus, Jupiter, Solana/EVM RPC).
---

# Chain Trace - 公共接口版多链土狗深度取证

仅使用**公开接口（无需 API key）**，深度追踪 ETH / Base / BSC / Solana 链上资金流 + 链下网站与 Twitter(X) 全量关联线索。

## 触发条件

用户提到：
- 追踪钱包/地址/合约资金来源
- 土狗、庄家、控盘、砸盘、洗盘分析
- pump.fun / Solana meme / EVM（ETH/Base/BSC）土狗风险
- 项目官网、Twitter、团队背景深挖
- 想要链上 + 链下联合取证

---

## 核心原则（强制）

1. **公共接口优先**：禁止依赖任何需要 key 的服务。
2. **证据链优先**：每个结论必须绑定原始来源链接与时间戳。
3. **风险与置信度分离**：高风险不等于高置信；数据缺失必须降置信。
4. **覆盖最大化**：链上（代币/钱包/LP/权限/交易）+ 链下（网站/Twitter/历史快照/域名基础设施）都要做。
5. **无法验证 = Unknown**：不能因为缺数据就当低风险。
6. **官方源优先**：官网/docs/官方 X 的原始声明优先级高于 DEX、聚合器、媒体转载。
7. **身份与行情分离**：价格/流动性页面只能证明“在交易”，不能证明“官方身份”。
8. **名称/符号不可作为身份依据**：同名同符号可并存，身份判定必须基于地址与官方背书。
9. **完整拼接优先**：最终报告必须把“已获取的全部关键证据”拼接成可复核叙事，禁止只给薄摘要。
10. **证据编号强制**：所有关键结论必须引用 `EID`（Evidence ID），做到“结论→证据”一跳可追溯。
11. **双层输出强制**：先给执行摘要，再给完整版深度报告与附录；用户要求详细时不得降级为简版。

---

## Phase 0: 目标标准化 & 实体图谱

输入可能包含：代币地址、钱包地址、Twitter、网站 URL、项目名。

先构建实体图（Entity Graph）：
- 链上节点：`token`, `deployer`, `top_holders`, `lp_pairs`, `funding_wallets`
- 链下节点：`website domains`, `twitter handles`, `telegram`, `discord`, `github`
- 关联边：`claims/links/references/mentions`

输出基础结构：
```json
{
  "entities": [],
  "claims": [],
  "edges": [],
  "unknowns": []
}
```

---

## Phase 1: 链识别与地址合法性

识别规则：
- `0x` 开头 42 位 hex → EVM（ETH / Base / BSC）
- Base58 32-44 位 → Solana
- 用户明确指定链时，以用户指定为准
- 未指定链但为 EVM 地址时：优先在 `eth -> base -> bsc` 顺序探测，命中后锁定链

合法性检查：
- EVM（ETH/Base/BSC）：hex 长度 + 字符集
- Solana：Base58 字符集 `[1-9A-HJ-NP-Za-km-z]`

---


## Phase 2: 公共数据源（无 key）

→ 详见 [references/endpoints.md](references/endpoints.md)：全部端点速查（DefiLlama/Blockscout/BSCScan/GeckoTerminal/GoPlus/Jupiter/Solana RPC/EVM RPC）
→ 详见 [references/phase3-strategies.md](references/phase3-strategies.md)：Phase 3 深度追踪策略（资金溯源/地址聚类/控盘/合约权限/洗盘/老鼠仓）
