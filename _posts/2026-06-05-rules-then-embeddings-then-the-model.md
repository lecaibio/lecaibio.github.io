---
title: "Rules, Then Embeddings, Then the Model"
description: "Quantitative analysis of free text with an LLM, done efficiently and under privacy limits. A million public comments, a weak model chosen by where the data could go, and most of the work finished before it."
date: 2026-06-05
layout: post
tags: [llm, reproducibility]
---

Imagine more than a million free-text comments. No structure, no length guarantee, no consistent
language: a single emoji, three languages in one sentence, the same line pasted by a thousand
accounts, replies to replies. Fewer than one in five hundred says anything useful about your
product, and the ones that do usually never name it, because **nobody types out a full product
name when they are posting a comment**.

Keyword search finds almost nothing. Reading a million comments is not an option. Surely there
is a magic word for this by now. LLM.

Well, you can hand all million to one and pay for every one of them. And what if privacy rules keep
the text inside your own account, the model you are allowed to hand it to is not the good one...

Or you can do this.

<div class="callout" markdown="1">
The interesting questions come later. Where did they hear about it. What are they comparing it
with. What do they like, what do they hate. None of those answers means anything until this one
is settled, a million times over: **is this comment about my product at all?**
</div>

**The short version:** rules, then embeddings, then the model, in that order because the model
is the layer you can least explain and least control. Each layer exists to reduce how much text
reaches the next one, and the model at the end was a weak one, picked for where the data could
go. Finding the comments is the problem here; what you do with them afterward is a separate one.

![The three layers, what each one removes, and the share of the corpus surviving each stage](/assets/images/rules_embeddings_model/01-funnel.svg)

## A comment does not say what it is about

The reason nobody names the product is structural. Every comment sits under something else, a
post or a video or a thread, and that parent already fixes what the comment is about. "Ordered
three of these, they're amazing." Three of what.

Reading it more carefully recovers nothing. The missing half was never in the text. A chat turn,
a forum reply, a fragment of a clinical note all have the same property, and everything below
works around it.

## Rules should cut only what is beyond question, and that is still most of the corpus

Before anything is embedded, most of the corpus goes to tests where nothing removed is in
question: wrong language, too short to carry a statement, nothing but a URL, nothing but emoji,
the same id twice, the same text under a thousand different ids.

Language detection is where "rules" is the wrong word. I used `langid`, a classifier published
over a decade ago, and it was right often enough that nothing downstream had to think about
language again. A rule here only has to be deterministic, auditable, and cheap enough to run on
everything. Whether a model does the work is beside the point.

Nothing past that class belongs here. A record dropped at this stage is gone from every later
stage and from every count, and the search that produced these comments was rate limited, so
nothing afterward can measure what a filter cost you. Whether a comment's parent is relevant
enough is a judgment, since relevance does not travel cleanly from a parent to the comments
beneath it, and judgments wait for a layer that can read the text.

These tests alone bring the corpus down to about a quarter, which is reason enough to leave
everything past them loose. The step feels unsophisticated next to the rest of the pipeline, and
it is the only one where I can state exactly why each record was dropped.

## A cluster has no meaning until you sample it deliberately

Each surviving comment goes through an embedding model and comes back as a vector of 1024
numbers, which is the width that model returns. Comments written about the same thing land near
each other in that space, and clustering finds the dense regions of it. It does not tell you
what any of them is about. A cluster is a set of vectors with a centroid and no name.

You get the meaning by sampling it, and where you sample from matters. The members nearest the
centroid confirm what the cluster is supposed to be. The ones at the far edge tell
you whether it is one thing at all: some clusters have a coherent center and a boundary of
leftovers, others are two topics sharing vocabulary. The most upvoted members tell you which
part of it anyone actually saw.

Read all three and you know whether the cluster is tight or loose, which is worth knowing before
its label goes anywhere. A tight one can carry a single description. A loose one gets split,
dropped, or recorded as unreliable before anything downstream depends on it. This is the step
people skip, and it decides whether the labels mean anything.

The cluster question comes first: what is this group discussing. Few clusters get dropped on
that answer alone. The filtering happens one level down, where each comment inside a cluster is
judged on its own with that description as context, and the question there is whether this
particular comment carries the signal I want. That is the step where three percent of the corpus
became two tenths of one percent.

Keeping the two questions apart matters, because asked together a wrong answer does not tell you
which failed. Both need a way to answer "none of these," because otherwise a model always chooses
something, and a confident label on a comment that carried no signal is invisible from then on.

## When the text cannot be shown, the summary is the data

The people who needed to see the corpus were not allowed to read it. That is common, and privacy
is usually why: patient records, support tickets, survey responses, **anything where individuals
can be recognized**. So the cluster became the unit of display, one description per cluster
instead of one per comment, each in four fields.

- **Gist.** One line, for scanning.
- **Description.** The longer version, for stopping on.
- **Key phrases.** Up to five, three to five words each.
- **Nearest clusters.** Closest by centroid distance, computed in the original
  1024-dimensional space.

Phrases beat keywords. A keyword names a topic and drops the context that disambiguates it; a
phrase of three to five words keeps it.

The first rule was no names at all, to protect privacy. Then I changed it to allow ambassadors,
creators, and other public figures, which keeps the exposure pathway visible: a summary with no
names cannot tell you who pointed people at the product. That was the line under that agreement,
on a public corpus. It is not a claim that public figures have no privacy, and on different data
I would draw it again from scratch.

## The map showed relatedness twice, and one of them was empty

HDBSCAN does not do well on 1024 dimensions, so something has to reduce them first. I used UMAP
to bring them down to two, because two is what you can plot. Then I built the result into an
interactive map, using the privacy-safe summaries so it could be shown to people who could not
read the comments, and I was proud of it.

It carried two encodings of relatedness on one screen. The edges came from the nearest-cluster
field, each cluster wired to its neighbors in the full 1024-dimensional space, and those were
correct. The positions came from the projection, and those were carrying nothing.

I found this out while presenting it. Cluster after cluster connected to a genuinely related
cluster on the far side of the map, and the room wanted to know why. Nothing on the screen was
false. But put an empty encoding beside a true one and people settle it in favor of position,
because position reads first.

Reducing to ten dimensions instead of two, and clustering there, gave more clusters and, on
sampling, more meaningful ones. It did not fix the map: UMAP preserves local neighborhood
structure, and distance between clusters is not preserved at any output width. So I stopped
making it. The clustering and the neighbor list were fine, so I kept both and put the
relationship in text instead of on a layout.

Then I stopped showing the overall picture at all and went into the few groups that mattered.
About three percent of the corpus ended up inside a cluster, and an overview of the rest would
mostly be noise. The remainder is not junk: rare is what unclustered looks like to a density
algorithm, and rare consumer behavior is often the interesting kind. I never got to it.

## The model was weak because the data could not leave

The model I used was not the best available. It was a basic Nova model on Bedrock, picked
because Bedrock keeps the text inside the account's own agreement instead of handing it to a
third party. These were public comments about a consumer product, and the model was still
chosen by where the data could go.

Once that is the situation, the funnel stops being an optimization. Every deterministic layer
upstream reduces the volume crossing a boundary and narrows what a constrained model is asked to
do. The same holds in clinical text with the constraint tightened: the prompt is the transfer,
and whatever agreement governs the data usually has something to say about third-party
processing. I have watched that version rather than run it, mentoring an intern on extraction
from clinical notes, and I would not claim more.

The layers also differ in how well they explain themselves afterward. A rule can be stated. A
cluster assignment reproduces from fixed embeddings and a fixed seed. A model's label does not
reproduce exactly and does not come with a reason. <mark>The model is the layer you can least
explain and least control, so it should get the least text.</mark>

## What this leaves out

There is no evaluation loop. I meant to build one and did not. Reading cluster edges and reading
raw outputs is what these labels have been held to, which is weaker than a held-out set with an
error rate per class.

Some corpora need none of this. If it is small enough to label completely, label it completely.
If the categories are known and the vocabulary is distinctive, rules may take you the whole way.
Short text is where the middle layer stops earning its place: if you cannot tell two clusters
apart at their boundaries, the geometry has not found anything.

The expensive mistake is reaching for the most capable tool first, because it is the one that
works on any input, and paying for that generality on a million records where a length check
would have removed a third of them.

## What to take away

The model is the layer you can least explain and least control, so it should get the least
text. The rest of the ordering follows from that.

- **Rules first.** They remove only what is beyond question, which is the only removal you can
  still justify afterward. Anything dropped here is gone from every count that follows.
- **Clustering second.** It groups without naming, so one decision covers a whole group.
- **The model last, and smallest.** Under a data use agreement, what reaches it is what crosses
  a boundary, so the ordering is a compliance question.
- **Read the edge before you trust the label.** The center only confirms what you expected.

<br><br>

---

<small>
**A note on this piece**
<br>
The work and the notes behind this are from June 2026. I finished writing it up and
published it in August.
<br>
This is an experience report rather than a reproducible analysis. The corpus was proprietary and
is not published, there is no notebook, and the figure is illustrative. The survival proportions
are real; the exact corpus size is withheld.
<br>
The comparison between clustering settings was qualitative and based on sampled clusters, not a
cluster quality metric. The final labels were never evaluated against a held-out set. The
paragraph about clinical text describes work I mentored rather than work I ran.
<br>
Nothing here depends on which embedding model, clustering implementation, or language model you
use.
<br>
AI helped me organize the writing; the judgments are mine.
<br>
The views here are my own and not affiliated with any employer or organization.
</small>
