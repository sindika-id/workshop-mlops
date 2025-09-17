# DVC + MinIO (S3-Compatible) — End-to-End Guide
This document provides a **complete, step-by-step** setup for using **DVC** to version your `data/` directory and store artifacts on **MinIO**. It uses your preferred flow:  
`dvc add data` → `git add data.dvc .gitignore`

> Bucket creation is performed **manually via the MinIO Console UI** (no CLI).

---

## 0) Prerequisites
- A Git repo at your project root (e.g., `cat-dog/`).
- Python 3.9+ and `pip` installed.
- **DVC (with S3 support):**
  ```bash
  pip install "dvc[s3]"
  dvc --version
  ```
- **MinIO** running and reachable, for example:
  - Console: `http://165.232.169.40:9000`
  - Example credentials: `minio` / `minio123` (replace with your own)
- (Optional) `.gitignore` already ignoring large/derived artifacts (`.venv/`, `__pycache__/`, etc.).

---

## 1) Manually Create a Bucket in MinIO (Console UI)
1. Open your browser and go to **`http://165.232.169.40:9000`**.
2. Log in with your MinIO **Access Key** and **Secret Key**.
3. From the left sidebar, click **Buckets**.
4. Click **Create Bucket**.
5. Enter a bucket name, e.g. **`dvcstore`**.
6. (Optional) Choose a region; default is fine for most cases.
7. Click **Create**.
8. (Recommended) Open your new bucket → **Settings** → Enable **Versioning**.
9. (Optional) Within the bucket, create a folder **`cat-dog/`** to organize your repo’s data.

> Now you have a bucket `dvcstore` (and optionally a prefix `cat-dog/`).

---

## 2) Initialize DVC in Your Repo
From the repo root:
```bash
dvc init
git add .dvc .gitignore
git commit -m "Initialize DVC"
```

---

## 3) Track the Entire `data/` Directory (Your Preferred Flow)
Make sure your dataset lives under `data/` (e.g., `data/train`, `data/val`, etc.). Then:
```bash
# Create/refresh DVC tracking for the full data directory
dvc add data

# Commit the pointer file + .gitignore update
git add data.dvc .gitignore
git commit -m "Track data directory with DVC"
```
**What happens here?**
- DVC calculates a hash for `data/` and creates **`data.dvc`** at the repo root.
- DVC also updates `.gitignore` so your raw `data/` is ignored by Git.
- Git tracks **only** the small `data.dvc` pointer (not large data files).

> If you previously tracked subfolders separately (e.g., `data/train.dvc`), remove them to avoid duplication:  
> ```bash
> dvc remove data/train.dvc
> git rm data/train.dvc
> git commit -m "Remove per-subfolder tracking; use data.dvc"
> ```

---

## 4) Configure DVC Remote to Use MinIO
We’ll point DVC to your **manually created** MinIO bucket/prefix.

```bash
# Use "-d" to set as the default remote for this repo
dvc remote add -d myremote s3://dvcstore/cat-dog
```

Tell DVC how to reach MinIO (endpoint/region/SSL):
```bash
dvc remote modify myremote endpointurl http://165.232.169.40:9000
dvc remote modify myremote use_ssl false          # set true if using https with a valid cert
dvc remote modify myremote region us-east-1
dvc remote modify myremote addressing_style path  # helps with MinIO
```

Store credentials **locally** (kept out of Git):
```bash
dvc remote modify --local myremote access_key_id minio
dvc remote modify --local myremote secret_access_key minio123
```

Inspect and commit the safe config (no secrets):
```bash
dvc remote list
dvc remote show myremote

git add .dvc/config
git commit -m "Configure DVC remote to MinIO"
```

---

## 5) Push Your Data to MinIO
Upload the tracked data:
```bash
dvc push -v
```
- Data is uploaded to `s3://dvcstore/cat-dog/…` on MinIO.
- You can verify via the **MinIO Console** → your bucket → objects under `cat-dog/`.

> If you see errors like **403/SignatureDoesNotMatch**, double-check endpoint URL, keys, and `addressing_style path`.

---

## 6) Typical Day-to-Day Workflow
When you change files under `data/`:
```bash
# Update the tracked state
dvc add data

# Commit the pointer change (data.dvc)
git add data.dvc
git commit -m "Update data"

# Upload new/changed objects to MinIO
dvc push
```

To fetch data on another machine (or fresh clone):
```bash
git clone <repo-url>
cd <repo>
pip install "dvc[s3]"

# Recreate MinIO credentials locally
dvc remote modify --local myremote access_key_id <your-minio-key>
dvc remote modify --local myremote secret_access_key <your-minio-secret>

# If endpoint differs, set it too
dvc remote modify myremote endpointurl http://<minio-host>:9000
dvc remote modify myremote use_ssl false
dvc remote modify myremote region us-east-1
dvc remote modify myremote addressing_style path

# Pull data from MinIO
dvc pull -v
```

---

## 7) Verifying What’s Tracked
See current DVC-tracked outputs:
```bash
dvc list .
dvc status
```
View the DAG:
```bash
dvc dag
```
Show remote URL and config:
```bash
dvc remote list
dvc remote show myremote
```

---

## 8) Troubleshooting
- **`ERROR: bad DVC file name 'data/train.dvc' is git-ignored.`**  
  You tracked subfolders inside `data/` while `.gitignore` ignores `data/`. Prefer the **single `data.dvc`** approach (this guide), or re-include `!data/*.dvc` in `.gitignore`.
- **403/SignatureDoesNotMatch**  
  Ensure `addressing_style path` is set; confirm endpoint/key/secret. Some MinIO setups require `region us-east-1`.
- **SSL/TLS issues**  
  If using HTTPS with self-signed certs, set `use_ssl true` and export `AWS_CA_BUNDLE` to your CA file. For labs/dev, HTTP is simpler.
- **Bucket not found**  
  Create the bucket first in the **MinIO Console** (Section 1), and use the exact name in `dvc remote add`.
- **Slow push**  
  First push uploads everything; subsequent pushes are incremental. Ensure you’re on a stable network and MinIO has enough I/O.

---

## 9) Clean-Up & Changes
- Stop tracking the `data/` directory via DVC:
  ```bash
  dvc remove data.dvc
  git add -A
  git commit -m "Remove data from DVC"
  # (Optional) delete objects from MinIO via Console if desired
  ```
- Change the remote prefix later:
  ```bash
  dvc remote modify myremote url s3://dvcstore/new-prefix
  ```

---

## 10) Quick Reference (Cheat Sheet)
```bash
# One-time
dvc init
dvc remote add -d myremote s3://dvcstore/cat-dog
dvc remote modify myremote endpointurl http://165.232.169.40:9000
dvc remote modify myremote use_ssl false
dvc remote modify myremote region us-east-1
dvc remote modify myremote addressing_style path
dvc remote modify --local myremote access_key_id <KEY>
dvc remote modify --local myremote secret_access_key <SECRET>

# Daily
dvc add data
git add data.dvc .gitignore
git commit -m "Update data"
dvc push

# New machine
pip install "dvc[s3]"
dvc remote modify --local myremote access_key_id <KEY>
dvc remote modify --local myremote secret_access_key <SECRET>
dvc pull -v
```

---

### Summary
- You **manually created** a MinIO bucket in the Console.
- You tracked your entire `data/` folder with a single `data.dvc` file.
- You configured a DVC remote pointing to MinIO and pushed your dataset.
- Your team can now **`git pull && dvc pull`** to stay in sync without committing large files to Git.