# ADR-0003: ChromaDB as the vector store

**Date:** 2026-07-09 · **Status:** Accepted

## Context
We need somewhere to store chunk vectors + text + metadata and run top-k cosine-similarity queries. Requirements: local, persistent, pip-installable, minimal ceremony.

## Decision
ChromaDB in embedded mode (`PersistentClient`), persisting to `./chroma_db/`. Think "SQLite for vectors."

## Alternatives considered
- **NumPy brute force:** we build this for real first (Steps 4–5) behind a shared `VectorStore` interface, then swap in Chroma in Step 6 and compare — see ADR-0005 for the sequencing decision. Brute force lacks persistence/metadata ergonomics, which is exactly what the comparison teaches.
- **FAISS:** a similarity-search library, not a database — no built-in persistence or metadata; great at massive scale we don't have.
- **pgvector:** the strong production answer inside an existing Postgres stack; too much server for a local learning project.
- **Pinecone/Weaviate (managed):** accounts, network, cost; nothing extra learned locally.

## Consequences
- (+) One `pip install`, persists across sessions, metadata filtering built in.
- (+) Clean production story for interviews: "Chroma locally; pgvector or a managed store at scale, with ANN indexes like HNSW once counts get large."
- (−) Not what most large enterprises run in production — which is fine, because we can say why.
