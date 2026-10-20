# Security Incident Report & Template

> [!NOTE]
> **DISCLAIMER**: The incident scenario, IP addresses, hashes, and timestamps contained in Section 1 below are **fictional and illustrative**, created strictly for educational demonstration within this SOC home lab.

---

# Section 1: Sample Completed Incident Report

## Incident Overview: Unauthorized Process Execution & Local Persistence Simulation

| Metadata Field | Value |
| :--- | :--- |
| **Incident ID** | INC-2026-1020-01 |
| **Severity** | **Medium** (CVSS: 6.2 Equivalent) |
| **Status** | Closed / Resolved (Lab Simulation) |
| **Date / Time Detected** | 2026-10-20 14:15:22 UTC |
| **Lead Analyst** | SOC Tier-1 Analyst (Lab Portfolio) |
| **Detection Source** | Wazuh SIEM + Microsoft Sysmon (Agent ID: 001) |

---

### 1. Executive Summary
On October 20, 2026, at 14:15:22 UTC, the Wazuh SIEM alerted on suspicious obfuscated PowerShell execution on workstation `SOC-WINDOWS-ENDPOINT` (`192.168.56.20`). Subsequent telemetry correlation revealed execution of Base64-encoded command parameters followed by local user account creation (`soc_test_user`). 

Because this activity was confined to an isolated virtualization environment, no production systems were impacted. The simulated threat actor actions were contained, the rogue user account was expunged, and detection rules were verified.

---

### 2. Affected Assets & Accounts

- **Hostname**: `SOC-WINDOWS-ENDPOINT`
- **Internal IP**: `192.168.56.20`
- **Operating System**: Windows 10 Enterprise (Build 19045)
- **Targeted Account**: `SOC-WINDOWS-ENDPOINT\socuser` (Local Administrator)
- **Rogue Account Created**: `SOC-WINDOWS-ENDPOINT\soc_test_user`

---

### 3. Chronological Timeline of Events

| Timestamp (UTC) | Source / Component | Event ID | Activity Description |
| :--- | :--- | :--- | :--- |
| **14:12:05** | Sysmon | Event ID 1 | `cmd.exe` launched by `socuser` (PID: 4892) |
| **14:14:10** | Sysmon | Event ID 1 | `powershell.exe` executed with `-EncodedCommand` parameter |
| **14:14:11** | Wazuh Manager | Rule 91802 | **Alert Triggered**: Obfuscated PowerShell execution detected (Severity 8) |
| **14:15:02** | Security Log | Event ID 4720 | A local user account `soc_test_user` was created |
| **14:15:03** | Wazuh Manager | Rule 60109 | **Alert Triggered**: New Windows local user account created |
| **14:17:30** | SOC Analyst | N/A | Analyst initiated triage and isolated endpoint |
| **14:22:00** | Security Log | Event ID 4726 | Rogue user account `soc_test_user` deleted by analyst |

---

### 4. MITRE ATT&CK Mapping

| Tactic | Technique ID | Technique Name | Observed Technical Action |
| :--- | :--- | :--- | :--- |
| **Execution** | T1059.001 | PowerShell | Execution of interactive PowerShell session |
| **Defense Evasion** | T1027 | Obfuscated Files/Information | Base64-encoded command-line parameter (`-EncodedCommand`) |
| **Persistence** | T1136.001 | Create Account: Local Account | Execution of `net user soc_test_user <password> /add` |
| **Discovery** | T1087.001 | Local Account Discovery | Execution of `net user` and `net localgroup administrators` |

---

### 5. Technical Evidence & Artifacts

#### Sysmon Process Creation Telemetry (Event ID 1)
```json
{
  "win": {
    "system": {
      "providerName": "Microsoft-Windows-Sysmon",
      "eventID": "1",
      "computer": "SOC-WINDOWS-ENDPOINT"
    },
    "eventdata": {
      "image": "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe",
      "commandLine": "powershell.exe -NoProfile -ExecutionPolicy Bypass -EncodedCommand V3JpdGUtSG9zdCAiU09DIExhYiBUZWxlbWV0cnkgVmVyaWZpY2F0aW9uIg==",
      "parentImage": "C:\\Windows\\System32\\cmd.exe",
      "parentCommandLine": "cmd.exe",
      "user": "SOC-WINDOWS-ENDPOINT\\socuser",
      "hashes": "SHA256=A2F6B8C0D3E14795E7D890... (Illustrative)",
      "currentDirectory": "C:\\Users\\socuser\\Desktop\\"
    }
  }
}
```

---

### 6. Containment & Remediation Actions

1. **Immediate Containment**:
   - Monitored host network adapter disabled to prevent potential outbound communication.
2. **Eradication**:
   - Rogue local user account `soc_test_user` deleted (`net user soc_test_user /delete`).
   - Inspected scheduled tasks and registry run keys (`HKLM\Software\Microsoft\Windows\CurrentVersion\Run`) for persistence hooks; none found.
3. **Recovery & Verification**:
   - Restored network connectivity.
   - Re-verified integrity of the Wazuh agent communication.

---

### 7. Lessons Learned & Recommendations

- **Audit Policy Enhancement**: Ensure PowerShell Script Block Logging (Event ID 4104) is enabled across all endpoints to capture unencoded de-obfuscated script contents.
- **Privilege Management**: Restrict non-administrative users from executing unconstrained PowerShell commands via AppLocker / Software Restriction Policies.
- **Alert Tuning**: Create a correlated Wazuh rule to elevate alert severity if PowerShell execution is followed by `net.exe /add` within a 120-second sliding window.

---

# Section 2: Reusable Incident Report Template

Students and analysts can copy the template below for their own portfolio exercises:

```markdown
# Incident Report: [Incident Title]

## Metadata
- **Incident ID**: INC-YYYYMMDD-XX
- **Severity**: [Low / Medium / High / Critical]
- **Status**: [New / In Progress / Contained / Closed]
- **Date/Time Detected**: YYYY-MM-DD HH:MM UTC
- **Lead Analyst**: [Analyst Name / Title]
- **Detection Mechanism**: [SIEM Alert / Sysmon / EDR / User Reported]

---

## 1. Executive Summary
[Brief 2-3 paragraph non-technical summary explaining what occurred, affected systems, root cause, and business impact.]

---

## 2. Scope & Affected Assets
- **Hostname(s)**: 
- **IP Address(es)**: 
- **Affected User Account(s)**: 
- **Operating System(s)**: 

---

## 3. Incident Timeline
| Timestamp (UTC) | Source / System | Event ID | Description of Action |
| :--- | :--- | :--- | :--- |
| YYYY-MM-DD HH:MM | Sysmon | 1 | Process spawned |
| YYYY-MM-DD HH:MM | Wazuh | Rule ID | Alert triggered |

---

## 4. Threat Mapping (MITRE ATT&CK)
| Tactic | Technique ID | Technique Name | Evidence |
| :--- | :--- | :--- | :--- |
| | | | |

---

## 5. Technical Evidence
```json
// Paste relevant log snippets or JSON payloads here
```

---

## 6. Containment & Eradication Steps
1. [Action 1]
2. [Action 2]

---

## 7. Lessons Learned & Preventative Measures
- [Recommendation 1]
- [Recommendation 2]
```
