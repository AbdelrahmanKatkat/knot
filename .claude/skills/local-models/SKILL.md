---
name: local-models
description: Set up and manage local models — faster-whisper for transcription and Ollama for embeddings/offline LLM. Use when touching STT, model download, first-run wizard or the health check.
---

# Local models

- **Ollama does not run Whisper.** Transcription = `faster-whisper` via `adapters/stt/faster_whisper.py` implementing `STTPort`.
- Model sizes: `tiny`/`base` (fast, weak), `small` (default laptop CPU), `large-v3-turbo` (best with GPU). Choose by detected hardware (CUDA, Apple Silicon, CPU cores/RAM) in the wizard; download on demand into `KNOT_DATA_DIR/models`, with progress.
- Always run VAD filtering, 16 kHz mono input (ffmpeg), `condition_on_previous_text=False` for long audio to limit hallucination loops, and chunk >30 min files with overlap merge.
- Language: auto-detect, allow override; keep Arabic/English code-switching in mind.
- Ollama: detect at `OLLAMA_HOST`; if absent, wizard offers install link; pull `nomic-embed-text` via the Ollama API with progress. Offline LLM fallback model is user-selectable.
- `/health` must report: ffmpeg, whisper model present, Ollama reachable + models, Gemini key set, Notion connected.
- Tests use tiny fixtures and mock the engines; never download models in CI.
