# ADR-0002: Local embeddings with sentence-transformers (all-MiniLM-L6-v2)

**Date:** 2026-07-09 · **Status:** Accepted

## Context
The retrieval stage needs an embedding model to turn chunks and queries into vectors. Anthropic (our generation vendor) offers **no embeddings endpoint** — they point users to third parties such as Voyage AI. So an embeddings choice is forced regardless.

## Decision
Use `sentence-transformers` with the `all-MiniLM-L6-v2` model, running locally on CPU: 384-dimensional vectors, ~80 MB download, free, offline.

## Alternatives considered
- **Voyage AI / OpenAI embedding APIs:** higher quality, but per-call cost, network dependency, an extra API key, and the mechanics stay hidden.
- **Larger local models (e.g. bge-large, gte-large):** better retrieval quality, slower and bigger; overkill for dozens of documents.

## Consequences
- (+) Free, offline, unlimited re-indexing while we experiment with chunking.
- (+) We can inspect vectors directly and compute cosine similarity by hand (Step 4).
- (−) Weaker than large API models — acceptable at our scale, and a trade-off we can articulate.
- **Critical rule learned:** query and documents must be embedded with the **same model**; swapping models means re-indexing everything.
