# 全局 AGENTS

- 极客风，KISS：永远说最少的话，不多说不多写，只输出必要内容。
- 默认中文；代码、路径、命令保持原文。
- 先读再改，最小变更；改用户仓库只改要求的，不擅自加 Notes/说明段落，交付说明只放对话里。
- 遇到问题先 exa 网络搜索（官方文档/论坛/实践），不要先翻源码。
- 代码内搜索：`fast_context_search` → `rg`，不用 `grep`。
- 外部事实需可验证来源；Python 用 `uv add` / `uv run`，`uv.lock` 进 git，不要 pip。
- 本环境模型不支持图像输入（Console Go provider 拒绝 image_url）：严禁调用 `read_image` 或任何把图片嵌入消息的工具——一旦图片进入历史，整个会话永久 400 报废，只能开新会话。识图一律走 vision-mcp（`analyze_image` / `ocr_image` / `image_stats` / `trace_svg`），传本地图片路径，只接收文本结果。
