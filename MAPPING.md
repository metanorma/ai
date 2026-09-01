# Reference-implementation file map

The production reference is `oimlsmart/rag` (private). The MN 116
contract itself is proposed to live in `metanorma/mko`
(metanorma/metanorma-document#54) — this package will link it as THE
contract reference once extracted. This map says which
parts are the GENERIC system (extract these for a shared package) versus
OIML/estate-specific (every deployer reimplements or omits).

## Generic — the system proper

| Area | Files (reference paths) |
|---|---|
| Serving pipeline | `workers/worker_public/src/pipeline.ts` (retrieve, fusion, ranking, steering, pin, message assembly), `lexical.ts`, `understand.ts`, `grader.ts`, `reflect.ts`, `faithfulness.ts`, `anchors.ts`, `refs.ts` (answer contract v2), `tablecontext.ts` |
| Ask orchestration | `workers/worker_public/src/index.ts` (ask loop, caching, verification loop, multimodal attach, research loop) |
| Prompts as data | `workers/worker_public/prompts/*.md` |
| Config | `src/config.ts` (MODELS/LIMITS/DATASETS + roleModel overrides) |
| Ingest | `ingest/` (parse, chunk, mko.py [MN 116 consumer], fts.py [lexical + langid + incremental], enrich, graph, tables, cf) + `scripts/wire_mko.py`, `export_mko.rb` |
| Typed-block UI | `site/src/chat/UnitBlocks.vue`, citations lib, SourceChips |
| Eval | `tests/retrieval.mjs` (witness-span), golden sets, e2e, ui/browser suites, `scripts/variance.mjs`, `scripts/feedback-triage.mjs` |
| Ops | `scripts/deploy.sh` (guarded), admin endpoints (enrich/vectors/caption), migrations |

## Corpus/estate-specific — not portable as-is

- OIML datasets/auth: `DATASETS` entries, OIDC estate specifics, quota
  policy, the internal-federation service binding (the two-audience
  isolation pattern IS portable; the smartcab corpus is not).
- OIML golden sets and witness values (yours will differ — write them
  from your corpus, 20–30 questions, before tuning anything).
- Publication-URL derivation, OIML docidentifier parsing conventions.
- The site shell/chrome (federation header) — the chat components are
  generic; the shell belongs to the estate.

## Extraction order for a shared package

1. contracts: prompts/, config schema, migrations, answer-contract types
2. serving: the worker src/ tree minus estate auth
3. ingest: the ingest/ tree (corpus loaders parameterized)
4. eval: the suites parameterized by your golden files
5. UI chat components over a neutral shell
