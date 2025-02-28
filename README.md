# pratham-kubespray

----------------------------------------------------

# Kubernetes Cluster Deployment using Kubespray & Ansible

This guide details how to set up a Kubernetes cluster using **Kubespray** and **Ansible** on Ubuntu servers.

## Prerequisites
- Ubuntu 22.04 LTS on all nodes
- Ansible installed on the master node
- SSH key-based authentication enabled between nodes

## Step 1: Setup SSH Access
Ensure all master and worker nodes have proper SSH key-based authentication.
```bash
# On master2, copy public key to authorized_keys
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# Test SSH access
ssh root@master1
```

## Step 2: Verify IP Addresses
```bash
hostname -I  # Check the assigned IP of your node
```

## Step 3: Clone Kubespray Repository
```bash
git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray
```

## Step 4: Setup Ansible Inventory
Create an inventory file listing your cluster nodes.
```bash
mkdir -p inventory/mycluster
cp -rfp inventory/sample inventory/mycluster
```
Modify the inventory file to include your nodes.
```bash
nano inventory/mycluster/hosts.yaml
```
Example `inventory.ini`:
```ini
[all]
master1 ansible_host=10.128.0.11 ip=10.128.0.11
master2 ansible_host=10.128.0.12 ip=10.128.0.12
master3 ansible_host=10.128.0.13 ip=10.128.0.13
worker1 ansible_host=10.128.0.14 ip=10.128.0.14

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

[k8s_cluster:children]
kube_control_plane
kube_node
```

## Step 5: Deploy Kubernetes Cluster with Ansible
Run the following command to deploy the cluster:
```bash
ansible-playbook -i inventory/k8s-cluster/inventory.ini cluster.yml --become --become-user=root
```

## Step 6: Verify Kubernetes Installation
After successful deployment, verify the cluster status:
```bash
kubectl get nodes
```

## Debugging & System Info Commands
If any issues occur, check system info:
```bash
ls
pwd
df -h /
ls -l /home/ubuntu/kubespray/inventory/k8s-cluster/inventory.ini
```

## Notes
- Ensure you have **kubectl** installed on the control plane nodes.
- If SSH access fails, manually copy the SSH key using:
  ```bash
  ssh-copy-id root@masterX
  ```
- If playbook execution fails, check Ansible logs and rerun the playbook.
- after playbook run succusefuly then check kubectl get node
- If you want to use master node also to deploy cluster then untaint the master nodes
```bash
kubectl describe node master1 | grep Taint
kubectl describe node master2 | grep Taint
kubectl describe node master3 | grep Taint
kubectl describe node worker1 | grep Taint
```
- After check the taint status
- If you want to untaint Nodes
```bash
kubectl taint nodes master1 node-role.kubernetes.io/control-plane:NoSchedule-
kubectl taint nodes master2 node-role.kubernetes.io/control-plane:NoSchedule-
kubectl taint nodes master3 node-role.kubernetes.io/control-plane:NoSchedule-
kubectl taint nodes worker1 node-role.kubernetes.io/control-plane:NoSchedule-
```
- Your Taints are untaint properly to check run again - *IF YOU WANT
```bash
kubectl describe node master1 | grep Taint
kubectl describe node master2 | grep Taint
kubectl describe node master3 | grep Taint
kubectl describe node worker1 | grep Taint
````

## References
- [Kubespray GitHub Repo](https://github.com/kubernetes-sigs/kubespray)
- [Kubernetes Official Docs](https://kubernetes.io/docs/)
