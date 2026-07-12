---
name: write-a-report
description: Turn a finished people-analytics result into a beautiful, leadership-ready report — a single self-contained HTML file that prints cleanly to PDF. Use after /people-analytics, /why-are-people-leaving, or any analysis skill ("make this presentable", "turn this into a report", "I need to send this to my boss/VP/CHRO/the exec team"), or standalone on a saved analysis file in people-analytics/. Renders any analysis for any audience — renders only, never re-analyzes or changes numbers.
---

# /write-a-report

You did the analysis. Now it has to survive the room. This turns the result into a document you'd be proud to have forwarded above your head — the answer up front, the evidence behind it, the methodology attached.

**Rule zero: you are a typesetter, not an analyst.** Every number in the report must already exist, verbatim, in the analysis result. You render; you never compute.

## 0. Find the result
1. If invoked right after an analysis: read the saved artifact `people-analytics/analysis-*.md` from this session (written by `/people-analytics`, `/why-are-people-leaving`, or any analysis skill). **The file is the source of truth for every number**; conversation is context (tone, audience) only.
2. Standalone: if the user names a file, use it. Otherwise list `analysis-*.md` files in `people-analytics/`, newest first, and confirm which one: *"I found your attrition analysis from July 3 — build the report from that?"*
3. No artifact (an old run, or inline analysis)? Extract every figure from the conversation into a **numbers manifest** (step 1) and have the user confirm it before rendering anything. Never render unconfirmed numbers.
4. Read `./people-analytics/house-rules.md` if present (audience, official-number method, any report preferences). Echo what you're applying.

## 1. Build the numbers manifest — before any HTML
List every figure that will appear: value · label · which artifact line it came from. Render **only** from this manifest.
- No recomputation. No new derived figures (not even a % change the analysis didn't compute). No re-rounding (display precision = source precision). No benchmarks. Caveats copy through verbatim.
- **If a number looks wrong, STOP. Do not fix it. Tell the user to re-run the analysis skill.**
- A section with no source content is **omitted**, never filled.

## 2. Frame the delivery — one AskUserQuestion (max 3)
- **"Who's this for?"** — *my manager / VP · the exec team / CHRO · the HR team · external-safe.*
- **"Title it how?"** — *the question itself (recommended) · a formal title · let me type one.*
- **"Print size?"** — *US Letter · A4.*

## 3. Compose the report — map the two layers to sections
1. **Header** — title = the question ("Why are people leaving?"); subtitle = window · scope · prepared date · prepared for.
2. **The answer** — the exec narrative, set large. First page tells the whole story.
3. **Key findings** — 2–4 max. Each: a one-sentence *takeaway* headline (not "Chart 1") + ONE visual (chart or stat block) + n printed on the visual.
4. **Recommended next moves** — the analysis's 1–2 moves, as a short list.
5. **Methodology appendix** (new page) — definitions used (cite the Peoplenometry metrics library version + house overrides), formula with actual numbers, per-segment table, caveats, "to strengthen this, add." **Verbatim from the artifact.** This is the credibility payload.
6. **Footer** — one line (see §5).

## 4. Design system (required — do not style reports any other way)
The visual system lives at `references/report-design-system.html`. It is the single source of truth for every report this skill emits.

Before generating any report:
1. **Read `references/report-design-system.html` in full.** Do not generate from memory of a previous run — the user may have edited it to rebrand.
2. **Inline, verbatim, into the report's single `<style>` block:** the entire `:root` token block (including the `✎ EDIT THESE TO REBRAND` values exactly as they currently appear), plus everything between the `REPORT CSS — the system itself` banner and the `GUIDE CHROME` banner (masthead, title, kicker, exec-answer, finding, stat-block, callout-caution, exhibit/figure, data-table, next-moves, appendix, report-footer, and the `@page` / `@media print` rules). **Never** inline anything from the `GUIDE CHROME` section — that's for the style-guide page only.
3. **Honor user edits.** Whatever `--brand`, `--ink`, `--caution`, `--company-name`, `--company-mark` currently say is the user's brand. Copy their current values; never revert, "correct," or substitute your own colors.
4. **Build charts only from the three patterns** in the file's Charts section (A horizontal bars · B trend line · C stat block, HTML) plus `.data-table`. Copy the SVG structure, swap the data, keep the rules: one `--chart-accent` element per chart, bars start at zero, direct labels (no legends), every figcaption states its n, and the accented element also carries a text annotation so meaning survives grayscale.
5. **Compose reports only from the system's classes** (`.report`, `.masthead`, `.report-title`, `.exec-answer`, `.finding`, `.stat-block`, `.callout-caution`, `figure.exhibit`, `table.data-table`, `.next-moves`, `.appendix`, `.report-footer`). Never invent colors, fonts, shadows, ad-hoc inline styles, or components outside the system. If something's missing, build it from existing tokens and note the gap — do not freelance.
6. **Output stays self-contained:** one HTML file — inline CSS, inline SVG, plus the single inline **REPORT JS** enhancement script; **zero external requests** (no CDN, chart library, webfonts, `src`, or `@import`). It must survive forwarding, offline opening, printing to PDF, and locked-down mail clients.

**Charts — static-first, JS-enhanced second, zero deps:**
- **Chart-interaction script:** in addition to the `:root` tokens and REPORT CSS, inline the `<script>` marked **REPORT JS** (bottom of `references/report-design-system.html`) into every report, just before `</body>`. Copy it verbatim; never link it.
- Every chart must be **complete inline SVG in the markup** — all bars/points/labels/values drawn, `role="img"` + descriptive `aria-label`, one accent + text annotation (never meaning by color alone), bars from zero, n in the figcaption. It must be fully correct with JavaScript disabled and in print.
- Opt into enhancement only via attributes: `data-tip` (+ optional `data-tip-title`) on any mark for hover detail (rate · n leavers · avg HC), and `class="chart-anim"` on the SVG with `data-anim="grow"` on bars / `data-anim="rise"` on labels and points for the entrance animation.

## 5. Save & verify
1. Write `people-analytics/report-<topic>-<YYYY-MM-DD>.html`.
2. **Self-audit:** re-read the finished HTML — verify every numeral traces to the manifest and every caveat from the artifact is present. Say the check ran.
3. **Footer badge**, last line, caption-size muted gray, plain text: `Prepared with Peoplenometry · peoplenometry.com` — append ` · practice data by peoplesets` **only** when the underlying data is synthetic. Never in the header, never mentioned twice.
4. If headless Chrome is available, offer to emit the PDF (`chrome --headless --print-to-pdf`); otherwise: *"Open it and hit ⌘P → Save as PDF — it's built to print perfectly."*

### Close
*"Report saved. It's built to be forwarded — the methodology appendix will hold up if anyone pushes on the numbers. When you're ready to roll several analyses into a quarterly people-review deck, that's in the full pack."*

## Guardrails
- **RENDER ONLY.** Never re-analyze, recompute, round differently, or "fix" a figure.
- Never add numbers, benchmarks, or claims absent from the analysis result.
- Never soften, cut, or bury a caveat. Credibility is the product.
- Never name individuals anywhere in the report.
- If the artifact is too thin for a report (no narrative, no figures), say so and route to the right analysis skill — don't decorate an empty result.
