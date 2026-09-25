# fields-v5 + 9-specialist roster — DEV_21 result (2026-09-25)

Self-graded against Foundalis ground truth (`data/bongard-classic/solutions.json`), not the Groq grader.

## Config
- Perception: `fields-v5`, `PERCEPTION_MODE=perpanel` (12 independent side-blind gpt-4o calls/problem)
- Reasoner: `openai/gpt-oss-120b` (Groq), arm `ask_eye` (look-again) + judge
- Roster (9): shape, contour, size, quantity, **fill (new)**, spatial, topology, **gestalt (new)**, relation

## Result (n=21)
| arm | strict | lenient |
|---|---|---|
| grid-v4, 6-specialist (baseline) | 13/21 (62%) | — |
| perpanel-v4, 6-specialist | 12/21 (57%) | — |
| **perpanel-v5, 9-roster (this)** | **13/21 (62%)** | **16/21 (76%)** |

Net strict = wash vs baseline, but composition shifted (see below).

## Per-problem (this run)
- ✅ solid (13): bp0004, bp0007, bp0011, bp0022, bp0025★, bp0028★, bp0033, bp0036, bp0038, bp0047, bp0050★, bp0056★, bp0057
- 🟡 borderline (3): bp0010 ("no triangles" — negation), bp0031 ("single vs multiple objects" ≈ 1 vs 2), bp0097 ("non-triangular" — negation)
- ❌ wrong (5): bp0002 (chased "holes" distractor; GT=size), bp0030 (judge passed over self-crossing candidate), bp0046 (missed on-top relation), bp0049 (points-density; gestalt gap), bp0096 (no rule/abstain)

★ = recovered vs the 6-specialist baseline (fill + symmetry) — the concept dimensions that had no owner before.

## Diagnosis
- **Levers proven:** fill (bp0025/0028/0056) and symmetry (bp0050) recovered exactly as the 100-problem audit predicted.
- **Net stuck because ~4 gains cancelled by ~4 regressions/downgrades**, all traced to the JUDGE, not perception:
  - distractor-chasing (bp0002), passing over a correct candidate (bp0030), abstaining (bp0096), leaving negations un-reworded (bp0010, bp0097).

## Next (judge-prompt fix, this file's follow-up run)
FINAL_PROMPT hardened: prefer simplest/most-general rule + ignore incidental detail; name both sides positively (no "no X"/"non-X"); always decide (no abstain). Expected to upgrade bp0010, bp0097 (negation→positive) and recover bp0002/bp0096, without losing the fill/symmetry gains. Re-run on cached v5 perception (fast). Remaining hard gap: bp0049 (points density).

Baselines preserved in scratch `baselines_v4/`.
