# 6. Publishing Artifacts and MLflow in CI

## 🎯 Learning Objectives
- Learn how to save and publish trained models and logs as CI artifacts.  
- Understand how to integrate MLflow with CI/CD workflows.  
- Automate model registration and experiment tracking.  

---

## 📘 Why Publish Artifacts?

In CI/CD workflows, artifacts allow you to:
- Store trained models (`.pth`, `.pkl`, `.onnx`) for later use.  
- Save experiment logs, plots, and metrics.  
- Share results with your team directly from the CI system.  

Combined with MLflow, artifacts create a complete **model lineage**:  
data → code → experiment → model → deployment.  

---

## 🛠 Step 1: Uploading Artifacts in GitHub Actions

Example workflow step to upload trained model:

```yaml
- name: Upload trained model
  uses: actions/upload-artifact@v4
  with:
    name: catdog-model
    path: catdog_model.pth
```

Artifacts will be available for download in the GitHub Actions UI.

---

## 🛠 Step 2: MLflow in CI/CD

Run training inside a CI job and log metrics to MLflow:

```yaml
- name: Train model with MLflow
  run: |
    pip install -r requirements.txt
    python train.py
```

Ensure MLflow server is running (local, Docker Compose, or remote).  
Set tracking URI before running training:

```bash
export MLFLOW_TRACKING_URI=http://mlflow-server:5000
```

Or inside Python:
```python
import mlflow
mlflow.set_tracking_uri("http://mlflow-server:5000")
```

---

## 🛠 Step 3: Registering Model in MLflow from CI

After training, register the model automatically:

```python
import mlflow

result = mlflow.register_model(
    "runs:/<RUN_ID>/model",
    "CatDogClassifier"
)
print("Registered model:", result.name, "version:", result.version)
```

This ensures new models are tracked and versioned automatically.

---

## 🛠 Step 4: Example Workflow

```yaml
name: ML Training and Artifacts

on:
  push:
    branches: [ "main" ]

jobs:
  train:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - name: Set up Python
      uses: actions/setup-python@v5
      with:
        python-version: "3.10"

    - name: Install dependencies
      run: |
        pip install -r requirements.txt
        pip install mlflow

    - name: Run training
      env:
        MLFLOW_TRACKING_URI: http://mlflow-server:5000
      run: |
        python train.py

    - name: Upload trained model
      uses: actions/upload-artifact@v4
      with:
        name: catdog-model
        path: catdog_model.pth
```

---

## ✅ Summary
- CI pipelines can **publish trained models as artifacts** for download.  
- MLflow logs all metrics, parameters, and artifacts automatically.  
- You can register models in MLflow directly from CI/CD.  
- This creates a reliable, automated **MLOps workflow**.  