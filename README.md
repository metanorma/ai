# Metanorma AI — deploy citation-grounded Q&A over your Metanorma corpus

A deployment package for standards bodies and Metanorma users: the complete
architecture, contracts and runbook for a question-answering service over
your own publications — clause-level citations, verbatim quote anchors for
normative values, typed renderings of tables/equations/figures, and
mechanical post-generation verification. Everything here is extracted from
a production deployment (**OIML SMART AI**, ai.oimlsmart.org) whose
measured numbers: golden-suite recall@5 ≈94% (witness-span graded), ≈$0.001
per answer, ≈$75 one-time corpus preparation for ~42k chunks.

**The one-line answer to the architecture question:** this is THREE
independently deployable components — an ingest pipeline (offline, writes
your index), a serving API (stateless, reads your index, calls hosted
models), and a UI (static, talks only to the API) — coupled only by
explicit contracts, not by code. See `ARCHITECTURE.md`.

## What's in this repo

| File | What it gives you |
|---|---|
| `ARCHITECTURE.md` | The three components, their boundaries, and why they are separate |
| `PIPELINE.md` | Stage-by-stage data flow: document → vectors → answer |
| `DEPLOY.md` | The Cloudflare runbook: resources, bindings, bootstrap order |
| `CONFIGURATION.md` | Everything a new deployment MUST change (and must not) |
| `MAPPING.md` | Reference-implementation file map: generic vs corpus-specific |
| `docs/guidelines.md` | The build-from-scratch programme guide (9 stages, measured) |

## Quickstart (the shape of the work)

1. **Export** your Metanorma sources as MKO bundles (MN 116) — one bundle
   per document, typed units + graph + glossary + bibliography.
2. **Ingest** (Python, runs anywhere): chunks along clause boundaries,
   typed units stay atomic; one-time contextual enrichment (~$0.002/chunk);
   embed; write vectors + lexical index + typed payloads + graph to your
   storage.
3. **Serve** (edge worker): deploy the API bound to your storage and model
   host; wire auth (or run anonymous-only).
4. **Present**: deploy the static UI against the API.
5. **Measure before you tune**: golden set + witness grading + faithfulness
   judge from day one — a change without a number is a guess.

Cost posture (open-weight models, single platform): serving is cost-first
(every question pays it), one-time work is quality-first (its quality
persists into every future answer).

## Status

v0.1 — documentation and contracts complete; code extraction is by
reference to the production implementation (see `MAPPING.md`). The
reference deployment is live and measured; this repo makes it yours.
