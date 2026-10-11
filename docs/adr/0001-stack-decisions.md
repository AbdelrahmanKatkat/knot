# ADR 0001 — Stack decisions

Status: proposed · Date: 2026-10-10

## 1. Whisper runs via faster-whisper, not Ollama
Ollama serves LLMs/embeddings; it has no audio-transcription endpoint (searches found no official support; verify against release notes before the build starts). `STTPort` hides the engine. Ollama is still used for local embeddings and an optional offline LLM. If Ollama gains Whisper, add an adapter only.

## 2. Desktop shell: Tauri v2 + Python sidecar
Small installers, native tray/hotkey/mic permissions, auto-update. Alternatives: Electron (heavy), pywebview (simpler but weaker installers/updates), PWA only (not a real install, no tray/system audio). Cost: freezing Python with PyInstaller; accepted.

## 3. Vector store: SQLite + sqlite-vec + FTS5
One file, trivially bundled, hybrid search without a service. Chroma/Qdrant rejected for packaging weight.

## 4. Embeddings: Ollama `nomic-embed-text`
Keeps transcript text local during indexing. Switchable to Gemini embeddings.

## 5. Notion via MCP stdio with an internal integration token
Official `@notionhq/notion-mcp-server`, launched with `NOTION_TOKEN`; requires Node (bundled/checked at first run). Hosted Notion MCP (OAuth) is a later option. Verify tool names against the server at build time.

## 6. CrewAI Flows for orchestration
Deterministic pipeline (Flow) with agent crews inside steps; map-reduce for long transcripts.
