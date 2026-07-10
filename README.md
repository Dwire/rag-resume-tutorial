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
   `documents/` (see `documents/README.md`). Record progress in `LEARNING_LOG.md`,
   write an ADR in `docs/decisions/` for each significant choice, and let each
   step's Key Takeaways + interview Q&A collect in `interview-prep/` — that bank
   compiles into a single `INTERVIEW_CHEATSHEET.md` at the final step.

### Optional: let an AI agent tutor you (Highly Encouraged)

This repo ships a teaching contract that turns an AI coding agent into a tutor
that explains before it builds, checkpoints after each step, and quizzes you for
interviews.

**Getting started is just this:**

1. Clone your copy of the repo and open it in your AI coding agent.
2. Say something like **"Let's start the course — begin Step 1."**
3. That's it. The agent picks up the teaching contract and drives the rest: it
   explains what you're about to build and why, waits for your go-ahead, builds
   the step, then stops for a review — walking the code, key takeaways, and a
   short interview quiz — before moving on. **One step per session; it won't run
   ahead.** You just keep saying "next step" when you're ready.

Which file the agent reads:

- **Claude Code** → reads `CLAUDE.md` automatically. Just open the repo and ask.
- **Codex / other agents** → point them at `AGENTS.md` (which defers to `CLAUDE.md`).

It's entirely optional — the course stands on its own if you'd rather work solo.

## Using a different LLM (or no key yet)

The API key powers **only the generation steps (8–9)**. Everything else is
key-free:

- **Embeddings run locally** via `sentence-transformers` — no API, no cost. You
  can complete the entire RAG core (Steps 1–7) with no key at all.
- **Generation is provider-swappable.** Retrieval hands a finished prompt to
  `src/generator.py`; that one file is the only place the LLM provider lives.
  Swap the Claude call for OpenAI, a local model via Ollama, or anything else by
  editing it — the rest of the pipeline doesn't change. The course uses Claude by
  default for output quality (see `docs/decisions/`).

## Why RAG for such a small dataset?

**RAG is not what you'd normally reach for at this scale.** A single 
person's experience write-ups are a handful of short documents
that fit comfortably inside a modern model's context window — so in production
you'd most likely **skip retrieval entirely and just put all the documents
straight into the prompt** (full-context "stuffing"), letting the model read
everything at once. Retrieval only starts earning its keep when the corpus grows
past what you can afford to send every call — more documents than fit the window,
or where sending them all gets slow or expensive.

So why build RAG here? **Because the goal of this course is understanding, not the
app.** It's designed to make you fluent enough in RAG — chunking, embeddings,
vector similarity, retrieval strategy, grounding, evaluation — to discuss it
confidently in an interview. Building a small, honest system around a real,
useful task (tailoring your own job-application materials) gives you something
concrete to reason about and demo, without a corpus so large it becomes a
distraction. Knowing *when RAG is and isn't the right tool* — and being able to
say "for a dataset this small I'd just stuff the context; RAG matters once you
outgrow the window" — is itself one of the sharper answers you can give.

## Not enough reference documents?

The pipeline is only as good as what's in `documents/`. If you don't have enough
written up, ask an AI assistant to comb through your personal or professional
projects on GitHub and pull your contributions into reference documents you can
drop in. Point it at your repos and use the prompt below as a starting point.

<details>
<summary><strong>Example prompt — have AI turn your GitHub history into reference documents</strong></summary>

```text
You are reviewing GitHub PRs and commits authored by Riley Doe (GitHub: RiDoe).

Scope:
- Analyze merged PRs and commits only
- Focus on production-impacting work (features, architecture, performance, reliability, DX)
- Ignore trivial changes (formatting, renames, dependency bumps unless meaningful)

Output FIVE sections in this exact order:

1. TECHNOLOGY LIST
   - languages
   - frameworks
   - tools
   - packages
   - Only list technologies that I used and implemented. Do NOT include additional dependencies.

2. DETAILED CONTRIBUTIONS
   - List major contributions grouped by feature or system
   - For each item include:
     - PR or commit ID
     - Repo name
     - What was built or changed
     - Why it mattered (problem solved)
     - Any technical complexity (state mgmt, async flows, infra, migrations, etc.)

3. EXECUTIVE BULLET SUMMARY
   - 6–10 concise bullets summarizing overall contribution themes
   - No PR numbers
   - Focus on outcomes and ownership

4. INTERVIEW-READY NARRATIVE
   - One tight paragraph
   - Written in first person
   - Emphasize decision-making, tradeoffs, and impact
   - Assume interviewer cannot see the code

5. RESUME-READY BULLET POINTS (MOST IMPORTANT)
   - Produce 6–10 bullets suitable for a mid → senior software engineer resume
   - Each bullet MUST:
     - Start with a strong action verb
     - Describe a concrete technical contribution
     - Include scope, scale, or impact where inferable (users, performance, reliability, dev velocity)
     - Be ≤ 2 lines each
     - Avoid internal jargon, repo names, or PR numbers
   - Prioritize:
     - Architecture & system design
     - Production features shipped
     - Performance/reliability improvements
     - Cross-cutting work reused across features
     - Leadership or ownership signals (even without direct reports)

If impact metrics are not explicit in the code, infer conservatively and state impact
qualitatively (e.g., "improved reliability," "reduced complexity," "unblocked feature
development").

Do NOT:
- Invent metrics
- Use filler language
- Repeat the same contribution across bullets
```

</details>

> **Note:** the example prompt names a specific GitHub user — swap in your own
> name and username before running it.

## Improving the course

Found a bug in the syllabus or docs? Open an issue or PR against the **template**
repo. If you're working in your private copy, fix it there and copy the change
back up to the template — there's no automatic sync.
## License

MIT — see `LICENSE`.
