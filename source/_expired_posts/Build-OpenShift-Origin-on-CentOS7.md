layout: build
title: Build OpenShift Origin on CentOS7
date: 2018-03-07 22:53:16
tags:
- centos
- openshift
---

## Objectives
- 3 OpenShift master
    - pichuang{1,2,4}.on.ec
- 3 OpenShfit node
    - pichuang{5,6,7}.on.ec
- 3 ETCD server
    - pichuang{1,2,4}.on.ec

## Install Steps
### Fresh Install CentOS7
1. Please choose minimal installation
2. `DO NOT disable` selinux feature after complete installation

<!-- more -->

### Host Preparation include OpenShift master and nodes
{% codeblock %}
cat << EOF > /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.101.61 pichuang1 pichuang1.on.ec
192.168.101.62 pichuang2 pichuang2.on.ec openshift.on.ec
192.168.101.66 pichuang4 pichuang4.on.ec
192.168.101.67 pichuang5 pichuang5.on.ec
192.168.101.68 pichuang6 pichuang6.on.ec
192.168.101.69 pichuang7 pichuang7.on.ec
EOF

ssh-keygen -b 2048 -t rsa -f /root/.ssh/id_rsa -q -N ""
ssh-copy-id -i ~/.ssh/id_rsa.pub root@pichuang1.on.ec -oStrictHostKeyChecking=no
ssh-copy-id -i ~/.ssh/id_rsa.pub root@pichuang2.on.ec -oStrictHostKeyChecking=no
ssh-copy-id -i ~/.ssh/id_rsa.pub root@pichuang4.on.ec -oStrictHostKeyChecking=no
ssh-copy-id -i ~/.ssh/id_rsa.pub root@pichuang5.on.ec -oStrictHostKeyChecking=no
ssh-copy-id -i ~/.ssh/id_rsa.pub root@pichuang6.on.ec -oStrictHostKeyChecking=no
ssh-copy-id -i ~/.ssh/id_rsa.pub root@pichuang7.on.ec -oStrictHostKeyChecking=no
{% endcodeblock %}

### Only for `pichuang1.on.ec`

All operation just work on `pichuang1.on.ec`. Don't take care another OpenShift node.

{% codeblock %}
# Install package
yum update -y
yum install -y iptables ansible openshift-ansible openshift-ansible-playbooks
{% endcodeblock %}
- Openshift use `iptables`, not `firewalld` 
- Ansible playbook is your good friend whne you dont want to step by step install

### Config Inventory file in Ansible
{% codeblock %}
cat << EOF > /etc/ansible/hosts
[OSEv3:children]
masters
nodes
etcd

[OSEv3:vars]
ansible_ssh_user=root
deployment_type=origin
openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider', 'filename': '/etc/origin/master/htpasswd'}]
openshift_release=v3.5
openshift_master_default_subdomain='apps.on.ec'
openshift_master_cluster_method=native
openshift_master_cluster_hostname=openshift.on.ec
openshift_master_cluster_public_hostname=openshift.on.ec

[masters]
pichuang1.on.ec
pichuang2.on.ec
pichuang4.on.ec

[etcd]
pichuang1.on.ec
pichuang2.on.ec
pichuang4.on.ec

[nodes]
pichuang1.on.ec
pichuang2.on.ec
pichuang4.on.ec
pichuang5.on.ec openshift_node_labels="{'app':'true','region': 'infra', 'zone': 'default'}"
pichuang6.on.ec openshift_node_labels="{'app':'true','region': 'infra', 'zone': 'default'}"
pichuang7.on.ec openshift_node_labels="{'app':'true','region': 'infra', 'zone': 'default'}"

EOF
{% endcodeblock %}


### Install OpenShift
{% codeblock %}
ansible-playbook /usr/share/ansible/openshift-ansible/playbooks/byo/config.yml
{% endcodeblock %}
- You may wait 1hr+ if you don't have local mirror

## After Install OpenShift
### Create new account and assgin permission
{% codeblock %}
htpasswd /etc/origin/master/htpasswd pichuang
oadm policy add-cluster-role-to-user cluster-admin pichuang
{% endcodeblock %}

### Login and Check Status
{% codeblock %}
oc login -u redhat -n default
oc get nodes
oc get all -o wide
{% endcodeblock %}


