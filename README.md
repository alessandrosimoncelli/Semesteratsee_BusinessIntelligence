# Semester at Sea: Leads Funnel and Enrollment Intelligence

A Power BI model built on a live Azure MySQL database. It takes raw operational
data from a 630 capacity study abroad program and turns it into a dashboard that
catches a misleading trend before it reaches a board meeting.

## The core finding

Every conversion metric improved year over year. Applications fell by a third.
At first glance that looks like a quality story: fewer applicants, but better
ones. It isn't. The improvement is a selection artifact. Top of funnel
abandonment nearly doubled, which quietly filtered out weak applicants before
they could pull the conversion rate down. Once that is accounted for, the real
problem is volume, not quality, and the right fix is re engaging people who
started an application and never finished it, not tightening the funnel
further.

One comparison makes this concrete: bringing abandonment back down to prior
year levels recovers roughly 35 times more students than trying to squeeze
further gains out of a conversion rate that is already at 82% and running out
of applicants left to convert.

## What I actually built and learned doing it

**Learned to check a metric before trusting it.**
A deposits measure was showing fewer deposits than confirmed bookings, which
is logically impossible. Tracing it back, the date field it relied on was
blank for every historical record in the database. The fix wasn't just a
formula change. It was learning to build a sanity check into every important
measure (for example, deposits should never exceed confirmations) before
trusting what it says.

**Learned to test whether a strong pattern is actually causal.**
Room type showed the single largest abandonment gap in the whole dataset, over
40 percentage points, the kind of number that is tempting to feature
prominently. Cross checking it against application status showed that the
room type was a default value assigned automatically before any real choice
was made, not something the applicant actually decided. Using it would have
meant predicting abandonment with a variable that is really just a side
effect of abandoning. Recognizing that difference, and being willing to
remove the best looking number in the dataset once it failed that test, was
one of the most useful skills this project reinforced.

**Learned to build and check a scoring model without labeled data.**
Two weighted scores were built, one for how likely an applicant is to convert
and one for how likely they are to abandon. Each variable's weight was set
based on how strong its real, measured effect was in the data, not chosen by
feel. Both scores were then checked by grouping applicants into High, Medium,
and Low tiers and comparing outcomes across those groups. The results are
reported honestly as a lift, meaning how much more likely the high group is to
convert or abandon compared to the average, rather than presented as a
formally validated statistical model.

**Learned that a flat result can be the actual finding.**
Three enrollment forecast scenarios, a conservative one, a middle one, and an
optimistic one, all landed within about ten students of each other. The
instinct is to see that as an uninteresting result. It is actually the
sharpest one, because it shows that the remaining applicant pipeline is too
small for conversion improvements to change the outcome in any meaningful
way. That finding is what pushes the recommendations toward getting more
applicants in the door, rather than converting the ones already there more
efficiently.

**Learned to design a dashboard that shows reasoning, not just conclusions.**
The pages that look at conversion and abandonment predictors are split into
what worked and what didn't, side by side, so the variables that were tested
and ruled out stay visible instead of quietly disappearing. Every chart title
states a conclusion instead of just describing what is on the axis. The goal
was for someone looking at the dashboard for the first time to be able to
follow the analysis, not just read the final answer.

## What the dashboard contains

A five page executive report built for direct presentation to Semester at
Sea's admissions and marketing leadership. It covers year over year funnel
performance, the traits that actually predict conversion and abandonment
versus the ones that look interesting but don't hold up, and a three scenario
enrollment forecast. It closes with two specific, data backed recommendations
tailored to the admissions and marketing teams.

## Technical scope

Modeling: a star schema built from eight raw operational tables coming from a
live MySQL source. This included finding and fixing a grain mismatch where
the main fact table had one row per booking rather than one row per
application, an error that was silently double counting results anywhere a
measure used COUNTROWS instead of DISTINCTCOUNT on application ID.

DAX: the two scoring columns are built with dynamic weight redistribution, so
if one input variable is missing for an applicant, the remaining variables
are reweighted automatically instead of returning a broken or misleading
score. There is also a minimum data threshold so a score is not calculated
when too little information is available. All measures live in dedicated
measurement tables rather than being scattered across visuals.

Format: the entire semantic model is written in TMDL, which makes it
readable and version controllable as plain text.

## Known limitations, stated on purpose

The scores are validated using the same data they were built on. A real
predictive claim would require testing them against a future cohort that
wasn't used to build the model. The forecast assumes the applicant pipeline
is fixed as of the date the data was pulled. And one variable with a very
strong looking effect, room type, was deliberately left out after failing a
causality check, which is worth mentioning because the temptation to keep it
in was real.

## Stack

Power BI Desktop, Power Query (M), DAX, TMDL, Azure Database for MySQL
