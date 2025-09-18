# 4) Serving **from MLflow** — CatDog FastAPI

This guide updates the serving app so it **loads the model from MLflow** (Registry or a specific Run) instead of a local `models/catdog_model.pth` file.

---

## 🎯 What You’ll Achieve
- Configure `MLFLOW_TRACKING_URI` and a **model URI**.  
- Start the FastAPI server that pulls your model from MLflow.  
- Keep the same `/predict`, `/health`, and Swagger `/docs`.  

---

## 🔧 Prerequisites
- An MLflow Tracking Server and Artifact Store (S3/MinIO) reachable from this service.
- Your trained model logged via `mlflow.pytorch.log_model(...)` (as in your training script).

---

## 🧭 Choose a Model URI
Set **one** of the following as `MLFLOW_MODEL_URI`:

- **Registry (recommended)**
  - **Production stage**: `models:/catdog-v1.0/Production`
  - Specific version: `models:/catdog-v1.0/1`
- **Specific Run**
  - `runs:/<RUN_ID>/model` (if you logged with `mlflow.pytorch.log_model(model, "model")`)

Also set `MLFLOW_TRACKING_URI` to your MLflow server, e.g.:  
`http://<MLFLOW_HOST>:5000`

---

## 📦 Requirements (Serving)
Create / update `requirements-serve.txt`:

```txt
fastapi>=0.110.0
uvicorn[standard]>=0.29.0
torch>=2.2.0
torchvision>=0.17.0
pillow>=10.0.0
mlflow>=2.10.0
```

> If your MLflow server uses S3/MinIO, export the usual env vars (`MLFLOW_S3_ENDPOINT_URL`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, etc.).

## Create Alias 
Via UI:
Model Registry → catdog-v1.0 → row “Version 3” → Aliases → Add → type ```cherry```
---

## 🧩 FastAPI App: **app_mlflow.py**

```python
# app_mlflow.py
import io
import os
from typing import Tuple

from fastapi import FastAPI, File, UploadFile, HTTPException
from fastapi.responses import JSONResponse
import uvicorn

import torch
from torch import nn
from torchvision import transforms
from PIL import Image

import mlflow
import mlflow.pytorch

# ---- Config ----
MLFLOW_TRACKING_URI = os.getenv("MLFLOW_TRACKING_URI", "http://localhost:5000")
MLFLOW_MODEL_URI = os.getenv("MLFLOW_MODEL_URI", "models:/catdog-v1.0/cherry")  # e.g., models:/catdog-v1.0/3 or models:/catdog-v1.0/<alias>
LABELS = os.getenv("LABELS", "cat,dog").split(",")
DEVICE = torch.device("cuda" if torch.cuda.is_available() else "cpu")

# ---- Preprocess ----
tf = transforms.Compose([
    transforms.Resize((224, 224)),
    transforms.ToTensor(),
])

def read_image(file_bytes: bytes) -> Image.Image:
    return Image.open(io.BytesIO(file_bytes)).convert("RGB")

def predict_image(img: Image.Image, model: torch.nn.Module) -> Tuple[str, float]:
    x = tf(img).unsqueeze(0).to(DEVICE)
    with torch.no_grad():
        logits = model(x)
        probs = torch.softmax(logits, dim=1)[0]
        idx = int(torch.argmax(probs).item())
        return LABELS[idx], float(probs[idx].item())

app = FastAPI(title="CatDog Inference API (MLflow)", version="1.0.0")

# ---- Load model from MLflow at startup ----
model = None
load_error = None
try:
    if not MLFLOW_MODEL_URI:
        raise RuntimeError("Missing MLFLOW_MODEL_URI env var (e.g., models:/catdog-v1.0/Production or runs:/<run_id>/model)")

    mlflow.set_tracking_uri(MLFLOW_TRACKING_URI)
    # Load the torch.nn.Module from MLflow
    model = mlflow.pytorch.load_model(MLFLOW_MODEL_URI, map_location="cpu")
    model.to(DEVICE).eval()
except Exception as e:
    load_error = str(e)

@app.get("/health")
def health():
    if model is None:
        return JSONResponse(
            status_code=500,
            content={"status": "error", "message": f"Model not loaded: {load_error}", "tracking_uri": MLFLOW_TRACKING_URI}
        )
    return {"status": "ok", "device": str(DEVICE), "labels": LABELS, "tracking_uri": MLFLOW_TRACKING_URI, "model_uri": MLFLOW_MODEL_URI}

@app.post("/predict")
async def predict(file: UploadFile = File(...)):
    if model is None:
        raise HTTPException(status_code=500, detail=f"Model not loaded: {load_error}")

    if not file.filename:
        raise HTTPException(status_code=400, detail="Empty filename.")

    try:
        content = await file.read()
        img = read_image(content)
        label, confidence = predict_image(img, model)
        return {"label": label, "confidence": confidence}
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Inference failed: {e}") from e

if __name__ == "__main__":
    uvicorn.run("app_mlflow:app", host="0.0.0.0", port=int(os.getenv("PORT", 8000)))
```

---

## 🐳 Dockerfile (Serving from MLflow)

Create `Dockerfile.serve-mlflow`:

```dockerfile
FROM python:3.11-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1

WORKDIR /app

# Deps first for better layer caching
COPY requirements-serve.txt ./
RUN apt-get update && apt-get install -y --no-install-recommends curl ca-certificates && \
    rm -rf /var/lib/apt/lists/* && \
    pip install -r requirements-serve.txt

# Copy the MLflow-serving app
COPY app_mlflow.py ./

# Optional: if you still want to ship local label map or assets, copy them here
# COPY models/ ./models/

EXPOSE 8000

# Healthcheck hitting /health
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -fsS http://127.0.0.1:8000/health || exit 1

CMD ["uvicorn", "app_mlflow:app", "--host", "0.0.0.0", "--port", "8000"]
```

---

## ▶️ Run (Local / Docker)

### Local
```bash
export MLFLOW_TRACKING_URI=http://<mlflow-host>:5000
export MLFLOW_MODEL_URI=models:/catdog-v1.0/Production  # or runs:/<run_id>/model
export LABELS=cat,dog

uvicorn app_mlflow:app --host 0.0.0.0 --port 8000 --reload
```

### Docker
```bash
docker build -f Dockerfile.serve-mlflow -t catdog-serve-mlflow:1.0.0 .

docker run --rm -p 8000:8000 \
  -e MLFLOW_TRACKING_URI=http://<mlflow-host>:5000 \
  -e MLFLOW_MODEL_URI=models:/catdog-v1.0/Production \
  -e LABELS=cat,dog \
  -e MLFLOW_S3_ENDPOINT_URL=http://<minio-host>:9000 \
  -e AWS_ACCESS_KEY_ID=<minio-user> \
  -e AWS_SECRET_ACCESS_KEY=<minio-pass> \
  -e AWS_DEFAULT_REGION=us-east-1 \
  catdog-serve-mlflow:1.0.0
```

> **Note**: If your Registry uses stages (`Staging`, `Production`), `models:/name/Production` automatically resolves to the latest version in that stage.

---

## 🔍 Test
- Swagger UI: `http://localhost:8000/docs`  
- Health: `curl http://localhost:8000/health`  
- Predict:
  ```bash
  curl -X POST "http://localhost:8000/predict" -F "file=@/path/to/image.jpg"
  ```

---

## 🛡️ Tips
- If loading fails with S3 errors, verify `MLFLOW_S3_ENDPOINT_URL`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and bucket policy.  
- Pin `mlflow` and `torch` versions that match your training environment to avoid deserialization issues.  
- To switch models at runtime, redeploy with a different `MLFLOW_MODEL_URI` (e.g., promote a new Registry version to `Production`).

---

## ✅ Summary
- Serving now **pulls the model from MLflow** via `MLFLOW_MODEL_URI`.  
- No more baking model weights into the image.  
- Compatible with Registry stages and per-run artifacts.  
- Same API surface (`/predict`, `/health`, Swagger at `/docs`).

