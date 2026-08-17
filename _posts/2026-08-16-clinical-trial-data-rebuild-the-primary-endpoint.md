---
title: "Before Modelling Clinical Trial Data, Rebuild Its Primary Endpoint"
description: "A trial's analysis columns are answers to the question that trial asked, not raw measurements. Working through CDISC's public pilot submission to find where those decisions are recorded, and which of them a different question can inherit."
date: 2026-08-16
layout: post
tags: [pin, clinical ml, reproducibility]
---

Imagine someone hands you a clinical trial dataset. One row per subject, forty-eight columns,
no free text, no duplicate keys, and across two hundred and fifty-four rows exactly one missing
value. There is nothing to clean. **It is the tidiest table you have ever been given, and it is
tidy because it was already used to answer a question.**

Now suppose there are five of them, from five finished trials, and one model you want to train
across all five. Your question will not be the question any of those trials asked. The column
names will still line up, because lining up names is what the standard is for. What sits
underneath them was decided one protocol at a time.

**The short version:** those decisions are written down, every one of them, and none of them
travel with the table. Which subjects count. Which record of a repeated measurement counts.
What happens when someone stops turning up. What a visit label means. Reproducing the trial's
own published answer checks that you have read them as they were meant, and only then can you
tell which your question inherits and which you have to make again.

## A CDISC submission: study report, SDTM and ADaM datasets, and define.xml linking them

The dataset above is real and public: the
**[CDISC SDTM/ADaM Pilot Project](https://github.com/cdisc-org/sdtm-adam-pilot-project)**, a
complete submission package for a fictional Alzheimer's study. It contains three things, and
the relationship between them is the point:

- **a 492-page study report**, with the actual tables, and, near the back, the raw
  statistical output behind several of them;
- **the datasets**, in two layers: tabulations that sit close to what was collected, and
  analysis datasets shaped for the specific analyses the protocol called for;
- **`define.xml`**, one file at each layer, saying for every column where it came from and how
  it was derived, and linking each table in the report back to the dataset, the rows and the
  variables that produced it.

That third file is the connection. Given a number in the report, it tells you which dataset to
open, which rows to select, and which column to analyse. It is a deliberate, long-standing
piece of engineering, and it is the only reason someone outside a sponsor company can do any of
what follows.

Laid out, with what this package actually holds at each step:

![How the layers of the submission connect, with counts](/assets/images/clinical_field_notes/adam-how-it-connects.png)

Each layer is derived from the one above it, and narrows as it goes. The questionnaire domain
alone holds 121,749 collected rows; the analysis dataset built from it holds 12,463; the set
that feeds the primary endpoint is 234, one per subject in the efficacy population. `define.xml`
sits alongside all of it rather than inside any one layer.

One caveat, stated once: this package is a 2013 refresh of a pilot first assembled in 2007 and
published as a teaching example. It is not a claim about how any current submission looks.

## Rebuilding the primary endpoint, and what a match rules out

Reading the metadata tells you what a column means. It does not tell you whether you read it
correctly, and a misreading is silent. So rebuild what the report prints and compare: a rebuilt
value can only agree if the same subjects were selected, the same record taken per visit,
missing visits handled the same way, and the same column analysed. Each quantity that matches
rules out readings that would have produced something else.

How strict the check can be depends on what the report gives you to compare against. A rounded
p-value is three digits of agreement. The table around it adds counts, means, standard errors
and confidence limits. The statistical output printed near the back adds sums of squares to six
decimal places and adjusted means to eight. This package carries all three levels, and the
sections below work through them.

The starting point is the study's primary endpoint: change from baseline in ADAS-Cog (11) at
Week 24. Higher is worse on that scale, so a positive change means the subject declined. The
report gives a dose-response p-value of 0.245.

Rebuilt from the analysis datasets, against Table 14-3.01 as printed:

|                                        | report          | rebuilt            |
| -------------------------------------- | --------------- | ------------------ |
| subjects per arm, placebo / low / high | 79 / 81 / 74    | 79 / 81 / 74       |
| mean change from baseline              | 2.5 / 2.0 / 1.5 | 2.54 / 2.00 / 1.47 |
| **p-value, dose response**             | **0.245**       | **0.2447**         |
| low minus placebo, difference (SE)     | −0.5 (0.82)     | −0.47 (0.82)       |
| high minus placebo                     | −1.0 (0.84)     | −1.01 (0.84)       |
| high minus low                         | −0.5 (0.84)     | −0.54 (0.84)       |

Getting there took the analysis metadata, the report's footnotes and its printed statistical
output, read together. The parameter code is given in the metadata's parameter list. The stored
visit label carries padding that has to be trimmed before a string comparison matches. The model
specification, including a baseline covariate, is stated in the table's footnote and visible in
the statistical output printed near the back of the report.

Two of those details can be measured directly. Fitting the model from the analysis metadata
alone returns 0.2532; fitting it with the baseline covariate stated in the footnote returns
0.2447, which is the printed 0.245. The adjusted-means statement specifies that the site term be
averaged by the number of subjects each site contributed rather than weighting all eleven
equally; applied that way the adjusted means match the printed values to eight decimal places,
and weighted equally they differ in the second decimal.

**Neither partial specification raises an error. Both return a number.**

## What an MMRM is, and how to read the trajectory

The primary endpoint is one timepoint. The trial also ran a repeated-measures analysis across
all three post-baseline visits, which in this field means an **MMRM**: a mixed model for
repeated measures.

Worth a paragraph, because an ML reader will meet it constantly and it is not a model most of
us are taught. Each subject is measured several times, so their measurements are correlated
with each other and cannot be treated as independent rows. An MMRM fits every visit at once,
estimates the correlation between a subject's own visits directly, and uses whatever
observations exist without filling anything in. Where LOCF answers a missing visit by
substituting an earlier value, an MMRM answers it by leaving the row out and letting the
correlation structure carry the information.

The model reproduces against the statistical output the report prints, down to the covariance
parameters and all four printed digits of the adjusted means, so it can be drawn per visit.
Trajectory plots like this are routine in clinical publications; this package holds the analysis
table rather than the graphics, so the picture is built rather than reproduced.

![ADAS-Cog change from baseline over time, by arm](/assets/images/clinical_field_notes/adam-mmrm-trajectory.png)

Reading it, for anyone whose instincts come from elsewhere:

All three curves rise, because the disease is progressive and the scale measures deficit. A
trial like this never asks whether subjects improve, only whether the treated arms climb more
slowly than placebo. I had arrived with a "did the patient get better" frame, under which the
figure appears to say the drug failed and so did the placebo, which is not what it says.

At Week 24 both active arms sit below placebo, about a point less worsening, and none of it
reaches significance: treatment p = 0.8184 in the model, 0.245 for dose response at Week 24,
every pairwise comparison above 0.2. There is no dose ordering either. At Week 8 the low dose
is the worst of the three arms and the curves cross twice.

The separation at the right-hand edge is mostly attrition, which the report says plainly in its
conclusions. The plot shows observed cases, so the denominator shrinks unevenly underneath it,
and by Week 24 the active arms have lost roughly half their subjects. Neither the subject count
per visit nor the reason people left is anywhere on the plot.

## What "Week 24" selects, and why a label belongs to its question

<mark>A column in an analysis dataset is not a measurement; it is an answer to a question, and
its name describes the answer.</mark>

The analysis selects `AVISIT == "Week 24"`, and the numbers matched, so the filter is correct.
Of the 234 rows it selects, **115 were recorded at a visit called Week 24.** The rest come from
Week 2, Week 4, Week 6, Week 8, Week 12, Week 16, Week 20, a visit called RETRIEVAL, and one
called AMBUL ECG REMOVAL.

Two mechanisms put them there, and the dataset states both, in columns sitting beside the ones
I was reading. An analysis window: `AWRANGE` says `>140`, so any assessment on study day 141 or
later is the Week 24 analysis point whatever the visit was called. And carry-forward: 79
subjects stopped coming, so their last observed value stands in.

`AVISIT` means precisely what it is defined to mean: which analysis timepoint this row
represents. Every one of those rows genuinely is that subject's Week 24 analysis value, and for
a carry-forward analysis that is the entire point. It only misleads if you read it as "when was
this measured", which is a different question, answered by the column next to it.

Once you have that, the same shape is everywhere in the package. Each of these is a decision,
each deliberate, each recorded, none visible from a column name:

| decision                     | what was chosen                                       | what it does                                                            |
| ---------------------------- | ----------------------------------------------------- | ----------------------------------------------------------------------- |
| which subjects               | ITT 254, safety 254, efficacy 234                     | different denominators, and one is defined by post-randomisation events |
| which measurement            | one `AVAL` column, fifteen parameter codes            | wrong code, right column name, meaningless numbers                      |
| which record per visit       | 257 rows narrowed to 234                              | repeats inside one visit window                                         |
| missing visits               | 79 of 234 carried forward; the NPI-X endpoint, none   | one endpoint looks complete, another is half absent                     |
| what a visit label means     | a window, plus carry-forward                          | the timepoint is nominal                                                |
| direction of the scale       | higher ADAS-Cog is worse                              | coefficients read backwards                                             |
| when a value became knowable | 19 of 48 subject-level columns are post-randomisation | leakage                                                                 |

## Two questions, two tables

The trial demonstrates this before any ML reader arrives. It asked two questions about the same
endpoint, and needed two tables to answer them: **the primary endpoint analysis runs on 234
rows, one per subject with values carried forward, and the MMRM runs on 539, observed only.**
Same study, same column, same six months, two questions, two datasets.

Now take two questions an ML practitioner might bring to it.

**Does baseline profile predict cognitive decline at six months?** The target is change at
Week 24, the features are demographics, disease history, baseline cognition. This inherits most
of the trial's decisions, because it is close to the question the trial asked. It gets 234 rows.
It also inherits the carry-forward: for 79 of those subjects the "six month" outcome is a
measurement taken earlier, so the model is partly learning about people who left.

**Who leaves the study early?** The target is discontinuation before Week 24, and now everything
inverts. `TRTDUR` and `COMP24FL` were leakage in the first question; here one of them _is_ the
target. The Week 8 assessment was an outcome in the first question; here it is a legitimate
feature, because all 234 subjects have an observed Week 8 and all 116 of the eventual leavers
were still present for it. And the carry-forward that was acceptable in the first question is
now fatal, since the imputed value is a direct function of the thing being predicted.

Same package. Two questions. Two different tables, two different row counts, and a column that
is poison in one and the label in the other.

This is why the sorting I did into technical, result and feature is worth having, and why
**the sort is not a property of the columns but of a column and a question together.** What the
metadata gives you is timing and derivation. Your question decides the role, and the trial
cannot decide it for you, because it was busy answering its own.

## Take-aways before you model clinical trial data

- **The data was shaped by a question, and the shape does not travel with the table.** Which
  subjects, which record, what a label means, what happens to people who left.
- **Reproducing the published results tests your reading of those decisions.** The more
  quantities match, and to more digits, the fewer wrong readings survive.
- **A partial reading returns a number, not an error.** The specification is spread across the
  metadata, the footnotes and the printed output, and using part of it still computes.
- **A label can be correct and still not mean what you assumed.** "Week 24" is a window plus a
  carry-forward rule, so only 115 of the 234 rows were measured at Week 24.
- **There is no context-free feature table.** Two questions on one study need two tables. Five
  studies pooled need five sets of decisions checked before the columns can be stacked.

The table at the end is the artifact, but the trace back is the method, and the method is
borrowed. Writing down where every analysis value came from, publishing the raw model output
beside the table, pre-specifying how missing visits get handled: that is decades of accumulated
discipline from a field that had to make its results checkable by strangers. I spent a few days
reading one example of it. What I got was not a shortcut past that expertise. It was a way to
let it tell me which of my columns were honest.

---

_The notebook works through all of this, including the visit-window mechanism above, and writes
out the feature table and column dictionary:
[03-adam-traceability](https://github.com/lecaibio/clinical-data-field-notes/tree/main/03-adam-traceability).
It downloads the package itself and runs in about ten seconds._
