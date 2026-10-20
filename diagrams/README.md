# Architecture & Network Diagrams

This directory contains visual architectural specifications, network diagrams, and telemetry pipeline diagrams for the **Cybersecurity SOC Home Lab**.

---

## 1. Network Topology & Port Map

This diagram illustrates how virtual machines connect across the isolated Host-Only network and which ports are utilized for log shipping and administrative operations:

```mermaid
graph TD
    Host["Physical Host System\nBrowser & SSH Client"]

    subgraph Hypervisor ["Oracle VM VirtualBox"]
        subgraph HostOnlyNet ["Isolated Host-Only Subnet (192.168.56.0/24)"]
            
            subgraph UbuntuVM ["Ubuntu Server (192.168.56.10)"]
                WazuhMgr["Wazuh Manager\nPort :1514 (Logs)\nPort :1515 (Enrollment)"]
                WazuhIdx["Wazuh Indexer\nPort :9200 (OpenSearch)"]
                WazuhDash["Wazuh Dashboard\nPort :443 (HTTPS Web UI)"]
                SSH["OpenSSH Server\nPort :22"]
                
                WazuhMgr -->|Alert Documents| WazuhIdx
                WazuhDash -->|Queries| WazuhIdx
            end

            subgraph WindowsVM ["Windows Endpoint (192.168.56.20)"]
                Sysmon["Microsoft Sysmon\nDeep OS Telemetry"]
                WinLogs["Windows Security Logs\nEvent Channels"]
                WazuhAgent["Wazuh Agent Service\n(WazuhSvc)"]

                Sysmon --> WazuhAgent
                WinLogs --> WazuhAgent
            end

            subgraph KaliVM ["Kali Linux - Optional (192.168.56.30)"]
                TestingSuite["Authorized Emulation Tools\n(Nmap / Hydra / Netcat)"]
            end

        end
    end

    WazuhAgent ==>|Encrypted Log Forwarding :1514/TCP| WazuhMgr
    TestingSuite -.->|Simulated Recon & Attacks| WindowsVM
    Host -->|HTTPS Web Management :443| WazuhDash
    Host -->|Administrative SSH :22| SSH
```

---

## 2. Telemetry Ingestion Sequence Diagram

This sequence diagram illustrates the lifecycle of a security event from execution on the endpoint to alert visualization:

```mermaid
sequenceDiagram
    autonumber
    actor Attacker as User / Actor
    participant OS as Windows OS
    participant Sysmon as Microsoft Sysmon
    participant Agent as Wazuh Agent (Endpoint)
    participant Manager as Wazuh Manager (Ubuntu)
    participant Indexer as Wazuh Indexer (OpenSearch)
    actor Analyst as SOC Analyst

    Attacker->>OS: Executes suspicious command (e.g. powershell -enc ...)
    OS->>Sysmon: Generates kernel process creation event
    Sysmon->>Sysmon: Enriches with hashes, parent PID, command line
    Sysmon->>Agent: Emits to Microsoft-Windows-Sysmon/Operational
    Agent->>Manager: Transmits encrypted log buffer over TCP :1514
    Manager->>Manager: Decodes event & evaluates detection rules
    Manager->>Indexer: Writes JSON alert document to index
    Analyst->>Indexer: Queries Dashboard over HTTPS :443
    Indexer-->>Analyst: Returns alert details & MITRE ATT&CK mapping
```

---

## 3. How to Render and Modify These Diagrams

All diagrams in this repository are written in **Mermaid.js** format:
- **GitHub Native Rendering**: GitHub automatically renders all ```mermaid blocks directly in your browser.
- **Editing Tools**:
  - [Mermaid Live Editor](https://mermaid.live/) (Web-based interactive diagram builder).
  - VS Code / Antigravity Markdown Preview with Mermaid support.
  - Export to PNG / SVG via `mermaid-cli` (`mmdc`) if high-resolution image files are desired.
