# Architecture

## 1. Goals
- Runs entirely on the user's laptop; installable (Windows/macOS/Linux) as a desktop app.
- Audio → transcript locally (Whisper). Summaries by Gemini via a CrewAI crew.
- Notes saved to Notion through an MCP connector.
- Follow-up Q&A across past meetings via local RAG.

## 2. System context
```
 ┌──────────────── Laptop ─────────────────────────────────────────────┐
 │  Tauri shell ── React UI ──HTTP/WS──► FastAPI backend (sidecar)     │
 │                                         │                           │
 │                  ┌──────────────────────┼───────────────┐           │
 │                  ▼                      ▼               ▼           │
 │            Audio + STT            CrewAI flows       RAG service    │
 │         (ffmpeg, faster-whisper)  (agents, tasks)   (sqlite-vec)    │
 │                  │                  │      │            │           │
 │                  ▼                  │      │            ▼           │
 │            SQLite store ◄───────────┘      │       Ollama (embed)   │
 └────────────────────────────────────────────┼────────────────────────┘
                                              │ cloud
                          Gemini API ◄────────┤   Notion MCP (stdio → Notion API)
```

## 3. Pipeline (per meeting)
1. **Capture/Import** — record mic (+ system audio where the OS allows) or import a file; ffmpeg normalizes to 16 kHz mono WAV.
2. **Transcribe** — `STTPort` (faster-whisper) with VAD, chunking, word/segment timestamps; progress streamed over WebSocket. Optional diarization (pyannote, off by default).
3. **Summarize crew** (`MeetingFlow`, CrewAI Flow):
   - `Cleaner` — fix transcript noise, segment into topics (map step, parallel chunks).
   - `Summarizer` — TL;DR + section summaries (reduce step).
   - `ActionExtractor` — action items {owner, task, due, source_ts}.
   - `DecisionLogger` — decisions, open questions, risks.
   - `Reviewer` — checks every action/decision is grounded in the transcript; drops ungrounded ones.
4. **Publish** — `Notion Publisher` agent uses MCP tools to upsert the page in the Meetings database (idempotent on `meeting_id`).
5. **Index** — chunk transcript + notes (≈500 tokens, 80 overlap, keep timestamps), embed via `EmbeddingsPort`, store in sqlite-vec + FTS5.
6. **Follow-up** — `FollowUpCrew`: `Retriever` (hybrid vector+BM25, MMR) → `Answerer` (Gemini, must cite meeting + timestamp). Also drafts follow-up emails/agendas and tracks open action items across meetings.

## 4. Components and ports
| Port (`adapters/*/base.py`) | Default adapter | Alternatives |
|---|---|---|
| `STTPort` | faster-whisper (CTranslate2) | whisper.cpp, Gemini audio (opt-in) |
| `LLMPort` | Gemini via CrewAI `LLM` | Ollama (offline fallback) |
| `EmbeddingsPort` | Ollama `nomic-embed-text` | Gemini embeddings |
| `PublisherPort` | Notion MCP (stdio) | Markdown export to disk |
| `VectorStorePort` | sqlite-vec | — |

## 5. Data model (SQLite)
`meetings(id, title, started_at, duration, language, audio_path, status, notion_page_id)` ·
`segments(meeting_id, idx, start, end, speaker, text)` ·
`notes(meeting_id, summary_json, actions_json, decisions_json, model, created_at)` ·
`chunks(id, meeting_id, start, end, text)` + `chunks_fts` + `chunk_vectors` (vec0) ·
`jobs(id, meeting_id, stage, state, error)` · `settings(key, value)` (no secrets).

## 6. Notion schema (Meetings database)
Properties: Title, Date, Duration, Attendees, Status (Processed/Needs review), Action Items (count), Meeting ID (text, upsert key), Source (Knot). Page body: TL;DR → Summary → Decisions → Action items (to-do blocks) → Open questions → collapsed Transcript.

## 7. API (FastAPI, localhost only, token-guarded)
`POST /meetings` (upload) · `POST /meetings/record/start|stop` · `GET /meetings`, `/meetings/{id}` · `POST /meetings/{id}/process` · `WS /meetings/{id}/events` · `POST /meetings/{id}/publish` · `POST /chat` (RAG, SSE stream) · `GET/PUT /settings` · `GET /health` (models, Ollama, Notion status).
Bind to `127.0.0.1` with a random per-launch token passed by the Tauri shell.

## 8. Packaging
Tauri v2 app bundles the built UI and a PyInstaller-frozen backend sidecar. First run wizard: choose Whisper size (tiny→large-v3-turbo; downloaded on demand), check/install Ollama + pull embedding model, paste Gemini key, connect Notion. ffmpeg bundled. Auto-update via Tauri updater. CI builds `.msi`, `.dmg`, `.AppImage/.deb` on tags.

## 9. Cross-cutting
- **Security/privacy**: localhost bind + token, keychain secrets, explicit cloud-data consent, "local-only mode" (Ollama LLM, no Notion).
- **Resilience**: job queue with resumable stages; Gemini retries/backoff; publish retried independently of summarization.
- **Observability**: structured logs without content; per-meeting token/cost counters.
- **Quality**: golden fixtures + evals (`evals/`) for summary faithfulness, action-item recall, RAG citation accuracy.
- **i18n**: Whisper is multilingual; UI supports RTL (Arabic) from day one.
