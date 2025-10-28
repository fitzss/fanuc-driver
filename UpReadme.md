# fanuc-driver-configuration
Configuration files of customers where we've deployed the fanuc driver

## FleetGlue Bridge Device Checklist

### 0. Pre-requisites
 1. Purchase: GMKtec Mini PC Intel N5105, 10nm 8GB RAM 128GB SSD ([amazon](https://www.amazon.com/dp/B0B75PT2RY?ref=fed_asin_title&th=1))
 2. The following checklist is Windows specific as that is the default OS of the GMKtec Mini

### 1. Configure Automatic Login while retaining Password Retention
1. Open Registry Editor:
   - Press **Win+R**, type `regedit`, and press **Enter**.

2. Navigate to the PasswordLess registry key:
   ```
   HKEY_LOCAL_MACHINE
     └─ SOFTWARE
         └─ Microsoft
             └─ Windows NT
                 └─ CurrentVersion
                     └─ PasswordLess
                         └─ Device
   ```

3. Configure DevicePasswordLessBuildVersion:
   - Find or create `DevicePasswordLessBuildVersion` (DWORD)
   - Set Value data to `0`
   - If the key doesn't exist:
     - Right-click → New → DWORD (32-bit) Value
     - Name it `DevicePasswordLessBuildVersion`
     - Set Value data to `0`

4. Configure Auto-logon:
   - Reboot the PC
   - Press **Win+R**, type `netplwiz`, and press **Enter** (or type `control userpasswords2` and press **Enter**)
   - The "Users must enter a user name and password to use this computer" checkbox will now be present
   - Uncheck the box
   - Click Apply
   - Enter username and password when prompted
   - Click OK
5. Run ```gpupdate /force```

6. Test the configuration:
   - Restart the device
   - Verify automatic login occurs
   - Confirm the password is still valid when needed

---

### 2. Configure Automatic Power-On
1. Access the BIOS/UEFI settings during boot (usually by pressing F2, F10, or Del)
2. Navigate to Power Management or Advanced settings
3. Look for and enable:
   - "AC Power Recovery" or "Restore on AC Power Loss"
   - "Power On After Power Failure"
4. Save and exit BIOS
5. Test the configuration by:
   - Disconnecting power
   - Reconnecting power
   - Verifying the device boots automatically

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
