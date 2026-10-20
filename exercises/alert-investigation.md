# Hands-On Exercise: SOC Alert Investigation Workflow

This exercise guides you through the standard investigative workflow used by professional Security Operations Center (SOC) analysts to triage, analyze, and document security alerts using Wazuh.

---

## 1. The Standard 5-Phase SOC Triage Lifecycle

When an alert fires in a SIEM, analysts follow a systematic lifecycle to avoid tunnel vision:

```
┌────────────────────────────────────────────────────────┐
│ Phase 1: Alert Triage & Initial Assessment             │
│ (Rule level, rule description, true vs false positive) │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│ Phase 2: Scope & Asset Identification                  │
│ (Target hostname, IP address, user account, agent ID)  │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│ Phase 3: Deep Telemetry Correlation & Timeline         │
│ (Sysmon process tree, command line, network sockets)   │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│ Phase 4: Impact & MITRE ATT&CK Attribution             │
│ (Persistence? Privilege escalation? Lateral movement?) │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
┌────────────────────────────────────────────────────────┐
│ Phase 5: Containment, Remediation & Incident Reporting │
│ (Isolate host, reset credentials, draft report)        │
└────────────────────────────────────────────────────────┘
```

---

## 2. Simulated Investigation Scenario: Suspicious Logon Activity

### Background Scenario
*Simulated Alert*: The SOC SIEM alerts on multiple consecutive failed logons on `SOC-WINDOWS-ENDPOINT` within a short window, followed immediately by a successful interactive logon.

- **MITRE ATT&CK Mapping**:
  - **T1110.001**: Credential Access - Password Guessing / Brute Force
  - **T1078.003**: Defense Evasion / Initial Access - Valid Accounts: Local Accounts

---

### Step 1: Simulate the Event on Windows
To simulate this safely without external attack tools, execute a rapid sequence of failed authentication attempts from PowerShell on the Windows VM:

```powershell
# Attempt authentication with invalid passwords 5 times
1..5 | ForEach-Object {
    net use \\127.0.0.1\IPC$ /user:socuser "WrongPassword$_" 2>$null
}

# Followed immediately by a successful logon check
whoami
```

---

### Step 2: Phase 1 — Alert Triage in Wazuh Dashboard
1. Open Wazuh Dashboard -> **Security events**.
2. Filter for:
   ```
   rule.groups : "windows" and data.win.system.eventID : 4625
   ```
3. Observe the multiple alerts:
   - **Rule ID 60122**: *Windows: Logon failure - Unknown user or bad password*.
   - **Event ID**: `4625` (An account failed to log on).
   - **Severity Level**: Evaluates higher as repeat failures occur within a defined threshold.

---

### Step 3: Phase 2 — Asset & Identity Scoping
Inspect the alert JSON payload to answer key scoping questions:
- **Target User**: `data.win.eventdata.targetUserName` (e.g., `socuser`)
- **Source Workstation / IP**: `data.win.eventdata.workstationName` and `data.win.eventdata.ipAddress`
- **Logon Type**:
  - `LogonType: 2` -> Interactive (console / direct keyboard)
  - `LogonType: 3` -> Network connection (e.g. SMB share, PsExec, net use)
  - `LogonType: 10` -> Remote Desktop (RDP)
- **Status / Sub-Status Code**:
  - `0xC000006A`: User name exists, but password was incorrect.
  - `0xC0000064`: User name does not exist.

---

### Step 4: Phase 3 — Correlate with Sysmon Telemetry
Did the actor execute processes after the authentication activity?
1. Remove the Event ID filter and filter by the target agent:
   ```
   agent.name : "windows-endpoint"
   ```
2. Narrow the time window to 2 minutes before and after the alert.
3. Look for **Sysmon Event ID 1 (Process Create)**:
   - Identify the parent process (`net.exe` spawned by `powershell.exe`).
   - Identify who initiated the PowerShell session.
   - Trace any secondary binaries executed (e.g., `whoami.exe`, `nltest.exe`, `net.exe`).

---

### Step 5: Phase 4 — True Positive Determination
Determine whether the activity was authorized or suspicious:
- Was this an administrator performing maintenance?
- Was an automated script misconfigured with an expired password?
- Or did an unauthorized actor attempt credential guessing?

In our lab scenario, this is classified as:
- **Classification**: **True Positive - Simulated Adversary Activity**
- **Severity**: Medium
- **Immediate Risk**: Low (occurred within isolated lab environment; local account was targeted).

---

### Step 6: Phase 5 — Analyst Containment & Next Steps
In a real enterprise environment, the analyst would:
1. Isolate the endpoint from the network via Wazuh Active Response or EDR.
2. Force an immediate password reset for the compromised account.
3. Terminate active user sessions on the endpoint.
4. Submit a completed Incident Report.

---

## Next Step
Review the [Sample Incident Report & Reusable Template](../incident-reports/sample-incident-report.md) to practice professional incident documentation.
