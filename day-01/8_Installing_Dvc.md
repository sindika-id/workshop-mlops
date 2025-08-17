# DVC + Google Drive Integration Guide

This document provides a **complete, production‑ready setup** for using **Data Version Control (DVC)** with **Google Drive** as a remote, including options for individual use, team collaboration, and CI automation via a **Service Account**.

---

## 1) Prerequisites

- **Git** initialized in your project.
- **Python 3.8+** and **pip** (or `conda`).
- A **Google account** with access to a Drive **folder** or **Shared Drive**.
- (Recommended) Use a dedicated **Shared Drive** for teams to avoid ownership issues.

---

## 2) Install DVC with Google Drive Support

### Using `pip` (recommended)
```bash
pip install "dvc[gdrive]"
```

![Install DVC with Google Drive Support](images/8_Installing_Dvc/1a.png)

### Using `conda`
```bash
conda install -c conda-forge dvc dvc-gdrive
```

![Install DVC with Google Drive Support](images/8_Installing_Dvc/1b.png)

Verify:
```bash
dvc --version
```
![DVC Version Check](images/8_Installing_Dvc/2.png)

> `dvc[gdrive]` installs the Google Drive backend via `pydrive2`.

---

## 3) Initialize DVC in Your Repository

```bash
git init            # if not already a Git repo
dvc init
git add .dvc .dvcignore
git commit -m "Initialize DVC"
```
![Initialize DVC](images/8_Installing_Dvc/3.png)

---

## 4) Create/Locate a Google Drive Folder

- Create a folder in **My Drive** or in a **Shared Drive** (recommended for teams).

![Create Google Drive Folder](images/8_Installing_Dvc/4a.png)

- Open the folder in the browser and copy the **Folder ID** from the URL:

```
https://drive.google.com/drive/folders/<FOLDER_ID>
```

![Copy Folder ID from URL](images/8_Installing_Dvc/4b.png)

- If you plan to share this DVC remote with team members or use it in CI/CD, right-click on the folder → **Share** → Change from "Restricted" to **"Anyone with the link"** and set permission to **"Editor"**.

![Configure Folder Permissions](images/8_Installing_Dvc/4c.png)

> Keep this `<FOLDER_ID>` handy. 
>
> For personal use only, you can skip this step.

---

## 5) Create OAuth Credentials (Required for Google Drive Access)

To avoid "This app is blocked" errors, you need to create your own OAuth credentials.

### 5a) Create or Select a Google Cloud Project

1. Go to **Google Cloud Console**: https://console.cloud.google.com/

![Go to Google Cloud Console](images/8_Installing_Dvc/5a1.png)

2. Click on the **Select a project button** at the top of the page

![Click Project Dropdown](images/8_Installing_Dvc/5a2.png)

3. Click **"NEW PROJECT"** to create a new project

![Create New Project](images/8_Installing_Dvc/5a3.png)

4. Enter a **project name** (e.g., "DVC-GoogleDrive") and click **"CREATE"**

![Enter Project Name](images/8_Installing_Dvc/5a4.png)

5. Wait for the project to be created, then **select** the newly created project

![Select New Project](images/8_Installing_Dvc/5a5.png)

### 5b) Enable Google Drive API

1. In the **Google Cloud Console**, navigate to **APIs & Services** → **Library**

![Navigate to API Library](images/8_Installing_Dvc/5b1.png)

2. Search for **"Google Drive API"** and Click on **"Google Drive API"** from the results

![Search Google Drive API](images/8_Installing_Dvc/5b2.png)

3. Click **"ENABLE"** to enable the API

![Enable Google Drive API](images/8_Installing_Dvc/5b3.png)

### 5c) Create Desktop App OAuth Credential

1. Navigate to **APIs & Services** → **Credentials**

![Navigate to Credentials](images/8_Installing_Dvc/5c1.png)

2. Click **"+ CREATE CREDENTIALS"** → **"OAuth client ID"**

![Create Credentials](images/8_Installing_Dvc/5c2.png)

3. If prompted, configure the **OAuth consent screen** first by clicking **"CONFIGURE CONSENT SCREEN"**

![Configure Consent Screen](images/8_Installing_Dvc/5c3.png)

4. Click **"GET STARTED"** to begin setting up the consent screen

![Click Get Started](images/8_Installing_Dvc/5c4.png)

5. Fill in the required fields and click **"SAVE AND CONTINUE"**

![Fill Required Fields - Page 1](images/8_Installing_Dvc/5c5.png)

![Fill Required Fields - Page 2](images/8_Installing_Dvc/5c6.png)

![Fill Required Fields - Page 3](images/8_Installing_Dvc/5c7.png)

![Fill Required Fields - Page 4](images/8_Installing_Dvc/5c8.png)

1. Go to **Clients** and click **"+ CREATE CLIENT"**

![Create OAuth Client ID](images/8_Installing_Dvc/5c9.png)

6. Select **Application Type**: Choose **"Desktop app"**

![Select Desktop App](images/8_Installing_Dvc/5c10.png)

8. Give it a name (e.g., "DVC Desktop Auth") and click **"CREATE"**

![Name and Create](images/8_Installing_Dvc/5c11.png)

9. **Copy Your Credentials**: Save the **Client ID** and **Client Secret** that appear

![Copy Credentials](images/8_Installing_Dvc/5c12.png)

### 5d) Add Yourself as Test User

1. Go to **Audience** and scroll down to **"Test users"** section and click **"+ ADD USERS"**

![Go to Audience](images/8_Installing_Dvc/5d1.png)

2. Enter **your Google email address** (same account that owns the Drive folder or you can add team members too) and click **"SAVE"**  

![Enter Email Address](images/8_Installing_Dvc/5d2.png)

---

## 6) Configure the DVC Remote with OAuth Credentials

```bash
# Add and set as default remote
dvc remote add -d gdrive gdrive://<FOLDER_ID>

# Configure OAuth credentials (replace with your actual values)
dvc remote modify gdrive gdrive_client_id YOUR_CLIENT_ID_HERE
dvc remote modify gdrive gdrive_client_secret YOUR_CLIENT_SECRET_HERE

# (Optional) for large/binary files Google may show a "can't scan for viruses" interstitial
dvc remote modify gdrive gdrive_acknowledge_abuse true

# Save remote configuration in project (safe fields only)
git add .dvc/config
git commit -m "Configure DVC remote on Google Drive"
```
![Configure DVC Remote](images/8_Installing_Dvc/6.png)

> **Important**: Replace `YOUR_CLIENT_ID_HERE` and `YOUR_CLIENT_SECRET_HERE` with the actual values from step 5a.
> 
> The OAuth token will be created automatically when you run your **first** `dvc push` or `dvc pull` command. DVC will open a browser window for Google authentication. The token is stored under `~/.config/dvc` (Linux/macOS) or `%APPDATA%\dvc` (Windows). **Do not commit tokens.**

---

## 7) Create Data Folder and Add Sample Data

- Create a directory to hold your dataset

```bash
# Create a data directory
mkdir data
```

![Create Data Folder](images/8_Installing_Dvc/7a.png)

 - Add some sample data (example)

![Add Sample Data](images/8_Installing_Dvc/7b.png)

---

## 8) Track Data with DVC

```bash
# Track the dataset directory with DVC
dvc add data/

# Stage DVC metadata (not the raw data)
git add data.dvc .gitignore
git commit -m "Track dataset with DVC"
```

![Track Data with DVC](images/8_Installing_Dvc/8.png)

---

## 9) Push Data to Google Drive (First-time OAuth)

```bash
# Upload data to Google Drive (will trigger OAuth flow)
dvc push
```

![Push Data to Google Drive](images/8_Installing_Dvc/9a.png)

**First-time authentication process:**

1. Browser will open automatically to Google warning screen, select your account that has been registered as tester

![Google account selection](images/8_Installing_Dvc/9b.png)

2. Click **"Continue"** to go to the next page

![Click Advanced](images/8_Installing_Dvc/9c.png)

3. Click **"Select all"** to grant dvc access to your google drive and click **"Continue"**

![Grant access](images/8_Installing_Dvc/9d.png)

4. There will be a page that's says **"The authentication flow has completed"** which mean that the authentication process is complete and you can close the page window

![Data Upload Success](images/8_Installing_Dvc/9e.png)

5. The data will be automatically uploaded to your google drive in the previous terminal

![Data Upload Success](images/8_Installing_Dvc/9f.png)


On another machine (or team member):
```bash
git clone <repo>
cd <repo>
dvc pull   # will prompt OAuth on first use
```

---

## 10) Team Collaboration Best Practices

- Use a **Shared Drive** so files are not tied to one person's My Drive ownership.
- Grant team members **Contributor** or **Content manager** access to the Drive folder.
- Keep **secrets** (tokens, service-account JSON) out of Git. Use `.gitignore`:

```gitignore
# DVC/Drive credentials
*.json
.dvc/tmp/
.dvc/cache/
**/.dvc/cache/
token.json
*.pem
```

- Project config vs local secrets:
  - Commit: `.dvc/config` (remote URL, flags).
  - Local only: `.dvc/config.local` (private credentials if ever needed).

![Team Collaboration Setup](images/8_Installing_Dvc/10.png)

---

## 11) Non‑Interactive/CI Setup (Service Account)

For CI or headless servers, use a **Google Cloud Service Account**.

> **Note**: Service Accounts require a Google Cloud project with billing enabled. You can use the **Google Cloud Free Trial** (which includes $300 in credits) for testing purposes, or you may need to purchase quota for production use.

### 11.1 Create & Enable
1. In the **Google Cloud Console**, Navigate to **APIs & Services** → **Credentials**

![Navigate to Credentials](images/8_Installing_Dvc/11a.png)

1. Click on **"+ CREATE CREDENTIALS"** and select **"Service account"** 

![Create Service Account](images/8_Installing_Dvc/11b.png)

1. Fill in the service account details and click **"CREATE"**

![Fill Service Account Details - Page 1](images/8_Installing_Dvc/11c.png)

![Fill Service Account Details - Page 2](images/8_Installing_Dvc/11d.png)

![Fill Service Account Details - Page 3](images/8_Installing_Dvc/11e.png)

4. Click on the **email of the created service account**

![Click Service Account Email](images/8_Installing_Dvc/11f.png)

5. Click on the **"Keys"** tab

![Click Keys Tab](images/8_Installing_Dvc/11g.png)

6. Click **"ADD KEY"** → **"Create new key"**

![Add Key - Create New Key](images/8_Installing_Dvc/11h.png)

7. Select **"JSON"** and click **"CREATE"**

![Select JSON and Create](images/8_Installing_Dvc/11i.png)

8. The **JSON key file** will be downloaded automatically. **Save it securely outside your repository folder**

![JSON Key Downloaded](images/8_Installing_Dvc/11j.png)

1. In Google Drive, **share the target folder** (or Shared Drive) with the service account's email (e.g., `my-ci@project.iam.gserviceaccount.com`) with **Editor** access.

![Share Folder with Service Account](images/8_Installing_Dvc/11k.png)


### 11.2 Configure DVC to Use Service Account
On the CI machine (or locally for testing):

**Linux/macOS (Bash):**
```bash
# Place JSON key securely (outside the repo)
export GDRIVE_SERVICE_ACCOUNT_JSON=/secure/keys/sa-drive.json

# Point DVC remote to service account credentials
dvc remote modify gdrive gdrive_use_service_account true
dvc remote modify gdrive gdrive_service_account_json_file_path "$GDRIVE_SERVICE_ACCOUNT_JSON"
```

**Windows (PowerShell):**
```powershell
$env:GDRIVE_SERVICE_ACCOUNT_JSON="C:\secure\keys\sa-drive.json"
dvc remote modify gdrive gdrive_use_service_account true
dvc remote modify gdrive gdrive_service_account_json_file_path "$env:GDRIVE_SERVICE_ACCOUNT_JSON"
```

> **Do not** commit the JSON key. In CI, inject it via secret variables and write it at runtime.

---

## 12) Common Commands

```bash
# Check remote(s)
dvc remote list

# See status vs remote
dvc status -r gdrive

# Push/pull data
dvc push
dvc pull

# Clean unused local cache safely
dvc gc -w   # keep used by current workspace
dvc gc -a   # keep used by all Git commits in repo history
```

---

## 13) Troubleshooting

- **"This app is blocked" error during OAuth**
  - Click **"Advanced"** → **"Go to [app name] (unsafe)"** → **"Allow"**
  - Ensure you've added yourself as a test user in OAuth consent screen
  - Verify you're using the correct Google account that's registered as a test user
- **Client ID/Secret not working**
  - Verify you selected "Desktop app" as application type
  - Double-check you copied the correct Client ID and Client Secret
  - Ensure there are no extra spaces when pasting credentials
- **Service Account authentication failures**
  - Verify the service account email has Editor access to the Google Drive folder
  - Check that the JSON key file path is correct and accessible
  - Ensure the JSON key file is valid and not corrupted
- **Billing/Quota issues with Service Account**
  - Enable billing on your Google Cloud project
  - Use Google Cloud Free Trial for testing ($300 credits)
  - Check quota limits in Google Cloud Console
- **Project selection issues**
  - Make sure you've selected the correct project in Google Cloud Console
  - Verify the project has the Google Drive API enabled
  - Check that you're using the same project for both OAuth and Service Account setup
- **OAuth page not opening on headless server**
  - Use Service Account (Section 11) instead of interactive OAuth.
- **403: insufficient permissions / file not found**
  - Ensure your Google account (or service account email) has access to the Drive folder/Shared Drive.
- **Quota / Rate limits**
  - Reduce concurrency: `DVC_NO_ANALYTICS=1 dvc push -j 2` (lower `-j`).
- **"Sorry, cannot scan this file for viruses" prompt**
  - Ensure `gdrive_acknowledge_abuse` is set to `true`.
- **Moved folder / changed owner**
  - If the Folder ID changes, update the remote URL:  
    `dvc remote modify gdrive url gdrive://<NEW_FOLDER_ID>`
- **Windows path issues**
  - Prefer absolute paths for service account JSON and avoid spaces or wrap paths in quotes.
- **JSON key file download issues**
  - Ensure pop-ups are not blocked in your browser
  - Check your Downloads folder for the automatically downloaded file
  - Verify the file is a valid JSON format and not corrupted

---

## 14) Minimal End‑to‑End Example

```bash
# 1) Setup
pip install "dvc[gdrive]"
git init && dvc init

# 2) Remote
dvc remote add -d gdrive gdrive://<FOLDER_ID>
dvc remote modify gdrive gdrive_acknowledge_abuse true
git add .dvc/config && git commit -m "DVC remote: Google Drive"

# 3) Track data
mkdir -p data && echo "hello" > data/sample.txt
dvc add data/
git add data.dvc .gitignore
git commit -m "Track data with DVC"

# 4) Upload
dvc push
```

On a team member's machine:

```bash
git clone <repo>
cd <repo>
pip install "dvc[gdrive]"
dvc pull
```

---

## 15) Security Notes

- Treat service‑account JSON like a password. Store using secret managers (GitHub Actions, GitLab CI, etc.).
- For extra safety, store only the **remote URL** in Git and keep credentials in `.dvc/config.local` or environment variables.
- Review Drive sharing to avoid unintentionally public data.

---

### References
- DVC Docs: Remotes → Google Drive
- Google Cloud Docs: Service Accounts, Drive API