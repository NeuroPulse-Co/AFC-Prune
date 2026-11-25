## Verdict

We asked whether we could rebuild the whole Artificial Structure Function Search (ASF-S) pipeline as one self-contained, tested package that reproduces our earlier CIFAR-10 code exactly. It does. All 73 tests pass on both the laptop and the GPU server. The golden regression tests compare the new package with the original code on fixed synthetic inputs, and they agree to within 1e-10 (1e-5 for logits).

## Evidence

- **Golden regression.** The first principal gradient (PG1), its eigenvalues, the correct-minus-incorrect contrast, the PCA, Principal Gradient Importance (PGI) in both its corrected and literal forms, and the pruning surgery all match the original code, on both Conv2Net and Conv6Net.
- **Model sizes.** Conv2Net has 183,434 parameters and Conv6Net 815,018, and Conv2Net needs 40,894,484 FLOPs. All three counts equal the paper's Table 4.
- **The paper's operating point.** Keeping 19 and 16 filters removes 75.84% of the parameters, which is the paper's "75.8%".
- **Across machines.** The suite also passes on the server (PyTorch 2.5.1 with CUDA), so the golden outputs do not depend on the device or the PyTorch version.

We also added parts the original code did not have: a guard that logs every read of the test set, a per-layer allocation of kept filters chosen on validation data, ten filter-scoring rules behind one interface, and five recovery tiers, from "change nothing" (T0) to full fine-tuning (T4).

## Why we think this happened

Nothing here was surprising: the mathematics did not change, only the structure of the code. The golden tests are what make this claim checkable rather than assumed.

## Limits

The golden tests use synthetic inputs, so they show that the package computes the same thing as the original code. They do not show that either one is correct. Agreement with real data came in the next study, where the package reproduced our earlier −10.50 result to −10.47.

## What this means for the project

Two definitions came out of this stage, and they shape everything after it:

1. **Our earlier −10.50 result is not strictly "head-only".** It retrained the fully connected (FC) layer while the batch-normalisation statistics kept adapting. The recovery tiers are defined so that this setting has its own name (T1fc).
2. **Most of Conv2Net's parameters are in the FC layer** (89%, or 163,850 of 183,434). Each kept filter in the second convolution carries 2,560 FC parameters, so how many filters each layer keeps matters more than which filters are kept. We therefore report kept counts with every result.

The next study, [S1](../s1-baselines/), uses this package to compare PGI with simple filter scores.
