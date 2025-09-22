# Install Cloudera Manager 7.11.3

This guide describes how to install **Cloudera Manager 7.11.3** and prepare it for managing your **CDP Runtime 7.1.9** cluster.

---

## 1. Configure Cloudera Repository

Download and configure the Cloudera Manager repository file.

```bash
wget https://archive.cloudera.com/p/cm7/7.11.3.0/redhat7/yum/cloudera-manager.repo -O /etc/yum.repos.d/cloudera-manager.repo
```

Edit the repository file `/etc/yum.repos.d/cloudera-manager.repo` and replace `(username):(password)` with your **Cloudera archive credentials**.

```ini
[cloudera-manager]
name=Cloudera Manager 7.11.3.0
baseurl=https://archive.cloudera.com/p/cm7/7.11.3.0/redhat7/yum/
gpgkey=https://archive.cloudera.com/p/cm7/7.11.3.0/redhat7/yum/RPM-GPG-KEY-cloudera
username=(username)
password=(password)
gpgcheck=1
enabled=1
autorefresh=0
type=rpm-md

[postgresql10]
name=Postgresql 10
baseurl=https://archive.cloudera.com/postgresql10/redhat7/
gpgkey=https://archive.cloudera.com/postgresql10/redhat7/RPM-GPG-KEY-PGDG-10
enabled=1
gpgcheck=1
module_hotfixes=true
```

---

## 2. Install Cloudera Manager Server, Agent, and Daemons

```bash
sudo dnf install -y cloudera-manager-daemons cloudera-manager-agent cloudera-manager-server
```

---

## 3. Setup Cloudera Manager Database (PostgreSQL)

Ensure PostgreSQL is installed and running.  
For full setup, refer to: [Install PostgreSQL 14.9](Install_PostgreSQL_14.9.md).

Configure CM database in `/etc/cloudera-scm-server/db.properties` if CM fails to start.

---

## 4. Start Cloudera Manager Services

```bash
systemctl start cloudera-scm-server
systemctl start cloudera-scm-agent
systemctl enable cloudera-scm-server
systemctl enable cloudera-scm-agent
```

Check services:
```bash
systemctl status cloudera-scm-server
systemctl status cloudera-scm-agent
```

Access the web UI on:
```
http://(hostname):7180
```

---

## 5. Fix OS Release Compatibility (if CM Agent Fails)

Sometimes CM agent may not recognize the OS version.  
Backup and modify `/usr/lib/os-release` and `/etc/os-release` to trick detection.

### Backup first
```bash
cp /usr/lib/os-release /usr/lib/os-release.back
cp /etc/os-release /etc/os-release.back
```

### Inject RHEL 8.8 values
```bash
cat > /usr/lib/os-release <<EOF
NAME="Red Hat Enterprise Linux"
VERSION="8.8 (Ootpa)"
ID="rhel"
ID_LIKE="rhel fedora"
VERSION_ID="8.8"
PRETTY_NAME="Red Hat Enterprise Linux 8.8 (Ootpa)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:redhat:enterprise_linux:8.8:GA"
HOME_URL="https://www.redhat.com/"
BUG_REPORT_URL="https://bugzilla.redhat.com/"
REDHAT_BUGZILLA_PRODUCT="Red Hat Enterprise Linux 8"
REDHAT_BUGZILLA_PRODUCT_VERSION=8.8
REDHAT_SUPPORT_PRODUCT="Red Hat Enterprise Linux"
REDHAT_SUPPORT_PRODUCT_VERSION="8.8"
EOF

# Restart CM agent
systemctl restart cloudera-scm-agent
```

### Restore original files after CM installation/parcels are complete
```bash
mv /usr/lib/os-release.back /usr/lib/os-release
mv /etc/os-release.back /etc/os-release
systemctl restart cloudera-scm-agent
```

---

## 6. Verify Installation

- Ensure database is running.  
- Ensure CM agent and server are both active.  
- Access Cloudera Manager UI via port `7180`.

---

## 📷 Reference Screenshots

Screenshots should be referenced like this:

- ![Download Repo CM](images/download_cm_repo.png)
- ![Configure Repo](images/configure_repo.png)
- ![Cloudera Manager Login](images/cm_login.png)
- ![Add Private Cloud Base Cluster](images/cm_add_cluster.png)
- ![Parcel Repo Setup](images/cm_parcel_repo.png)
- ![Inspect Cluster](images/cm_inspect.png)

> Replace with your own captured screenshots in the `images/` folder.

---
