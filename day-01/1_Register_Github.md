# Register a GitHub Account — Step-by-Step Guide

> This Markdown file walks you through creating a GitHub account and completing essential setup so you can push code securely from your machine.

---

## 1) Create Your GitHub Account

1. Open `https://github.com/` and click **Sign up**.
   
   <img src="images/1_Register_Github/1_Sign_Up_Github/1.SignUp.png" alt="Sign Up" width="600">

2. Choose Sign up, then click **Continue with Google**.
   
   <img src="images/1_Register_Github/1_Sign_Up_Github/2.Register.png" alt="Click Sign Up" width="600">

3. Choose your **email address**.
   
   <img src="images/1_Register_Github/1_Sign_Up_Github/3.ChooseEmail.png" alt="Choose Email" width="600">

4. Click **Continue** to grant email access.
   
   <img src="images/1_Register_Github/1_Sign_Up_Github/4.Continue.png" alt="Grant Email" width="600">

5. The form will be auto-filled with your email, username, and country. You can change the username and country, but the email address cannot be changed.
   
   <img src="images/1_Register_Github/1_Sign_Up_Github/5.InsertEmail-Username-Country.png" alt="Auto Fill" width="600">

6. Click **Create account** and wait for verification to complete.
7. You will be redirected to the GitHub homepage.
   
   <img src="images/1_Register_Github/1_Sign_Up_Github/6.HomePageGitHub.png" alt="Account Verification" width="600">

---

## 2) Download Google Authenticator on Your Phone

### Scan to get the code

1. Download **Google Authenticator** from the Play Store.
   
   <img src="images/1_Register_Github/2_Download_Authenticator/3.0.jpeg" alt="Download Authenticator" width="200">

2. Open Google Authenticator.
   
   <img src="images/1_Register_Github/2_Download_Authenticator/3.1.jpeg" alt="Open Authenticator" width="200">

3. Tap **Add a code**.
   
   <img src="images/1_Register_Github/2_Download_Authenticator/3.2.jpeg" alt="Add a code" width="200">

4. Tap **Scan a QR code**.
   
   <img src="images/1_Register_Github/2_Download_Authenticator/3.3.jpeg" alt="Scan QR code" width="200">

## 3) Secure Your Account (Recommended)

### Enable Two-Factor Authentication (2FA)

1. In GitHub, click your avatar → **Settings** → **Password and authentication**.
   
   <img src="images/1_Register_Github/3_Enable_Two_Factor_Authentication/1.Setting.png" alt="Settings" width="600">

2. Under **Two-factor authentication**, click **Enable two-factor authentication**.
    
    <img src="images/1_Register_Github/3_Enable_Two_Factor_Authentication/2.EnableTwoFactorAuthentication.png" alt="Enable 2FA" width="600">

3. Open the Authenticator app, then tap **Scan a QR code**.
    
    <img src="images/1_Register_Github/3_Enable_Two_Factor_Authentication/3.3.jpeg" alt="Authenticator scan" width="600">

4. Scan the QR code.
   
   <img src="images/1_Register_Github/3_Enable_Two_Factor_Authentication/3.3.jpeg" alt="Scan QR" width="600">

5. Enter the code into the form.
   
   <img src="images/1_Register_Github/3_Enable_Two_Factor_Authentication/3.5.jpeg" alt="Enter code" width="200">

6. Receive and save your recovery codes.
   
   <img src="images/1_Register_Github/3_Enable_Two_Factor_Authentication/4.authenticated.png" alt="Recovery codes" width="600">

7. Click **Download recovery codes**.
   
   <img src="images/1_Register_Github/3_Enable_Two_Factor_Authentication/5.SaveRecoveryCode.png" alt="Download recovery codes" width="600">

8. Two-factor authentication is now enabled.
   
   <img src="images/1_Register_Github/3_Enable_Two_Factor_Authentication/6.DoneSaveSetTwoFactorAuthentication.png" alt="2FA enabled" width="600">

---

## 4) Install Git Locally

### Windows

1. Download **Git for Windows** from `https://git-scm.com/download/win`.
   
   <img src="images/1_Register_Github/4_Install_Git_Locally/1.0DownloadGit.png" alt="Download Git for Windows" width="600">

2. Run the installer and follow the prompts.
   
   <img src="images/1_Register_Github/4_Install_Git_Locally/1.1.InstallGit.png" alt="Run installer" width="600">

3. If Windows warns about the installer, click **More info** and then **Run anyway**.
   
   <img src="images/1_Register_Github/4_Install_Git_Locally/2.WindowsProtected.png" alt="Windows protected" width="400">

4. Continue through the installation steps.
   
   <img src="images/1_Register_Github/4_Install_Git_Locally/3.MoreInfo.png" alt="Continue installation" width="400">

5. Click **Install**.
   
   <img src="images/1_Register_Github/4_Install_Git_Locally/4.InstallGit.png" alt="Install Git" width="400">

6. Click **Finish** when installation completes.
   
   <img src="images/1_Register_Github/4_Install_Git_Locally/5.FinishInstsall.png" alt="Finish install" width="400">

7. Verify Git from Command Prompt:
   ```powershell
   git --version
   ```
   
   <img src="images/1_Register_Github/4_Install_Git_Locally/6.CheckGitFromCMD.png" alt="Check git" width="600">

### macOS

Install the Xcode command line tools:
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

<img src="images/1_Register_Github/5_Configure_Git/1.ConfigureGit.png" alt="Configure Git" width="600">

> Use the same email you verified on GitHub so your commits are linked to your account.

---

## 6) Connect to GitHub via SSH (Recommended)

### 6.1 Check Existing Keys

**Windows (PowerShell)**

```powershell
Get-ChildItem $env:USERPROFILE\.ssh
```

<img src="images/1_Register_Github/6_Connect_Github_With_SSH/1.png" alt="List SSH keys" width="600">

**macOS / Linux (Terminal)**

```bash
ls -al ~/.ssh
```

---

### 6.2 Generate a New SSH Key

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

<img src="images/1_Register_Github/6_Connect_Github_With_SSH/2.png" alt="Generate SSH key" width="600">

---

### 6.3 Start the SSH Agent & Add Your Key

**macOS / Linux**

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

**Windows (PowerShell)** — run PowerShell as Administrator:
```powershell
Start-Service ssh-agent
ssh-add $env:USERPROFILE\.ssh\id_ed25519
```

Steps to enable ssh-agent on Windows:

1. Open PowerShell as Administrator.
   
   <img src="images/1_Register_Github/6_Connect_Github_With_SSH/3.png" alt="Start ssh-agent" width="600">

2. Start the agent:
```powershell
Start-Service ssh-agent
```
   
   <img src="images/1_Register_Github/6_Connect_Github_With_SSH/5.png" alt="Start service" width="600">

3. Check the service status:
```powershell
Get-Service ssh-agent
```
   
   <img src="images/1_Register_Github/6_Connect_Github_With_SSH/6.png" alt="Service status" width="600">

If the service exists but won't start:
```powershell
Set-Service ssh-agent -StartupType Automatic
Start-Service ssh-agent
```
   
   <img src="images/1_Register_Github/6_Connect_Github_With_SSH/7.png" alt="Enable service" width="600">

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

   <img src="images/1_Register_Github/6_Connect_Github_With_SSH/4.png" alt="Public key" width="600">

2. In GitHub: **Settings** → **SSH and GPG keys** → **New SSH key**.

   <img src="images/1_Register_Github/6_Connect_Github_With_SSH/8.png" alt="New SSH key" width="600">
   <img src="images/1_Register_Github/6_Connect_Github_With_SSH/9.png" alt="Add SSH key" width="600">

3. Copy the content of your public key file (e.g., `id_ed25519.pub`).

   <img src="images/1_Register_Github/6_Connect_Github_With_SSH/11.png" alt="Copy public key" width="600">

4. Paste the key, give it a descriptive title (e.g., "Laptop-ED25519"), and click **Add SSH key**.

   <img src="images/1_Register_Github/6_Connect_Github_With_SSH/12.png" alt="Paste SSH key" width="600">
   <img src="images/1_Register_Github/6_Connect_Github_With_SSH/13.png" alt="Add SSH key" width="600">

---

### 6.5 Test the Connection

```bash
ssh -T git@github.com
```

<img src="images/1_Register_Github/6_Connect_Github_With_SSH/14.png" alt="Test SSH connection" width="600">

---

## 7) (Alternative) Use HTTPS with a Personal Access Token (PAT)

If you prefer HTTPS:

1. GitHub → **Settings** → **Developer settings** → **Personal access tokens**.

   <img src="images/1_Register_Github/7_Connect_Github_With_HTTPS/1.png" alt="Create PAT" width="600">

2. Create a **fine-grained token**, select the repository scope, and grant **Contents: Read and write**.

   <img src="images/1_Register_Github/7_Connect_Github_With_HTTPS/3.png" alt="Token scope" width="600">
   <img src="images/1_Register_Github/7_Connect_Github_With_HTTPS/4.png" alt="Token creation" width="600">

3. Use the token as the **password** when Git prompts during `git push`.

---

## Troubleshooting

- **Permission denied (publickey)** → Ensure your SSH key is added to GitHub.
- **Username/password prompts for HTTPS** → Use a **Personal Access Token**.
- **Email not linked to commits** → Check `git config user.email`.

---

**You are now registered and ready to use GitHub.**
