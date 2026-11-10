---
name: "brightdata-scrape"
displayName: "Add web scraping to any app with Bright Data"
description: "Detect your project's stack and add production-ready web scraping — generates the right integration pattern (module, API route, or agent tool), wires Bright Data MCP into your project, handles pagination and bot detection."
keywords: ["scrape", "scraping", "scraper", "crawl", "crawler", "web-data", "extract", "extract-data", "competitor", "pricing-monitor", "lead-generation", "amazon", "linkedin", "instagram", "tiktok", "youtube", "serp", "google-search", "search-engine", "brightdata", "bright-data", "web-unlocker", "browser-api", "captcha", "bot-detection", "pagination", "agent-tools", "mcp"]
author: "Bright Data"
---

# 使用 Bright Data 为任何应用添加网页爬虫功能

此 Power 会检测你正在处理的项目类型，并以合适的形式添加可用于生产环境的抓取功能：可复用模块、API 路由或智能体工具，底层由 Bright Data 的网络解锁器、网页爬虫工具 API、浏览器 API 和搜索引擎 API 提供支持。它还会将 Bright Data MCP 服务器接入你的项目，让任何基于该项目运行的 AI 智能体（Claude Code、Cursor、Cline、Kiro 本身）都能获得实时网页工具。

**它适用于任何语言**，但会为 Python 和 TypeScript/JavaScript 提供一流的代码生成功能。其他语言会获得一个可供你调整的通用 `curl`/HTTP 模板。

## 你可以做什么

- “每天将 example.com 上的竞争对手价格抓取到我的 Next.js 控制面板中”
- “向我的 Express 应用添加一个 `/api/scrape` 路由”
- “为我的 Claude SDK 智能体添加一个可搜索 Google 并读取结果的工具”
- “从这个 Shopify 商店提取所有商品列表”
- “监控销售管道联系人中的 LinkedIn 个人资料”

## 入门设置

使用此 Power 前，请完成以下步骤。

### 第 1 步：获取 Bright Data API 令牌

在[注册 brightdata.com](https://www.bright.cn/)页面注册（或登录）。在 [API 令牌设置](https://www.bright.cn/cp/setting/users)中生成 API 令牌。免费套餐包括**每月 5,000 次请求**，其中包含 Pro 工具。

### 第 2 步：配置令牌（任选一种）

**选项 A：环境变量（推荐用于 CI / 生产环境）：**

```bash
export BRIGHTDATA_API_KEY=<your-token>
```

将其添加到你的 shell 配置文件（`.zshrc`、`.bashrc`）或项目的 `.env` 文件中。生成的 `mcp.json` 会引用 `${BRIGHTDATA_API_KEY}`，因此可自动生效。

**选项 B：硬编码到用户级 Kiro 配置中：**

编辑 `~/.kiro/settings/mcp.json` 并添加：

```json
{
  "mcpServers": {
    "brightdata": {
      "url": "https://mcp.brightdata.com/mcp?token=YOUR_TOKEN_HERE",
      "disabled": false
    }
  }
}
```

这样即可在你设备上的每个 Kiro 项目中使用 Bright Data MCP。

### 第 3 步（可选）：设置网络解锁器区域

新账户的默认网络解锁器区域名为 `unblocker`。如果你已将其重命名或遇到“无区域”错误，请设置：

```bash
export BRIGHTDATA_UNLOCKER_ZONE=<your-zone-name>
```

或者在[区域管理页面](https://www.bright.cn/cp/zones)创建区域或重命名区域。

---

## 如何使用此 Power

对于任何抓取任务，**始终**先读取编排器引导文件：

```
Call action "readSteering" with powerName="brightdata-scrape", steeringFile="scrape-workflow.md"
```

编排器会依次运行四个阶段，每个阶段之间都有确认关卡：

1. **检测与规划**：检查项目、选择正确的集成模式，并询问要抓取的内容。
2. **抓取操作手册**：根据目标网站选择正确的 Bright Data API 和选择器。
3. **集成**：在用户项目中生成爬虫模块、API 路由或智能体工具。
4. **MCP 与验证**：接入 Bright Data MCP 服务器、运行冒烟测试并编写 README 片段。

**不要自行发挥。不要跳过任何阶段。** 引导文件中包含每个阶段的确切指令。

---

## 许可证与支持

- **许可证**：此 Power 及其底层的 [Bright Data MCP 服务器](https://github.com/bright-cn/brightdata-mcp)均根据 [MIT 许可证](https://github.com/bright-cn/brightdata-mcp/blob/main/LICENSE)发布。它调用的 Bright Data 平台工具（网络解锁器、搜索引擎 API、Web Data API、浏览器 API）属于商业服务，受 Bright Data 的[许可协议](https://www.bright.cn/license)和[可接受使用政策](https://www.bright.cn/acceptable-use-policy)约束。
- **隐私政策**：[www.bright.cn/privacy](https://www.bright.cn/privacy)
- **支持与联系**：可通过 [www.bright.cn/contact](https://www.bright.cn/contact) 获取全天候实时支持、查阅 [Bright Data 文档](https://docs.brightdata.com)，或在 [MCP 服务器仓库](https://github.com/bright-cn/brightdata-mcp/issues)中提交问题。
