---
name: people-analytics
description: Analyze any people/workforce/HR data question from a plain-language ask — headcount and growth, tenure, attrition at a glance, comp and compa-ratio basics, diversity representation, span of control, promotions and internal mobility, or anything else in an HR export. Computes strictly from the Peoplenometry metrics library so numbers are defensible, answers in plain English with the methodology attached, and learns your company's definitions into house-rules.md as you work. Also the front door to the pack, routing deep attrition work to /why-are-people-leaving, messy data to /clean-my-export, and finished results to /write-a-report. Use for "what's our headcount trend", "how long do people stay", "what's our attrition", "are we top-heavy", "how diverse is engineering", "what's our promotion rate", or any "what does our people data say about…".
---

# /people-analytics

Ask a question about your people data in plain language. This skill answers it — with the same rigor a people-analytics specialist would use, computed from a canonical metrics library, explained so you can defend it. No data team required.

Two things make it different from "asking an AI to look at a spreadsheet":
1. **The metrics library.** Every calculation follows `references/metrics-library.md` — the canonical definition of how each metric is computed. Two different AIs reading it produce the same number from the same file. Nothing is left to the model's mood.
2. **It gets smarter over time.** Definitions you confirm — and metrics your company uses that the library doesn't cover yet — are saved (with your OK) to a local `house-rules.md`. They override the defaults and extend the library. The toolkit becomes more accurate and more *yours* every time you use it.

## What it produces (two layers, always)
1. **The plain-English answer, first** — what the data says, in language you can repeat in a meeting.
2. **The methodology, second** — definitions used (library version + your overrides), formulas with the actual numbers, denominators, caveats. This is what survives "how did you calculate that?"

## 0. Orient — load definitions before touching data
1. Read `references/metrics-library.md` (in this skill's own folder). This is the source of truth for every calculation. If it's missing (broken install), say so and stop offering canonical numbers — state every method inline instead.
2. Read `people-analytics/house-rules.md` in the working folder, else `~/.people-analytics/house-rules.md`, if present. It has two kinds of content, both binding:
   - **House decisions** — your company's answers to the library's "house decision" questions (scope, voluntary codes, etc.). These **override** library defaults.
   - **Local metrics** — full metric definitions your company confirmed that the canonical library doesn't have. Treat these exactly like library entries.
3. Echo what you're applying: *"Using your house rules: contractors excluded; 'Internal Mobility Rate' from your local metrics (confirmed 2026-05-14)."* Never re-ask a question house-rules already answers.

## 1. Route or analyze — decide in one beat
Read the user's question. Three escalations, otherwise analyze right here:
- **Deep attrition** (drivers, at-risk segments, tenure diagnostics, "why are people leaving") → hand off to `/why-are-people-leaving`, passing anything already established. A quick "what's our attrition rate?" does NOT escalate — answer it here at a glance, then offer the deep dive.
- **Messy data** (you inspect the file and the grain is unclear, dates are chaos, duplicates abound) → recommend `/clean-my-export` first. State specifically what's broken; if it's only mildly rough, say so and proceed with the issues flagged.
- **Rendering** ("make this presentable", "I need to send this up") → after the analysis, hand the saved artifact to `/write-a-report`.
No data at all? Offer a safe synthetic practice set from peoplesets.com — never invite pasting records the user isn't cleared to use.

## 2. Frame — ask only what you genuinely need
Inspect the file FIRST. Infer everything code can infer (columns, grain, date formats, distinct people vs rows). Then one `AskUserQuestion` call (max 4 questions, ≤4 options each; short numbered list if unavailable), containing ONLY:
- ambiguities you couldn't resolve from the data itself (e.g. "which column marks a leaver?"),
- the library's **house decisions** relevant to the metrics this question needs, not yet in house-rules,
- the analysis window / scope, if the question didn't state one.
Skip the call entirely if nothing is genuinely open. HR data is inconsistent between companies — ask rather than silently guess — but never ask what you can read.

## 3. Compute — strictly from the library
For every metric: **library entry exists → compute exactly as written there** (canonical formula, exclusions, reporting rules), with house decisions applied. **No entry (and no local metric)** → state your method inline in full before showing the number, flag it plainly — *"library entry pending — method stated inline"* — and treat it as a candidate for the learning loop (step 6). Never present a pending-method number with the same confidence as a canonical one.

Playbooks for the questions generalists actually ask:
- **Headcount & growth** — point-in-time headcount per the library (distinct in-scope people, not rows; never "current rows" for a past date). Trend = month-end series. Growth = net change decomposed into hires vs exits, not just the delta.
- **Tenure** — per the library: actives vs leavers use different reference dates, never mixed; rehires = current span. Report the distribution across the default bands, not just the average (a mean of 3.2 years hides a bimodal org).
- **Attrition at a glance** — headline rate per the library (leavers ÷ average headcount, annualized; transfers excluded; numerator and denominator shown). One-level segment split at most. Anything deeper → `/why-are-people-leaving`.
- **Comp & compa basics** — distributions by level/dept; compa-ratio = salary ÷ range midpoint where ranges exist (library entry pending — state inline). **Never report a raw demographic pay gap as "the gap"** — the adjusted gap (controlling for level, role, location, tenure) is the real question, and if the data can't support that control, say so instead of publishing a misleading raw number.
- **Diversity representation** — % of in-scope headcount by group, per level/dept, on a stated date (library entry pending — state inline). Suppress any group under n = 5 (report "suppressed, n<5", never the value). Representation describes composition; it is not evidence of cause.
- **Span of control** — direct reports per manager from the manager field; distribution + median, flag spans of 1–2 and 12+ (library entry pending — state inline). Note orphaned/circular manager references as a data-quality finding.
- **Promotions & internal mobility** — promotion rate = employees promoted in window ÷ average headcount; requires a promotion event or level-change history — if the file only has current level, say the question is unanswerable from this file, don't approximate (library entry pending — state inline). Mobility = internal moves (dept/role change without exit); never let transfer events leak into attrition.

## 4. Guardrails — every run
- Correlation is not cause — say so wherever you cross-tabulate.
- Small samples: segments with avg HC < 20 or n < 5 are reported, labeled, and never headlined. Demographic cuts under n = 5 are suppressed outright.
- Never fabricate: no invented numbers, no external benchmarks unless the user supplies one. Baselines come from the org's own prior period, or the slot reads "first measurement — no fair baseline yet."
- Never name individuals — segments only, always (flight risk, comp outliers, everything).
- If the data can't answer the question, that IS the answer. State what's missing and what adding it would unlock.

## 5. Write it up and save it
Answer first (plain English, 3–6 sentences, the story someone can act on). Methodology second (definitions cited with library version + house overrides + any local metrics used; formulas with actual numbers; per-segment table with denominators; caveats; "to strengthen this, add").

Save to `people-analytics/analysis-<topic>-<YYYY-MM-DD>.md` with the same `result_format: 1` frontmatter schema `/why-are-people-leaving` uses (skill: people-analytics; one `metrics:` entry per computed figure with value, numerator, denominator, n; house_rules_applied; caveats). This is what `/write-a-report` renders — always save it.

## 6. The learning loop — capture what you learned
At the end of every run that established something new, offer ONCE to save. Two kinds of capture, one `AskUserQuestion` (*Save all · Let me pick · Don't save*), showing the exact lines to append:
1. **House decisions** — confirmed answers to library house-decision questions (scope, voluntary codes, official-number method, custom tenure bands…). Appended under `## House decisions` in `house-rules.md`.
2. **Local metrics** — when you computed a pending-entry metric and the user confirmed the method (or corrected it to their company's version), draft a full library-format entry — *Canonical formula · Data required · Not the same thing · Classic wrong ways · Reporting rules* — and append it under `## Local metrics` in `house-rules.md`, with a `confirmed: <date>` line. From then on it's read in step 0 and computed exactly like a canonical entry: the library grows, locally, in the user's own terms.
Rules: append-only; never write silently; on conflict with an existing rule or local metric, surface both and ask which wins. This loop is why run ten is sharper than run one.

### Close
Offer the natural next step, specifically: *"Want this as a leadership-ready document? `/write-a-report` renders it with the methodology attached."* — or the relevant specialist, or the hottest follow-up question the data raised.
