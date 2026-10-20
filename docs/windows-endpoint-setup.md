# Windows Endpoint Setup: Sysmon & Wazuh Agent

This guide covers preparing your Windows 10/11 virtual machine, installing Microsoft Sysmon for deep operating system telemetry, and deploying the Wazuh Agent to ship logs to your Wazuh Manager.

---

## 1. Network & Prerequisites Check

1. On your Windows VM, open **PowerShell as Administrator**.
2. Verify you have connectivity to your Wazuh server over the Host-Only network:
   ```powershell
   # Ping the Wazuh Server
   ping 192.168.56.10

   # Test that the agent communication port is open and listening
   Test-NetConnection -ComputerName 192.168.56.10 -Port 1514
   ```
   Both checks must succeed before proceeding.

---

## 2. Installing Microsoft Sysinternals Sysmon

### Why Sysmon?
Default Windows Security event logs omit critical details needed by SOC analysts, such as parent process names, hash calculations, and full command-line arguments. **Sysmon (System Monitor)** bridges this gap by recording detailed telemetry into a dedicated Windows Event Log channel.

### Step-by-Step Installation
Run the following commands in an elevated PowerShell session:

```powershell
# 1. Create a temporary working directory
New-Item -ItemType Directory -Force -Path C:\Tools
Set-Location C:\Tools

# 2. Download Sysmon from Microsoft Sysinternals
Invoke-WebRequest -Uri "https://download.sysinternals.com/files/Sysmon.zip" -OutFile "Sysmon.zip"
Expand-Archive -Path "Sysmon.zip" -DestinationPath "C:\Tools\Sysmon" -Force

# 3. Download the high-quality SwiftOnSecurity baseline configuration
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml" -OutFile "C:\Tools\sysmonconfig.xml"

# 4. Install Sysmon as a system service
cd C:\Tools\Sysmon
.\Sysmon64.exe -i C:\Tools\sysmonconfig.xml -accepteula
```

### Verification
1. Verify the service is running:
   ```powershell
   Get-Service Sysmon64
   ```
   *Status should be `Running`.*
2. Check the Windows Event Viewer:
   - Press `Win + R`, type `eventvwr.msc`, and press Enter.
   - Navigate to: **Applications and Services Logs** -> **Microsoft** -> **Windows** -> **Sysmon** -> **Operational**.
   - You should see recent events such as **Event ID 1** (Process Create) and **Event ID 4** (Sysmon service state changed).

---

## 3. Deploying the Wazuh Windows Agent

### Step 1: Download and Silent Install
Run the following PowerShell command to download and install the Wazuh Agent MSI, automatically pointing it to your Wazuh Server (`192.168.56.10`):

```powershell
# Download the Wazuh Windows Agent installer (v4.x)
Invoke-WebRequest -Uri "https://packages.wazuh.com/4.x/windows/wazuh-agent-4.9.0-1.msi" -OutFile "$env:TEMP\wazuh-agent.msi"

# Execute unattended installation and register with manager
Start-Process msiexec.exe -Wait -ArgumentList "/i `"$env:TEMP\wazuh-agent.msi`" /q WAZUH_MANAGER='192.168.56.10' WAZUH_REGISTRATION_SERVER='192.168.56.10' WAZUH_AGENT_NAME='windows-endpoint'"
```

---

## 4. Ingesting Sysmon Logs in Wazuh

By default, the Wazuh Windows agent collects Application, System, and Security event channels. We must explicitly instruct it to monitor the **Sysmon Operational** channel.

### Step 1: Edit `ossec.conf`
1. Open PowerShell as Administrator.
2. Open the configuration file in Notepad:
   ```powershell
   notepad "C:\Program Files (x86)\ossec-agent\ossec.conf"
   ```
3. Scroll to the `<ossec_config>` section where `<localfile>` blocks are defined.
4. Add the following XML block:
   ```xml
   <!-- Ingest Microsoft Sysmon Operational Channel -->
   <localfile>
     <location>Microsoft-Windows-Sysmon/Operational</location>
     <log_format>eventchannel</log_format>
   </localfile>
   ```
5. Save the file (`Ctrl + S`) and close Notepad.

### Step 2: Start the Wazuh Agent Service
```powershell
# Start the Wazuh Agent Service
NET START WazuhSvc

# Or restart if it was already running
Restart-Service WazuhSvc
```

---

## 5. Verification & Validation

### Check Local Agent Connection Log
Inspect the last few lines of the agent log:
```powershell
Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Tail 20
```
Look for confirmation lines:
- `INFO: Requesting a key from server: 192.168.56.10`
- `INFO: Valid key received`
- `INFO: Connected to the server: 192.168.56.10:1514`

### Check Wazuh Dashboard
1. Open your browser to `https://192.168.56.10`.
2. Click the hamburger navigation icon -> **Agents**.
3. You should see `windows-endpoint` listed with a green badge indicating **Active**.
4. Click on the agent to see its operating system details, IP address, and incoming event stream.

---

## Next Step
Proceed to [Troubleshooting](troubleshooting.md) if you encounter connectivity issues, or jump straight to [Windows Event Monitoring Exercises](../exercises/windows-event-monitoring.md).
