---
id: s1-baselines
repo: AFC-Prune
title: Under one fair protocol, does corrected PGI select better filters than simple scores?
workstream: W2
status: running
owner:
- Dewmina Wijekoon
started: '2026-11-24'
summary: We compare twelve ways of choosing which convolutional filters to keep, including our corrected
  Principal Gradient Importance (PGI), at six pruning budgets and several recovery levels, on three separately
  trained models. So far, PGI beats random choice but loses to simple scores such as Taylor importance
  and weight magnitude.
question: When every selector gets the same budget, the same per-layer allocation rule and the same recovery
  step, does corrected PGI keep more validation accuracy on Conv2Net with CIFAR-10 than simple filter
  scores (L1 and L2 norm, Taylor, batch-normalisation scale, FPGM, HRank, mean activation) and random
  choice?
hypothesis: If PGI captures how filters shape the output layer's functional connectivity, it should beat
  the simple scores, most clearly when only the output layer is allowed to recover.
success_criterion: Over three dense models trained with different seeds, corrected PGI's mean validation
  accuracy change is at least as good as the best simple selector at most budgets, judged on paired per-seed
  differences, with the closed-form ridge head (tier T1r) as the headline recovery tier.
criterion: post_hoc
key_results:
- label: Taylor importance, 75.8% of parameters removed, closed-form ridge head (T1r)
  value: -8.35
  unit: accuracy change (points)
  split: val
  n_seeds: 3
  std: 1.4
  compare:
    label: paper (Table 5)
    value: -3.78
- label: Corrected PGI, same setting
  value: -10.65
  unit: accuracy change (points)
  split: val
  n_seeds: 3
  std: 1.02
- label: Random selection (3 draws per seed), same setting
  value: -11.39
  unit: accuracy change (points)
  split: val
  n_seeds: 3
  std: 1.37
- label: 'Reproduction gate: corrected PGI at (19, 16) kept, FC fine-tuning (T1fc), paper protocol'
  value: -10.47
  unit: accuracy change (points)
  split: test
  n_seeds: 1
  compare:
    label: cifar10-repro step 8, 3-seed mean
    value: -10.5
evidence:
- type: diagram
  title: One fair comparison of filter-selection methods
  caption: Each of three separately trained Conv2Net networks is pruned by our corrected Principal Gradient
    Importance (PGI, in blue) and by eleven comparison methods, including filter pruning via geometric
    median (FPGM) and feature-map rank (HRank). Every method gets the same budgets, the same rule for
    splitting the budget between the two layers, and the same recovery levels, and every pruning choice
    is made on validation images. The amber level, gradient fine-tuning, is unfinished because the GPU
    server lost power, so the results so far come from the four recovery levels that need no gradient
    training.
  src: ../figures/s1-baselines/evidence-01-one-fair-comparison-of-filter-selection-methods.svg
  source_url: https://github.com/NeuroPulse-Co/AFC-Prune/blob/main/studies/s1-baselines/diagrams/method.excalidraw.svg
- type: chart
  title: At the paper's pruning level, corrected PGI loses to simple scores
  caption: Each dot is one of three separately trained networks; random filters have three draws per network.
    Only the final layer is refitted after pruning. Taylor importance, batch-norm scale and the smallest
    weights cluster around −8.4 to −8.8, corrected PGI averages −10.65, and random choice −11.39. Nothing
    comes near the paper's −3.78.
  kind: dots
  x: selector
  y: d_acc
  y_label: Change in validation accuracy after removing 75.8% of parameters (points)
  split: val
  x_label: How filters were chosen
  reference:
    label: Paper
    value: -3.78
  labels:
    pgi_aligned: PGI, corrected
    pgi_literal: PGI, authors' formula
    taylor: Taylor importance
    bn_gamma: Batch-norm scale
    l1: Smallest weights (L1)
    l2: Smallest weights (L2)
    fpgm: Geometric median (FPGM)
    hrank: Feature-map rank (HRank)
    act_mean: Mean activation
    random_s0: Random filters
    random_s1: Random filters
    random_s2: Random filters
  rows:
  - selector: pgi_aligned
    d_acc: -10.32
  - selector: pgi_literal
    d_acc: -10.98
  - selector: l1
    d_acc: -8.28
  - selector: l2
    d_acc: -9.08
  - selector: random_s0
    d_acc: -11.32
  - selector: random_s1
    d_acc: -11.58
  - selector: random_s2
    d_acc: -9.88
  - selector: bn_gamma
    d_acc: -8.08
  - selector: fpgm
    d_acc: -9.62
  - selector: taylor
    d_acc: -7.66
  - selector: hrank
    d_acc: -9.0
  - selector: act_mean
    d_acc: -9.32
  - selector: pgi_aligned
    d_acc: -11.8
  - selector: pgi_literal
    d_acc: -15.74
  - selector: l1
    d_acc: -9.6
  - selector: l2
    d_acc: -9.82
  - selector: random_s0
    d_acc: -11.78
  - selector: random_s1
    d_acc: -11.78
  - selector: random_s2
    d_acc: -15.22
  - selector: bn_gamma
    d_acc: -9.5
  - selector: fpgm
    d_acc: -9.94
  - selector: taylor
    d_acc: -9.96
  - selector: hrank
    d_acc: -11.48
  - selector: act_mean
    d_acc: -10.36
  - selector: pgi_aligned
    d_acc: -9.84
  - selector: pgi_literal
    d_acc: -10.9
  - selector: l1
    d_acc: -8.52
  - selector: l2
    d_acc: -10.6
  - selector: random_s0
    d_acc: -10.02
  - selector: random_s1
    d_acc: -11.52
  - selector: random_s2
    d_acc: -9.38
  - selector: bn_gamma
    d_acc: -7.5
  - selector: fpgm
    d_acc: -8.26
  - selector: taylor
    d_acc: -7.42
  - selector: hrank
    d_acc: -9.92
  - selector: act_mean
    d_acc: -9.66
  data: ../data/s1-baselines/evidence-02-at-the-paper-s-pruning-level-corrected-pgi-loses.csv
  source_url: https://github.com/NeuroPulse-Co/AFC-Prune/tree/main/studies/s1-baselines/results
- type: chart
  title: The ranking holds at every pruning level
  caption: 'The same comparison at five pruning levels. At every level, corrected PGI sits between Taylor
    importance and random choice: better than chance, worse than a simple score. The gap to random is
    widest at 90%, where random choice loses about five points more than PGI.'
  kind: dots
  x: budget
  y: d_acc
  y_label: Change in validation accuracy (points)
  split: val
  series: selector
  x_label: Share of parameters removed
  labels:
    '50.0': 50% removed
    '70.0': 70% removed
    '75.8': 75.8% removed
    '85.0': 85% removed
    '90.0': 90% removed
    pgi_aligned: PGI, corrected
    taylor: Taylor importance
    random_s0: Random filters
    random_s1: Random filters
    random_s2: Random filters
  rows:
  - budget: '50.0'
    selector: pgi_aligned
    d_acc: -3.34
  - budget: '70.0'
    selector: pgi_aligned
    d_acc: -8.14
  - budget: '75.8'
    selector: pgi_aligned
    d_acc: -10.32
  - budget: '85.0'
    selector: pgi_aligned
    d_acc: -16.28
  - budget: '90.0'
    selector: pgi_aligned
    d_acc: -20.08
  - budget: '50.0'
    selector: random_s0
    d_acc: -5.32
  - budget: '70.0'
    selector: random_s0
    d_acc: -10.3
  - budget: '75.8'
    selector: random_s0
    d_acc: -11.32
  - budget: '85.0'
    selector: random_s0
    d_acc: -17.34
  - budget: '90.0'
    selector: random_s0
    d_acc: -23.24
  - budget: '50.0'
    selector: random_s1
    d_acc: -3.8
  - budget: '70.0'
    selector: random_s1
    d_acc: -11.24
  - budget: '75.8'
    selector: random_s1
    d_acc: -11.58
  - budget: '85.0'
    selector: random_s1
    d_acc: -16.66
  - budget: '90.0'
    selector: random_s1
    d_acc: -23.78
  - budget: '50.0'
    selector: random_s2
    d_acc: -3.24
  - budget: '70.0'
    selector: random_s2
    d_acc: -6.96
  - budget: '75.8'
    selector: random_s2
    d_acc: -9.88
  - budget: '85.0'
    selector: random_s2
    d_acc: -14.54
  - budget: '90.0'
    selector: random_s2
    d_acc: -22.14
  - budget: '50.0'
    selector: taylor
    d_acc: -2.68
  - budget: '70.0'
    selector: taylor
    d_acc: -7.94
  - budget: '75.8'
    selector: taylor
    d_acc: -7.66
  - budget: '85.0'
    selector: taylor
    d_acc: -12.56
  - budget: '90.0'
    selector: taylor
    d_acc: -18.06
  - budget: '50.0'
    selector: pgi_aligned
    d_acc: -6.36
  - budget: '70.0'
    selector: pgi_aligned
    d_acc: -9.04
  - budget: '75.8'
    selector: pgi_aligned
    d_acc: -11.8
  - budget: '85.0'
    selector: pgi_aligned
    d_acc: -16.76
  - budget: '90.0'
    selector: pgi_aligned
    d_acc: -20.9
  - budget: '50.0'
    selector: random_s0
    d_acc: -4.72
  - budget: '70.0'
    selector: random_s0
    d_acc: -9.5
  - budget: '75.8'
    selector: random_s0
    d_acc: -11.78
  - budget: '85.0'
    selector: random_s0
    d_acc: -19.92
  - budget: '90.0'
    selector: random_s0
    d_acc: -25.52
  - budget: '50.0'
    selector: random_s1
    d_acc: -6.04
  - budget: '70.0'
    selector: random_s1
    d_acc: -9.5
  - budget: '75.8'
    selector: random_s1
    d_acc: -11.78
  - budget: '85.0'
    selector: random_s1
    d_acc: -18.92
  - budget: '90.0'
    selector: random_s1
    d_acc: -24.78
  - budget: '50.0'
    selector: random_s2
    d_acc: -7.94
  - budget: '70.0'
    selector: random_s2
    d_acc: -10.28
  - budget: '75.8'
    selector: random_s2
    d_acc: -15.22
  - budget: '85.0'
    selector: random_s2
    d_acc: -21.14
  - budget: '90.0'
    selector: random_s2
    d_acc: -28.6
  - budget: '50.0'
    selector: taylor
    d_acc: -3.96
  - budget: '70.0'
    selector: taylor
    d_acc: -7.76
  - budget: '75.8'
    selector: taylor
    d_acc: -9.96
  - budget: '85.0'
    selector: taylor
    d_acc: -13.58
  - budget: '90.0'
    selector: taylor
    d_acc: -19.38
  - budget: '50.0'
    selector: pgi_aligned
    d_acc: -2.68
  - budget: '70.0'
    selector: pgi_aligned
    d_acc: -6.5
  - budget: '75.8'
    selector: pgi_aligned
    d_acc: -9.84
  - budget: '85.0'
    selector: pgi_aligned
    d_acc: -13.9
  - budget: '90.0'
    selector: pgi_aligned
    d_acc: -17.86
  - budget: '50.0'
    selector: random_s0
    d_acc: -4.2
  - budget: '70.0'
    selector: random_s0
    d_acc: -7.6
  - budget: '75.8'
    selector: random_s0
    d_acc: -10.02
  - budget: '85.0'
    selector: random_s0
    d_acc: -17.26
  - budget: '90.0'
    selector: random_s0
    d_acc: -26.04
  - budget: '50.0'
    selector: random_s1
    d_acc: -4.26
  - budget: '70.0'
    selector: random_s1
    d_acc: -9.06
  - budget: '75.8'
    selector: random_s1
    d_acc: -11.52
  - budget: '85.0'
    selector: random_s1
    d_acc: -16.78
  - budget: '90.0'
    selector: random_s1
    d_acc: -24.78
  - budget: '50.0'
    selector: random_s2
    d_acc: -2.9
  - budget: '70.0'
    selector: random_s2
    d_acc: -7.52
  - budget: '75.8'
    selector: random_s2
    d_acc: -9.38
  - budget: '85.0'
    selector: random_s2
    d_acc: -15.14
  - budget: '90.0'
    selector: random_s2
    d_acc: -26.12
  - budget: '50.0'
    selector: taylor
    d_acc: -2.3
  - budget: '70.0'
    selector: taylor
    d_acc: -5.94
  - budget: '75.8'
    selector: taylor
    d_acc: -7.42
  - budget: '85.0'
    selector: taylor
    d_acc: -12.48
  - budget: '90.0'
    selector: taylor
    d_acc: -18.32
  data: ../data/s1-baselines/evidence-03-the-ranking-holds-at-every-pruning-level.csv
  source_url: https://github.com/NeuroPulse-Co/AFC-Prune/tree/main/studies/s1-baselines/results
- type: table
  title: Cheap recovery steps barely help
  caption: Each pruned network at 75.8%, with four levels of recovery that need no gradient training.
    With nothing refitted, pruning costs 44 to 57 points. Refitting the final layer brings that back to
    between −7.4 and −11.8. Re-estimating batch-norm statistics and refitting the next layer then change
    the result by less than one point, and mostly make it slightly worse.
  split: val
  columns:
  - key: tag
    label: Network
  - key: selector
    label: Selection
  - key: tier
    label: Recovery
  - key: val_acc
    label: Validation accuracy (%)
  - key: d_acc
    label: Change (points)
  labels:
    pgi_aligned: PGI, corrected
    taylor: Taylor importance
    l1: Smallest weights (L1)
    T0: Nothing refitted
    T1r: Final layer refitted
    T2r: + batch-norm statistics
    T3r: + next layer refitted
  rows:
  - tag: clean_s42
    selector: pgi_aligned
    tier: T0
    val_acc: 18.24
    d_acc: -57.16
  - tag: clean_s42
    selector: pgi_aligned
    tier: T1r
    val_acc: 65.08
    d_acc: -10.32
  - tag: clean_s42
    selector: pgi_aligned
    tier: T2r
    val_acc: 64.98
    d_acc: -10.42
  - tag: clean_s42
    selector: pgi_aligned
    tier: T3r
    val_acc: 64.46
    d_acc: -10.94
  - tag: clean_s42
    selector: l1
    tier: T0
    val_acc: 19.94
    d_acc: -55.46
  - tag: clean_s42
    selector: l1
    tier: T1r
    val_acc: 67.12
    d_acc: -8.28
  - tag: clean_s42
    selector: l1
    tier: T2r
    val_acc: 66.6
    d_acc: -8.8
  - tag: clean_s42
    selector: l1
    tier: T3r
    val_acc: 66.46
    d_acc: -8.94
  - tag: clean_s42
    selector: taylor
    tier: T0
    val_acc: 28.1
    d_acc: -47.3
  - tag: clean_s42
    selector: taylor
    tier: T1r
    val_acc: 67.74
    d_acc: -7.66
  - tag: clean_s42
    selector: taylor
    tier: T2r
    val_acc: 67.34
    d_acc: -8.06
  - tag: clean_s42
    selector: taylor
    tier: T3r
    val_acc: 67.32
    d_acc: -8.08
  - tag: clean_s43
    selector: pgi_aligned
    tier: T0
    val_acc: 23.44
    d_acc: -52.14
  - tag: clean_s43
    selector: pgi_aligned
    tier: T1r
    val_acc: 63.78
    d_acc: -11.8
  - tag: clean_s43
    selector: pgi_aligned
    tier: T2r
    val_acc: 62.98
    d_acc: -12.6
  - tag: clean_s43
    selector: pgi_aligned
    tier: T3r
    val_acc: 64.0
    d_acc: -11.58
  - tag: clean_s43
    selector: l1
    tier: T0
    val_acc: 31.38
    d_acc: -44.2
  - tag: clean_s43
    selector: l1
    tier: T1r
    val_acc: 65.98
    d_acc: -9.6
  - tag: clean_s43
    selector: l1
    tier: T2r
    val_acc: 65.5
    d_acc: -10.08
  - tag: clean_s43
    selector: l1
    tier: T3r
    val_acc: 65.7
    d_acc: -9.88
  - tag: clean_s43
    selector: taylor
    tier: T0
    val_acc: 18.52
    d_acc: -57.06
  - tag: clean_s43
    selector: taylor
    tier: T1r
    val_acc: 65.62
    d_acc: -9.96
  - tag: clean_s43
    selector: taylor
    tier: T2r
    val_acc: 65.18
    d_acc: -10.4
  - tag: clean_s43
    selector: taylor
    tier: T3r
    val_acc: 65.4
    d_acc: -10.18
  - tag: clean_s44
    selector: pgi_aligned
    tier: T0
    val_acc: 20.08
    d_acc: -54.7
  - tag: clean_s44
    selector: pgi_aligned
    tier: T1r
    val_acc: 64.94
    d_acc: -9.84
  - tag: clean_s44
    selector: pgi_aligned
    tier: T2r
    val_acc: 64.98
    d_acc: -9.8
  - tag: clean_s44
    selector: pgi_aligned
    tier: T3r
    val_acc: 65.06
    d_acc: -9.72
  - tag: clean_s44
    selector: l1
    tier: T0
    val_acc: 19.78
    d_acc: -55.0
  - tag: clean_s44
    selector: l1
    tier: T1r
    val_acc: 66.26
    d_acc: -8.52
  - tag: clean_s44
    selector: l1
    tier: T2r
    val_acc: 66.0
    d_acc: -8.78
  - tag: clean_s44
    selector: l1
    tier: T3r
    val_acc: 66.06
    d_acc: -8.72
  - tag: clean_s44
    selector: taylor
    tier: T0
    val_acc: 19.72
    d_acc: -55.06
  - tag: clean_s44
    selector: taylor
    tier: T1r
    val_acc: 67.36
    d_acc: -7.42
  - tag: clean_s44
    selector: taylor
    tier: T2r
    val_acc: 66.72
    d_acc: -8.06
  - tag: clean_s44
    selector: taylor
    tier: T3r
    val_acc: 66.72
    d_acc: -8.06
  data: ../data/s1-baselines/evidence-04-cheap-recovery-steps-barely-help.csv
  source_url: https://github.com/NeuroPulse-Co/AFC-Prune/tree/main/studies/s1-baselines/results
runs:
- id: 20261124-0000-server-dense-s42
  started: ''
  host: server
  seed: 42
  commit: 12e17ef
  exit_code: null
  backfilled: true
- id: 20261124-0000-server-dense-s43
  started: ''
  host: server
  seed: 43
  commit: 12e17ef
  exit_code: null
  backfilled: true
- id: 20261124-0000-server-dense-s44
  started: ''
  host: server
  seed: 44
  commit: 12e17ef
  exit_code: null
  backfilled: true
- id: 20261124-0000-server-s1-closed
  started: ''
  host: server
  seed: null
  commit: fffac12
  exit_code: null
  backfilled: true
- id: 20261124-0000-server-s1-gate
  started: ''
  host: server
  seed: 42
  commit: fffac12
  exit_code: null
  backfilled: true
- id: 20261124-0000-server-s1-gradient
  started: ''
  host: server
  seed: 42
  commit: fffac12
  exit_code: null
  backfilled: true
code_url: https://github.com/NeuroPulse-Co/AFC-Prune/tree/main/studies/s1-baselines
---

<!-- Generated by record.py promote from studies/s1-baselines/study.yaml and FINDINGS.md. Edit those files, not this one. -->

*This study is still running. Results will appear here when it closes.*
