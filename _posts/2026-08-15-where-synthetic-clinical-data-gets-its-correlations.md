---
title: "Where Synthetic Clinical Data Gets Its Correlations"
date: 2026-08-15
layout: post
---

# Where Synthetic Clinical Data Gets Its Correlations

Synthetic patient data is easy to get and easy to check badly. This is what happens when you
run the same depression questionnaire through a real national survey and a synthetic
generator, and then go read the code that produced the synthetic half.

**The short version:** Synthea's variables are correlated through care protocol, not through
biology. That makes it good data for learning how care is delivered and poor data for
learning what a measurement means. The usual sanity check does not distinguish the two.

## The two datasets

**[NHANES](https://www.cdc.gov/nchs/nhanes/index.html)** is a real, population-level survey:
the CDC's National Health and Nutrition Examination Survey, which samples the US civilian
population, interviews people at home, and examines most of them in a mobile clinic. Public
microdata comes out in two-year cycles, with sampling weights, because the sample is
clustered and deliberately oversamples some groups. It is a standard source for US
prevalence estimates.

**[Synthea](https://github.com/synthetichealth/synthea)** is synthetic: an open-source
patient generator from MITRE that simulates a lifetime of encounters per patient from birth.
Its clinical behaviour lives in modules, one JSON state machine per condition or care
pathway, deciding what happens to a patient and when. No record belongs to a real person, so
there are no privacy restrictions, which is why it is common in health IT testing, demos, and
teaching.

One measures people. The other simulates records. Both get used to answer questions about
patients.

## One instrument, two sources

The PHQ-9 is a nine-item depression questionnaire scored 0 to 27. A score of 10 or above
indicates moderate or worse depressive symptoms and is the conventional point at which a
clinician would follow up, so it is the threshold most analyses use.

For each source, three numbers: what share of adults were given the questionnaire, what share
of those scored 10 or higher, and the product of the two.

| | NHANES 2017–2018 | Synthea v4.0.0 |
|---|---|---|
| given the questionnaire | 91.6% | 11.7% |
| scored 10+, of those given it | 9.1% | 63.8% |
| scored 10+, of all adults | **8.3%** | **7.4%** |

![Three rates, NHANES against Synthea](/assets/images/clinical_field_notes/01-three-rates.png)

The bottom row agrees to within a point. Neither row above it agrees at all: screening is off
by a factor of eight in one direction, the positive rate by a factor of seven in the other,
and the two errors cancel.

Validating a synthetic dataset, the one thing you would check is prevalence. That is the
number that matches.

## The distribution

The same data one level down: the full distribution of scores among people who were screened.

![PHQ-9 score distribution among those screened](/assets/images/clinical_field_notes/02-score-distribution.png)

NHANES looks like a screening instrument pointed at a general population. A third of
respondents score zero, the mass falls off steeply, the tail past 10 is thin. Synthea is a
flat slab, every score from 5 to 27 about equally common. No clinical measurement is
distributed this way.

## Reading the module

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

## The same module, read as a protocol

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
system does about the science. The module library is Synthea's product; the synthetic
patients are its output, and most users take the output without opening the modules.

Both halves of the generator follow from this. The marginals are anchored to census and
public health data, so any single variable is plausible. The dependencies come entirely from
the modules, so what is correlated with what is correlated through the care pathway. Nowhere
in between is there a latent biological state producing a measurement and also driving an
outcome.

## What the score doesn't reach

In a two-year window, 516 adults scored 10 or higher, some at the top of the scale. In the
twelve months after those screens: zero depression diagnoses, zero antidepressant
prescriptions.

![What follows a positive screen in Synthea](/assets/images/clinical_field_notes/03-what-follows.png)

The reason is a one-line search. Grep the module tree for `phqa_score` and every hit is inside
the file that writes it. Nothing in Synthea reads the score.

Antidepressants do exist in the output. Their recorded reasons are post-traumatic stress
disorder, major depression in the veteran module, and fibromyalgia, each ordered by a module
that reached its own conclusion. The screening pathway is encoded and stops where it would
hand off to diagnosis and treatment, which are separate modules with their own entry
conditions.

## What that predicts about transfer

In real records the chain runs biology → symptoms → measured value → outcome. A high PHQ-9 is
informative about what happens next because it is a noisy readout of a latent state, and that
same state drives the outcome. The measurement is predictive because it sits downstream of
something real.

In Synthea the chain stops. The score is conditionally independent of everything after it,
because nothing after it reads it. A model that learns `P(outcome | score)` here learns,
correctly for this data, that the score carries no information: the wrong answer for real
patients, on exactly the variables you care about, and a confident one, since process
correlations in synthetic data are cleaner than clinical correlations ever are.

What is learnable here transfers fine. Which orders follow which findings, what a wellness
encounter contains, which codes travel together: those are what the modules encode.

## Where it holds and where it doesn't

| Holds up | Doesn't hold up |
|---|---|
| Learning the conditional logic of ordering: what triggers what | Predicting a biological outcome given a measurement |
| Utilisation and volume forecasting | Treating a lab or score as a proxy for patient state |
| Coding, billing, claims logic | Learning which measurements matter, or their thresholds |
| Workflow and care-pathway simulation | Risk models meant to transfer to real patients |
| Format and terminology conformance, FHIR pipeline testing | Estimating real-world prevalence, effect sizes, associations |
| System integration, load testing, demo data | Pretraining a representation of *patients* rather than of *records* |

Much of what a health-tech team builds early sits in the left column, and the first row is
knowledge you would otherwise extract from PDFs by hand. The risk is crossing the table
without noticing, and the prevalence check will not catch it.

The two columns invert each other. In real EHR data, a model that learns that a patient with
a PHQ-9 on file is more likely to be depressed has learned that clinicians order tests when
suspicious, and removing that leakage is much of the honest work. In Synthea the ordering
pattern is not leakage but the signal, and nothing remains without it.

## The general question

Evaluating a synthetic dataset by its outputs tells you less than asking where its structure
came from. Three common answers:

1. **Fitted to real data.** Models trained on real records. The dependencies are whatever the
   fitting captured, usually strong pairwise structure, weaker higher-order structure, and
   privacy properties worth checking.
2. **Hand-authored.** Rules written by people, as in Synthea. The dependencies are the ones
   the authors wrote down, which in clinical software means protocol. Anything nobody encoded
   is absent, and its absence does not show up in summary statistics.
3. **None.** Independent marginals, each plausible alone, joint distribution meaningless.

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
