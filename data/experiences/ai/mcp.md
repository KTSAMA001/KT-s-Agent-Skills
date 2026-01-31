---

## MCP 协议与 Agent 服务开发经验

**收录日期**：2026-02-01
**来源日期**：2026-01-30 ~ 2026-02-01
**标签**：#MCP #Agent #AI #协议 #服务集成
**状态**：✅ 已验证
**适用版本**：MCP 0.3+ / FastMCP 0.2+ / AstrBot v4.13.2+

**问题/场景**：

AI Agent 生态（如 AstrBot、Claude、Cursor）需要统一的工具协议，支持多种传输模式（stdio、SSE），并能在 Docker 容器内高效部署。

**解决方案/结论**：

- MCP 协议推荐优先使用 stdio 传输，SSE 受限于容器网络（如 DNS rebinding）。
- FastMCP 框架适合快速开发 Python 工具服务，支持 Playwright、BeautifulSoup4 等库。
- 工具命名建议加前缀（如 kt_）避免与内置工具冲突。
- 搜索引擎爬取建议多选择器兜底，优先 Bing、DuckDuckGo，最后 Baidu。
- Docker 内部署需提前安装 Playwright 浏览器依赖。

**关键代码**：

```python
# MCP 服务启动示例
from mcp.server.fastmcp import FastMCP
FastMCP.run(tools=[...], transport="stdio")
```

**参考链接**：
- [MCP 官方文档](https://github.com/fastmcp/fastmcp)
- [Playwright 官方文档](https://playwright.dev/python/)

**验证记录**：
- [2026-01-30] MCP 服务本地开发，SSE 受限于 Docker 网络
- [2026-02-01] AstrBot 容器内 stdio 部署，工具正常加载

**相关经验**：
- [Docker 容器部署经验](../tools/misc.md)
- [Python 自动化爬虫](../python/automation.md)
