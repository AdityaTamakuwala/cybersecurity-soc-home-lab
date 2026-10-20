# Lab Prerequisites & System Requirements

Before beginning the installation of your Cybersecurity Security Operations Center (SOC) Home Lab, ensure your host computer meets the hardware and software specifications outlined below.

---

## 1. Hardware Specifications

Running multiple virtual machines simultaneously places significant demands on RAM, CPU, and storage. The table below outlines minimum and recommended hardware requirements for the host machine.

| Component | Minimum Specification | Recommended Specification |
| :--- | :--- | :--- |
| **Processor (CPU)** | 4 Cores (x86_64) with VT-x / AMD-V | 6–8 Cores (x86_64) with nested virtualization support |
| **Memory (RAM)** | 16 GB DDR4/DDR5 | 32 GB DDR4/DDR5 |
| **Storage** | 100 GB free space on SSD | 250+ GB free space on NVMe SSD |
| **Host Operating System** | Windows 10/11, macOS (Intel/Apple Silicon*), or Linux | Windows 11 Pro / Ubuntu 22.04+ / macOS |

> [!NOTE]
> *Apple Silicon (M1/M2/M3) Note: VirtualBox 7.x supports ARM64 hosts with developer preview features, but x86_64 Windows evaluation images require emulation or alternative hypervisors such as UTM or VMware Fusion Pro. This guide assumes standard x86_64 virtualization.*

### Virtual Machine Resource Breakdown

| Virtual Machine | Role | Min RAM | Recommended RAM | vCPUs | Storage Allocated |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Ubuntu Server** | Wazuh Manager, Indexer, Dashboard | 4 GB | 6–8 GB | 2 | 40 GB (Dynamically Allocated) |
| **Windows 10/11** | Monitored Endpoint (Sysmon + Agent) | 4 GB | 4–6 GB | 2 | 50 GB (Dynamically Allocated) |
| **Kali Linux** *(Optional)* | Authorized Testing & Traffic Generation | 2 GB | 4 GB | 1–2 | 30 GB (Dynamically Allocated) |
| **Host Overhead** | Running Host OS & Hypervisor | 4 GB | 8 GB | Remaining | N/A |

---

## 2. Software & Download Sources

Always download hypervisors, operating system images, and security utilities directly from official vendor sources.

### Core Virtualization Software
- **Oracle VM VirtualBox (7.0.x or latest)**
  - Hypervisor: [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads)
  - VirtualBox Extension Pack (matching your installed version): [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads)

### Guest Operating System Images
- **Ubuntu Server 22.04 LTS (Jammy Jellyfish)**
  - Official ISO Download: [https://releases.ubuntu.com/jammy/](https://releases.ubuntu.com/jammy/)
  - *Recommendation: Use standard Ubuntu Server ISO (x86_64) without desktop GUI to conserve memory.*
- **Windows 10 / 11 Enterprise (Evaluation)**
  - Microsoft Evaluation Center (free 90-day evaluation): [https://www.microsoft.com/evalcenter/evaluate-windows-10-enterprise](https://www.microsoft.com/evalcenter/evaluate-windows-10-enterprise) or [https://www.microsoft.com/evalcenter/evaluate-windows-11-enterprise](https://www.microsoft.com/evalcenter/evaluate-windows-11-enterprise)
- **Kali Linux (Optional Testing Machine)**
  - Official VirtualBox Image: [https://www.kali.org/get-kali/#kali-virtual-machines](https://www.kali.org/get-kali/#kali-virtual-machines)

### Telemetry & Security Software
- **Wazuh Security Platform (v4.x)**
  - Documentation & Installation Scripts: [https://documentation.wazuh.com/current/quickstart.html](https://documentation.wazuh.com/current/quickstart.html)
  - Wazuh Windows Agent: Downloaded directly through the Wazuh Dashboard or package repository.
- **Microsoft Sysinternals Sysmon**
  - Official Download: [https://learn.microsoft.com/sysinternals/downloads/sysmon](https://learn.microsoft.com/sysinternals/downloads/sysmon)
  - Recommended Baseline Configuration: [SwiftOnSecurity sysmon-config on GitHub](https://github.com/SwiftOnSecurity/sysmon-config)
- **Wireshark**
  - Official Download: [https://www.wireshark.org/download.html](https://www.wireshark.org/download.html)
- **MITRE ATT&CK Framework Reference**
  - Enterprise Matrix: [https://attack.mitre.org/matrices/enterprise/](https://attack.mitre.org/matrices/enterprise/)

---

## 3. Host System Verification Checklist

Before creating virtual machines, confirm that hardware virtualization is active on your host system:

1. **Hardware Virtualization in BIOS/UEFI**:
   - Intel CPUs: **Intel VT-x** / **Intel Virtualization Technology** must be **Enabled**.
   - AMD CPUs: **AMD-V** / **SVM (Secure Virtual Machine) Mode** must be **Enabled**.
   - *Verification on Windows*: Open Task Manager -> **Performance** tab -> **CPU** -> Look for **Virtualization: Enabled**.
   - *Verification on Linux*: Run `egrep -c '(vmx|svm)' /proc/cpuinfo` (a value greater than 0 confirms hardware support).
   - *Verification on macOS*: Run `sysctl -a | grep machdep.cpu.features` and check for `VMX`.

2. **Hypervisor Conflict Check (Windows Hosts)**:
   - If using VirtualBox on Windows 11, WSL2 or Hyper-V may coexist, but performance is highest when VirtualBox has direct access or when Hyper-V integration is properly managed.

3. **Storage Allocation Check**:
   - Ensure the disk partition where VirtualBox saves VMs has at least 80–100 GB of free space.
   - Use dynamic disk allocation for guest disks so storage is consumed only as needed.

---

## Next Step
Proceed to [Lab Architecture](lab-architecture.md) to understand how the components connect and how telemetry flows through your virtual SOC.
