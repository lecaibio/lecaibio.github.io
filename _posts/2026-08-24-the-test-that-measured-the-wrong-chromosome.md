---
title: "The Test That Measured the Wrong Chromosome"
description: "A cannabis diagnostic lost its biggest vendor six years in, when breeders removed the question it answered. Four years later, genome assemblies showed the chromosome it detected was never the determinant."
date: 2026-08-24
layout: post
tags: [industry, diagnostics]
---

In 2021 my then-boss asked me to look into a cannabis plant-sex qPCR kit. It was a fair thing to
ask. We already sold qPCR kits, hemp was booming, and the biology looked like a homework
problem.

I came back with three reasons not to, all of them about our own lab: the sample type was wrong
for our kits, the published biology had holes in it, and as the only PhD in the company I had no
room to bring up a new crop. That was the end of it. The biggest name in that market shut its
testing service down before the year was out, and not for any reason on my list.

**The short version:** the test worked. Its biggest vendor quit anyway, in 2021, because seed
companies had started selling seed that grows out female almost every time and there was nothing
left to ask. Then in 2026 researchers read both sex chromosomes end to end and found that the
gene deciding whether a plant turns out female sits on the X, which every plant carries. The Y
that every test detects is not the switch.

## Only the unpollinated female flower is worth money, and one male can take the crop

Cannabis plants are male or female on separate plants, like holly. Only the female is worth
growing. What gets sold and smoked is her flower cluster, picked before anything pollinates it.
The trade term is _sinsemilla_, "without seed." Let a male open nearby and the females it
reaches switch from filling resin glands to making seed, and seeded flower sells for a fraction
of the price.

From regular seed about half your plants are male, and you cannot tell by looking for four to
eight weeks. Everything spent on a male in that window goes to a plant you will pull: space,
water, light, and in regulated markets a slot against a legal plant count. Miss one and it
opens, and you lose the value of everything its pollen reaches.

A DNA test answers in seven to ten days, from a leaf punch that does not kill the seedling. That
is a month of inputs saved per male. What a grower is buying is the weeks of uncertainty.

## The XY story has exceptions, and they are not rare

Cannabis carries twenty chromosomes: nine ordinary pairs, plus a sex pair that is XX in females
and XY in males. It is a real XY system, one of the few in plants obvious enough to be spotted
down a microscope in the 1920s. That is the version that makes the test look like a homework
problem: find the Y.

Then the exceptions start. Monoecious cultivars, which carry both flower types on one plant, are
XX. Monoecious plants with XY have also been found. And ordinary XX females throw male flowers
under stress: heat, a light leak, damage, a late harvest. Growers call these nanners, and they
ruin rooms.

The hormone logic is backwards from animal intuition. Ethylene pushes flowers female, so to get
pollen out of a female you block ethylene. Silver thiosulfate does that. Spray it on an XX plant
and you get XX pollen; cross that onto another female and every seed is female. This is how
feminized seed is made, and the industry has done it for decades without knowing why it works.

## My three reasons were about the sample and the biology, not the market

Most of the case for it was right. Hemp had been legal since the 2018 Farm Bill, growers were
losing real money to a problem with a price on it, and there was a Y chromosome to go find. My
then-boss came out of software engineering, and from that seat the work reads as a configuration
change: point the same instrument at a new target, design primers, validate, ship. Three of
those four steps really are the same. The one the list leaves out is getting clean DNA out of
the sample, and that is the one that does not carry over from one organism to another.

My smallest objection was capacity. I was the only PhD in the company, and in 2021 that meant
the COVID assays that were paying our bills, the beer kits, and the regulatory paperwork that
came with both. Bringing up a new crop species takes months of doing that instead of the other
work, and we had nobody to hand the other work to while I did it.

The second was the sample. Our kits ran on saliva and on beer culture, and neither of those is
an easy matrix to work in. I came from a plant molecular lab and knew what plant tissue does to
a polymerase. Cannabis leaf is full of polyphenols, polysaccharides and terpenes, and our format
was a fast direct-lysis workflow that traded cleanup for speed. New primers were the easy part.
Getting a one-step kit to hold up on macerated leaf, across cultivars and growth stages and
whatever the grower's hands had been in, is a sample-prep program, not a product line extension.

The third was that the biology did not close. Nothing I read explained the exceptions above, and
I could not find anyone who claimed to. I did not know what they meant, and it made me cautious
anyway. A presence/absence test is a bad bet on a trait whose mechanism has holes in it, because
the holes are where the test fails, and until somebody understands the mechanism nobody can tell
you how often that will be.

## Phylos put in writing that feminized seed ended its testing business

Phylos Bioscience stopped accepting samples on September 30, 2021, and [its notice is still
up](https://phylos.bio/legal/phylos-testing-services). Among the reasons it gives: "Increased
demand for feminized seeds has resulted in a decrease in demand for sex testing services." It is
not often you get a company saying in its own words that a substitute good took its market.

The substitution never competed on speed, price or accuracy. It removed the occasion. A grower
planting feminized seed sexes nothing, because there is nothing to sort. The silver thiosulfate
step still happens, but it happens upstream at the seed producer, on a few dozen plants, once,
instead of on every plant in every customer's field.

Pollen from outside the fence was another risk the test never covered, and breeders took that
one too. Farmers lose crops to pollen drifting from a neighbor's field or from wild hemp, and no
sex test helps, because the plant that seeds your crop is not yours to test. Triploids answer it
structurally: cross a tetraploid with a diploid, the odd chromosome count wrecks meiosis, and
the plants set very little seed even when pollen lands on them. Cornell and Oregon CBD field
trials under deliberate pollen challenge got far fewer seeds than from diploids, though not
zero.

The market ran from about 2015, when Steep Hill [announced its GenKit sexing
kit](https://www.prnewswire.com/news-releases/steep-hill-announces-release-of-genkit-290554691.html),
and prices never moved across the whole run. Phylos charged $15 a test in 2015. LeafWorks sells
at $10 to $15 a plant today. Eleven years of competition bought nobody any pricing power, which
usually means buyers are comparing on the answer, not the method.

## The 2026 assemblies put the determinant on the X, which every plant carries

In 2026 a group led from HudsonAlpha published properly phased X and Y assemblies for cannabis
and hop ([Carey et al., _Nature Communications_](https://doi.org/10.1038/s41467-026-73233-7)).
The Y turned out to be a wreck. Most of its genes are gone, only three of the survivors are
expressed differently between the sexes, and the authors write that none of them "appear to be
obvious sex-determination genes."

Their candidate is on the X: a gene for an enzyme that makes ethylene, switched on in floral
tissue only in females and monoecious plants, possibly working by dose. They call the
arrangement a "background-Y" system, meaning the Y still does real work in fertility but does
not hold the switch. Three other groups landed on X-linked candidates the same year, all of them
touching ethylene, which is the signal silver thiosulfate has been interrupting commercially
since before anyone knew what it was interrupting.

Nobody has nailed a causal gene yet; the papers all say "candidate." The supported claim is
narrower and enough here: the determinant is not on the Y.

That also explains the nanners. If maleness needed a Y, a plant without one making pollen is
unaccounted for. If the X-linked gene drives the female program, male is what you get when that
program fails. Stress destabilizes ethylene signaling, part of the plant reverts, pollen sacs
appear on a female. Silver thiosulfate does the same thing on purpose.

## The plants that ruin crops were never in the denominator of the 99%

Every vendor claims about 99% accuracy, all self-reported, and I could not find an independent
comparison of the kits anywhere. But the number is the smaller problem. It measures Y detection,
not what the plant will do.

A hermaphrodite is a genetic female. Medicinal Genomics says so plainly on its own product page.
So the test calls it female, which is correct by the assay's own standard, and the case never
enters the error rate. A plant the test got right can still be the plant that seeds the room.
Growers complained about this for a decade and it was filed as a limitation of the assay.

The test that would actually help is one that predicts which females are prone to reverting, and
nobody sells it. Feminized seed did not remove that risk the way it removed the sex question.
Every plant in a feminized crop is XX, and XX is what reverts under stress. Reversion propensity
looks heritable and is the trait breeders currently select against by hand, so it is a plausible
target, and now that the pathway has a name it is a more tractable one.

## A diagnostic survives where the customer cannot breed the problem away

<mark>A diagnostic is worth the cost of the uncertainty it removes, so anyone who removes that
uncertainty upstream takes the value to zero no matter how good the assay is.</mark> What matters is
who controls the risk. Cannabis plant sex sat entirely inside the seed supplier's control, and
once it was worth their while to fix it there, the test had nothing left to price.

The contrast is sitting in the same catalogs. Hop latent viroid can cost half the cannabinoid
content of an infected plant, it spreads on cuttings and shared tools, and no breeding program
removes it, because it arrives from outside. More than twenty US labs now sell HpLVd testing.
LeafWorks and Farmer Freeman, who both sold sex tests through the whole run above, now sell
viroid tests alongside them. One line held and the other did not, for reasons that had nothing
to do with how well either assay worked.

All three of my 2021 reasons were about whether we could build the thing. The question I would
ask first now is whether anyone will still need it: can the customer's supplier make this
problem stop existing? For cannabis sex the answer was yes, and the biggest vendor was gone six
years after the first kit shipped.

## What to take away

- **Ask who else could make the problem go away.** If the customer's supplier can remove it
  upstream, the market lasts until doing so is worth their while. Cannabis seed companies could,
  and did. Nobody can breed hop latent viroid out of a plant, which is why more than twenty labs
  sell that test today.
- **A test is scored against what it detects, not against the decision the customer is making.** The
  sex tests were about 99% right on the Y and silent on whether a plant would throw pollen, and no
  number on any product page showed that gap.
- **Changing organisms means changing sample preparation, and that is what sets the schedule.**
  Primers take a week. Getting clean DNA out of a new tissue, reliably, in someone else's hands,
  is the project.

<br><br>

---

<small>
**A note on this piece**
<br>
The 2021 conversation is reconstructed from memory, checked against dated public sources but not against any document of my own. I am not claiming I anticipated the 2026 result.
<br>
I could not find an independent head-to-head validation of the commercial sex tests, or any commercial reversion-propensity assay. Both are my failure to find one rather than proof none exists, and I would like the citation if you have it.
<br>
AI helped me organize the writing and close out the source list; the argument and the judgments are mine.
<br>
The views here are my own and not affiliated with any employer or organization.
</small>
