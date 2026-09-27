# Final Discussion

## 1. What was run

- Models trained: VGG16, ResNet18, ResNet34, D_ResNet
- Identical configuration for every model: 60 epochs, batch size 32, Adam at lr 0.0001, CrossEntropyLoss, seed 42.
- Best model by test accuracy: **ResNet18** (test accuracy 0.8036).
- Weakest model by test accuracy: D_ResNet (test accuracy 0.6964).
- Mean test accuracy across the four models: 0.7500.
- Mean accuracy difference vs the paper's reported values: -0.0825 (our value minus paper value).

## 2. Reproduction limitations

**D-ResNet is an approximation.** The paper's D-ResNet is a 50-layer bottleneck residual network. Its original source implementation and training hyperparameters were not available, so `torchvision` ResNet50 was used as a practical approximation. The `D_ResNet` row in every table is ResNet50, **not** the original D-ResNet, and no claim of exact reproduction is made.

**Augmented dataset size differs from the paper.** The paper reports an augmented dataset of approximately 1468 images. This implementation applies the paper's described transforms (horizontal flip plus rotations of 0/30/45/90/180/270 degrees) to the training split only, producing a different count. The count was not forced to match. See `results/augmentation/augmentation_count_comparison.csv`.

**Dataset size differs from the paper's stated total.** The paper's text states 357 images, but the paper's own six class counts sum to 367, which is the number actually present in the dataset. Actual counts are reported in `results/dataset_summary/dataset_summary.csv`.

**Split ordering differs from the paper, deliberately.** The paper augments before splitting, which places rotated or flipped siblings of a training image into the test set. This implementation splits first at the source-image level and augments the training split only, so test images have no counterpart in training. This is a reproducibility safeguard required by the project specification, and it is the most likely reason our numbers differ from the paper's.

## 3. Why our accuracy may be lower than the paper's

Three factors, in order of expected importance:

1. **Leakage-free test set.** Our test images are unseen originals. The paper's protocol places augmented variants of training images into its test set, which inflates measured accuracy. Comparing the two directly is not like-for-like.
2. **Small test set.** The 15% clean test split is a few dozen images, so a single image moves accuracy by a couple of percentage points. Differences of a few points between models should not be over-interpreted.
3. **The 367-image dataset is tiny for six-way classification.** With no hyperparameter tuning and a single seed, every reported figure is a point estimate without a run-to-run error bar.

## 4. Training behaviour and what the numbers can support

The best model, ResNet18, reached its peak validation accuracy at epoch 30 and finished epoch 60; its final training accuracy was 1.0000 against a final validation accuracy of 0.8364.

**Training accuracy saturates almost immediately.** First epoch at which training accuracy reached 0.99 or higher:

| Model | saturation epoch | final train accuracy |
|---|---|---|
| VGG16 | 6 | 1.0000 |
| ResNet18 | 2 | 1.0000 |
| ResNet34 | 2 | 1.0000 |
| D_ResNet | 2 | 1.0000 |

The training split expands 256 source photographs to 3072 samples, but there are only **256 unique source images**. Augmentation multiplies pixels, not information, so the effective sample size is the number of unique photographs. Every model fits them within the first few epochs and the remaining epochs add overfitting rather than accuracy. The 60-epoch budget was run in full as specified, but it is not the operative setting.

**Best validation epochs:** VGG16 4, ResNet18 30, ResNet34 2, D_ResNet 8.

**The accuracy ranking is not statistically significant.** 95% Wilson intervals on the test split:

| Model | correct | n | test accuracy | 95% CI |
|---|---|---|---|---|
| VGG16 | 43 | 56 | 0.7679 | 0.642-0.859 |
| ResNet18 | 45 | 56 | 0.8036 | 0.682-0.887 |
| ResNet34 | 41 | 56 | 0.7321 | 0.604-0.830 |
| D_ResNet | 39 | 56 | 0.6964 | 0.567-0.801 |

The best model (ResNet18, 45/56) and the weakest (D_ResNet, 39/56) differ by 6 test images and their confidence intervals **overlap**. No claim is made that one architecture is better than another on this dataset; the ordering is a point estimate from a single seed on a few dozen test images.

**Confound carried from a separate audit of this dataset:** background composition was found to correlate with class in an independent analysis, which would make every accuracy here an upper bound on mineral identification. That analysis was not re-run as part of this project, so it is reported as a known risk rather than a measured result here.

## 5. Metric definitions

- **Recall and sensitivity** are the same quantity for single-label multiclass classification and are both reported.
- **Specificity** is computed one-vs-rest as TN / (TN + FP) for each class, then macro- and weighted-averaged. sklearn does not provide this directly, so it is derived from the confusion matrix in notebook 02.
- Best-checkpoint selection used **validation accuracy only**. The test set was evaluated once per model, after training finished.

## 6. Not reproduced

- **Unseen 18-image experiment (3 images per class).** Not reproduced: no separate unseen dataset was available. Not fabricated.
- **RMSE / error analysis over SVM, CART, KNN, Random Forest, SGB, Cubist.** Not reproduced: the required data and methodology for those stages were unavailable. Not fabricated.

## 7. Artefact index

| Artefact | Path |
|---|---|
| Model comparison table | `results/comparisons/model_comparison.csv` |
| Model comparison graph | `results/comparisons/model_comparison.png` |
| Paper vs our results | `results/comparisons/paper_vs_our_results.csv` |
| Paper vs our accuracy | `results/comparisons/paper_vs_our_accuracy.png` |
| Per-model histories | `results/metrics/<Model>_history.csv` |
| Per-model reports | `results/metrics/<Model>_classification_report.csv` |
| Confusion matrices | `results/confusion_matrices/<Model>_confusion_matrix.{png,csv}` |
| Training curves | `results/training_curves/<Model>_{loss,accuracy}.png` |
| Checkpoints | Google Drive `BSM_Mineral_Classification/models/` |