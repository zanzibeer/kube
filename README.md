# ru.sadkomed.kubernetes

## Supported OS

- Debian 11

## Required Ansible on bootstrap
Ansible version required `2.10+`

-----

**Prepare all nodes cluster kubernetes on Debian 11:**

- Install sudo

```
apt install sudo
```

- Add current user into sudo group

```
usermod -aG sudo user
```

-----

## Tasks in the role KUBERNETES-BOOTSTRAP

This role contains tasks to:

- Install basic packages required
- Setup standard system requirements - Disable Swap, Configure timezone, Modify sysctl, Disable SELinux
- Install and configure a container runtime of your choice - **CRI-O** or **Containerd**
- Install the Kubernetes packages - kubelet, kubeadm and kubectl
- Install and configure Helm


## Tasks in the role KUBERNETES-INITIALIZE

This role contains tasks to:

- Initialize k8s cluster
- Install CNI
- Add ControlPlane's to cluster
- Add worker nodes to cluster


## How to use this roles

- Configure `/etc/hosts` file in your bastion or workstation with all nodes and ip addresses. Example:

```
192.168.200.10 k8s-master01.example.com k8s-master01
192.168.200.11 k8s-master02.example.com k8s-master02
192.168.200.12 k8s-master03.example.com k8s-master03

192.168.200.13 k8s-node01.example.com k8s-node01
192.168.200.14 k8s-node02.example.com k8s-node02
192.168.200.15 k8s-node03.example.com k8s-node03
192.168.200.16 k8s-ingress01.example.com k8s-ingress01

```
- Clone the Project:
```
# git clone git@gitbranch.ru:SADKOMED/ru.sadkomed.kubernetes.git
```

- Enter to project directory

```
# cd ru.sadkomed.kubernetes
```


- Copy inventory from example-file && update your inventory:

```
cp -r hosts_example hosts
$ vim hosts
[k8snodes]
k8s-master01
k8s-master02
k8s-master03
k8s-node01
k8s-node02
k8s-node03
k8s-node04
```

- Update variables in playbook file

```
$ vim k8s-prep.yml
---
- name: Setup
  hosts: k8snodes
  remote_user: root
  become: yes
  become_method: sudo
  #gather_facts: no
  vars:
    k8s_version: "1.23.3"                                # Kubernetes version to be installed / https://kubernetes.io/releases/
    crictl_version: "1.23.0"                             # Crictl version to be installed / https://github.com/kubernetes-sigs/cri-tools/releases
    nerdctl_version: "0.16.1"                            # Nerdctl version to be installed / https://github.com/containerd/nerdctl/releases
    selinux_state: permissive                            # SELinux state to be set on k8s nodes
    timezone: "Europe/Moscow"                            # Timezone to set on all nodes
    k8s_cni: "calico"                                    # calico, flannel
    container_runtime: "containerd"                      # cri-o, containerd
    pod_network_cidr: "10.244.0.0/16"                    # pod subnet
    configure_firewalld: false                           # true / false (keep it false, k8s>1.19 have issues with firewalld)
    username_sudo: "user"                                # username for connection with sudo
    helm_release: "3.8.0"                                # Helm version to be installed / https://github.com/helm/helm/releases
    balancer_address: "api.k8s.local"                    # address of load balancer
    # Docker proxy support
    setup_proxy: false                                   # Set to true to configure proxy
    proxy_server: "proxy.example.com:8080"               # Proxy server address and port
    docker_proxy_exclude: "localhost,127.0.0.1"          # Adresses to exclude from proxy
  roles:
    - kubernetes-bootstrap
    - kubernetes-initialize
```

If you are using non root remote user, then set username and enable sudo:

```
become: yes
become_method: sudo
```

- Install ansible on **bootstrap-server**, then run command for install package community modules:

```
ansible-galaxy collection install community.general
```

- Run **playbook_ssh.yml** for add your key-file in all cluster nodes to connect ssh over pubkey:

```
ansible-playbook playbook_ssh.yml --ask-pass --ask-become-pass
```

- Run playbook to install cluster KUBERNETES on all nodes:
```
ansible-playbook k8s-prep.yml
```

- Check the success init Kube cluster after start **kubernetes-initialize** role on k8s-master01

```
watch kubectl get nodes -o wide
```

- Check the success install and running state of all pods on all cluster nodes

```
watch kubectl get pods -A
```

- Mark the worker nodes with label "node" on k8s-master01

```
kubectl label node k8s-node01 node-role.kubernetes.io/node=""
kubectl label node k8s-node02 node-role.kubernetes.io/node=""
kubectl label node k8s-node03 node-role.kubernetes.io/node=""
```
