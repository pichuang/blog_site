layout: post
title: 自定義 QCOW2 映像檔
author: Phil Huang
tags:
  - qcow
  - kvm
  - libvirt
  - infra
categories:
  - infra
date: 2019-11-18 23:04:00
toc: true
---
virt-customize 主要是協助自定義 QCOW2 (QEMU Copy On Write) 映像檔的工具，由 `libguestfs-tools` 提供檔案。該格式經常用於 KVM、OpenStack、oVirt 等以 KVM 為基礎的虛擬化平台，該修改方式也為 Red Hat 官方所正式支援的方式，本文將提供下列常用自定義 qcow2 映像檔之指令使用方式

<!--more-->

## 安裝 virt-customize
```bash
yum install -y libvirt libguestfs libguestfs-tools
virt-customize -V 
```

### 1. 更新 root 密碼
```bash
# virt-customize -a ./rhel-server-7.7-update-2-x86_64-kvm.qcow2 --root-password password:<YOUR PASSWORD>

virt-customize -a ./rhel-server-7.7-update-2-x86_64-kvm.qcow2 --root-password password:root
```

### 2. 執行特定指令
```bash
# virt-customize -a ./rhel-server-7.7-update-2-x86_64-kvm.qcow2 --run-command '<COMMAND>'

virt-customize -a ./rhel-server-7.7-update-2-x86_64-kvm.qcow2 --run-command 'subscription-manager repos --disabled=*'
```

### 3. 安裝特定套件
```bash
virt-customize -a ./rhel-server-7.7-update-2-x86_64-kvm.qcow2 --install vim,git,wget,curl,htop
```

### 4. 上傳檔案
```bash
virt-customize -a ./rhel-server-7.7-update-2-x86_64-kvm.qcow2 --upload redhat.repo:/etc/yum.repos.d/redhat.repo
```

### 5. 設定時區
```
virt-customize -a ./rhel-server-7.7-update-2-x86_64-kvm.qcow2  --timezone "Asia/Taipei"
```

### 6. 上傳 SSH Public Key
```bash
virt-customize -a ./rhel-server-7.7-update-2-x86_64-kvm.qcow2 --ssh-inject root:file:./id_rsa.pub
```

### 7. Relabel SELinux
```bash
virt-customize -a ./rhel-server-7.7-update-2-x86_64-kvm.qcow2 --selinux-relabel
```