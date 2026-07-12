# People Analytics Skills

**Ask your people data anything.** Point your AI tool at a workforce export and ask a question in plain language — headcount trends, tenure, attrition, comp, representation, span of control, promotions. You get a defensible answer, computed from a canonical metrics library, in plain English with the methodology attached. No data team required.

A toolkit for the HR generalist who owns the data but was never handed the data-science function. Built by [Peoplenometry](https://peoplenometry.tools). Powered by [peoplesets](https://peoplesets.com) — no data to practice on? Generate a safe, realistic synthetic workforce set instead of pasting real records.

## Install

```
npx skills add steven-shoemaker/peoplenometry-skills
```

Then run any skill in Claude Code, Cursor, Codex, or any tool that supports skills.

## Why it's different from pasting a spreadsheet into an AI

- **A canonical metrics library.** Every number is computed from a shared [metrics library](skills/people-analytics/references/metrics-library.md) that defines *exactly* how each metric works — the test each entry passes is that two different AIs reading it produce the same number from the same file. The classic wrong ways are blocked by name (attrition on *average* headcount, never "terminations ÷ everyone"; retention is not 1 − attrition; never the raw pay gap; never the average eNPS).
- **It learns your company.** Definitions you confirm, and metrics your company uses that the library doesn't cover yet, accrete into a local `house-rules.md` that overrides and extends the library. Run ten is sharper than run one, and the toolkit becomes *yours*.
- **Two-layer output, always.** A plain-English answer first; the methodology, sample sizes, and caveats second — built to survive the moment Finance asks "how did you calculate that?"
- **It asks before it assumes.** HR data is inconsistent between companies, so the skills ask which column is the term date, what counts as voluntary, what window — then never re-ask.

## The skills

| Skill | What it does |
|---|---|
| `/people-analytics` | **Start here.** Ask any question about your people data. It computes the answer from the metrics library, and escalates to a specialist when one fits. |
| `/clean-my-export` | Turn a messy HRIS export into an analysis-ready file with an honest data-quality report. Run first when the data's rough. |
| `/why-are-people-leaving` | The deep attrition flagship: rates vs fair baselines, segment and tenure diagnostics, drivers, and which segments are at risk next. |
| `/write-a-report` | Render any finished analysis into a leadership-ready, self-contained HTML report that prints to PDF. Renders only; never recomputes. |

## See it work

A full report the toolkit generated end-to-end from a synthetic 4,962-person company:

→ **[examples/sample-attrition-report.html](examples/sample-attrition-report.html)** (open in a browser — self-contained, prints to PDF)
→ [the analysis it was built from](examples/sample-attrition-analysis.md)

Headline attrition looked healthy; the report led with the real finding underneath — most departures happening inside the first year, a hiring problem rather than a retention one. The methodology appendix is attached so it holds up when someone pushes on the numbers.

## A note on privacy

These skills run wherever your AI tool runs — your data never leaves your session. Never paste records you're not cleared to use. To learn or demo without touching real data, generate a synthetic set with [peoplesets](https://peoplesets.com).

---

MIT licensed. Fork it, adapt it, ship it.
