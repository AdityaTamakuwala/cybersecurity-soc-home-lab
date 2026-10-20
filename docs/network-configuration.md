# Safe Virtual Networking Guide

Network design is the most critical safety factor when building a cybersecurity home lab. A properly isolated network ensures that simulated threats, malware samples, or reconnaissance traffic cannot escape into your home LAN or reach the public internet.

---

## 1. VirtualBox Network Modes Explained

VirtualBox provides several networking modes. Understanding their differences is essential for choosing the right topology:

| Network Mode | Can VM access Internet? | Can Host access VM? | Can VMs talk to each other? | Suitable for Security Lab? |
| :--- | :--- | :--- | :--- | :--- |
| **NAT (Default)** | Yes | No (unless port-forwarded) | No | Good for downloading updates only |
| **Bridged Adapter** | Yes | Yes (acts as device on home router) | Yes (and with all home devices) | ⚠️ **DANGEROUS** for security labs |
| **Internal Network** | No | No | Yes (only VMs on same internal net) | Very safe, but requires manual routing |
| **Host-Only Network** | No | Yes (via virtual host adapter) | Yes | **Recommended** for lab management |
| **NAT Network** | Yes | No (unless port-forwarded) | Yes | Good for shared internet access |

> [!CAUTION]
> **NEVER use Bridged Networking for security testing VMs or vulnerable endpoints.** 
> Bridged networking places the virtual machine directly onto your physical home or office network. Any malicious payload, unauthorized scan, or vulnerable service would be exposed to physical devices on your LAN.

---

## 2. Recommended Lab Network Architecture: Dual-Adapter Strategy

To balance **internet access for installing updates and packages** with **complete isolation and inter-VM communication**, we use a **Dual-Adapter Strategy** on the virtual machines:

```
┌────────────────────────────────────────────────────────┐
│ Host Machine (Your Physical PC / Mac)                  │
│ VirtualBox Host-Only Ethernet Adapter: 192.168.56.1    │
└───────────────▲────────────────────────▲───────────────┘
                │                        │
   Host-Only    │ (192.168.56.0/24)      │ (192.168.56.0/24)
   Inter-VM     │                        │
   Subnet       ▼                        ▼
      ┌──────────────────┐     ┌──────────────────┐
      │ Ubuntu Server    │     │ Windows Endpoint │
      │ (Wazuh SIEM)     │     │ (Monitored Host) │
      │                  │     │                  │
      │ Adapter 1: NAT   │     │ Adapter 1: NAT   │  <-- Disabled during
      │ Adapter 2: Host- │     │ Adapter 2: Host- │      active testing
      │            Only  │     │            Only  │
      │ IP: 192.168.56.10│     │ IP: 192.168.56.20│
      └──────────────────┘     └──────────────────┘
```

1. **Adapter 1 (NAT)**:
   - Used temporarily during setup to download packages (`apt update`, Wazuh installation script, Sysmon installer).
   - Can be unchecked/disabled in VirtualBox VM settings once the lab is fully configured.
2. **Adapter 2 (Host-Only Network)**:
   - Dedicated private subnet (`192.168.56.0/24`).
   - Handles all inter-VM communication: Wazuh agent logs transmitted to Wazuh server on port `1514`.
   - Allows your host browser to access the Wazuh Dashboard at `https://192.168.56.10`.
   - Never routes traffic out to the physical internet.

---

## 3. Step-by-Step Configuration in VirtualBox

### Step 1: Create the Host-Only Network
1. Launch **VirtualBox**.
2. Go to **File** -> **Tools** -> **Network Manager** (or click the hamburger menu on the right and select **Network**).
3. Select the **Host-only Networks** tab.
4. Click **Create**. A new interface (e.g., `vboxnet0` on Linux/macOS or `VirtualBox Host-Only Ethernet Adapter` on Windows) will appear.
5. In the **Adapter** tab:
   - IPv4 Address: `192.168.56.1`
   - IPv4 Network Mask: `255.255.255.0`
6. In the **DHCP Server** tab:
   - Check **Enable Server** (optional, recommended for automatic IP assignment).
   - Server Address: `192.168.56.2`
   - Server Mask: `255.255.255.0`
   - Lower Address Bound: `192.168.56.100`
   - Upper Address Bound: `192.168.56.200`
   - *Note: We assign static IPs (`192.168.56.10`, `192.168.56.20`) below the DHCP pool for stability.*
7. Click **Apply**.

---

### Step 2: Assign Adapters to Ubuntu Server VM
1. In VirtualBox, right-click your Ubuntu Server VM and choose **Settings**.
2. Navigate to **Network**:
   - **Adapter 1**:
     - Check **Enable Network Adapter**.
     - Attached to: **NAT**.
   - **Adapter 2**:
     - Check **Enable Network Adapter**.
     - Attached to: **Host-only Adapter**.
     - Name: Select the Host-only network created above (e.g., `VirtualBox Host-Only Ethernet Adapter` or `vboxnet0`).
     - Advanced -> Promiscuous Mode: **Deny**.
3. Click **OK**.

---

### Step 3: Assign Adapters to Windows Endpoint VM
1. Right-click your Windows VM and choose **Settings**.
2. Navigate to **Network**:
   - **Adapter 1**:
     - Check **Enable Network Adapter**.
     - Attached to: **NAT**.
   - **Adapter 2**:
     - Check **Enable Network Adapter**.
     - Attached to: **Host-only Adapter**.
     - Name: Same Host-only network interface.
     - Advanced -> Promiscuous Mode: **Deny**.
3. Click **OK**.

---

## 4. Static IP Configuration for Ubuntu Server

Because the Wazuh Agent must always send logs to a fixed address, configure a static IP on the Ubuntu Server's Host-Only adapter.

Ubuntu 22.04 LTS uses **Netplan** for network configuration:

1. Identify the interface name for Adapter 2 (usually `enp0s8` or `eth1`):
   ```bash
   ip -brief address
   ```
2. Edit the Netplan configuration file:
   ```bash
   sudo nano /etc/netplan/00-installer-config.yaml
   ```
3. Configure static addressing for the second interface (replace `enp0s8` with your actual interface name):
   ```yaml
   network:
     version: 2
     renderer: networkd
     ethernets:
       enp0s3:
         dhcp4: true
       enp0s8:
         dhcp4: no
         addresses:
           - 192.168.56.10/24
   ```
4. Test and apply the configuration:
   ```bash
   sudo netplan try
   sudo netplan apply
   ```
5. Verify the IP address:
   ```bash
   ip addr show enp0s8
   ```

---

## 5. Security & Isolation Verification

After completing network setup, perform these checks before generating any security events:

1. **Verify Inter-VM Communication**:
   - From Windows Command Prompt:
     ```cmd
     ping 192.168.56.10
     ```
     *(If ping fails, verify that Ubuntu's firewall allows ICMP or test port 1514 reachability).*
2. **Verify Host-to-Server Communication**:
   - From your host machine's terminal:
     ```bash
     ping 192.168.56.10
     ```
3. **Verify Guest Isolation from Physical LAN**:
   - Confirm that the Windows VM cannot ping physical IP addresses on your home subnet (e.g., your home router `192.168.1.1` or other family computers).

---

## Next Step
Proceed to [Installation Guide](installation-guide.md) to install and configure Ubuntu Server and Windows.
