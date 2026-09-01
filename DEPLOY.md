# Deploy runbook (Cloudflare reference)

One platform, minimal accounts. Resources in creation order:

1. **Vectorize index** — dims must equal your embedding model's output
   (e.g. 1024). Create metadata indexes for every filter field
   (doc_number, edition, language, doctype) or filters silently return 0.
2. **D1 database** — apply the migrations: chunks + FTS5 (porter) with the
   sync triggers, unit_payloads, graph tables, telemetry/spend/feedback.
3. **KV namespace** — answer cache (keyed by index version), semantic
   cache, enrichment contexts (30d TTL).
4. **R2 bucket(s)** — figure assets at immutable, unit-keyed URLs; one
   bucket per audience if you serve scoped corpora.
5. **Worker (serving API)** — bindings: AI (model host), VECTORIZE, DB,
   CACHE, ASSETS; vars: INDEX_VERSION, quotas, model ids per role;
   secrets: ADMIN_TOKEN, session/OIDC secrets if you gate access.
6. **Static UI** — build and deploy as the worker's assets (or any static
   host pointed at the API, CORS configured).

Then: run the ingest pipeline against your corpus → run the eval suites
BEFORE going live → deploy via a guarded script (branch checks, version
bump, post-deploy smoke). Reference commands and the full bootstrap script
shape are in MAPPING.md's pointers.

Ops facts from the reference deployment: incremental index updates are
seconds (changed ids only); full lexical rebuild ≈30–40 min per ~40k
chunks; stage-timing logs on the ask path are the aim point for latency
work; a weekly spend ledger review against the model policy.
