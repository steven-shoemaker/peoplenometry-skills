---
result_format: 1
skill: why-are-people-leaving
date: 2026-07-12
question: Why are people leaving, and is our attrition actually a problem?
window: 2024 full year (12 months)
metrics:
  - name: Attrition Rate (annualized)
    value: 15.5%
    numerator: 486
    denominator: 3131
    n: 486
  - name: Voluntary Attrition (annualized)
    value: 8.8%
    numerator: 274
    denominator: 3131
    n: 274
  - name: Involuntary Attrition (annualized)
    value: 6.8%
    numerator: 212
    denominator: 3131
    n: 212
  - name: First-year exit share
    value: 57%
    numerator: 275
    denominator: 486
    n: 486
  - name: Naive dashboard rate (terms / all-ever-employed)
    value: 29.9%
    numerator: 1485
    denominator: 4962
    n: 1485
house_rules_applied: []
caveats:
  - Reasons are self-reported exit codes; correlation, not cause.
  - No department clears the elevated-segment bar — team-level rates are within normal spread.
  - Compensation appears as the top voluntary reason but a compa-ratio / below-band analysis was not run; treat comp as a lead to test, not a proven driver.
---

## The answer

Attrition over 2024 ran **15.5% annualized** (voluntary **8.8%**, involuntary **6.8%**) against an average headcount of **3,131** — squarely in the healthy band for a company this size. If you've seen a **29.9%** "turnover rate" on a dashboard, that figure divides all-time terminations by everyone ever employed; it is not an annual rate and it overstates the picture by roughly 2×. On a fair, annualized basis, the company is not bleeding people.

The problem isn't *how many* leave — it's *when*. **57% of 2024 leavers were gone inside their first year** (275 of 486). That front-loading holds on both sides: early voluntary quits (top reason: **compensation**, then *better opportunity*) and early involuntary exits (skills mismatch, PIP failure). No single team drives it — **no department clears the elevated-segment bar** (highest voluntary rate is Product at 13.8%, within normal spread). This is a front-door problem — hiring fit and early-tenure comp — not a failing team.

**Highest ongoing risk:** the first-year cohort, org-wide. **Recommended next:** (1) tighten screening + first-90-day onboarding — early involuntary exits skew to skills-mismatch/PIP, a hiring-quality signal; (2) run an early-tenure comp positioning check, since compensation is the #1 voluntary reason and would confirm or kill the comp hypothesis.

## Methodology appendix

**Definitions used:** Attrition Rate = leavers in window ÷ average headcount × (12 ÷ window months); average headcount = mean of the 12 month-end active counts. Voluntary/involuntary split by `termination_type`. Internal transfers excluded from leavers (none present in this data). — Peoplenometry metrics library v1.0; no house overrides.

**Formula + numbers:** 486 leavers ÷ avg HC 3,131 × (12/12) = **15.5%**. Voluntary: 274 ÷ 3,131 = **8.8%**. Involuntary: 212 ÷ 3,131 = **6.8%**. Month-end headcounts ranged 2,782 (Jan) → 3,477 (Dec).

**Reconciliation to the 29.9% dashboard figure:** the dashboard uses 1,485 total terminations ÷ 4,962 all-time employees = 29.9%. That denominator includes three years of hires and every past leaver, and the numerator spans all three years — it is a cumulative ratio, not an annual rate. Annualizing against average headcount for a single 12-month window yields 15.5%.

**Per-segment (voluntary, annualized):** Product 13.8% (n=34, HC 247) · People & Culture 12.3% (19, 154) · Customer Success 9.1% (22, 240) · Infrastructure 9.1% (20, 219) · Legal 9.1% (10, 110) · Engineering 8.7% (98, 1121) · Marketing 8.3% (19, 228) · Data Science 7.3% (13, 177) · Sales 6.8% (26, 383) · Finance 5.8% (10, 172) · Operations 3.9% (3, 78). None reach 1.5× the 8.8% company voluntary rate at the required sample size.

**Tenure at exit (all leavers):** <6mo 165 · 6–12mo 110 · 1–2yr 105 · 2–3yr 80 · 3yr+ 26.

**Caveats:** exit reasons are self-reported codes (correlation, not cause); survivorship (we see who left, not the counterfactual); comp is a lead, not a proven driver, until a compa-ratio analysis runs.

**To strengthen this, add:** compa-ratio / band position at hire and at exit; new-hire 90-day survival by source/recruiter; engagement scores for the first-year cohort.
