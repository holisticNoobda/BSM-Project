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
│   └── 03_Final_Comparison.ipynb
└── results/
    ├── dataset_summary/      inventory, class distribution, dimensions, samples
    ├── preprocessing/        wrangling, imputation, splits, version summary
    ├── augmentation/         augmentation manifest + examples
    ├── metrics/              per-epoch history, classification reports, final results
    ├── confusion_matrices/   PNG + numeric CSV per model
    ├── training_curves/      loss / accuracy curves per model
    └── comparisons/          model comparison, paper-vs-ours, discussion
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

## 7. Checkpoints

Best-validation-accuracy weights are written to Google Drive at:

```
/content/drive/MyDrive/BSM_Mineral_Classification/models/
├── VGG16_best.pth
├── ResNet18_best.pth
├── ResNet34_best.pth
└── D_ResNet_best.pth
```

`.pth` / `.pt` / `.ckpt` files are git-ignored and are never pushed.

## 8. Not reproduced

| Paper element | Status |
|---|---|
| Unseen 18-image experiment (3 per class) | **Not reproduced** — no separate unseen dataset was available. |
| RMSE / error analysis over SVM, CART, KNN, Random Forest, SGB, Cubist | **Not reproduced** — required data and methodology were unavailable for those stages. |

Neither is fabricated or estimated.

## 9. Configuration

| Parameter | Value | Source |
|---|---|---|
| Image size | 224 x 224 x 3 RGB | implementation requirement |
| Normalisation | ImageNet mean/std | implementation requirement for pretrained torchvision weights |
| Epochs | 60 | paper (epochs 1–60) |
| Batch size | 32 | engineering choice |
| Optimiser | Adam, lr 1e-4 | engineering choice |
| Loss | CrossEntropyLoss | engineering choice |
| Split | 70 / 15 / 15, source-group level | engineering choice + leakage safeguard |
| Seed | 42 | reproducibility |
| Model selection | best validation accuracy | engineering choice |

## 10. License / attribution

Dataset © Prasannavenkatesan T., distributed via Kaggle and Mendeley Data.
