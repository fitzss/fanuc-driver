# fanuc-driver-configuration
Configuration files of customers where we've deployed the Fanuc driver

## FleetGlue Bridge Device Checklist (GMKtec KB5 / Win11 Pro 24H2)

> Hardware: **GMKtec KB5 (Intel N5105, 8GB/128GB)**  
> OS: **Windows 11 Pro 24H2** (Modern Standby **S0**)

---

### 0) Pre-requisites
- Track each bridge in your **Drivers-Setup-Tracker** (hostname, static IP/subnet/gateway, robot IP, ZeroTier network ID, local admin credentials).
- Put installers on USB/share: **Fanuc driver**, **Fluent Bit**, **Mosquitto**, **ZeroTier**.
- Plan log path: `C:\FleetGlue\logs\`.

---

### 1) Configure automatic login (keep password usable)
1. **Enable the password checkbox**
   - `Win+R → regedit`  
   - Go to  
     `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\PasswordLess\Device`  
   - Create/Set **DWORD** `DevicePasswordLessBuildVersion = 0`  
   - **Reboot**
2. **Turn on auto-logon**
   - `Win+R → netplwiz` (or `control userpasswords2`)  
   - **Uncheck** “Users must enter a user name and password to use this computer.”  
   - Click **Apply**, enter the local admin **username + password**, **OK**
3. **Test**
   - Restart → it should land on desktop automatically (password still works for UAC/RDP)

---

### 2) Configure automatic power-on after power loss (BIOS) + quick test
**Enter BIOS/UEFI**  
- During boot press **Del/F2** (or: **Settings → System → Recovery → Advanced startup → Restart now → Troubleshoot → Advanced options → UEFI Firmware Settings → Restart**)

**Set AC-loss behavior (GMKtec/Aptio BIOS)**
- **Boot → State After G3 (a.k.a. AC Back / Restore on AC Power Loss)** → **S0 / Power On**
- (Optional) **Chipset → Wake on LAN = Enabled** (if present)

**Save & Exit** (F4), then **test**: pull AC for ~5s → plug in → the PC should power on automatically and auto-logon.

---

### 3) Disable sleep, screen blanking, USB suspend & auto-logoff (S0-friendly)

#### 3A) Power Options (Advanced)
- `Win+R → powercfg.cpl` → **Change plan settings** (active plan)  
  - **Turn off the display = Never**  
  - **Put the computer to sleep = Never** → **Save**
- **Change advanced power settings**
  - **Hard disk → Turn off hard disk after = 0 (Never)**
  - **Sleep → Sleep after = 0 (Never)**
  - **Sleep → Hibernate after = 0 (Never)**
  - **Sleep → Allow wake timers = Disabled** *(or “Important wake timers only” if required)*
  - **USB settings → USB selective suspend = Disabled**
  - **PCI Express → Link State Power Management = Off**

> Note: On **Modern Standby (S0)** devices you won’t see **Allow hybrid sleep**. That’s expected—ignore it.

*(Optional, command equivalents — run as Admin PowerShell):*
```powershell
powercfg /hibernate off
powercfg -change -disk-timeout-ac 0
powercfg -change -standby-timeout-ac 0
powercfg -change -monitor-timeout-ac 0
powercfg -setacvalueindex SCHEME_CURRENT SUB_SLEEP AWAKETIMERS 0
powercfg -SetActive SCHEME_CURRENT
