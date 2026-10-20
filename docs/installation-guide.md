# Step-by-Step VM Installation Guide

This guide walks you through creating and configuring the primary Virtual Machines (Ubuntu Server and Windows 10/11 Endpoint) in Oracle VM VirtualBox.

---

## 1. VirtualBox & Extension Pack Setup

1. **Install VirtualBox**:
   - Download the installer for your host operating system from [VirtualBox Downloads](https://www.virtualbox.org/wiki/Downloads).
   - Follow standard installation prompts.
2. **Install VirtualBox Extension Pack**:
   - Download the Extension Pack matching your VirtualBox version.
   - Open VirtualBox -> **File** -> **Tools** -> **Extension Pack Manager** -> Click **Install** -> Accept the license.

---

## 2. Virtual Machine 1: Ubuntu Server (Wazuh SIEM)

### VM Specifications
- **Name**: `SOC-Wazuh-Server`
- **Type**: Linux
- **Version**: Ubuntu (64-bit)
- **Base Memory**: 4096 MB (4 GB minimum, 6144 MB / 6 GB recommended)
- **Processors**: 2 vCPUs
- **Storage**: 40 GB dynamically allocated VDI

### Step-by-Step Creation
1. Open VirtualBox and click **New**.
2. Name: `SOC-Wazuh-Server`, Type: `Linux`, Version: `Ubuntu (64-bit)`.
3. Check **Skip Unattended Installation** (recommended so you can manually select network and disk options).
4. Assign **4096 MB** RAM and **2 vCPUs**.
5. Create a virtual hard disk now: **40 GB**, VDI format, **Dynamically allocated**.
6. Click **Finish**.
7. With the VM selected, click **Settings**:
   - **Storage**: Select the empty optical drive under *Controller: IDE* -> Click the disc icon -> Choose your downloaded `ubuntu-22.04-live-server-amd64.iso`.
   - **Network**:
     - Adapter 1: Attached to **NAT**.
     - Adapter 2: Attached to **Host-only Adapter** (select `VirtualBox Host-Only Ethernet Adapter` or `vboxnet0`).
   - Click **OK**.

### Ubuntu OS Installation Steps
1. Start the VM. Choose **Try or Install Ubuntu Server**.
2. Select your language and keyboard layout.
3. Choose base installation type: **Ubuntu Server** (minimize packages).
4. Configure Network: Note the interface names (e.g., `enp0s3` and `enp0s8`). Leave both on DHCP during installation.
5. Disk Configuration: Choose **Use an entire disk** (default 40 GB disk).
6. Profile setup:
   - Your name: `SOC Admin`
   - Server name: `wazuh-server`
   - Username: `socadmin`
   - Password: Use a strong passphrase for the lab.
7. SSH Setup: Check **Install OpenSSH server** (allows remote management from your host terminal).
8. Featured Server Snaps: Leave unselected and continue.
9. Complete the installation, remove the ISO when prompted, and reboot.

### Initial Server Preparation
Once booted into Ubuntu Server, log in as `socadmin` and update packages:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl apt-transport-https lsb-release gnupg2 tar
```

---

## 3. Virtual Machine 2: Windows Endpoint

### VM Specifications
- **Name**: `SOC-Windows-Endpoint`
- **Type**: Microsoft Windows
- **Version**: Windows 10 (64-bit) or Windows 11 (64-bit)
- **Base Memory**: 4096 MB (4 GB)
- **Processors**: 2 vCPUs
- **Storage**: 50 GB dynamically allocated VDI

### Step-by-Step Creation
1. In VirtualBox, click **New**.
2. Name: `SOC-Windows-Endpoint`, Type: `Microsoft Windows`.
3. Check **Skip Unattended Installation**.
4. Set memory to **4096 MB** and **2 vCPUs**.
5. Create a **50 GB** dynamically allocated virtual hard disk.
6. Click **Finish**.
7. Open **Settings**:
   - **System** -> **Motherboard**: Ensure **Enable EFI** is checked if installing Windows 11.
   - **Storage**: Attach the Windows 10/11 Enterprise Evaluation ISO to the optical drive.
   - **Network**:
     - Adapter 1: **NAT**.
     - Adapter 2: **Host-only Adapter** (same host-only network).
   - Click **OK**.

### Windows Installation Steps
1. Start the VM and press any key to boot from CD/DVD.
2. Select language, time, and keyboard settings -> Click **Install Now**.
3. Choose **Windows 10/11 Enterprise Evaluation**.
4. Accept license terms -> Select **Custom: Install Windows only (advanced)**.
5. Select Drive 0 Unallocated Space and click **Next**.
6. When prompted during OOBE (out-of-box experience):
   - Choose **Domain join instead** or **Set up for work/school** -> **Domain join** to create a standard local administrator account.
   - Username: `socuser` or `analyst`
   - Set a memorable password.
   - Turn OFF unnecessary diagnostic/advertising telemetry toggles.

### Post-Install Windows Optimizations
1. **Install VirtualBox Guest Additions**:
   - In VirtualBox menu: **Devices** -> **Insert Guest Additions CD Image...**.
   - Open File Explorer in Windows -> CD Drive -> Run `VBoxWindowsAdditions.exe` as Administrator.
   - Reboot when finished.
2. **Configure Host-Only Network Adapter**:
   - Open **Settings** -> **Network & Internet** -> **Network Connections**.
   - Locate the second network adapter connected to the Host-Only network.
   - Right-click -> **Properties** -> **Internet Protocol Version 4 (TCP/IPv4)**.
   - Assign static IP:
     - IP address: `192.168.56.20`
     - Subnet mask: `255.255.255.0`
     - Default gateway: *Leave blank (so internet traffic routes through Adapter 1 NAT)*.
   - Click **OK**.

---

## 4. Take Base VM Snapshots

> [!IMPORTANT]
> **Take a snapshot NOW before installing any security software.**
> Snapshots allow you to reset your lab to a clean, known-good state if an exercise breaks a system or if an evaluation license expires.

1. In the VirtualBox main window, select `SOC-Wazuh-Server`.
2. Click the hamburger menu next to the VM name -> **Snapshots**.
3. Click **Take** -> Name: `Clean Base OS` -> Description: `Ubuntu 22.04 LTS updated, network configured`.
4. Repeat the same process for `SOC-Windows-Endpoint` -> Name: `Clean Windows Base`.

---

## Next Step
Proceed to [Wazuh Setup](wazuh-setup.md) to install the Wazuh SIEM/XDR manager and access the dashboard.
