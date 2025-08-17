# 4. CI/CD with GitHub Actions

## 🎯 Learning Objectives
- Understand how Continuous Integration (CI) and Continuous Deployment (CD) apply to ML projects.  
- Learn to set up GitHub Actions for automated testing, building, and deployment.  
- Automate Docker builds and push images to a registry.  

---

## 📘 Why CI/CD for ML?

Machine Learning projects need CI/CD to:  
- Run **tests** automatically (data checks, unit tests, training sanity checks).  
- Ensure **reproducibility** (build Docker images the same way each time).  
- Automate **deployment** of models and APIs.  

---

## 🛠 Step 1: GitHub Actions Basics

Workflows are defined in `.github/workflows/` as YAML files.  

Example folder structure:
```
.github/
└── workflows/
    └── ci-cd.yml
```

---

## 🛠 Step 2: Simple Workflow for ML

```yaml
name: CI/CD for ML

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repo
      uses: actions/checkout@v4

    - name: Set up Python
      uses: actions/setup-python@v5
      with:
        python-version: "3.10"

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt

    - name: Run tests
      run: |
        pytest tests/
```

This workflow installs dependencies and runs unit tests on every push/PR.  

---

## 🛠 Step 3: Build & Push Docker Image

Add a job to build Docker images and push to GitHub Container Registry (GHCR):

```yaml
  docker:
    runs-on: ubuntu-latest
    needs: build

    steps:
    - name: Checkout repo
      uses: actions/checkout@v4

    - name: Log in to GHCR
      uses: docker/login-action@v3
      with:
        registry: ghcr.io
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}

    - name: Build and push Docker image
      uses: docker/build-push-action@v5
      with:
        push: true
        tags: |
          ghcr.io/${{ github.repository }}/ml-app:latest
          ghcr.io/${{ github.repository }}/ml-app:${{ github.sha }}
```

---

## 🛠 Step 4: Adding Model Artifacts

You can also upload MLflow runs, trained models, or logs as artifacts:

```yaml
    - name: Upload artifacts
      uses: actions/upload-artifact@v4
      with:
        name: trained-model
        path: catdog_model.pth
```

This makes your trained model available for download from the GitHub Actions UI.  

---

## 🧩 Step 5: Extending the Workflow

- Add **data validation** (Great Expectations).  
- Run **linting** (flake8, black).  
- Deploy model API automatically with **Docker Compose** on a server.  

---

## ✅ Summary
- GitHub Actions enables automated testing, building, and deployment.  
- You can build & tag Docker images on every push.  
- Upload trained models as artifacts or deploy them automatically.  
