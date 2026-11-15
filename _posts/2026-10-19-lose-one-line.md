---
layout: distill
title: "Lose one line: where the linear shortcut fails"
description: "N-1 security, AC vs DC power flow, and why reactive power hides overloads, on a small standard test grid."
tags: [power-systems, machine-learning, gnn]
categories: [grid-n1]
giscus_comments: false
related_posts: false
published: true
thumbnail: assets/img/posts/grid-n1/thumb_post1.png
mermaid:
  enabled: false
authors:
  - name: Aleksei Liuliakov
    url: "https://alekseiliu.github.io"
toc:
  - name: "N-1: lose one line, stay within limits"
  - name: A small grid with one weak corridor
  - name: Three kinds of power in one wire
  - name: The linear shortcut reads low
  - name: Scanning all 41 outages
  - name: How fast security tips over
  - name: What comes next
---

**TL;DR.** A routine task for transmission system operators (TSOs) is to check that losing any one line overloads nothing else. The fast shortcut for this check, DC power flow, comes at a price: it ignores reactive power, so it can underestimate a line's real loading and miss an overload. On IEEE 30-bus, a small standard test grid, at nominal load and before any outage, DC puts the one overloaded line at 77 %; exact AC power flow puts it at 112 %.

Electricity does not take the route you choose. When a line in a meshed grid trips, its flow jumps onto every parallel path at once, guided by physics. Checking what happens after a failure takes a physics calculation.

My background is in graph machine learning, and I want to know how it can help power grids, which keep growing in importance and complexity. This post is the 101 I worked through first: the N-1 check on IEEE 30-bus, a small standard test grid. The notebook and code are in the [companion repo](https://github.com/AlekseiLiu/grid-n1).

## N-1: lose one line, stay within limits

N-1 is the grid minus any one element, for example a line. The N-1 criterion says the grid must stay within its limits after every such outage: no line above its rating, no voltage outside its band. Operators check this in advance, and if an outage would overload another line, they act before it happens. A common remedy is shifting generation (redispatch).

{% include figure.liquid path="assets/img/posts/grid-n1/d1_n1_concept.svg" class="img-fluid rounded z-depth-1" %}

<div class="caption">
  A toy grid. Dots are buses, the points where generators, loads and lines connect; ~ marks a generator. Every line is rated 65 MW, and arrow
  width is MW. When the top line trips, the middle line goes from 46 % to 104 % of its rating.
</div>

After a change in topology, the check looks for the grid's new steady state: the operating point that satisfies the power flow equations, which balance the power injected and consumed at every bus. Those equations are nonlinear in the bus voltages and angles.

## A small grid with one weak corridor

IEEE 30-bus serves 189 MW, and its generators could produce 335 MW. Supply is not the problem; line capacity in the right place is.

{% include figure.liquid path="assets/img/posts/grid-n1/p1_grid30.png" class="img-fluid rounded z-depth-1" zoomable=true %}

<div class="caption">
  Lines coloured by AC loading at nominal load, light grey (empty) to near-black (full); red is above 100 %. Triangles: generators; square:
  slack bus, which balances the rest.
</div>

Bus 8, near the top, is the largest load: 30 MW plus 30 Mvar of reactive power (more on that below). The original IEEE case had a synchronous condenser there, a machine that supplies only reactive power; this version of the data dropped it. So bus 8's reactive demand comes in mostly over line 6-8, which is at 112 % before anything fails. A what-if in the notebook confirms the cause: a 30 Mvar capacitor at bus 8, which supplies most of its reactive demand on site, brings the line down to 79 %, while the line's active power barely changes (24.8 to 24.9 MW). What overloads the line is not the MW. It is the reactive power.

## Three kinds of power in one wire

A line is limited by heat, and heat comes from current. Active power P does work. Reactive power Q moves back and forth every cycle to build the magnetic fields of motors and transformers. It delivers no net energy, but it is real current and heats the wire just the same.

The two parts of the current are 90° apart in phase, so they add like perpendicular vectors. The apparent power S, which sets the total current, is the hypotenuse: S² = P² + Q². What actually limits a line is heat, so its real limit is a current, in amperes. Grid engineers prefer to work in power units, and at a fixed voltage current is proportional to S. So line ratings are given in MVA: a current limit expressed in power units, with P and Q both counting toward it.

{% include figure.liquid path="assets/img/posts/grid-n1/d2_pqs.svg" class="img-fluid rounded z-depth-1" %}

<div class="caption">
  Line 6-8 at nominal load, to scale. Grey: P, the only part DC power flow keeps. Orange: Q. Black: S. Dashed red arc: the 32 MVA rating.
</div>

Line 6-8 carries 24.8 MW and 24.4 Mvar. The MW alone fit inside the 32 MVA rating; together they make about 35 MVA, outside it.<d-footnote>pandapower reports loading as current over rated current. Bus 8 sits at 0.961 p.u. (per unit: voltage as a fraction of its nominal value), so the same MVA takes a little more current, and the AC loading of 112 % is a bit higher than 35 ÷ 32.</d-footnote> Heat grows with the square of the current, so the reactive part causes 49 % of this line's heat loss.

> This part took me a while to figure out. I first read MW, Mvar and MVA as three quantities that should not be added. They are one unit,
> volts times amperes, under three labels. P and Q come from the in-phase and the 90° parts of the same current, which is why they add like
> the sides of a right triangle.

## The linear shortcut reads low

Exact AC power flow solves the nonlinear power flow equations for the unknown bus voltages and angles with an iterative numerical method, usually Newton-Raphson, repeated until it converges. DC power flow makes four simplifying assumptions (flat voltages, small angles, no resistance, no reactive power) and turns them into a system of linear equations, solved cheaply in a single step. That is why it is the fast screening tool in practice: operators precompute sensitivity tables on it and screen thousands of outages at once.

DC gets the MW nearly right: on this grid it is off by 0.39 MW per line on average. With voltage fixed and Q gone, MVA and MW become the same number, so DC divides its MW by the MVA rating and calls that loading. The real limit is on current, and current includes Q.

{% include figure.liquid path="assets/img/posts/grid-n1/p1_ac_vs_dc.png" class="img-fluid rounded z-depth-1" zoomable=true %}

<div class="caption">
  The 12 of 41 lines where DC is furthest from AC, at nominal load. Black: AC; grey: DC; dashed red: the limit. Orange bars: reactive share
  Q/S.
</div>

On line 6-8, AC says 112 % and DC says 77 %: "fine" for the one line that is overloaded. Over all 41 lines DC reads only 6.3 points low on average. Its errors sit on lines with a large reactive share (70 % on 6-8), and those include the two most loaded lines, 6-8 and 21-22 (95 % in AC, 64 % in DC).

## Scanning all 41 outages

This is where the shortcut has to earn its place: one fast pass over every outage. An N-1 scan takes each line out in turn and checks every limit. I ran it at 80 % load, because at full load line 6-8 is over its rating before any outage.

{% include figure.liquid path="assets/img/posts/grid-n1/p1_n1_scan.png" class="img-fluid rounded z-depth-1" zoomable=true %}

<div class="caption">
  One column per outage at 0.8× load. Top: worst AC line loading after the trip (dot) and the DC estimate (grey dash). Bottom: lowest bus
  voltage. Colour: why the outage is critical.
</div>

7 of the 41 outages are critical:

- **Thermal (red), 2.** Losing 6-8 pushes 8-28 to 118 %; losing 8-28 pushes 6-8 to 110 %.
- **Voltage only (violet), 4.** A bus drops below 0.95 p.u. Losing 6-8 does that too, on top of its overload.
- **Islanding (hollow), 1.** Losing 25-26 cuts off bus 26.

Here is the lesson. DC sees both thermal outages, but places the surviving line below its limit both times, so a thermal screen that trusted it would have passed both. Its overall ranking of outages is decent (rank correlation 0.74 with AC). What it gets wrong is which side of 100 % a line lands on, and for a thermal check that is the deciding question. Fast and usually close is not the same as safe.

## How fast security tips over

I repeated the full scan, all 41 outages, at load levels from 0.5× to 1.5× nominal, scaling all loads and generation together. In the figure, each bar is one outage, and its height is the worst loading of any line after that trip. The slider sets the load level.

<div class="l-page">
  <iframe
    src="{{ '/assets/plotly/grid-n1/p1_load_slider.html' | relative_url }}"
    frameborder="0"
    scrolling="no"
    height="540px"
    width="100%"
  ></iframe>
</div>

<div class="caption">
  Interactive. One bar per outage, sorted by severity at 0.8× load; red above 100 %. Grey ticks: the DC estimate for the same outage. Drag the
  slider to change the load.
</div>

Move the slider from 0.85× to 0.9×: thermally critical outages jump from 2 to 19, while the intact grid is still within limits and every grey tick stays below the line. One corridor is already almost fully loaded; once it sits near its limit, almost any redistribution tips it over. DC flags its first overload only at 1.0×, a whole regime late. From 0.95× on, the intact grid itself is over its limit, so every outage counts as critical, trivially.

## What comes next

DC power flow gets the MW nearly right in one linear solve. It fails in one place, on lines with large reactive flow near their limit, which is exactly where an N-1 check needs to be right.

In the next post I move to a 118-bus standard test grid and ask the operator's question directly: how much of the exact AC work can a screen skip without missing overloads? It compares the operator's linear shortcut, a simple physics correction of it, and a graph neural network.

The next post is out: [Test before you trust: what a GNN adds to outage screening, and where it stops]({% post_url 2026-11-15-test-before-you-trust %}).

_Built with pandapower (AC and DC power flow, and the IEEE 30-bus case), marimo notebooks, NumPy, pandas, Matplotlib and Plotly, in a uv-pinned environment with pytest in CI. Thanks to the pandapower team for making grid analysis this accessible. Code written with AI coding agents (Claude Code, Codex) in a test-guided loop; results reviewed and every number checked against its source._

{% include repo_footer.liquid repo="AlekseiLiu/grid-n1" %}
