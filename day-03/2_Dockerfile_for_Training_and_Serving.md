# 2) Dockerfile for **Serving** — CatDog FastAPI

## 🎯 Learning Objectives
- Write a **serving** Dockerfile for the CatDog FastAPI inference app.  
- Understand how serving images differ from training images.  
- Build lightweight, reproducible images for deployment (local/CI/CD).  

---

## 📘 Why Separate Training and Serving?
- **Training containers** are heavy: include Jupyter, notebooks, CUDA toolkits, data science libs—optimized for experimentation.  
- **Serving containers** are lean: only runtime + web framework (FastAPI/Uvicorn) and the model artifact.  
- Separation improves build time, security surface, and cold‑start latency.  

---

## 🗂 Reference App (Serving Target)
This guide assumes the single‑file FastAPI service from previous step.

**Project layout**
```
catdog-api/
├─ app.py                 # FastAPI app (serving)
├─ requirements.txt       # (dev) full deps if you prefer; we will use a lighter serving set
├─ requirements-serve.txt # serving-only deps (shown below)
└─ models/
   └─ catdog_model.pth    # trained weights
```

**app.py** exposes:  
- `POST /predict` (multipart upload, field: `file`)  
- `GET /health` (device + label list)  
- Swagger UI at **`/docs`** (auto)  

---

## 🧾 `requirements-serve.txt`
Use a minimal set for inference:
```txt
fastapi>=0.110.0
uvicorn[standard]>=0.29.0
torch>=2.2.0
torchvision>=0.17.0
pillow>=10.0.0
python-multipart
```

> If you target CUDA in production, pin torch/torchvision to versions matching your CUDA base image. For CPU-only, the PyPI wheels are fine.

---

## 🛠 Serving Dockerfile (CPU)
Create **`Dockerfile.serve`** at the repo root:

```dockerfile
# ---------- Base (CPU-only) ----------
FROM python:3.11-slim

# Avoid interactive prompts & reduce image size
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1

# System deps (keep minimal)
RUN apt-get update && apt-get install -y --no-install-recommends \
    curl ca-certificates \
 && rm -rf /var/lib/apt/lists/*

# App directory
WORKDIR /app

# Install serving dependencies first for better layer caching
COPY requirements-serve.txt ./
RUN pip install -r requirements-serve.txt

# Copy only what's needed for inference
COPY app.py ./
COPY models/ ./models/

# Expose HTTP port
EXPOSE 8000

# Healthcheck (optional)
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -fsS http://127.0.0.1:8000/health || exit 1

# Start FastAPI with uvicorn
CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Build
```bash
docker build -f Dockerfile.serve -t catdog-serve:latest .
```
### Run
```bash
docker run --rm -p 8000:8000 catdog-serve:latest
```

### Run (mount model from host, optional)
```bash
# Linux/macOS (bash/zsh)
docker run --rm -p 8000:8000 \
  -v "$(pwd)/models:/app/models" \
  catdog-serve:latest

# Windows PowerShell
docker run --rm -p 8000:8000 \
  -v "${PWD}\models:/app/models" \
  catdog-serve:latest
```

> If `catdog_model.pth` is copied during build, the bind mount is optional. Use the mount when you want to update weights **without** rebuilding the image.

---

## ⚙️ Environment Variables
- `MODEL_PATH` (default: `models/catdog_model.pth`)  
  Example:
  ```bash
  docker run --rm -p 8000:8000 \
    -e MODEL_PATH=/app/models/catdog_model.pth \
    catdog-serve:latest
  ```

---

## 🔍 Verify & Use
**Swagger UI**: http://localhost:8000/docs  
**ReDoc**: http://localhost:8000/redoc  

**Health**
```bash
curl http://localhost:8000/health
```

**Predict**
```bash
curl -X POST "http://localhost:8000/predict" \
  -F "file=@/path/to/image.jpg"
```

---

## 🚀 Optional: Multi‑Stage Serving Image
Further reduce size by compiling wheels in a builder stage and copying only site-packages into the runtime.

```dockerfile
# ---------- Builder ----------
FROM python:3.11-slim AS builder
WORKDIR /wheels
COPY requirements-serve.txt ./
RUN pip wheel --wheel-dir=/wheels -r requirements-serve.txt

# ---------- Runtime ----------
FROM python:3.11-slim
ENV PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1 PIP_NO_CACHE_DIR=1
WORKDIR /app
COPY --from=builder /wheels /wheels
RUN pip install --no-index --find-links=/wheels /wheels/*

COPY app.py ./
COPY models/ ./models/
EXPOSE 8000
CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Build & run**
```bash
docker build -f Dockerfile.serve -t catdog-serve:slim .
docker run --rm -p 8000:8000 catdog-serve:slim
```

---

## ⚡ GPU Serving (NVIDIA, optional)
If you deploy on GPU, prefer an NVIDIA CUDA base (e.g., `nvidia/cuda:12.1.1-cudnn8-runtime-ubuntu22.04`) and install matching `torch` build.

**Example (skeleton)**:
```dockerfile
FROM nvidia/cuda:12.1.1-cudnn8-runtime-ubuntu22.04

RUN apt-get update && apt-get install -y --no-install-recommends \
    python3 python3-pip curl ca-certificates \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY requirements-serve.txt ./
# Pin torch/vision to CUDA build here:
# RUN pip install torch==<cuda-build> torchvision==<cuda-build> -f https://download.pytorch.org/whl/torch_stable.html
RUN pip install -r requirements-serve.txt

COPY app.py ./
COPY models/ ./models/
EXPOSE 8000
CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```
Run with GPU access:
```bash
docker run --rm --gpus all -p 8000:8000 catdog-serve:cuda
```

---

## 🔒 Production Tips
- Run with a **non‑root** user and read‑only rootfs where possible.  
- Add resource limits (CPU/Memory) and liveness/readiness probes in K8s.  
- Pin dependency versions for deterministic builds.  
- Consider `gunicorn` workers if you need pre‑fork scaling, e.g.:  
  ```bash
  gunicorn -w 2 -k uvicorn.workers.UvicornWorker app:app -b 0.0.0.0:8000
  ```

---

## ✅ Summary
- Use a **lean serving Dockerfile** for the FastAPI app.  
- Keep only inference dependencies and the model artifact.  
- Optional multi‑stage builds and GPU variants are provided for size/perf needs.  
- Access Swagger at **`/docs`** and test predictions with `curl`.
