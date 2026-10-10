---
name: rag-followup
description: Build or tune the RAG system that answers follow-up questions across past meetings — chunking, embeddings, hybrid retrieval, citations, cross-meeting action tracking.
---

# RAG follow-up

- Index at end of every pipeline run and on re-process (delete old chunks for that `meeting_id` first).
- Chunking: ~500 tokens, 80 overlap, split on segment boundaries; each chunk keeps `meeting_id`, `start`, `end`, speaker. Also index the generated notes as separate chunks flagged `kind=notes`.
- Embeddings via `EmbeddingsPort` (default Ollama `nomic-embed-text`, prefix `search_document:` / `search_query:`). Record the embedding model id; a model change requires a re-index.
- Retrieval: hybrid (sqlite-vec top-k + FTS5 BM25, reciprocal-rank fusion), then MMR; k≈8. Support filters: meeting, date range, tag.
- Answering: Gemini, answer only from retrieved context, cite `[meeting title · mm:ss]`. If nothing relevant, answer "Not found in your meetings." Return citations as structured data for the UI to deep-link to the transcript.
- Follow-up features: open action items across meetings, "what changed since last meeting", draft follow-up email/agenda.
- Evals (`evals/rag/`): citation accuracy, answer faithfulness, retrieval recall@k. Run before changing chunking or embedding model.
