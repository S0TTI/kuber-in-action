##Metod 1
## Kubernetes Installation with Kubespray (Docker Image)
## This guide explains how to install Kubernetes on a 3-node cluster using Kubespray with its official Docker image.

Prerequisites
```
3 Linux nodes with SSH access.

Docker installed in a manager linux

Public and private SSH keys configured for passwordless SSH access.
```

Installing docker
```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl -y
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

Run the kubespray Docker Images
```
docker run --rm -it --mount type=bind,source="$(pwd)"/inventory/sample,dst=/inventory \
  --mount type=bind,source="${HOME}"/.ssh/id_rsa,dst=/root/.ssh/id_rsa \
  quay.io/kubespray/kubespray:v2.27.0 bash

```


Steps to Install Kubernetes

1. Clone Kubespray Repository

git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray

2. Create Inventory File

cp -rfp inventory/sample inventory/mycluster
vi inventory/mycluster/hosts.ini

Paste the inventory format provided above and modify as per your setup.

Inventory File Format
```

[kube_control_plane]
node1 ansible_host=192.168.6.130 ip=192.168.6.130 etcd_member_name=etcd1
node2 ansible_host=192.168.6.131 ip=192.168.6.131 etcd_member_name=etcd2
node3 ansible_host=192.168.6.132 ip=192.168.6.132 etcd_member_name=etcd3

[etcd:children]
kube_control_plane

[kube_node]
node1 ansible_host=192.168.6.130 ip=192.168.6.130
node2 ansible_host=192.168.6.131 ip=192.168.6.131
node3 ansible_host=192.168.6.132 ip=192.168.6.132

[k8s_cluster:children]
kube_control_plane
kube_node

```

##Metod 2
## Kubernetes Installation with Kubespray (on linux with installed python and ansible)

Prerequisites
```
3 Linux nodes with SSH access.

Python and Ansible installed.

Docker installed on the control node.

Public and private SSH keys configured for passwordless SSH access.
```
Installing ansible
```
sudo apt update && sudo apt upgrade -y
sudo apt install ansible  git  -y

```

# Clone Kubespray
```
mkdir kubernetes_installation/
cd kubernetes_installation/
git clone https://github.com/kubernetes-sigs/kubespray.git
cd kubespray/
```
# install requirements for kubespray 
```
apt update && apt install python3-pip ansible
pip install -r requirements.txt  --break-system-packages
pip3 install ruamel.yaml --break-system-packages
# Run it on ALL Nodes
apt install sshpass -y  
```

Create Inventory File
cp -rfp inventory/sample inventory/mycluster
vi inventory/mycluster/hosts.ini

Paste the inventory format provided above and modify as per your setup.

Inventory File Format
```

[kube_control_plane]
node1 ansible_host=192.168.6.130 ip=192.168.6.130 etcd_member_name=etcd1
node2 ansible_host=192.168.6.131 ip=192.168.6.131 etcd_member_name=etcd2
node3 ansible_host=192.168.6.132 ip=192.168.6.132 etcd_member_name=etcd3

[etcd:children]
kube_control_plane

[kube_node]
node1 ansible_host=192.168.6.130 ip=192.168.6.130
node2 ansible_host=192.168.6.131 ip=192.168.6.131
node3 ansible_host=192.168.6.132 ip=192.168.6.132

[k8s_cluster:children]
kube_control_plane
kube_node

```
# You will be asked for ssh pass and sudo pass
```
ansible-playbook -i inventory/mycluster/hosts.yaml --user geek --become -kK cluster.yml
```
