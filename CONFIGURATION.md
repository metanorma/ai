# Configuration — what a new deployment changes

## Must change (corpus identity)
- **DATASETS catalog** (config): id, label, description, per-corpus prompt
  notes, audience gating. Corpus behavior travels WITH this file, not code.
- **Corpus sources**: your Metanorma repos; INGEST_LANGUAGES (the reference
  is English-only by directive; output language is serving-time).
- **Registry derivation**: status from supersession edges, never fields —
  wire your bibliography (Relaton or equivalent) and surface the families
  with no derivable current edition instead of guessing.
- **OIDC / access**: the reference is a relying party of an external OP
  with per-client claims policies; anonymous-only is a valid first deploy.
- **Quotas and tiers**: anon/key/member day limits.

## Must review (model policy)
- Model ids per role, EXACTLY as your host's catalog lists them; per-role
  env overrides (`<ROLE>_MODEL`) enable A/B without code changes.
- Per-model call parameters FROM THE CARD: reasoning-mode controls and
  defaults, sampling, output budgets (three separate live incidents in the
  reference deployment trace to unset defaults — see the guidelines).
- Cost posture: cost-first on the serving path, quality-first one-time.

## Must not change (the contracts)
- Chunk metadata schema (the serving profile: scalar-only, byte-budgeted).
- The answer contract (inline citations, quote anchors, [[u:…]] blocks).
- Prompts are data files; eval suites are CI gates.

## Should keep
- Prompts-as-data, no keyword rules on user input (the understanding model
  decides; the same rules serve every language).
- The refusal contract: one canonical sentence, never cached.
- Verification before caching, always.
