# Card 02 — Choose & document the stack

Notion card: Cooking-RAG / M1 / Choose & document the stack
Status: Done — 2026-08-12

## Problem & objective
- **Pain points:** Card 1 already had to make snap stack decisions (FAISS, `sentence-transformers`,
  Gemini Flash) just to unblock environment setup, and the README paragraph documenting them
  stated the picks but not the reasoning against alternatives. A stated-but-undefended choice is
  weak both technically (no evidence it was the right call) and for interview purposes (can't
  answer "why not X" convincingly without having actually compared X).
- **Objective statement:** Formally document the RAG stack decision — embedding model, vector
  store, LLM API — with trade-off comparisons against the broader landscape, in a form that
  doubles as interview-ready material, not just a changelog entry.

## Quantitative metrics
- Stack decision documented in README (`README.md` → Stack section): **1 paragraph**, present
  since card 1, now cross-linked to this deeper comparison.
- Landscape coverage: **3 decision axes** (embeddings, vector store, LLM API), each compared
  against **≥4 named alternatives** with explicit pick/reject reasoning.
- Adjacent RAG techniques catalogued: **5** (chunking strategy, hybrid search, reranking, query
  transformation, evaluation), each mapped to a concrete trigger condition and the project
  milestone (M2/M4) it belongs to.
- Reading list curated: **6 foundational papers** (arXiv), **4 living references** (leaderboard,
  vendor learning centers, framework docs), **2 courses** — all verified as real, stable,
  well-known sources rather than guessed.

## Qualitative benchmarks
- Bar: for each of the 3 stack choices, can state in one sentence *why* it was picked and one
  sentence naming the *trigger condition* that would change the choice — the actual test of
  "can articulate trade-offs cold" for an interview setting, not just "can recite the picks."
- Self-check applied: every "why not X" alternative below has a named, concrete trigger
  condition (e.g. "need metadata filtering → Chroma," "hit free-tier rate limits → Claude
  Haiku," "exact-term misses → hybrid search") rather than a vague "might be better someday."

## Acceptance criteria
- [x] Embedding model picked: `sentence-transformers` / `all-MiniLM-L6-v2`.
- [x] Vector store picked: FAISS.
- [x] LLM API picked: Gemini Flash (free tier), via the `google-genai` SDK.
- [x] One paragraph in README explaining why (written in card 1, still accurate).
- [x] Full trade-off comparison across landscape alternatives (this document) — goes beyond the
      Notion card's literal minimum bar, done deliberately since the stated goal for this card
      is portfolio/interview readiness, not just unblocking setup.

## Trade-offs

### Embedding models
What actually varies: dimensionality, max chunk context length, MTEB retrieval score,
multilingual coverage, API vs. self-hostable.

| Option | Type | Why you'd pick it | Why you wouldn't |
|---|---|---|---|
| OpenAI `text-embedding-3-small/large` | API | Cheap, zero infra, strong general retrieval (MTEB ~62–65) | Recurring cost + latency per call, vendor lock-in, data leaves infra |
| Cohere Embed v4 | API | Strong retrieval (~65), good multilingual | Same API trade-offs, separate vendor from the LLM |
| `sentence-transformers` (BGE, E5, MiniLM, Jina, Qwen3-Embedding) | Local/open-weight | Free, data stays local, full control, Apache-licensed | Heavier install (`torch`), you own the ops; quality ranges from "fine" (MiniLM) to SOTA (Qwen3-Embedding-8B) |
| NV-Embed / KaLM / Qwen3-Embedding | Local, SOTA-tier | Best-in-class open-weight retrieval as of 2026 | Multi-GB models, real compute needed — overkill at a few dozen documents |

**Picked `all-MiniLM-L6-v2`.** Trade-off to defend: the cheapest model that clears the bar,
because at this corpus size embedding-model quality isn't the bottleneck — chunk design and
retrieve-both routing are. **Trigger to reconsider:** measured recall failures on the eval set
traceable specifically to embedding quality (see Future/improvements — not yet tested).

### Vector store
Real axis: managed vs. self-hosted, and whether metadata filtering / hybrid search / horizontal
scale are actually needed — at small scale, everything is fast enough.

| Option | Model | Why | Why not |
|---|---|---|---|
| FAISS | In-process library | No server, no extra deps beyond the lib, industry-standard ANN (HNSW/IVF) | No native metadata filtering or persistence — build it yourself |
| Chroma | Embedded DB | Ergonomic API, built-in metadata filtering, easy local persistence | Heavier dependency tree; known `sqlite3`-version friction on some hosts (incl. Streamlit Cloud) |
| Qdrant / Weaviate / Milvus | Self-hosted server | Production-grade filtering, hybrid search, horizontal scale | Needs a running server — real infra this project doesn't need yet |
| Pinecone / managed cloud | Fully managed | Zero ops, scales to billions of vectors | $ cost, vendor lock-in, network latency per query |
| pgvector | Postgres extension | One less moving part if Postgres already exists | Slower ANN than purpose-built libraries at scale |

**Picked FAISS.** Trade-off to defend: no metadata filtering or horizontal scale needed at 50
documents, so optimized for zero deployment friction over query ergonomics. **Trigger to
reconsider:** need per-layer metadata filtering beyond two flat indices → Chroma is the first
thing to reach for.

### LLM API

| Option | Why | Why not |
|---|---|---|
| Claude (Haiku/Sonnet/Opus) | Strong grounded-QA + citation faithfulness | No free tier |
| GPT-4o / GPT-4o-mini | Widely benchmarked, strong tool-use, big ecosystem | No free tier |
| Gemini Flash | Genuinely free tier, fast, decent quality | Rate-limited by design |
| Open-weight via Groq/Together (Llama, Mistral, Qwen) | Cheap/fast inference, no model lock-in | Still a paid API; quality varies by model size |
| Fully local (Ollama) | $0, full privacy | Needs real compute — not deployable on Streamlit Cloud's free tier |

**Picked Gemini Flash.** Trade-off to defend: free-tier LLM cost is the right optimization
target for a portfolio project with near-zero query volume — the eval harness is what proves
the pipeline works, not which model answers. **Trigger to reconsider:** free-tier rate limits
hit during heavy eval-harness runs (M2, 50+ questions) → documented fallback is Claude Haiku,
an isolated change inside `core/llm.py`.

### Adjacent techniques (not built now, but load-bearing for the interview narrative)
- **Chunking:** fixed-size / recursive-split / semantic chunking (split where embedding
  similarity between adjacent sentences drops) are the standard options; we used the simplest
  viable one — one chunk per recipe/note — because our source docs already have natural,
  author-chosen boundaries, so mechanical chunking would be solving a problem we don't have.
- **Hybrid search (dense + BM25):** pure embedding search misses exact-term matches (ingredient
  names, brand terms). **Trigger:** eval failures on exact-match queries → M4 candidate.
- **Reranking:** retrieve ~20 candidates cheaply, rerank with a cross-encoder (Cohere Rerank,
  BGE-reranker) down to 3–5 before the LLM call — generally the highest-leverage precision
  upgrade in production RAG stacks. **Trigger:** borderline retrieval quality on card 8's 5
  real-question test → worth prototyping even before M4.
- **Query transformation:** HyDE, multi-query expansion, query routing between layers — this is
  literally what the project's own M4 milestone ("agentic upgrade... driven by eval failures")
  already targets.
- **Evaluation:** retrieval metrics (recall@k, MRR, NDCG) vs. generation metrics (faithfulness,
  answer relevance) — RAGAS is the standard framework for the latter, directly matching this
  project's stated differentiator (the eval harness).

## Non-goals
- Not implementing hybrid search, reranking, or query routing now — named as explicit upgrade
  triggers for M2/M4, not built in M1.
- Not empirically benchmarking embedding models against our own corpus (no local MTEB-style
  bake-off) — the decision rests on published benchmarks + reasoning about corpus scale, not
  head-to-head testing on our own data. Flagged below as a future improvement.
- Not evaluating self-hosted vector DBs (Qdrant/Weaviate/Milvus) or managed cloud (Pinecone) in
  depth — ruled out early since they require infra this project doesn't need at current scale;
  covered only for landscape completeness above.

## Blockers & rabbit holes
- The README's stack paragraph (written during card 1) stated the decision but lacked the
  "why not the alternatives" comparison — required this dedicated research pass to fill
  properly, since the portfolio/interview value comes from demonstrated breadth of
  consideration, not just a stated pick.
- Needed to verify 2026-current specifics via web search (MTEB leaderboard state, current
  best embedding models, RAG best-practice trends), since general knowledge alone was stale on
  specific current model names — the same pattern that bit `requirements.txt` versions in card
  1. Cross-checked search results against known-solid foundational sources (arXiv papers, the
  MTEB leaderboard itself) since several blog hits from the search were lower-confidence SEO
  content rather than primary sources.

## Future / improvements
- Run an actual small-scale embedding-model bake-off on the real corpus once cards 3/4 land
  (e.g. MiniLM vs. a stronger open-weight model like BGE-small on card 8's 5 known-answer
  questions) — upgrades "picked based on published benchmarks" to "picked based on measured
  results on our own data," a stronger interview claim.
- Hybrid search (dense + BM25) is the most likely first M4 upgrade if eval failures show misses
  on exact ingredient/brand-name queries.
- Reranking is the highest-leverage precision upgrade generally cited in current RAG
  literature — worth prototyping even before M4 if card 8 shows borderline retrieval quality.
- Revisit LLM choice if Gemini free-tier rate limits are hit during M2's eval harness runs —
  Claude Haiku is the pre-documented fallback.

## Reading list
**Foundational papers:**
- Lewis et al., ["Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks"](https://arxiv.org/abs/2005.11401) (2020) — the original RAG paper.
- Karpukhin et al., ["Dense Passage Retrieval for Open-Domain Question Answering"](https://arxiv.org/abs/2004.04906) (2020).
- Liu et al., ["Lost in the Middle: How Language Models Use Long Contexts"](https://arxiv.org/abs/2307.03172) (2023) — relevant to how retrieved chunks are ordered in the prompt.
- Gao et al., ["Precise Zero-Shot Dense Retrieval without Relevance Labels"](https://arxiv.org/abs/2212.10496) (HyDE, 2022).
- Muennighoff et al., ["MTEB: Massive Text Embedding Benchmark"](https://arxiv.org/abs/2210.07316) (2022).
- Es et al., ["RAGAS: Automated Evaluation of Retrieval Augmented Generation"](https://arxiv.org/abs/2309.15217) (2023) — maps directly to this project's eval-harness milestone.

**Living references:**
- [Hugging Face MTEB Leaderboard](https://huggingface.co/spaces/mteb/leaderboard) — current embedding-model rankings.
- [Pinecone Learning Center](https://www.pinecone.io/learn/) — practical explainers on chunking, hybrid search, reranking.
- Anthropic engineering blog, "Contextual Retrieval" — case study on why naive chunking loses context.
- LangChain / LlamaIndex conceptual docs — neutral explainers of RAG patterns (parent-document retrieval, self-query, RAG-fusion).

**Courses:**
- DeepLearning.AI, "Building and Evaluating Advanced RAG" and "Advanced Retrieval for AI with Chroma" — free, ~1–2 hrs each, hands-on.
