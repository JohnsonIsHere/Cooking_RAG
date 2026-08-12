# PROGRESS.md — Taiwanese Cooking RAG

Session-to-session state. Update at the **end of each session**. Reload this after `/clear`.
Durable project facts live in `CLAUDE.md`; task board lives in Notion.

---

## Current milestone
**M1 — v0 shipped ugly** (deployed, working, first 10 evals).
Definition of done: a public URL that answers Taiwanese cooking questions from the corpus,
plus `evals/eval_set.json` with 10 question/expected-answer pairs committed.

## Status (update each session)
- **Last session did:** repo initialized (git, `.gitignore`), venv created, `requirements.txt`
  pinned, `.env.example` + `pyproject.toml` + README stub written, stack + chunking strategy
  decided (below), `data/recipes/` and `data/notes/` scaffolded.
- **Next up:** collect ~15–20 Taiwanese recipes (card 3) and 10–15 technique/substitution notes
  (card 4) — the one true hard blocker before chunk+embed (card 5) can start.
- **Currently blocked on:** _(none — content authoring is on the user, not a technical blocker)_
- **Latest card report:** [`docs/cards/01-repo-python-env-setup.md`](docs/cards/01-repo-python-env-setup.md) — card 1 done locally; GitHub push pending user confirmation.

## Open decisions (resolve and move to CLAUDE.md once settled)
- [x] Vector store: **FAISS**, two flat in-memory indices (recipes + notes), rebuilt from
      `data/` on every run. No persistence needed at this corpus size; sidesteps deploy
      filesystem concerns entirely.
- [x] Embedding model: **`sentence-transformers` / `all-MiniLM-L6-v2`**, local, free, no API key.
- [x] LLM API + model: **Gemini Flash (free tier)** via the `google-genai` SDK (not the
      deprecated `google-generativeai` package). Claude Haiku noted as a cheap fallback if
      free-tier rate limits become a problem — not built now.
- [x] Chunking strategy: **one chunk per recipe, one chunk per note/topic**, no overlap by
      default (sub-split + ~1–2 sentence overlap only if a single file covers multiple
      sub-topics or an unusually long multi-component recipe). No chunking library needed.
- [x] Deploy target: **Streamlit Community Cloud** (free hosting) — confirmed over a
      local-only/clone-and-run alternative, since a live public URL is worth more for a
      portfolio project than requiring reviewers to set up a Python env.

## Decisions made (log as you go)
- 2026-08-12 — Full M1 stack + sequencing plan agreed (see decisions above). Model/provider
  choice deliberately de-emphasized and kept behind thin wrapper modules
  (`core/embeddings.py`, `core/llm.py`) since the project's differentiator is the eval
  harness, not which LLM answers the query.

## Notes / scratch
- Narrow to Taiwanese first for sharp evaluation; broaden later.
- Retrieve-both for v0; query routing is an M4 upgrade, not now.
- Keep core importable/testable; UI is a thin layer on top.

---

### Session-end checklist
- [ ] Updated "Last session did" + "Next up"
- [ ] Logged any decisions made
- [ ] Committed + pushed
- [ ] Notion board reflects reality
