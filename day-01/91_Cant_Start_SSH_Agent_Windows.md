# Windows OpenSSH `ssh-agent` — Troubleshooting Guide

This guide helps you diagnose and fix issues starting the **OpenSSH Authentication Agent** (`ssh-agent`) on Windows.  
Run all commands in **PowerShell (Run as Administrator)** unless stated otherwise.

---

## Prerequisites

- **OpenSSH Client** feature must be installed.
- You need **Administrator** privileges to start/enable services.
- GitHub SSH keys live in: `C:\Users\<YourName>\.ssh\`

Verify OpenSSH Client:
```powershell
Get-WindowsCapability -Online | ? Name -like 'OpenSSH.Client*'
```
If not `Installed`, run:
```powershell
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
# Reboot if prompted
```

---

## 1) Inspect the Service State Precisely

```powershell
Get-CimInstance -ClassName Win32_Service -Filter "Name='ssh-agent'" |
  Format-List Name, State, StartMode, PathName
```
**Interpretation**
- **StartMode**  
  - `Disabled` → Go to **Step 2** to enable.  
  - `Manual` or `Auto` but **State** is `Stopped` → Try starting (Step 2).
- **PathName**  
  - Should be something like: `C:\Windows\System32\OpenSSH\ssh-agent.exe`  
  - If missing/invalid → Reinstall Client (Step 4).

You can also view the basic service object:
```powershell
Get-Service ssh-agent
```

---

## 2) Enable the Service, Then Start It

Set the startup type and start:
```powershell
Set-Service -Name ssh-agent -StartupType Manual   # or Automatic
Start-Service ssh-agent
```

If `Set-Service` fails or `Start-Service` still errors, force via **SC**:
```powershell
sc.exe config ssh-agent start= demand
Start-Service ssh-agent
```
> **Note:** The space after `start=` is required by `sc.exe` syntax.

Confirm it’s running:
```powershell
Get-Service ssh-agent
```

---

## 3) Add Your SSH Key to the Agent

```powershell
ssh-add $env:USERPROFILE\.ssh\id_ed25519
ssh-add -l    # lists loaded keys
```
If you used RSA:
```powershell
ssh-add $env:USERPROFILE\.ssh\id_rsa
```

---

## 4) Reinstall / Repair OpenSSH Client (If Start Still Fails)

Sometimes the Windows capability is corrupted. Reinstall it:

```powershell
Remove-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
Add-WindowsCapability    -Online -Name OpenSSH.Client~~~~0.0.1.0
```

Verify the binary exists:
```powershell
Test-Path "$env:SystemRoot\System32\OpenSSH\ssh-agent.exe"   # should return True
```
Re-apply startup and start:
```powershell
Set-Service ssh-agent -StartupType Manual
Start-Service ssh-agent
```

If `Test-Path` is **False** after reinstall, ensure you’re on a supported Windows build and repeat the reinstall. Consider a system reboot between remove/install actions.

---

## 5) Check the Windows Event Log for Detailed Errors

```powershell
Get-WinEvent -LogName System -MaxEvents 200 |
  Where-Object { $_.Message -like "*ssh-agent*" } |
  Select-Object TimeCreated, Id, LevelDisplayName, Message
```
Look for errors (e.g., missing file, access denied, policy restrictions). These messages are crucial for further diagnosis.

---

## Common Causes & Remedies

### A) **StartMode: Disabled**
Cause: Service disabled by default or group policy.  
Fix: Enable per **Step 2**.

### B) **Access is denied**
Cause: PowerShell not running as Administrator, or corporate policy blocks starting services.  
Fix: Run **PowerShell (Admin)**. If on a managed device, contact IT or use an **alternative** below.

### C) **Binary missing / Path invalid**
Cause: Broken OpenSSH Client install.  
Fix: Reinstall per **Step 4**.

### D) **Conflicting OpenSSH installations**
Cause: 3rd-party OpenSSH installs shadow system binaries.  
Fix: Ensure you use system path first: `C:\Windows\System32\OpenSSH`.  
Check what runs:
```powershell
(Get-Command ssh-agent).Source
```

### E) **Profile or permission issues on .ssh folder**
Ensure your key files exist and are readable:
```powershell
Get-ChildItem $env:USERPROFILE\.ssh -Force
```

---

## Alternatives (If Service Is Not Allowed)

### Option 1 — Use Git Bash’s Agent (Per Session)
Open **Git Bash**:
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
ssh-add -l
```
This runs a user-mode agent for the current shell session only.

### Option 2 — Use SSH Without an Agent (Key File Direct)
Create or edit `C:\Users\<YourName>\.ssh\config`:
```
Host github.com
  HostName github.com
  User git
  IdentityFile C:\Users\<YourName>\.ssh\id_ed25519
  IdentitiesOnly yes
```
This tells SSH to use your key directly—no agent needed.

---

## Appendix: Generate a New SSH Key (Windows)

```powershell
ssh-keygen -t ed25519 -C "your_email@example.com"
# Accept default path: C:\Users\<YourName>\.ssh\id_ed25519
# Optionally set a passphrase
```
Show your public key to copy into GitHub → *Settings → SSH and GPG keys*:
```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub
```

Test GitHub authentication:
```powershell
ssh -T git@github.com
# Expect: "Hi <username>! You've successfully authenticated..."
```

---

