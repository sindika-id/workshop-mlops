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
Follow the [Git Installation Guide](Git_Installation_Guide.md) for Windows, macOS, or Linux.  
Verify installation:
```bash
git --version
```

### Step 2 — Initialize a Repository
Create a new folder for your ML project and initialize Git:
```bash
mkdir ml-project
cd ml-project
git init
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

## 4. Tagging Model Versions
Use **tags** to mark important commits, such as when you train a model ready for production.

**Example:**
```bash
# Create a tag
git tag -a v1.0-model -m "First production-ready model"

# Push tag to remote
git push origin v1.0-model
```
**Naming Tips:**
- Use semantic versioning: `v1.0-model`, `v1.1-model`.
- Include experiment purpose if needed: `v2.0-churn-prediction`.

---

## 5. Best Practices for ML Git Workflow
- Commit **code changes** frequently with descriptive messages:
  ```bash
  git commit -m "Add preprocessing script for cleaning null values"
  ```
- Never commit large datasets directly — use DVC or cloud storage.
- Keep feature branches focused on one task (e.g., "feature/add-augmentation").
- Merge into `develop` only after passing tests or review.
- Merge `develop` into `main` only for stable, tested releases.

---

## 6. Quick Command Reference
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
