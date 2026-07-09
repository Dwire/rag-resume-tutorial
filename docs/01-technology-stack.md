# Technology Stack

Every technology in this project, what it does for us, why it beat the alternatives, and how likely it is to come up in a technical interview. Philosophy: **boring, well-documented, locally-runnable, explainable.**

## The stack at a glance

| Layer | Choice | Interview relevance |
|---|---|---|
| Language & runtime | Python 3.11+ | ⭐⭐⭐ (assumed baseline) |
| Environment | `venv` + `pip` + `requirements.txt` | ⭐ (hygiene, rarely quizzed) |
| Document loading | Python stdlib (`pathlib`) on `.md`/`.txt` files | ⭐⭐ |
| Chunking | **Hand-written** (~50 lines of our own code) | ⭐⭐⭐⭐⭐ |
| Embeddings | `sentence-transformers` (`all-MiniLM-L6-v2`, local) | ⭐⭐⭐⭐⭐ |
| Vector store | **Hand-written NumPy store first**, then ChromaDB behind the same interface | ⭐⭐⭐⭐⭐ |
| Testing | `pytest` (deterministic stages) + output validators (LLM stages) | ⭐⭐⭐ |
| Retrieval logic | **Hand-written** (query construction, top-k, thresholds) | ⭐⭐⭐⭐⭐ |
| Prompt assembly | **Hand-written** (grounded-generation prompt) | ⭐⭐⭐⭐⭐ |
| Generation LLM | Claude API — `claude-opus-4-8` via `anthropic` SDK | ⭐⭐⭐⭐ |
| Config / secrets | `python-dotenv` + `.env` | ⭐ |
| UI (first) | CLI (`argparse`) | ⭐ |
| UI (later) | Streamlit | ⭐⭐ |
| Orchestration framework | **None — deliberately** | ⭐⭐⭐⭐⭐ (the *decision* is interview gold) |

## Details

### Python 3.11+
- **What it is:** The language. The entire ML/RAG ecosystem is Python-first.
- **Role:** Everything.
- **Why over alternatives (TypeScript/Node):** Every RAG library, tutorial, and interview discussion assumes Python. `sentence-transformers` and ChromaDB are Python-native.
- **Interviews:** Assumed. You won't be asked "why Python," but you'll write/read it.

### venv + pip + requirements.txt
- **What it is:** Python's built-in virtual environment tool plus the standard package installer.
- **Role:** Isolates our dependencies from the system.
- **Why over alternatives (poetry, uv, conda):** Zero extra tooling to learn; every tutorial and workplace understands it. `uv` is faster but adds a tool to explain; conda is heavyweight.
- **Interviews:** Low. Basic hygiene.

### Python stdlib for document loading
- **What it is:** `pathlib.Path.glob()` + `read_text()` to walk a folder of Markdown/plain-text documents.
- **Role:** The "ingestion" stage. Your personal docs go in a folder; we read them into memory with metadata (filename, section).
- **Why over alternatives (LangChain loaders, `unstructured`, PDF parsers):** Your source documents are things you write yourself, so we control the format — Markdown. Parsing PDFs/Word is plumbing, not a RAG concept; we can add `pypdf` later if ever needed.
- **Interviews:** Medium — "how do you handle messy source data" comes up; we'll discuss it even though we sidestep it.

### Hand-written chunking
- **What it is:** ~50 lines of our own Python that splits documents into overlapping chunks (paragraph/heading-aware, with a size cap and overlap).
- **Role:** Turns whole documents into retrieval-sized pieces. **The single highest-leverage design decision in any RAG system.**
- **Why hand-written:** Chunking strategy (size, overlap, semantic boundaries) is the #1 RAG interview topic. Calling `RecursiveCharacterTextSplitter(...)` teaches you nothing; writing it teaches you everything.
- **Interviews:** Very high. "How did you chunk your documents and why?" is nearly guaranteed.

### sentence-transformers (`all-MiniLM-L6-v2`)
- **What it is:** An open-source Python library (built on PyTorch + Hugging Face) that runs embedding models locally. `all-MiniLM-L6-v2` is the classic starter model: 384-dimensional vectors, ~80 MB, fast on CPU.
- **Role:** The embedding stage — converts each chunk (and each job-posting query) into a vector whose *direction* encodes meaning, so "led a migration to Kubernetes" lands near "container orchestration experience."
- **Why over alternatives:**
  - **Anthropic has no embeddings endpoint** (they point users to third parties like Voyage AI) — so "just use the same vendor" isn't an option, which is itself a good interview fact.
  - **OpenAI/Voyage embedding APIs:** work fine, but cost money per call, need a network, and hide the mechanics. Local = free, offline, re-indexable at will.
  - Trade-off we accept: MiniLM is weaker than large API embedding models. Fine at our scale (dozens of docs), and *knowing* the trade-off is the point.
- **Interviews:** Very high. Embeddings, dimensionality, and cosine similarity are core RAG questions.

### Vector store: hand-written NumPy first, then ChromaDB (ADR-0005)
- **What it is:** We define a tiny `VectorStore` interface (`add`, `search`) and implement it **twice**: first as NumPy brute force (linear scan + cosine similarity, Steps 4–5), then as ChromaDB — an open-source vector database that runs embedded in your Python process (like SQLite for vectors) and persists to a local folder (Step 6).
- **Role:** Stores (vector, chunk-text, metadata) triples and answers "give me the k nearest chunks to this query vector."
- **Why build it twice:** After running real retrieval on your own brute-force store, the swap makes it *felt* what a vector DB actually adds (persistence, metadata filtering, upserts) and what it doesn't (the math is the same cosine you wrote yourself). Also teaches programming-to-an-interface — swapping implementations without touching callers.
- **Why Chroma over other real stores:**
  - **FAISS:** a library, not a database — blazing fast but no persistence/metadata story out of the box.
  - **pgvector:** the right production answer if you already run Postgres, but stands up a whole DB server for a learning project.
  - **Pinecone/Weaviate cloud:** managed services; nothing to learn locally, adds accounts and network.
- **Interviews:** Very high. "What does a vector DB actually do? What would you use in production?" — answered from having built both halves.

### pytest
- **What it is:** Python's standard testing framework.
- **Role:** Unit tests for every deterministic stage (chunker boundary cases, retriever result counts, word-count validators). The LLM stage is non-deterministic, so it gets *output validators and evals* instead — a distinction interviewers probe.
- **Why over alternatives (unittest):** less boilerplate, industry default.
- **Interviews:** Medium — "how did you test a pipeline with a non-deterministic component?" has a crisp answer: deterministic stages get unit tests; LLM outputs get validators and the Step 10 eval harness.

### Claude API (`claude-opus-4-8`, `anthropic` Python SDK)
- **What it is:** Anthropic's hosted LLM, called via the official Python SDK.
- **Role:** The generation stage — takes our assembled prompt (job posting + retrieved chunks + strict grounding rules) and writes the resume/cover letter.
- **Why over alternatives:**
  - **Fully-local LLM (Ollama + Llama/Mistral):** genuinely local, but small local models follow grounding constraints ("use ONLY these facts, ≤300 words, one page") much less reliably — and output quality directly affects your job applications. The *retrieval* half stays fully local either way.
  - **Which Claude model:** `claude-opus-4-8` is the current recommended default. Each generation call is a few cents; we make a handful per session, so quality beats penny-pinching. Easy to swap the model string later.
- **Interviews:** High. "How did you prevent hallucination?" is answered in our prompt design, not the model choice — but knowing model trade-offs (cost/latency/capability tiers) is asked too.

### python-dotenv
- **What it is:** Loads `ANTHROPIC_API_KEY` from a `.env` file (gitignored).
- **Role:** Keeps secrets out of code.
- **Interviews:** Low, but leaking a key in a repo is an instant red flag — so it matters.

### CLI first, Streamlit later
- **What it is:** We drive everything from the command line until the pipeline works end-to-end, then add Streamlit — a Python library that turns a script into a local web UI with ~30 lines.
- **Why:** A UI adds zero RAG understanding; doing it last keeps every earlier step focused. Streamlit over Flask/FastAPI because there's no web-framework ceremony to learn.
- **Interviews:** Low. Nice for demos.

### The dog that didn't bark: no LangChain / LlamaIndex
- **What they are:** Orchestration frameworks that pre-package loaders, splitters, retrievers, and chains.
- **Why not:** For a first RAG system they invert the learning: you configure components instead of understanding them, and interviews punish that ("so what does the retriever actually do?" … ). Our whole pipeline is ~300 lines of readable Python. **Interview-ready framing:** "I built it without a framework so I understood every stage; I'd reach for LangChain in production for its integrations and observability once the concepts are solid."
- **Interviews:** The decision itself is one of the strongest talking points this project produces. (Full rationale: ADR-0001.)
