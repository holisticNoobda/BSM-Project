# 05 — Tuning vs Baseline (generated)

## Outcome per architecture

Selection was on **validation** accuracy across three seeds; the test columns are reported for transparency and were not used to choose.

| Model | A_baseline test (mean±sd) | B_reg test (mean±sd) | Δ test | Verdict |
|---|---|---|---|---|
| D_ResNet | 0.7143 ± 0.0309 | 0.7857 ± 0.0309 | +0.0714 | inconclusive (inside seed spread) |
| ResNet18 | 0.7679 ± 0.0179 | 0.8036 ± 0.0357 | +0.0357 | inconclusive (inside seed spread) |
| ResNet34 | 0.7321 ± 0.0472 | 0.7619 ± 0.0273 | +0.0298 | inconclusive (inside seed spread) |
| VGG16 | 0.7143 ± 0.0179 | 0.7321 ± 0.0179 | +0.0179 | inconclusive (inside seed spread) |

## Seed spread of the notebook 02 configuration

| Model | seed 42 (60 ep) | mean of 3 seeds | sd | movement in test images |
|---|---|---|---|---|
| D_ResNet | 0.6964 | 0.7143 | 0.0309 | 1.7 |
| ResNet18 | 0.8036 | 0.7679 | 0.0179 | 1.0 |
| ResNet34 | 0.7321 | 0.7321 | 0.0472 | 2.6 |
| VGG16 | 0.7679 | 0.7143 | 0.0179 | 1.0 |

## Limitations

- **Aggregate only.** Notebook 04 recorded test accuracy per trial and no per-class reports, confusion matrices or checkpoints, so the tuned models cannot be broken down by class. Only the baseline has that detail.
- **56 test images.** One image is 1.8% of accuracy. Wilson intervals for any single configuration are wide, and three seeds narrow the baseline's spread but do not shrink the test set.
- **One split, three seeds.** These seeds measure training variability on a single fixed split. They do not measure split variability, which for 256 unique training images is likely the larger of the two.
- **20-epoch trials** against a 60-epoch baseline. Comparable in the sense that best epochs were early, but the baseline numbers here are not a re-trained 20-epoch control.
- **Near-duplicate and background confounding** documented in the README applies unchanged to every tuned model.
- **No architecture ranking is claimed** from either notebook.
