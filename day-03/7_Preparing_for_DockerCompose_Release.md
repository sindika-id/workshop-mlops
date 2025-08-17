# 7. Preparing for Docker Compose Release

## 🎯 Learning Objectives
- Learn how to orchestrate ML services using Docker Compose.  
- Prepare training, serving, and MLflow services for integration.  
- Define environment variables, volumes, and networks for reproducibility.  

---

## 📘 Why Docker Compose for ML?

- Run **multiple services** (MLflow, training container, serving API, database) together.  
- Simplifies local testing of production-like environments.  
- Provides **reproducible deployments** across teams.  

---

## 🛠 Step 1: Define `.env` File

Centralize configuration in `.env`:

```env
MLFLOW_PORT=5000
API_PORT=8000
POSTGRES_PORT=5432
```

---

## 🛠 Step 2: Example `docker-compose.yml`

```yaml
version: "3.9"

services:
  mlflow:
    image: ghcr.io/mlflow/mlflow:latest
    container_name: mlflow
    ports:
      - "${MLFLOW_PORT}:5000"
    command: mlflow server --backend-store-uri sqlite:///mlflow.db                            --default-artifact-root /mlruns                            --host 0.0.0.0
    volumes:
      - ./mlruns:/mlruns
      - ./mlflow.db:/mlflow.db

  api:
    build:
      context: .
      dockerfile: Dockerfile.serve
    container_name: ml-api
    ports:
      - "${API_PORT}:8000"
    environment:
      - MLFLOW_TRACKING_URI=http://mlflow:5000
    depends_on:
      - mlflow

  trainer:
    build:
      context: .
      dockerfile: Dockerfile.train
    container_name: ml-trainer
    environment:
      - MLFLOW_TRACKING_URI=http://mlflow:5000
    volumes:
      - ./data:/app/data
      - ./outputs:/app/outputs
    depends_on:
      - mlflow
```

---

## 🛠 Step 3: Build & Run

```bash
docker compose up -d --build
```

Check running services:
```bash
docker ps
```

- MLflow UI → http://localhost:${MLFLOW_PORT}  
- API docs (FastAPI) → http://localhost:${API_PORT}/docs  

---

## 🛠 Step 4: Override for Production

Use `docker-compose.override.yml` for production settings:

```yaml
services:
  api:
    restart: always
    environment:
      - LOG_LEVEL=info
```

Deploy:
```bash
docker compose -f docker-compose.yml -f docker-compose.override.yml up -d
```

---

## ✅ Summary
- Use `.env` to centralize configuration.  
- Docker Compose runs **MLflow + Training + Serving API** together.  
- Override files allow different configs for dev/prod.  
- This sets up the foundation for Day-04: full end-to-end pipeline.  