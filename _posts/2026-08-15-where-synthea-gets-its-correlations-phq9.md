---
title: "Where Synthea Gets Its Correlations: Reading the PHQ-9 Module"
description: "Synthea's variables are correlated through care protocol, not through biology: good data for learning how care is delivered, bad data for learning what a measurement means."
date: 2026-08-15
layout: post
tags: [clinical ml, synthetic data]
# Frozen to the path this post was published under. The file was renamed after it went
# live; without this, the old URL 404s.
permalink: /2026/08/15/where-synthetic-clinical-data-gets-its-correlations.html
---

I ran the same depression questionnaire through a real national survey and through a
synthetic patient generator, compared the two, and then read the code that produced the
synthetic half.

**The short version:** Synthea's variables are correlated through care protocol, not through
biology. It is good data for learning how care is delivered and bad data for learning what a
measurement means. Prevalence, the number most people would check, comes out right either
way.

## The comparison

### The two datasets: NHANES and Synthea

**[NHANES](https://www.cdc.gov/nchs/nhanes/index.html)** is a real, population-level survey:
the CDC's National Health and Nutrition Examination Survey, which samples the US civilian
population, interviews people at home, and examines most of them in a mobile clinic. Public
microdata comes out in two-year cycles. It is a standard source for US prevalence estimates.

**[Synthea](https://github.com/synthetichealth/synthea)** is synthetic: an open-source
patient generator from MITRE that simulates each patient, in its own phrase, from cradle to
grave. Its clinical behavior lives in modules, one JSON state machine per condition or care
pathway, deciding what happens to a patient and when. No record belongs to a real person, so
there are no privacy restrictions, which is why it is common in health IT testing, demos, and
teaching.

One measures people. The other simulates records. Both get used to answer questions about
patients.

### One instrument (PHQ-9), two sources

The PHQ-9 is a nine-item depression questionnaire scored 0 to 27. A score of 10 or above
indicates moderate or worse depressive symptoms and is the conventional point at which a
clinician would follow up, so it is the threshold most analyses use.

For each source, three numbers: what share of adults were given the questionnaire, what share
of those scored 10 or higher, and the product of the two.

|                               | NHANES 2017–2018 | Synthea v4.0.0 |
| ----------------------------- | ---------------- | -------------- |
| given the questionnaire       | 91.6%            | 11.7%          |
| scored 10+, of those given it | 9.1%             | 63.8%          |
| scored 10+, of all adults     | **8.3%**         | **7.4%**       |

![Three rates, NHANES against Synthea](/assets/images/clinical_field_notes/01-three-rates.png)

The bottom row agrees to within a point. Neither row above it agrees at all: screening is off
by a factor of eight in one direction, the positive rate by a factor of seven in the other,
and the two errors cancel.

Validating a synthetic dataset, the one thing you would check is prevalence. That is the
number that matches.

### The distribution: where they disagree

The same data one level down: the full distribution of scores among people who were screened.

![PHQ-9 score distribution among those screened](/assets/images/clinical_field_notes/02-score-distribution.png)

NHANES looks like a screening instrument pointed at a general population. A third of
respondents score zero, the mass falls off steeply, the tail past 10 is thin. Synthea is a
flat slab, every score from 5 to 27 about equally common. No clinical measurement is
distributed this way.

## Where the numbers come from

### Reading the module: `depression_screening`

The explanation is about forty lines of JSON, in
`src/main/resources/modules/encounter/depression_screening.json`.

**When it fires.** It is called from `wellness_encounters.json` with probability 0.8 per
wellness encounter. Screening is routine, attached to the checkup.

**Who is excluded.** The module first drops patients under 12, patients who are pregnant,
patients already carrying a major depressive disorder diagnosis, and patients with an active
anti-suicide careplan.

**How the score is produced.** Adults get a PHQ-2 first, with a hard-coded 6% positive rate,
and only a positive proceeds to the PHQ-9. Of the adults who get that far, 80% take a branch
that sets the score to `UNIFORM(5, 27)`.

![The depression screening module as a flow](/assets/images/clinical_field_notes/04-module-flow.svg)

That last branch is the flat slab. The score is not the sum of nine simulated item responses,
nor a draw from a fitted distribution. It is a uniform random number between two integer
literals in a JSON file.

The arithmetic closes. Those two constants alone predict that **63.6%** of screened adults
should land at 10 or above. Observed: 63.8%. The second row of the table is one
multiplication.

Which leaves the bottom row, where a low screening rate times a high positive rate happens to
come out near the real prevalence. I don't know whether the constants were chosen to make
that happen or whether it fell out by luck. Either way, the number an analyst would
sanity-check is the one that is right.

### The same module, read as a protocol of care

Everything above describes the module as a data generator. Read as a description of care,
most of it is accurate.

Screening attached to the wellness visit rather than triggered by suspicion is what
population screening means. The two-stage design is real: the short PHQ-2 first, escalating
to the full PHQ-9 only on a positive, because that is how you screen a whole panel. The
instrument does switch to a teen version for adolescents. And the exclusion list is not an
oversight: you do not screen a patient already carrying a diagnosis or an active safety
careplan, because they are past screening and into monitoring.

So the same forty lines are a reasonable care pathway and an unusable score generator.

Synthea's module library is 242 files, 82 of which cite a specific guideline or source in
their remarks block, with named authors, version numbers, and stewards. There is a module for
adolescent idiopathic scoliosis built on SOSORT guidelines, and one for opioid prescribing in
chronic pain. Several carry "Module Steward: ONC" and a named contract developer.

That layer is hard to get anywhere else. Which finding in which patient triggers which order
is spread across specialty society PDFs, institutional protocols, payer rules, and practice
nobody wrote down. It sits on top of biology rather than restating it, encoding what the
system does about the science. <mark>The module library is Synthea's product; the synthetic
patients are its output.</mark> Most users take the output without opening the modules.

Both halves of the generator follow from this. The marginals are anchored to census and
public health data, so any single variable is plausible. The dependencies come entirely from
the modules, so what is correlated with what is correlated through the care pathway. Nowhere
in between is there a latent biological state producing a measurement and also driving an
outcome.

### What the score doesn't reach: no diagnosis, no prescription

Cradle to grave sets an expectation. In a real life, a patient who scores 20 on a depression
questionnaire is on the way somewhere: a diagnosis, a prescription, follow-up scores that
move. The disease develops and the record tracks it. That is what simulating a lifetime
sounds like it means.

In a two-year window, 516 adults scored 10 or higher, some at the top of the scale. In the
twelve months after those screens: zero depression diagnoses, zero antidepressant
prescriptions.

![What follows a positive screen in Synthea](/assets/images/clinical_field_notes/03-what-follows.png)

The reason is a one-line search. Grep the module tree for `phqa_score` and every hit is inside
the file that writes it. Nothing in Synthea reads the score.

The lifetime is there; the body underneath is not. Synthea simulates a life as a sequence of
encounters, each module firing on its own entry conditions and keeping its own state. There
is no shared patient for one module to write to and another to read from. A score of 24 does
not put a patient on a worse path than a score of 6, because there is no patient to be on a
path — only a record that now carries a number.

Antidepressants do exist in the output. Their recorded reasons are post-traumatic stress
disorder, major depression in the veteran module, and fibromyalgia, each ordered by a module
that reached its own conclusion, none of them by way of a screening score.

## What this means for ML

### What a model learns here: the score means nothing

In real records the chain runs biology → symptoms → measured value → outcome. A high PHQ-9 is
informative about what happens next because it is a noisy readout of a latent state, and that
same state drives the outcome. The measurement is predictive because it sits downstream of
something real.

In Synthea the chain stops. The score is conditionally independent of everything after it,
because nothing after it reads it. A model that learns `P(outcome | score)` here learns,
correctly for this data, that the score carries no information — the wrong answer for real
patients, on exactly the variables you care about.

### Where it holds and where it doesn't: workflow against patient state

| Holds up                                                       | Doesn't hold up                                                     |
| -------------------------------------------------------------- | ------------------------------------------------------------------- |
| Learning the conditional logic of ordering: what triggers what | Predicting a biological outcome given a measurement                 |
| Utilization and volume forecasting                             | Treating a lab or score as a proxy for patient state                |
| Coding, billing, claims logic                                  | Learning which measurements matter, or their thresholds             |
| Workflow and care-pathway simulation                           | Risk models meant to transfer to real patients                      |
| Format and terminology conformance, FHIR pipeline testing      | Estimating real-world prevalence, effect sizes, associations        |
| System integration, load testing, demo data                    | Pretraining a representation of _patients_ rather than of _records_ |

Much of what a health-tech team builds early sits in the left column, and the first row is
knowledge you would otherwise extract from PDFs by hand. The risk is crossing the table
without noticing, and the prevalence check will not catch it. The two columns invert each
other.

<div class="callout" markdown="1">
In real EHR data, a model that learns that patients with a PHQ-9 on file are more often
depressed has learned that clinicians order the test when they suspect depression. Removing
that leakage is much of the honest work. In Synthea the ordering pattern is not leakage; it
is the only signal there is, and nothing remains without it.
</div>

## What to take away

- Ask where a synthetic dataset's dependencies come from before asking whether its numbers
  look right. Matching marginals tell you nothing about the joint structure.
- For a hand-authored generator, there is a cheap check: find the variable you care about and
  grep the rule set for anything that reads it. If nothing does, it is a leaf, and no quantity
  of it will teach a model what that variable means.
- Synthea's protocol library is the part worth having. Read the modules, not just the patients
  they emit.

<br><br>

---

<small>
**A note on this piece**
<br>
The notebook, the exact commands, the seed, and every number above are in <a href="https://github.com/lecaibio/clinical-data-field-notes">clinical-data-field-notes</a>. NHANES is a public download; the Synthea population regenerates from a pinned tag and a fixed seed.
<br>
Proportions are unweighted. NHANES uses a complex sampling design and I ignored its weights, which shifts the levels slightly and none of the comparisons. The claim about pretraining transfer is reasoned from the generator's mechanism, not measured; I ran no training experiment.
<br>
AI helped me organize the writing; the analysis, the readings of the code, and the judgments are mine.
<br>
The views here are my own and not affiliated with any employer or organization.
</small>
