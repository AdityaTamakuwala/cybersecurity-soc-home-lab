# Hands-On Exercise: Windows Event & Telemetry Monitoring

This hands-on exercise teaches you how to trigger benign system actions on your Windows endpoint, observe the generated telemetry in Windows Event Viewer and Sysmon, and verify that Wazuh ingests, normalizes, and alerts on these actions.

---

## Exercise 1: Validating End-to-End Telemetry Flow

### Objective
Verify that routine process creation events generated on the Windows endpoint are captured by Sysmon and forwarded to the Wazuh SIEM.

### Step 1: Execute Benign Command
On the Windows endpoint, open Command Prompt or PowerShell and run:
```cmd
whoami /all
```

### Step 2: Observe Local Sysmon Telemetry
1. Open Windows Event Viewer (`eventvwr.msc`).
2. Navigate to: **Applications and Services Logs** -> **Microsoft** -> **Windows** -> **Sysmon** -> **Operational**.
3. Locate the most recent event with **Event ID 1** (Process Create):
   - **Image**: `C:\Windows\System32\whoami.exe`
   - **CommandLine**: `whoami /all`
   - **ParentImage**: `C:\Windows\System32\cmd.exe` (or `powershell.exe`)
   - **User**: `SOC-WINDOWS-ENDPOINT\socuser`
   - **Hashes**: SHA256 / MD5 hashes calculated automatically by Sysmon.

### Step 3: Verify in Wazuh Dashboard
1. On your host browser, open the Wazuh Dashboard (`https://192.168.56.10`).
2. Go to **Security events** (or open the **Discover** tab).
3. Set the time range to **Last 15 minutes**.
4. In the search bar, add a filter:
   ```
   data.win.eventdata.image : *whoami.exe
   ```
5. Expand the matched JSON document. Confirm all fields (command-line arguments, parent process, user, hashes) are parsed into structured JSON fields.

---

## Exercise 2: Local Account Enumeration & Creation

Adversaries often discover existing accounts and create backdoors using built-in administrative tools (Living off the Land).

- **MITRE ATT&CK Mapping**:
  - **T1087.001**: Discovery - Local Account Enumeration
  - **T1136.001**: Persistence / Create Account - Local Account

### Step 1: Execute Enumeration Commands
In Windows Command Prompt (Administrator):
```cmd
net user
net localgroup administrators
```

### Step 2: Create a Benign Test Account
Create a temporary local user account to simulate unauthorized account generation:
```cmd
net user soc_test_user TestPassword123! /add
```

### Step 3: Investigate the Event in Wazuh
1. In the Wazuh Dashboard, filter by:
   ```
   rule.groups : "windows" and rule.level >= 5
   ```
2. You will observe alerts triggered by Wazuh's default Windows ruleset:
   - **Windows Security Event ID 4720**: *A user account was created.*
   - **Sysmon Event ID 1**: Process creation for `net.exe` with arguments `net user soc_test_user ... /add`.
   - **Wazuh Rule ID 60109 / 60111**: High-level alert for local account addition.

### Step 4: Cleanup
Always clean up lab test accounts:
```cmd
net user soc_test_user /delete
```
*Observe that Windows Event ID 4726 (A user account was deleted) is subsequently logged.*

---

## Exercise 3: Detecting Encoded PowerShell Execution

Attackers frequently use Base64 encoding (`powershell -enc ...`) to bypass command-line text filters and obfuscate their intent.

- **MITRE ATT&CK Mapping**:
  - **T1059.001**: Execution - Command and Scripting Interpreter: PowerShell
  - **T1027**: Defense Evasion - Obfuscated/Encoded Commands

### Step 1: Execute Benign Encoded Command
On the Windows endpoint, open PowerShell and execute this harmless encoded string (which merely prints `"SOC Lab Telemetry Verification"` to the console):

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -EncodedCommand V3JpdGUtSG9zdCAiU09DIExhYiBUZWxlbWV0cnkgVmVyaWZpY2F0aW9uIg==
```

### Step 2: Observe Sysmon Telemetry
In Event Viewer -> **Sysmon/Operational**, open the newest **Event ID 1**:
- Notice the **ParentProcess**: `powershell.exe` spawned a child `powershell.exe`.
- Notice the full `-EncodedCommand` payload captured verbatim in the `CommandLine` field.

### Step 3: Investigate the Wazuh Alert
1. In Wazuh Dashboard -> **Security events**:
2. Filter for:
   ```
   data.win.eventdata.commandLine : *EncodedCommand*
   ```
3. Look at the matched alert:
   - **Rule ID**: `91802` / `60000+` series (PowerShell executed with encoded parameters or suspicious flags).
   - **MITRE ATT&CK**: Flagged under technique **T1059.001** and **T1027**.

---

## Exercise 4: Network Connection Telemetry (Sysmon Event ID 3)

### Objective
Capture outbound network socket connection attempts initiated by processes.

- **MITRE ATT&CK Mapping**:
  - **T1071**: Command and Control - Application Layer Protocol

### Step 1: Trigger Outbound Connection
In PowerShell on the Windows VM:
```powershell
Invoke-WebRequest -Uri "https://192.168.56.10" -SkipCertificateCheck -UseBasicParsing
```

### Step 2: Verify Sysmon Event ID 3 in Wazuh
1. In Wazuh Dashboard -> **Discover**:
2. Search for:
   ```
   data.win.system.eventID : 3 and data.win.eventdata.destinationIp : "192.168.56.10"
   ```
3. Expand the event:
   - `data.win.eventdata.image`: `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`
   - `data.win.eventdata.destinationPort`: `443`
   - `data.win.eventdata.protocol`: `tcp`
   - `data.win.eventdata.initiated`: `true`

---

## Summary of Observed Event IDs

| Source | Event ID | Description | Security Significance |
| :--- | :--- | :--- | :--- |
| **Sysmon** | 1 | Process Creation | Captures parent-child process relationships, hashes, and CLI parameters |
| **Sysmon** | 3 | Network Connection | Detects external C2 connections or internal lateral movement |
| **Sysmon** | 11 | File Create | Detects dropped staging binaries or web shells |
| **Windows Security** | 4624 | Successful Account Logon | Establishes user logon sessions and logon types |
| **Windows Security** | 4625 | Failed Account Logon | Identifies brute-force or credential stuffing attempts |
| **Windows Security** | 4720 | User Account Created | Detects persistence mechanisms via account addition |

---

## Next Step
Proceed to [Alert Investigation Exercise](alert-investigation.md) to practice triage and root-cause analysis like a real Tier-1 SOC analyst.
