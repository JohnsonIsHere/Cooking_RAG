# Taiwanese Cooking RAG

A retrieval-augmented question-answering system over Taiwanese home cooking — technique,
substitution, and recipe questions, grounded in a curated corpus. Portfolio project targeting
AI Engineer roles; the differentiator is the evaluation harness, not the demo.

## Architecture

Two knowledge layers, retrieved together ("retrieve-both"), then answered by an LLM:

1. **Structured recipes** — name, ingredients, steps, ratios.
2. **Technique / substitution notes** — prose on soy types, wine subs, failure modes.

Flow: `query → embed → retrieve top-k from both layers → assemble prompt → LLM answer (+ sources)`.

## Stack

- **Vector store:** FAISS, two flat in-memory indices (one per layer), rebuilt from `data/` on
  every run — no persistence needed at this corpus size, which also keeps deployment simple.
- **Embeddings:** `sentence-transformers` (`all-MiniLM-L6-v2`), local, no API key/cost.
- **LLM:** Gemini Flash (free tier) for answer generation.
- Model choice is deliberately not the differentiator here — both are behind thin wrapper
  modules (`core/embeddings.py`, `core/llm.py`) so they're swappable without touching the rest
  of the pipeline.
- Full trade-off comparison against alternatives (other embedding models, vector stores, LLM
  APIs) and a curated reading list: [`docs/cards/02-choose-stack.md`](docs/cards/02-choose-stack.md).

## Status

M1 in progress: repo/env set up, stack chosen. Not yet deployed — see `PROGRESS.md` for
current state and `CLAUDE.md` for full project conventions.

## What's next

Collect the recipe/notes corpus, build the retrieval pipeline, write the first eval set,
ship a minimal Streamlit UI, deploy publicly.
