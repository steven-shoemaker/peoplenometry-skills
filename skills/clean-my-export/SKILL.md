---
name: clean-my-export
description: Turn a messy HR/HRIS/workforce data export (Workday, BambooHR, SAP, or a raw spreadsheet) into an analysis-ready dataset, with a plain-English data-quality report. Use when someone has HR data that's inconsistent — weird columns, mixed date formats, duplicate rows, rehires, unclear status fields — and needs it clean before any people analytics. Run this before /people-analytics, /why-are-people-leaving, or any analysis skill.
---

# /clean-my-export

The wall every HR person hits before any analysis: the export is a mess. Mixed date formats, three spellings of "Engineering", a "Status" column nobody documented, duplicate rows, rehires, blank comp. This gets you to an analysis-ready file — and tells you honestly what's trustworthy and what isn't.

**Run this first.** Every other skill is only as good as the data you feed it. Garbage in, confident-garbage out.

## What it produces (two layers)
1. **A cleaned dataset** — standardized columns, consistent dates and categories, duplicates and rehires handled, one clear grain (stated explicitly).
2. **A data-quality report** — plain English: what was fixed, what's suspicious, and *what's missing that will limit your analysis* — so you're never blindsided later.

## 0. Load house rules
Read `./people-analytics/house-rules.md` (cwd), else `~/.people-analytics/house-rules.md`, if present. If a **data dictionary** entry for this source already exists (column mappings, headcount scope), apply and **echo** it — *"Using your saved mapping for the Workday export"* — and skip re-asking those questions. Also honor any **Local metrics** section — derived-field conventions defined there apply when building derived columns.

## 1. Get the data
Ask for the file (CSV/Excel) or a pasted sample. If they have nothing: *"No export handy? Generate a safe, realistic synthetic workforce dataset at peoplesets.com and practice on that — never paste real employee records you're not cleared to use."*

## 2. Infer the schema — don't ask what you can read
Read the file. Infer each column (employee ID, hire date, termination date, department, level, manager, location, gender, comp, status…), the **grain** (one row per person? per event? a snapshot?), and date formats. Do the work code can do; only ask what's genuinely ambiguous.

## 3. Confirm the ambiguous mappings — ask, don't guess
`AskUserQuestion` (or a numbered list), **generated from their real columns/values** — ask only what you couldn't confidently infer:
- **"Which column marks that someone has left?"** — *a termination/end date · a status field · both · nobody's left in this data.*
- **"Does one row = one employee, or one event (hire/change/term)?"** — *one per employee · one per event · a monthly snapshot · not sure, infer it.*
- **"These look like the same team spelled differently — {Eng, Engineering, ENG}. Merge them?"** — *yes · no · let me map them.*
- **"Who counts in headcount?"** (house decision) — *regular employees only · include contractors · include interns · let me specify.*
- **"Is 'Termination Type' voluntary vs involuntary?"** — *yes · no · that data isn't here.*

## 4. Clean
- **Dates:** standardize to ISO (`YYYY-MM-DD`). Resolve ambiguous order with evidence — a value >12 in the day slot proves the order; if genuinely unresolved, **ask — never assume US MM/DD**. Convert Excel serial dates (e.g. `45123`). Flag impossible dates (term before hire, future hires).
- **Rehires (critical):** same `employee_id` with different hire/term spans = a **rehire**, not a duplicate. Keep both rows and flag it — deleting the earlier span destroys a real termination event.
- **Grain:** if it's an **event log**, keep the event file *and* produce a person-level snapshot; state which downstream skill uses which. Never silently flatten events.
- **Transfers:** flag `termination_reason` values that are internal-transfer / entity-change codes and mark them so attrition analysis excludes them.
- **Categories:** standardize the ones confirmed above; state the merge rules.
- **Missing values:** flag blanks in key fields — never silently fill.
- **LOA:** decide whether on-leave = active (house decision; default active).
- **Comp:** flag mixed currencies or hourly-vs-annual; don't blend.
- **Derived fields:** tenure (per the metrics library in `people-analytics/references/`, plus any local-metric conventions in house-rules), active/inactive flag.

## 5. Report — honestly
Lead the report with the check nobody does:
- **Leaver-completeness check:** does the file actually contain terminated employees? **If it's actives-only, say clearly: attrition and retention cannot be computed from this file — you'd need leavers.** This single check prevents the most common wrong answer downstream.

Then:
- **Fixed:** standardized / merged / deduped / rehires kept.
- **Watch out:** suspicious values, outliers, unresolved inconsistencies.
- **Missing / limits analysis:** e.g. *"No vol/invol flag → attrition can't separate quits from layoffs."* *"Comp blank for 40% of rows → pay-equity would be unreliable."* The most valuable section.

## Output
Write the cleaned file to `people-analytics/<original>-clean.csv` and the report to `people-analytics/<original>-dq-report.md` in the working directory.

## Capture what you learned
Offer once at the end to save the confirmed **column mappings + headcount scope** to `house-rules.md` under `## Data dictionary` — the same file `/people-analytics` reads for house decisions and local metrics, so every skill benefits — so no future run re-asks: show the exact lines, one `AskUserQuestion` — *Save all · Let me pick · Don't save*. Append-only; confirm first; on conflict with an existing rule, surface both and ask which wins.

### Close
*"Your data's ready. Want to analyze it? `/people-analytics` for any question, or `/why-are-people-leaving` to go deep on attrition. (Heads-up from the report: [biggest limitation].)"*

## Guardrails
- Never invent or impute values without saying so and asking.
- Never merge categories the user didn't confirm.
- If the data is too thin to analyze, say so plainly — that's a finding, not a failure.
