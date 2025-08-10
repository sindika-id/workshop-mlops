# Tracking Datasets & Models with DVC

This guide will walk you step-by-step through integrating **DVC (Data Version Control)** into your `cat-dog` project to track datasets, models, and metrics.

---

## 0) Prerequisites

Before starting, ensure you have:
- A working `cat-dog` project with `data/`, `train.py`, etc.
- Git initialized and connected to GitHub.
- Python virtual environment activated.

Install DVC (with support for popular storage backends):
```bash
pip install "dvc[ssh,gs,s3,gdrive]"
# Or minimal:
# pip install dvc
```

---

## 1) Initialize DVC in the Project

```bash
dvc init
git add .dvc .gitignore
git commit -m "Initialize DVC"
```

**What happens here:**
- `.dvc/` control directory created.
- `.gitignore` updated to exclude large files that will be tracked with DVC.

---

## 2) Track the Dataset with DVC

**Add your dataset to DVC:**
```bash
dvc add data/train data/val
git add data/train.dvc data/val.dvc .gitignore
git commit -m "Track dataset with DVC"
```

**Explanation:**
- Actual dataset remains in your working directory.
- DVC creates `.dvc` pointer files, which are small and safe to commit to Git.

---

## 3) Configure a DVC Remote Storage

### Option A — Local storage (offline/simple):
```bash
mkdir -p ../dvcstore
dvc remote add -d localstore ../dvcstore
git add .dvc/config
git commit -m "Add local DVC remote"
```

### Option B — Cloud storage (for collaboration):

**S3 Example:**
```bash
dvc remote add -d s3store s3://your-bucket/your-prefix
dvc remote modify s3store endpointurl https://s3.your-provider.com
```

**Google Drive Example:**
```bash
dvc remote add -d gdrivestore gdrive://<folder-id>
```

Commit the configuration:
```bash
git add .dvc/config
git commit -m "Configure DVC remote"
```

---

## 4) Push Data to Remote

```bash
dvc push
```

Now the dataset is stored in your remote, keeping your Git repo lightweight.

---

## 5) Create a DVC Pipeline for Training

**Create `dvc.yaml` with a training stage:**
```yaml
stages:
  train:
    cmd: python train.py
    deps:
      - train.py
      - data/train
      - data/val
    outs:
      - models/catdog_model.pth
```

**Commit it:**
```bash
git add dvc.yaml
git commit -m "Add DVC training pipeline"
```

**Run the pipeline:**
```bash
dvc repro
```

**Push the model:**
```bash
dvc push
git add models/.gitignore
git commit -m "Track model with DVC"
```

---

## 6) Track Metrics for Model Comparisons

Modify `train.py` to save metrics:
```python
import json, os
os.makedirs("metrics", exist_ok=True)
with open("metrics/metrics.json", "w") as f:
    json.dump({"val_accuracy": float(correct/total)}, f)
```

Update `dvc.yaml`:
```yaml
stages:
  train:
    cmd: python train.py
    deps:
      - train.py
      - data/train
      - data/val
    outs:
      - models/catdog_model.pth
    metrics:
      - metrics/metrics.json
```

**Re-run and push:**
```bash
dvc repro
dvc push
git add dvc.yaml metrics/metrics.json
git commit -m "Add metrics to DVC pipeline"
```

**View metrics:**
```bash
dvc metrics show
dvc metrics diff
```

---

## 7) Collaboration Workflow

**Author:**
```bash
# Update data
dvc add data/train
git add data/train.dvc
git commit -m "Add more training data"
dvc push

# Retrain model
dvc repro
dvc push
git add metrics/metrics.json
git commit -m "Retrain model with new data"
git push
```

**Teammate:**
```bash
git clone https://github.com/<org>/<repo>.git
cd repo
pip install -r requirements.txt
pip install dvc
dvc pull
```

---

## 8) Versioning Best Practices

- For **new data**, update the folder and run:
```bash
dvc add data/train
git add data/train.dvc
dvc push
```

- For **new models**, update code, run:
```bash
dvc repro
dvc push
```

---

## 9) DVC Experiments

**Run quick experiments without committing to Git:**
```bash
dvc exp run
dvc exp show
```

**Promote winning experiment:**
```bash
dvc exp apply <exp-id>
git commit -am "Promote best experiment"
```

---

## 10) Troubleshooting

- **No remote found:** `dvc remote add -d ...`
- **Teammate missing data:** Run `dvc pull`
- **Large Git repo size:** Ensure big files are tracked by DVC.

---

### ✅ Outcome
- **Datasets** and **models** are stored in DVC remote.
- **Git commits** point to exact dataset/model versions.
- Full **reproducibility** for ML pipelines.
