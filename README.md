# BSM Mineral Image Classification

Reproduction of the research paper **"D-ResNet: deep residual neural network for exploration, identification and classification of beach sand minerals"** (Theerthagiri & Prasannavenkatesan, 2022).

Six beach sand mineral (BSM) classes are classified from microscope images:
`Ilmenite`, `Rutile`, `Zircon`, `Garnet`, `Monazite`, `Sillimanite`.

---

## 1. Reproduction limitation — read this first

The paper proposes **D-ResNet**, described as a 50-layer residual architecture built
from bottleneck blocks. The original source implementation and all training
hyperparameters are **not publicly available**.

> **Because the original D-ResNet source implementation is not available,
> torchvision ResNet50 is used as a practical approximation of the paper's
> 50-layer bottleneck residual architecture. This is not claimed to be an exact
> reproduction of the original D-ResNet.**

No result in this repository should be read as a claim of exact reproduction.

## 2. Reproducibility safeguards

Two deviations from the paper were made deliberately and are documented in
`results/preprocessing/dataset_version_summary.csv`:

| Safeguard | Paper behaviour | This implementation |
|---|---|---|
| **Split ordering** | Augment first, then split | **Split first (at source-image level), augment train split only** |
| **Test set** | Augmented test set | Clean, non-augmented test set |

Without this, augmented siblings of a training image land in the test set, and
test scores become a measurement of memorisation rather than generalisation.
Split-before-augment is an implementation safeguard required by the project
specification, not a claim about the paper.

## 3. Dataset

| | |
|---|---|
| Source | Kaggle `prasannavenkatesant/beach-sand-mineral-bsm` (public) |
| Mirror | Mendeley Data `10.17632/9kyrz3shk8` |
| Classes | 6 |
| Images present | **367** |

The paper's text states **357** images, but the paper's own six class counts sum
to **367**, and several are attached to the wrong mineral. This implementation
reports the **actual** counts found on disk and never forces the total to match
the paper. See `results/dataset_summary/dataset_summary.csv`.

Raw images are **not** committed. See `data/README.md` and `data_versioning/README.md`.

## 4. Execution environment

Google Colab, GPU runtime.

```
Runtime -> Change runtime type -> Hardware accelerator -> GPU
```

Expected: `Tesla T4`, ~15 GB VRAM, CUDA available. Training aborts before the
full run if `torch.cuda.is_available()` is `False` — four models are never
silently trained on CPU.

## 5. Project structure

```
BSM-Project/
├── README.md
├── requirements.txt
├── .gitignore
├── .dvcignore
├── data/                 # raw images live here; never committed
├── data_versioning/      # DVC metadata + versioning strategy
├── models/               # README only; checkpoints go to Google Drive
├── notebooks/
│   ├── 01_Data_Preprocessing_and_EDA.ipynb
│   ├── 02_Model_Experiments.ipynb
│   ├── 03_Final_Comparison.ipynb
│   ├── 04_Tuning.ipynb
│   └── 05_Tuning_vs_Baseline.ipynb
└── results/
    ├── dataset_summary/      inventory, class distribution, dimensions, samples
    ├── preprocessing/        wrangling, imputation, splits, version summary
    ├── augmentation/         augmentation manifest + examples
    ├── metrics/              per-epoch history, classification reports, final results
    ├── confusion_matrices/   PNG + numeric CSV per model
    ├── training_curves/      loss / accuracy curves per model
    ├── comparisons/          model comparison, paper-vs-ours, discussion
    ├── tuning/               24 trials, per-trial curves, verdict JSON
    └── tuning_vs_baseline/   tuning-vs-baseline CSV, figure, conclusion
```

All code lives in `.ipynb` notebooks. There are no `.py` files in this project.

## 6. How to run

On Colab, in order:

1. `01_Data_Preprocessing_and_EDA.ipynb` — validates, wrangles, imputes,
   augments, splits at source-group level, writes manifests.
2. `02_Model_Experiments.ipynb` — GPU check, smoke test, then trains
   VGG16 / ResNet18 / ResNet34 / D-ResNet(=ResNet50) for 60 epochs each.
3. `03_Final_Comparison.ipynb` — reads the generated CSVs only. **Does not
   retrain anything.**
4. `04_Tuning.ipynb` — 24 trials: 2 configurations x 3 seeds x 4 architectures,
   20 epochs each. Resumes from `results/tuning/trials.csv` if interrupted.
5. `05_Tuning_vs_Baseline.ipynb` — reads the tuning outputs plus the notebook 02
   baseline. **Does not retrain anything.**

The notebooks in this repository are the **executed** copies, with their outputs
intact, so the delivered code carries its own evidence of having run.

### Running on Kaggle instead of Colab

Notebooks 01 and 02 target Colab, but the sequence runs on Kaggle Notebooks with
a T4 as well. Two differences matter:

- Kaggle has no `/content` and no Drive mount. Write the API token to
  `/content/kaggle.json` (and `/root/.kaggle/kaggle.json`) from a code cell so
  notebook 01's credential lookup finds it, and clone the repo into
  `/kaggle/working/BSM-Project` so the project-root detection resolves.
- Run notebooks **in the foreground**. A `nohup ... &` background job is killed
  when the Kaggle kernel ends, which loses the run. Download outputs from the
  **Output** panel before the session ends — the working directory is discarded.

Notebook 05's packager detects the absence of Drive and writes its zip to
`/kaggle/working` instead.

### Running headless (`nbconvert`) on Colab — required first step

Colab Secrets and `drive.mount()` are only reachable from the **interactive
Colab UI kernel**. `jupyter nbconvert --execute` starts a subprocess kernel where
they are unavailable: `userdata.get()` blocks and then raises `TimeoutException`,
and the Drive mount fails. Run this once in a Colab **UI cell** before any
headless run:

```python
from google.colab import userdata, drive
import json, os

json.dump({"username": userdata.get("KAGGLE_USERNAME"),
           "key": userdata.get("KAGGLE_KEY")},
          open("/content/kaggle.json", "w"))
os.chmod("/content/kaggle.json", 0o600)
drive.mount("/content/drive")
```

Notebook 01 looks for credentials in the order `/content/kaggle.json` →
environment variables → Colab secrets, and notebook 02 reuses an existing
`/content/drive/MyDrive` rather than re-mounting.

```bash
git clone https://github.com/holisticNoobda/BSM-Project.git && cd BSM-Project
jupyter nbconvert --to notebook --execute notebooks/01_Data_Preprocessing_and_EDA.ipynb \
  --inplace --ExecutePreprocessor.timeout=1800
nohup jupyter nbconvert --to notebook --execute notebooks/02_Model_Experiments.ipynb \
  --inplace --ExecutePreprocessor.timeout=14400 > /content/nb02.log 2>&1 &
```

Notebook 02 takes roughly an hour on a T4. Poll the log from another cell while
it runs; notebook 03 is then quick and produces a single
`BSM_results_<timestamp>.zip` on Drive containing all results plus the three
executed notebooks, with `.pth` files excluded.

## 7. Results

Executed on Google Colab, Tesla T4, 60 epochs per model, batch 32, Adam `1e-4`,
seed 42, AMP. Split 3072 train / 55 validation / 56 test (source-grouped).
Total training time 56 minutes for all four models.

| Model | test accuracy | macro F1 | spec (macro) | best epoch | 95% CI (Wilson) | time |
|---|---|---|---|---|---|---|
| ResNet18 | 0.8036 (45/56) | 0.8018 | 0.9608 | 30 | 0.682 – 0.887 | 7.1 min |
| VGG16 | 0.7679 (43/56) | 0.7667 | 0.9536 | 4 | 0.642 – 0.859 | 26.2 min |
| ResNet34 | 0.7321 (41/56) | 0.7303 | 0.9464 | 2 | 0.604 – 0.830 | 9.2 min |
| D_ResNet (=ResNet50) | 0.6964 (39/56) | 0.6964 | 0.9393 | 8 | 0.567 – 0.801 | 13.5 min |

**Read this before quoting the ranking.** Two limitations bound what these
numbers mean, and both are properties of the dataset rather than the models:

1. **The accuracy ranking is not statistically significant.** The best and
   weakest models differ by **6 test images**, and their 95% Wilson intervals
   overlap. No claim is made that any architecture is better than another here;
   the ordering is a point estimate from a single seed on 56 test images.
2. **There are only 256 unique photographs.** The 3072 training samples are 256
   sources expanded by 6 rotations x 2 flips. Augmentation multiplies pixels,
   not information. Every model reached ~1.000 training accuracy by epoch 2
   (VGG16 by epoch 6) and then overfitted for the remaining ~55 epochs. The
   60-epoch budget was run in full as specified, but it is not the operative
   setting — peak validation accuracy arrives in epochs 2–30.

A known confound, carried from a separate audit of this same dataset:
background composition was found to correlate with class, which would make
every figure above an **upper bound** on mineral identification. That analysis
was not re-run in this project, so it is reported as a risk, not a result.

Full discussion, including the paper-vs-our comparison and the two paper
elements that were not reproduced, is in
[`results/comparisons/final_discussion.md`](results/comparisons/final_discussion.md).

## 8. Tuning study and the seed-spread correction

Notebook 02 reported one accuracy per architecture from a single seed. Section 7
already flags that the ranking is not significant; notebook 04 measured *how
much* it is not significant, by repeating the identical configuration across
three seeds.

24 trials — 2 configurations x 3 seeds (42, 43, 44) x 4 architectures, 20 epochs
each. `A_baseline` is notebook 02's configuration unchanged and doubles as the
control. `B_reg` adds weight decay `1e-4` and label smoothing `0.1`. Dropout was
deliberately *not* tuned: torchvision's VGG16 already contains two `Dropout(0.5)`
layers while the ResNets contain none, so a dropout knob would reduce
regularisation for one architecture and add it for the others. Weight decay and
label smoothing apply uniformly.

| Model | A_baseline (mean±sd) | B_reg (mean±sd) | Δ test | verdict |
|---|---|---|---|---|
| D_ResNet | 0.7143 ± 0.0309 | 0.7857 ± 0.0309 | +0.0714 | inconclusive |
| ResNet18 | 0.7679 ± 0.0179 | 0.8036 ± 0.0357 | +0.0357 | inconclusive |
| ResNet34 | 0.7321 ± 0.0472 | 0.7619 ± 0.0273 | +0.0298 | inconclusive |
| VGG16 | 0.7143 ± 0.0179 | 0.7321 ± 0.0179 | +0.0179 | inconclusive |

Across all 12 trials per configuration, mean test accuracy rose from
**0.7321 ± 0.0349** to **0.7708 ± 0.0372** (+0.0387) while mean validation
accuracy moved only **0.8455 → 0.8530** (+0.0075).

**The honest conclusion is that regularization is not demonstrated to help.**
All four models improved on test, which is a consistent direction, but the gain
sits inside the seed spread, and validation barely moved while test moved nearly
six times as far. When validation and test disagree that sharply, the test
movement is more plausibly seed luck than a real effect. **No configuration is
selected using test accuracy**: each trial's checkpoint was chosen on validation
first, and the configuration decision stands on validation.

### The seed-spread finding revises the section 7 numbers

The single-seed results in section 7 sit **above** their own three-seed means for
both leading models, which means those reported figures were favourable draws
rather than typical outcomes:

| Model | seed 42 (60 ep, section 7) | mean of 3 seeds (20 ep) | sd | movement in test images |
|---|---|---|---|---|
| ResNet18 | 0.8036 | 0.7679 | 0.0179 | 1.0 |
| VGG16 | 0.7679 | 0.7143 | 0.0179 | 1.0 |
| ResNet34 | 0.7321 | 0.7321 | 0.0472 | 2.6 |
| D_ResNet | 0.6964 | 0.7143 | 0.0309 | 1.7 |

Any architecture gap smaller than roughly 2–3 test images was never resolvable
from a single-seed run. This is the same conclusion section 7 reaches from the
overlapping Wilson intervals, now backed by measured seed variability.

Two further limits on this study, both recorded in
[`results/tuning_vs_baseline/tuning_vs_baseline.md`](results/tuning_vs_baseline/tuning_vs_baseline.md):

- **Aggregate only.** Notebook 04 recorded test accuracy per trial but no
  per-class reports, confusion matrices or checkpoints, so tuned models cannot
  be broken down by class. Only the baseline has that detail.
- **One split, three seeds.** These seeds measure training variability on a
  single fixed split. They do not measure *split* variability, which for 256
  unique training images is likely the larger of the two.

## 9. Checkpoints

Best-validation-accuracy weights are written to Google Drive at:

```
/content/drive/MyDrive/BSM_Mineral_Classification/models/
├── VGG16_best.pth
├── ResNet18_best.pth
├── ResNet34_best.pth
└── D_ResNet_best.pth
```

`.pth` / `.pt` / `.ckpt` files are git-ignored and are never pushed.

## 10. Not reproduced

| Paper element | Status |
|---|---|
| Unseen 18-image experiment (3 per class) | **Not reproduced** — no separate unseen dataset was available. |
| RMSE / error analysis over SVM, CART, KNN, Random Forest, SGB, Cubist | **Not reproduced** — required data and methodology were unavailable for those stages. |

Neither is fabricated or estimated.

## 11. Configuration

| Parameter | Value | Source |
|---|---|---|
| Image size | 224 x 224 x 3 RGB | implementation requirement |
| Normalisation | ImageNet mean/std | implementation requirement for pretrained torchvision weights |
| Epochs | 60 | paper (epochs 1–60) |
| Batch size | 32 | engineering choice |
| Optimiser | Adam, lr 1e-4 | engineering choice |
| Loss | CrossEntropyLoss | engineering choice |
| Split | 70 / 15 / 15, source-group level | engineering choice + leakage safeguard |
| Seed | 42 (42, 43, 44 in the tuning study) | reproducibility |
| Model selection | best validation accuracy | engineering choice |

Notebook 04 additionally tests `weight_decay=1e-4` with `label_smoothing=0.1`,
varied over seeds 42/43/44 at 20 epochs. It is the **control-and-candidate**
pair for the section 8 comparison; the section 7 baseline configuration is
unchanged.

## 12. License / attribution

Dataset © Prasannavenkatesan T., distributed via Kaggle and Mendeley Data.
