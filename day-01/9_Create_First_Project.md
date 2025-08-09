# New Python Project: Bootstrap & GitHub Push Guide

A practical, end‑to‑end checklist to create a clean Python project, initialize Git, and push to GitHub. Includes commands for **Windows (PowerShell)** and **macOS/Linux (bash)**.

---

## ✅ High‑Level Checklist

- [ ] Create project folder
- [ ] Initialize Git
- [ ] Create and activate Python virtual environment
- [ ] Create `README.md`
- [ ] Add `.gitignore` (Python template)
- [ ] Create `requirements.txt`
- [ ] (Optional) Add `requirements-dev.txt`
- [ ] (Optional) Initialize DVC (for data projects)
- [ ] First commit
- [ ] Create empty GitHub repository
- [ ] Add remote & push
- [ ] Protect secrets (`.env`, tokens) via `.gitignore`
- [ ] (Optional) Add pre-commit hooks (format/lint)
- [ ] (Optional) Set up CI (GitHub Actions)

---

## 1) Create Project Folder

```bash
# macOS/Linux
mkdir cat-dog && cd cat-dog
```
```powershell
# Windows PowerShell
mkdir cat-dog; cd cat-dog
```

---

## 2) Initialize Git

```bash
git init
git config user.name  "Your Name"
git config user.email "you@example.com"
```

> If you use SSH for GitHub, ensure your key is loaded (`ssh -T git@github.com`).

---

## 3) Create & Activate Virtual Environment

**macOS/Linux**
```bash
python3 -m venv .venv
source .venv/bin/activate
python -V
pip -V
```

**Windows (PowerShell)**
```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -V
pip -V
```

> To deactivate later: `deactivate`

---

## 4) Create Starter Files

```bash
# macOS/Linux
touch README.md requirements.txt
mkdir -p src/cat-dog tests
```
```powershell
# Windows
ni README.md -ItemType File; ni requirements.txt -ItemType File
mkdir src\cat-dog, tests
```

Minimal **README.md**
```markdown
# cat-dog

Short description.
```

## Setup
```bash
python -m venv .venv
source .venv/bin/activate   # Windows: .\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

---

## 5) Add `.gitignore` (Python)

Create `.gitignore` with:

```
# Virtual env
.venv/
env/
venv/

# Byte-compiled / cache
__pycache__/
*.py[cod]
*.pyo

# Distribution / packaging
build/
dist/
*.egg-info/

# Tools
.pytest_cache/
.coverage
htmlcov/
.mypy_cache/
.ipynb_checkpoints/

# OS
.DS_Store
Thumbs.db

# Secrets
.env
*.env
```
Create file quickly:
```bash
# macOS/Linux
cat > .gitignore <<'EOF'
# (paste the block above)
EOF
```
```powershell
# Windows
@'
# (paste the block above)
'@ | Out-File -Encoding utf8 .gitignore
```

---

## 6) Pin Dependencies

Edit `requirements.txt` (example):
```
numpy~=1.26
pandas~=2.2
```

(Optional) `requirements-dev.txt`:
```
black~=24.3
ruff~=0.5
pytest~=8.2
pre-commit~=3.7
```

Install:
```bash
pip install -r requirements.txt
# Optional dev tools:
pip install -r requirements-dev.txt
```

Freeze (optional):
```bash
pip freeze > requirements.lock.txt
```

---

## 7) (Optional) Pre-commit Hooks (format/lint)

```bash
pre-commit install
```

`.pre-commit-config.yaml`:
```yaml
repos:
  - repo: https://github.com/psf/black
    rev: 24.3.0
    hooks: [ {id: black} ]
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.5.0
    hooks: [ {id: ruff} ]
```

---

## 8) Initialize DVC (Data Projects)

```bash
pip install "dvc[gdrive]"    # choose remote as needed
dvc init
git add .dvc .dvcignore
git commit -m "Initialize DVC"
```

---

## 9) First Commit

```bash
git add .
git commit -m "chore: bootstrap project structure"
```

---

## 10) Create a New GitHub Repository

1. Go to **https://github.com/new**
2. **Repository name**: `cat-dog`
3. Choose **Private** or **Public**
4. **Do not** initialize with README/.gitignore/license (we already have them)
5. Click **Create repository**

GitHub will show the remote URL, e.g.:
- SSH: `git@github.com:yourname/cat-dog.git`
- HTTPS: `https://github.com/yourname/cat-dog.git`

---

## 11) Add Remote & Push

**SSH**
```bash
git remote add origin git@github.com:yourname/cat-dog.git
git branch -M main
git push -u origin main
```

**HTTPS**
```bash
git remote add origin https://github.com/yourname/cat-dog.git
git branch -M main
git push -u origin main
```

---

## 12) Project Skeleton (Suggested)

```
cat-dog/
├─ .gitignore
├─ README.md
├─ requirements.txt
├─ requirements-dev.txt        # optional
├─ .pre-commit-config.yaml     # optional
├─ src/
│  └─ cat-dog/
│     └─ __init__.py
├─ tests/
│  └─ test_smoke.py
└─ .venv/                      # created locally, not committed
```

Create a smoke test (optional):
```python
# tests/test_smoke.py
def test_truth():
    assert True
```

Run:
```bash
pytest -q
```

---

## 13) Quick “All-in-One” Script (bash)

```bash
# Adjust repo/user names before running
set -e
PROJECT=cat-dog
USER_GH=yourname

mkdir "$PROJECT" && cd "$PROJECT"
git init
python3 -m venv .venv && source .venv/bin/activate
printf "# %s

Short description.
" "$PROJECT" > README.md
printf "numpy~=1.26
pandas~=2.2
" > requirements.txt

cat > .gitignore <<'EOF'
.venv/
__pycache__/
*.py[cod]
build/
dist/
*.egg-info/
.pytest_cache/
.coverage
htmlcov/
.mypy_cache/
.ipynb_checkpoints/
.DS_Store
Thumbs.db
.env
*.env
EOF

mkdir -p src/$PROJECT tests && touch src/$PROJECT/__init__.py tests/test_smoke.py
git add . && git commit -m "chore: bootstrap project"
git branch -M main
git remote add origin git@github.com:$USER_GH/$PROJECT.git
git push -u origin main
```

---

## 14) Final Review Checklist (Before Sharing)

- [ ] `README.md` describes setup and how to run
- [ ] Virtual env is excluded by `.gitignore`
- [ ] Secrets are not committed
- [ ] `requirements*.txt` install successfully in a clean env
- [ ] Tests (if any) pass locally
- [ ] Repo pushed to GitHub and visible

---

### Notes
- For corporate/lab PCs, ensure **Git user/email** match your org policy.
- Prefer **SSH** for GitHub to avoid repeated credential prompts.
- Consider enabling **branch protection** and **required status checks** later.
