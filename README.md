# AWS SOC (Wazuh) Project

This project demonstrates building a **Security Operations Center (SOC)** using **Wazuh** deployed on AWS. It integrates Windows and Linux systems, including **Windows Server 2025** and **Windows 10**, to monitor events, file integrity, and security logs.

---

## 🌍 AWS Setup (Mumbai `ap-south-1`)

### EC2 Instances
- **Wazuh Manager**: Ubuntu 22.04 LTS — `t2.xlarge`
- **Windows Server 2025** (Domain Controller + Agent) — `t2.large`
- **Ubuntu Agent** — `t2.micro`
- **Windows 10 PC** — External (public IP, enrolled as Wazuh agent)

### Security Groups (Inbound Ports)
- **22** (SSH, Ubuntu only)
- **80, 443** (Wazuh Dashboard & API)
- **1514, 1515** (Wazuh Agent communication)
- **3389** (RDP, Windows Server/PC if needed)

⚠️ Agents must connect to the **public IP of the Wazuh server**.

---

## ⚙️ Wazuh Manager Installation
Run the all-in-one installer:
```bash
curl -sO https://packages.wazuh.com/4.9/wazuh-install.sh
sudo bash wazuh-install.sh -a -i 
```

Login dashboard:  
`https://<WAZUH_PUBLIC_IP>` (user: `admin`, password auto-generated)

---

## 🖥️ Agent Enrollment

### 1. Windows Server 2025
1. Download the Windows agent MSI:
   - [Wazuh Windows Agent](https://packages.wazuh.com/4.x/windows/wazuh-agent-4.9.msi)
2. Run installer → provide:
   - **Wazuh Manager Address**: `WAZUH_PUBLIC_IP`
   - **Agent Name**: `WinServer2025`
3. Configure agent (`C:\Program Files (x86)\ossec-agent\ossec.conf`):
   ```xml
   <client>
     <server>
       <address>WAZUH_PUBLIC_IP</address>
       <port>1514</port>
       <protocol>tcp</protocol>
     </server>
   </client>
   ```
4. Start service:
   ```powershell
   net start WazuhSvc
   ```
Important:- ##...Also you can use gui interface for Configure Agent in windows platformes..##

### 2. Windows 10 PC
(Same as Windows Server steps, just change Agent Name → `Win10PC`)

### 3. Ubuntu Agent
```bash
curl -so wazuh-agent.deb https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.9.0-1_amd64.deb
sudo WAZUH_MANAGER='WAZUH_PUBLIC_IP' dpkg -i ./wazuh-agent.deb
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

---
Imporatant: ## Sometimes agent should be manually register in wazuh manager using this command ##
sudo /var/ossec/bin/manage_agents 

## 🔍 Monitoring Rules

### File Integrity Monitoring (FIM)
Enabled by default in Wazuh:
- Watches Windows Registry
- Windows File changes in pre defined locations
  (Open the following configuration file: C:\Program Files (x86)\ossec-agent\ossec.conf
  and Add the following entry inside the Directory block: <directories realtime="yes"> Location you want to monitor(Path) </directories> )
- Tracks changes in `/etc/` (Linux)
- Logs visible in **Security Events → FIM**

Imporatant: After any of chanage should restart the Agent

### Login Event Monitoring
- Windows → Security Logs via Agent
- Linux → `/var/log/auth.log`

---

## ✅ Validation
- In Wazuh Dashboard → **Agents** → confirm `WinServer2025`, `Win10PC`, and `UbuntuAgent` are active.
- Generate events (e.g., login failure, file change) → visible in **Security Events**.

---

## 📂 Project Files
- `AWS-SOC-Wazuh-README.md` (this file)
- `AWS-SOC-Wazuh-Project.pdf` (detailed PDF guide)
- `/docs/architecture.png` (diagram — PNG File)
- `/Screenshots of Project` (PNG Files)

---

## 🔒 Best Practices
- Restrict ports with AWS Security Groups
- Use IAM roles for least privilege
- Rotate Wazuh dashboard credentials
- Enable multi-factor authentication (MFA) on AWS account
- Enable daily EBS snapshot backups

---

## 📊 Architecture Diagram
(Will be placed in `/docs/architecture.png`)

##**Swadesh Rawana**  
📍 Cybersecurity & Network Systems Student  
🔗 [LinkedIn Profile](http://linkedin.com/in/swadesh-rawana-297858352)
