# Build Spec: Inspired Hygiene "Hygiene Analysis" App

**Purpose of this document:** This is a complete build brief for an AI app builder (Lovable, Bolt, Replit Agent, Claude Code, v0, or similar). Upload this file and say: *"Build the application described in this spec. Follow it exactly. Ask me only about items listed under Open Decisions."*

**Client:** Inspired Hygiene (InspiredHygiene.com), a hygiene-department coaching company for dental practices.
**Built by:** Brighter Idea Marketing.
**Replaces:** Two PDFs: the fillable "Mini Hygiene Analysis Worksheet" (Adobe FormsCentral) and the "Perio Percentage Calculator" referenced on the "Hygiene Metrics Quick Reference" handout.

---

## 1. What we are building

A branded, mobile-friendly web app where a dental practice owner or office manager enters a few numbers from their practice management software and instantly gets:

1. Their four core hygiene metrics, calculated exactly as Inspired Hygiene calculates them today.
2. A color-coded status for each metric against Inspired Hygiene's benchmarks.
3. An AI-written, personalized analysis that explains what the numbers mean and pulls the right coaching questions from Inspired Hygiene's own question bank (Section 6).
4. A clear call to action to book a discovery meeting with Inspired Hygiene.
5. A downloadable/emailed PDF report, and the lead plus their results sent into the CRM.

**The golden rule of this app:** All math is done in deterministic code. The AI never calculates, estimates, or changes a number. The AI only explains numbers the code already produced.

---

## 2. User flow

Build it as a short multi-step flow, not one long form.

**Step 1: Quick Perio Check (no email required)**
- 3 inputs: Adult Prophy (D1110), SRP (D4341 + D4342), Perio Maintenance (D4910). All counts for the last 12 months.
- On submit, immediately show their Perio % with the tier pyramid (Section 5.4).
- Below the result: "Want the full picture? Get your complete Hygiene Analysis in 2 minutes." Button to Step 2.
- This step is the hook. It must be fast, free, and give value before we ask for anything.

**Step 2: Hygiene Financials**
- Total annual hygiene compensation (salary + benefits)
- Net hygiene production, last 12 months
- Total practice production, last 12 months
- Total hygiene days worked per month (whole department)

**Step 3: Contact info (gate for the full report)**
- First name, last name, email, phone, practice name, role (dropdown: Practice Owner / Doctor, Office Manager, Hygienist, DSO / Group Leader, Other), number of hygienists, city/state.
- "How did you hear about us?" (required dropdown: Podcast, Referral from a colleague, Elite Dental Alliance (EDA), Event / Speaking, Google Search, Social Media, Other + text field). **Referral source is required** because Inspired Hygiene has a referral agreement that depends on it.
- Consent checkbox for email/SMS follow-up.

**Step 4: Results Dashboard**
- 4 metric cards (Section 5) with value, goal, and status color.
- AI Analysis section (Section 7).
- "Our Standard" strip: Win for Patients / Win for the Team / Win for the Practice. "Every Inspired Hygiene recommendation must satisfy all three. We never advocate quantity over quality."
- Primary CTA: "Schedule a Discovery Meeting" (booking link, see Open Decisions).
- Secondary: "Download my report (PDF)", "Email me my report", "Listen to the Inspired Hygiene Podcast with Rachel Wall".
- "Edit my numbers" button that returns to the inputs with values preserved and recalculates.

---

## 3. Input fields (exact labels and help text)

All numeric inputs: accept commas and dollar signs on entry, strip them before math, reject negatives, allow decimals on dollar fields, whole numbers on counts and days. Show inline help text under each field (the person entering data is often not the doctor, so the help text matters).

| ID | Label | Help text | Type |
|---|---|---|---|
| `comp` | Total annual hygiene compensation | Total paid to ALL hygienists in the last 12 months, including benefits: health insurance, paid vacation/holidays, CE compensation, uniform allowance, 401k match, bonuses. | Currency |
| `hygProd` | Net hygiene production (last 12 months) | Net (collectible) production for your ENTIRE hygiene department, not one hygienist. Exclude doctor exam codes. | Currency |
| `totalProd` | Total practice production (last 12 months) | Total production for the whole office, all providers. | Currency |
| `hygDays` | Total hygiene days worked per month (whole department) | Add up the days every hygienist works in a typical month. Example: 2 hygienists each working 4 days a week = 16 days each = **32** hygiene days per month. | Integer |
| `prophy` | Adult Prophy (D1110) | Number of times your team completed this procedure in the last 12 months. | Integer |
| `srp` | Scaling & Root Planing (D4341 + D4342) | Combined count of D4341 and D4342 completed in the last 12 months. | Integer |
| `pm` | Perio Maintenance (D4910) | Number of times your team completed this procedure in the last 12 months. | Integer |

Put a worked example next to `hygDays`, because this is the field most likely to be entered wrong (people enter one hygienist's days instead of the department total).

**Sanity warnings (soft, non-blocking, shown inline):**
- `hygProd` > `totalProd`: "Hygiene production is higher than total practice production. Please double-check."
- `hygDays` < 4 or > 200: "That looks unusual for a monthly department total. Remember to add up days for all hygienists."
- Perio % calculates above 70%: "That's unusually high. Double-check that your procedure counts are for 12 months."
- Production per day above $4,000: "That's unusually high. Make sure 'hygiene days' is the total for all hygienists."

---

## 4. Calculations (source of truth)

These formulas were extracted directly from the calculation scripts inside Inspired Hygiene's existing PDF worksheet, plus one new metric Jennifer specified. **Do not alter them.**

```
// 1. Hygiene Profitability Ratio  (display as "X.X : 1")
profitRatio = hygProd / comp

// 2. Production per Hygiene Day  (display as currency, no cents)
prodPerDay = (hygProd / 12) / hygDays

// 3. Hygiene % of Total Office Production  (display with 1 decimal + "%")
hygPct = (hygProd / totalProd) * 100

// 4. Perio Percentage  (display with 1 decimal + "%")
perioPct = ((srp / 2) + pm) / (prophy + pm + (srp / 2)) * 100
```

Notes:
- The Perio % formula intentionally counts SRP as `srp / 2`. That is Inspired Hygiene's existing method (SRP is billed per quadrant, so the count is adjusted toward patients). Keep it, and show a small "How we calculate this" tooltip with the formula in plain English.
- If any denominator is 0 or empty, do not show `Infinity`, `NaN`, or 0. Show "Need more data" on that card and skip it in the AI analysis.
- Round only for display. Keep full precision for status logic.
- Put all formulas in one pure function module (e.g. `lib/calculations.ts`) with unit tests (Section 10).

---

## 5. Benchmarks and status logic

Store every threshold in a single config file (e.g. `config/benchmarks.json`) so Inspired Hygiene can change goals without a developer touching the logic. Status colors: Below = red/coral, On Goal = Inspired Hygiene blue, Above = gold, plus green for Perio "Ultimate Goal".

### 5.1 Hygiene Profitability (Goal 3:1)
Net hygiene production vs. total hygiene compensation.
- `< 3.0` = **Below goal** → use "Below 3:1" questions
- `>= 3.0` and `<= 3.2` = **On goal**
- `> 3.2` = **Above goal** → use "Above 3:1" questions

### 5.2 Production per Day (Minimum goal $1,200 to $1,500)
Net production per hygienist per day, excluding doctor exams.
- `< 1200` = **Below goal**
- `1200 to 1500` = **On goal**
- `> 1500` = **Above goal**

### 5.3 Hygiene % of Total Office Production (Goal 25% to 30%)
- `< 25` = **Below goal**
- `25 to 30` = **On goal**
- `> 30` = **Above goal**

### 5.4 Perio Awareness (Foundation: 30% or higher)
% of hygiene patients receiving perio therapy (SRP D4341/D4342) or maintenance (D4910).
Display as a pyramid/tier graphic like the current handout:
- `0 to 20` = **Needs Immediate Attention** (red)
- `20 to 30` = **Below Foundation** (orange)
- `30 to 35` = **Great Foundation** (blue)
- `35 to 50` = **Ultimate Goal** (green)
- `> 50` = **Ultimate Goal** (green) plus the "unusually high, double-check" soft warning only if > 70

Question routing: under 30 uses "Below 30%" questions; 30 to 35 uses "30 to 35%" questions; every result also gets the "At any level" notes.

---

## 6. Inspired Hygiene question bank (the AI may only use these)

This is Inspired Hygiene's intellectual property, taken word-for-word from the Hygiene Metrics Quick Reference. Store it in `config/question-bank.json`. The AI selects from it and may lightly rephrase for flow, but must not invent new coaching questions.

**Profitability, below 3:1**
- What services beyond prophy and X-rays is the team performing? Any adjuncts?
- What is the perio percentage? Are SRP codes used appropriately?
- Is there room to optimize the hygiene service mix and schedule efficiency?

**Profitability, above 3:1**
- When did the hygienist(s) last receive a pay increase?
- What benefits are included? (Medical, 401k, bonuses?)
- What's being done proactively to retain these hygienists?

**Production per day, below goal**
- What is the service mix? Primarily prophy and X-rays only?
- How much open time is in the schedule? What is the cancellation rate?
- Are there any low-reimbursing PPO plans worth stepping out of?
- Have fees been analyzed for a potential increase?

**Production per day, above goal**
- Is this net production? Net is the only collectible number.
- Are all hygienists producing at the same level?
- If hygiene is over 3:1 profitability, there may be room for a production-based incentive program.

**Hygiene % of total production, below 25%**
- Is the doctor a high-volume specialist or super-GP? (Explains a low %, but doesn't eliminate opportunity.)
- Is there heavy reliance on temp hygienists? Temps tend to underdiagnose.
- How far out is the schedule booked? Backlogs suppress diagnosis.

**Hygiene % of total production, above 30%**
- Are hygienists consistently teeing up restorative before the doctor enters for the exam?
- Are they taking photos, presenting plans, and starting the diagnostic conversation?
- Is the doctor doing hygiene in their own column? (Reduces doctor production.)

**Perio awareness, below 30%**
- Are you aware that about 50% of adults 30+ have some form of gum disease? (CDC/AAP)
- What are the documented diagnosis standards across all hygienists?
- How far out is the schedule? A 3+ month backlog drives underdiagnosis.

**Perio awareness, 30% to 35%**
- Are all hygienists contributing equally, or is one carrying the others?
- Is the number driven by maintenance (D4910) or active therapy (D4341/D4342)?
- Are scheduling systems in place to support perio therapy?

**Perio awareness, at any level**
- There is always room to elevate, even at 30%+. Are diagnosis standards consistent?
- A high perio % is a clinical and financial win: less risk, better outcomes, recurring production.

**Why it matters (use in the summary)**
Clinical risk (undertreated disease = liability), production risk (lost recurring revenue), and patient outcomes are all tied to perio awareness. At any level, there's almost always opportunity to optimize diagnosis, communication, and treatment standards.

---

## 7. The agentic layer (AI analysis)

### 7.1 What the AI receives
After calculation, the app sends the model a structured JSON payload only. Never send raw form text.

```json
{
  "practice": { "name": "...", "role": "...", "hygienists": 2 },
  "metrics": {
    "profitRatio": { "value": 2.6, "display": "2.6 : 1", "goal": "3 : 1", "status": "below" },
    "prodPerDay":  { "value": 812.5, "display": "$813", "goal": "$1,200 to $1,500", "status": "below" },
    "hygPct":      { "value": 21.7, "display": "21.7%", "goal": "25% to 30%", "status": "below" },
    "perioPct":    { "value": 11.8, "display": "11.8%", "goal": "30%+ (35 to 50% ultimate)", "tier": "needs_immediate_attention" }
  },
  "eligibleQuestions": { "profitRatio": ["..."], "prodPerDay": ["..."], "hygPct": ["..."], "perioPct": ["..."] }
}
```
The app pre-filters `eligibleQuestions` from the question bank using the status logic, so the AI can only choose from the right set.

### 7.2 System prompt for the analysis model
Use this verbatim:

> You are the Hygiene Analysis assistant for Inspired Hygiene, a coaching company that helps dental practices elevate their hygiene departments. You will receive a practice's calculated hygiene metrics and a list of approved coaching questions.
>
> Write a short, warm, professional analysis for the practice owner. Rules:
> 1. Never calculate, estimate, round differently, or invent any number. Use only the `display` values provided.
> 2. Never diagnose patients or give clinical treatment advice. You are discussing business and system metrics.
> 3. Use only questions from `eligibleQuestions`. You may lightly rephrase for flow. Do not add new coaching questions.
> 4. Every recommendation must create a win for patients, the team, and the practice. Never suggest quantity over quality, upselling, or pushing treatment patients do not need.
> 5. Do not shame. Frame gaps as opportunity. Acknowledge strengths first.
> 6. If a metric is marked "Need more data," skip it and say so in one line.
> 7. Do not mention pricing, guarantees, or specific revenue promises.
>
> Output format (JSON):
> - `headline`: one sentence summarizing where this hygiene department stands.
> - `strengths`: 1 to 2 sentences on what is working (if anything is on or above goal).
> - `biggestOpportunity`: the single metric with the most room to grow and why, in 2 to 3 sentences. Perio awareness takes priority when it is below 30%.
> - `metricInsights`: for each metric, 1 to 2 sentences of interpretation plus 2 questions from the approved list "to discuss with your team."
> - `nextStep`: 1 to 2 sentences inviting them to a discovery meeting with Inspired Hygiene to build a plan. Not salesy.

### 7.3 Implementation notes
- Use structured output / JSON mode and validate the response against a schema. If the AI call fails or times out (8 seconds), show the metric cards plus a static fallback: the relevant questions from the bank, listed under each card. The results page must never break because of the AI.
- Cache the AI result with the submission so the PDF and email use the same text.
- Model: any current Claude model via API (server-side only; never expose the API key in the browser).

### 7.4 Optional phase 2: "Ask about my results" chat
A chat panel on the results page, scoped to this submission's data and the question bank, with the same guardrails. Hard cap of 10 messages per session. Every 3rd reply ends with the discovery meeting link.

---

## 8. Outputs and integrations

### 8.1 PDF report
One to two pages, branded, generated server-side:
- Header: Inspired Hygiene logo, "Your Hygiene Analysis," practice name, date.
- 4 metric cards (value, goal, status).
- Perio tier pyramid with their position marked.
- AI analysis text.
- "Our Standard" strip.
- CTA block: "Ready to elevate hygiene? Schedule a discovery meeting." + booking link + QR code, Clients@InspiredHygiene.com, (704) 541-0995, InspiredHygiene.com.
- Podcast block: "The Inspired Hygiene Podcast with Rachel Wall" + link.

### 8.2 Email
Send the PDF to the user immediately after Step 3 (transactional email). Subject: "Your Hygiene Analysis Results from Inspired Hygiene". Send an internal notification to Clients@InspiredHygiene.com with a results summary.

### 8.3 CRM (GoHighLevel)
On submission, POST to a GoHighLevel inbound webhook (URL provided as an environment variable `GHL_WEBHOOK_URL`). Payload: all contact fields, referral source, all 7 raw inputs, all 4 calculated values, all 4 statuses, the AI `biggestOpportunity` text, and a link to the PDF. Brighter Idea will map these to GHL custom fields and tags inside GHL (e.g. tag `ha-perio-below-20`, `ha-profit-below`, `ha-completed`). Also fire a separate webhook event when someone completes only Step 1 and leaves (for anonymous stats, no contact data).

### 8.4 Data storage
Store every submission (inputs, outputs, AI text, timestamp, UTM parameters, referral source) in the app's database. Build a simple password-protected admin page listing submissions with search and CSV export.

---

## 9. Design and brand

- Match the existing "Hygiene Metrics Quick Reference" handout.
- Colors: primary sky blue (approx. `#3DB8E5`), accent gold/yellow (approx. `#F6CF5E`), dark gray text (`#3A3A3A`), white backgrounds, light blue tint panels (`#EAF6FC`). Confirm exact hex codes from the brand kit.
- Typography: bold, uppercase geometric sans for headings (Montserrat or similar), clean sans for body.
- Logo: Inspired Hygiene "IH" circle mark + wordmark (asset to be provided).
- Metric cards: numbered 1 to 4, blue header bar, gold goal badge on the right, exactly like the handout.
- Fully responsive. Must work well on a phone, since many users arrive from a QR code on the printed handout.
- Embeddable: provide an iframe-friendly route (`/embed`) so the calculator can be dropped into the Inspired Hygiene website and a GoHighLevel funnel page.
- Accessibility: labels on every input, status never shown by color alone (always include the word Below / On Goal / Above).

---

## 10. Test cases (the build is not done until all pass)

**Test A: sample data from the existing PDF worksheet**
Inputs: comp 100,000 / hygProd 300,000 / totalProd 10,000,000 / hygDays 32 / prophy 895 / srp 15 / pm 20
Expected: profitRatio **3.0 : 1** (on goal) / prodPerDay **$781** (below) / hygPct **3.0%** (below) / perioPct **3.0%** (needs immediate attention)

**Test B: healthy practice**
Inputs: comp 180,000 / hygProd 600,000 / totalProd 2,200,000 / hygDays 32 / prophy 2,400 / srp 600 / pm 900
Expected: profitRatio **3.3 : 1** (above) / prodPerDay **$1,563** (above) / hygPct **27.3%** (on goal) / perioPct **33.3%** (great foundation)

**Test C: struggling practice**
Inputs: comp 150,000 / hygProd 390,000 / totalProd 1,800,000 / hygDays 40 / prophy 3,000 / srp 200 / pm 300
Expected: profitRatio **2.6 : 1** (below) / prodPerDay **$813** (below) / hygPct **21.7%** (below) / perioPct **11.8%** (needs immediate attention)

**Test D: missing data**
Inputs: comp 0, all else as Test B.
Expected: Profitability card shows "Need more data"; other three cards calculate normally; AI skips profitability; no `Infinity` or `NaN` anywhere.

**Test E: formatted entry**
Entering `$1,200,000.00` in a currency field is parsed as `1200000`.

**Test F: AI failure**
Simulate the AI call failing. Results page still shows all four cards plus fallback questions, and the PDF still generates.

---

## 11. Out of scope (do not build)

- Hygiene Open Time and the Prophy : Perio Maintenance ratio from the old PDF. Inspired Hygiene confirmed these are not part of the current metrics guide.
- User accounts / logins for practices (phase 2 consideration).
- Multi-location roll-ups (phase 2).

---

## 12. Open decisions (confirm with Inspired Hygiene before launch)

1. **Benchmark conflicts between the old worksheet and the new Quick Reference.** The old PDF said Hygiene % goal is 30 to 35% and Perio goal is 35%+. The new Quick Reference says Hygiene % goal is 25 to 30% and Perio foundation is 30%+, with the calculator graphic using 0-20 / 20-35 / 35-50 tiers. This spec uses the new Quick Reference. Confirm, and confirm the 20 to 30% Perio label.
2. **"On goal" band for Profitability.** The handout only says below or above 3:1. This spec treats 3.0 to 3.2 as on goal. Confirm or adjust.
3. **SRP divided by 2** in the Perio % formula. Confirm with Rachel this is the intended method.
4. **Booking link** for the discovery meeting CTA (current scheduling tool vs. a GoHighLevel calendar).
5. **Phone number:** Quick Reference shows (704) 541-0995; the old worksheet shows 877-237-7230. This spec uses (704) 541-0995.
6. **Podcast link** URL.
7. **App name.** Working title "Hygiene Analysis." Alternatives: "Hygiene Health Check," "Perio Percentage Calculator + Hygiene Scorecard."
8. **Gate placement.** This spec gives Perio % free and gates the full report behind contact info. Confirm.
