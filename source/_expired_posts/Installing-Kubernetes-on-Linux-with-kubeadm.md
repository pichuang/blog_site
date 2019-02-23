---
layout: post
title: Installing Kubernetes on Linux with kubeadm
date: 2017-05-24 09:38:10
updated: 2017-05-24 09:38:10
category:
- Infra
- Container
tags:
- k8s
- kubernetes
---

## Objectives
- Install a secure Kubernetes cluster on 3 machines
- Install a pod network -- [Calico][3] on the cluster so that application components (pods) can talk to each other

## Prerequisites
- Master: k8s-1.on.ec
    - Ubuntu 16.04
    - 192.168.100.52
- Worker: k8s-2.on.ec
    - Ubuntu 16.04
    - 192.168.100.53
- Worker: k8s-3.on.ec
    - Ubuntu 16.04 
    - 192.168.100.54
- Software
    - Kuburnetes: v1.7.0
    - Calico: v2.3

<!--more-->

## Instructions
### Installing {docker-ce, kubelet, kubeadm, kubectl, kubernetes-cni} on hosts (Include k8s-{1-3}.on.ec)
```bash
#!/bin/bash
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y \
    linux-image-extra-$(uname -r) \
    linux-image-extra-virtual
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
apt-get update
apt-get install -y docker-ce kubelet kubeadm kubectl kubernetes-cni
```

### Initializing master (k8s-1.on.ec)
Depended on [Calico Requirements and Limitations][4]

```bash
sudo kubeadm init --pod-network-cidr 192.168.0.0/16 --service-cidr 10.96.0.0/12 --service-dns-domain "on.ec" --apiserver-advertise-address 192.168.100.52
```
- Tricky
  - [weave-net CrashLoopBackOff for the second node #34101][7]

### Loading k8s config
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
export KUBECONFIG=$HOME/.kube/config
```
- Make a record of the kubeadm join command that kubeadm init outputs
    - `kubeadm join --token 5b706a.5c9c84f7de52d5dc 192.168.100.52:6443`

### Join new worker node (k8s-{2-3}.on.ec) into master node (k8s-1.on.ec)
```bash
#In worker node
sudo kubeadm join --token d7a85b.84a4669feb85b510 192.168.100.52:6443
```

### Kubeadm hosted install Calico
```bash
kubectl apply -f http://docs.projectcalico.org/v2.3/getting-started/kubernetes/installation/hosted/kubeadm/1.6/calico.yaml
```
- Make sure you install the add-on. If not, you will get status 'NotReady'.

### Check status
```bash
$ kubectl get nodes
NAME          STATUS    AGE       VERSION
k8s-1.on.ec   Ready     1h        v1.7.0
k8s-2.on.ec   Ready     1h        v1.7.0
k8s-3.on.ec   Ready     1h        v1.7.0
```

## Tear down
```bash
# At master node
kubectl drain k8s-1.on.ec --delete-local-data --force --ignore-daemonsets
kubectl delete node k8s-1.on.ec
kubectl drain k8s-2.on.ec --delete-local-data --force --ignore-daemonsets
kubectl delete node k8s-2.on.ec
kubectl drain k8s-3.on.ec --delete-local-data --force --ignore-daemonsets
kubectl delete node k8s-3.on.ec

# Every nodes
kubeadm reset
```

## Reference
- [Kubernetes - kubeadm][1]
- [kubectl-cheatsheet][2]
- [Calico][3]
- [Install calico by kubeadm][4]
- [kubectl user guide][6]

[1]: https://kubernetes.io/docs/getting-started-guides/kubeadm/
[2]: https://github.com/kubernetes/kubernetes/blob/master/docs/user-guide/kubectl-cheatsheet.md
[3]: https://www.projectcalico.org/
[4]: http://docs.projectcalico.org/v2.0/getting-started/kubernetes/installation/hosted/kubeadm/
[5]: http://docs.projectcalico.org/v2.2/getting-started/kubernetes/installation/hosted/kubeadm/1.6/calico.yaml
[6]: https://kubernetes.io/docs/user-guide/kubectl/v1.6/
[7]: https://github.com/kubernetes/kubernetes/issues/34101#issuecomment-253235083