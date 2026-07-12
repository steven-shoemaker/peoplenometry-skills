

> Part of the **Peoplenometry** skill. Invoked by SKILL.md when the task calls for it.

# Attrition deep dive — playbook

Your VP asks "why is attrition up in engineering?" and you've got a pivot table and dread. This turns your export into a real answer: how bad it actually is, where it's concentrated, what's likely driving it, which teams are at risk next — written the way your CHRO needs to hear it.

**Needs analysis-ready data.** If the export is messy, run `the data-cleaning playbook (cleaning-data.md)` first — attrition math is very sensitive to bad dates, duplicate rows, and rehires.

## What it produces (two layers)
1. **The exec narrative** (first): the headline attrition rate vs a fair baseline, the 2–3 segments driving it, the reasons the data supports, and recommended next moves — in five sentences a CHRO will act on.
2. **The methodology appendix** (second): the definitions used, formulas with the actual numbers, per-segment denominators, the caveats, and what data would make it stronger. This is what makes it survive scrutiny.

## 0. Orient — load definitions before touching data
1. Read `metrics-library.md`. This skill uses **Attrition Rate (+ voluntary/involuntary/first-year/regrettable variants), Average Headcount, Retention Rate, Tenure** — compute them **exactly** as defined there. If the library isn't found (partial install), use the inline fallback at the bottom and tell the user the library is missing.
2. Read `.Peoplenometry/house-rules.md` (cwd), else `~/.people-analytics/house-rules.md`, if present. **House rules override library defaults.** Echo what you're applying: *"Using your house rules: contractors excluded; voluntary = Resignation + Job Abandonment."* Never re-ask a question house-rules already answers. If house-rules contains **Local metrics** relevant to this analysis (e.g. a company-specific regrettable or mobility definition), apply them exactly like library entries and cite them in the appendix.
3. Any "House decision" the library flags for these metrics that house-rules doesn't answer → fold into the framing questions below.

## 1. Frame the question — ask, don't assume
One `AskUserQuestion` call, **max 4 questions, ≤4 options each** (skip any answered by house-rules; if unavailable, ask as a short numbered list):
- **"What time window?"** — *last 12 months · this year to date · last 24 months · a specific range.*
- **"Which kind of leaving?"** — *voluntary (quits) · involuntary (layoffs) · both · the data doesn't distinguish.* (If the clean step found no vol/invol flag, say so and note the limit.)
- **"Where to look first?"** — *the whole org · a department · by manager · by location.*
- **"Does your company already report an official attrition number?"** — *yes (I'll match its method) · no · not sure.* (House decision — save it.)

## 2. Analyze — per the metrics library
- **Compute attrition exactly per the library** (leavers ÷ average headcount, annualized). Show numerator and denominator.
- **Sanity-check before you headline.** Typical annual attrition is ~8–20%. If the number comes out implausibly high (>25%) or low (<3%), treat it as a likely **data-completeness problem first** — usually the file is missing active employees, the window is wrong, or leavers are double-counted. Flag it, say what to check, and do not present an implausible rate as settled fact.
- **Baseline (do not hallucinate one):** for the headline, use the **org's own prior equivalent period**; for segments, use the **org overall rate**. If neither is computable, the baseline slot reads *"first measurement — no fair baseline yet."* **Never** an external/industry benchmark unless the user supplies it.
- **If the company reports an official number** (from Q4), match its method for the headline and reconcile the difference in the appendix — being right but off their board number loses the room.
- **Volunteer the reconciliation — don't wait to be asked.** Naive dashboards commonly divide *all terminations ÷ everyone ever employed* (a cumulative, multi-year ratio) or use a single point-in-time headcount — either can run ~2× the honest annualized rate. If your computed rate diverges materially from a cruder figure the user cited or is plausibly staring at, **say both numbers, name the method difference in one sentence, and show the reconciliation in the narrative itself** (not only the appendix). Quietly being right while their dashboard screams a scarier number loses the room.
- **Screen out internal transfers / entity-change** codes from leavers (per the library's wrong-way #3).
- **Segment** by department, tenure band, job level, manager, location. Flag a segment as elevated only if **rate ≥ 1.5× baseline AND ≥ 5 leavers AND avg HC ≥ 20**; below that, report it but label small-sample and never headline it. **If NO segment clears the bar (common under ~500 employees), do not headline the top department — say plainly that no team has enough leavers to call out reliably, and lead the narrative with the tenure curve or the org-level pattern instead.**
- **Tenure curve** — *when* do people leave? (0–6mo: onboarding/role-fit/bad hiring · 12–24mo: growth or comp ceiling · long-tenure: different story.) Often the most diagnostic view.
- **Cross with what's available** (engagement, comp/level, promotion history, manager) — **correlation only**, say so.
- **At-risk read:** name the *segments* where the pattern predicts continued loss. Elevated risk, not prophecy — and **never name individuals** (legal exposure for your customer).

## 3. Guardrails — state these, don't skip
- Correlation is not cause. · Small segments are noise — always show the denominator. · Survivorship — you see who left, not the counterfactual. · No prediction theater ("elevated risk," never a fake probability). · **Never name an individual as a flight risk.**

## 4. Write it up
**Narrative first:**
> Attrition over [window] ran **X%** annualized (vs [baseline]). It's concentrated in **[segment]** (**Y%**, n=Z) and among **[tenure band]**. The data most supports **[driver]** [with the caveat that …]. Highest ongoing risk: **[segment]**. Recommended next: **[1–2 concrete moves].**

**Then the methodology appendix (fill every slot):**
> **Definitions used:** [Attrition Rate = leavers ÷ avg HC × annualization; scope: …] — Peoplenometry metrics library v2.0; house overrides + local metrics: [list].
> **Formula + numbers:** [18 leavers / avg HC 142 → 12.7% annualized].
> **Per-segment table:** segment · rate · leavers (n) · avg HC.
> **Caveats:** [what's correlational, small samples, missing data].
> **To strengthen this, add:** [exit-reason data / engagement scores / comp bands].

## 5. Capture what you learned
If the user confirmed definitions or nuances this run (voluntary codes, regrettable definition, scope, official-number method), **offer once at the end** to save them: show the **exact lines** to append to `house-rules.md`, one `AskUserQuestion` — *Save all · Let me pick · Don't save*. Append-only; never write silently; if a new rule contradicts an existing one, surface both and ask which wins. If a *method* was confirmed that the library doesn't define (not just a house decision), save it as a full entry under `## Local metrics` in library format with a `confirmed: <date>` line — the same learning loop `Peoplenometry` uses.

## 6. Save the result (so it can become a report)
Write the full two-layer output to `people-analytics/analysis-attrition-<YYYY-MM-DD>.md` so `the report playbook (writing-reports.md)` can render it later. Frontmatter uses schema `result_format: 1` — keep to exactly these fields:
```yaml
---
result_format: 1
skill: why-are-people-leaving
date: <YYYY-MM-DD>
question: <the framed question>
window: <the analysis window>
metrics:
  - name: Attrition Rate (annualized)
    value: <e.g. 12.7%>
    numerator: <leavers>
    denominator: <avg headcount>
    n: <leavers>
  # one entry per computed figure (segment rates, tenure bands, etc.)
house_rules_applied: [<list>]
caveats: [<list>]
---
```
The markdown body below the frontmatter = the exec narrative + methodology appendix, verbatim.

### Close
*"Done. Want me to turn this into a leadership-ready report? Run `the report playbook (writing-reports.md)` — it builds a self-contained HTML/PDF you can send up, with the methodology attached so it holds up. Or I can go deeper on [the hottest segment]."*

---
> **Inline fallback** (only if the metrics library is missing): Attrition Rate = leavers in window ÷ average monthly headcount × (12 ÷ window months); baseline = the org's own prior period, never external; exclude transfers; flag segments ≥1.5× baseline with ≥5 leavers and avg HC ≥20.
