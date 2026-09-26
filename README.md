# bongard-see-describe-decide

A multi-agent LLM system for **Bongard problems**: 12 small pictures, where the 6 on the left share a hidden
visual rule and the 6 on the right break it.

```
 SEE        one vision call (GPT-4o) describes the panels  +  OpenCV measures them
 DESCRIBE   six specialist agents on open LLMs (LangGraph) each reason over their own facts,
            and can send the vision model back with targeted questions ("look again")
 DECIDE     a judge reviews the whole team and picks the answer; every answer traces to named agents and panel facts
```

Every step is a notebook. There are no `.py` files: functions used by more than one notebook live in
`notebooks/00_common.ipynb`, and each notebook runs its component end to end.

## The five parts of the agent

| Part | In this repo |
|---|---|
| **Tools** | GPT-4o vision (the *eye*), OpenCV measurements (the *ruler*), the look-again vision query |
| **Memory** | Shared LangGraph state carrying panel facts and every round's answers; cached perception, so nothing is paid for twice |
| **Guardrails** | The eye sees the panels shuffled and never knows which side a panel is on, so it can describe but not solve; each specialist reads only its own evidence; look-again questions that mention sides are dropped; the judge may only pick a rule some agent proposed; a hard spend ceiling per notebook |
| **Orchestration** | round 1 → agreement check → look again (or debate) → round 2 → judge, with switches for A/B tests |
| **Observability** | Per problem: every agent's answer and evidence, the questions asked and the eye's answers, the judge's pick and reason, calls and cost |

## How it works

```
 STEP 1 PERCEIVE   eye: one GPT-4o call on the 12 panels, shuffled and labelled P1..P12
                   → a fixed checklist per panel at four levels (parts, objects, relations, whole) + sameness
                   ruler: OpenCV → size, position, long axis, convexity, corners, corner angles
                   code: which properties change across the 12 panels, which are the same in all 12
 STEP 2 ROUND 1    6 specialists (shape, topology, quantity, spatial, size, relation), each on its own facts
                   → a rule, evidence, and optionally one question for the eye
 STEP 3 AGREE?     free code check; if the team agrees, go to the judge
 STEP 4 ROUND 2    look again: all distinct questions → one more GPT-4o call on the same shuffled grid
                   → answers added to the facts → the six specialists answer again
 STEP 5 DECIDE     the judge reads every candidate with its agents and evidence, and picks one
```

What the eye is asked, and why: [`docs/eye_perception_rules.txt`](docs/eye_perception_rules.txt). It derives the
checklist from Hofstadter's and Foundalis's accounts of how people solve Bongard problems, and audits it against
the visual concepts the 100 original problems need.

## Notebooks

| Notebook | What it does end to end | Writes |
|---|---|---|
| `00_common.ipynb` | Shared functions: keys, problem sets, cost meter, model calls, OpenCV perception, the eye and look-again calls, the measured-rule verifier, the grader | – |
| `01_perception.ipynb` | The eye's checklist and the ruler's measurements for every panel, and what changes across the 12 panels | `results/perception/` |
| `02_pipeline.ipynb` | The multi-agent pipeline (LangGraph), with switches: round 2 = look again / debate / none; final = judge / supervisor | `results/predictions/pipeline__*.json` |
| `03_baselines.ipynb` | GPT-4o direct · measured-rule (contrast) baseline · single agent · self-consistency at the pipeline's compute | `results/predictions/` |
| `04_evaluation.ipynb` | 3-vote grading · accuracy with 95% bootstrap CI · McNemar tests · design vs held-out sets · cost per problem · debate/conformity analysis · grader validation sheet | `results/tables/`, `results/figures/` |
| `05_shapley_sage.ipynb` | Shapley attribution: which measured features and which specialists carry the solves | `results/tables/`, `results/figures/` |

## Run it

```bash
pip install -r requirements.txt
cp .env.example .env            # add OPENAI_API_KEY and GROQ_API_KEY
cd notebooks
PROBLEM_SET=SMOKE jupyter nbconvert --execute --to notebook --inplace 02_pipeline.ipynb
```

Run in order: `01 → 02 → 03 → 04 → 05`.

**Settings** (environment variables, or edit the top cell of a notebook):

| Variable | Values |
|---|---|
| `PROBLEM_SET` | `SMOKE` (3 problems) · `DEV_21` (development) · `HELDOUT2` (20 never used in design) · `TEST_30` (all not used in design) · `ALL_100` |
| `ARMS` (02) | `main` = look again + judge · `ab` = also debate + judge · `all` = also the clone-agent control and a smaller model |
| `FIELDS_VERSION` | `fields-v4` (default checklist) · older cached readings: `fields-v2`, `fields-v3` |
| `BUDGET_USD` | Spend ceiling per notebook (default 2.0) |

**Models:** vision = `gpt-4o`; agents and judge = `openai/gpt-oss-120b` on Groq; grader = `openai/gpt-oss-120b`.
Override in `.env`.

## Evaluation honesty
- **Problem sets:** `DEV_DESIGN` problems were used to design the system. `DEV_HELDOUT` and `DEV_HELDOUT2` were not.
  Results are reported separately for each.
- **Only `ALL_100` is comparable with published results** (o1 ≈ 43/100, GPT-4o ≈ 24/100).
- **Grading:** a 3-vote LLM grader compares each rule with Bongard's own answer. A hand-validation sheet
  (Cohen's κ) checks the grader.
- **Earlier designs** are kept as `results/predictions/pipeline-v*` for comparison.

## Data
`data/bongard-classic/`: the 100 original Bongard problems, cropped into 12 panels each, with the answers
from Foundalis' index (see `SOLUTIONS_PROVENANCE.md`).

## Working as a team
- One component per notebook; keep each runnable top to bottom.
- Put a function in `00_common.ipynb` only when a second notebook needs it.
- Never commit `.env`.
