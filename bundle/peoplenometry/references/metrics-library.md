# Peoplenometry Metrics Library
**Library version: 1.1 (2026-07)**

The source of truth for how these skills calculate things. Every skill reads this before touching data and computes metrics **exactly** as defined here — calculations are never left to chance. Your company's specific choices (confirmed as you work) live in a separate `house-rules.md` and **override** the defaults below; this file stays canonical.

Each entry: canonical formula · data required · variants · what it's confused with · classic wrong ways · house decisions (things companies genuinely answer differently) · reporting rules. The test every entry must pass: *two different AIs reading it produce the same number from the same file.*

## How this library layers (read this first)
Three layers, in override order:
1. **This file — canonical.** The default definition of every metric. Versioned, never edited locally.
2. **House decisions** (`house-rules.md → ## House decisions`) — your company's confirmed answers to the "House decisions" questions below. They **override** the defaults (scope, codes, windows) without changing the formulas.
3. **Local metrics** (`house-rules.md → ## Local metrics`) — full entries, in this file's format, for metrics your company uses that aren't canonical yet. Skills read them exactly like entries here. They're created by the learning loop: when a skill computes a "pending" metric and you confirm the method, it offers to save the definition. Each carries a `confirmed:` date.

The result: the library gets more accurate and more *yours* the more the pack is used — without ever forking the canonical file.

---

## Headcount (point-in-time)
**Aliases:** active headcount, HC on date · **Category:** Headcount & Workforce · **Core**

### Canonical formula
Headcount on date D = count of **in-scope** employees where
`hire_date ≤ D AND (termination_date is empty OR termination_date > D)`.
Count distinct people, not rows (an event log has many rows per person).

### Data required
`employee_id, hire_date, termination_date` (empty for actives), `employee_type`.

### Not the same thing
Headcount ≠ number of rows. Point-in-time HC ≠ average HC (use the average as the denominator for rates).

### Classic wrong ways
1. Using "current active rows in the file" as headcount for a *past* date.
2. Counting event rows instead of distinct in-scope people.
3. Filtering on `hire_date` only and leaving terminated people in.

### House decisions (confirm once → house-rules.md)
- **Scope:** are contractors / interns / fixed-term / employees on leave (LOA) counted? (default: regular employees; LOA counted as active)

### Reporting rules
State the date and scope with any headcount ("142 regular employees active on 2026-06-30, contractors excluded").

---

## Average Headcount
**Category:** Headcount & Workforce · **Core** — the building block for every rate

### Canonical formula
Average headcount over a window = **mean of month-end point-in-time headcounts** across the window (compute Headcount on each month-end, then average).
Fallback, *only* if month-ends can't be computed: `(HC at window start + HC at window end) ÷ 2`.

### Classic wrong ways
1. Using start- or end-of-period headcount alone — distorts any growing/shrinking org.
2. Averaging just two endpoints when monthly data exists (hides mid-period swings).

### Reporting rules
Show the denominator used ("avg HC 138 across 12 month-ends").

---

## Attrition Rate
**Aliases:** turnover rate, termination rate, churn · **Category:** Attrition & Retention · **Core**

### Canonical formula
Attrition Rate (annualized) = ( Leavers in window ÷ Average headcount over window ) × ( 12 ÷ window length in months )

- **Leavers:** employees whose `termination_date` falls within [window start, window end], inside headcount scope, **excluding internal transfers and entity changes recorded as terminations**. Count termination *events*: a rehired employee who left twice is two events — note it if it moves the number.
- **Average headcount:** see Average Headcount entry.
- **Window:** default trailing 12 months (annualization factor = 1).
- **If window < 6 months:** annualization amplifies noise — always report the raw period rate and leaver count alongside the annualized figure.

### Data required
`employee_id, hire_date, termination_date` (empty for actives), `employee_type`; `termination_reason/type` for variants.
**Precondition — the file must include leavers.** An actives-only export cannot produce an attrition rate — say so and stop; do not approximate.

### Variants
| Variant | What changes | When to use |
|---|---|---|
| Voluntary | numerator = voluntary terms only | culture/retention — usually what execs mean |
| Involuntary | numerator = company-initiated terms | perf mgmt, RIFs; report separately, never blend silently |
| First-year (new-hire) | **cohort math:** of employees hired in period P, % gone within 365 days of hire. Denominator = the hire cohort, NOT average HC | onboarding / recruiting quality |
| Regrettable | voluntary terms meeting the house "regrettable" definition | leadership reporting |
| Rolling 12-month | same formula, trailing-12 window recomputed monthly | trend lines (preferred over monthly) |

### Not the same thing
**Retention Rate ≠ 1 − Attrition Rate** (see Retention). In a growing org they diverge — compute separately.

### Classic wrong ways (check every run)
1. Leavers ÷ *current* headcount — understates when growing, overstates when shrinking.
2. Annualizing a 1–3 month window and presenting it as "the rate."
3. Counting internal transfers / entity changes as terminations (Workday and SAP exports do this).
4. Using an export that drops people hired-and-terminated inside the window.
5. Blending voluntary and involuntary so a layoff reads as a culture problem.
6. Deduping a rehire as a "duplicate row," deleting a real termination event.

### House decisions (confirm once → house-rules.md)
- Scope: contractors / interns / fixed-term counted?
- Which `termination_reason` codes map to **voluntary**?
- **Regrettable** = ? (top-rated leavers · manager-flagged · not tracked)
- Does the company already report an **official attrition number**? If yes: match its method for the headline; reconcile against this canonical method in the appendix.

### Reporting rules
Always show numerator and denominator ("18 leavers / avg HC 142 → 12.7%"). Segments with avg HC < 20 or < 5 leavers: report, label small-sample, **never headline**.

---

## Retention Rate
**Category:** Attrition & Retention · **Core**

### Canonical formula
Of employees in-scope and active at window **start**, the % still active (not terminated) at window **end**. Cohort-based: the denominator is a fixed group (everyone active at the start), NOT average headcount.

### Not the same thing
**Retention ≠ 1 − Attrition Rate.** Attrition's numerator includes people hired *and* gone within the window; retention's cohort excludes anyone hired after the start. Never derive one from the other.

### Classic wrong ways
1. Reporting "retention = 100% − attrition%" (wrong whenever there's mid-period hiring).
2. Letting people hired during the window into the retention denominator.

### Reporting rules
State the cohort ("of 120 active on 2025-07-01, 104 still active on 2026-07-01 → 86.7% retention").

---

## Tenure (derived field)
**Category:** Headcount & Workforce · **Core** — used for segmentation, rarely reported alone

### Canonical formula
Tenure = ( reference_date − hire_date ), in years.
- **Active employees:** reference_date = as-of / analysis date.
- **Leavers:** reference_date = `termination_date` (tenure at exit).
Be explicit which you're using; mixing them corrupts tenure-band analysis.

### Rehire handling
Multiple hire/term spans (a rehire) → tenure of the **current/most-recent span** by default. Note if cumulative tenure would change a conclusion.

### Default tenure bands (segmentation)
0–6mo · 6–12mo · 1–2yr · 2–5yr · 5yr+ (companies may set their own → house decision).

### Classic wrong ways
1. Using today's date for a leaver's tenure (inflates it).
2. Ignoring rehire gaps and counting the first hire date only.

---

## Pending entries
Not yet canonical. Until an entry lands here, Peoplenometry computes these with its method stated inline, flags "library entry pending," and — once you confirm the method — offers to save it as a Local metric in your house-rules. The guardrails below are binding even while pending.

- **Pay Gap (raw AND adjusted)** — *Adjusted* = the gap remaining after controlling for level, role, location, tenure. Raw averages alone are the classic mistake — never report the raw gap as "the gap."
- **Compa-Ratio** — salary ÷ range midpoint.
- **Promotion Rate (+ by-demographic)** — employees promoted in window ÷ average headcount; needs a promotion event or level-change history, not just current level.
- **Span of Control** — direct reports per manager from the manager field; report the distribution + median, flag spans of 1–2 and 12+. Orphaned/circular manager references are a data-quality finding, not a span.
- **Representation** — % of in-scope headcount by group, per level/dept, on a stated date. Suppress any group under n = 5 (report "suppressed, n<5", never the value). Composition, not cause.
- **Internal Mobility Rate** — internal moves (dept/role change without exit) ÷ average headcount. Moves are never terminations; never let transfer events leak into attrition.
- **Time to Fill / Time to Hire** — clock definition is a house decision (req-approval vs req-open → offer-accept vs start).
- **Funnel Pass-Through Rate** — stage-to-stage conversion.
- **Offer Acceptance Rate**.
- **eNPS** — %(9–10) − %(0–6). **Never the average score.**
- **Survey Participation Rate + small-group suppression** — never report a group under the suppression floor (default n < 5).
