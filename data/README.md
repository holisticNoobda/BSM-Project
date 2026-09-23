# data/

**No images are stored in git.** This directory holds raw and processed image
folders locally and on Colab only.

Expected layout (created automatically by notebook 01):

```
data/
├── raw/BSM/
│   ├── Ilmenite/
│   ├── Rutile/
│   ├── Zircon/
│   ├── Garnet/
│   ├── Monazite/
│   └── Sillimanite/
├── processed/     # 224x224 resized copies
└── augmented/      # rotation / horizontal-flip variants, TRAIN SPLIT ONLY
```

## Getting the data

**On Google Colab** — notebook 01 downloads it automatically from the public
Kaggle dataset `prasannavenkatesant/beach-sand-mineral-bsm`. Credentials are
read from Colab secrets (`KAGGLE_USERNAME`, `KAGGLE_KEY`) or an uploaded
`/content/kaggle.json`. **No credential is ever written into a notebook or
committed to git.**

**Locally** — download and extract so that the six class folders land in
`data/raw/BSM/`.

## Why `.gitignore` blocks it

`.gitignore` ignores everything under `data/` except this README. 367 JPEGs is
only ~7 MB, but committing raw dataset images to a public repository is bad
practice and is explicitly prohibited by the project specification. Data is
versioned by **manifest** instead — see `../data_versioning/README.md`.
