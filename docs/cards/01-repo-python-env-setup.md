# Card 01 — Set up repo + Python env

Notion card: Cooking-RAG / M1 / Set up repo + Python env
Status: Done — 2026-08-12

## Problem & objective
- **Pain points:** No repo, no environment, no locked dependencies, and no documented stack
  existed yet. Every later card (corpus collection, embedding, retrieval) needs a stable,
  reproducible foundation to build on, and a fresh RAG stack has several fast-moving
  dependencies (embedding libs, vector stores, LLM SDKs) where "guess a version" silently
  breaks reproducibility.
- **Objective statement:** Stand up a working local Python environment and git repo, with the
  M1 stack decisions (vector store, embeddings, LLM) made and documented, so subsequent cards
  have zero setup ambiguity.

## Quantitative metrics
- `pip install -r requirements.txt` completes with **0 errors** on a clean venv (verified).
- All 4 core libraries (`faiss-cpu`, `sentence-transformers`, `google-genai`, `streamlit`)
  import successfully with **0 import errors** (verified via direct `python -c` check).
- `requirements.txt`: **7/7 top-level packages pinned** to exact resolved versions — 0 loose
  ranges.
- Clean-clone reproduction time: not yet measured — flagged as a future check before card 12
  finalizes the README (see Future/improvements).

## Qualitative benchmarks
- Stack rationale (FAISS + local `sentence-transformers` + Gemini Flash) is written as a clear,
  justified paragraph in `README.md` a technical reviewer could read cold without follow-up
  questions — this is the card 2 "one paragraph in README on why" requirement, pulled forward
  since the decision was made during this card.
- Self-verified via `git status` before commit that no secrets are staged (`.env` correctly
  ignored; `.env.example` documents the required var with no value).

## Acceptance criteria
- [x] Git repo initialized locally (`main` branch).
- [x] `.gitignore` excludes `venv/`, `.env`, caches, `.DS_Store`.
- [x] venv created; `requirements.txt` installs cleanly.
- [x] `requirements.txt` pinned to exact resolved versions.
- [x] README stub committed (what it is, architecture, stack + rationale, status, what's next).
- [x] Initial commit made locally.
- [x] **GitHub repo created and pushed publicly** — https://github.com/JohnsonIsHere/Cooking_RAG,
      pushed after explicit user confirmation. (Initially created under the wrong `gh`-authenticated
      account, `jtjohnsontw`; caught immediately, `gh auth login` re-run for the correct account
      `JohnsonIsHere`, and the repo transferred over via the GitHub API — see Blockers below.)

## Trade-offs
- **FAISS vs Chroma** (vector store): chose FAISS — smaller dependency footprint, and avoids a
  known `sqlite3`-version conflict Chroma hits on Streamlit Community Cloud. Con: loses
  Chroma's more ergonomic metadata-filtering API; acceptable at this corpus scale (a few dozen
  documents, two flat indices matching "retrieve-both" directly).
- **Local `sentence-transformers` vs an embeddings API** (e.g. OpenAI): chose local for
  genuine $0 cost and no second API key to manage. Con: pulls in `torch` (~111MB), a much
  heavier and slower install than an API call — acceptable since embedding only happens on
  index rebuild, not per query.
- **Gemini Flash (free tier) vs Claude Haiku/OpenAI** (LLM): chose Gemini to match the
  confirmed "free where reasonable" preference. Con: free-tier rate limits could throttle
  heavy eval-harness runs later; Claude Haiku is documented as a near-zero-cost fallback
  behind the same `core/llm.py` wrapper if that happens.
- **`google-genai` vs `google-generativeai`**: mid-setup, importing `google-generativeai`
  surfaced a `FutureWarning` — Google has ended all support for that package. Swapped to the
  current `google-genai` SDK before any pipeline code was written against it, avoiding
  near-term rework.
- **Pinned exact versions vs loose ranges**: pinned for reproducibility. Con: will drift from
  "latest" over time and need periodic manual bumps — acceptable trade for a portfolio project
  where a reviewer cloning the repo getting a broken install is worse than slightly stale deps.

## Non-goals
- No retrieval/embedding pipeline code yet (cards 5, 7) — this card is environment + decisions
  only.
- No corpus content (cards 3, 4) — `data/recipes/` and `data/notes/` are scaffolded empty
  (`.gitkeep` only); content is the user's own cooking knowledge, not something to fabricate.
- No deploy yet (card 11) — Streamlit Community Cloud confirmed as the target, but not
  executed.
- No final README polish (live link, etc.) — that's card 12, after deploy.

## Blockers & rabbit holes
- Initial `requirements.txt` used version numbers from stale training-data knowledge (e.g.
  `faiss-cpu==1.8.0.post1`), which no longer exist on PyPI as of 2026 — the current API
  landscape has moved well past what was assumed. Resolved by dropping pins, letting `pip`
  resolve latest-compatible versions on Python 3.13, then re-pinning to what actually
  installed. Lesson for future cards: resolve-then-pin, don't guess exact versions for
  fast-moving packages.
- `google-generativeai` deprecation warning caught immediately on first import test — swapped
  to `google-genai` (2.17.0) before it became load-bearing anywhere.
- GitHub push was intentionally held until explicit user confirmation, per the working
  agreement to confirm before public/visible actions — not a technical blocker.
- The only `gh`-authenticated account on the machine at push time was `jtjohnsontw`, which
  turned out not to be the user's regular account. Caught right after the first push. Fixed by
  logging into the correct account (`gh auth login`, one retry needed after a transient
  `connection reset by peer` on the OAuth token exchange), then transferring the repo via
  `gh api repos/jtjohnsontw/Cooking_RAG/transfer` (no `gh repo transfer` subcommand exists),
  and re-pointing the local `origin` remote. Full history preserved, no data lost.

## Future / improvements
- Measure actual clean-clone-to-running time and state it in the README once the full pipeline
  exists (card 12), for reviewer confidence.
- Once corpus size grows well past a few hundred docs (M4+), revisit Chroma or a hosted vector
  DB for metadata filtering — not a concern at current scale.
- If Gemini free-tier rate limits become a real problem during eval runs (M2), swap in the
  documented Claude Haiku fallback — should be an isolated change inside `core/llm.py` only.
