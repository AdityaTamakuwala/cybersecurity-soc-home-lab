# Wazuh SIEM & XDR Setup Guide

This guide covers installing the Wazuh central security platform on your Ubuntu Server VM using the official Wazuh installation assistant.

---

## 1. Prerequisites Verification

Before starting the installation on your Ubuntu Server:
1. Verify that your VM has at least **4 GB of RAM** and **2 vCPUs**:
   ```bash
   free -h
   nproc
   ```
2. Verify network connectivity to the internet (via NAT adapter) and check the static IP on the host-only adapter (`192.168.56.10`):
   ```bash
   ip -brief address
   curl -I https://packages.wazuh.com
   ```

---

## 2. All-in-One Installation Assistant

Wazuh provides an automated installation assistant (`wazuh-install.sh`) that installs the **Wazuh Indexer**, **Wazuh Server (Manager)**, and **Wazuh Dashboard** on a single node.

### Step 1: Download the Installation Assistant
Run the following commands on the Ubuntu Server:
```bash
cd ~
curl -sO https://packages.wazuh.com/4.x/wazuh-install.sh
chmod +x wazuh-install.sh
```

### Step 2: Run the Assistant
Execute the assistant with the `-a` (all-in-one) flag:
```bash
sudo bash ./wazuh-install.sh -a
```

> [!NOTE]
> The installation takes between 5 to 15 minutes depending on CPU performance and internet speed. It performs the following automated steps:
> 1. Installs OpenSearch/Wazuh Indexer and generates TLS certificates.
> 2. Starts and initializes the indexing cluster.
> 3. Installs the Wazuh Manager, API, and core decoders/rules.
> 4. Installs Filebeat to ship alerts from the Manager to the Indexer.
> 5. Installs the Wazuh Dashboard and configures HTTPS.

---

## 3. Post-Installation Credentials & Access

When the script finishes, it prints a success summary and saves all generated administrative credentials into an archive file named `wazuh-install-files.tar`.

### Step 1: Extract and View Passwords
```bash
sudo tar -O -xvf wazuh-install-files.tar wazuh-passwords.txt
```
Make note of the `admin` password under **Wazuh dashboard**.

### Step 2: Verify Services are Active
Check that all three core services are running:
```bash
sudo systemctl status wazuh-indexer --no-pager
sudo systemctl status wazuh-manager --no-pager
sudo systemctl status wazuh-dashboard --no-pager
```

All three services should display `active (running)`.

### Step 3: Configure Ubuntu Firewall (UFW)
Ensure necessary ports are open to the Host-Only network:
```bash
sudo ufw allow 22/tcp    # SSH management
sudo ufw allow 1514/tcp  # Wazuh Agent communication
sudo ufw allow 1515/tcp  # Wazuh Agent enrollment
sudo ufw allow 443/tcp   # Wazuh Dashboard Web UI
sudo ufw enable
sudo ufw status
```

---

## 4. Accessing the Wazuh Dashboard

1. On your **physical host workstation**, open a modern browser (Chrome, Firefox, Safari, or Edge).
2. Navigate to:
   ```
   https://192.168.56.10
   ```
3. **Handle Self-Signed Certificate**:
   - The browser will display a security warning (*"Your connection is not private"* or *"Potential Security Risk Ahead"*).
   - This is normal: Wazuh creates internal self-signed TLS certificates for local encryption.
   - Click **Advanced** -> **Proceed to 192.168.56.10 (unsafe)**.
4. **Log In**:
   - Username: `admin`
   - Password: `<the password extracted from wazuh-passwords.txt>`
5. You will be greeted by the Wazuh Welcome screen and the main security overview dashboard.

---

## 5. Basic Platform Health Check

Run a quick health check from the Ubuntu terminal to confirm the indexer is healthy:
```bash
# Query the indexer cluster health API using the admin certificate
sudo curl -k -u admin:<your_admin_password> https://127.0.0.1:9200/_cluster/health?pretty
```
Expected output:
```json
{
  "cluster_name" : "wazuh-cluster",
  "status" : "green",
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1
}
```

---

## Next Step
Proceed to [Windows Endpoint Setup](windows-endpoint-setup.md) to install Sysmon and connect the Windows VM to your Wazuh Server.
