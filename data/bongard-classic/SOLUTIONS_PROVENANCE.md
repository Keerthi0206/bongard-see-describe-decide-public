# Bongard-Classic answer key — provenance

> ✅ **UPDATED 2026-09-03: concept labels are now the REAL Foundalis concepts.**
> The 100 images are the real Foundalis classic-100 (1197 distinct panels). The `concept` labels were
> previously templated `project-legacy` placeholders (~19 concepts across 100 puzzles) — those have
> been **replaced with the authoritative solutions from Foundalis' solutions page** (Bongard's 1967
> appendix): 93 distinct concepts, `sources` includes `foundalis`. Backup of the old templated key:
> `solutions.templated.bak.json`.
>
> **Remaining step:** the labels are authoritative but **not yet human eye-verified against the
> cropped panels** (`verified_by` is still null). Run `verify_solutions.py` to eyeball each crop vs its
> concept and stamp `verified_by`. Until then, semantic numbers are "authoritative-draft" (real
> concepts, crop-alignment unconfirmed); the OBJECTIVE verifier accuracy never depended on labels.
>
> Note: CV cannot express many of these concepts (world-knowledge like bp100 "letter А vs Б",
> clockwise arrangement, etc.), so a low CV *semantic* score on those is a perception ceiling, not a
> label error.

`solutions.json` is structured (not a flat string) and machine-validated for SCHEMA, but the label
CONTENT is not yet verified (see the warning above).

## Schema (per `bpNNNN`)
| field | meaning |
|---|---|
| `concept` | human semantic gloss — the LLM-judge target |
| `left_rule` / `right_rule` | side-specific natural-language rule |
| `concept_rule` | optional machine-checkable rule (verifier) |
| `evaluable` | in our evaluable subset? (drives `subset.EVALUABLE_SUBSET`) |
| `exclude_reason` | why not evaluable (OCR / world-knowledge / pending curation) |
| `sources` | ≥1 of: `bongard-caption`, `foundalis`, `depeweg`, `project-legacy`, `manual` |
| `verified_by` / `verified_date` | set ONLY by a human via `verify_solutions.py` |
| `notes` | free text |

## How to (re)build
```bash
# 1. scaffold all 100 + migrate legacy + (optional) draft concepts from Foundalis
python scripts/build_solutions.py --foundalis

# 2. human eye-verification (renders 6-left|6-right montages, stamps verified_by)
python scripts/verify_solutions.py --all-evaluable --by <name>

# 3. validate schema + coverage (also enforced by tests/test_solutions.py)
python -c "from bongard_debate.benchmarks.eval.solutions import validate; print(validate() or 'OK')"
```

## Sources (triangulate — never a single source)
- **Bongard captions** — the original intended concepts (Depeweg et al. appendix lists them).
- **Foundalis** index — http://www.foundalis.com/res/bps/bpidx.htm
- **Depeweg, Rothkopf & Jäkel (2024)** — the expressible set (they solve 35/39).

## Current status
- BP 1–10: migrated from the project's legacy key as **drafts** (`project-legacy`),
  `evaluable:true`, **not yet eye-verified** (`verified_by: null`).
- BP 11–100: scaffolded `evaluable:false, exclude_reason:"pending curation"`.
- `EVALUABLE_SUBSET` is derived from `evaluable:true` entries.

**Verification is a human gate** — no script marks an entry verified. Run
`verify_solutions.py` and record who verified what before publishing numbers.
