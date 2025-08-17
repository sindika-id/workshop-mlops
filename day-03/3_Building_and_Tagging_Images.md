# 3. Building and Tagging Images

## 🎯 Learning Objectives
- Learn how to build Docker images for ML workflows.  
- Understand best practices for tagging images.  
- Push images to a registry for sharing and deployment.  

---

## 🛠 Step 1: Building Docker Images

From the project root (with a Dockerfile present):

```bash
docker build -t ml-app:latest .
```

- `-t` → assigns a name (tag) to the image.  
- `ml-app` → repository name (local image name).  
- `latest` → tag (version label).  

Check images:
```bash
docker images
```

---

## 🛠 Step 2: Tagging Best Practices

Instead of using only `latest`, tag with **semantic versions or Git commits**.

Examples:
```bash
docker build -t ml-app:1.0.0 .
docker build -t ml-app:1.0.1 .
docker build -t ml-app:$(git rev-parse --short HEAD) .
```

📌 Recommendation:  
- Use **semantic versioning** (1.0.0, 1.1.0, etc.) for releases.  
- Use **Git commit hash** for reproducibility.  
- Keep `latest` as the most recent build.  

---

## 🛠 Step 3: Running Images

Run the image locally:
```bash
docker run --rm -it ml-app:1.0.0
```

Map ports for APIs:
```bash
docker run --rm -it -p 8000:8000 ml-serve:1.0.0
```

Mount volumes for data/models:
```bash
docker run --rm -it -v $(pwd)/data:/app/data ml-train:1.0.0
```

---

## 🛠 Step 4: Pushing to a Registry

### Docker Hub
1. Login:
```bash
docker login
```
2. Tag the image:
```bash
docker tag ml-app:1.0.0 your-dockerhub-username/ml-app:1.0.0
```
3. Push:
```bash
docker push your-dockerhub-username/ml-app:1.0.0
```

### GitHub Container Registry (GHCR)
```bash
docker tag ml-app:1.0.0 ghcr.io/<username>/ml-app:1.0.0
docker push ghcr.io/<username>/ml-app:1.0.0
```

---

## 🧩 Step 5: Automating with CI/CD

In your CI/CD pipeline (e.g., GitHub Actions, GitLab CI):
- Build Docker images automatically on push.  
- Tag with both `latest` and commit hash.  
- Push to Docker Hub or GHCR.  

Example GitHub Actions step:
```yaml
- name: Build and push Docker image
  uses: docker/build-push-action@v5
  with:
    push: true
    tags: |
      ghcr.io/${{ github.repository }}/ml-app:latest
      ghcr.io/${{ github.repository }}/ml-app:${{ github.sha }}
```

---

## ✅ Summary
- Use `docker build` to create images.  
- Always tag images with **version numbers** or **commit hashes**.  
- Push images to a registry for sharing and deployment.  
- Automate builds in CI/CD for consistency.  
