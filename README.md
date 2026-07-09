# RAG Resume Tailor — a build-it-yourself RAG course

A hands-on, interview-focused course for building your first Retrieval-Augmented
Generation system. You build a small, local app that reads a folder of your own
experience write-ups, retrieves the pieces most relevant to a job posting, and
drafts a one-page resume or a ≤300-word cover letter **grounded only in what your
documents actually say — no fabricated experience, ever.**

The point isn't the app. **The point is that you can explain every moving part of
a RAG pipeline in a technical interview** — chunking, embeddings, vector
similarity, retrieval strategy, prompt construction, grounding, and the trade-offs
behind each.

## What you'll build

`ingest → chunk → embed → store → retrieve → prompt → generate`, hand-rolled (no
LangChain/LlamaIndex) so nothing is a black box. Highlights:

- Local embeddings with `sentence-transformers` (free, offline).
- A vector store built **twice** — first hand-rolled with NumPy, then swapped for
  ChromaDB behind a shared interface, with a side-by-side comparison.
- Generation with the Claude API, where the anti-hallucination work lives in
  prompt construction you own.

## Who it's for

Someone comfortable writing Python who has never built a RAG system and wants to
be able to talk about one credibly. No ML background required.

## Prerequisites

- Python 3.10+
- An Anthropic API key (for the generation steps) — https://console.anthropic.com/
- Comfort with a terminal and `git`

## How to use this

1. **Click "Use this template" → Create a new repository** (make it **Private** —
   it will hold your personal documents). Clone your copy.
2. Copy `.env.example` to `.env` and add your `ANTHROPIC_API_KEY`.
3. Read `docs/` in order:
   - `docs/01-technology-stack.md` — every tool and why it was chosen
   - `docs/02-architecture.md` — the system and the core RAG concepts
   - `docs/03-learning-syllabus.md` — the numbered steps you'll implement
4. Work the syllabus one step at a time. Put your own experience write-ups in
   `documents/` (see `documents/README.md`). Record progress in `LEARNING_LOG.md`
   and write an ADR in `docs/decisions/` for each significant choice — those are
   your interview study notes.

### Optional: let an AI agent tutor you

This repo ships a teaching contract that turns an AI coding agent into a tutor
that explains before it builds, checkpoints after each step, and quizzes you for
interviews. If you use one:

- **Claude Code** → it reads `CLAUDE.md` automatically.
- **Codex / other agents** → point them at `AGENTS.md` (which defers to `CLAUDE.md`).

It's entirely optional — the course stands on its own.

## Improving the course

Found a bug in the syllabus or docs? Open an issue or PR against the **template**
repo. If you're working in your private copy, fix it there and copy the change
back up to the template — there's no automatic sync.

## License

MIT — see `LICENSE`.
