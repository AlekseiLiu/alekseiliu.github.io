---
layout: page
title: "grid-n1: thermal N-1 screening"
description: N-1 contingency analysis on standard IEEE test grids, and how much exact power-flow work a screen can skip.
img: assets/img/posts/grid-n1/thumb_post1.png
importance: 1
category: work
---

grid-n1 is a study of N-1 contingency analysis on standard IEEE test grids: for every single line or transformer that could trip, will another branch overload? Stage 0 shows why the fast linear (DC) shortcut hides overloads near the limit. Stage 1 measures how much exact AC power-flow work a screen can skip at a calibrated recall target.

The first post explains the problem: [Lose one line: where the linear shortcut fails]({% post_url 2026-10-19-lose-one-line %}). The code is at [github.com/AlekseiLiu/grid-n1](https://github.com/AlekseiLiu/grid-n1).
