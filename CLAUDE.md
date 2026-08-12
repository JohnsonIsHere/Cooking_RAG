# CLAUDE.md — Taiwanese Cooking RAG

Persistent project context. Keep lean (<200 lines). Update after each major architecture decision.
Session-to-session state lives in `PROGRESS.md`; task tracking lives in Notion — not here.

## Commands
> Fill in exact commands as they solidify. These are the highest-value lines in this file.

```bash
# run the app (local)
streamlit run app.py            # (placeholder — update once app.py exists)

# rebuild the vector store from the corpus
python scripts/build_index.py   # (placeholder)

# run the eval harness
python scripts/run_evals.py     # (placeholder — added in M2)

# lint + format
ruff check . && ruff format .

# tests
pytest -q
```

## What this is
A retrieval-augmented (RAG) question-answering system over **Taiwanese home cooking**.
It answers technique, substitution, and recipe questions grounded in a curated corpus.

Portfolio project targeting **AI Engineer** roles. The differentiator is not the demo —
it's the **evaluation harness** (the builder has 15 yrs home-cooking experience and can judge
answers cold). Values honest evaluation over impressive-looking output.

## Architecture (current — M1)
Two knowledge layers, retrieved together ("retrieve-both"), then answered by an LLM:

1. **Structured recipes** — records: name, ingredients, steps, ratios.
2. **Technique / substitution notes** — prose: soy types, wine subs, failure modes, why-things-break.

Flow: `query → embed → retrieve top-k from both layers → assemble prompt → LLM answer (+ show sources)`.

- **Scope now:** Taiwanese cuisine only (narrow first, for sharp evaluation).
- **Vector store:** local (Chroma/FAISS) — decision pending, document once chosen.
- **LLM + embeddings:** API-based — document exact models once chosen.

## Conventions
- Python, `venv`, dependencies pinned in `requirements.txt`.
- Keep retrieval logic importable and testable — **UI imports the core; core never imports Streamlit.**
- Every retrieval/answer path must be runnable headless (no UI) so it can be tested and eval'd.
- Corpus lives as files in `data/` (recipes + notes); index is rebuildable from source, never hand-edited.
- Small, named functions over monoliths. Prefer clarity over cleverness.
- No secrets in the repo; API keys via env vars / `.env` (gitignored).

## Eval discipline (the differentiator)
- Eval set = question + expected-answer pairs in `evals/eval_set.json`.
- Judge answers honestly; record failures rather than hiding them.
- When retrieval or prompt changes, re-run evals and compare — measure, diagnose, improve.
- Never tune against the eval set in a way that would inflate results; keep it a fair test.

## Roadmap (not yet built — do not write conventions for these until they exist)
- **M1 (now):** v0 shipped ugly — retrieve-both, Taiwanese corpus, deployed, first 10 evals.
- **M2:** eval harness — 50+ eval questions, LLM-as-judge / metric scorer, regression tests.
- **M3:** MLOps wrapper — Docker, CI/CD (GitHub Actions), monitoring (latency/cost/failures), versioning.
- **M4:** agentic upgrade — query routing between layers, driven by eval failures.
- **M5–M6:** write-ups, polish, applications.

## Working style
- Be honest, not flattering. Flag over-engineering and scope creep.
- Ship over polish: a deployed ugly v0 beats a perfect local prototype.
- Depth over breadth — one strong artifact, not many toy demos.
