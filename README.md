<h1 align="center">Sliver C2 Simple Server Deployment Guide</h1>

<p align="center">
<pre>
 ██████  ██      ██ ██    ██ ███████ ██████  
██       ██      ██ ██    ██ ██      ██   ██ 
 █████   ██      ██ ██    ██ █████   ██████  
     ██  ██      ██  ██  ██  ██      ██   ██ 
██████   ███████ ██   ████   ███████ ██   ██ 
</pre>
  <b><i>">> Stealthy C2 Infrastructure on Azure <<"</i></b>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Azure-%230072C6.svg?style=for-the-badge&logo=microsoftazure&logoColor=white" />
  <img src="https://img.shields.io/badge/Nginx-%23009639.svg?style=for-the-badge&logo=nginx&logoColor=white" />
  <img src="https://img.shields.io/badge/Sliver-C2-red?style=for-the-badge" />
</p>

<blockquote>
  <b>"As a Red Teamer, you must know how to build your own C2 infrastructure. If you're a newcomer, follow the guide below to get started."</b>
</blockquote>

---

## Minimum System Requirements
Based on standard Red Team operations:

* **Redirector**: At least **1 vCPU** and **1GB RAM**.
* **C2 Server**: At least **2 vCPUs** and **2GB RAM**.

---

## Network Architecture & Security Posture

<p align="center">
  <img src="diagram.png" width="650px" alt="C2 Network Diagram">
</p>

The infrastructure is designed with a **Tier-2 Stealth Model**, ensuring the C2 backend remains completely invisible to the public internet.

### 1. Firewall Configuration (NSG Rules)
We implement a strict **Allow-by-Exception** policy. All "Allow" rules must have a priority **under 1000**.

#### **A. Redirector (Frontend)**
| Direction | Port | Source | Action |
| :--- | :--- | :--- | :--- |
| Inbound | `80`, `443` | Anywhere | **Allow** |
| Inbound | `22` | Admin Client IP | **Allow** |
| Outbound | `8443` | C2 Private IP | **Allow** |

#### **B. C2 Server (Isolated Backend)**
| Direction | Port | Source | Action |
| :--- | :--- | :--- | :--- |
| Inbound | `80`, `443`, `8443`, `22` | Redir Private IP | **Allow** |
| Outbound | `80`, `443`, `8443`, `22` | Redir Private IP | **Allow** |
| **Any** | **Any** | **Anywhere** | **Deny (Priority 1000)** |

---

<h2> Phase 1: Accessing the Infrastructure</h2>

<p>Securely connect to your Azure instances using the private key provided during provisioning.</p>

> [!WARNING]
> **Key Management:** Azure provides the private key only **once**. Keep it safe!

<p><b>1. Set Secure Key Permissions</b></p>

```bash
chmod 400 your-private-key.pem
```
<p><b>2. Establish SSH Connection</b></p>
<p>Run the following command to access your Redirector instance:</p>

```bash
ssh -i <your-private-key.pem> <user>@<redirector-ip>
```
<h2>🚀 Phase 2: Environment Setup & SSL Configuration</h2>

<p>Once connected, the first priority is to update the system and install the necessary dependencies for the reverse proxy and SSL management.</p>

<p><b>1. Update System & Install Dependencies</b></p>
<p>Run these commands to ensure your Redirector is up-to-date and equipped with Nginx and Certbot:</p>
```bash
sudo apt update && sudo apt upgrade -y
sudo apt-get install -y nginx certbot net-tools
```
<p><b>2. Provision SSL Certificate with Certbot</b></p>
<p>Before proceeding, ensure your domain's <b>A Record</b> is already pointed to your Redirector's Public IP. We need to stop Nginx temporarily to allow Certbot to use port 80 for verification.</p>
```bash 
# Stop Nginx to free up port 80
sudo systemctl stop nginx

# Request a standalone SSL certificate
sudo certbot certonly --register-unsafely-without-email -d [www.yourdomain.com](https://www.yourdomain.com)
```
