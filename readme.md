# DVC Practical

A small MLOps practice project that shows how to combine Git (code/version history) with DVC (data/version history).

This repository downloads and transforms a customer dataset, stores the transformed CSV in `data/customer.csv`, and tracks that file with DVC.

## Project Overview

- `src/data_ingestion.py`
Creates the dataset by downloading raw customer data and applying simple transformations.
- `data/customer.csv.dvc`
DVC metafile that tracks the data artifact (`data/customer.csv`) by checksum.
- `data/customer.csv`
Generated data file (tracked by DVC, not by Git directly).

## What the Ingestion Script Does

`src/data_ingestion.py` performs these steps:

1. Loads dataset from a public GitHub raw CSV URL.
2. Keeps columns from index `3` onward.
3. Filters rows where `Length of Membership > 1`.
4. Drops `Avg. Session Length` column.
5. Writes final dataset to `data/customer.csv`.

## Prerequisites

- Git installed
- DVC installed
- Python 3.9+
- Python packages: `pandas`, `numpy`

Install Python dependencies:

```bash
pip install pandas numpy
```

Install DVC (if needed):

```bash
pip install dvc
```

## Initial Setup (First Time)

```bash
git init
dvc init
```

Add remote repositories:

```bash
git remote add origin https://github.com/shivam2003-dev/dvc-practical.git
dvc remote add -d myremote /tmp/dvc-storage
```

Notes:
- `origin` is your GitHub repository.
- `myremote` is a local filesystem DVC remote (`/tmp/dvc-storage`). Change it if you want persistent/cloud storage.

## Generate and Track Data

Run ingestion:

```bash
python3 src/data_ingestion.py
```

Track generated file with DVC:

```bash
dvc add data/customer.csv
```

This updates `data/customer.csv.dvc`. Commit code + DVC metafiles with Git:

```bash
git add src/data_ingestion.py data/customer.csv.dvc .gitignore readme.md
git commit -m "Update ingestion and data version metadata"
```

## Push Workflow (Git + DVC)

Push Git-tracked files:

```bash
git push origin main
```

Push DVC-tracked data artifacts:

```bash
dvc push
```

Use both commands so teammates get:
- code and metadata from Git
- actual data files from DVC remote

## Pull Workflow (On Another Machine)

```bash
git clone https://github.com/shivam2003-dev/dvc-practical.git
cd dvc-practical
pip install dvc pandas numpy
```

Fetch data tracked by DVC:

```bash
dvc pull
```

If needed, you can also use:

```bash
dvc fetch
dvc checkout
```

## Useful Validation Commands

Check DVC status:

```bash
dvc status
```

Check Git status:

```bash
git status
```

Preview generated data:

```bash
head -n 5 data/customer.csv
```

## Common Mistakes

- Running only `git push` and forgetting `dvc push`.
- Running `dvc add` but not committing the updated `.dvc` file to Git.
- Using `/tmp/dvc-storage` and losing artifacts after system cleanup.

## DVC Best Practices

- Use a persistent remote (S3/GCS/Azure/SSH/shared storage), not temporary paths like `/tmp`.
- Always run both `git push` and `dvc push` after updating data or model artifacts.
- Commit `.dvc` files (and `dvc.lock`/`dvc.yaml` if using pipelines) in the same Git commit as code changes.
- Keep raw/large artifacts in DVC only; avoid force-adding large data directly to Git.
- Run `dvc status` before commits to confirm metadata and workspace are in sync.
- After switching branches or pulling new commits, run `dvc pull` (or `dvc checkout`) to materialize correct data versions.
- Use tags/releases in Git for milestone experiments so code and data versions are reproducible.
- Store DVC remote credentials securely (environment variables/credential managers), not inside committed files.
- Use `dvc gc` carefully and only when you are sure old cache objects are no longer needed.

## DVC Resources

- AWS S3 Setup Guide (this repo): `docs/dvc-aws-s3-setup.md`
- Official Docs: https://dvc.org/doc
- Get Started Guide: https://dvc.org/doc/start
- Command Reference: https://dvc.org/doc/command-reference
- Pipeline Stages (`dvc repro`, `dvc.yaml`): https://dvc.org/doc/user-guide/project-structure/dvcyaml-files
- Remotes and Storage Backends: https://dvc.org/doc/user-guide/data-management/remote-storage
- DVC + Git Concepts: https://dvc.org/doc/use-cases/versioning-data-and-models
- DVC GitHub Repository: https://github.com/iterative/dvc

## DVC and Git Cheatsheet (Side by Side)

| Goal | Git | DVC |
|---|---|---|
| Initialize project tracking | `git init` | `dvc init` |
| Add remote storage | `git remote add origin <git-url>` | `dvc remote add -d myremote <path-or-url>` |
| Track file changes | `git add <file>` | `dvc add <data-file-or-dir>` |
| See local changes | `git status` | `dvc status` |
| Save snapshot | `git commit -m "message"` | (No direct commit. DVC metadata is committed with Git.) |
| Upload to remote | `git push` | `dvc push` |
| Download from remote | `git pull` | `dvc pull` |
| Download objects only | N/A | `dvc fetch` |
| Materialize files locally | checkout from branch/commit | `dvc checkout` |
| Compare history | `git log` | `dvc diff` |
| Remove tracking | `git rm --cached <file>` | `dvc remove <file>.dvc` |
