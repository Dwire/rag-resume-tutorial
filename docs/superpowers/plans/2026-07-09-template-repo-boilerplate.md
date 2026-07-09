# Public Template Repo Boilerplate — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Convert this pre-code learning repo into a public GitHub *template repository* others can clone to learn RAG (interview-focused), while the student does the lessons in a separate private copy with no risk of leaking personal résumé documents.

**Architecture:** Genericize the existing foundation (teaching contract, syllabus, ADRs) into a self-guided course front-ended by a README; add privacy scaffolding (`.gitignore`, `.env.example`, `documents/` convention) so personal data cannot land in the public repo; add dual AI-tutor accelerators (`CLAUDE.md` as source of truth, `AGENTS.md` as a pointer). Deployment is a manual GitHub step performed by the student.

**Tech Stack:** Git, GitHub template repositories, Markdown. (The RAG stack itself — Python, chromadb, sentence-transformers, anthropic, pytest — is only *declared* here in `requirements.txt`; it gets built in later syllabus steps.)

## Global Constraints

- No reference solutions ship in the public repo — scaffolding only.
- Personal résumé documents must never be committable to the public repo (gitignore + folder convention enforce this).
- `CLAUDE.md` is the single source of truth for the teaching contract; `AGENTS.md` is a thin pointer to it.
- Course is tool-agnostic at its core; AI-tutor files are documented as an *optional* accelerator.
- Never add AI co-author trailers to commits.
- Commit after each task (frequent commits).
- Work happens on `main` in this directory (this directory becomes the public template).

---

## File Structure

| File | Responsibility |
|------|----------------|
| `.gitignore` | Exclude personal docs, secrets, vector store, Python cruft. Privacy foundation. |
| `.env.example` | Document the `ANTHROPIC_API_KEY` variable with no value. |
| `documents/README.md` | Explain the personal-documents convention. |
| `documents/sample-experience.md` | One fictional sample doc showing expected format. |
| `README.md` | Front door: course framing, prerequisites, syllabus overview, setup, how-to-use, backport note, license reference. |
| `LEARNING_LOG.md` | Genericized to an empty template (no Step 0 entry). |
| `CLAUDE.md` | Reframed: optional AI-tutor accelerator; source of truth for the teaching contract. |
| `AGENTS.md` | Thin pointer to `CLAUDE.md` for Codex-style agents. |
| `requirements.txt` | Declared Python deps for the eventual RAG build (starter set). |
| `LICENSE` | MIT. |

Tasks are ordered so the privacy foundation lands first, then the folder convention it protects, then learner-facing docs, then deployment.

---

### Task 1: Privacy foundation — `.gitignore` and `.env.example`

**Files:**
- Create: `.gitignore`
- Create: `.env.example`

**Interfaces:**
- Produces: a git-ignore ruleset that keeps `documents/` private except the two committed sample files; later tasks (Task 2) create those two files and rely on them NOT being ignored.

- [ ] **Step 1: Create `.gitignore`**

```gitignore
# --- Personal data: never commit real résumé / experience write-ups ---
# Ignore everything under documents/ EXCEPT the committed sample + its README.
documents/*
!documents/README.md
!documents/sample-experience.md

# --- Secrets ---
.env

# --- Vector store (persistent Chroma data, built in later steps) ---
.chroma/
chroma/

# --- Python ---
__pycache__/
*.py[cod]
.pytest_cache/
.venv/
venv/
env/
*.egg-info/

# --- OS / editor cruft ---
.DS_Store
.idea/
.vscode/
```

- [ ] **Step 2: Create `.env.example`**

```dotenv
# Copy this file to .env and fill in your key. .env is git-ignored.
# Get a key at https://console.anthropic.com/
ANTHROPIC_API_KEY=
```

- [ ] **Step 3: Verify the ignore rules behave (this is the test)**

The privacy guarantee is the whole point, so prove it before trusting it. These
commands create throwaway paths and check git's decision, then clean up.

Run:
```bash
mkdir -p documents
printf 'secret real resume\n' > documents/my-real-resume.md
git check-ignore documents/my-real-resume.md   # expect: prints the path (IS ignored)
git check-ignore .env                           # expect: prints .env (IS ignored)
git check-ignore -v documents/README.md; echo "exit=$?"   # expect: exit=1 (NOT ignored)
rm documents/my-real-resume.md
```
Expected: the first two commands echo the path back (ignored); the `README.md`
check exits non-zero / prints nothing (not ignored). If `documents/README.md`
shows as ignored, the negation rules are wrong — fix before committing.

- [ ] **Step 4: Commit**

```bash
git add .gitignore .env.example
git commit -m "Add privacy foundation: gitignore for personal docs/secrets, .env.example"
```

---

### Task 2: Personal-documents convention — `documents/`

**Files:**
- Create: `documents/README.md`
- Create: `documents/sample-experience.md`

**Interfaces:**
- Consumes: the `.gitignore` negation rules from Task 1 (these two files are the exceptions that stay tracked).
- Produces: the `documents/` folder convention that later syllabus steps (document loading in Step 1) read from.

- [ ] **Step 1: Create `documents/README.md`**

```markdown
# Your documents

This folder is the RAG pipeline's knowledge base. Drop in write-ups of your own
past work, side projects, and skills — one topic per file, plain Markdown or
text. The retriever indexes these and pulls the most relevant pieces for a given
job posting. **Nothing you write is fabricated by the app; it can only surface
what's actually here.**

## Rules

- **Real files here are git-ignored on purpose.** Only `README.md` and
  `sample-experience.md` are tracked. Your personal documents never get
  committed — see the repo's `.gitignore`.
- **One subject per file** works best for retrieval (e.g. `payments-migration.md`,
  `open-source-cli.md`, `leadership.md`).
- **Write in specifics**: numbers, technologies, outcomes. Vague files retrieve
  poorly and generate weak output.

## Format

There's no required schema. A short title, then a few paragraphs or bullets of
concrete detail. See `sample-experience.md` for an example you can delete.
```

- [ ] **Step 2: Create `documents/sample-experience.md`** (fictional — a learner deletes it)

```markdown
# Payments platform migration (sample — safe to delete)

> This is a fictional example so you can see the expected format. Replace the
> `documents/` folder with your own write-ups.

At Acme Corp (2021–2023) I led the migration of the checkout service from a
monolith to three independently deployable services. Cut p95 checkout latency
from 820ms to 240ms and reduced payment-failure retries by 38%.

- Designed the idempotency-key scheme that eliminated duplicate charges during
  network retries (~1,200 avoided per month).
- Introduced contract tests between the cart and payments services, dropping
  cross-team integration bugs to near zero over two quarters.
- Tech: Python, PostgreSQL, Stripe API, Kafka, Docker, GitHub Actions.

## Skills demonstrated
Service decomposition, idempotency and exactly-once semantics, performance
profiling, cross-team API contracts, incremental migration under live traffic.
```

- [ ] **Step 3: Verify both files are tracked despite the ignore rule**

Run:
```bash
git check-ignore -v documents/README.md documents/sample-experience.md; echo "exit=$?"
```
Expected: `exit=1` and no output — neither file is ignored.

- [ ] **Step 4: Commit**

```bash
git add documents/README.md documents/sample-experience.md
git commit -m "Add documents/ convention with README and a fictional sample doc"
```

---

### Task 3: Front door — `README.md`

**Files:**
- Create: `README.md`

**Interfaces:**
- Consumes: references the syllabus (`docs/03-learning-syllabus.md`), the tutor files (`CLAUDE.md`/`AGENTS.md`), `documents/`, and `.env.example` — all created in other tasks. Names must match exactly.

- [ ] **Step 1: Create `README.md`**

```markdown
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
```

- [ ] **Step 2: Verify cross-references resolve**

Run:
```bash
for f in docs/03-learning-syllabus.md docs/01-technology-stack.md docs/02-architecture.md .env.example documents/README.md; do
  test -e "$f" && echo "ok: $f" || echo "MISSING: $f"
done
```
Expected: all `ok:` (note `CLAUDE.md`, `AGENTS.md`, `LICENSE` are created in later tasks — that's fine; re-check at the end).

- [ ] **Step 3: Commit**

```bash
git add README.md
git commit -m "Add README front door: course framing, setup, how-to-use"
```

---

### Task 4: Genericize `LEARNING_LOG.md` to an empty template

**Files:**
- Modify: `LEARNING_LOG.md` (currently contains this student's Step 0 entry — replace with a clean template)

**Interfaces:**
- Produces: an empty log template a fresh learner fills in. The student's real Step 0 entry is preserved only in their private copy / git history, not shipped publicly.

- [ ] **Step 1: Replace the entire contents of `LEARNING_LOG.md`**

```markdown
# Learning Log

> Update this file at the end of EVERY step. Newest entry at the top.
> Each entry: step number/name, date, what was built, key decisions + rationale,
> concepts covered, quiz results & struggles (revisit later), next step.
>
> If you use the AI-tutor accelerator (see `CLAUDE.md`), it maintains this for
> you. Working solo? Keep it yourself — it's your interview revision trail.

## Current status

- **Current step:** not started — begin with Step 1 in `docs/03-learning-syllabus.md`
- **Concepts to revisit:** none yet

---

<!--
Template for each entry (copy above the previous one):

## Step N — <name> (YYYY-MM-DD)

**What was built:**

**Key decisions (link ADRs in docs/decisions/):**

**Concepts covered:**

**Quiz results / struggles:**

**Next step:**
-->
```

- [ ] **Step 2: Verify the student's private Step 0 entry is not lost**

The genericized template drops the personal Step 0 entry on purpose. It survives
in git history and in the student's private copy. Confirm history still has it:

Run:
```bash
git log --oneline -- LEARNING_LOG.md | head -3
git show HEAD:LEARNING_LOG.md | grep -c "Step 0" || echo "no Step 0 in current HEAD (expected after edit)"
```
Expected: history exists; after this edit the working file has no "Step 0" entry.

- [ ] **Step 3: Commit**

```bash
git add LEARNING_LOG.md
git commit -m "Genericize LEARNING_LOG to an empty template for new learners"
```

---

### Task 5: Reframe `CLAUDE.md` as an optional accelerator

**Files:**
- Modify: `CLAUDE.md` (prepend an "About this file" banner; generalize the session-start protocol so it survives a fresh clone with an empty log)

**Interfaces:**
- Produces: the single source of truth for the teaching contract that `AGENTS.md` (Task 6) points to. The banner's framing ("optional accelerator") must match how `README.md` (Task 3) describes it.

- [ ] **Step 1: Prepend this banner as the very top of `CLAUDE.md`** (above the current `# RAG Resume Tailor — Learning Project` line)

```markdown
# 🤖 Optional AI-Tutor Accelerator

> **This file is optional.** The course in `docs/` stands on its own — you can
> work it solo or with any assistant. This file is a *teaching contract*: drop it
> in and an AI coding agent (Claude Code reads it automatically) will tutor you
> instead of just writing code — explaining before building, checkpointing after
> each step, and quizzing you for interviews. Using Codex or another agent? See
> `AGENTS.md`, which points here.
>
> If you'd rather not use an AI tutor, ignore this file entirely.

---

```

- [ ] **Step 2: Make the session-start protocol robust to a fresh clone**

In the "On every session start" list, the current step recap assumes a populated
log. Replace step 3 of that list:

Find:
```markdown
3. Briefly recap the last checkpoint (what was built, key concepts, anything the student struggled with).
```
Replace with:
```markdown
3. Briefly recap the last checkpoint from the log (what was built, key concepts, anything the student struggled with). If the log is still the empty template (a fresh clone), say so and treat Step 1 as the starting point.
```

- [ ] **Step 3: Verify the edits landed and the contract body is intact**

Run:
```bash
head -12 CLAUDE.md                       # expect the accelerator banner
grep -c "Teaching contract" CLAUDE.md    # expect 1 — body preserved
grep -c "fresh clone" CLAUDE.md          # expect 1 — session-start generalized
```
Expected: banner present, both greps ≥1.

- [ ] **Step 4: Commit**

```bash
git add CLAUDE.md
git commit -m "Reframe CLAUDE.md as an optional AI-tutor accelerator"
```

---

### Task 6: Add `AGENTS.md` pointer

**Files:**
- Create: `AGENTS.md`

**Interfaces:**
- Consumes: the teaching contract in `CLAUDE.md` (Task 5) — must not duplicate it.

- [ ] **Step 1: Create `AGENTS.md`**

```markdown
# AGENTS.md — for Codex and other AI coding agents

This project ships a teaching contract that turns you into a **tutor**, not just a
code generator. The full contract lives in **`CLAUDE.md`** — read it now and
follow it exactly. It is the single source of truth; this file only points to it
so it never drifts.

**In short (but read `CLAUDE.md` for the binding version):**

- This is a **learning project**. The deliverable is the learner's understanding,
  not the app. Optimize for interview readiness.
- Act as **tutor → architect → pair programmer**, in that priority order.
- **One syllabus step per session** (`docs/03-learning-syllabus.md`). Never build
  ahead.
- **Explain before you build; checkpoint, summarize, and quiz after you build.**
- **Never fabricate résumé content** — outputs are grounded only in the learner's
  `documents/`.

If anything here seems to conflict with `CLAUDE.md`, `CLAUDE.md` wins.
```

- [ ] **Step 2: Verify it points at CLAUDE.md and stays thin**

Run:
```bash
grep -c "CLAUDE.md" AGENTS.md            # expect ≥2
wc -l AGENTS.md                          # expect small (≈15-20 lines) — it's a pointer, not a copy
```
Expected: references present; file is short.

- [ ] **Step 3: Commit**

```bash
git add AGENTS.md
git commit -m "Add AGENTS.md pointing Codex-style agents to the CLAUDE.md contract"
```

---

### Task 7: Declared dependencies and license — `requirements.txt`, `LICENSE`

**Files:**
- Create: `requirements.txt`
- Create: `LICENSE`

**Interfaces:**
- Produces: a starter dependency list later syllabus steps pin exactly; a license `README.md` references.

- [ ] **Step 1: Create `requirements.txt`** (starter floors; Step 1 of the syllabus finalizes exact pins)

```text
# Core RAG stack (installed/pinned as the syllabus reaches each step)
anthropic>=0.40
chromadb>=0.5
sentence-transformers>=3.0
numpy>=1.26

# Testing
pytest>=8.0

# Config
python-dotenv>=1.0
```

- [ ] **Step 2: Create `LICENSE`** (MIT)

```text
MIT License

Copyright (c) 2026 Gregory Dwyer

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

- [ ] **Step 3: Commit**

```bash
git add requirements.txt LICENSE
git commit -m "Add starter requirements.txt and MIT LICENSE"
```

---

### Task 8: Final self-check + GitHub deployment (manual, student-performed)

**Files:** none (verification + GitHub UI actions)

**Interfaces:**
- Consumes: everything from Tasks 1–7.

- [ ] **Step 1: Full cross-reference and privacy re-check**

Run:
```bash
# All referenced files now exist
for f in README.md CLAUDE.md AGENTS.md LICENSE requirements.txt .env.example \
         LEARNING_LOG.md documents/README.md documents/sample-experience.md \
         docs/01-technology-stack.md docs/02-architecture.md docs/03-learning-syllabus.md; do
  test -e "$f" && echo "ok: $f" || echo "MISSING: $f"
done
# Privacy still holds
printf 'x\n' > documents/leaktest.md
git check-ignore documents/leaktest.md && echo "GOOD: real docs ignored" || echo "BAD: not ignored"
rm documents/leaktest.md
# Working tree clean
git status --short
```
Expected: all `ok:`, "GOOD: real docs ignored", and a clean `git status`.

- [ ] **Step 2: Push to GitHub as a public repo** (student runs; requires the `gh` CLI or the GitHub web UI)

```bash
# Option A — gh CLI:
gh repo create rag-resume-tutorial --public --source=. --remote=origin --push
```
Or, web UI: create an empty public repo, then
`git remote add origin <url> && git push -u origin main`.

- [ ] **Step 3: Mark it a template repository**

GitHub → the repo → **Settings** → tick **Template repository**. (No CLI toggle
is guaranteed across `gh` versions; use the web UI.)

- [ ] **Step 4: Generate your private working copy**

GitHub → the template repo → **Use this template → Create a new repository** →
name it (e.g. `rag-resume-mine`) → **Private** → Create. Then:
```bash
git clone <private-repo-url> ~/path/to/rag-resume-mine
cd ~/path/to/rag-resume-mine
cp .env.example .env   # add your ANTHROPIC_API_KEY
```
Do Step 1 of the syllabus in *that* repo. This directory stays as the clean
public template.

- [ ] **Step 5: (Optional) Restore your personal LEARNING_LOG in the private copy**

Your Step 0 entry still exists in this template's git history. If you want it in
your private working copy, copy it over manually from
`git show <commit>:LEARNING_LOG.md` in this repo, or just start fresh from Step 1.

---

## Self-Review

**Spec coverage:**
- Repo topology (public template + private copy) → Task 8.
- README front door → Task 3.
- `LEARNING_LOG` genericized → Task 4.
- `documents/` convention + sample → Task 2.
- `CLAUDE.md` reframed / `AGENTS.md` pointer → Tasks 5, 6.
- `.gitignore` / `.env.example` / privacy safeguards → Task 1 (verified again in Task 8).
- `requirements.txt` / `LICENSE` → Task 7.
- Deployment flow → Task 8.
- Backport note → in README (Task 3).
- "Out of scope" items (no solutions, no sync tooling, no CI) → honored; none added.

**Placeholder scan:** No TBD/TODO. Every file's full content is inline. `requirements.txt` is intentionally a starter set (spec says exact pins are finalized in syllabus Step 1) — this is stated, not a placeholder.

**Type/name consistency:** File names referenced across tasks (`CLAUDE.md`, `AGENTS.md`, `documents/README.md`, `documents/sample-experience.md`, `.env.example`, `LICENSE`, `requirements.txt`, the three `docs/` files) match the manifest and each other. Repo names (`rag-resume-tutorial` public, `rag-resume-mine` private) are consistent between README and Task 8.
```
