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

## From raw data to finished dashboard

This project covers the full lifecycle of a business intelligence build, not
just the final visuals. Below is what that process actually involved, in
order.

**Connecting to the source.**
The data lived in a live Azure hosted MySQL database with eight separate
operational tables covering applications, bookings, applicant details,
citizenships, room types, terms, and accounting transactions. Power Query was
connected directly to the database rather than working from a static export,
so the model reflects the live state of the data.

**Cleaning the raw data.**
Country names were inconsistent across records (for example "USA", "United
States of America", and "United States" all appearing separately), so a
standardization function was built and applied wherever citizenship shows up.
Junk rows, placeholder records, and a few invalid IDs were identified and
removed. Combined date and time fields were split into clean date columns,
and currency fields stored as text with dollar signs were converted into real
numbers.

**Merging and joining the data.**
Applications and bookings lived in separate but related tables and had to be
joined correctly, along with a bridge table connecting applications,
bookings, and individual applicants. Getting the join direction or row grain
wrong here would have quietly broken every calculation downstream.

**Building the data model.**
The cleaned tables were organized into a star schema: one central fact table
surrounded by clearly separated dimension tables for people, room types, and
terms. Two calendar tables were built manually, using year independent
columns so year over year comparisons land on the same calendar day
regardless of which year is selected.

**Auditing the calculations.**
Every measure was checked against logic that should always hold true before
being trusted. This caught real issues, including a deposits measure showing
fewer deposits than confirmed bookings, which is impossible, and a
mislabeled acceptance rate that was actually measuring something else. Each
was traced to its root cause and corrected.

**Engineering new fields for analysis.**
Applicant age was calculated at the time of application, not today's date,
so past years stayed accurate. Citizenship was grouped into broader world
regions to surface patterns that were otherwise buried across dozens of
individual countries.

**Analyzing the funnel and testing predictors.**
Each funnel stage was compared year over year using the same calendar window
in both years, to remove seasonality from the comparison. Applicant traits
such as returning status, room choice, dietary acknowledgment, and region
were tested one at a time to see which ones actually predicted conversion or
abandonment, and which only looked interesting without holding up.

**Building predictive scoring models.**
Two weighted scores were built from the traits that passed testing: one for
likelihood to convert, one for likelihood to abandon partway through.
Weights were set from the strength of each trait's real effect in the data,
and both scores were validated by checking whether applicants grouped into
risk tiers actually behaved differently in practice.

**Forecasting the outcome.**
Three enrollment scenarios were modeled for the upcoming term based on
different conversion assumptions for the remaining pipeline, and compared
directly against the program's enrollment target.

**Designing the executive dashboard.**
The final output is a multipage report meant to be read by leadership
directly, not just by another analyst. Predictor pages are laid out to show
both what worked and what was ruled out, chart titles state findings instead
of describing axes, and the whole report closes with two specific,
data backed recommendations for the admissions and marketing teams.

## What makes this build worth looking at

**Knowing how to connect to a real, live data source.**
Power Query was connected directly to a live Azure hosted MySQL database
rather than working from a static export or a spreadsheet someone handed
over. That distinction matters. A static file is already dead by the time
you get it, while a live connection means the model reflects the actual
current state of the data and needs to be built in a way that keeps working
as the underlying data keeps changing. Setting up that connection and
designing the queries around it is a different skill than cleaning a file
that was already prepared for you.

**Handling messy, real world data instead of a tidy sample.**
The raw tables had the kind of problems that only show up in a real
operational database: the same country spelled three different ways, dates
stored as combined text instead of proper date values, currency values
stored as text with dollar signs attached, and leftover placeholder rows
that were never meant to be treated as real records. None of this is
glamorous work, but it is the work that determines whether every number
built on top of it can actually be trusted. Writing a reusable
standardization function instead of fixing values one by one, and knowing
which rows to remove versus which ones just needed retyping, was as
important to this project as any of the analysis that came after it.

**Designing a model, not just connecting tables.**
Getting a working join between tables is one thing. Deciding the right grain
for the fact table, splitting people, rooms, and terms into their own
dimension tables, and building calendar tables by hand instead of relying on
a default one, is a different level of decision making. It also meant
catching a real mistake mid build, where the fact table was accidentally set
up at the wrong grain, one row per booking instead of one row per
application, which was quietly inflating every count downstream until it was
caught and fixed.

**Checking a metric before trusting it.**
A deposits measure was showing fewer deposits than confirmed bookings, which
is logically impossible. Tracing it back, the date field it relied on was
blank for every historical record in the database. The fix wasn't just a
formula change. It was learning to build a sanity check into every important
measure, for example deposits should never exceed confirmations, before
trusting what it says.

**Testing whether a strong pattern is actually causal.**
Room type showed the single largest abandonment gap in the whole dataset,
over 40 percentage points, the kind of number that is tempting to feature
prominently. Cross checking it against application status showed that the
room type was a default value assigned automatically before any real choice
was made, not something the applicant actually decided. Using it would have
meant predicting abandonment with a variable that is really just a side
effect of abandoning. Recognizing that difference, and being willing to
remove the best looking number in the dataset once it failed that test, was
one of the most useful parts of this project.

**Building and checking a scoring model without labeled data.**
Each variable's weight was set based on how strong its real, measured effect
was in the data, not chosen by feel. The results are reported honestly as a
lift, meaning how much more likely the high scoring group is to convert or
abandon compared to the average, rather than presented as a formally
validated statistical model.

**Letting a flat result be the finding.**
Three enrollment forecast scenarios, a conservative one, a middle one, and an
optimistic one, all landed within about ten students of each other. That is
actually the sharpest result in the whole forecast, because it shows the
remaining applicant pipeline is too small for conversion improvements to
change the outcome in any meaningful way, which pushes the recommendations
toward getting more applicants in the door rather than converting the ones
already there more efficiently.

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

## Author

Alessandro Simoncelli
