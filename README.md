# brightdata-powers

一个用于为任何应用添加网页爬虫功能的 Bright Data Kiro Power。

## 此仓库包含的内容

- **[`brightdata-scrape/`](brightdata-scrape/)**：Kiro Power 本体（`POWER.md`、`mcp.json`、引导文件和代码模板）。从此目录将其安装为 Kiro Power，或将 `brightdata-scrape/` 复制到上游 `kirodotdev/powers` 集合中以参与贡献。
- **`powers/`**：用作灵感来源的 [`kirodotdev/powers`](https://github.com/kirodotdev/powers) 克隆。它不属于此仓库的目录树（已被 gitignore 忽略）。
- **[`scripts/validate_power.py`](scripts/validate_power.py)**：用于验证 Power 结构的 CLI。
- **[`tests/test_validate_power.py`](tests/test_validate_power.py)**：验证器的 pytest 测试套件。

## `brightdata-scrape` 的作用

这是一个 Kiro Power，它会检测你的项目技术栈，并以合适的形式添加可用于生产环境的 Bright Data 抓取功能：可复用模块、API 路由或智能体工具，底层由 Bright Data 的[网络解锁器](https://www.bright.cn/products/web-unlocker)、[搜索引擎 API](https://www.bright.cn/products/serp-api)、[Web Data API](https://www.bright.cn/products/web-scraper) 和[浏览器 API](https://www.bright.cn/products/scraping-browser) 提供支持。它还会将 [Bright Data MCP 服务器](https://mcp.brightdata.com)接入你的项目，让任何基于该项目运行的 AI 智能体（Claude Code、Cursor、Cline、Kiro 本身）都能获得实时网页工具。

## 快速开始

1. 获取 [Bright Data API 令牌](https://www.bright.cn/cp/setting/users)（每月免费 5,000 次请求）。
2. 进行设置：`export BRIGHTDATA_API_KEY=<your-token>`。
3. 在 Kiro 中安装此 Power（参见下方的[安装](#安装)）。
4. 告诉 Kiro：*"为 X 添加一个抓取工具"* / *"为我的智能体提供网页搜索功能"* / *"从 amazon.com 提取商品价格"*。

## 安装

打开 Kiro 中的 **Powers** 面板，然后选择以下方式之一：

- **从本地路径添加 Power**：将 Kiro 指向此仓库本地克隆中的 `brightdata-scrape/` 目录。
- **从 GitHub 添加 Power**：粘贴仓库 URL 以及 Power 目录的路径：
  ```
  https://github.com/bright-cn/kiro-powers/tree/main/brightdata-scrape
  ```
  （此仓库是包含多个 Power 的单体仓库：`brightdata-scrape/` 才是该 Power。仅将 Kiro 指向仓库 URL 不会生效，因为 `POWER.md` 位于子目录中。）

安装后，当出现“抓取”“提取数据”“竞争对手价格”“为我的智能体提供网页搜索功能”等关键词时，Kiro 会激活此 Power。完整关键词列表和入门步骤请参阅 [`brightdata-scrape/POWER.md`](brightdata-scrape/POWER.md)。

## 验证 Power

```bash
python3 scripts/validate_power.py brightdata-scrape
python3 -m pytest tests/test_validate_power.py -v
```

## 另请参阅

- Bright Data 文档：[docs.brightdata.com](https://docs.brightdata.com)
