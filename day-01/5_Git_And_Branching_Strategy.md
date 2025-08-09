# Git & Branching Strategy for ML Projects

**Purpose:**  
Keep **code**, **data references**, and **models** organized so teams can collaborate effectively, track progress, and reproduce experiments reliably.

---

## 1. Why Git Matters in ML Projects
Machine Learning projects are more complex than standard software projects because they involve:
- **Code** (model training, data processing scripts).
- **Data references** (not storing huge files in Git, but storing pointers/metadata).
- **Models** (binary artifacts that need versioning).
- **Experiments** (different parameter combinations, hyperparameters).

Git allows:
- Collaboration without overwriting each other's work.
- Historical tracking of what was done and when.
- Integration with CI/CD for automation.
- Seamless rollbacks to previous stable states.

---

## 2. Setting Up Git for ML

### Step 1 — Install Git
Follow the [Git Installation Guide](2_Git_Install.md) for Windows, macOS, or Linux.  
Verify installation:
```bash
git --version
```

### Step 2 — Initialize a Repository and Push to GitHub
We'll use a repository name `mlops-workshop-pens` for this workshop.

1. **Create a new folder and initialize Git**
```bash
mkdir mlops-workshop-pens
cd mlops-workshop-pens
git init
```

2. **Create an initial file**
```bash
echo "# MLOps Workshop PENS" > README.md
```

3. **Stage and commit the file**
```bash
git add .
git commit -m "Initial commit for MLOps Workshop PENS"
```

4. **Create a new repository on GitHub**
   - Go to [https://github.com/new](https://github.com/new)
   - Repository name: `mlops-workshop-pens`
   - Keep it **empty** (no README, license, or .gitignore — we already created them locally).
   - Click **Create repository**.

5. **Link local repo to GitHub remote**  
Replace `<your-username>` with your GitHub username:
```bash
git remote add origin https://github.com/<your-username>/mlops-workshop-pens.git
```

6. **Push local repo to GitHub**
```bash
git branch -M main
git push -u origin main
```

### Step 3 — Create a `.gitignore` File
In ML projects, we **must not** commit:
- Large datasets  
- Model binary files (track with DVC instead)  
- Temporary files, caches, or environment files  

Example `.gitignore`:
```gitignore
# Data & model artifacts
data/
models/
*.pt
*.onnx
*.h5

# Virtual environment & cache
.venv/
__pycache__/
*.pyc
.ipynb_checkpoints/

# Secrets
.env

# DVC cache (local only)
.dvc/tmp/
.dvc/cache/
```

---

## 3. Branching Strategy for ML

### Recommended Workflow
- **`main`** → Stable branch, only production-ready code.
- **`develop`** → Integration branch for features ready to test.
- **`feature/<name>`** → Experimental work or new features.

**Flow Example:**
```bash
# Create and switch to main
git checkout -b main

# Create develop branch from main
git checkout -b develop

# Create feature branch from develop
git checkout -b feature/data-cleaning
```

### Branching Diagram
```
 main      ----o----------o--------o
               \                  
 develop         o----o-----o------o
                  \   \
 feature           o   o
 ```
---


## 4. Quick Command Reference
```bash
# Clone a repository
git clone <repo_url>

# Create branch
git checkout -b feature/new-feature

# Switch branch
git checkout develop

# Stage & commit changes
git add .
git commit -m "Your message"

# Push branch to remote
git push origin feature/new-feature

# Tag a release
git tag -a v1.0-model -m "Release model v1.0"
git push origin v1.0-model
```

---

✅ **Outcome After This Tutorial:**  
You will have:
- A Git repo with `.gitignore` ready for ML work.
- A clear branching strategy.
- A tagging system for model versions.
- A team workflow for collaboration and reproducibility.
