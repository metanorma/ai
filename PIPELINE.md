# Pipeline — stage by stage, document to answer

## A. Document → index (ingest; offline)

```
metanorma sources (.adoc)
  → MKO export (MN 116)          one bundle per document; manifest SHA-verified
  → ingest                        clause chunks + atomic typed units; stable
                                  content-hash ids; provenance on every chunk
  → contextual enrichment         one-time model preambles (KV-cached by id;
                                  unchanged chunks are free on re-runs)
  → embed                         ONE embedding model both sides (mandatory)
  → index write                   vector index (dense) · FTS5 full-corpus rows
                                  (lexical) · typed payloads (D1) · graph
                                  projection · figure assets (R2, immutable)
  → data-quality report           counts, missing anchors, oversized payloads
```

Ingest rules that earned their place by measurement:

- Lexical retrieval must scan the WHOLE corpus as a first stage
  (re-scoring dense hits cannot recover what dense missed: +9pp recall@5
  measured on the reference corpus).
- Tables are atomic; the typed unit carries its parent clause anchor.
- Every typed unit is retrievable (no payload-only unit types).
- Incremental re-ingest diffs by (document, unit id) + content hash — a
  wording change reprocesses only the affected chunks.

## B. Question → answer (serving; per request)

```
question (any language)
  → understand (small LLM)        language · named publication · edition ·
                                  defined terms · complexity ∥ (concurrently)
                                  the dense lane embeds + queries unfiltered
  → retrieve                      dense ∥ full-corpus BM25 → RRF fusion;
                                  doc/edition metadata filters from
                                  understanding; graph lane for terminology
  → rank                          cross-encoder; listwise pass for complex
                                  queries (bounded, ~2.5s); edition steering
                                  (family-relative demotion of superseded
                                  editions; corpus-corroborated pins)
  → generate                      answer model; passages-only; inline
                                  citations; verbatim quote anchors; symbolic
                                  [[u:…]] references; figure images attached
                                  for multimodal interpretation
  → verify (mechanical)           quote anchors ⊆ used passages (verbatim);
                                  [[u:…]] resolve against used passages;
                                  retyped-table detection; faithfulness
                                  judge on the USED passages
  → serve/cache                   verified answers cached by query hash +
                                  index version; refusals never cached
```

## C. Answer → screen (UI; static)

SSE stream → markdown with linkable citation chips; typed blocks rendered
from the producer's payloads (real tables, KaTeX, images); feedback thumbs
into the evaluation loop.

## The evaluation loop (cross-cutting)

Golden set with WITNESS citations (a hit requires span containment, not
just the right document) · paraphrase probes · behavioural e2e ·
faithfulness battery · answer-variance (determinism) · user feedback
triage. All suites are CI gates; every retrieval change bumps the index
version so no answer is served from a superseded index.
