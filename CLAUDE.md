# CLAUDE.md — Knot

Local-first meeting summarizer: audio → local Whisper → CrewAI crew (Gemini) → Notion (MCP) + local RAG for follow-up. Desktop-installable on a laptop. Read `ARCHITECTURE.md` before structural changes and `docs/DESIGN.md` before touching UI.

## Stack (decided — see docs/adr/0001-stack-decisions.md)
- Backend: Python 3.11+, FastAPI, CrewAI, `uv` for deps. Package at `backend/src/knot`.
- STT: `faster-whisper` (local). **Ollama does not serve Whisper**; do not write code that assumes it does.
- LLM: Gemini via CrewAI `LLM` (model id from `KNOT_LLM_MODEL`). Ollama LLM is an optional offline fallback.
- Embeddings: Ollama `nomic-embed-text` (local default); Gemini embeddings optional.
- Store: SQLite (+ FTS5 + `sqlite-vec`) in `KNOT_DATA_DIR`. No server DB.
- Notion: MCP stdio server (`@notionhq/notion-mcp-server`) driven through CrewAI MCP support.
- UI: React + Vite + Tailwind in `ui/`. Shell: Tauri v2 in `desktop/`, backend shipped as a PyInstaller sidecar.

## Layout
```
backend/src/knot/  api/ audio/ crew/{config,flows}/ adapters/{stt,llm,embeddings,notion}/ rag/ store/
backend/tests/     ui/    desktop/    evals/    scripts/    docs/    .claude/skills/
```

## Rules
1. **Ports and adapters.** `crew/`, `rag/` and `api/` depend on interfaces in `adapters/*/base.py`, never on faster-whisper, Gemini, Ollama or Notion directly. New provider = new adapter.
2. **Agents and tasks live in YAML** (`crew/config/agents.yaml`, `tasks.yaml`). Python only wires them. Every task has a Pydantic `output_pydantic` model.
3. **Local first / privacy.** Audio and raw transcripts never leave the machine except transcript text sent to Gemini, and only after the user has acknowledged that in Settings. Never log transcript content or API keys.
4. **Secrets** come from the OS keychain (`keyring`), with `.env` for dev only. Never commit `.env`. `.env.example` must stay in sync.
5. **Notion writes are idempotent**: keyed by `meeting_id`; re-running updates the page, never duplicates it.
6. **RAG answers must cite** meeting id + timestamp. No citation → say "not found in your meetings".
7. Long audio is chunked; never assume a transcript fits one LLM call (map-reduce summarization).
8. UI follows the painting design system; use tokens from `ui/src/theme`, no hard-coded hex values.
9. Prefer small PRs, conventional commits (`feat:`, `fix:`, `docs:`), tests with each feature. Do not open a PR unless asked.

## Commands (available after Phase 0; keep this list current)
```
uv sync --project backend            # install
uv run --project backend pytest      # tests
uv run --project backend ruff check . && uv run --project backend ruff format .
uv run --project backend knot serve  # API on :8765
pnpm --dir ui dev                    # UI
pnpm --dir desktop tauri dev         # desktop shell
python evals/run.py                  # summary/RAG quality evals
```

## Skills (`.claude/skills/`)
`add-crew-agent`, `notion-mcp-publish`, `rag-followup`, `knot-design-system`, `local-models`, `package-desktop`.

## Definition of done
Tests + lint pass, `ARCHITECTURE.md`/skills updated if behavior or structure changed, no secrets or transcripts in logs or fixtures.
