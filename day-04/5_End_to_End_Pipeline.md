# 5. End-to-End Pipeline

## 🎯 Learning Objectives
- Understand how all MLOps components connect together.  
- Run a full pipeline: dataset → training → experiment tracking → deployment → monitoring.  
- Automate workflows with Docker Compose.  

---

## 📘 Why End-to-End Pipelines?

In production, ML systems are not just about training. Pipelines ensure:  
- **Data Reproducibility** (DVC).  
- **Experiment Tracking** (MLflow).  
- **Continuous Integration** (GitHub Actions).  
- **Deployment & Scaling** (Docker Compose).  
- **Monitoring** (Prometheus + Grafana).  

This gives a complete **MLOps lifecycle**.  

---

## 🛠 Step 1: Workflow Overview

1. **Data Versioning (DVC)**  
   - Track `data/train` and `data/val` with DVC.  
   - Push dataset to remote storage.  

2. **Experiment Tracking (MLflow)**  
   - Train models, log metrics, parameters, and artifacts.  
   - Register models in MLflow registry.  

3. **CI/CD (GitHub Actions)**  
   - Run tests, linting, data validation.  
   - Build and push Docker images for API and training.  

4. **Deployment (Docker Compose)**  
   - Run MLflow server, API, and trainer in containers.  
   - Deploy monitoring stack (Prometheus + Grafana).  

5. **Monitoring & Logging**  
   - Logs from API stored in structured format.  
   - Prometheus scrapes metrics → Grafana dashboards.  

---

## 🛠 Step 2: Running the Pipeline

```bash
# 1. Pull dataset from DVC remote
dvc pull

# 2. Start services
docker compose up -d --build

# 3. Train a model
docker compose run trainer

# 4. Check experiments in MLflow
http://localhost:5000

# 5. Test inference API
http://localhost:8000/docs

# 6. Check metrics in Grafana
http://localhost:3000
```

---

## 🛠 Step 3: Automating Retraining

- Detect new dataset versions (DVC).  
- Trigger retraining workflow (GitHub Actions or cron job).  
- Log new experiments in MLflow.  
- Update API container with new model.  

---

## 🧩 Step 4: Example Pipeline Flow

```text
[DVC Dataset] → [Training in Trainer Container] → [MLflow Logs & Registry] 
    → [Docker Image Build] → [FastAPI Model Service] → [Prometheus/Grafana Monitoring]
```

---

## ✅ Summary
- End-to-end pipelines unify **data, code, training, deployment, and monitoring**.  
- Docker Compose makes it possible to run the full stack locally.  
- CI/CD ensures automation and reproducibility.  
- This setup forms a solid foundation for production-ready MLOps systems.  