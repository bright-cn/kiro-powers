# 阶段 1：检测与规划

检查用户的工作区并进行分类，然后提出一种集成模式。继续进入阶段 2 前，先与用户确认。

## 第 1 步：清单检测

按顺序在工作区根目录中查找以下清单文件：

| 清单 | 语言 |
|----------|----------|
| `package.json` | TypeScript / JavaScript |
| `pyproject.toml`, `requirements.txt`, `Pipfile` | Python |
| `go.mod` | Go |
| `Cargo.toml` | Rust |
| `Gemfile` | Ruby |
| `pom.xml`, `build.gradle` | Java / Kotlin |
| `composer.json` | PHP |

**未找到清单** → 进入**全新项目模式**。跳到第 4 步并询问用户：“使用 Python 还是 TypeScript？”

**子目录中存在多个清单（单体仓库）** → 询问用户要集成到哪个子项目中。**不要**自动选择。

## 第 2 步：技术栈特征

在选定的清单中查找以下依赖特征：

### Web 框架依赖项（路由模式）

**TypeScript/JavaScript**：在 `dependencies` 和 `devDependencies` 中查找：
- `next`、`next.js` → Next.js
- `express` → Express
- `fastify` → Fastify
- `hono` → Hono
- `koa`、`koa-router` → Koa
- `@nestjs/core` → NestJS
- `remix`、`@remix-run/react` → Remix

**Python**：在 `pyproject.toml`、`[project.dependencies]` 或 `requirements.txt` 中查找：
- `fastapi` → FastAPI
- `flask` → Flask
- `django` → Django
- `aiohttp` → aiohttp
- `starlette` → Starlette

### 智能体框架依赖项（智能体工具模式）

**TypeScript/JavaScript：**
- `langchain`、`@langchain/*` → LangChain
- `@anthropic-ai/sdk` → Anthropic SDK
- `openai` → OpenAI SDK
- `mastra`、`@mastra/*` → Mastra
- `@ai-sdk/*`、`ai` → Vercel AI SDK
- `@modelcontextprotocol/sdk` → MCP SDK

**Python：**
- `langchain`、`langchain-*` → LangChain
- `anthropic` → Anthropic SDK
- `openai` → OpenAI SDK
- `llama-index` → LlamaIndex
- `crewai` → CrewAI
- `mcp` → MCP SDK

## 第 3 步：选择唯一模式

应用以下决策树：

| 特征 | 模式 | 模板系列 |
|---------|---------|-----------------|
| 存在 Web 框架依赖项 | **API 路由** | `templates/route/` |
| 存在智能体框架依赖项，但没有 Web 框架 | **智能体工具** | `templates/tool/` |
| 同时存在 Web 框架和智能体框架 | 询问用户更偏好哪种接口（路由或工具） |
| 库 / CLI / 无法识别 | **模块** | `templates/module/` |
| 全新项目（无清单） | **模块**，使用上面询问的语言 | `templates/module/` |

**告知用户所选结果，但不要让用户投票决定**：检测结果就是最终选择。

## 第 4 步：针对性抓取问题

在**一条**消息中询问：

> 你想抓取什么内容，需要哪些字段？（例如：“从 amazon.com 搜索结果中获取商品名称 + 价格 + 图片”“从 greenhouse.io 职位列表中获取所有职位名称 + 公司”。）大约有多少个页面或条目？

不要单独询问分页类型、输出格式或语言偏好：
- **分页类型** → 在阶段 2 的侦察中确定。
- **输出格式** → 默认为类型化返回值；由调用方决定如何处理。
- **语言** → 已根据清单确定（`package.json` 使用 TypeScript，`pyproject.toml`/`requirements.txt` 使用 Python）；全新项目模式已在第 1 步中处理。

### 规范框架 slug

输出决策记录的 `framework` 字段时，请使用以下确切 slug（确保阶段 3 的模板查找正常工作）：

| 框架 | Slug |
|-----------|------|
| Next.js（App Router） | `next-app-router` |
| Next.js（Pages Router） | `next-pages-router` |
| Express | `express` |
| Fastify | `fastify` |
| Hono | `hono` |
| Koa | `koa` |
| NestJS | `nestjs` |
| Remix | `remix` |
| FastAPI | `fastapi` |
| Flask | `flask` |
| Django | `django` |
| aiohttp | `aiohttp` |
| Starlette | `starlette` |
| LangChain（TS） | `langchain-ts` |
| LangChain（Python） | `langchain-py` |
| Anthropic SDK（TS） | `anthropic-sdk-ts` |
| Anthropic SDK（Python） | `anthropic-sdk-py` |
| OpenAI SDK（TS） | `openai-ts` |
| OpenAI SDK（Python） | `openai-py` |
| Mastra | `mastra` |
| Vercel AI SDK | `vercel-ai-sdk` |
| LlamaIndex | `llama-index` |
| CrewAI | `crewai` |

如果检测到的框架不在此列表中，请使用纯小写包名，并告知用户将使用后备模板。

## 第 5 步：展示计划并等待确认

格式：

```
## Plan

**Detected:** <stack signature, e.g., "Next.js 14 (App Router) project, TypeScript">
**Pattern:** <module | API route | agent tool>
**Scraping target:** <user's site/data>
**Estimated volume:** <single | small (<50) | bulk (>=50)>

**Phases I'll run:**
  1. ✓ Detect & plan (this phase, just confirmed)
  2. Scraping playbook — pick the right Bright Data API and selectors
  3. Integrate — generate <files I'll write> in your project
  4. MCP & verify — wire Bright Data MCP, run a smoke test

Ready to proceed to Phase 2?
```

返回控制权给编排器之前，**等待用户确认。**

## 本阶段的输出

编排器会将此决策记录传递到后续阶段：

```
{
  "language": "typescript" | "python" | "other",
  "language_other_name": "<e.g., 'go'>" | null,
  "pattern": "module" | "route" | "tool",
  "framework": "<e.g., 'next-app-router', 'fastapi', 'anthropic-sdk-ts'>",
  "target_url": "<user's site>",
  "fields": ["<field1>", "<field2>", ...],
  "volume": "single" | "small" | "bulk",
  "monorepo_subproject": "<path>" | null
}
```

后续阶段会按名称引用此记录。如果其中缺少任何内容，请返回阶段 1。
