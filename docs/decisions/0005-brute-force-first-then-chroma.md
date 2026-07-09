# ADR-0005: Build a brute-force vector store first, then swap in ChromaDB behind a shared interface

**Date:** 2026-07-09 · **Status:** Accepted (supersedes the "Chroma from the start" sequencing implied in ADR-0003)

## Context
ADR-0003 chose ChromaDB as the vector store. The original syllabus introduced Chroma immediately after the embeddings step, with a brief NumPy demo folded into the embeddings session. Reviewing an alternative plan surfaced a stronger sequencing: give the hand-rolled store a full step, run real retrieval on it, and only then swap in Chroma.

## Decision
1. Define a minimal `VectorStore` interface (`add`, `search`) in Step 4.
2. Implement it first with NumPy brute force (linear scan + cosine similarity) and use it for actual retrieval in Step 5.
3. In Step 6, implement `ChromaVectorStore` against the same interface, swap it in with no changes to the rest of the pipeline, and compare the two side by side.

## Alternatives considered
- **Chroma from day one** (original plan): one session shorter, but the "what does a vector DB actually do?" understanding stays theoretical.
- **Never using a real vector DB:** misses persistence, metadata filtering, and the production vocabulary interviews expect.

## Consequences
- (+) The comparison is *experienced*, not recited: you know exactly what Chroma added (persistence, metadata filtering, upserts) and what it didn't (the math is the same cosine similarity you wrote yourself).
- (+) Teaches programming-to-an-interface — swapping implementations without touching callers — a general engineering lesson.
- (+) Interview line: "I built brute-force retrieval first; the vector DB bought me persistence and an ANN-ready interface, not magic."
- (−) One extra session. Accepted: the payoff is the strongest single talking point in the project.
