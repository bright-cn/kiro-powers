# 阶段 2：抓取操作手册（精简版）

为用户的目标选择正确的 Bright Data API 和选择器。输出供下一阶段使用的决策记录。

> 这是长篇 `scraper-builder` 技能的**精简**版本。如需更深入的分析（反机器人升级、多站点并行、重试语义、浏览器会话复用），请以 brightdata-plugin 的 `scraper-builder` 技能为准。本阶段是一个可用的子集。

## 第 1 步：决策树（四行核心逻辑）

1. **此域名有预构建爬虫工具吗？** → 使用**网页爬虫工具 API**（无需解析，直接返回结构化 JSON）。
2. **静态 HTML，且不需要交互？** → 使用**网络解锁器**（成本最低、最简单）。
3. **由 JS 渲染，或需要点击/滚动/填写表单？** → 使用**浏览器 API**（完整自动化）。
4. **搜索引擎结果页（Google/Bing/Yandex）？** → 使用**搜索引擎 API**。

## 第 2 步：检查预构建爬虫工具

使用一次 curl 调用发现所有可用的预构建爬虫：

```bash
curl -H "Authorization: Bearer $BRIGHTDATA_API_KEY" \
     https://api.brightdata.com/datasets/list
```

在响应中搜索目标域名。如果找到匹配项，请记录 `dataset_id` 并**跳到第 5 步**：无需侦察。

常见匹配项：`amazon`、`linkedin`、`instagram`、`tiktok`、`youtube`、`facebook`、`twitter`（X）、`reddit`、`walmart`、`ebay`、`crunchbase`、`zillow`、`booking`、`yahoo-finance`、`google-play`、`apple-app-store`。

## 第 3 步：侦察（没有预构建爬虫工具）

### 第 3a 步：获取原始 HTML

```bash
curl -X POST https://api.brightdata.com/request \
  -H "Authorization: Bearer $BRIGHTDATA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"zone": "'"${BRIGHTDATA_UNLOCKER_ZONE:-unblocker}"'", "url": "<TARGET_URL>", "format": "raw"}'
```

### 第 3b 步：检查

- **数据是否在 HTML 中？** 如果是 → 网络解锁器足以完成任务。
- **HTML 是否基本只是一个空壳**（`<div id="root"></div>`、`<div id="__next"></div>`、`ng-app`）？→ 内容由客户端渲染 → **升级到浏览器 API**。
- **页面是否引用内部 JSON 端点**（查找 `fetch('/api/...')`、`XMLHttpRequest`、`<script>` 标签中硬编码的 JSON）？→ 通过网络解锁器直接访问 API 端点比解析 HTML 更简洁。

### 第 3c 步：选择选择器（按优先级排序）

优先使用数据属性（`data-*`），而不是结构选择器：它们更能适应页面改版。

1. **`data-*` 属性**（例如 `[data-testid="product-price"]`）：更能适应页面改版。
2. **语义化类名**（例如 `.product-card .price`）。
3. **`id` 属性**：具有唯一性，但可能发生变化。
4. **结构选择器**（例如 `div > span:nth-child(2)`）：脆弱，**应避免使用**。

## 第 4 步：分页模式

| 模式 | 检测方式 | 实现方式 |
|---------|-----------|----------------|
| **基于 URL** | 类似 `?page=N`、`?offset=20` 的 URL | 循环 `?page=N`，直到响应不再包含条目 |
| **下一页链接** | HTML 包含 `<a rel="next">` 或 `.pagination .next a` | 持续访问下一页链接，直到链接不存在 |
| **游标** | API 响应包含 `next_cursor`、`next_token` | 使用上一响应中的 `?cursor=<token>` 循环请求 |
| **无限滚动** | 滚动时加载内容，URL 不变 | 仅限浏览器 API：持续滚动，直到条目数量不再增加 |

## 第 5 步：并发

如果用户在阶段 1 中说明数据量为“批量”（50 个以上 URL），生成的代码**必须**使用受信号量限制的异步处理，并将 `CONCURRENCY=20` 设为默认值。此数据量下不可接受顺序执行的 `time.sleep(1)` 循环。

## 第 6 步：输出决策记录

交回编排器：

```
{
  "approach": "web-unlocker" | "browser-api" | "pre-built" | "serp",
  "dataset_id": "<if pre-built>" | null,
  "selectors": { "<field>": "<css selector>", ... },
  "pagination": "url-based" | "next-link" | "cursor" | "infinite-scroll" | "none",
  "concurrency_required": true | false,
  "hidden_api_endpoint": "<URL or null>"
}
```

返回编排器前，使用一条简短消息向用户确认选择：

> 我将使用**网络解锁器**和以下选择器：`<list>`。分页方式为**基于 URL**。可以吗？

等待确认，然后返回。
