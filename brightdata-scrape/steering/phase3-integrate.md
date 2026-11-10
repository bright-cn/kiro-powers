# 阶段 3：集成

在用户项目中生成爬虫代码。模式（模块 / 路由 / 工具）已在阶段 1 中选择，API 和选择器已在阶段 2 中选择。本阶段会选择正确的模板、填充模板并写入文件：但仅在与用户确认每个路径后执行。

## 第 1 步：选择模板

使用阶段 1 决策记录中的 `pattern` 和 `framework` 字段。查找规范模板：

| 模式 | 框架 | 模板路径 |
|---------|-----------|---------------|
| `module` | TypeScript | `templates/module/ts-cheerio.ts`（如果用户已安装 cheerio）或 `templates/module/ts-fetch.ts`（否则） |
| `module` | Python | `templates/module/py-bs4.py`（如果可以使用 bs4）或 `templates/module/py-stdlib.py`（否则） |
| `route` | Next.js App Router | `templates/route/next-app-router.ts` |
| `route` | Next.js Pages Router | `templates/route/next-pages-router.ts` |
| `route` | Express | `templates/route/express.ts` |
| `route` | Fastify | `templates/route/fastify.ts` |
| `route` | Hono | `templates/route/hono.ts` |
| `route` | Koa | `templates/route/koa.ts` |
| `route` | FastAPI | `templates/route/fastapi.py` |
| `route` | Flask | `templates/route/flask.py` |
| `route` | Django | `templates/route/django.py` |
| `tool` | LangChain（TS） | `templates/tool/langchain-ts.ts` |
| `tool` | LangChain（Python） | `templates/tool/langchain-py.py` |
| `tool` | Anthropic SDK（TS） | `templates/tool/anthropic-sdk-ts.ts` |
| `tool` | Anthropic SDK（Python） | `templates/tool/anthropic-sdk-py.py` |
| `tool` | OpenAI SDK（TS） | `templates/tool/openai-ts.ts` |
| `tool` | OpenAI SDK（Python） | `templates/tool/openai-py.py` |
| `tool` | Mastra | `templates/tool/mastra.ts` |
| `tool` | Vercel AI SDK | `templates/tool/vercel-ai-sdk.ts` |
| **其他语言** | 任意 | `templates/fallback/curl.sh`（手动调整，并告知用户） |

**如果模板文件不存在**（因为 v1 仅包含部分模板），请回退到同一系列中最接近的可用模板，并告知用户：

> “我目前还没有适用于 `<framework>` 的规范模板：我会以 `<closest>` 模板为起点并进行调整。你可能需要调整导入或语法。”

### 路由和工具也需要模块模板

路由和工具模式会从同级模块文件导入爬虫。因此，当模式为 `route` 或 `tool` 时，还必须选择一个**模块**模板：

- TypeScript 项目：如果项目已依赖 `cheerio`，使用 `templates/module/ts-cheerio.ts`；否则使用 `templates/module/ts-fetch.ts`（无解析器依赖）。
- Python 项目：如果可以添加 `beautifulsoup4`（或它已是依赖项），使用 `templates/module/py-bs4.py`；否则使用 `templates/module/py-stdlib.py`（仅使用标准库）。

换言之：路由和工具模式会生成**两个**文件，路由/工具包装器及其导入的模块。

## 第 2 步：填充模板

使用决策记录中的值替换模板里的以下占位符：

- `{{TARGET_URL}}` → 阶段 1 的 `target_url`
- `{{TARGET_NAME}}` → 从 URL 主机派生的 snake_case 名称（例如 `amazon_com`、`competitor_prices`）
- `{{FIELDS}}` → 阶段 1 的 `fields`，按模板要求格式化（TS 接口、Python TypedDict 等）
- `{{SELECTORS}}` → 阶段 2 的 `selectors`（CSS 字符串）
- `{{PAGINATION}}` → 阶段 2 的 `pagination`（'url-based' / 'next-link' / 'cursor' / 'infinite-scroll' / 'none'）
- `{{APPROACH}}` → 阶段 2 的 `approach`
- `{{DATASET_ID}}` → 阶段 2 的 `dataset_id`（仅用于预构建爬虫）
- `{{CONCURRENCY}}` → 如果为 `concurrency_required`，则设为 20；否则设为 1
- `{{HIDDEN_API_ENDPOINT}}` → 阶段 2 的 `hidden_api_endpoint`（如果存在）

生成的文件始终从环境中读取 **`BRIGHTDATA_API_KEY`** 和 **`BRIGHTDATA_UNLOCKER_ZONE`**（默认为 `"unblocker"`）。

## 第 3 步：选择目标路径

对于**模块**模式：
- TS：`src/scrapers/{{TARGET_NAME}}.ts`
- Python：`src/scrapers/{{TARGET_NAME}}.py`，如果 `src/` 不存在，则使用项目现有的源代码目录（检查 `app/`、`lib/` 和顶层 `.py` 文件）

对于**路由**模式：
- Next.js App Router：`app/api/scrape/route.ts`
- Next.js Pages Router：`pages/api/scrape.ts`
- Express / Fastify / Hono / Koa：`src/routes/scrape.ts`（或项目现有路由所在位置，先检查）
- FastAPI：`app/api/scrape.py`（向用户展示路由器 include 行）
- Flask：`app/scrape.py`（蓝图，并展示 `app.register_blueprint(...)` 行）
- Django：`<app>/views/scrape.py`（展示 URL pattern 行）

路由模式**还会**生成模块文件（路由会导入该模块）。

对于**工具**模式：
- TS：`src/tools/scrape.ts`
- Python：`src/tools/scrape.py`

工具模式**还会**生成模块文件（工具会调用该模块）。

## 第 4 步：更新环境配置和 README

始终更新：

- `.env.example`：追加以下内容（跳过已存在的行）：
  ```
  # Get token at https://brightdata.com/cp/setting/users
  BRIGHTDATA_API_KEY=
  # Optional — defaults to the account's "unblocker" zone if unset.
  # Manage zones at https://brightdata.com/cp/zones
  BRIGHTDATA_UNLOCKER_ZONE=
  ```

- `README.md`：追加（或创建）一个 `## Web scraping` 部分，其中包含一个展示如何调用所生成函数/路由/工具的用法示例。

## 第 5 步：确认关卡

写入**任何**文件之前，先向用户展示文件列表：

```
I'll write or modify these files:
  • CREATE src/scrapers/competitor_prices.ts
  • CREATE app/api/scrape/route.ts
  • MODIFY .env.example (append BRIGHTDATA_API_KEY, BRIGHTDATA_UNLOCKER_ZONE)
  • MODIFY README.md (append "## Web scraping" section)

OK to write?
```

**等待确认。在用户同意之前，不要编辑任何文件。**

## 第 6 步：现有文件安全

- **绝不覆盖现有爬虫文件。** 如果 `src/scrapers/competitor_prices.ts` 已存在，请建议使用 `competitor_prices_2.ts`，或询问用户要将其放在哪里。
- **绝不完整覆盖 `README.md` 或 `.env.example`。** 只能追加。如果 `BRIGHTDATA_API_KEY` 已存在于 `.env.example` 中，则跳过。如果 `## Web scraping` 部分已存在，则追加一个子部分。
- **绝不重复添加路由注册行。** 如果用户的 `app.ts` 已包含 `app.use('/api/scrape', ...)`，则跳过该建议。

## 第 7 步：输出

写入后，向用户汇总：

```
Files written:
  ✓ src/scrapers/competitor_prices.ts
  ✓ app/api/scrape/route.ts
  ✓ .env.example
  ✓ README.md

Next: Phase 4 will wire Bright Data MCP and run a smoke test.
```

将控制权交回编排器。
