# Register a GitHub Account — Step-by-Step Guide

> This Markdown file walks you through creating a GitHub account and completing essential setup so you can push code securely from your machine.

---

## 1) Create Your GitHub Account

1. Open `https://github.com/` and click **Sign up**.
2. Enter your **email address** and confirm the verification code sent to your inbox.
3. Set a **strong password**.
4. Choose a unique **username**.
5. Select the **Free** plan
6. Complete onboarding preferences .

---

## 2) Verify Your Email

- Check your email for a message from GitHub and click **Verify email address**.  
  *You must verify your email to create repositories and receive notifications.*

---

## 3) Secure Your Account (Recommended)

### Enable Two-Factor Authentication (2FA)
1. In GitHub, click your avatar → **Settings** → **Password and authentication**.
2. Under **Two-factor authentication**, click **Enable two-factor authentication**.
3. Choose **Authenticator app**.
4. Save your **Recovery codes** in a secure place.

---

## 4) Install Git Locally

### Windows
- Install **Git for Windows** from `https://git-scm.com/download/win`.
- During setup, accept defaults (Git Credential Manager included).

### macOS
```bash
xcode-select --install
```
or install via Homebrew:
```bash
brew install git
```

### Linux (Debian/Ubuntu)
```bash
sudo apt update && sudo apt install -y git
```

---

## 5) Configure Git (Identity & Defaults)

```bash
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
git config --global init.defaultBranch main
```

> Use the same email you verified on GitHub to associate commits with your account.

---


## 6) Connect to GitHub via SSH (Recommended)

### 6.1 Check Existing Keys

**macOS / Linux (Terminal)**
```bash
ls -al ~/.ssh
```

**Windows (PowerShell)** 

```powershell
Get-ChildItem $env:USERPROFILE\.ssh
```

---

### 6.2 Generate a New SSH Key
```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

---

### 6.3 Start the SSH Agent & Add Your Key

**macOS / Linux**
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

**Windows (PowerShell)**

Run as Administrator
```powershell
Start-Service ssh-agent
ssh-add $env:USERPROFILE\.ssh\id_ed25519
```

---

### 6.4 Add the Public Key to GitHub
1. Copy your public key:

   **macOS / Linux**
   ```bash
   cat ~/.ssh/id_ed25519.pub
   ```

   **Windows (PowerShell)**
   ```powershell
   type $env:USERPROFILE\.ssh\id_ed25519.pub
   ```

2. In GitHub: **Settings** → **SSH and GPG keys** → **New SSH key**.
3. Paste the key, name it (e.g., “Laptop-ED25519”), and save.

---

### 6.5 Test the Connection
```bash
ssh -T git@github.com
```

---

## 7) (Alternative) Use HTTPS with a Personal Access Token (PAT)

If you prefer HTTPS:

1. GitHub → **Settings** → **Developer settings** → **Personal access tokens**.
2. Create a **Fine-grained token**, select your repo scope, grant **Contents: Read and write**.
3. Use the token as the **password** when Git prompts during `git push`.

---

## Troubleshooting

- **Permission denied (publickey)** → Ensure SSH key is added to GitHub.
- **Username/Password prompts for HTTPS** → Use a **Personal Access Token**.
- **Email not linked to commits** → Check `git config user.email`.

---

**You are now registered and ready to use GitHub.**
