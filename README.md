# 📘 Install Cloudera CDP (CM 7.11.3 + CDP 7.1.9) – RHEL/CentOS 8

This repository is a **step-by-step installation guide** for setting up **Cloudera Manager 7.11.3** and **CDP Private Cloud Base 7.1.9** on **RHEL/CentOS 8**.  
The goal is to provide a **clear, repeatable playbook** for installation and validation.

---

## 📖 Contents

1. [Prerequisites & OS Preparation](prerequisites.md)  
   - Disable Transparent Huge Pages (THP)  
   - Adjust swappiness  
   - Disable firewall & set SELinux permissive  
   - Install Python 3.8, Java 11, PostgreSQL JDBC  

2. [Install Cloudera Manager & CDP Runtime](install_cm.md)  
   - Configure Cloudera repo  
   - Install CM server, agent, and daemons  
   - Setup CM database (PostgreSQL)  
   - Start CM services and verify UI  
   - Deploy CDP 7.1.9 parcels  
   - Add cluster via CM wizard  
   - Assign services and role configuration  
   - First run and health checks  

---

## 📂 Repository Structure
```text
Install-Cloudera-CDP-CM-7.11.3-CDP-7.1.9-RHEL-CentOS-8/
├── README.md
├── Prerequisites_OS_Preparation.md
├── install_cm.md
├── Install_PostgreSQL_14.9.md
└── images/ 
```
---

## 🛠 Requirements
- OS: RHEL/CentOS 8 (Stream supported with adjustments)  
- Java: OpenJDK 11  
- Python: 3.8  
- Database: PostgreSQL 10+ (example uses PostgreSQL 14)  
- Internet access to Cloudera repos (Cloudera subscription required)  
