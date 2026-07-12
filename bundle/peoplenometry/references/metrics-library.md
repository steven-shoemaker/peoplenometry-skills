# Peoplenometry Metrics Library
**Library version: 2.0 (2026-07)**

The source of truth for how these skills calculate things. Every skill reads this before touching data and computes metrics **exactly** as defined here — calculations are never left to chance. Your company's specific choices (confirmed as you work) live in a separate `house-rules.md` and **override** the defaults below; this file stays canonical.

Each entry: canonical formula · data required · variants · what it's confused with · classic wrong ways · house decisions (things companies genuinely answer differently) · reporting rules. Mechanical metrics get concise entries (formula · gotchas · reporting rule). The test every entry must pass: *two different AIs reading it produce the same number from the same file.*

Some entries are marked **Composite / house-defined**: they are models, survey composites, or business rubrics — not a single arithmetic formula. For these, the library defines what the metric measures, the common construction, and the traps; the exact formula must come from the house (survey vendor, model owner, or a confirmed house rule). Skills never invent a formula for these.

## How this library layers (read this first)
Three layers, in override order:
1. **This file — canonical.** The default definition of every metric. Versioned, never edited locally.
2. **House decisions** (`house-rules.md → ## House decisions`) — your company's confirmed answers to the "House decisions" questions below. They **override** the defaults (scope, codes, windows) without changing the formulas.
3. **Local metrics** (`house-rules.md → ## Local metrics`) — full entries, in this file's format, for metrics your company uses that aren't canonical yet. Skills read them exactly like entries here. They're created by the learning loop: when a skill computes a "pending" metric and you confirm the method, it offers to save the definition. Each carries a `confirmed:` date.

The result: the library gets more accurate and more *yours* the more the pack is used — without ever forking the canonical file.

## Global conventions (apply to every entry)
- **Scope** means the headcount scope confirmed in house rules (default: regular employees; contractors/interns excluded; LOA counted as active). Every metric inherits it unless its entry says otherwise.
- **Windows** are inclusive of both endpoints. Default reporting window: trailing 12 months. Default as-of date: the latest date the data supports, stated explicitly.
- **Count distinct people (or distinct events, where the entry says events), never rows.** Exports are often event logs with many rows per person.
- **Small-sample rules:** any demographic breakdown suppresses groups with n < 5 ("suppressed, n<5" — never the value). Any *rate* for a segment with average HC < 20 or < 5 numerator events is reported, labeled small-sample, and never headlined.
- **Medians over means** for durations and money unless the entry says otherwise; distributions are skewed.
- **Data source flags:** entries note when the metric needs a system beyond a standard HRIS/workforce export (ATS, LMS, survey platform, goal system, compliance system, comp planning, finance). If the required source isn't in the data provided, the skill says "you'd need X" and stops — it never approximates from proxy fields.
- **Never name individuals** in outputs for sensitive metrics (PIP, flight risk, performance, pay equity). Aggregate only.

---

# 1 · Headcount & Workforce

## Headcount (point-in-time)
**Aliases:** active headcount, actual headcount, HC on date · **Core**

### Canonical formula
Headcount on date D = count of **in-scope** employees where
`hire_date ≤ D AND (termination_date is empty OR termination_date > D)`.
Count distinct people, not rows (an event log has many rows per person).

### Data required
`employee_id, hire_date, termination_date` (empty for actives), `employee_type`.

### Not the same thing
Headcount ≠ number of rows. Point-in-time HC ≠ average HC (use the average as the denominator for rates). Headcount ≠ FTE (see Cost per FTE — FTE sums fractional appointments).

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
**Core** — the building block for every rate

### Canonical formula
Average headcount over a window = **mean of month-end point-in-time headcounts** across the window (compute Headcount on each month-end, then average).
Fallback, *only* if month-ends can't be computed: `(HC at window start + HC at window end) ÷ 2`.

### Classic wrong ways
1. Using start- or end-of-period headcount alone — distorts any growing/shrinking org.
2. Averaging just two endpoints when monthly data exists (hides mid-period swings).

### Reporting rules
Show the denominator used ("avg HC 138 across 12 month-ends").

---

## New Hires
**Aliases:** hires, new starts · **Concise**

**Formula:** count of in-scope employees with `hire_date` in [window start, window end]. Count hire *events*: a rehire is a new hire event (flag rehires separately if the field exists).
**"New starts" vs "new hires":** treated as the same metric by default. If the company distinguishes them (e.g., starts = external hires + internal transfers-in, or hires counted at offer accept vs first day), that's a house decision — confirm before reporting both words as one number.
**Gotchas:** (1) An export filtered to current actives undercounts hires — anyone hired *and* terminated inside the window disappears. Check for this before reporting. (2) Don't count conversion events (contractor→FTE, entity change) as hires unless house rules say to.
**Reporting rule:** state window and scope; note rehires if they move the number ("31 hires in FY26 H1, incl. 2 rehires").

---

## Terminations
**Aliases:** exits, leavers, terms · **Concise**

**Formula:** count of termination *events* where `termination_date` falls in [window start, window end], in-scope, **excluding internal transfers and entity changes recorded as terminations** (Workday and SAP exports produce these). This is the numerator of Attrition Rate — the same exclusions apply.
**Gotchas:** (1) A rehired employee who left twice is two events. (2) Future-dated terminations (notice given, last day after window end) are not terminations yet — exclude and note.
**Reporting rule:** report alongside the voluntary/involuntary split whenever `termination_reason` exists; a raw terms count with a layoff inside it misleads.

---

## Net Headcount Change
**Aliases:** net adds, headcount growth · **Concise**

**Formula:** `HC at window end − HC at window start` (same scope, both point-in-time per the Headcount entry).
**Reconciliation check (always run):** net change should equal `hires − terminations` within the window and scope. If it doesn't, the file has scope drift, transfers counted as hires/terms, or missing events — report the discrepancy as a data-quality finding, don't silently pick one number.
**Reporting rule:** show all three ("+13 net: 31 hires − 18 terms; reconciles").

---

## Employee Type Breakdown
**Concise**

**Formula:** distribution of point-in-time Headcount on date D by `employee_type` (regular / contractor / intern / fixed-term …). Percentages of total in-scope-plus-listed-types; must sum to 100%.
**Gotchas:** (1) This is the one headcount view where out-of-scope types *are* shown — say so, since every other metric excludes them. (2) Blank/unknown type is its own category, not silently dropped.
**Reporting rule:** state date; show counts and percentages.

---

## Geographic Distribution
**Concise**

**Formula:** distribution of point-in-time Headcount on date D by location field (country default; site/state/region if asked). Percentages sum to 100%; blank/unknown is its own category.
**Gotchas:** (1) "Location" fields vary — work location ≠ legal entity country ≠ home address. State which field you used. (2) Remote workers may carry an office code they never visit.
**House decision:** which field is the canonical location.
**Reporting rule:** state date and the field used.

---

## Job Family Distribution
**Concise**

**Formula:** distribution of point-in-time Headcount on date D by `job_family` (or nearest equivalent: job function, department if no family field). Percentages sum to 100%.
**Gotchas:** blank/unmapped job family is a Job Architecture Coverage finding (see §10), not a category to hide.
**Reporting rule:** state date and the field used; if you substituted department for family, say so.

---

## Tenure (derived field)
**Core** — used for segmentation, rarely reported alone

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

# 2 · Attrition & Retention

## Attrition Rate
**Aliases:** turnover rate, termination rate, churn · **Core**

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
| First-year (new-hire) | see the dedicated New-Hire Attrition entry — **cohort math, different denominator** | onboarding / recruiting quality |
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
- Which `termination_reason` codes map to **voluntary**? (Ambiguous codes — "mutual agreement," "end of contract," "job abandonment" — must be assigned by the house, not guessed.)
- **Regrettable** = ? (top-rated leavers · manager-flagged · not tracked)
- Does the company already report an **official attrition number**? If yes: match its method for the headline; reconcile against this canonical method in the appendix.

### Reporting rules
Always show numerator and denominator ("18 leavers / avg HC 142 → 12.7%"). Segments with avg HC < 20 or < 5 leavers: report, label small-sample, **never headline**.

---

## Voluntary / Involuntary Attrition
**Core** — the split that changes the story

### Canonical formula
Same as Attrition Rate; numerator restricted by `termination_reason/type` mapped per house rules to **voluntary** (employee-initiated: resignation, retirement by default) or **involuntary** (company-initiated: performance termination, layoff/RIF, end of assignment by default).

### Classic wrong ways
1. Guessing the mapping for ambiguous codes ("mutual," "job abandonment," "end of contract") — these are house decisions.
2. Reporting voluntary alone in a period containing a layoff without stating the involuntary number next to it.
3. Letting retirement inflate "voluntary" in an aging workforce without noting it — flag if retirements are >20% of voluntary terms.

### Reporting rules
Voluntary + involuntary + total must reconcile (unmapped codes reported as "unclassified," never silently binned). Show all three rates together.

---

## New-Hire (First-Year) Attrition
**Aliases:** first-year attrition, 90-day/first-year turnover · **Core** — cohort math, not the standard formula

### Canonical formula
Of employees **hired in cohort period P**, the % terminated within **365 days of their own hire date**.
`first_year_attrition(P) = leavers within 365d of hire ÷ hires in P`
- Denominator = the hire cohort. **Not average headcount.** This is the defining difference from the standard rate.
- Only cohorts whose members have all had a full 365 days of exposure are complete. For recent cohorts, report the *observed-so-far* rate labeled "incomplete — X months of exposure," or use a shorter fixed horizon (90-day) that is complete.

### Data required
`hire_date, termination_date` per person; the file must include leavers.

### Not the same thing
Not "attrition rate filtered to tenure < 1yr" — that mixes denominators and double-counts exposure. It's a survival question about a hire cohort.

### Classic wrong ways
1. Dividing first-year leavers by average HC (produces a meaningless small number).
2. Comparing an incomplete recent cohort against complete older cohorts without labeling exposure.
3. Excluding people hired-and-gone inside the window because the export dropped them — this metric is *made of* those people.

### House decisions
- Horizon: 365 days default; some houses track 90-day separately.
- Do involuntary first-year terms count? (default: yes, but show the split — first-year involuntary usually means hiring-quality issues, first-year voluntary usually means onboarding/expectation issues.)

### Reporting rules
Name the cohort and horizon ("of 40 hired in FY25, 9 gone within 365 days → 22.5% first-year attrition"). Label incomplete cohorts.

---

## Regrettable Attrition
**Core** — house-gated variant

### Canonical formula
Attrition Rate with numerator = voluntary leavers meeting the house **"regrettable"** definition. There is **no default definition** — common constructions: leavers with top performance ratings; manager-flagged at exit; leavers from critical roles. Until the house confirms one, this metric is not computable — say so rather than substituting "voluntary."

### Classic wrong ways
1. Treating all voluntary attrition as regrettable.
2. Using a performance-rating proxy without confirming ratings exist and are trusted.

### House decisions
- The definition itself (this *is* the metric). Record it verbatim in house-rules.md.

### Reporting rules
State the definition in the output every time ("regrettable = voluntary + last rating ≥ 4").

---

## Retention Rate
**Core**

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

# 3 · Mobility & Career Progression

## Internal Mobility Rate
**Core**

### Canonical formula
Internal Mobility Rate (annualized) = ( employees with ≥1 qualifying internal move in window ÷ Average headcount over window ) × ( 12 ÷ window months )
- **Qualifying move (default):** a change of department, job code, or role **without a termination** — includes promotions, lateral moves, and transfers. (Report Promotion Rate and Lateral Move Rate separately for the split.)
- **Count distinct people by default**, not move events. If someone moved twice, note it; switch to events only as a confirmed house decision.

### Data required
A job/position **event history** (effective-dated job changes) or at minimum two snapshots to diff. A single current-state snapshot **cannot** produce this metric — say so.

### Not the same thing
Moves are never terminations. If transfer events appear as term+hire pairs in the export, they belong here — and must be *removed* from attrition (see Attrition classic wrong way #3).

### Classic wrong ways
1. Computing from a single snapshot (impossible; produces zero or garbage).
2. Counting compensation-only or title-cosmetic changes as moves.
3. Counting a re-org (department renamed/split over everyone's head) as thousands of individual moves — mass same-day changes affecting a whole unit are a re-org, not mobility; flag and exclude.

### House decisions
- What counts as a move (dept change? job code change? manager change alone does NOT count by default).
- People vs events counting.

### Reporting rules
Show numerator basis ("14 employees moved / avg HC 142 → 9.9% annualized"). State whether promotions are included (default: yes).

---

## Promotion Rate
**Core**

### Canonical formula
Promotion Rate (annualized) = ( employees with ≥1 promotion event in window ÷ Average headcount over window ) × ( 12 ÷ window months )
- **Promotion (default):** an upward change in job level/grade recorded as an event, or a level-field increase between effective-dated records.
- Distinct people, not events.

### Data required
Promotion events or a **job-level history**. Current level alone is not enough — you cannot see who changed. If only current level exists, say so and stop.

### Not the same thing
Not Internal Promotion Fill Rate (that's a % of *filled requisitions*, see §7). Not a lateral move (no level change). A promotion accompanying a transfer counts once, as a promotion.

### Classic wrong ways
1. Inferring promotions from title-string changes (retitling ≠ promotion).
2. Using end-of-period headcount as the denominator.
3. Counting level *corrections* (data cleanup) as promotions — same-day mass level changes are suspect.

### House decisions
- Which event/reason codes mean promotion; whether in-level compensation promotions ("promotion in place") count.

### Reporting rules
Show numerator/denominator. For by-demographic promotion rates, see §9 — additional suppression and comparison rules apply.

---

## Time in Role
**Concise**

**Formula:** for each employee, `as_of_date − start date of current role` (the effective date of their most recent job/role change; hire date if never changed). Report the **median** in months, plus distribution.
**Data required:** job-change history; a snapshot with a "job entry date" field also works — state which you used.
**Gotchas:** (1) Mean is skewed by long-tenured incumbents — median only. (2) Re-org renames reset the clock falsely; treat mass same-day changes as non-events if detectable.
**Reporting rule:** state the field/logic used ("median 19 months in current role, from job_entry_date").

---

## Time Since Last Promotion
**Concise**

**Formula:** for each employee with ≥1 promotion on record, `as_of_date − most recent promotion effective date`, median in months. Employees never promoted are a **separate group** reported as "never promoted, median tenure X" — never blended in using hire date, which silently mixes two populations.
**Data required:** promotion event history (same precondition as Promotion Rate).
**Gotcha:** history exports often only go back a few years — a truncated history makes veterans look never-promoted. Check the earliest event date and caveat.
**Reporting rule:** report the two groups separately.

---

## Lateral Move Rate
**Concise**

**Formula:** ( employees with ≥1 lateral move in window ÷ average headcount ) × annualization. **Lateral move:** a job/department change with **no level change** and no termination.
**Data required:** job event history including level.
**Gotcha:** requires level data to separate from promotions; if level is missing, you can only compute total Internal Mobility, not the lateral/promotion split — say so.
**Reporting rule:** report alongside Promotion Rate; the two plus demotions should sum to Internal Mobility (by events).

---

## Manager Change Frequency
**Concise**

**Formula:** over the window, per in-scope employee: count of changes in the `manager_id` field (from an effective-dated history). Report the **median count** and the **% of employees with ≥2 manager changes** in the window.
**Data required:** manager-field history or successive snapshots. A single snapshot cannot produce this.
**Gotchas:** (1) A re-org or a manager's own departure changes `manager_id` for whole teams at once — that's organizational churn, still real, but distinguish "manager left/re-org" from "employee moved" where reason codes allow. (2) Interim/acting manager assignments can double-count a single transition.
**Reporting rule:** state the window and that mass events are included/excluded.

---

# 4 · Leadership & Org Design

## Span of Control
**Core**

### Canonical formula
For each manager (any in-scope employee with ≥1 in-scope **direct** report on date D, per the `manager_id` field): span = count of direct reports on D.
Report the **median span and the distribution** — never the average alone. Flag spans of 1–2 (fragmented) and 12+ (overloaded) as findings.

### Data required
`employee_id, manager_id` on a stated date; scope fields.

### Not the same thing
Direct span ≠ total org size under a leader (rollup). "Manager" here = has direct reports in the data — not the job title, not a "people manager" flag (use the flag only to cross-check).

### Classic wrong ways
1. Averaging spans and headlining the mean (one exec assistant pool distorts it).
2. Counting out-of-scope workers (contractors) as reports — state whether they're included; default follows headcount scope.
3. Treating orphaned records (manager_id blank or pointing to a terminated/absent employee) or circular chains as spans — these are **data-quality findings, not spans**. Report the count of orphans separately.

### House decisions
- Do contractors/interns count as reports? Do open positions count (position-based systems)?

### Reporting rules
"Median span 6 across 24 managers; 3 managers with span ≥12; 2 orphaned records excluded."

---

## Management Ratio
**Aliases:** manager ratio, IC-to-manager ratio · **Concise**

**Formula:** ( count of managers on date D ÷ total in-scope headcount on D ) × 100. Manager = ≥1 direct report in the data (same definition as Span of Control).
**Alias form:** IC-to-manager ratio = ( HC − managers ) ÷ managers — report as "X ICs per manager." State which form you're using; they're transforms of each other but read very differently.
**Gotcha:** player-coaches (managers who also hold IC work) are managers here; the data can't see the split.
**Reporting rule:** state date, form, and manager definition.

---

## Layers from CEO
**Aliases:** org depth, reporting layers · **Concise**

**Formula:** for each employee, walk the `manager_id` chain to the top. The employee with no manager (or self-reporting) at the top = **layer 1** (the CEO); each step down adds one. Report the **maximum depth** and the **distribution of employees by layer**.
**Gotchas:** (1) Broken chains (orphans, cycles) make depth uncomputable for those employees — report them as a data-quality count, not layer 0. (2) Multiple roots (subsidiaries, board members in the file) — confirm which root is "the CEO." (3) Dotted-line/matrix reporting isn't in a single manager field; this metric is solid-line only.
**Reporting rule:** "7 layers max; 60% of employees at layers 3–4; 3 records with broken chains."

---

## Managerial Effectiveness Score
**Aliases:** manager effectiveness score (Employee Experience list — same metric) · **Composite / house-defined**

**What it measures:** how well managers manage, almost always from the **manager-effectiveness section of an engagement survey** (upward feedback items: "my manager gives useful feedback," etc.), sometimes blended with team outcomes (team retention, engagement delta).
**Common construction:** mean or favorable-% of a fixed item set per manager, aggregated. The item set, scale, favorability cutoffs, and any blending weights come from the **survey vendor or house** — there is no canonical arithmetic. Skills must obtain the construction before computing; never invent an item set or average unlabeled survey columns.
**Data source:** survey platform export — **not in a standard HRIS export**.
**Traps:** (1) Never report a score for a manager with < 5 respondents (identifiability + noise). (2) Never rank-order managers publicly from survey scores. (3) A change in the item set between cycles breaks trend comparability — check before trending.
**Reporting rule:** state the instrument, cycle, item count, and response floor with any score.

---

## Succession Coverage
**Composite / house-defined**

**What it measures:** whether critical roles have identified, ready successors.
**Common construction:** ( critical roles with ≥1 successor at the required readiness level ÷ total critical roles ) × 100 — but **"critical role"** and **"ready" (ready-now vs ready-1-2yr)** are house/talent-process definitions, and the data lives in a **succession/talent module, not a standard HRIS export**. Both definitions must be confirmed before computing.
**Traps:** (1) A named successor who is also named for three other roles inflates coverage — report unique-successor coverage alongside if the data allows. (2) Coverage says nothing about successor quality or flight risk. (3) Never name successors in generated output.
**Reporting rule:** state both definitions and the as-of date ("of 12 house-flagged critical roles, 8 have a ready-now successor → 67% coverage").

---

# 5 · Performance & Productivity

## Performance Rating Distribution
**Concise**

**Formula:** for a stated review cycle: % of **rated** employees at each rating value. Denominator = employees who received a rating in that cycle. Report **rating coverage** separately: rated ÷ eligible.
**Data source:** performance system export or an HRIS rating field — check the field is cycle-stamped; a "latest rating" field mixes cycles.
**Gotchas:** (1) Never average ordinal ratings as the headline (a 3.2 "average rating" hides the shape — the distribution IS the metric). (2) Ineligible employees (hired after cutoff) don't belong in the denominator.
**Reporting rule:** name the cycle, show coverage ("FY25 cycle: 92% of 130 eligible rated; distribution: 10/25/45/15/5").

---

## PIP Rate
**Concise · sensitive**

**Formula:** ( employees placed on a performance improvement plan in window ÷ average headcount ) × annualization.
**Data source:** performance/ER system — PIP events are **usually not in a standard HRIS export**; a termination reason of "performance" is not a PIP record.
**Gotchas:** (1) Extremely sensitive — aggregate only, never name or make small segments identifiable (suppress segments where numerator < 5). (2) PIP *outcomes* (survived/exited) are a separate analysis; don't conflate rate with outcome.
**Reporting rule:** aggregate figure with window; segment only at department level or higher.

---

## High Performer Retention
**Concise**

**Formula:** Retention Rate (cohort method, §2) restricted to employees carrying the house **"high performer"** flag at window start; always reported **next to the retention rate of everyone else** — the gap is the finding, not the absolute number.
**Data required:** a rating or HiPo flag as of window start (not current rating — survivorship bias) plus termination data.
**Gotchas:** (1) Using *current* ratings selects for survivors and inflates the number. (2) The high-performer definition is a house decision (top rating band? HiPo flag?) — confirm first.
**Reporting rule:** "of 25 top-rated at FY25 start, 22 retained (88%) vs 84% for all others."

---

## Goal Completion Rate
**Concise**

**Formula (default):** ( goals marked complete ÷ goals due in window ) × 100. House alternative: % of employees who completed all their due goals — confirm which the house wants; the two diverge widely.
**Data source:** goal-management system (OKR/goal module) — **not in a standard HRIS export**.
**Gotchas:** (1) Goals with no due date or still open aren't "incomplete" — they're excluded and counted separately. (2) Completion ≠ achievement; a checked box says nothing about quality or stretch.
**Reporting rule:** state the unit (goals vs people) and the window.

---

## Time to Productivity
**Aliases:** ramp time, time to full productivity · **Composite / house-defined**

**What it measures:** how long a new hire takes to reach expected output.
**Common construction:** days from start date to a defined productivity milestone — first sale, quota attainment %, certification completion, manager sign-off. **The milestone definition IS the metric and is entirely house-defined**; there is no canonical formula, and the milestone data lives in sales/ops/LMS systems, **not a standard HRIS export**.
**Traps:** (1) Only roles with a measurable milestone (sales, support) can carry this honestly — don't extend a proxy to all roles. (2) Survivor bias: hires who quit before ramping vanish from the average — report alongside first-year attrition.
**Reporting rule:** state the milestone and role scope; median days, not mean.

---

## Calibration Rate
**Concise**

**Formula (default):** ( employees whose final rating differs from the manager's pre-calibration rating ÷ employees whose ratings went through calibration ) × 100. House alternative: % of ratings reviewed in a calibration session at all — confirm which meaning the house intends before computing; the word is used both ways.
**Data source:** performance system with pre- and post-calibration values — **rarely exported; usually you'd need the perf-system's calibration audit data**.
**Gotcha:** a high change-rate isn't inherently bad (it can mean calibration is working) — report direction of changes (up/down) with the rate.
**Reporting rule:** name the cycle and the definition used.

---

# 6 · Learning & Development

## Training Completion Rate
**Concise**

**Formula:** ( training assignments completed ÷ assignments **due** in window ) × 100. Denominator = assignments with a due date in the window, for in-scope employees — not "all assignments ever created."
**Data source:** LMS export — **not in a standard HRIS/workforce export**. If no LMS data is present, say "you'd need the LMS export" and stop.
**Gotchas:** (1) Compliance-mandatory vs optional training are different metrics — split them; a blended rate hides compliance risk. (2) Assignments to terminated employees inflate the denominator — exclude people who left before the due date.
**Reporting rule:** state which catalog/courses and the due-date window ("mandatory compliance courses due in Q2: 94% completion").

---

## Learning Hours per Employee
**Concise**

**Formula:** total recorded learning hours in window ÷ average headcount over window.
**Data source:** LMS — **not in a standard HRIS export**.
**Gotchas:** (1) Recorded hours = course credit hours, not actual time; self-reported and external learning are usually invisible. (2) The distribution is heavily skewed (a few people take everything) — report the median per-person hours alongside the mean.
**Reporting rule:** state window and that only LMS-recorded hours are counted.

---

## Training Effectiveness Score
**Composite / house-defined**

**What it measures:** whether training changed anything — knowledge, behavior, or results (Kirkpatrick levels 2–4), not just attendance.
**Common construction:** post-training assessment scores, pre/post deltas, learner survey composites, or downstream KPI movement. **Which construction, which instrument, and what "effective" means are house/vendor decisions** — there is no canonical formula. Reaction surveys ("smile sheets") alone measure satisfaction, not effectiveness — label them as such.
**Data source:** LMS + assessment/survey tooling — **not in a standard HRIS export**.
**Reporting rule:** state the instrument and level ("level-2 assessment pass rate," not "training effectiveness") — never present completion rate as effectiveness.

---

## Internal Promotion Fill Rate
**Concise** — see also Internal Fill Rate (§7)

**Formula:** ( filled requisitions filled by an internal candidate **via promotion** ÷ all filled requisitions in window ) × 100. It is a subset of Internal Fill Rate (which also counts lateral internal fills).
**Data source:** ATS/requisition data joined to job-level history — **not in a standard HRIS export**.
**Gotcha:** don't confuse with Promotion Rate (§3): this metric's denominator is *requisitions*, Promotion Rate's is *headcount*. They answer different questions ("do we hire leaders from within?" vs "how often do our people get promoted?").
**Reporting rule:** report next to Internal Fill Rate with both denominators shown.

---

# 7 · Talent Acquisition

**Section precondition:** everything here needs **ATS/recruiting data** (requisitions, candidates, offers, sources, costs). A standard HRIS/workforce export contains none of it — if only HRIS data is present, the skill says "you'd need the ATS export" and stops. All duration metrics: **median calendar days**, stated explicitly (some houses use business days — house decision).

## Time to Fill
**Core**

### Canonical formula
Per filled requisition: `offer_accepted_date − requisition_approved_date`, in calendar days. Report the **median** across requisitions filled in the window (windowed by fill date).

### The clock is the whole metric
Start (req *approved* vs req *opened/posted*) and stop (offer accept vs start date) vary by house. **Default: approved → offer accepted.** Confirm both ends before comparing to any prior report or benchmark — a mismatched clock is the #1 source of "our number changed."

### Not the same thing
- **Time to Hire** starts from the *candidate's* application, not the req.
- **Time to Start** ends at day one, not offer accept.

### Classic wrong ways
1. Mean instead of median (one 300-day evergreen req wrecks it).
2. Including open/cancelled reqs (only *filled* reqs have a fill time; report aging of open reqs separately).
3. Mixing clock definitions across periods when trending.
4. Counting reqs reopened after a declined offer from the original approval date without noting it.

### House decisions
- Clock start and stop; business vs calendar days; how reposted/cloned reqs are handled.

### Reporting rules
"Median 42 calendar days, req-approval → offer-accept, across 18 reqs filled in H1."

---

## Time to Hire
**Core**

### Canonical formula
Per hire: `offer_accepted_date − candidate_application_date`, calendar days, **median** across hires in the window. Measures candidate-experience speed; Time to Fill measures req-level speed. Both, always, never interchanged.

### Classic wrong ways
1. Using the req open date as the start (that's Time to Fill).
2. Sourced/internal candidates with no clean "application date" — state how they're handled (default: first-contact/entered-process date; exclude if absent).

### Reporting rules
State both endpoints and the median ("median 23 days, application → offer accept").

---

## Time to Start
**Concise**

**Formula:** per hire: `start_date − requisition_approved_date`, calendar days, median across hires starting in the window. This is the business's true "how long were we without the person" clock — it includes notice periods that Time to Fill hides.
**Gotcha:** Time to Start − Time to Fill ≈ the notice-period gap; a house that only tracks Time to Fill systematically understates vacancy pain.
**Reporting rule:** state endpoints; report alongside Time to Fill.

---

## Offer Acceptance Rate
**Core**

### Canonical formula
( offers accepted ÷ offers extended ) × 100, windowed by **offer-extended date**; an offer extended in June and accepted in July belongs to June.
- Offers **rescinded by the company** are excluded from the denominator (default). Offers still pending at analysis time are excluded and counted separately.

### Data required
Offer events with extended/accepted/declined/rescinded status — ATS.

### Classic wrong ways
1. Windowing by accept date (drops decliners; inflates the rate).
2. Counting verbal + written stages of one offer as two offers.
3. Including pending offers in the denominator.

### House decisions
- Are rescinded offers excluded (default) or counted as non-accepts? Renegotiated offers = one offer or two?

### Reporting rules
"31 of 36 offers extended in H1 accepted (2 pending excluded, 1 rescinded excluded) → 86%."

---

## Cost per Hire
**Concise**

**Formula (SHRM/ANSI standard):** ( total internal recruiting costs + total external recruiting costs, for the period ) ÷ hires in the period. External: agency fees, job boards, ATS/tools, background checks, travel, sign-on bonuses (house decision). Internal: recruiting-team comp, referral bonuses.
**Data source:** finance/recruiting budget data + ATS hire counts — **cost data is never in an HRIS or ATS export alone; you'd need finance inputs**.
**Gotchas:** (1) State exactly which cost buckets are in — comparability across companies/benchmarks is meaningless otherwise. (2) A period with heavy agency use for one exec search distorts the average — note outliers.
**Reporting rule:** list the included cost buckets with the number.

---

## Candidate Diversity Ratio
**Concise · sensitive**

**Formula:** % of candidates from a stated demographic group at each recruiting funnel stage (applied → screened → interviewed → offered → hired), per group. The finding is the **change in group share across stages** (where the funnel narrows differentially), not any single %.
**Data source:** ATS with candidate self-ID data — **not in an HRIS export; often legally restricted; frequently high non-disclosure rates**.
**Gotchas:** (1) Non-disclosed is its own category — never impute. (2) Suppress groups with n < 5 at any stage. (3) Stage-share comparisons on small requisitions are noise — aggregate across reqs before reading anything.
**Reporting rule:** state stage definitions, disclosure rate, and suppression applied.

---

## Source of Hire Distribution
**Concise**

**Formula:** distribution of hires in window by ATS `source` field (referral, job board, agency, sourced, internal…). Percentages of hires with a known source; unknown/blank reported as its own share.
**Data source:** ATS.
**Gotchas:** (1) Source attribution rules differ — first-touch vs last-touch vs recruiter-entered; the field is only as good as the ATS discipline. State the caveat. (2) "Internal" hires here should reconcile with Internal Fill Rate.
**Reporting rule:** show unknown-source % with the distribution.

---

## Quality of Hire
**Composite / house-defined**

**What it measures:** whether recruiting is producing good employees, not just fast/cheap fills.
**Common construction:** a composite over new-hire cohorts — first-year retention, first performance rating, ramp/time-to-productivity, hiring-manager satisfaction survey — with house-chosen components and weights. **There is no canonical formula; the component set and weights are a house decision**, and the ingredients span ATS + performance system + surveys, **not a standard HRIS export**.
**Traps:** (1) Never present a single invented index number — report the components separately unless the house has a confirmed composite. (2) Any component using performance ratings inherits rating-coverage and calibration caveats. (3) Cohorts need a full observation period (like first-year attrition) before they're comparable.
**Reporting rule:** name the components and cohort; "QoH" without a stated construction is not a number.

---

## Vacancy Rate
**Concise**

**Formula:** ( open approved positions on date D ÷ (open approved positions + filled positions) on D ) × 100.
**Data source:** requisition/position data (ATS or position-management HRIS module) — headcount data alone can't see openings.
**Gotchas:** (1) "Approved" matters — draft/frozen reqs aren't vacancies. (2) In position-management systems, cross-check with Position-to-Worker Ratio (§10); they should tell one story.
**Reporting rule:** state date and what counts as an open position.

---

## Internal Fill Rate
**Concise**

**Formula:** ( filled requisitions filled by internal candidates ÷ all filled requisitions in window ) × 100. Internal = existing employee at time of application (promotion or lateral).
**Data source:** ATS with internal-candidate flagging.
**Gotchas:** (1) Denominator is *requisitions*, not headcount — see Internal Promotion Fill Rate (§6) for the promotion-only subset and the contrast with Promotion Rate. (2) Backfills created by the internal move are new reqs — internal mobility can *raise* total req volume; don't read a high internal fill rate as reduced hiring need.
**Reporting rule:** show the split ("28% internal: 18% promotion, 10% lateral").

---

# 8 · Compensation & Total Rewards

**Section conventions:** compare pay in **the same currency** — never convert salaries and compare across countries as if one market (convert only for total-cost aggregation, and say so). Use **FTE-annualized base pay** as the default pay field; state when using total cash (base + bonus/target).

## Compa-Ratio
**Core**

### Canonical formula
Individual compa-ratio = employee's FTE-annualized base salary ÷ **midpoint of the salary range for their grade** (same currency as the range).
Aggregate compa-ratio for a group = **mean of individual compa-ratios** (state n). 1.00 = at midpoint; healthy org ranges typically 0.80–1.20.

### Data required
Salary per employee **plus the salary structure** (grade → range midpoint, per currency/geo). **The structure lives in comp planning — usually NOT in a standard HRIS export.** Without it, compa-ratio cannot be computed; do not substitute the group average salary as a fake midpoint.

### Not the same thing
- Not **range penetration** ( (salary − range min) ÷ (range max − range min) ) — related, different scale, different question.
- Not market ratio (salary ÷ external market rate).

### Classic wrong ways
1. Dividing salary by the *average salary of the grade* when no structure exists (circular; always ≈1.0).
2. Mixing currencies or geo-differentiated ranges (a London salary against a US midpoint).
3. Aggregating as (sum of salaries ÷ sum of midpoints) without stating so — mean-of-ratios is the default; the two differ.
4. Using part-timers' actual pay against a full-time midpoint (FTE-annualize first).

### House decisions
- Base only vs total target cash; which structure version applies as of the analysis date.

### Reporting rules
State the structure version and pay basis ("mean compa-ratio 0.97 on FY26 ranges, base pay, n=130").

---

## Merit Increase Distribution
**Concise**

**Formula:** for employees receiving a merit increase in the cycle: merit % = increase amount ÷ pre-increase salary. Report the distribution (median, quartiles) and the **% of eligible employees receiving any increase**.
**Data source:** comp cycle data (comp planning module or effective-dated salary history with reason codes) — a single-snapshot export can't isolate merit.
**Gotchas:** (1) Separate merit from promotion increases and market adjustments by reason code — blending overstates "merit." (2) Zero-increase employees are part of the story — report the receiving-% alongside.
**Reporting rule:** name the cycle; "median merit 3.5%, 82% of eligible received an increase."

---

## Workforce Cost (Total / Average)
**Core**

### Canonical formula
Total workforce cost over a window = sum of compensation actually paid to in-scope employees in the window. **Fully loaded** = base + variable/bonus + employer taxes/benefits/equity cost.
Average workforce cost = total ÷ average headcount over the window.

### Data required — state what's actually in the file
A typical HRIS export carries **annual base salary only**. That yields "annualized base salary cost," not workforce cost. Fully loaded cost requires **payroll/finance data — usually NOT in a standard export**. Always label which you computed; never present base-only as total cost.

### Classic wrong ways
1. Sum of current salaries × 1 presented as the year's cost (ignores partial-year hires/terms — prorate by days employed in the window for the honest version).
2. Mixing currencies without stating the conversion date/rate.
3. Calling base-only "total cost of workforce" (typically understates by 25–40%).

### House decisions
- Cost buckets included; currency/conversion convention; whether contractor spend is added (separate line, never blended).

### Reporting rules
Label the basis every time ("annualized base-salary cost, prorated, USD-converted at 2026-06-30 rates").

---

## Cost per FTE
**Concise**

**Formula:** total workforce cost (state basis, per the Workforce Cost entry) ÷ **FTE count** = sum of FTE fractions on the reference date (two half-timers = 1.0 FTE).
**Gotchas:** (1) FTE ≠ headcount — using headcount as the denominator when part-timers exist understates cost per capacity unit. (2) If the file has no FTE-fraction field, you're computing cost per head — label it as such.
**Reporting rule:** state cost basis and denominator type.

---

## Pay Equity Ratio (raw AND adjusted)
**Core · sensitive** — the load-bearing warning: **never report the raw gap as "the gap"**

### Canonical formulas
- **Raw (unadjusted) ratio:** median FTE-annualized base pay of group A ÷ median of group B (e.g., women ÷ men). Raw gap % = 1 − ratio. This measures **composition** — who holds which jobs — not unequal pay for equal work.
- **Adjusted gap:** the pay difference **remaining after controlling for level, role/job family, location, and tenure** (minimum control set; house may add legitimate factors). Standard method: OLS regression of log(pay) on group + controls; the group coefficient ≈ the adjusted gap %. This is the number that speaks to "equal pay for equal work."

### Data required
Pay, demographic field, level, job family, location, hire date — all per employee. Adjusted analysis needs enough n per cell to be meaningful (rule of thumb: don't fit with < 30 usable observations or the result is noise — say so).

### Not the same thing
Raw and adjusted answer different questions: raw = representation/opportunity problem; adjusted = pay-decision problem. Both are real; presenting either as the other is the classic failure. Adjusted ≈ 0 with a big raw gap means the problem is *who gets which jobs*, not paycheck arithmetic.

### Classic wrong ways
1. Headline = raw gap, implying unequal pay for equal work.
2. Means instead of medians for the raw ratio (executive outliers).
3. Over-controlling: controlling for variables that are themselves outcomes of bias (e.g., performance rating, current level *if* level assignment is the suspected problem) — note which controls were used and this caveat.
4. Mixing currencies/geos without控制 for location.
5. Running it on 40 people and reporting a confident coefficient.

### House decisions
- Control set beyond the minimum; pay basis (base vs total cash); which demographic comparisons are in-scope legally (varies by country — some jurisdictions restrict processing).

### Reporting rules
Always report **both** numbers, labeled, with method ("raw median gap 12%; adjusted gap 2.1% controlling for level, job family, location, tenure; n=128"). Suppress any comparison where either group n < 5. State this is descriptive analysis, not a legal pay-equity audit.

---

# 9 · Compliance & DEI

## Representation
**Aliases:** gender/race representation, workforce composition · **Core**

### Canonical formula
% of in-scope point-in-time headcount (stated date) belonging to each demographic group — overall, and per level / department / job family as asked. Percentages within a cut sum to 100% including a "not disclosed" category (never impute, never drop).

### Data required
A demographic field with reasonable disclosure. If disclosure < 80%, report the disclosure rate prominently — composition of disclosers ≠ composition of the workforce.

### Not the same thing
Representation is **composition, not cause**. It says who is here, not why. Pipeline claims need Candidate Diversity Ratio (§7); progression claims need Promotion Rate by Demographic (below).

### Classic wrong ways
1. Reporting a group of 3 people as "2.1% representation" — **suppress any group under n = 5** ("suppressed, n<5", never the value).
2. Dropping non-disclosed and re-normalizing silently.
3. Comparing levels with wildly different n as if equally reliable.

### House decisions
- Which demographic fields are in-scope to analyze (legal constraints vary by country); category groupings.

### Reporting rules
Date, scope, disclosure rate, suppression applied — every time.

---

## Promotion Rate by Demographic
**Core · sensitive**

### Canonical formula
For each demographic group: Promotion Rate (§3 formula) computed **within the group** — promoted members of group ÷ average HC of group, annualized. Then report the **ratio between groups' rates** (e.g., women's rate ÷ men's rate) as the comparison figure.

### Classic wrong ways
1. Share-of-promotions vs share-of-headcount confusion ("30% of promotions went to women" is meaningless without women's % of the promotable population).
2. Comparing groups without conditioning on level mix — promotion velocity differs by level; at minimum report rates within level bands where n allows.
3. Headlining a group with avg HC < 20 or < 5 promotions (suppress/small-sample rules apply doubly here).

### Reporting rules
Show each group's numerator/denominator and the ratio; state that this is descriptive, not an adverse-impact test (that's a specific legal methodology — flag if the user needs it).

---

## Pay Equity Ratio by Demographic
Cross-reference: **use the Pay Equity Ratio entry (§8) exactly**, running the comparison per demographic group. All its rules apply — raw AND adjusted, both reported, n-floors, suppression, legal-scope house decision. No separate formula.

---

## Engagement by Demographic
**Concise · sensitive**

**Formula:** the house engagement score (survey composite — use the vendor/house construction, same discipline as Managerial Effectiveness §4) cut by demographic group.
**Data source:** survey platform — **not in an HRIS export**; requires the survey vendor's demographic-linked export, which many vendors restrict by design.
**Gotchas:** (1) Suppress groups with n < 5 respondents (vendor floors are often stricter — honor the stricter). (2) Response-rate differences between groups can explain score differences — report response rate per group. (3) Never present cuts fine enough to identify individuals (group × department × tenure = deanonymization).
**Reporting rule:** instrument, cycle, response rates, suppression — stated with every cut.

---

## Certification Expiration Rate
**Concise**

**Formula:** ( required certifications expired — or expiring within the stated look-ahead window, default 90 days — ÷ total required active certifications ) × 100, as of date D. Unit = certifications; report the count of affected *employees* alongside.
**Data source:** compliance/credential system or LMS — **not in a standard HRIS export**.
**Gotchas:** (1) "Required" is role-based — an expired cert someone no longer needs isn't a finding; check role mapping. (2) Expired vs expiring-soon are different urgencies — never blend without labels.
**Reporting rule:** date, look-ahead window, both units ("6 of 210 required certs expired; 14 expiring within 90 days; 17 employees affected").

---

## Overdue Policy Acknowledgement
**Concise**

**Formula:** ( employees past their acknowledgement deadline for a required policy ÷ employees required to acknowledge it ) × 100, per policy, as of date D.
**Data source:** policy/compliance system — **not in a standard HRIS export**.
**Gotchas:** (1) New hires inside their grace window aren't overdue. (2) Terminated employees must leave the denominator.
**Reporting rule:** per-policy rates, not a blended average across policies of different importance.

---

## Audit Findings Count
**Concise**

**Formula:** count of open audit findings as of date D, **by severity level and by age band** — it's a count and a shape, not a rate. Optionally trend: opened vs closed per period.
**Data source:** audit/GRC system — **not in an HRIS export**.
**Gotcha:** a falling count means nothing if the remaining findings are the old, severe ones — always show severity × age, never the bare total.
**Reporting rule:** date, severity breakdown, oldest-open age.

---

# 10 · Workforce Design & Planning

## Headcount Plan (Budgeted)
**Concise**

**Definition:** the approved/budgeted headcount (or FTE) per org unit per period, from **workforce/financial planning — NOT in an HRIS export**. This is an input, not a computed metric: the skill ingests it, validates units (heads vs FTE) and scope, and uses it for variance.
**Gotcha:** plans are versioned (original budget vs current forecast) — confirm which version before any variance math.
**Reporting rule:** state plan version, unit, and scope.

---

## Headcount Variance (Plan vs Actual)
**Concise**

**Formula:** `actual HC on date D − budgeted HC for the same date/period`, and as % of plan: variance ÷ budget × 100. Actual per the Headcount entry, **same scope and same unit as the plan**.
**Gotchas — the classic error is unit/scope mismatch:** (1) plan in FTE vs actual in heads; (2) plan includes open approved positions ("budgeted seats") vs actual counts only filled — reconcile with Vacancy Rate (§7); (3) plan scope includes contractors, actual doesn't. Verify all three before subtracting.
**Reporting rule:** "actual 142 vs plan 150 (heads, same scope) → −8 / −5.3%," per org unit and total.

---

## Job Level Distribution
**Concise**

**Formula:** % of point-in-time headcount at each job level/grade on date D. Blank/unmapped level is its own category (and feeds Job Architecture Coverage below).
**Gotcha:** a healthy shape is pyramid-ish; inversions (more seniors than juniors in a function) are the finding worth surfacing — descriptively, without prescribing.
**Reporting rule:** date, level scheme used, unmapped %.

---

## Job Architecture Coverage
**Concise**

**Formula:** ( in-scope employees mapped to a valid job code/level in the current job architecture ÷ total in-scope headcount ) × 100, on date D. "Valid" = the code exists in the architecture reference; requires the architecture list (comp/HR ops artifact — often not in the export; ask for it).
**Gotcha:** stale codes (valid once, since retired) pass a naive non-blank check — validate against the current architecture version, not just non-emptiness.
**Reporting rule:** architecture version, date, and the top unmapped populations.

---

## Org Design Stability Index
**Composite / house-defined**

**What it measures:** how much organizational churn employees experience — re-orgs, manager changes, team moves.
**Common construction:** % of employees with **no change** to manager AND department over the trailing window (default 12 months), from effective-dated history. Some houses build weighted indices over multiple change types — **the exact construction is a house decision; there is no canonical index**.
**Traps:** (1) Needs event history — a snapshot can't compute it. (2) A stable index isn't automatically good (frozen orgs stagnate) — report descriptively. (3) Mass re-org events dominate the number; report with and without the largest single event.
**Reporting rule:** state the construction used and the window.

---

## Position-to-Worker Ratio
**Concise**

**Formula:** ( total approved positions ÷ workers occupying positions ) on date D — meaningful only in **position-management** systems (Workday position management, SAP OM). Ratio > 1 → unfilled positions (cross-check Vacancy Rate); < 1 → overfills/shared positions (usually a data hygiene finding).
**Gotcha:** in job-management (non-position) systems this metric doesn't exist — don't fabricate positions from headcount.
**Reporting rule:** date, and reconcile the >1 remainder against open reqs.

---

## Role Criticality Index
**Composite / house-defined**

**What it measures:** which roles matter most to the business — for succession, retention, and planning focus.
**Common construction:** a rubric scoring roles on dimensions like revenue impact, scarcity of skills, time to replace, and single-point-of-failure risk. **This is a business judgment rubric, not arithmetic on HRIS data — the dimensions, weights, and scores are entirely house-defined.** The skill can *apply* a confirmed house rubric to role data; it never invents criticality scores.
**Traps:** (1) Criticality ≠ seniority — the rubric exists precisely because they differ. (2) Feeds Succession Coverage (§4); keep the critical-role list identical between the two.
**Reporting rule:** cite the rubric version; list dimensions used.

---

# 11 · Employee Experience

**Section precondition:** these come from a **survey platform, not an HRIS export**. Respect vendor anonymity floors; suppress any group with n < 5 respondents; always report response rates with scores.

## eNPS (Employee Net Promoter Score)
**Core** — the load-bearing warning: **never the average score**

### Canonical formula
From the 0–10 "how likely are you to recommend this company as a place to work" item:
`eNPS = % Promoters (9–10) − % Detractors (0–6)`
Passives (7–8) count in the denominator (total respondents) but not in either numerator. Range: −100 to +100. It is an integer-ish score, **not a percentage** — write "eNPS +12," never "12%."

### Data required
Raw response distribution (or promoter/passive/detractor counts) from the survey platform — **not in an HRIS export**. A pre-averaged "score" column cannot be converted to eNPS; you need the buckets.

### Not the same thing
- **Never the mean of the 0–10 responses.** A mean of 7.8 and an eNPS of −5 can coexist; the mean hides the detractor tail that the metric exists to expose.
- Not an engagement score (multi-item composite); eNPS is one item.

### Classic wrong ways
1. Averaging the responses.
2. Treating 7–8 as promoters (they're passives).
3. Reporting group eNPS for < 5 respondents.
4. Trending across surveys with different question wording or scale.

### House decisions
- Which survey/cycle is the source of record; whether contractors receive the survey.

### Reporting rules
Score + response rate + n, every time ("eNPS +12, 78% response, n=102"). Movements under ~10 points between cycles are usually noise at small n — say so.

---

## Manager Effectiveness Score
Cross-reference: this is the **Managerial Effectiveness Score — see §4**. Same metric, same survey-composite discipline (house/vendor construction, ≥5-respondent floor, no manager rankings, instrument-change trend break). Defined once to keep the two names from drifting apart.

---

## Survey Participation Rate
**Concise**

**Formula:** ( completed responses ÷ invited employees ) × 100, per survey. Denominator = invited, not total headcount (they can differ — state if they do).
**Gotcha:** partial responses — follow the vendor's completion definition and state it.
**Reporting rule:** report with every survey-derived metric; per-group participation gates every demographic cut (see Engagement by Demographic, §9).

---

# 12 · Risk & Governance

## Flight Risk Score
**Composite / house-defined · sensitive** — a model, not a formula

**What it measures:** the estimated likelihood that an employee (or, properly, a *segment*) will leave in a coming period.
**Common construction:** a predictive model (logistic regression / gradient boosting on tenure, time-since-promotion, comp position, manager changes, engagement…) or a vendor's proprietary score, or a manager-judgment rubric. **There is no canonical formula — the construction belongs to the model owner/vendor/house**, and building one is a modeling project, not a metric lookup. The skill can *describe* a confirmed house model's output; it never trains or improvises one inline.
**Traps — these are binding:**
1. **Never present a score as a probability** unless the model is demonstrably calibrated — "score 0.7" is not "70% chance of leaving."
2. **Never name an individual** as a flight risk in generated output. Aggregate to segments (≥ 5 people) only; individual scores are for the house's own governed process.
3. Correlation drivers ≠ causes; don't narrate "X is leaving because of pay" from a feature weight.
4. Scores decay — a model scored on last year's data says little without refresh cadence stated.
**Data source:** model output from an analytics team or vendor module — **not in a standard HRIS export**.
**Reporting rule:** state model owner/version, scoring date, and calibration status; report segment-level distributions only.

---

# Still pending
Not yet canonical. Until an entry lands here, Peoplenometry computes these with its method stated inline, flags "library entry pending," and — once you confirm the method — offers to save it as a Local metric in your house-rules. Guardrails noted are binding even while pending.

- **Funnel Pass-Through Rate** — stage-to-stage conversion in the recruiting funnel (ATS data; stage definitions are house-specific).
- **Absenteeism Rate** — needs time/absence data, rarely in a workforce export; definition (unplanned only?) is a house decision.
- **Overtime Rate / Utilization** — needs time-tracking data.
- **Contingent Worker Ratio** — contractors ÷ total workforce; scope definitions vary widely.
- **Adverse-Impact (4/5ths) Analysis** — a specific legal methodology, distinct from the descriptive demographic rates in §9; only with explicit house sign-off.

*Library version 2.0 (2026-07). Canonical file — never edited locally; house decisions and local metrics live in `house-rules.md`.*