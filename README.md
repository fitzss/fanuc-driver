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

5. Test the configuration:
   - Restart the device
   - Verify automatic login occurs
   - Confirm the password is still valid when needed

### 2. Configure Automatic Power-On
1. Click the Start button.
2. Navigate to Settings > System > Recovery.
3. Next to "Advanced startup," click Restart now.
4. After your PC restarts, select Troubleshoot.
5. Click Advanced options.
6. Select UEFI Firmware Settings.
7. Click Restart to reboot directly into your BIOS/UEFI.
8. Set: Boot → Wake on Power = S0 State
9. Save & Exit (F4).
10. Test: pull power for ~5s → plug back in → it should power itself on.
11. Save and exit BIOS


### 3. Disable Sleep and Auto-Logoff
#### 1. Configure Power Settings:
   - Press Win+R → powercfg.cpl → Enter.
   - Click Change plan settings on the active plan:
     - Turn off the display = Never
     - Put the computer to sleep = Never → Save
   - Click Change advanced power settings and set:
     - Hard disk → Turn off hard disk after = 0 (Never)
     - Sleep → Sleep after = Never (0)
     - Sleep → Hibernate after = Never (0)
     - Sleep → Allow wake timers = Disabled
(or “Important wake timers only” if you truly use scheduled wakes)
     - USB settings → USB selective suspend = Disabled
     - PCI Express → Link State Power Management = Off
   - Note: “Allow hybrid sleep” is not shown on S0 devices—ignore it.

#### 2. Disable Screen Saver:
   - Open Control Panel > Personalization or Type Screen Saver
   - Click "Screen Saver"
   - Set "Screen saver" to "None"
   - Uncheck "On resume, display logon screen"

#### 3. Disable Auto-Logoff:
   - Press **Win+R**, type `regedit`, and press **Enter**.
   - Navigate to: `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System`
   - Create or modify these DWORD values:
     - `InactivityTimeoutSecs` = `0`
     - `InactivityNoCsrss` = `0`
     - `InactivityNoDisk` = `0`

#### 4. Test the Configuration:
   - Leave the device idle for extended period
   - Verify it doesn't go to sleep
   - Verify it doesn't log off
   - Verify display stays on

### 4. Disable Windows Automatic Updates

1. Disable the Windows Update Service (All Editions):
   - Press **Win+R**, type `services.msc`, and press **Enter**.
   - Scroll to **Windows Update**, double-click it.
   - Under **Startup type**, select **Disabled**.
   - Click **Stop** (if running), then **OK**.

2. (Pro/Enterprise) Disable via Group Policy:
   - Press **Win+R**, type `gpedit.msc`, and press **Enter**.
   - Navigate to:  
     `Computer Configuration` → `Administrative Templates` → `Windows Components` → `Windows Update`
   - Double-click **Configure Automatic Updates**.
   - Set to **Disabled**, click **Apply**, then **OK**.
   - (Optional) Adjust **Do not include drivers with Windows Updates** or other scheduling options as needed.

### 5. Network Configuration and Driver Installation
1. Install ZeroTier:
   - Download ZeroTier from [ZeroTier's official website](https://www.zerotier.com/download/)
   - Run the installer
   - Follow the installation wizard

2. Join ZeroTier Network:
   - Open ZeroTier application
   - Click "Join Network"
   - Enter the network ID (contact rishabh@fleetglue.com for the network ID)
   - Wait for network authorization
  
3. Install Fanuc-Driver
   - For detailed installation instructions, refer to the [Fanuc-Driver Windows Installation Guide](https://docs.ladder99.com/en/latest/page/drivers/fanuc/installation-windows.html).
   - > Note: Ignore the direct links to releases in the documentation. Use the latest releases from the respective repositories.

4. Configure Fluent Bit logging service
   - Download fluent bit https://fluentbit.io/download/
   - Unzip it to C:\FluentBit
   - Modify the fluent-bit.conf file similar to what is shown in this repo
   - make sure this is in the parsers.conf file
      [PARSER]
         Name              json
         Format            json
         Time_Keep         On
   - using command prompt, run the following line to run fluent bit
   - <pre lang="en"><code>```cmd sc.exe create FluentBit binPath= "\"C:\Program Files (x86)\fluent-bit\bin\fluent-bit.exe\" -c \"C:\Program Files (x86)\fluent-bit\conf\fluent-bit.conf\"" start= auto ```</code></pre>

  
5. Configure Mosquitto service
   1. Download Mosquitto:
      - Go to [Eclipse Mosquitto Downloads](https://mosquitto.org/download/)
      - Download the Windows installer (e.g., `mosquitto-2.0.15-install-windows-x64.exe`)

   2. Install Mosquitto:
      - Run the installer as administrator
      - Choose "Complete" installation
      - Check "Install as a Windows Service"
      - Complete the installation

   3. Configure Mosquitto:
      - Open Command Prompt as administrator
      - Navigate to Mosquitto installation directory (typically `C:\Program Files\mosquitto`)
      - Locate a file named `mosquitto.conf` with these settings:
      ```
      listener 1883
      allow_anonymous true
      ```
      - Restart the Mosquitto service:
      ```
      net stop mosquitto
      net start mosquitto
      ```

   4. Verify Installation:
      - Open Command Prompt
      - Test the broker:
      ```
      mosquitto_sub -h localhost -t "test" -v
      ```
      - In another Command Prompt:
      ```
      mosquitto_pub -h localhost -t "test" -m "Hello World"
      ```

6. Configure Static IP for Ethernet:
   - Open Network & Internet Settings
   - Click on "Change adapter options"
   - Right-click on Ethernet adapter → Properties
   - Select "Internet Protocol Version 4 (TCP/IPv4)" → Properties
   - Select "Use the following IP address"
   - Enter IP address on the same subnet as the CNC machine
   - Enter Subnet mask (typically 255.255.255.0)
   - Enter Default gateway if required
   - Click OK to save settings

7. Verify Network Configuration:
   - Open Command Prompt
   - Run `ipconfig` to verify static IP settings
   - Ping the CNC machine to verify connectivity
   - Verify ZeroTier connection status

## Important Notes
- Always document the original settings before making changes
- Keep a backup of the current configuration
- Test all changes in a safe environment before deploying to production
- Document any custom settings specific to your deployment
- Ensure the device is connected to a UPS for stable power supply

## Support
For any issues or questions, please contact the development team.
