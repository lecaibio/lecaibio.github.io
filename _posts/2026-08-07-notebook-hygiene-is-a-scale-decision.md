---
title: "Notebook Hygiene Is a Scale Decision"
description: "Stripping notebook outputs is an exposure control, not tidiness. Only the layer on infrastructure you control is enforcement, and most projects need none of it."
date: 2026-08-07
layout: post
tags: [infrastructure, reproducibility]
---

I work on a small team with no platform engineer. When we needed notebook outputs kept out of
git history, I built the whole thing myself: a filter that strips them on commit, a check I
run locally, and a required check on the server that nobody can skip.

This is the reasoning that produced it, one problem at a time. The last section argues that
most projects should have none of it. That section is the point of the post, not a caveat at
the end of one.

## Notebook diffs are unreadable, and a filter fixes that

A notebook is a single JSON document holding code, results, and execution metadata together.
Running it rewrites parts that have nothing to do with what you changed: execution counters
increment, images are re-embedded as base64, environment metadata shifts when the file is
opened on a different machine.

So the diff of a one-line edit is unreadable, and a merge conflict lands inside a structure
where hand-editing risks corrupting the file. This is the complaint most people arrive with,
and the usual fix is a filter that strips outputs on the way into the repository while
leaving the file on your disk alone.

## Outputs are data, and git history is permanent

A traceback prints whatever was in scope, including connection strings. A `.head()` prints
rows. A plot embeds the values it was drawn from. None of this is a tidiness problem. It is
patient rows, or credentials, or a licensed extract, sitting in a file you are about to push.

Git history is the wrong place to learn this. A later commit does not remove earlier content,
and many managed repositories deny force-push, so the honest description after the fact is
that the blob is unreachable rather than gone. The question "can this repository be shared"
gets a permanent answer, decided by an accident.

Once you see the first problem as an instance of the second, the fix stops being optional and
starts being something that has to hold whether or not anyone remembers it.

## Only the layer you control is enforcement

A git filter only runs if it is installed, and the installation is per clone. The file
declaring which paths get filtered is version controlled; the config entry naming the command
that does the filtering is not, because it lives in the clone rather than in the repository.
So the protection is weakest for a person who is new to the repository, which is the same
person most likely to commit something they did not mean to.

Worth being explicit about who this protects against. The threat is an accidental commit by
someone doing their job, not a determined person with admin rights working around the check.
Nothing here stops the second, and building for it would cost more than it is worth.

The answer is layers, and the useful way to think about them is not as redundancy.

<mark>Only the layer running on infrastructure you control is enforcement. Everything else
reduces the cost of being caught.</mark> A local filter catches a mistake in under a second. A
local script catches it before you spend a round trip on a rejected pull request. A check on
the server catches it always, because it runs in a fresh environment with none of your
configuration in it.

![The three layers, what each one catches, and where the enforcement boundary sits](/assets/images/notebook_hygiene/01-three-layers.svg)

Two things make the server layer real rather than decorative. The check has to be required,
so a pull request cannot merge past a failure. And the merge has to land only what was
checked: if the check evaluates the merge result while the repository preserves each commit
on the branch, a dirty intermediate commit under a clean tip rides in behind a passing check.
Squashing closes that gap, because the commit that lands is the one that was evaluated.
Keeping the branch's own commits would carry the dirty one into the main line's history,
which is the outcome the whole arrangement exists to prevent.

The cost worth naming up front is that the local filter and the server check are the same
tool installed twice, configured independently. They will drift, neither copy knows the other
exists, and keeping them in agreement is ongoing work rather than a one-time setup.

## Checks belong on the repository's side of the filter

The filter strips on the way into the repository and does nothing on the way out. Your local
notebook keeps its results. The stored version has none. That asymmetry is the design, and it
is what makes the arrangement tolerable to work with.

It also means the file on your disk is not the object under version control, and any tool you
point at your files is answering a question about your disk.

That has a specific consequence for the local check. The natural implementation runs the
stripper over the notebooks on disk and reports whether anything changed. Do that and the
check either reports on the wrong thing or destroys the outputs in the notebook you currently
have open. It has to read what git already holds instead, which means committing before
checking. Slightly annoying, and correct.

The general form: anywhere a transform sits between your working directory and the
repository, checks belong on the repository's side of it.

<div class="callout" markdown="1">
Worth stating because the cost of getting it wrong is asymmetric for scientific work. An
output cell can represent a long query against a restricted database, or a run whose seed was
never fixed. A checker that quietly clears it is not a minor inconvenience.
</div>

## Most projects should build none of this

Every layer above taxes every commit. On a project that does not need it, the tax buys
nothing, and it makes exploration tedious enough that people start working outside the
repository, which is worse than having no system.

The conditions I would want to see before building this:

- **More than one person clones the repository.** This is the sharpest test, because the
  entire argument for enforcement is about the configuration a second person does not have.
  On a solo project the local filter alone is the whole system.
- **Results are going into a report or a decision**, rather than being looked at once.
- **The data has restrictions or real acquisition cost.**
- **Someone will need to reproduce the analysis later**, which means the repository is the
  record rather than a backup.

Short of that, save the notebook and move on. Exploratory work has a different job and pays a
real price for ceremony.

I built the full version. I would guess a fair number of projects that have something like it
should not, and I am not yet certain mine clears the bar I just wrote down.

<br><br>

---

<small>
**A note on this piece**
<br>
This describes an arrangement I set up recently and have not run for long. The reasoning is what I would defend; the specific configuration is not yet battle-tested, and I would not present it as a recommendation to copy.
<br>
Nothing here depends on which CI service you use, which repository host you are on, or which stripping tool you pick.
<br>
AI helped me organize the writing; the judgments are mine.
<br>
The views here are my own and not affiliated with any employer or organization.
</small>
