---
layout: page
title: Anomaly Detection on Dynamic Graphs
description: TGN-SVDD and RTGN-SVDD, unsupervised and noise-robust intrusion detection combining temporal graph networks with one-class classification.
importance: 2
category: work
related_publications: true
---

TGN-SVDD is the first end-to-end unsupervised intrusion detection model combining temporal graph network (TGN) embeddings with deep one-class classification (Deep SVDD). It shows that vanilla TGNs, while strong for link prediction, are insufficient for intrusion detection, where anomaly detection objectives are critical. On CIC-IDS2017, TGN-SVDD reaches ROC-AUC 0.994-0.999, against 0.268-0.690 for vanilla TGN, outperforming classical anomaly detectors (LOF, Isolation Forest) as well (ICANN 2023).

RTGN-SVDD extends TGN-SVDD into a probabilistic framework predicting Gaussian distributions over event embeddings, with dual scoring mechanisms — one to filter noise, another to detect true attacks. Under 50% label contamination, RTGN-SVDD improves ROC-AUC from 0.574 to 0.844 and from 0.561 to 0.880 relative to the deterministic model, demonstrating robustness where deterministic baselines fail (ESANN 2024).

Papers: <a href="https://arxiv.org/abs/2508.12885">arXiv:2508.12885</a> (TGN-SVDD), <a href="https://arxiv.org/abs/2508.14192">arXiv:2508.14192</a> (RTGN-SVDD). Code: <a href="https://github.com/AlekseiLiu/tgn_svdd">github.com/AlekseiLiu/tgn_svdd</a>, <a href="https://github.com/AlekseiLiu/rtgn_svdd">github.com/AlekseiLiu/rtgn_svdd</a>.
