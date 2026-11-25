# AFC-Prune

AFC-preserving structured pruning. This repository continues our reproductions of Artificial Structure Function Search (ASF-S) in the [ASF-S repository](https://github.com/NeuroPulse-Co/ASF-S). It keeps the seed paper's idea, which is to preserve the output layer's Artificial Functional Connectivity (AFC) so that the pruned layers need no retraining, and tests it under a protocol built to survive review.

Part of **NeuroPulse**, a final-year research project on structure-function-aware pruning for continual learning. Project site: https://neuropulse-co.github.io · **Workstream:** W2, AFC pruning protocol.

Positioning against prior work: [`docs/related_work.md`](docs/related_work.md). The project-wide plan, including the go/no-go decision for this workstream, is in [NeuroPulse-Co/.github](https://github.com/NeuroPulse-Co/.github/blob/main/standards/PLAN.md).

## Status

| Stage | What | Status | Records |
|---|---|---|---|
| S0 | Skeleton, ports, protocol guard, golden regression | **done** (closed-positive), 75 tests | [`studies/s0-skeleton`](studies/s0-skeleton/) |
| S1 | Dense models (3 clean seeds + run01) and the selector × tier × budget baseline factorial | **running**: closed-form tiers done on all checkpoints; gradient tiers interrupted by a server outage on 2026-11-25 | [`studies/s1-baselines`](studies/s1-baselines/) |
| S2 | Does AFC carry information beyond logit matching? (go / no-go) | planned | |
| S3 | Closed-form head and recovery tiers | planned | |
| S4 | Multi-layer AFC, FC-redundancy, iterative, allocation, class-conditional | planned | |
| S5 | Frozen configs, one test-set look, Conv6Net, paper scorecard | planned | |

So far, corrected Principal Gradient Importance (PGI) beats random choice but loses to simple scores. At 75.8% of parameters removed with a closed-form output head, Taylor importance loses 8.35 ± 1.40 points and PGI 10.65 ± 1.02 (validation, 3 seeds), against the paper's 3.78.

## Layout

| Module | Contents |
|---|---|
| `afc.py` | Eq. 1–8 (cosine AFC, Gaussian W, diffusion K, PG1), plus the top-k embedding, principal-angle subspace similarity and bootstrap PG1 stability |
| `contrast.py` | Eq. 9–11: correct − incorrect contrast ΔG, PCA |
| `scores/` | registry `scorer(model, ctx) → {conv: (F,)}`: `pgi_aligned`, `pgi_literal`, `l1`, `l2`, `random`, `bn_gamma`, `fpgm`, `taylor`, `hrank`, `act_mean` |
| `surgery.py` | exact filter removal (conv, BN, next conv, FC columns); `masked()` is the full-size equivalent |
| `allocate.py` | per-layer kept counts for a total-params↓ budget, chosen on val by one criterion for all selectors |
| `recovery.py` | tiers T0–T4: `ridge_head`, `bn_recalibrate`, `ls_compensate`, `finetune(fc / fc_frozen / full)` |
| `protocol.py` | splits; the test set can only be read inside `allow_test(reason)`, and every access is logged |
| `models.py` `data.py` `train.py` `metrics.py` | ported from `cifar10/` (Conv2Net / Conv6Net, GPU augmentation, fit loop, params / FLOPs / ECE) |

## Recovery tiers (the headline is T1)

| Tier | What may change after pruning |
|---|---|
| T0 | nothing |
| T1 | only the output FC: `fc_frozen` gradient FT or closed-form `ridge_head` |
| T2 | T1 + BatchNorm statistics re-estimated (forward passes only) |
| T3 | T2 + closed-form least-squares refit of the next conv's weights (no gradients) |
| T4 | full fine-tuning |

run01's −10.50 (cifar10 step 8) used mode `fc`, in which BN statistics adapt during FC training. That sits
between T1 and T2.

## Setup and tests

```bash
python -m venv .venv && .venv/bin/pip install torch torchvision -e '.[test]'   # pick the torch wheel for your driver
.venv/bin/python -m pytest -q
```

`tests/golden/golden.npz` freezes the outputs of the original ASF-S CIFAR-10 code (`studies/cifar10-repro/asfs` in the ASF-S repository) on synthetic inputs. `test_golden.py` checks that afcprune reproduces them exactly. Regenerate it only when that reference changes, with `tests/golden/make_golden.py` (see its docstring).

## Running experiments

`experiments/run_s1.sh` reads dense checkpoints from `$ASFS_OUTPUTS` and writes to `$AFCP_OUTPUTS`. On the server, point `PY` and `AFCP_DATA` at the ASF-S `studies/cifar10-repro` venv and data. Wrap new runs in the record tool so that the code version, seed and environment are captured:

```bash
uv run tools/record.py run s1-baselines --tag gradient-resume -- experiments/run_s1.sh gradient
```

## How we keep records

Each stage is a study in `studies/<id>/`, with a `study.yaml`, a `LOG.md`, a `FINDINGS.md` once it closes, and one `runs/<run-id>/run.yaml` per run. We follow the [NeuroPulse record standard](https://github.com/NeuroPulse-Co/.github/blob/main/standards/RECORDS.md). `uv run tools/record.py check` validates every record and also runs in CI. The package code (`afcprune/`), the tests and `experiments/` are shared by all stages, so they stay at the repository root.
