# Phase 3 深度追踪策略（资金/聚类/控盘/洗盘/老鼠仓）

（从主文件移出，按需加载）

## Phase 3: 链上深度追踪策略（资金、控盘、操纵）

### 3.1 资金来源追踪（Wallet Funding Trace）

1. 抽取目标地址最近 N 条交易（Solana: `getSignaturesForAddress` + `getTransaction`；EVM: `eth_getLogs` / `eth_getTransactionByHash`）。
2. EVM（ETH/Base/BSC）对代币资金流使用 `eth_getLogs(Transfer)` 追踪输入输出地址。
3. 标记来源类型：`CEX热钱包 / 桥接地址 / 混币 / 新钱包簇 / 未知`。
4. 记录每个地址的 `first_funder`、`first_fund_time`、`first_buy_time` 作为后续自动关联输入特征。

### 3.2 地址聚类（Address Clustering）

聚类依据：
- 同时段同金额模式
- 共享上游资金源
- 高频互转环
- 首次入金时间高度一致

#### 3.2.1 自动发现链上关联地址（必须执行）

构建地址关系图 `G=(V,E)`：
- `V`：wallet / token account / lp pair / sink wallet
- `E`：`funding_edge` / `trade_edge` / `transfer_edge` / `exit_edge`

对每个候选地址抽取特征：
- `first_funder`
- `delta_t_first_buy`（相对项目启动/首条官方宣发）
- `first_buy_amount`
- `buy_window_pattern`（短窗买入节奏）
- `sell_window_pattern`（短窗卖出节奏）
- `profit_sink`

关系分（0~1）建议：
```text
relation_score =
  0.30 * co_funder
  + 0.20 * co_time
  + 0.15 * co_amount
  + 0.20 * co_exit
  + 0.15 * shared_sink
```

阈值：
- `>= 0.75`：强关联地址簇（High-confidence linked cluster）
- `0.55 ~ 0.75`：疑似关联簇（Suspected cluster）
- `< 0.55`：弱关联，仅保留证据不下结论

#### 3.2.2 降误报约束（必须）

自动排除：
- CEX 热钱包、官方路由、桥接合约、LP 池地址、dead 地址

高置信结论至少满足：
- 3 条独立信号（如 `co_funder + co_time + shared_sink`）
- 且不存在强反例（例如公开做市地址、批量空投地址）

### 3.3 控盘与流动性

必须输出：
- Top 持仓集中度（排除 LP/Dead/已知路由）
- LP 深度（USD）与 LP/FDV 比
- 24h 交易量/流动性比

建议阈值（可按链微调）：
- Top10(净) > 50%：高控盘
- LP < $20K：高滑点高风险
- 24h Vol / LP > 8：疑似刷量或高波动操纵

### 3.4 合约与权限风险

必须检查：
- EVM（ETH/Base/BSC）: 可升级/owner 权限/黑名单/税率可改
- Solana: mint authority / freeze authority / metadata 可改性
- 是否存在 honeypot 税收异常与卖出限制

### 3.5 洗盘与做市伪装信号

重点信号：
- 高频小额重复买入（micro-buys）
- 同时间窗口对敲
- 新钱包占成交额异常高
- 价格拉升但净买家不增长

### 3.6 老鼠仓与捆绑地址自动识别（新增，必须执行）

#### A) 老鼠仓（Insider）识别信号

1. **宣发前吸筹**：在项目首次官方宣发前（推特/官网）5 分钟~24 小时出现集中买入。
2. **先手集中度**：早期地址簇累计持仓占比异常（例如净 Top 持仓中占比显著偏高）。
3. **同步出货**：宣发后短时间内（如 10 分钟）多个关联地址同步卖出。
4. **同源资金闭环**：多地址来自同一 `first_funder`，并最终回流到同一 `sink`。

#### B) 捆绑地址（Bundle/Coordinated）识别信号

1. **co-time**：同一小时间窗内重复共同行为（买入/卖出/转移）。
2. **co-amount**：交易金额分布高度相似（固定档位、同倍率）。
3. **co-funder**：首笔资金来源一致或高度重叠。
4. **co-trade**：交易路径一致（同 pair、同路由、同方向）。
5. **co-exit**：利润提现目标地址一致。

#### C) 自动评分与结论

```text
insider_score =
  0.25 * pre_pump_accumulation
  + 0.20 * early_cluster_share
  + 0.20 * synchronized_exit
  + 0.20 * shared_funder
  + 0.15 * shared_sink
```

解释阈值（建议，可按市场调参）：
- `insider_score >= 0.70`：高概率老鼠仓
- `0.50 ~ 0.70`：疑似老鼠仓
- `< 0.50`：证据不足

> 注意：老鼠仓/捆绑地址是概率推断，不是司法定性；必须附证据链和反例评估。

#### D) 信号分层：确定性 vs 启发式（必须区分）

**确定性信号（Deterministic，权重更高）**：
- 同一 `first_funder` 且首笔入金交易哈希可追溯
- 同一 `profit_sink`（最终提现目标地址一致）
- 同一交易/同一区块内明确共现（可由链上原始交易证明）

**启发式信号（Heuristic，需多项叠加）**：
- co-time（时间窗口共振）
- co-amount（金额分布相似）
- co-trade（路径/方向一致）
- pre-pump accumulation（宣发前吸筹）

结论约束：
- 仅启发式信号，不得直接给“高置信”结论
- 至少 1 条确定性 + 2 条启发式，才可标记为“高置信关联”

#### E) 老鼠仓/捆绑地址误报控制（必须输出）

每个高风险簇都要输出“替代解释”至少 1 条，例如：
- 做市/套利机器人常规路径
- 空投批量分发地址
- CEX 归集地址正常调仓

并记录：
- `deterministic_signals_count`
- `heuristic_signals_count`
- `false_positive_alternatives`

---

## Phase 4: 网站深度情报（Website Intelligence）

> 目标：不是只看首页，而是抽取“全站可验证证据”。

### 4.1 全站抓取入口
```bash
curl -sL "{website}"
curl -sL "{website}/robots.txt"
curl -sL "{website}/sitemap.xml"
curl -sL "{website}/.well-known/security.txt"
curl -sL "{website}/manifest.json"
```

### 4.2 域名与基础设施

```bash
# RDAP 域名年龄/注册信息
curl -sL "https://rdap.org/domain/{domain}"

# DNS over HTTPS
curl -s "https://cloudflare-dns.com/dns-query?name={domain}&type=A" -H "accept: application/dns-json"
curl -s "https://cloudflare-dns.com/dns-query?name={domain}&type=NS" -H "accept: application/dns-json"
curl -s "https://cloudflare-dns.com/dns-query?name={domain}&type=TXT" -H "accept: application/dns-json"

# 历史快照密度（Wayback）
curl -s "https://web.archive.org/cdx/search/cdx?url={domain}/*&output=json&limit=200"

# 证书透明日志（子域名爆破线索）
curl -s "https://crt.sh/?q=%25.{domain}&output=json"

# urlscan 历史公开扫描
curl -s "https://urlscan.io/api/v1/search/?q=domain:{domain}"
```

### 4.3 站内结构化提取

从 HTML/JS 中提取：
- 钱包地址（EVM / Solana）
- 社交链接（X、Telegram、Discord、GitHub）
- 团队身份字段（姓名、职位、邮箱、领英）
- 风险词（guaranteed/100x/risk-free 等）
- 合约地址与白皮书地址是否一致

### 4.4 网站红旗

- 域名年龄极短（如 < 90 天）
- 网站结构极薄（仅单页营销，无文档）
- 联系方式缺失或全部跳转匿名渠道
- 多域名共用同模板且项目叙事不一致

---

## Phase 5: Twitter(X) 深度情报（无 key）

### 5.1 数据来源（公开）

1) **x-tweet-fetcher（推荐，零 key）**
```bash
# 单条推文
X_FETCHER="$HOME/.agents/skills/x-tweet-fetcher/scripts/fetch_twitter.py"
[ -f "$X_FETCHER" ] || X_FETCHER="$HOME/.claude/skills/x-tweet-fetcher/scripts/fetch_twitter.py"
python3 "$X_FETCHER" --url "https://x.com/{user}/status/{id}" --text-only

# 用户时间线（需 Camofox）
python3 "$X_FETCHER" --user {username} --limit 200 --text-only
```

2) **X Syndication 公开端点（单条推文 JSON）**
```bash
curl -s "https://cdn.syndication.twimg.com/tweet-result?id={tweet_id}&token=1"
```

3) **oEmbed（公开）**
```bash
curl -s "https://publish.twitter.com/oembed?url=https://x.com/{user}/status/{tweet_id}"
```

4) **Nitter RSS（公开镜像，作为备份）**
```bash
curl -s -A "Mozilla/5.0" -L "https://nitter.net/{username}/rss"
```

5) **搜索引擎反查（发现更多相关推文/账号）**
```bash
# X 站内公开索引反查
site:x.com "{project_name}"
site:x.com "{token_address}"
site:x.com "{domain}"
site:x.com/@{username}
```

> 说明：Syndication 常用于“已知 tweet_id 的深挖”；未知 ID 的发现靠搜索引擎 + 时间线抓取。

### 5.2 深挖范围（必须覆盖）

- 官方账号最近 N 条推文（内容、频率、互动、链接）
- 关联账号（被 @、经常互转、固定评论群）
- 从推文提取所有 URL、钱包地址、邀请码
- 提取所有外链并做 URL 展开（避免短链跳转伪装）
- 与网站披露信息做一致性比对（团队、路线图、合约地址）
- 项目上线窗口前后发文节奏变化
- 收集“谁在扩散”：高频转发账号、早期喊单账号、疑似机器人簇

### 5.3 Twitter 红旗

- 新号突然高频营销（且技术内容稀薄）
- 互动异常（低质量账号集中点赞/转发）
- 官网地址与推特资料链接不一致
- 推文里合约地址与链上真实地址不一致
- 关键推文被删除/频繁改名但历史叙事对不上
- 多账号共享同一外链落地页或同一钱包收款地址

### 5.4 JS-only 页面处理（必须）

当 `webfetch/curl` 返回 `Enable JavaScript` / `JavaScript is not available`：

1. **X 页面**：优先 `x-tweet-fetcher`，其次 `cdn.syndication.twimg.com` + `oEmbed`。
2. **Launchpad 页面**：优先平台公开 API/缓存快照/搜索引擎收录片段。
3. **仍无法获得关键字段**：将该字段标记为 `Unknown (JS-gated)`，并降低相关结论置信度。

禁止：在 JS 页面缺字段时自行补全“猜测地址/状态”。

### 5.5 时间线重建协议（新增，必须）

目标：确保时间线“可复核、可解释、可追因果”，而不是仅列零散事件。

1) **时间标准化**
- 全部时间统一为 `UTC`；若来源为本地时区，必须同时给原始时区与转换规则。
- 链上事件必须附 `block/slot + tx hash` 至少一项作为锚点。

2) **事件粒度标准**
- 标准模式：时间线事件数 `>= 8`
- 深挖模式（`--deep`）：时间线事件数 `>= 15`
- 每个事件至少绑定 1 条 `EID`，关键事件（宣发、部署、首池、大额转移、首轮分发、异动）必须绑定 >=2 条独立 EID。

3) **事件类型覆盖（至少覆盖 6 类，深挖建议全覆盖）**
- 官宣/预热（官网/推特/公告）
- 合约/代币部署
- 流动性建立/迁移
- 首轮大额分发或可疑聚集转移
- 价格/成交量结构性异动
- 权限变更（owner/mint/freeze/upgrade）
- 关键辟谣/澄清/删帖
- 资金回流/套现/跨链出逃

4) **时间线清晰度要求**
- 必须给出“相对时间差” (`Δt`)：当前事件相对上一关键事件的间隔。
- 必须标注“因果方向假设”：`宣发→交易` / `交易先行` / `证据不足`。
- 必须给出“拐点列表”（Turning Points）：至少 3 个关键拐点及其证据。

5) **时间线冲突处理**
- 若出现事件先后矛盾（例如不同源时间冲突），必须记录到 Contradiction Log 并降低置信度。
- 未解析时间冲突不得给“高置信因果结论”。

禁止：
- 只给单行时间线（例如“某日上线”）而无锚点/证据。
- 用“约在某时”替代可提取的精确时间。

---

## Phase 6: 链上 + 链下交叉验证（关键）

### 6.1 官方 CA 判定协议（新增，必须执行）

先构建 `canonical_ca_matrix`：

| 证据级别 | 来源类型 | 用途 | 单独可判官方吗 |
|---|---|---|---|
| P0 | 官网/docs/官方 X 原文声明 | 身份主证据 | 否 |
| P1 | 官方已知地址链上背书（持有/签名/绑定） | 链上佐证 | 否 |
| P2 | DEX/聚合器/媒体/第三方榜单 | 发现线索 | 否 |

判定规则：
- `Official`：至少 `1条P0 + 1条P1` 且无强冲突。
- `Unverified`：仅有 P2，或 P0/P1 任一缺失。
- `Contradicted`：官方源明确否认（如“未发币/无CA”）但外部存在候选地址。

硬约束：
- `symbol/name` 命中、行情存在、池子存在、launchpad 展示，均不能直接判定官方。
- `poolAddress != mintAddress`，不得把池子地址当 token mint。
- 若官方文档声明“尚未发布 CA”，所有外部候选默认 `Unverified` 或 `Contradicted`。

执行 2-of-3 身份一致性规则：
1. 合约/项目地址在官网出现
2. 合约/项目地址在 Twitter 官方账号出现
3. 官网与 Twitter 相互反向链接

> 注意：2-of-3 仅是身份一致性检查，不可替代上面的官方 CA 判定协议。

若 2-of-3 不成立：
- 风险等级至少上调一级
- 置信度显著下调

并追加“地址关联交叉验证”：
- 关联簇内地址是否在官网/推特/公告中出现
- 宣发时间与关联簇交易行为是否存在先后因果
- 若出现链下身份矛盾（自称官方却地址不一致），强制上调风险

---

## Phase 7: 风险评分 + 置信度评分

### 7.1 风险分（Risk, 0-100）

推荐维度：
- 链上资金与控盘：30%
- 地址关联/老鼠仓/捆绑检测：25%
- 流动性与交易微结构：20%
- 合约权限与安全：15%
- 链下身份与舆情：10%

### 7.2 置信度分（Confidence, 0-100）

```text
Confidence = 100 * (0.4 * coverage + 0.4 * agreement + 0.2 * freshness)
```

地址关联子置信度（用于簇级别）：
```text
link_confidence =
  100 * (0.5 * deterministic_strength + 0.3 * cross_source_agreement + 0.2 * temporal_stability)
```

- `coverage`：关键字段覆盖率（地址/流动性/权限/社交）
- `agreement`：跨来源一致性（同字段多源比对）
- `freshness`：数据时间新鲜度

### 7.3 评分解释要求

每个高风险结论都要附：
- 证据来源 URL
- 抓取时间
- 原始片段（可复核）
- 关联簇 ID（cluster_id）与关键关系边（edge list）
- 是否存在替代解释（false-positive alternative）

### 7.4 可执行评分模板（无 key，本地计算）

> 目的：把 relation/insider/link 评分从“文字规则”落成可复算结果，便于复核与回归比较。

1) 使用现成模板：`templates/score_input.example.json`

2) 直接运行评分脚本：
```bash
python3 scripts/score_models.py --input templates/score_input.example.json
```

3) 若要自定义样例，可参考以下字段结构：
```json
{
  "co_funder": 0.9,
  "co_time": 0.8,
  "co_amount": 0.6,
  "co_exit": 0.7,
  "shared_sink": 0.8,

  "pre_pump_accumulation": 0.9,
  "early_cluster_share": 0.7,
  "synchronized_exit": 0.6,
  "shared_funder": 0.9,
  "shared_sink_insider": 0.8,

  "deterministic_strength": 0.8,
  "cross_source_agreement": 0.7,
  "temporal_stability": 0.9
}
```

4) 输出分级建议：
- `relation_score >= 0.75` → 强关联簇
- `insider_score >= 0.70` → 高概率老鼠仓
- `link_confidence >= 75` 且满足“1确定性 + 2启发式” → 高置信结论

### 7.5 阈值校准（Calibration）流程（必须）

为减少误报，阈值必须按链与流动性分层：

1. 数据分桶：
- 链：`ETH` / `Base` / `BSC` / `Solana`
- 流动性：`LP < 20K`、`20K~100K`、`>100K`

2. 基准集：
- `Positive set`：历史高置信恶意样本
- `Negative set`：已知做市/套利/正常项目样本

3. 校准目标：
- 保持召回率前提下压低误报（尤其 bundle/insider）
- 记录每桶最优阈值：`relation_t`, `insider_t`, `link_conf_t`

4. 每次更新必须记录：
- 样本窗口时间、样本量、链别
- 调整前后误报率/漏报率
- 主要误报类型（做市、CEX归集、空投分发）

5. 报告输出时必须声明：
- 使用的是“默认阈值”还是“校准阈值”
- 若为默认阈值，置信度最多标记到 `Medium-High`，不得标 `Very High`

6. 本地阈值校准脚本：
```bash
python3 scripts/calibrate_thresholds.py --input templates/calibration_dataset.example.json
```

输出包含：
- `buckets.{chain:lp_bucket}.thresholds.{relation_t, insider_t, link_conf_t}`
- `buckets.{chain:lp_bucket}.metrics.{fpr, fnr, precision, recall}`
- `sample_size`

### 7.6 反例库（False-Positive Library）最小模板

每次判定高风险簇时，至少对以下 3 类反例做排除：

1. **做市/套利路径**：
- 同一策略地址跨多个无关代币重复出现
- 入场/出场围绕价差而非叙事窗口

2. **CEX 归集/调仓**：
- 地址命中已知 CEX 热钱包或归集模式
- 资金流方向以归集为主，不呈现项目特异性

3. **空投/批处理分发**：
- 大量小额同模转账，但后续行为不协同拉盘/出货
- 缺失共享 profit sink

若反例解释成立，必须：
- 下调 `insider_score` / `relation_score`
- 将最终结论降级为 `Suspected` 或 `Unknown`

### 7.7 矛盾证据日志（Contradiction Log，必须）

每个报告必须包含最少一张矛盾表：

| claim_id | 声明内容 | 来源 | 时间 | 证据级别(P0/P1/P2) | 状态(Confirmed/Unverified/Contradicted) | 备注 |
|---|---|---|---|---|---|---|

规则：
- 出现 `Contradicted` 时，最终结论不能标“高置信官方”。
- 若关键 claim 状态为 `Unverified`，风险结论可高，但置信度必须下调。

### 7.8 报告完整性门禁（Completeness Gate，必须）

在进入最终输出前，必须满足以下门禁；任一未满足则判定 `NOT COMPLETE`，继续补证：

1. **证据覆盖门禁**
- 关键域覆盖：`官方身份`、`候选地址/CA`、`链上交易`、`流动性/持仓`、`网站`、`X/Twitter`、`矛盾证据`。
- 每个关键域至少 1 条可复核证据；缺失项必须显式标注 `Unknown`。

2. **证据可追溯门禁**
- 每条关键证据必须有 `EID`、URL、抓取时间、原始片段（quote/snippet）。
- 每条核心结论至少绑定 2 条独立证据（或声明“单证据结论并降置信”）。

3. **叙事连贯门禁**
- 必须提供“时间线”（Timeline）把宣发、上链、交易、流动性变化串起来。
- 必须提供“声明矩阵”（claim-by-claim）显示 `Confirmed / Unverified / Contradicted`。
- 时间线事件数：标准模式 `>= 8`；深挖模式 `>= 15`。
- 时间线必须包含 `Δt` 与“拐点列表（>=3）”。

4. **身份判定门禁**
- 必须分离回答两个问题：
  - Q1: 项目/平台合作是否成立？
  - Q2: 候选 CA 是否 canonical 官方地址？
- 禁止把 Q1 的证据直接复用于 Q2 结论。

5. **估值透明门禁**
- 必须列出估值输入（供应、硬顶、价格、MC/FDV锚点）与公式。
- 必须给 `Bear/Base/Bull + EV`，并显式区分“条件估值”和“风险折价估值”。

6. **深挖深度门禁（新增）**
- 链上交易：标准模式解码 `>= 30` 笔；深挖模式 `>= 80` 笔。
- 关键地址：至少输出 `deployer + fee path + top holders + authority path`。
- 社媒侧：标准模式分析官推 `>= 20` 条；深挖模式 `>= 50` 条。
- 站点侧：至少覆盖 `首页 + docs + program/faq + launchpad/镜像`（若存在）。

7. **薄报告拦截门禁（Hard Fail）**
- 若输出缺少任一核心章节（证据清单/时间线/矛盾日志/CA矩阵/估值表），视为失败。
- 用户明确要求“详细报告”时，禁止只返回 3-5 条 bullets。
- 用户明确要求“深挖”时，禁止省略“时间线拐点分析 + 深挖附录”。

---

## Phase 8: 输出报告（完整版，默认）

```markdown
## Token Forensics Report: {TokenName} [{Chain}]

### A. Executive Summary
- Risk Score: {0-100}
- Confidence Score: {0-100}
- Verdict: Low / Medium / High / Critical
- Why: 3-5 条最关键证据
- Canonical CA Status: Official / Unverified / Contradicted

### B. Scope & Method
- 目标问题：{本次要回答的核心问题}
- 调查范围：{链上/链下/时间窗/样本限制}
- 数据来源层级：P0/P1/P2
- 采集时间窗：{start} ~ {end}
- 关键限制：{rate-limit / JS-gated / 无索引器}

### C. Entity Graph Snapshot
- On-chain entities: [{entity_id, type, address, role}]
- Off-chain entities: [{entity_id, type, handle/domain, role}]
- Key edges: [{from, to, edge_type, evidence_eid}]

### D. Evidence Inventory (EID)
| EID | 类型 | 来源URL | 抓取时间 | 关键片段 | 关联结论 |
|---|---|---|---|---|---|
| E001 | 官方文档 | ... | ... | "..." | Q2 |

### E. Chronology Timeline
| 序号 | 时间(UTC) | 链上锚点(block/slot/tx) | Δt(相对上一事件) | 事件类型 | 证据EID | 影响 |
|---|---|---|---|---|---|---|
| T01 | ... | ... | - | 官宣/部署/流动性/分发/异动/权限 | E00x | ... |

- 事件数量要求：标准模式 `>= 8`；深挖模式 `>= 15`
- 覆盖要求：至少覆盖宣发、部署、流动性、分发/交易异动四类
- 每个关键事件（部署、首池、首轮分发、大额转移、官方澄清）需 >=2 条独立证据

### E2. Timeline Turning Points（必须）
| 拐点ID | 时间(UTC) | 触发事件 | 因果方向假设 | 证据EID | 置信度影响 |
|---|---|---|---|---|---|
| TP1 | ... | ... | 宣发→交易 / 交易先行 / 证据不足 | E00x | + / - |

- 至少提供 `>= 3` 个拐点
- 若存在时间冲突，必须在 H 或 L 节显式标注并下调置信度

### F. On-Chain Evidence
- Contract/Mint: ...
- Deployer/Funding Path: ...
- Top Holders (net of LP/dead): ...
- Liquidity & Volume microstructure: ...
- Authority / Owner / Tax / Honeypot checks: ...
- Auto-discovered linked clusters: {cluster_count}, strongest cluster: {cluster_id}
- Suspected insider wallets: [{addr, insider_score, key_signals}]
- Suspected bundle clusters: [{cluster_id, relation_score, members}]
- Signal tiers per cluster: {deterministic_signals_count, heuristic_signals_count}
- False-positive alternatives per cluster: [...]

### G. Off-Chain Evidence
- Website integrity: domain age, historical snapshots, infra fingerprints
- Website extracted entities: wallets, socials, team links, docs
- Twitter/X graph: official + related accounts + content themes + suspicious patterns

### H. Claim Resolution Matrix
| claim_id | 声明内容 | 证据级别 | 状态 | 证据EID | 备注 |
|---|---|---|---|---|---|
| C01 | ... | P0/P1/P2 | Confirmed/Unverified/Contradicted | E00x | ... |

### I. Cross-Validation
- Canonical CA Matrix: {P0_count, P1_count, P2_count, status}
- Q1（合作关系）: PASS / FAIL + evidence
- Q2（canonical CA）: PASS / FAIL + evidence
- 2-of-3 identity linkage: PASS / FAIL
- Address consistency across chain/website/twitter: PASS / FAIL
- Promotion-before-trade causality check: PASS / FAIL
- Insider/bundle evidence consistency: PASS / FAIL

### J. Scenario Valuation
- 输入假设：{supply, hardcap, launch MC, breakeven MC, SOL/USD, liquidity}
- 公式：{FDV/MC/EV 计算公式}
- 情景：Bear / Base / Bull（含概率）
- 输出：{每情景市值区间 + EV}
- 风险折价：{身份不确定性折扣如何施加}

### K. Red Flags Checklist
- [ ] High holder concentration
- [ ] Weak liquidity
- [ ] Mutable dangerous permissions
- [ ] Wash-trading signatures
- [ ] 疑似老鼠仓（宣发前吸筹 + 同步出货）
- [ ] 疑似捆绑地址簇（co-funder/co-time/co-exit）
- [ ] Website/Twitter identity mismatch
- [ ] No verifiable team or history

### L. Data Quality & Unknowns
- Missing critical fields:
- Source conflicts:
- Rate-limit impact:
- JS-gated fields impact:

### M. Final Action Guidance
- 建议：参与 / 观望 / 回避
- 若参与：最大仓位建议、止损条件、监控触发器

### N. Appendix (Raw Extracts)
- 原始推文摘录：
- 原始链上交易摘录：
- 原始网页/文档摘录：
- 复现实验命令：

### O. Deep-Dive Annex（深挖附录，用户要求深挖时必填）
- 地址画像卡（Address Cards）：`deployer / treasury / fee receiver / suspected cluster leaders`
- 关键交易解码明细：标准模式 `>= 10` 条；深挖模式 `>= 25` 条
- 资金路径分层图：`source -> routing -> sink`
- 争议点复盘：每个争议点给“支持证据 / 反证 / 当前判定”
- 未决问题清单：`Unknown` 项目 + 下一步补证命令
```

> 输出约束（强制）：
> 1) 先输出 6-12 行执行摘要；
> 2) 紧接输出完整版 A~N 章节；
> 3) 用户要求“详细/完整”时，禁止省略 D/E/E2/H/J/N；
> 4) 用户要求“深挖”时，必须追加 O 章节，并满足深挖门禁。

---

## 常用命令（公共接口模式）

```bash
# 快速分析
/chain-trace {token_or_wallet}

# 深度链上+链下
/chain-trace {token_or_wallet} {website_url} {twitter_url}

# 强制深挖模式（更广更深，耗时更长）
/chain-trace {target} --deep --all-sources --evidence-graph

# 自动发现关联地址 + 老鼠仓/捆绑检测
/chain-trace {target} --deep --auto-link --insider --bundle --evidence-graph

# 使用 uv + cloudscraper 探测公开 RPC 池（CF/403/429 场景）
uv run python scripts/rpc_probe_cloudscraper.py --chain solana --tries 2
uv run python scripts/rpc_probe_cloudscraper.py --chain eth --tries 2
uv run python scripts/rpc_probe_cloudscraper.py --chain base --tries 2
uv run python scripts/rpc_probe_cloudscraper.py --chain bsc --tries 2

# 本地评分（公式复算）
python3 scripts/score_models.py --input templates/score_input.example.json

# 本地阈值校准（分桶输出）
python3 scripts/calibrate_thresholds.py --input templates/calibration_dataset.example.json
```

---

## 已知地址库

### Ethereum
```
Core assets:
- 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2 (WETH)
- 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48 (USDC)

Routers / Infra:
- 0x7a250d5630B4cF539739dF2C5dAcb4c659F2488D (Uniswap V2 Router)
- 0xE592427A0AEce92De3Edee1F18E0157C05861564 (Uniswap V3 SwapRouter)

Dead:
- 0x000000000000000000000000000000000000dEaD
- 0x0000000000000000000000000000000000000000
```

### Base
```
Core assets:
- 0x4200000000000000000000000000000000000006 (WETH on Base)
- 0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913 (USDC on Base)

Dead:
- 0x000000000000000000000000000000000000dEaD
- 0x0000000000000000000000000000000000000000
```

### BSC
```
Binance Hot Wallets:
- 0xeb2d2f1b8c558a40207669291fda468e50c8a0bb
- 0x8894e0a0c962cb723c1976a4421c95949be2d4e3
- 0xe2fc31f816a9b94326492132018c3aecc4a93ae1

PancakeSwap Routers:
- 0x10ED43C718714eb63d5aA57B78B54704E256024E (V2)
- 0x13f4EA83D0bd40E75C8222255bc855a974568Dd4 (V3)

Dead:
- 0x000000000000000000000000000000000000dEaD
- 0x0000000000000000000000000000000000000000
```

### Solana
```
System:
- 11111111111111111111111111111111
- TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA

DEX / Infra:
- 675kPX9MHTjS2zt1qfr1NYHuzeLXfQM9H24wFSUt1Mp8 (Raydium AMM V4)
- 5Q544fKrFoe6tsEbD7S8EmxGTJYAKtTVhAW5Q5pge4j1 (Raydium Authority V4)
- JUP6LkbZbjS1jKKwapdHNy74zcZ3tLUZoi5QNyVTaV4 (Jupiter V6)

pump.fun Program:
- 6EF8rrecthR5Dkzon8Nwu78hRvfCKubJ14M5uBEwF6P

Burn:
- 1nc1nerator11111111111111111111111111111111
```

---

## 注意事项

- 本版只依赖公开接口，**不需要配置任何 API key**。
- 公共接口会限流：必须实现重试、退避、降级，并在报告里反映置信度下降。
- ETH/Base/BSC/Solana 在无索引器条件下，部分“完整历史归因”不可 100% 还原，必须标记 `Unknown`。
- 同名同符号资产可并存：**不得用 ticker/symbol 作为官方身份判据**。
- DEX/聚合器/媒体页面用于发现线索，不是官方背书来源。
- 本分析不构成投资建议，仅用于取证与风控参考。