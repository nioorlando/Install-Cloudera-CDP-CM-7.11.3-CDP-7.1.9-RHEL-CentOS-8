# Prerequisites & OS Preparation

Before installing **Cloudera Manager 7.11.3** and **CDP Runtime 7.1.9**, prepare the OS and base dependencies.  
All commands should be executed as **root** (or with `sudo`).

---

## 1. Kernel & OS Tuning 

### 1.1 Disable Transparent Huge Pages (THP)
```bash
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag
echo "echo never > /sys/kernel/mm/transparent_hugepage/enabled" >> /etc/rc.d/rc.local
echo "echo never > /sys/kernel/mm/transparent_hugepage/defrag"  >> /etc/rc.d/rc.local
chmod +x /etc/rc.d/rc.local
```

### 1.2 Set Swappiness
```bash
echo "vm.swappiness = 1" >> /etc/sysctl.conf
sysctl vm.swappiness=1
```

### 1.3 Disable Firewall & iptables
```bash
systemctl disable --now firewalld
systemctl disable --now iptables || true 
```
- Misal ada error: Failed to disable unit: Unit file iptables.service does not exist.
- Lewati saja step ini karena emang niat awal untuk disable jadi kalo tidak ada juga tidak masalah

### 1.4 Set SELinux to Permissive
```bash
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=permissive/' /etc/selinux/config 
```

---

## 2. System Checks 

### 2.1 OS Version
```bash
cat /etc/os-release
```

### 2.2 Python3 Version
```bash
python3 --version
```

### 2.3 Hostname (shortname and FQDN must be consistent)
```bash
hostname
hostname -f
```

---

## 3. Install Dependencies 

### 3.1 Python3.8 Installation 
```bash
dnf -y install python38
python3.8 --version
```

### 3.2 Java (JDK 11 or 1.8)
Cloudera supports Java 8 and Java 11.  
Cek [Support Matrix Cloudera](https://supportmatrix.cloudera.com/).

```bash
dnf -y install java-11-openjdk
java -version
```

### 3.3 PostgreSQL (or another supported DB) 
Cloudera Manager requires a database. You can use:

- PostgreSQL 
- MySQL / MariaDB
- Oracle DB (enterprise)

➡️ See [Install PostgreSQL 14.9](Install_PostgreSQL_14.9.md) for full step-by-step guide.
