# 🐳 Deploying Ceph Cluster using Docker & Cephadm

> A quick-start guide to deploying a Ceph cluster using **Docker** and **Cephadm** on Ubuntu-based systems.

---

## ⚙️ Prerequisites
Install required packages on **all nodes**:

```bash
sudo apt update
sudo apt install -y docker.io curl lvm2
sudo systemctl enable --now docker
```

> 💡 Tip: Each node should have two disks — one for OS (e.g., 50GB) and one for Ceph OSD (e.g., 1TB).

---

## 📍 IP and Hostname Planning
Update `/etc/hosts` on all nodes:

```
192.168.1.31    ceph-admin
192.168.1.32    ceph-n1
192.168.1.33    ceph-n2
192.168.1.34    ceph-n3
```

---

## 1️⃣ Setup Cephadm on Admin Node

### On `ceph-admin` node:
```bash
curl --silent --remote-name https://raw.githubusercontent.com/ceph/ceph/main/src/cephadm/cephadm
chmod +x cephadm
sudo mv cephadm /usr/local/bin/
```

### Add Ceph Repo and Install Common Tools
```bash
sudo cephadm add-repo --release quincy
sudo cephadm install ceph-common
```

---

## 2️⃣ Bootstrap the Cluster
```bash
sudo cephadm bootstrap \
  --mon-ip 192.168.1.31 \
  --initial-dashboard-user admin \
  --initial-dashboard-password admin123
```

> This sets up MON, MGR, and Dashboard on the admin node.

---

## 3️⃣ Add Other Nodes to the Cluster
```bash
sudo ceph orch host add ceph-n1 192.168.1.32
sudo ceph orch host add ceph-n2 192.168.1.33
sudo ceph orch host add ceph-n3 192.168.1.34
```

### Label nodes for OSDs:
```bash
sudo ceph orch host label add ceph-n1 osd
sudo ceph orch host label add ceph-n2 osd
sudo ceph orch host label add ceph-n3 osd
```

---

## 4️⃣ Deploy OSDs on Each Node
### First, list available devices:
```bash
sudo ceph orch device ls
```

### Then deploy:
```bash
sudo ceph orch daemon add osd ceph-n1:/dev/sdb
sudo ceph orch daemon add osd ceph-n2:/dev/sdb
sudo ceph orch daemon add osd ceph-n3:/dev/sdb
```

---

## 5️⃣ Access Ceph Dashboard
- Visit: `http://192.168.1.31:8080`
- Username: `admin`
- Password: `admin123`

If dashboard module isn't enabled:
```bash
sudo ceph mgr module enable dashboard
```

---

## 📦 Deploy RGW (Object Gateway)
```bash
sudo ceph orch apply rgw default --placement="1 ceph-admin"
```

### Create S3 admin user:
```bash
radosgw-admin user create --uid=admin --display-name="Admin" --system
```

---

## 🔍 Check Cluster Health
```bash
sudo ceph -s
```

---

## 📌 Notes
- This guide is designed for **test and development** setups.
- For production, consider advanced configuration and monitoring.

---

> Created by [Mohammad Talebi](https://linkedin.com/in/mtlbd) – DevOps Engineer 👨‍💻
