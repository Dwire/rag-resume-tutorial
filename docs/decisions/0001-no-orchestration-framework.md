# ADR-0001: Build the RAG pipeline by hand — no LangChain/LlamaIndex

**Date:** 2026-07-09 · **Status:** Accepted

## Context
This is a first-ever RAG project whose primary deliverable is the builder's understanding, aimed at technical interviews. Orchestration frameworks (LangChain, LlamaIndex) pre-package loaders, splitters, retrievers, and chains.

## Decision
Write the pipeline as plain Python (~300 lines total): our own loader, chunker, retriever, and prompt assembly. Use libraries only where hand-rolling teaches nothing (the embedding model itself, the vector DB, the LLM API).

## Alternatives considered
- **LangChain:** huge ecosystem, but you configure components instead of understanding them; heavy abstraction churn; interviews punish "the framework did it."
- **LlamaIndex:** best-in-class ingestion/index abstractions, same learning-inversion problem.

## Consequences
- (+) Every stage is explainable line by line — the whole point.
- (+) No framework API churn; fewer dependencies.
- (−) We rebuild small conveniences (splitter, retriever glue) ourselves — acceptable, that *is* the curriculum.
- Interview framing: "I built it framework-free to understand every stage; in production I'd consider LangChain for integrations and observability."
