# Inspired Hygiene: Hygiene Analysis

Interactive hygiene analysis for dental practices, built by Brighter Idea Marketing for Inspired Hygiene. It replaces the fillable "Mini Hygiene Analysis Worksheet" PDF and the "Perio Percentage Calculator."

## What it does
1. **Perio check** (free, no email): D1110, D4341+D4342, D4910 counts give the perio % on the tier pyramid.
2. **Hygiene financials:** compensation, net hygiene production, total practice production, total hygiene days per month.
3. **Contact info:** lead capture with required referral source.
4. **Results:** four core metrics vs. Inspired Hygiene benchmarks, coaching questions from IH's question bank, discovery meeting CTA, branded PDF report.

## Files
- `index.html`: the full single-file app (HTML, CSS, JS, logo embedded). Opens directly in a browser.
- `docs/BUILD-SPEC.md`: full build spec (formulas, benchmarks, question bank, AI prompt, GHL webhook, test cases). Hand this to an AI builder for the production version.
- `assets/`: Inspired Hygiene logo and mark (PNG, transparent).

## Formulas
```
Profitability ratio  = net hygiene production / total hygiene compensation        (goal 3:1)
Production per day   = (net hygiene production / 12) / hygiene days per month      (goal $1,200-1,500)
Hygiene % of total   = net hygiene production / total practice production x 100    (goal 25-30%)
Perio %              = (SRP/2 + PM) / (Prophy + PM + SRP/2) x 100                   (foundation 30%+)
```
Thresholds and the question bank live in the `CONFIG`, `BENCH` and `QB` objects at the top of the script in `index.html`.

## Config to update before launch
- `CONFIG.bookingUrl`: discovery meeting booking link (currently a placeholder)
- `CONFIG.podcastUrl`: podcast link (currently a placeholder)

## Notes
- The AI analysis and PDF download use Claude artifact capabilities (`window.claude.use("sample")`, `use("downloads")`). Outside Claude, the page falls back to the standard rule-based analysis automatically. For public hosting, swap these for a server-side Claude API call and a direct PDF download (see `docs/BUILD-SPEC.md`, sections 7 and 8).
- The contact form does not submit anywhere yet. Production version posts to a GoHighLevel inbound webhook (spec section 8.3).
