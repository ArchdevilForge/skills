# X 推荐算法要点（基于开源代码）

## 开源状态

| 仓库 | 内容 | 时间 |
|------|------|------|
| github.com/twitter/the-algorithm | 推荐算法主仓库（Scala），AGPL-3.0 | 2023-03 |
| github.com/twitter/the-algorithm-ml | heavy ranker 训练代码（MaskNet） | 2023-03 |
| github.com/xai-org/x-algorithm | 更新版，含权重参数 home-mixer/params/param.rs | 2026-01 |

**局限（别指望读代码拿流量密码）：**
- 是代码快照，不是生产实时同步；社区 PR 不会进生产
- 实时信号、完整模型参数、屏蔽/降权逻辑很多不在仓库
- 但结构完整：能看懂「算法怎么想」，权重是训练出来的

## For You 管道（每次请求实时组装）

1. **Candidate sourcing**（10 亿 → 数千，并行查询）
   - In-Network：`thunder` 内存中你关注账号的近期帖子
   - Out-of-Network：`phoenix` 向量检索（embed 你和帖子找最近邻）+ `simclusters` 社群聚类（谁和谁互动，就互相推荐）
2. **Candidate hydration**：补全元数据（文本/媒体/作者/引用/语言/互动/订阅状态）
3. **Pre-scoring filters**：预过滤
4. **Scoring**
   - Light ranker（Earlybird）：低延迟粗筛，~1000 候选 → ~100
   - Heavy ranker（Phoenix, MaskNet 多任务）：预测你对每个帖子做每种互动的概率
   - RankingScorer：概率加权和 → 最终分
5. **三个调整**
   - Author diversity：同一作者第二条起乘衰减系数（防刷屏）
   - Out-of-network discount：非关注账号的帖子打折；关注账号的回复和转推也打折
   - **New-author boost：公开快照中存在曝光阈值相关机制；是否适用于当前账号需用数据验证，不承诺新号扶持**
6. **VMRanker**：DPP 多样性重排（牺牲一点分数换相邻帖子不相似）
7. **Selection**：TopK 选出 ~50 条
8. **Visibility filtering**：黑名单/举报/敏感标签（排名和可见性是两套系统）

## 打分公式

```
Final Score = Σ(weight_i × P(action_i))
```

- 权重在 `home-mixer/params/param.rs`，随平台指标调整
- heavy ranker 预测 ~17 种行为：favorite / retweet / reply / **dwell time（停留时间）** / share / report / block / mute / unfollow…
- **关键误区**：权重乘的是「预测概率」，不是原始互动数。比如 report 权重是 like 的 468 倍 ≠ 1 个举报抵消 468 个赞 —— 它乘的是「你自己点举报的概率」。你的历史行为决定你自己的 P(action)

## 代码 → 策略映射（仅作实验假设）

下表来自公开代码快照，不是当前线上规则；把每条运营含义当作待验证假设，不当作流量保证。

| 代码信号 | 实验含义 |
|----------|----------|
| reply / dwell time / share 等行为 | 用可讨论观点和清晰结构测试回复、停留与分享；不诱导或刷互动 |
| 早期互动速度 | 作为首小时回复善意评论的实验变量，不代表决定分发 |
| New-author boost（代码快照） | 可测试冷启动窗口，但不假定当前账号一定获得扶持 |
| 多样性惩罚 + VMRanker（代码快照） | 避免同文刷屏；一条爆文是否优于多条普通帖要用数据验证 |
| 外链与停留可能相关 | 比较来源链接放正文、评论或 bio 的结果；不把外链必限流当规则 |
| phoenix 读取互动历史（代码快照） | 保持主题清晰，观察受众是否更匹配；不保证推荐给某类人 |
| simclusters 按互动聚类（代码快照） | 与相关创作者进行真实、有增量的互动；不要组织互刷网络 |
| 长文/Article 形态（代码快照） | 作为深度分析实验，不假定有额外曝光；不要硬塞进不适合的长度 |

## 参考

- https://github.com/twitter/the-algorithm（RETREIVAL_SIGNALS.md 有完整候选信号表）
- https://github.com/xai-org/x-algorithm（最新管道图 + 权重 + 误区澄清）
- https://github.com/twitter/the-algorithm-ml（heavy ranker 与 17 种互动预测）
