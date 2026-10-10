---
name: notion-mcp-publish
description: Work on saving meeting notes to Notion through the MCP connector — publisher agent, database schema, idempotent upsert, auth. Use for anything under adapters/notion or the Publish step.
---

# Notion MCP publishing

- Server: `@notionhq/notion-mcp-server` over stdio, started with `NOTION_TOKEN` (internal integration). The target database/page must be shared with the integration.
- Connect through CrewAI MCP support (`StdioServerParameters` + `MCPServerAdapter`, or the agent `mcps` field). Keep the process lifetime inside a context manager.
- Behind `PublisherPort`; a `MarkdownPublisher` fallback writes `.md` to `KNOT_DATA_DIR/exports` so the pipeline never blocks on Notion.
- **Discover tools at runtime** (`adapter.tools`) and pick by name; do not hard-code tool signatures. Verify them against the installed server version when upgrading.
- Schema bootstrap: on first run, search for the "Meetings" database, create it if missing with the properties listed in `ARCHITECTURE.md §6`.
- Idempotency: query by `Meeting ID`; update the existing page and replace body blocks, otherwise create. Store `notion_page_id` in `meetings`.
- Page body order: TL;DR, Summary, Decisions, Action items (to-do blocks), Open questions, collapsed Transcript. Respect Notion block limits (100 children per request, 2000 chars per rich-text item) by chunking.
- Retry with backoff on 429/5xx; publish failures set `status=publish_failed` and never discard local notes.
- Tests mock the MCP session; never call the real Notion API in CI.
