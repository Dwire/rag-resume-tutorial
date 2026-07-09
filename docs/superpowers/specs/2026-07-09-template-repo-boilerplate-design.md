# Design: RAG course as a public template + private working copy

**Date:** 2026-07-09
**Status:** Approved (pending spec review)

## Problem

This repo currently holds the pre-code foundation of a learning project (teaching
contract, syllabus, architecture doc, ADRs). The student wants to:

1. Publish the scaffolding as a **public boilerplate resource** others can clone to
   learn RAG with an interview focus.
2. Do the lessons themselves in **private**, so their progress, learning-log entries,
   and — critically — their personal résumé documents never leak into the public
   resource.

The central risk is data leakage: personal experience write-ups used as RAG inputs
must never be committed to the public repo.

## Decisions (from brainstorming)

- **Audience:** tool-agnostic core (self-guided RAG course) with an AI-tutor
  accelerator documented as an optional bonus. Not Claude-Code-only.
- **Solutions:** scaffolding only — no reference solutions ship publicly. The
  student's private repo becomes the de-facto reference.
- **AI-tutor accelerator:** support both ecosystems — `CLAUDE.md` (Claude Code) and
  `AGENTS.md` (Codex) — sharing one teaching contract so they don't drift.
- **Hosting:** GitHub **template repository**, not a Gist (multi-folder project).
- **This directory becomes the template.** The student generates a fresh private copy
  from it via "Use this template" and does the lessons there.

## Repo topology

- **Public template repo** (e.g. `rag-resume-tutorial`): curriculum + scaffolding +
  stubs + tutor accelerators. No solutions, no personal data. Marked as a GitHub
  template repository.
- **Private working repo** (e.g. `rag-resume-mine`): generated via "Use this
  template", set to Private. Where Steps 1–12 actually happen. Fully isolated.
- **Backporting:** curriculum/doc fixes discovered mid-lesson are made in the public
  template and optionally pulled down. Manual backport (low frequency); documented in
  README.

## Boilerplate file manifest

| File | Action | Notes |
|------|--------|-------|
| `README.md` | **new** | Front door. Course framing, interview focus, prerequisites, 13-step syllabus overview, setup, "how to use this" (solo or with an AI tutor), backport note, license. |
| `docs/01-technology-stack.md` | keep | Curriculum backbone, unchanged. |
| `docs/02-architecture.md` | keep | Unchanged. |
| `docs/03-learning-syllabus.md` | keep | Unchanged. |
| `docs/decisions/0001–0005` | keep | Interview study notes, unchanged. |
| `LEARNING_LOG.md` | **genericize** | Ship as an empty template: headers + "newest entry on top" instructions, no Step 0 entry. Student's real log lives only in the private repo. |
| `documents/README.md` | **new** | Explains "put your personal write-ups here". |
| `documents/sample-experience.md` | **new** | One fictional sample doc so the folder structure and expected format are obvious. |
| `CLAUDE.md` | **rework** | Demoted from centerpiece to optional AI-tutor accelerator; session-start protocol reworded for a fresh learner, not just this student. Teaching-contract content preserved. |
| `AGENTS.md` | **new** | Codex equivalent of `CLAUDE.md`, same teaching contract. |
| `.gitignore` | **new** | See safeguards below. |
| `.env.example` | **new** | `ANTHROPIC_API_KEY=` with no value. |
| `requirements.txt` | **new** | Pinned deps for the stack (chromadb, sentence-transformers, anthropic, pytest, etc.). Exact pins finalized in Step 1; a starter set is fine now. |
| `LICENSE` | **new** | MIT. |

### Keeping CLAUDE.md and AGENTS.md in sync

The teaching contract is authored once. Options to avoid drift (decide in planning):
either (a) `AGENTS.md` is a thin pointer that says "read `CLAUDE.md` — the teaching
contract there applies to you too," or (b) both files embed the same contract body.
Preference: (a) — single source of truth, minimal duplication.

## Privacy & secrets safeguards

`.gitignore` (public template) excludes:

- `documents/*` **except** `documents/README.md` and `documents/sample-experience.md`
- `.env`
- `.chroma/` (persistent vector store)
- `__pycache__/`, `*.pyc`, `.pytest_cache/`
- common venv dirs (`.venv/`, `venv/`)

`.env.example` documents the API key variable with no real value. Personal résumé
material exists only in the private repo's `documents/`, which is git-ignored there
too, so even privately the student chooses what (if anything) to commit.

## Deployment flow (the "how do I do this" answer)

1. `git init` in this directory; build out the boilerplate files; commit.
2. Create the public GitHub repo, push, then enable Settings → Template repository.
3. "Use this template → Create a new repository", name it, set **Private** → working copy.
4. Clone the private repo; do Step 1 there. This directory stays as the clean template.

## What changes vs. the existing plan

- `CLAUDE.md`: reframed as optional accelerator; session-start protocol generalized.
  Teaching-contract substance preserved.
- Syllabus / architecture / ADRs: unchanged.
- The RAG design itself is unchanged — still Step 1 (project setup & document loading)
  next, same stack.

## Out of scope (YAGNI)

- Automated template↔private sync tooling (manual backport is enough).
- Reference solutions, per-step solution branches/folders.
- CI, GitHub Actions, badges — can be added later if the resource gains traction.
- URL-based job-posting ingestion (already a later syllabus item, unrelated here).

## Success criteria

- A stranger can land on the public README and understand what they'll build, the
  prerequisites, and how to start — with or without an AI tutor.
- "Use this template" produces a clean private starter with no personal data.
- It is structurally impossible to commit personal résumé docs to the public repo by
  accident (gitignore + folder convention).
- The student's own Step 1 work happens in the private repo, unaffected.
