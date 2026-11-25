<!-- Lab log for study `s1-baselines`. Newest entry first.
     Add an entry with `record.py log s1-baselines`; the format is in standards/RECORDS.md.
     Entries before 2026-11-25 were written in the repo-wide afcprune LOG and moved here unchanged. -->

# 2026-11-25 — Server lost power during the gradient tiers; afcprune moved into its own repository
Study: s1-baselines · Author: Dewmina Wijekoon · Host: laptop · Commit: a76ab67 · Runs: 20261124-0000-server-s1-gradient

## Run
| What | Command or run id | Result |
|---|---|---|
| Gradient tiers T1g, T1fc, T4 (run01_paper, then clean_s42) | 20261124-0000-server-s1-gradient | interrupted by a power loss on the GPU server; progress unknown |
| Move afcprune out of ASF-S | git subtree split of afcprune/ | history kept; this is now the AFC-Prune repository |

## Findings
- The GPU server went down, most likely from a power cut, while the gradient tiers were running. The committed snapshot holds only the first three gradient rows (run01_paper, corrected PGI at 50%: T1g −3.26, T1fc −3.18, T4 −1.47, paper protocol). Everything after that is either on the server's disk or lost.
- None of the closed-form results are affected. All four checkpoints finished before the outage, and they are committed in `results/`.
- We rebuilt the records under the NeuroPulse record standard. This study keeps its earlier log entries unchanged below; six runs were backfilled from this log and marked as such. The S0 entry moved to `studies/s0-skeleton/LOG.md`.
- `experiments/run_s1.sh` no longer assumes it sits inside ASF-S. On the server, set `PY` and `AFCP_DATA` to the ASF-S `studies/cifar10-repro` venv and data before resuming.

## Next
- [ ] When the server is back, check `~/afcp_outputs/s1/*/results.csv` and the tmux session `afcp_s1` to see how far the gradient tiers got.
- [ ] Pull the restructured repositories on the server, set `PY` and `AFCP_DATA`, and resume `run_s1.sh gradient` (finished rows are skipped). Wrap it with `tools/record.py run s1-baselines --tag gradient-resume`.
- [ ] Decide the S2 go/no-go criterion in writing before the gradient results come in (plan.md, R0).

# 2026-11-24 17:40 — S1 closed-form complete (3 clean seeds) + run01 gate PASSES

## Gate
run01 + pgi_aligned @ (19,16), T1fc (fc FT, BN adapting), seed 42: **ΔAcc −10.47** vs −10.50 (3-seed mean
in cifar10 step 8). PASS (±0.5). afcprune reproduces the earlier pipeline end to end on real data.

## Dense (clean protocol) val acc
s42 75.40, s43 75.58, s44 74.78.

## T1r (closed-form ridge head), mean ± std over 3 clean seeds (ΔAcc val)
| selector | 50% | 70% | 75.8% | (19,16) | 85% | 90% |
|---|---|---|---|---|---|---|
| taylor | −3.0±0.9 | −7.2±1.1 | **−8.4±1.4** | −9.2±0.8 | **−12.9±0.6** | −18.6±0.7 |
| bn_gamma | −3.2±0.9 | −6.7±1.1 | **−8.4±1.0** | **−9.1±1.1** | −13.5±1.0 | **−17.1±0.9** |
| l1 | **−2.9±0.3** | **−6.6±1.1** | −8.8±0.7 | −12.4±1.0 | −14.0±2.0 | −19.7±2.6 |
| l2 | −3.1±0.3 | −6.7±0.9 | −9.8±0.8 | −11.7±0.8 | −14.2±0.9 | −18.8±0.7 |
| pgi_aligned | −4.1±2.0 | −7.9±1.3 | −10.7±1.0 | −11.3±0.2 | −15.7±1.5 | −19.6±1.6 |
| random | −4.7±1.3 | −9.1±0.9 | −11.4±1.4 | −14.8±2.1 | −17.5±2.1 | −25.0±1.7 |
| pgi_literal | −4.1±0.1 | −8.4±0.6 | −12.5±2.8 | −13.3±1.8 | −17.2±1.8 | −24.0±4.8 |

Paired per-seed differences (T1r), mean [min, max] over the 3 seeds:
- **pgi_aligned − taylor: worse at every budget** (−0.7 to −2.8). At 75.8% it is −2.3 [−2.7, −1.8], all three
  seeds negative.
- **pgi_aligned − l1: worse at 4 of 6 budgets** (−1.2 to −1.9). Better only at the fixed (19,16) point (+1.1).
- pgi_aligned − random: better everywhere (+0.6 to +5.4).
- taylor − l1 and taylor − bn_gamma: within about ±1 except at (19,16). **The top cluster (Taylor, BN-γ, L1,
  L2) is statistically tied at most budgets.**

## Findings
1. **Corrected PGI is a weak criterion.** It beats random but loses to every simple baseline, consistently
   across seeds. The paper's ranking (PGI > L1/L2/random) does not hold under matched budgets.
2. **Head-only ceiling at 75.8% ≈ −8.4** (best selectors, closed form), against the paper's −3.78.
3. **T3r (LS compensation of conv2) helps only when conv1 is pruned.** The gain is +1.25 at the fixed
   (19,16) point and ≈ 0 at val-chosen allocations, which keep conv1 whole. That is consistent: with conv1
   intact, conv2's inputs are unchanged and there is nothing to compensate. T2r (BN recalibration) never
   helps.
4. Seed-to-seed std is about 1 point, so single-seed differences below about 2 points (as in the cifar10
   branch) were not interpretable.

## Implication for S2
The bar for an AFC selector is the Taylor / BN-γ / L1 cluster (≈ −8.4 at 75.8%, T1r), not random. Gradient
tiers (running now) will show whether T4 closes the gap to the paper's number.

---

# 2026-11-24 16:30 — S1 closed-form tiers: first results (run01_paper, clean_s42 complete)

288 rows per checkpoint: 12 selectors (random ×3) × 6 budgets × T0/T1r/T2r/T3r. Raw data in
`results/s1/<tag>/` (rsynced from the server). Values are ΔAcc on val vs dense; for clean_s42 the dense val
acc is 75.40. **One dense seed each, no CIs yet.**

clean_s42, T1r (closed-form ridge head only, strict head-only):

| selector | 50% | 70% | 75.8% | (19,16) | 85% | 90% |
|---|---|---|---|---|---|---|
| taylor | −2.7 | −7.9 | **−7.7** | **−8.8** | **−12.6** | −18.1 |
| bn_gamma | −3.2 | **−6.0** | −8.1 | −8.7 | −14.1 | **−16.3** |
| l1 | −3.0 | −6.1 | −8.3 | −12.3 | −15.0 | −20.8 |
| pgi_aligned | −3.3 | −8.1 | −10.3 | −11.2 | −16.3 | −20.1 |
| random (mean of 3) | −4.1 | −9.5 | −10.9 | −13.6 | −16.2 | −23.1 |
| pgi_literal | −4.1 | −8.1 | −11.0 | −13.6 | −16.9 | −20.5 |

## Findings
1. **PGI is not a good criterion, even corrected.** At 75.8%, aligned PGI (−10.3) is close to random (−10.9)
   and 2.6 points behind Taylor. The original (literal) PGI equals random. Same picture on run01.
2. **Taylor and BN-γ are the strongest baselines**, then L1/L2. The bar for any AFC method is Taylor, not L1.
3. **Nothing comes close to the paper's −3.78 at 75.8% head-only.** The best is −7.7 (Taylor, T1r).
4. **BN recalibration (T2r) and conv2 LS compensation (T3r) barely help** (±0.5, sometimes worse). Whatever
   the head cannot recover is not a linear distortion of conv2 inputs. Expect the gap to close only with
   gradient training of the conv layers (T4).
5. **Allocation keeps conv1 whole** (32/15 at 75.8% for most selectors), confirming the cifar10 grid
   finding that conv1 pruning hurts most.
6. **Allocation-criterion weakness.** The criterion (val accuracy after BN recalibration, no head refit) sits
   at 20–50% for heavy pruning, so it is noisy and does not predict post-refit accuracy well. Example:
   run01 pgi_literal picked 2/17 at 70%, then scored −22.6 after the ridge head. **Proposed fix for the next
   run:** judge candidates by ridge-head val accuracy on a fixed 10k train subsample. It is cheap, matches
   the headline tier, and is still the same rule for all selectors. Not changed mid-run.
7. P3 holds on clean data too: clean_s42 has σ 0.857, top-4 var 0.931, min λ2−λ3 0.136, every group at
   S = 50, and bootstrap PG1 \|cos\| ≈ 0.99.

## Next
- Let clean_s43/s44 finish, then compute mean ± std over the 3 clean seeds before trusting any ranking.
- The gradient phase (T1g/T1fc/T4, run01 + clean_s42) starts automatically afterwards.
- Rerun the allocation with the ridge-based criterion (finding 6).

---

# 2026-11-24 — S1 launched (server, tmux `afcp_s1`, commit fffac12)

## Queue (`~/afcp_outputs/s1_queue.sh`, results in `~/afcp_outputs/s1/<tag>/`)
1. Wait for the dense trainings (tmux `afcp_dense`) to finish.
2. `run_s1.sh closed`: T0/T1r/T2r/T3r × 12 selectors (random ×3) × 6 budgets, on run01_paper, clean_s42,
   clean_s43 and clean_s44. Estimate about 1.7 h per model.
3. `run_s1.sh gate`: pgi_aligned @ (19,16), T1fc, on run01 (paper protocol).
4. Gradient tiers T1g/T1fc/T4 on **run01_paper and clean_s42 only** (about 10 h each). The other seeds are
   decided after looking at these.

## Dense models (clean protocol, 45k/5k, cifar10 step 1 recipe)
| seed | best val acc | test acc | best epoch / run |
|---|---|---|---|
| 42 | 75.40 | 75.64 | 99 / 100 |
| 43, 44 | (fill in when rsynced) | | |

- **Test-set looks outside the guard: one per dense model.** cifar10 `step1_train.py` evaluates test at
  the end and writes it to gate.json. It is reported only and never used to choose anything, but it counts
  toward the audit (plan P4). afcprune's own test reads still go only through `protocol.allow_test`.
- Seed 42's best epoch is 99 of 100: the paper's 100-epoch recipe stops while val loss is still falling.
  This is the same recipe as the paper, so it is not changed. A longer schedule is the cheapest way to make
  the dense baseline stronger, if needed later.
- Still open from plan P9: the weak (~72.7%) checkpoint. Queue it after S1's closed phase (for example,
  stop seed 45 at the epoch where val ≈ 72.7).

---

# 2026-11-24 — S1 prep: runner, smoke test on the server, dense training started

## Server state
- Host `intellisense08-EWISPro9900G` (RTX 2080). The repo is on `afc_prune` at 12e17ef. The afcprune suite
  passes there too: **73 passed** with torch 2.5.1+cu121, so the golden outputs hold across torch versions
  and devices.
- Dense checkpoints (run01, seed43, diag_tv) are backed up to the laptop under `~/asfs_backups/`.
- Clean-protocol dense models started in tmux `afcp_dense` with `cifar10/steps/step1_train.py --protocol
  clean --seed {42,43,44} --run-id afcp_dense_s{seed}`. That is the same model, data and loop code as
  afcprune's port.

## New code (uncommitted)
- `afcprune/pipeline.py`: output-layer AFC → PG1 → ΔG → PCA in one call, with the same seeded draw as
  cifar10 step 3, plus P3 diagnostics. Test: `tests/test_pipeline.py` (2 passed).
- `experiments/s1_baselines.py`: selector × budget × tier for one checkpoint, val-chosen allocation
  (BN-recalibrated val accuracy over the 40 nearest candidates), resumable CSV.
- `experiments/run_s1.sh`: phases `closed` / `gradient` / `gate`.

## Smoke run (scratch copy `~/afcp_scratch`, not the repo)
| | |
|---|---|
| Command | `s1_baselines.py --dense run01 --protocol paper --budgets t5pt,70 --selectors pgi_aligned,l1 --tiers T0,T1r,T2r,T3r,T1fc --max-epochs 1 --alloc-max 3` |
| Time | 3.1 min |
| Contrast diagnostics | dense 75.32, σ 0.831, top-4 var 0.927, min λ2−λ3 0.152, min group 50. All equal run01's step 2–5 logs, so the port matches on real data. Bootstrap PG1 median \|cos\| 0.993: PG1 is stable on CIFAR-10 (P3) |

Smoke numbers (val = test, 1 FT epoch; **not results**):

| | t5pt T1r | t5pt T3r | 70% T1r | 70% T3r |
|---|---|---|---|---|
| pgi_aligned | −11.34 | −9.94 | −8.10 | −7.72 |
| l1 | −12.72 | −10.62 | −7.58 | −7.48 |

- The closed-form ridge head alone (about 10 s) lands near run01's gradient fc FT (−10.50). The
  allocation at 70% picks 31/19, keeping conv1 almost whole, as the cifar10 grid predicted.
- Gate caveat: run01's −10.50 is a **3-seed mean**. The gate runs one FT seed with ±0.5, so seed noise
  alone could fail it. If it does, rerun with `--ft-seed 43/44` and compare the mean.
