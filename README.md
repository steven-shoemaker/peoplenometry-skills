# People Analytics Skills

**Do people analytics without a data team.** Paste your workforce export into your AI tool and get the real analysis — proper rates, fair comparisons, honest caveats — written up as a story your CHRO will act on, with the methodology underneath so it holds up when Finance pushes back.

A pack of installable AI skills for HR and People teams. Built by [Peoplenometry](https://peoplenometry.tools). Powered by [peoplesets](https://peoplesets.com) — no data to practice on? Generate a safe, realistic synthetic workforce set instead of pasting real records.

## Install

```
npx skills add stevenshoemaker/peoplenometry-skills
```

Then run any skill in Claude Code, Cursor, Codex, or any tool that supports skills.

## Why it's different

Most "HR analytics" help gives you a plausible number and hopes you don't check it. This pack is built the other way around — **accessible on the way in, rigorous on the way out**:

- **Real methods, not vibes.** Formulas come from a shared [metrics library](skills/people-analytics/references/metrics-library.md), not guesswork. Attrition is leavers ÷ *average* headcount, annualized — not the naive "terminations ÷ everyone" that dashboards quietly report and that can run ~2× too high.
- **Two-layer output, always.** A plain-English answer first; the methodology, sample sizes, and caveats second. Credibility *is* the product.
- **It asks before it assumes.** HR data is wildly inconsistent between companies, so the skills ask which column is the term date, what counts as voluntary, what window — then remember your answers.
- **It gets more yours the more you use it.** Confirmed definitions save to a local `house-rules.md` the skills read on every run.

## See it work

A full report the pack generated end-to-end from a synthetic 4,962-person company:

→ **[examples/sample-attrition-report.html](examples/sample-attrition-report.html)** (open in a browser — self-contained, prints to PDF)
→ [the analysis it was built from](examples/sample-attrition-analysis.md)

The company's own dashboard called turnover **29.9%**. Measured honestly — annualized, against average headcount — it was **15.5%**. Same people, same exits; the method was the whole story. That gap, and the appendix that reconciles it, is why this exists.

## The method — Question → Data → Analysis → Narrative

Start with a question, point a skill at your export, it does the analysis, and hands you back the *story* — not a chart you still have to explain.

## Start here

- **Not sure what to run?** → `/people-analytics` — tell it your situation in a sentence, it routes you.
- **Messy export?** → `/clean-my-export` first — get to an analysis-ready file. Attrition math is very sensitive to bad dates, duplicate rows, and rehires.

## The skills

| Skill | The question it answers |
|---|---|
| `/people-analytics` | Start here — the method, and routing to the right skill. |
| `/clean-my-export` | "My data's a mess — get it analysis-ready." |
| `/why-are-people-leaving` | "Why is attrition up, and who's at risk?" |
| `/brief-my-chro` | "Turn this analysis into a report I can send upward." |

More analyses (pay equity, hiring funnel, engagement) are in progress. Until they ship, `/people-analytics` handles them inline using the same metrics library and principles.

## A note on privacy

These skills run wherever your AI tool runs — your data never leaves your session. Never paste records you're not cleared to use. To learn or demo without touching real data, generate a synthetic set with [peoplesets](https://peoplesets.com).

---

MIT licensed. Fork it, adapt it, ship it.
