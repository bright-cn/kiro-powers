# 阶段 4：MCP 接入与冒烟测试

将 Bright Data MCP 服务器接入项目，运行单页冒烟测试，并完成 README 收尾内容。

## 第 1 步：项目级 MCP 配置

读取或创建 `.kiro/settings/mcp.json`。应用以下合并逻辑：

- **文件不存在** → 使用以下内容创建文件：
  ```json
  {
    "mcpServers": {
      "brightdata": {
        "url": "https://mcp.brightdata.com/mcp?token=${BRIGHTDATA_API_KEY}",
        "disabled": false
      }
    }
  }
  ```

- **文件存在，但没有 `brightdata` 键** → 将上述条目合并到 `mcpServers` 中。

- **文件存在，且 `brightdata` 键使用相同 URL** → 不执行任何操作。

- **文件存在，但 `brightdata` 键使用不同 URL** → 向用户展示差异并询问：
  > “现有的 `brightdata` MCP 服务器指向 `<existing-url>`。你希望我：(a) 将其替换为标准 URL，(b) 保持不变，还是 (c) 追加 `&pro=1` 以启用 Pro 工具？”

**写入前始终展示差异。** 不要静默写入。

## 第 2 步：用户级替代方案

告诉用户：

> “如果你希望在**每个** Kiro 项目中使用 Bright Data MCP（而不只是此项目），请改为将相同的 `brightdata` 块复制到 `~/.kiro/settings/mcp.json`。如果两者都存在，项目级 `.kiro/settings/mcp.json` 会覆盖用户级配置。”

## 第 3 步：内联检查令牌配置

验证令牌在运行时确实可用：

1. **环境变量路径：** `echo $BRIGHTDATA_API_KEY` 应为非空值。
2. **硬编码路径：** `.kiro/settings/mcp.json`（或 `~/.kiro/settings/mcp.json`）中的 URL 不包含字面量 `${BRIGHTDATA_API_KEY}`（它已被替换为真实令牌）。

如果两者均不满足，**停止**并要求用户设置环境变量或硬编码令牌，然后再继续。不要跳到第 4 步。

在**全新项目模式**下（没有项目 shell 上下文），可以提出仅为冒烟测试临时设置内联环境变量：

```bash
BRIGHTDATA_API_KEY=<token> python scrape.py
```

告诉用户这仅用于冒烟测试：之后应将其添加到 shell 配置文件或 `.env` 中。

## 第 4 步：冒烟测试

使用用户的目标 URL 运行生成的爬虫，获取 1 页样本（或 1–3 个条目，以更快者为准）。

**对于模块：** 通过单行 REPL 调用或临时脚本直接调用函数。

**对于路由：** 启动开发服务器（如果可行，也可直接调用处理函数），然后使用一次 curl 请求访问端点。

**对于智能体工具：** 使用示例输入直接调用工具的 `run`/`call`/`execute` 函数。

向用户展示结果。

### 结果处理

- **结果为空、所有字段均为 null 或出现错误** → **返回阶段 2 的侦察步骤**，修复选择器，在阶段 3 中重新生成受影响的模块文件，并重新运行冒烟测试。**不要宣布成功。**
- **部分结果**（部分字段有值，部分为 null）→ 询问用户当前结果是否足够，或是否要继续迭代。
- **结果完整** → 继续执行第 5 步。

## 第 5 步：README 收尾

追加到 `README.md`：

```markdown
## Bright Data MCP

This project has the Bright Data MCP server configured (`.kiro/settings/mcp.json`).
Any AI agent (Claude Code, Cursor, Cline, Kiro) running against this project
gains live web tools — `search_engine`, `scrape_as_markdown`, and structured-data
extractors for 40+ platforms.

To enable the full 60+ Pro tools (e.g., `web_data_amazon_product`,
`web_data_linkedin_person_profile`), append `&pro=1` to the MCP URL in
`.kiro/settings/mcp.json`. To enable specific groups only, use
`&groups=social,ecommerce` (groups: `social`, `ecommerce`, `business`,
`finance`, `research`, `app_stores`, `travel`, `browser`, `advanced_scraping`).
```

如果该部分已存在，**跳过**：不要重复添加。

## 第 6 步：最终总结

告诉用户：

```
Done. Here's what was added:

  Code:
    ✓ <files written by Phase 3>

  MCP:
    ✓ .kiro/settings/mcp.json — Bright Data server registered
    ℹ︎ Append `&pro=1` to the MCP URL to enable 60+ Pro tools

  Smoke test:
    ✓ Scraped 1 page — extracted <N> items, all fields populated

Try it out:
    <one-line invocation appropriate to the pattern>
```

工作流已完成。返回编排器，无需采取其他操作。
