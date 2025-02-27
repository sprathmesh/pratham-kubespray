# Kubernetes Cluster Deployment with Kubespray

This README provides step-by-step commands for deploying a Kubernetes cluster using Kubespray with Ansible.

## Prerequisites

- Ubuntu 22.04 (or compatible Linux OS) on all nodes
- SSH access between nodes
- Ansible installed on the master node

## Step 1: Clone Kubespray Repository

```sh
cd /home/ubuntu/
git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray
```

## Step 2: Install Dependencies

```sh
pip install -r requirements.txt
```

## Step 3: Configure Inventory

Copy the sample inventory for customization:

```sh
cp -rfp inventory/sample inventory/k8s-cluster
```

Edit the inventory file with node details:

```sh
nano inventory/k8s-cluster/inventory.ini
```

Example configuration:

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

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr
```

## Step 4: Setup SSH Keys

On the master node, generate an SSH key (if not already created):

```sh
ssh-keygen -t rsa -b 4096
```

Copy the SSH key to all nodes:

```sh
ssh-copy-id root@10.128.0.11
ssh-copy-id root@10.128.0.12
ssh-copy-id root@10.128.0.13
ssh-copy-id root@10.128.0.14
```

## Step 5: Deploy the Cluster

Run the following command to start the deployment:

```sh
ansible-playbook -i inventory/k8s-cluster/inventory.ini cluster.yml --become --become-user=root
```

Deployment time may vary depending on your network and system resources.

## Step 6: Verify Kubernetes Cluster

After deployment, check the cluster status:

```sh
kubectl get nodes
```

If `kubectl` is not found, configure it:

```sh
export KUBECONFIG=/etc/kubernetes/admin.conf
```

## Additional Commands

### Check Disk Space
```sh
df -h /
```

### Reset Cluster (If Needed)
```sh
ansible-playbook -i inventory/k8s-cluster/inventory.ini reset.yml --become --become-user=root
```

### Scale the Cluster
```sh
ansible-playbook -i inventory/k8s-cluster/inventory.ini scale.yml --become --become-user=root
```

## Conclusion

This setup ensures a highly available Kubernetes cluster using Kubespray. Modify configurations as needed for your use case. 🚀
