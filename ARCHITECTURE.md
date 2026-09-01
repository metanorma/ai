# Architecture — three components, explicit contracts

## The question this answers

"Document → vectors pipeline, API that loads a model and returns results,
results presentation of UI — are all separate components?"

**Yes — three independently deployable components, plus the contracts that
couple them.** The serving API *never* loads model files and *never* reads
your documents: models come from a model host (Workers AI / any inference
platform) and the corpus arrives only as the index the ingest pipeline
wrote. Each component can be replaced without touching the others, as long
as the contracts hold.

```
┌──────────────────────┐      writes       ┌──────────────────────────┐
│ 1. INGEST PIPELINE   │ ────────────────► │  STORAGE (the index)     │
│ (offline, batch)     │  vectors, lexical,│ vector index · FTS5 rows │
│ Python; runs any-    │  typed payloads,  │ typed payloads · graph   │
│ where; no runtime    │  assets           │ assets · caches          │
└──────────────────────┘                   └────────────┬─────────────┘
                                                        │ reads (bindings)
┌──────────────────────┐      HTTPS/SSE    ┌────────────▼─────────────┐
│ 3. UI                │ ◄──────────────── │ 2. SERVING API           │
│ (static site)        │                   │ (stateless edge worker)  │
│ citations, typed     │                   │ understand → retrieve →  │
│ blocks, feedback     │                   │ rank → generate → VERIFY │
└──────────────────────┘                   │ models: the host serves  │
                                           └──────────────────────────┘
```

## Component 1 — the ingest pipeline (producer side, offline)

Runs on a laptop or CI. Per corpus, then incrementally per change:

1. **Export**: each Metanorma document → an MKO bundle (MN 116): typed
   units (clauses, tables, terms, equations, figures, requirements), a
   section graph (part_of / cites / defines), glossary, bibliography.
2. **Chunk**: clause boundaries, never token windows; tables/equations/
   figures stay ATOMIC typed units that cite their parent clause. Stable
   ids (content-hash) make re-ingest incremental.
3. **Contextual enrichment** (one-time, quality-first): a strong model
   writes a 2–3 sentence preamble situating each chunk; embed
   preamble+text. ~$0.002/chunk; the single highest-ROI one-time spend.
4. **Index**: vectors (dense lane), full-corpus lexical rows (BM25 lane),
   typed payloads (for symbolic rendering), graph projection, figure
   assets (immutable URLs + vision captions).
5. **Data-quality report** every build: counts by type, missing anchors,
   oversized payloads — serving keeps score; upstream fixes follow.

Decoupling: the pipeline never serves. You can rerun it on a corpus fix
while the API is live (incremental mode rewrites only changed ids).

## Component 2 — the serving API (stateless)

Per request: **understand** (small LLM: language, named publication,
edition, terms — no keyword rules) ∥ **retrieve** (dense + full-corpus
BM25 in parallel; metadata filters for doc/edition; graph lane for
terminology) → **rank** (cross-encoder; optional listwise for complex
queries) → **generate** (answers from the retrieved passages only, inline
citations + verbatim quote anchors + symbolic unit references) →
**verify** (mechanical: every quoted phrase exists in a used passage;
every unit reference resolves; a faithfulness judge sees the passages the
answer was actually built from). Unverifiable answers never enter the
cache. Off-corpus → one canonical refusal, never cached.

The API "loads a model" only in the sense of *calling* the model host per
request (config-driven model ids, per-role, from the host's catalog).
Swap models by config; gate every swap with the eval suites.

## Component 3 — the UI (static)

Talks ONLY to the API (SSE streaming + JSON). Renders citations as
linkable source chips, typed blocks as REAL objects (tables from the
producer's columns/rows, equations via KaTeX, figures from the asset
store), feedback thumbs wired to the evaluation loop. No corpus content
in the client; no auth logic beyond a session check.

## The contracts (the only coupling)

1. **MKO (MN 116)** — what the pipeline consumes.
2. **Chunk metadata schema** — provenance on every chunk: docidentifier,
   edition, language, clause anchor, status, unit_id/block; scalar-only,
   byte-budgeted for vector stores ("serving profile").
3. **Answer contract** — inline citations `[DOC ed §clause]`, verbatim
   quote anchors for normative values, symbolic unit references
   `[[u:…]]` resolved server-side to typed payloads returned alongside
   prose as `blocks`.
4. **DATASETS catalog / prompts as data** — corpus behavior and prompts
   are configuration files, not code.

## Why the separation matters

- Replace the UI without touching retrieval; replace the model host
  without touching ingest; rerun ingest without downtime.
- The access boundary is structural: a public deployment simply does not
  bind the storage of a private corpus.
- Evaluation composes per component (retrieval metrics / faithfulness /
  behavioural e2e) — see the guidelines, Stage 7.
