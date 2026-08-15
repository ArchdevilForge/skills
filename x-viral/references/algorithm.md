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
   - **New-author boost：曝光低于阈值的作者被抬升 —— 新号有扶持**
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

## 代码 → 策略映射（写推文时用）

| 算法事实 | 运营含义 |
|----------|----------|
| reply / dwell time / share 权重高 | 抛可争论观点引回复；写让人停留的内容，别让人秒划走 |
| 首小时互动速度决定分发 | 发帖 1 小时内必须回所有评论 |
| New-author boost | 冷启动有扶持期，前几周别弃更 |
| 多样性惩罚 + VMRanker | 刷屏同质内容会被压；一条爆款比十条平庸强 |
| 外链跳走 = 减少 dwell time | 链接放评论区，正文只留钩子 |
| phoenix 看你最近互动历史 | 账号定位窄而清晰，算法才知道把你推给谁 |
| simclusters 按互动聚类 | 你转评谁，你的帖子就进谁的候选池 —— 投喂算法靠互动网络，不是发帖量 |
| 长文有曝光加成 | 深度分析写长文，别硬塞进 280 字 |

## 参考

- https://github.com/twitter/the-algorithm（RETREIVAL_SIGNALS.md 有完整候选信号表）
- https://github.com/xai-org/x-algorithm（最新管道图 + 权重 + 误区澄清）
- https://github.com/twitter/the-algorithm-ml（heavy ranker 与 17 种互动预测）
