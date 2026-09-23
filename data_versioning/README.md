# data_versioning/

How data is version-controlled in this project.

## The rule

**Git tracks code and metadata. DVC tracks dataset metadata. Neither tracks raw
image bytes.**

| Artefact | Tracked by | In git? |
|---|---|---|
| Notebooks, README, requirements | Git | yes |
| Results (CSV, PNG) | Git | yes |
| Checkpoints (`.pth`) | Google Drive | **no** |
| Raw images | nothing (regenerable from Kaggle) | **no** |
| Manifests / inventories / hashes | Git | yes |

## Why manifests are the versioning unit

The dataset is small (~7 MB) but still must not enter git. It is instead pinned
by a **content fingerprint**: `results/preprocessing/clean_dataset_manifest.csv`
and `results/preprocessing/final_split_manifest.csv` record, for every image,
its source id, relative path, class, split, group id, byte size, width, height,
channels and a SHA-256 content hash.

If a future run produces a different manifest, the dataset has changed, and the
commit that changed it is the record of what changed and when.

## DVC usage

DVC is initialised in the repository and configured to track **metadata only**:

- `.dvcignore` excludes every image extension, so DVC cannot hash image content.
- `dvc.yaml` (stage definitions) and DVC-tracked manifests give stage-level
  reproducibility of the preprocessing pipeline.
- **No DVC remote is configured and none is required.** No credentials are
  needed to clone this repository and reproduce the pipeline.

If you want to add a remote yourself:

```bash
dvc remote add -d myremote <your-storage-url>
dvc push
```

That step is optional and intentionally left to the user.

## Git commit roles

| Commit | Contents |
|---|---|
| initial | project scaffold, notebooks, docs |
| `Add BSM data manifests and preprocessing results` | manifests, wrangling/imputation reports |
| `Run BSM mineral classification experiments and add results` | metrics, confusion matrices, training curves, comparisons |

## Checklist before any push

- [ ] no `*.pth`, `*.pt`, `*.ckpt`
- [ ] no images under `data/`
- [ ] no `kaggle.json`, `.env`, tokens
- [ ] no `.py` files
