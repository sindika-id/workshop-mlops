# 3) Building and Tagging Images — **Aligned with CatDog Serving**

## 🎯 Learning Objectives
- Build the **serving** image for the CatDog FastAPI app.  
- Apply sensible **tagging** strategies (semantic version + commit SHA).  
- Push images to a container registry (Docker Hub or GHCR).  

> This step aligns with **02-Dockerfile-Serving.md** which defines `Dockerfile.serve`, `app.py`, and `requirements-serve.txt` for the **serving** container.

---

## 🛠 Step 1: Build the Serving Image

From the project root (containing `Dockerfile.serve` and `app.py`):

```bash
# Build a versioned image
docker build -f Dockerfile.serve -t catdog-serve:1.0.0 .
```

Verify images:
```bash
docker images | grep catdog-serve
```

---

## 🛠 Step 2: Tagging Best Practices

Use both **semantic versions** and **Git commit hashes**; keep `latest` for the most recent stable build.

Examples:
```bash
# Semantic version
docker build -f Dockerfile.serve -t catdog-serve:1.0.0 .

# Latest (rolling)
docker tag catdog-serve:1.0.0 catdog-serve:latest

# Git commit short SHA
docker tag catdog-serve:1.0.0 catdog-serve:$(git rev-parse --short HEAD)
```

**Recommendation**
- Releases: `1.0.0`, `1.1.0`, …  
- CI builds: tag with `${GIT_SHA}` for reproducibility.  
- Keep `latest` pointing to the most recent stable release.  

---

## 🛠 Step 3: Run the Image Locally

### 3.1 Basic run (model **baked into image**)
```bash
docker run --rm -p 8000:8000 catdog-serve:1.0.0
```

### 3.2 Run with model **mounted** (update weights without rebuild)
**Bash (Linux/macOS):**
```bash
docker run --rm -p 8000:8000 \
  -v "$(pwd)/models:/app/models" \
  catdog-serve:1.0.0
```

**PowerShell (Windows):**
```powershell
docker run --rm -p 8000:8000 \
  -v "${PWD}\models:/app\models" \
  catdog-serve:1.0.0
```

### 3.3 Override model path
```bash
docker run --rm -p 8000:8000 \
  -e MODEL_PATH=/app/models/catdog_model.pth \
  catdog-serve:1.0.0
```

> Swagger UI: http://localhost:8000/docs  
> Health: `curl http://localhost:8000/health`  
> Predict: `curl -X POST "http://localhost:8000/predict" -F "file=@/path/to/image.jpg"`

---

## 🛠 Step 4: Push to a Registry

### 4.1 Docker Hub

1) **Login**
```bash
docker login
```

2) **Tag for Docker Hub namespace**
```bash
# Replace YOUR_DH_USERNAME with your Docker Hub username
docker tag catdog-serve:1.0.0 YOUR_DH_USERNAME/catdog-serve:1.0.0
docker tag catdog-serve:latest YOUR_DH_USERNAME/catdog-serve:latest
```

3) **Push**
```bash
docker push YOUR_DH_USERNAME/catdog-serve:1.0.0
docker push YOUR_DH_USERNAME/catdog-serve:latest
```

> Troubleshooting TLS handshake or Docker Desktop issues: restart Docker Desktop and retry `docker login`. On WSL2 hosts: `wsl --shutdown` then reopen Docker Desktop.

---

### 4.2 GitHub Container Registry (GHCR)

1) **Login with PAT**
```bash
# Replace <username> with your GitHub username
docker login ghcr.io -u <username>
# Use a Personal Access Token (classic) with: read:packages, write:packages, delete:packages
```

2) **Tag for GHCR**
```bash
# Replace <username>/<repo> accordingly
docker tag catdog-serve:1.0.0 ghcr.io/<username>/<repo>/catdog-serve:1.0.0
docker tag catdog-serve:latest ghcr.io/<username>/<repo>/catdog-serve:latest
```

3) **Push**
```bash
docker push ghcr.io/<username>/<repo>/catdog-serve:1.0.0
docker push ghcr.io/<username>/<repo>/catdog-serve:latest
```

---

## 🧩 Step 5: Automate with CI/CD

Build and push on every commit; tag with both `latest` and `${{ github.sha }}` (or GitLab `$CI_COMMIT_SHA`).

**GitHub Actions example**
```yaml
name: Build & Push CatDog Serve

on:
  push:
    branches: [ main ]

jobs:
  docker:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to GHCR
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and Push
        uses: docker/build-push-action@v5
        with:
          context: .
          file: Dockerfile.serve
          push: true
          tags: |
            ghcr.io/${{ github.repository }}/catdog-serve:latest
            ghcr.io/${{ github.repository }}/catdog-serve:${{ github.sha }}
```

> For Docker Hub, replace the `registry` and `tags` accordingly and create `DOCKERHUB_USERNAME`/`DOCKERHUB_TOKEN` secrets.

---

## ✅ Summary
- Build the serving image using `Dockerfile.serve` → `catdog-serve:<version>`.  
- Tag with semantic versions and commit SHAs; keep `latest` for stable.  
- Push to Docker Hub or GHCR; automate via CI/CD.  
- Run locally with port mapping `-p 8000:8000`, optional model mount, or `MODEL_PATH` override.
