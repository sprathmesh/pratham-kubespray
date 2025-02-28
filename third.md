# Kubernetes Cluster Deployment on GCP using Kubespray & Ansible

## Overview
This guide provides a step-by-step process for deploying a Kubernetes cluster on Google Cloud Platform (GCP) using **Kubespray** and **Ansible**. The setup consists of **3 master nodes** and **1 worker node** with high availability (HA) enabled.

---
## 📌 Prerequisites
1. **Google Cloud Platform (GCP) Account**
2. **Compute Engine API Enabled** in GCP
3. **Ansible Installed** on the control node (master1)
4. **SSH Access Enabled** between nodes
5. **Firewall Rules Configured** for Kubernetes networking

---

## 🚀 Step 1: Create VM Instances in GCP
Create 4 VM instances manually in **GCP Console**:

- **Machine Type:** e2-medium or e2-standard-2 (2 vCPUs, 4GB RAM)
- **OS Image:** Ubuntu 22.04 LTS
- **Disk:** 30GB Standard Persistent Disk
- **Network Tag:** `k8s-cluster`
- **Assign Static Internal IPs (Recommended)**

### Instance Names and Internal IPs:
| Name    | Role       | Internal IP     |
|---------|-----------|----------------|
| master1 | Master    | 10.128.0.7      |
| master2 | Master    | 10.128.0.8      |
| master3 | Master    | 10.128.0.9      |
| worker1 | Worker    | 10.128.0.10     |

---

## 🔥 Step 2: Configure Firewall Rules
Manually allow required ports in **GCP Console**:

1. Go to **VPC Network** → **Firewall Rules**
2. Click **Create Firewall Rule**
3. Set **Name:** `k8s-allow`
4. Set **Targets:** Specified target tags → `k8s-cluster`
5. Set **Source:** `0.0.0.0/0`
6. Allowed **Protocols and Ports:**
   ```bash
   tcp:6443,tcp:2379-2380,tcp:10250,tcp:10251,tcp:10252,udp:8472,tcp:51820,tcp:51821
   ```
7. Click **Create**

---

## 🖥️ Step 3: Set Up the Ansible Control Node
Use `master1` as the **Ansible Control Node**.

### 1️⃣ SSH into master1
```bash
ssh user@master1-public-ip
```

### 2️⃣ Install Required Packages
```bash
sudo apt update && sudo apt install -y ansible git python3-pip
```

### 3️⃣ Clone Kubespray Repository
```bash
git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray
pip3 install -r requirements.txt
```

---

## ⚙️ Step 4: Configure Ansible Inventory
### 1️⃣ Copy the Sample Inventory
```bash
cp -rfp inventory/sample inventory/k8s-cluster
```

### 2️⃣ Edit the Inventory File
```bash
nano inventory/k8s-cluster/inventory.ini
```
Modify the inventory with **GCP Private IPs**:
```ini
[all]
master1 ansible_host=10.128.0.7 ip=10.128.0.7
master2 ansible_host=10.128.0.8 ip=10.128.0.8
master3 ansible_host=10.128.0.9 ip=10.128.0.9
worker1 ansible_host=10.128.0.10 ip=10.128.0.10

[kube_control_plane]
master1
master2
master3

[etcd]
master1
master2
master3

[kube_node]
worker1

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr
```

---

## 🔑 Step 5: Configure SSH Access
1️⃣ **Generate SSH Key on master1**
```bash
ssh-keygen -t rsa -b 4096
```

2️⃣ **Copy Key to Other Nodes**
```bash
ssh-copy-id user@10.128.0.8
ssh-copy-id user@10.128.0.9
ssh-copy-id user@10.128.0.10
```
📌 Ensure master1 can SSH into all nodes **without a password**.

---

## 🛠️ Step 6: Configure Ansible Variables
Edit Kubernetes cluster settings:
```bash
nano inventory/k8s-cluster/group_vars/k8s_cluster/k8s-cluster.yml
```
Modify the following:
```yaml
kube_version: v1.31.0
container_manager: containerd
etcd_deployment_type: kubeadm
```

---

## 🚀 Step 7: Deploy Kubernetes with Kubespray
Run the **Ansible playbook** to deploy Kubernetes:
```bash
ansible-playbook -i inventory/k8s-cluster/inventory.ini --become --become-user=root cluster.yml
```
✅ Kubespray will now deploy Kubernetes on all nodes.

---

## 🔍 Step 8: Verify Kubernetes Cluster
### 1️⃣ Check Nodes
```bash
kubectl get nodes -o wide
```
✅ You should see **3 master nodes & 1 worker node** in **Ready** state.

### 2️⃣ Check Running Pods
```bash
kubectl get pods -A
```

### 3️⃣ Check Cluster Health
```bash
kubectl cluster-info
```

---

## 🎯 Summary
✅ **3 Master Nodes (HA Enabled)**
✅ **1 Worker Node**
✅ **Ansible-Based Automation**
✅ **Kubespray Deployment**
✅ **Configured Firewall & Networking**

📌 This setup is **production-ready** and **scalable**! 🚀

Let me know if you need modifications. Happy DevOps-ing! 🤖🔥

