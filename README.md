# Semesteratsee_BusinessIntelligence
Power BI model built on a live Azure MySQL source that catches a misleading YoY conversion trend hiding a top-funnel abandonment crisis, includes weighted scoring models, star schema, and TMDL-versioned semantic model.


---

## The core finding

Every conversion metric improved year over year. Applications fell 33%. Read at
face value, that's a quality story: fewer, better applicants. It isn't. The
"improvement" is a selection artifact - top-of-funnel abandonment nearly doubled,
quietly filtering out weak applicants before they could pull the conversion rate
down. Once that's isolated, the real problem is volume, not quality, and the
fix is re-engagement, not tighter targeting.

The model proves this with one comparison: recovering abandonment back to prior
levels yields roughly 35× more students than squeezing further gains out of a
conversion funnel that's already 82% efficient and nearly out of applicants to
convert.

---

## What I actually built and learned doing it

**Learned to distrust a metric before trusting a trend.**
A "deposits" measure showed fewer deposits than confirmed bookings - logically
impossible. Tracing it back, the date field it relied on was silently null for
every historical record. The fix wasn't a formula tweak, it was recognizing that
a metric can be *directionally wrong* while still returning a plausible-looking
number, and building the habit of checking a measure against a sanity constraint
(deposits ≤ confirmations) before using it anywhere downstream.

**Learned to reject a strong-looking predictor on its own evidence.**
Room type showed the single largest abandonment premium in the dataset - 43
percentage points, the kind of number that's tempting to lead a slide with.
Cross-tabbing it against application status showed it was a system default
assigned before any real choice was made, not a behavioral signal. Leaving it in
would have meant predicting abandonment with a variable that is itself a symptom
of abandoning. Knowing how to test whether a signal is causal or circular - and
being willing to cut the best-looking number in the dataset once it fails that
test - was the most useful discipline this project reinforced.

**Learned to build and validate a scoring model without a labeled dataset.**
Two weighted composite scores (conversion likelihood, abandonment risk) with
weights set from measured premiums rather than picked by feel, tested against
High/Medium/Low buckets, and reported with an honest lift metric rather than an
implied statistical guarantee. Explicitly separating "this is a strong in-sample
signal" from "this is a validated predictive model" - and saying so out loud
instead of letting a confident chart imply more rigor than it has.

**Learned to let a flat forecast be the finding.**
Three enrollment scenarios (conservative / base / optimistic conversion) came
back within 10 students of each other. The instinct is to see that as a boring
result. It's actually the sharpest one: it proves the remaining pipeline is too
thin for conversion tactics to move the outcome at all, which redirects every
downstream recommendation toward acquisition instead. Recognizing when a "flat"
or "uninteresting" result is itself the insight - rather than something to dig
past - was a genuinely new instinct to build.

**Learned to design a dashboard as an argument, not a report.**
Predictor pages are split into what worked and what didn't, side by side, with
the eliminated variables left visible instead of quietly dropped. Every chart
title states a conclusion instead of describing an axis. The goal was to make
the analytical *process* legible to someone looking at the dashboard for the
first time, not just the conclusions.

---

## Technical scope

- **Modeling:** star schema built from 8 raw operational tables (live MySQL
  source), including catching and fixing a grain mismatch where the fact table
  was one row per *booking*, not per application - an error that silently
  double-counted every measure using `COUNTROWS` instead of
  `DISTINCTCOUNT(application_id)`
- **DAX:** weighted scoring columns with dynamic weight redistribution when
  input variables are missing, a minimum-component threshold to avoid scoring
  on insufficient data, and every measure isolated in dedicated
  measurement tables rather than embedded in visuals
- **Format:** TMDL throughout, for version control and reproducibility of the
  entire semantic model as text

---

## Known limitations - stated deliberately

- Scores are validated in-sample; a real predictive claim would need testing
  against a future, unseen cohort
- The forecast assumes the pipeline is closed at the data-pull date
- One high-premium variable was deliberately excluded after failing a
  causality check (see above) - worth noting because the temptation to keep it
  in was real

---

## Stack

Power BI Desktop · Power Query (M) · DAX · TMDL · Azure Database for MySQL
