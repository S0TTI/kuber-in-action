## ssh key 
```
ssh-keygen
ssh-copy-id -i ~/.ssh/id_rsa.pub  user@host
cat  ~/.ssh/id_rsa.pub
# Copy this key to /root/.ssh/authorized_kesy
ssh user@host
sudo apt install vim -y
sudo vim /root/.ssh/authorized_kesy
sudo vim /root/.ssh/authorized_keys
sudo mv /root/.ssh/authorized_kesy   /root/.ssh/authorized_keys
###edit file /etc/ssh/sshd_config
sudo vim /etc/ssh/sshd_config
###check this line below

#PermitRootLogin prohibit-password
PermitRootLogin yes
#PubkeyAuthentication yes
PubkeyAuthentication yes
# Expect .ssh/authorized_keys2 to be disregarded by default in future.
#AuthorizedKeysFile     .ssh/authorized_keys .ssh/authorized_keys2
AuthorizedKeysFile      .ssh/authorized_keys .ssh/authorized_keys2

###save the file
## restart ssh service
sudo systemctl restart ssh
```
# Test below command you should connect to the host without pass
ssh root@172.16.37.104

## Method 1
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
sudo apt-get install ca-certificates curl vim -y
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

Run the kubespray Docker Images and ***execute command in the container***
```
docker run --rm -it --mount type=bind,source="${HOME}"/.ssh/id_rsa,dst=/root/.ssh/id_rsa \
  quay.io/kubespray/kubespray:v2.27.0 bash

```
#Create Inventory File
cp -rfp inventory/sample inventory/mycluster
vim inventory/mycluster/inventory.ini

Paste the inventory format provided above and modify as per your setup.
#in vim editor can use this command to replace some things
###   :%s/192.168.6.130/172.16.37.101/g
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
## Now run kubespray ansible to install kubernetes in all your node
ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml
## OR use this 
# You will be asked for ssh pass and sudo pass
```
ansible-playbook -i inventory/mycluster/inventory.ini --user geek --become -kK cluster.yml
```

################################################################################
## Method 2
## Kubernetes Installation with Kubespray (on linux with installed python and ansible)

Prerequisites
```
3 Linux nodes with SSH access.

Public and private SSH keys configured for passwordless SSH access.
```
# Clone Kubespray
```
mkdir kubernetes_installation/
cd kubernetes_installation/
git clone https://github.com/kubernetes-sigs/kubespray.git
```
# install requirements for kubespray 
```
sudo apt install python3  python3-pip  python3-venv  -y
python3 -m pip install --upgrade pip setuptools
##https://github.com/kubernetes-sigs/kubespray/blob/master/docs/ansible/ansible.md#installing-ansible
## check your python version and compare with the site documentaion
python3 --version
python --version


python3 -m pip install  ansible
python3 -m pip install  ansible-core

sudo apt update
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt install python3.10 python3.10-venv python3.10-dev -y

VENVDIR=kubespray-venv
KUBESPRAYDIR=kubespray
python3.10 -m venv $VENVDIR
source $VENVDIR/bin/activate
cd $KUBESPRAYDIR
pip install -U -r requirements.txt

##https://github.com/kubernetes-sigs/kubespray/blob/master/docs/ansible/ansible.md#installing-ansible
## check your ansible version
## now check your version
ansible --version
## if not install install with this command
python3 -m pip install "ansible==2.16.4"
pip install "ansible==2.16.4"

```

#Create Inventory File
cp -rfp inventory/sample inventory/mycluster
vim inventory/mycluster/inventory.ini

Paste the inventory format provided above and modify as per your setup.
#in vim editor can use this command to replace some things
###   :%s/192.168.6.130/172.16.37.101/g
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
## Now run kubespray ansible to install kubernetes in all your node
```
ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml
```
## OR use this 
# You will be asked for ssh pass and sudo pass
# for test befor run main command run this command
ansible all -i inventory/mycluster/inventory.ini -u geek -k -b -K -m ping
```
ansible-playbook -i inventory/mycluster/inventory.ini --user geek --become -kK cluster.yml
```
