# Plan

## Gaps filled (things not in the original brief)
1. **Whisper cannot run on Ollama** → faster-whisper behind `STTPort` (ADR 0001).
2. **Audio capture**: mic is easy; system/loopback audio (Zoom/Meet) differs per OS (WASAPI loopback, macOS needs a virtual device/ScreenCaptureKit, PulseAudio monitor). MVP: mic + file import; system audio in Phase 5.
3. **Speaker diarization** (who said what) for action-item owners — optional pyannote (needs HF token) or LLM-inferred names from the transcript.
4. **Long meetings** — chunking and map-reduce; Gemini context is not the limit, quality and cost are.
5. **Privacy/consent** — recording consent reminder, cloud-data disclosure, local-only mode.
6. **Secrets** — OS keychain, not plaintext config.
7. **Model/dependency management** — first-run wizard: Whisper download, Ollama detection, ffmpeg, Node for the Notion MCP server.
8. **Offline/failure behavior** — Gemini down → queue and retry, or Ollama fallback; Notion down → keep local, retry publish.
9. **Evals and test fixtures** — public sample meetings, faithfulness checks.
10. **Languages** — Whisper multilingual; confirm Arabic/English mixed meetings and RTL UI.
11. **Cost tracking**, **export** (Markdown/PDF), **calendar attendee import** (later), **licensing** (pick MIT/Apache-2.0).

## Phases
| Phase | Deliverable | Exit criteria |
|---|---|---|
| 0 Scaffold | `uv` backend, FastAPI health, Vite UI, Tauri shell, CI (lint, test, build), theme tokens | `knot serve` + UI render themed empty state |
| 1 Transcribe | Import audio → faster-whisper → segments in SQLite, progress WS | 1 h file transcribed with timestamps |
| 2 Crew | MeetingFlow with Pydantic outputs, Gemini, map-reduce, Reviewer | Evals pass on 5 fixtures |
| 3 Notion | MCP publisher, idempotent upsert, schema bootstrap | Re-run updates same page |
| 4 RAG | Chunk, embed (Ollama), hybrid retrieval, cited chat, cross-meeting follow-ups | Citation accuracy ≥ 90% on eval set |
| 5 Capture | Mic recording, hotkey, tray, system audio per OS | Record → notes end to end |
| 6 Package | Sidecar freeze, installers, updater, first-run wizard | Clean-VM install works offline except Gemini/Notion |
| 7 Polish | Design pass, a11y, RTL, diarization option, export | Design review vs DESIGN.md |

## Decisions needed from you
- Languages to support at launch (English only, or Arabic/English)?
- Primary OS for first installer (Windows / macOS / Linux)?
- OK to send transcript text to Gemini (cloud), or must "local-only" be the default?
- License for the repo.
