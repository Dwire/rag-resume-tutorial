# Architecture Overview

## The two pipelines

Every RAG system is really **two pipelines that meet at a vector store**:

1. **Indexing (offline, run when your documents change):** documents → chunks → embeddings → vector store.
2. **Query (online, run per job application):** job posting → query embedding(s) → similarity search → retrieved chunks → prompt → LLM → formatted output.

"Offline/online" is standard RAG vocabulary worth using in interviews: indexing is done ahead of time; querying happens at request time.

## Diagram

```
════════════════ INDEXING PIPELINE (offline) ═══════════════════

 personal_docs/            ┌─────────────┐      ┌──────────────┐
 ├── job_history.md   ───▶ │  1. LOAD    │ ───▶ │  2. CHUNK    │
 ├── side_projects.md      │  read files │      │  split into  │
 └── skills.md             │  + metadata │      │  ~300-token  │
                           └─────────────┘      │  overlapping │
                                                │  pieces      │
                                                └──────┬───────┘
                                                       │ chunks + metadata
                                                       ▼
                           ┌──────────────────────────────────┐
                           │  3. EMBED (sentence-transformers)│
                           │  each chunk → 384-dim vector     │
                           └──────────────┬───────────────────┘
                                          │ (vector, text, metadata)
                                          ▼
                           ┌──────────────────────────────────┐
                           │  4. VECTOR STORE                 │
                           │  (VectorStore interface:         │
                           │   NumPy brute force first,       │
                           │   ChromaDB swapped in later —    │
                           │   persists to ./chroma_db/)      │
                           └──────────────────────────────────┘
                                          ▲
════════════════ QUERY PIPELINE (per job) ║ ════════════════════
                                          ║ similarity search
 job_posting.txt          ┌───────────────╨──────────────────┐
 (pasted text) ─────────▶ │  5. RETRIEVE                     │
                          │  a. extract the posting's key    │
                          │     requirements (queries)       │
                          │  b. embed each query             │
                          │  c. top-k nearest chunks by      │
                          │     cosine similarity            │
                          └───────────────┬──────────────────┘
                                          │ ranked, deduped chunks
                                          ▼
                          ┌──────────────────────────────────┐
                          │  6. PROMPT ASSEMBLY              │
                          │  system rules (ground ONLY in    │
                          │  provided chunks, length caps)   │
                          │  + job posting + chunks          │
                          └───────────────┬──────────────────┘
                                          ▼
                          ┌──────────────────────────────────┐
                          │  7. GENERATE (Claude API)        │
                          │  claude-opus-4-8                 │
                          └───────────────┬──────────────────┘
                                          ▼
                          ┌──────────────────────────────────┐
                          │  8. OUTPUT FORMATTING            │
                          │  resume.md (1 page) and/or       │
                          │  cover_letter.md (≤300 words)    │
                          │  + word-count / grounding checks │
                          └──────────────────────────────────┘
```

## Stage-by-stage

| # | Stage | What it does | Data in → out |
|---|---|---|---|
| 1 | **Load** | Read every `.md`/`.txt` in `personal_docs/`, attach metadata (filename, headings) | files → `Document{text, metadata}` |
| 2 | **Chunk** | Split documents into overlapping ~300-token pieces at paragraph/heading boundaries. Too big → retrieval gets vague; too small → chunks lose context. | documents → `Chunk{text, metadata}` list |
| 3 | **Embed** | Run each chunk through a local embedding model; meaning becomes geometry (similar meaning → nearby vectors) | chunk text → 384-float vector |
| 4 | **Store** | Keep (vector, text, metadata) triples behind a small `VectorStore` interface so we index once and query many times. Implemented twice: NumPy brute force first, then ChromaDB (persistent) swapped in behind the same interface (ADR-0005). | vectors → in-memory NumPy arrays, later a ChromaDB collection on disk |
| 5 | **Retrieve** | Turn the job posting into one or more query strings, embed them, and pull the k most similar chunks (cosine similarity). Multiple targeted queries beat one blob — a key insight we'll test ourselves. | posting → top-k chunks with scores |
| 6 | **Prompt assembly** | Build the message for the LLM: strict grounding rules ("use ONLY the provided excerpts; if something isn't there, don't claim it"), the posting, the chunks, format constraints | chunks + posting → prompt string |
| 7 | **Generate** | One Claude API call writes the tailored resume/cover letter | prompt → draft text |
| 8 | **Output** | Save Markdown files; verify the 300-word cap and spot-check every claim traces to a chunk | draft → `output/*.md` |

## Core RAG concepts vs. app-specific plumbing

Interviewers care about the left column; the right column just makes *this* app work.

| Core RAG concepts (transferable) | App-specific plumbing |
|---|---|
| Chunking strategy: size, overlap, semantic boundaries | Reading `.md` files from a folder |
| Embeddings & vector similarity (cosine) | The 300-word cover-letter cap |
| Vector store: indexing, top-k search, metadata filtering | Resume Markdown template |
| Query construction (posting → multiple retrieval queries) | CLI flags / Streamlit UI |
| Grounded generation & hallucination prevention via prompt | `.env` API-key handling |
| Evaluation: is retrieval finding the right chunks? Is output grounded? | File naming, output folder |

## Where this differs from "production RAG" (know for interviews)

- **Scale:** we have dozens of chunks; production has millions → approximate-nearest-neighbor indexes (HNSW), sharding.
- **Retrieval quality:** production adds hybrid search (keyword BM25 + vectors) and rerankers; we'll discuss these in Step 9.
- **Evaluation:** production uses automated eval harnesses (retrieval recall, groundedness scoring); we'll do a manual/lightweight version.
- **Freshness:** production pipelines re-index continuously; we re-run a script.

Being able to say "here's what I built, and here's what I'd change at scale" is exactly the interview posture this project is designed to produce.
