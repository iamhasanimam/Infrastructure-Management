# 🛠️ WSUS Setup Guide (Windows Server 2016/2019/2022)

## 1. Prerequisites
- A Windows Server (2016/2019/2022) domain-joined.
- Enough disk space for update storage (50–500 GB+ depending on products).
- Internet access (at least once, to sync with Microsoft).
- Ports open:
  - **8530 (HTTP)**
  - **8531 (HTTPS, if SSL enabled)**

---

## 2. Install the WSUS Role
1. Open **Server Manager → Add Roles and Features**.
2. Select **Windows Server Update Services**.
3. When prompted:
   - Choose **WSUS Services** and **WSUS Management Console**.
   - Choose **WID Database** (Windows Internal Database) or SQL Server.
   - For a lab, **WID** is fine.

---

## 3. WSUS Configuration Wizard
1. Launch **WSUS Configuration Wizard** from Server Manager.
2. Choose **where to store updates**:
   - Example: `D:\WSUS` (avoid C: drive in production).
3. Sync settings:
   - Connect to Microsoft Update (requires internet).
   - Choose **products** (Windows Server, Windows 10/11, Office, etc.).
   - Choose **classifications** (Critical, Security, Updates, Service Packs).
   - Set **sync schedule** (manual or automatic).
4. Perform initial sync → downloads catalog from Microsoft.

---

## 4. Approve Updates
- Go to **WSUS Console → Updates**.
- Approve selected updates for **Test Group** first.
- Later approve for **Production Group**.
- This controls staged rollout.

---

## 5. Configure Group Policy (Clients → WSUS)
1. Open **Group Policy Management** on Domain Controller.
2. Create a GPO (e.g., `WSUS Policy`).
3. Configure:
   ```
   Computer Configuration → Policies → Administrative Templates → Windows Components → Windows Update
   ```
   - **Specify intranet Microsoft update service location:**
     `http://WSUSServerName:8530`
   - **Configure Automatic Updates:**
     - Auto download and notify / schedule install.
   - Optional: **Do not connect to Windows Update internet locations** → Enabled.
4. Link GPO to target OU.

---

## 6. Client Verification
On a client (e.g., RDS02, DC02):
1. Run:
   ```powershell
   gpupdate /force
   gpresult /r
   ```
   → Confirms GPO applied.
2. Force detection/report:
   ```powershell
   wuauclt /detectnow
   wuauclt /reportnow
   UsoClient StartScan
   ```
3. Check Windows Update log:
   ```powershell
   Get-WindowsUpdateLog
   ```
4. In WSUS Console → **Computers**, verify client appears.

---

## 7. Ongoing Management
- Approve updates gradually (Test OU → Production OU).
- Decline superseded/expired updates.
- Run **cleanup wizard** regularly.
- Monitor compliance reports.

---

## ✅ Workflow Recap
1. Install WSUS Role + DB.
2. Configure sync (products, classifications).
3. Approve updates.
4. Apply GPO to clients.
5. Force detection/report.
6. Monitor compliance.

---

## ⚡ Analogy
WSUS is like a **school library**:
- You (Admin) choose which books (updates) come in.
- Students (clients/servers) must use the school library instead of the public library.
- Rulebook (GPO) enforces it.
- Librarian (WSUS Console) tracks who borrowed what.
