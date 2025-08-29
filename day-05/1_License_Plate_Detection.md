# Day 05 — License Plate Detection (RFDETR checkpoint + Faster R-CNN)

## 🎯 Learning objectives
- Train a license-plate detector using an RFDETR checkpoint to initialize a Faster R-CNN backbone.
- Track experiments, metrics and artifacts with MLflow (local or remote tracking server).
- Save and inspect trained model artifacts and logs for later analysis or serving.

---

## 📘 Project overview
This guide explains how to run the license-plate detection training using the existing `train.py` in this repository. The script is expected to implement: COCO-style data loading, a model construction that uses Faster R-CNN, loading of an RFDETR checkpoint into the Faster R-CNN backbone, MLflow logging of parameters/metrics/artifacts, and saving models to a local `models/` directory.

---

## 🔧 Repository layout
```
project-root/
├── data/                # dataset files (images + COCO annotations)
├── models/              # saved checkpoints (.pth)
├── mlruns/              # MLflow local tracking folder (if used)
├── src/                 # train.py and helper modules
├── requirements.txt     # Python dependencies
└── README.md
```

---

## 🏠 Local Environment Setup

### ⚙️ Dependencies
Install required packages in your local Python environment:

```bash
pip install torch torchvision matplotlib tqdm torchmetrics mlflow pandas rfdetr
```

### 📁 Prepare data & checkpoint
1. Prepare your dataset in COCO format (train/val/test). Place images and JSON annotations under `data/` or another path your `train.py` expects.
2. Obtain the RFDETR checkpoint file (e.g., `rf-detr-base.pth`). Place it in a known location and set `PRETRAINED_WEIGHTS` in `train.py` or pass it via an environment variable/CLI argument.

```python
# This imports the RFDETRBase class from the rfdetr package
from rfdetr import RFDETRBase
import torch

# This downloads the RFDETR base model checkpoint
model = RFDETRBase()

# Confirm files exist
import os
print("Data files:", os.listdir('data'))
print("Checkpoint exists:", os.path.exists('rf-detr-base.pth'))
```

Note: after this, set `PRETRAINED_WEIGHTS='rf-detr-base.pth'` in `train.py` or pass it as an argument so the training script loads the checkpoint from the project root.

### ✅ Run checklist (local)
1. Install dependencies and activate the appropriate Python environment.  
2. Place dataset files and RFDETR checkpoint in accessible paths.  
3. Configure `PRETRAINED_WEIGHTS` in `train.py` or pass it as an env var/CLI arg.  
4. (Optional) Set `MLFLOW_TRACKING_URI` to `file:./mlruns` or your remote MLflow server.  
5. Run training: `python train.py`  
6. After training, verify `models/` and `mlruns/` contain expected artifacts — specifically check `mlruns/<experiment_id>/<run_id>/artifacts/models/` for the saved model files.

### 📌 Quick commands (local shell)
```bash
# Install dependencies
pip install -r requirements.txt

# Confirm files
ls -la data
ls -la rf-detr-base.pth

# Run training
python train.py

# Compress artifacts for transfer
zip -r artifacts.zip models mlruns
```

---

## 🧭 Kaggle Environment Setup

### ⚙️ Dependencies
Install required packages in your Kaggle notebook:

```python
!pip install torch torchvision matplotlib tqdm torchmetrics mlflow pandas rfdetr
```

### 📁 Prepare data & checkpoint
1. Prepare your dataset in COCO format (train/val/test). Upload it as a Kaggle dataset and attach it to your notebook.
2. Obtain the RFDETR checkpoint file programmatically in the notebook.

Example: download RFDETR checkpoint in Kaggle notebook:

```python
# This imports the RFDETRBase class from the rfdetr package
from rfdetr import RFDETRBase
import torch

# This downloads the RFDETR base model checkpoint
model = RFDETRBase()

```

Note: after this, set `PRETRAINED_WEIGHTS='/kaggle/working/rf-detr-base.pth'` in `train.py` or pass it as an argument.

### ✅ Run checklist (Kaggle)
1. Install dependencies in the first notebook cell.  
2. Attach your dataset and download/obtain the RFDETR checkpoint.  
3. Configure `PRETRAINED_WEIGHTS` in `train.py` or pass it as an env var/CLI arg.  
4. (Optional) Set `MLFLOW_TRACKING_URI` to `file:/kaggle/working/mlruns` or your remote MLflow server.  
5. Run training: `!python train.py`  
6. After training, verify `/kaggle/working/models/` and `/kaggle/working/mlruns/` contain expected artifacts — specifically check `mlruns/<experiment_id>/<run_id>/artifacts/models/` for the saved model files.

### 📌 Quick commands (Kaggle notebook cells)
```python
# 1) Install dependencies (cell 1)
!pip install torch torchvision matplotlib tqdm torchmetrics mlflow pandas rfdetr

# 2) Download RFDETR checkpoint (cell 2)
from rfdetr import RFDETRBase
import torch
model = RFDETRBase()

# 3) Set environment variables (cell 3)
import os
os.environ['DATA_INPUT_PATH'] = '/kaggle/input/<dataset-name>/data'
os.environ['PRETRAINED_WEIGHTS'] = '/kaggle/working/rf-detr-base.pth'
os.environ['MLFLOW_TRACKING_URI'] = 'file:/kaggle/working/mlruns'

# 4) Write or copy train.py to working directory (cell 4)
%%writefile /kaggle/working/train.py
# (paste your train.py content here)

# 5) Run training (cell 5)
!python /kaggle/working/train.py

# 6) List outputs (cell 6)
!ls -la /kaggle/working/models
!ls -la /kaggle/working/mlruns

# 7) Compress results (models are stored under mlruns/<experiment_id>/<run_id>/artifacts/models/) (cell 7)
!zip -r /kaggle/working/artifacts.zip /kaggle/working/mlruns
```

---

## 🔄 How RFDETR -> Faster R-CNN integration works (high-level)
- RFDETR is a DETR-family model. Some releases include a convolutional backbone (often ResNet) whose weights can be reused.
- Common approach: load the RFDETR checkpoint `state_dict()` and copy matching backbone weights into the Faster R-CNN backbone (`model.backbone.body`).
- Use key filtering and prefix handling to map RFDETR checkpoint keys to the Faster R-CNN backbone keys.

Minimal example (adapt to your checkpoint keys):

```python
import torch
# load checkpoint
ckpt = torch.load('path/to/rf-detr-base.pth', map_location='cpu')
ckpt_sd = ckpt.get('model', ckpt)
# extract backbone weights from ckpt_sd using the checkpoint's key naming
model_backbone_sd = {k.replace('backbone.body.', ''): v for k, v in ckpt_sd.items() if 'backbone' in k}

# build Faster R-CNN without pretrained weights
from torchvision.models.detection import fasterrcnn_resnet50_fpn
model = fasterrcnn_resnet50_fpn(weights=None, pretrained_backbone=False)

backbone_sd = model.backbone.body.state_dict()
for k in list(backbone_sd.keys()):
    if k in model_backbone_sd:
        backbone_sd[k] = model_backbone_sd[k]

model.backbone.body.load_state_dict(backbone_sd)
```

Important: checkpoint key names vary. Inspect `list(ckpt_sd.keys())` and `model.backbone.body.state_dict().keys()` to determine the correct mapping.

---

## ✍️ MLflow usage (local or remote)
- For simple local tracking, point MLflow to a local `mlruns` folder. For remote tracking, set the tracking URI to your MLflow server.

Set tracking URI via environment variable or in code:

```python
import os
# local folder
os.environ['MLFLOW_TRACKING_URI'] = 'file:./mlruns'
# or remote server
# os.environ['MLFLOW_TRACKING_URI'] = 'http://your-mlflow-server:5000'
```

In `train.py` ensure MLflow is used to log params, metrics and artifacts (models, plots). Note: artifacts logged with MLflow are stored inside the run folder: `mlruns/<experiment_id>/<run_id>/artifacts/` (for example `artifacts/models/best_model.pth`). If you plan to register models, configure access to an MLflow Model Registry.

---

## 🧪 Validation & inspection
- Inspect MLflow runs locally: `mlflow ui --backend-store-uri ./mlruns` and open the UI in your browser (if running locally or on a reachable server).
- View artifacts (plots, sample predictions) under `mlruns/<experiment>/<run>/artifacts/`.
- Test the saved `best_model.pth` with a separate inference script to validate predictions.

---

## 🛠 Troubleshooting tips
- Permission errors when MLflow creates directories: ensure `MLFLOW_TRACKING_URI` points to a writable location (e.g., `file:./mlruns`).
- Checkpoint key mismatches: print `list(ckpt_sd.keys())` and `list(model.backbone.body.state_dict().keys())` then map prefixes.  
- Out-of-memory or slow training: reduce `BATCH_SIZE`, use smaller image sizes, or run on a machine with more GPU memory.  
- If model save/load fails, confirm `torch.save` and `torch.load` paths are correct and that the same model architecture is used when loading.

---

## 🔗 References
- RFDETR documentation: https://rfdetr.roboflow.com/ 
- TorchVision detection models: https://pytorch.org/vision/stable/models.html  
- MLflow docs: https://mlflow.org