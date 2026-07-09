# ADR-0004: Claude API (claude-opus-4-8) for the generation stage

**Date:** 2026-07-09 · **Status:** Accepted

## Context
The final stage writes a one-page resume / ≤300-word cover letter grounded ONLY in retrieved chunks. Output quality directly affects real job applications, and constraint-following (no fabrication, hard length caps) is the critical capability.

## Decision
Call the Claude API with model `claude-opus-4-8` via the official `anthropic` Python SDK. Model string kept in config so it's swappable.

## Alternatives considered
- **Fully local LLM (Ollama + Llama/Mistral):** keeps everything offline, but small local models follow strict grounding/length constraints far less reliably — the one place quality really matters here.
- **Smaller/cheaper Claude tiers:** viable, but we make only a handful of calls per session (cents each); `claude-opus-4-8` is the current recommended default and constraint-following is the priority.

## Consequences
- (+) Best available constraint-following for the anti-hallucination prompt we design in Step 7.
- (+) Learning the raw Messages API (system prompts, max_tokens, streaming) is itself interview-relevant.
- (−) Requires an API key, network, and small per-call cost.
- Note: only *generation* leaves the machine — retrieval (embeddings + vector store) is fully local. Retrieved chunks are sent to the API as prompt context, which is inherent to RAG.
