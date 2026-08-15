# 抓取 linux.do（及 Discourse 论坛）帖子讨论

## 坑（都试过，不行）

| 方法 | 结果 |
|------|------|
| `https://linux.do/t/<ID>.json` | Cloudflare 拦截（"Just a moment..."） |
| `https://linux.do/t/<ID>.rss`（不带 slug） | 返回「找不到页面」HTML（Discourse 需要完整路径） |
| `https://linux.do/search.json` | Cloudflare 拦截 |
| exa web_fetch / exa search | 超时或搜不到正文 |
| r.jina.ai 代理 | 免费层按 IP 封禁（AS36352） |
| web.archive.org | 无快照 |
| 浏览器直开 | 需过 Cloudflare 人机验证 |

## 可行方法：RSS + slug

linux.do 的帖子 URL 是 `https://linux.do/t/topic/<ID>`，**slug 固定是 `topic`**。RSS 端点必须带 slug：

```
curl -sL -A "Mozilla/5.0" "https://linux.do/t/topic/<ID>.rss"
```

**挑热门不挑最新**（用户教训：最新 ≠ 可发，热门才是）：

```
curl -sL -A "Mozilla/5.0" "https://linux.do/top.rss?period=weekly"   # 周榜
curl -sL -A "Mozilla/5.0" "https://linux.do/top.rss?period=daily"    # 日榜
curl -sL -A "Mozilla/5.0" "https://linux.do/latest.rss"              # 最新（仅作补充）
```

- `.rss` 端点不触发 Cloudflare，直接返回完整 XML（RSS 2.0，Discourse 原生支持）
- 每个 `<item>` 是一条回复/楼层，`<description>` 是正文 HTML（**含图片 URL、外链、引用内容**）
- 图片在 `cdn3.ldstatic.com/original/4X/...`，用 `original` 版本（`optimized` 是缩略图，注意会有带尾逗号的重复项，去重时小心）
- 主楼（[0]）通常是最长最全的——测试报告、结论文档都在里面

## 解析要点

```python
import re, html
s = open('r.xml', encoding='utf-8').read()
for it in re.findall(r'<item>(.*?)</item>', s, re.S):
    d = re.search(r'<description>(.*?)</description>', it, re.S)
    if not d: continue
    d = html.unescape(d.group(1))
    imgs  = re.findall(r'(https?://[^"\'\s<>]+\.(?:png|jpe?g|gif|webp)[^"\'\s<>]*)', d, re.I)
    links = re.findall(r'<a href="(https?://[^"]+)"', d)
    txt   = re.sub(r'\s+', ' ', re.sub(r'<[^>]+>', ' ', d)).strip()
```

- 楼层排序即讨论顺序；主楼 + 结尾「结论/汇总」楼是证据核心，中间的楼提取事实性发言
- 外链（GitHub 分析文档、benchmark 预览链接）优先于截图，能抓原文就抓原文
- 用户要求"有证据有图片"时：把关键图下载到本地交付（`curl -O` 原始图），文章里引用

## 通用化

任何 Discourse 论坛（`/t/<slug>/<id>` 路径 + `.rss` 后缀）大概率同法可读：
`<slug>` 缺失时先请求 `/t/<id>.rss`，若返回「找不到页面」HTML 而 RSS 没被 CDN 拦，再带 slug 试。
