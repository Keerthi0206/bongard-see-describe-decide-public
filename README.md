# Bongard problems — see once, reason together

A Bongard problem shows 6 **left** panels that share a hidden visual rule and 6 **right** panels that break it.
This repo tests one idea:

> **Use a vision model once per problem to read the panels, then let small, cheap, open agents reason.**

Every step is a notebook. There are no `.py` files: functions used by more than one notebook live in
`notebooks/00_common.ipynb`, and each notebook runs its component end to end.

## Notebooks

| Notebook | What it does end to end | Writes |
|---|---|---|
| `00_common.ipynb` | Shared functions only: keys, problem sets, cost meter, model calls, CV perception, CV verifier, montages, the one-call VLM perception, the grader | – |
| `01_perception.ipynb` | The **ruler** (CV: size, position, long axis, elongation, convexity, corners, corner angles, blob count, inside/overlap) and the **eye** (one VLM call: type, number of sides, fill, pointing, count, relations, lines, symmetry, texture) — no fact comes from both | `results/perception/` |
| `02_pipeline.ipynb` | The multi-agent pipeline (LangGraph): 6 agents → consensus judge → debate (switch only with a named counterexample) → supervisor → check → one look-again. Two styles: **specialist** (each agent reads different evidence) and **clone** (six identical generalists, the control) | `results/predictions/pipeline__*.json` |
| `03_baselines.ipynb` | CV contrast · gpt-4o direct · router · **single agent** · **self-consistency** at the pipeline's compute | `results/predictions/` |
| `04_evaluation.ipynb` | 3-vote grading · accuracy with 95% bootstrap CI · McNemar tests · conformity in debate · grader hand-validation sheet · published comparison · figures | `results/tables/`, `results/figures/` |
| `05_shapley_sage.ipynb` | Exact Shapley attribution: which CV feature family and which specialist carries the solves | `results/tables/`, `results/figures/` |

## Run it

```bash
pip install -r requirements.txt
cp .env.example .env            # add OPENAI_API_KEY and GROQ_API_KEY
cd notebooks
```
Run in order: `01 → 02 → 03 → 04 → 05` (03's self-consistency matches 02's call count). Choose the problems with the `PROBLEM_SET` environment variable
(`SMOKE` = 3 problems, `DEV_21` = development set, `HELDOUT2` = 20 problems never used in design,
`TEST_30` = every problem not used in design, `ALL_100` = every classic problem), or edit the
`PROBLEMS = ...` line at the top of a notebook.

- `ARMS=all` runs all three pipeline arms in 02 (specialist-120b, clone-120b, specialist-20b); the default
  runs only specialist-120b, to keep test runs cheap.
- `FIELDS_VERSION=fields-v2` reuses the older cached eye readings instead of making new vision calls.

```bash
PROBLEM_SET=DEV_21 jupyter nbconvert --execute --to notebook --inplace 02_pipeline.ipynb
```

- **Cost control:** each notebook has one spend ceiling (`BUDGET_USD`, default $2) and prints its total.
- **Nothing is paid for twice:** VLM readings, predictions and grades are cached in `results/`; runs resume.
- **Models:** vision = `gpt-4o` (one call per problem); agents = `openai/gpt-oss-120b` or `-20b` on Groq;
  grader = `openai/gpt-oss-120b`. Override in `.env`.

## Working as a team
- One component per notebook; keep each runnable top to bottom.
- Put a function in `00_common.ipynb` only when a second notebook needs it.
- Never commit `.env`. Commit `results/` so everyone reads the same numbers.
- `DEV_DESIGN` problems were used to design the system; `DEV_HELDOUT` and `DEV_HELDOUT2` were not. Judge design
  changes on the held-out sets only, and report all three separately.
- Older runs are kept as `pipeline-v1__*` and `pipeline-v4__*` in `results/predictions/`, for before/after comparison.

## Data
`data/bongard-classic/`: the 100 original Bongard problems, cropped into 12 panels each, with Bongard's own
answers from Foundalis' solutions page (see `SOLUTIONS_PROVENANCE.md`). The answers are authoritative but the
crops have not yet been eye-checked one by one.

## Results
See `results/tables/accuracy.csv` and `results/figures/`. Numbers on `SMOKE` or `DEV_21` are development signals;
only `ALL_100` is comparable with published results (o1 ≈ 43/100, GPT-4o ≈ 24/100).
