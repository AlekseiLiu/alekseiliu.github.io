---
layout: page
title: "DGPO: RL-Steered Graph Diffusion for Neural Architecture Generation"
description: RL-steered graph diffusion for neural architecture search; matches NAS-Bench-201 optima on all three tasks (by benchmark lookup).
img: assets/img/dgpo_poster_header.png
importance: 1
category: work
related_publications: true
---

DGPO extends RL-steered discrete graph diffusion from undirected molecular graphs to directed acyclic graphs, using topological node ordering and positional encoding to represent neural architectures. A reward-driven policy steers the generative distribution toward high-performing structures, enabling controllable architecture generation.

{% include figure.liquid loading="eager" path="assets/img/dgpo_poster_priors_panel.png" class="img-fluid rounded z-depth-1" %}
<div class="caption">IJCNN 2026 poster panel: transferable structural priors (pretrained on 7% of NAS-Bench-101).</div>

On NAS-Bench-201, DGPO matches the benchmark optima on all three tasks by benchmark lookup: CIFAR-10 (91.61%), CIFAR-100 (73.49%), and ImageNet-16-120 (46.77%), with fixed cell connectivity so generation chooses operations. On NAS-Bench-101 it achieves 94.50% (benchmark lookup). Pretrained on only 7% of NAS-Bench-101 topology data, the best generated architecture over evaluation draws lands within 0.32 pp of the full-data model and 7.27 pp above the pretraining ceiling, demonstrating transferable structural priors (3 seeds; ~115 single-GPU hours on one A40). A multi-objective extension (MO-DGPO) reaches Pareto hypervolume 0.9901 on NAS-Bench-201, matching the union of three single-task runs.

A bidirectional-control ablation, inverting the reward, drives accuracy to 9.5% (chance level), validating directional control of the generator.

Accepted at IJCNN 2026 (WCCI). Paper: <a href="https://arxiv.org/abs/2602.19261">arXiv:2602.19261</a>. Code: <a href="https://github.com/AlekseiLiu/DGPO">github.com/AlekseiLiu/DGPO</a> (MIT, Makefile one-command reproduction, pinned requirements).

Full poster: [PNG](/assets/img/dgpo_poster_full.png).
