---
layout: distill
title: "Test before you trust: what a GNN adds to outage screening, and where it stops"
description: "On a standard test grid, plain physics with one simple correction closes most of the gap to the exact calculation and a GNN adds a little more, until conditions leave familiar ground."
date: 2026-11-15
tags: [power-systems, machine-learning, gnn]
categories: [grid-n1]
giscus_comments: false
related_posts: false
published: true
thumbnail: assets/img/posts/grid-n1/thumb_post2.png
mermaid:
  enabled: false
authors:
  - name: Aleksei Liuliakov
    url: "https://alekseiliu.github.io"
toc:
  - name: Why this matters
  - name: The question, and what I expected
  - name: The setup in one picture
  - name: What worked
  - name: What did not work, and why
  - name: The take-away
  - name: Limits and details
---

**TL;DR.** On a standard test grid I asked how many exact outage calculations a screen can safely skip. The plain physics shortcut with one correction skips 73.7 % of them. A graph neural network adds about 5-7 points on top. Off familiar ground the learned screen misses more than its target allows. Fix the baseline first, then test off familiar ground.

## Why this matters

A grid operator needs to know what would happen if one single line trips ([background]({% post_url 2026-10-19-lose-one-line %})). The [rule, called N-1](https://www.legislation.gov.uk/eur/2017/1485/article/3/adopted), is that the grid must then stay within its limits. So you take a solved state of the grid, remove the lines one by one, and each time recompute it and look for overloads.

The exact check is one full grid calculation per outage. This test grid has 186 branches (lines and transformers), so that many exact calculations for one operating state. A screen sorts the outages first. Clearly safe ones are skipped; the rest get the exact calculation.

Below, one line fails. The exact calculation puts another well above its limit; the plain shortcut, a simple estimate, reads it below.

{% include figure.liquid path="assets/img/posts/grid-n1/p2_case.png" class="img-fluid rounded z-depth-1" zoomable=true %}

<div class="caption">
  A selected example, not a typical one. Line 65-68 fails; the exact calculation puts line 49-69 at 127 %, the plain shortcut at
  97 % (before its safety margin). Blue: the GNN.
</div>

The question is how good that sorting can be.

## The question, and what I expected

I asked how many exact runs a screen can skip while it still catches 99 of 100 overloaded cases: the target. A case is one operating state with one outage; it is overloaded if the exact calculation puts any remaining branch above its rating. I also asked how much of that is learning and how much plain physics.

I come from machine learning, not from power engineering. I learned the grid side for this project, out of interest in grids that carry more renewable power. A grid is a graph, so I expected a graph neural network (GNN) to be expressive enough to capture the non-linear physics the plain shortcut misses. I also expected it to struggle on patterns it never saw in training, where deep learning often breaks. So the plan included tests off familiar ground, meaning load levels and outages unlike those in training.

## The setup in one picture

On a [standard test grid](https://github.com/power-grid-lib/pglib-opf) I made synthetic operating states (random load levels, cost-optimised generation) and solved every outage exactly, as the reference.

The plain shortcut is a simplified linear calculation. It uses [fixed factors](https://www.powerworld.com/WebHelp/Content/MainDocumentation_HTML/Line_Outage_Distribution_Factors_LODFs.htm) that say how a failed branch's flow moves onto the other branches. The corrected shortcut adds one simple correction. A plain neural network sees the grid as a flat list of numbers; the GNN sees nodes joined by branches and learns a correction to the plain shortcut.

{% include figure.liquid path="assets/img/posts/grid-n1/d4_pipeline.svg" class="img-fluid rounded z-depth-1" %}

<div class="caption">
  One case is one operating state with one outage. The screen sends a case to the exact calculation when its worst estimate plus the
  safety margin crosses 100 %.
</div>

Each screen gets one safety margin, an offset added to every estimate, set on held-back states so that the target is met there. No held-back or test state was used for training. The target is not a guarantee: the margin is calibrated for it, and only for cases like the held-back states. Then I count the exact runs skipped: how sharp a screen is, not time saved.

## What worked

A perfect screen, which sends on only the real overloads and the outages that split the grid, could skip 79.7 % of the exact runs. The plain shortcut skips only 28.6 %. It is oversimplified: it counts only part of what loads a branch, so it can read too low and needs a wide margin.

That natural gap mostly goes away with one pre-computed ratio. For each branch I divide its exact loading by the shortcut's loading in the intact grid, limit that ratio to between half and double, and apply it after the outage. No learning. With this one correction the same shortcut skips 73.7 %. I built it after the main results, to check that the baseline was fair. It is my own simple version of a known idea.

The correction assumes the ratio stays the same after the outage. In reality flows redistribute in a more complicated way, which is left to learn. The GNN skips 79.4 %, almost the maximum, with the target met on average over three training runs. Over the corrected shortcut, learning adds about 5-7 points, depending on that limit. Honestly, that is not a big difference.

{% include figure.liquid path="assets/img/posts/grid-n1/p2_ladder.png" class="img-fluid rounded z-depth-1" zoomable=true %}

<div class="caption">
  Exact runs each screen lets you skip on familiar ground, margin set to catch 99 of 100 overloaded cases. Whiskers: three limits
  for the ratio (corrected shortcut), training runs (networks).
</div>

## What did not work, and why

Then I left familiar ground: states with higher load than any in training, and unseen outages (branches that never failed in training or calibration, at both load levels).

On unseen outages the GNN caught 95-98 of 100 overloaded cases. At higher load it caught 96 to just under 99, recomputed afterwards without rounding: both short of the target. The plain neural network caught only about 70 on unseen outages. The plain shortcut kept its target, but then skipped very little.

{% include figure.liquid path="assets/img/posts/grid-n1/p2_offground.png" class="img-fluid rounded z-depth-1" zoomable=true %}

<div class="caption">
  Overloaded cases caught per 100, three training runs of the GNN, margin set on familiar ground. The unseen-outage set includes
  normal and higher load; higher-load values recomputed afterwards.
</div>

One planned test failed too. Networks trained separately should disagree where they are unsure. I had seven screening rules, and most used that disagreement as a warning sign. The rules and the pass mark were written down before the first run. None passed.

The margin can only hold for cases like the ones it was set on. The rest was checked afterwards. The networks did notice that the grid had changed. But where they were wrong, they tended to be wrong together: three copies of one network were off in the same direction. So their disagreement did not point at the misses. The GNN's missed overloads also got bigger: none more than one point above the limit on familiar ground, some far above it off it.

## The take-away

In short: learned approximations can be useful, but be aware of where they apply.

- **Physics first.** Fix the baseline before you credit the model.
- **Test before you trust.** A calibrated margin is not a guarantee. Off familiar ground a learned screen needs a new test, or a physics fallback.
- **For an operator,** a learned screen is a first filter, for ranking outages and for planning studies with many scenarios. It does not replace the security check.

## Limits and details

- One standard test grid. Its line limits and my operating states are synthetic, and the states are not N-1 secure: overloads are far more common than in real operation. I checked overloads only, not voltage or stability.
- The target is nominal. Cases from one state resemble each other, and even on familiar ground a single training run can land just below it.
- No speed claim: on a grid this small a fast solver needs a fraction of a second for nearly all outages of a state.

For specialists: the grid is the IEEE 118-bus case, and exact means AC power flow. The plain shortcut is DC power flow with line outage factors; it leaves out reactive power. The margin is set by [split conformal](https://arxiv.org/abs/2107.07511) calibration. The corrected shortcut follows the [hot-start](https://arxiv.org/html/2509.19976v2) idea of starting from a solved state; I did not compare the established versions. Related work: learned models for grid outages ([one](https://arxiv.org/abs/1908.08314), [two](https://arxiv.org/abs/2310.04213)), a [calibrated screen for multiple outages](https://arxiv.org/abs/2602.07995) and a [screen tested under changing conditions](https://arxiv.org/abs/2607.13221). Nothing here is a new method. Full tables are in [RESULTS.md](https://github.com/AlekseiLiu/grid-n1/blob/main/RESULTS.md).

_Built with pandapower, lightsim2grid and PyTorch Geometric, and with AI coding agents under tests. I chose the question, directed the work and reviewed the results; the headline numbers were recomputed independently with separate code._

{% include repo_footer.liquid repo="AlekseiLiu/grid-n1" %}
