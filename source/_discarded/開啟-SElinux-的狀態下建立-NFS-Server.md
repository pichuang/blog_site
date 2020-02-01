layout: post
title: 開啟 SElinux 的狀態下建立 NFS Server
author: Phil Huang
tags: []
categories: []
date: 2019-11-28 11:02:00
---
遇過非常多人使用 `docekr-distribution` 和 RHEL/CentOS 來建立人生第一個容器映像檔倉庫 (Container Registry) 時，都會遇到 SSL/TLS 相關的問題，大多都是用 [`insecure-registries`][1] 來避掉，今天要來分享自簽憑證的作法

<!--more-->

## 系統規格
- Hostname: nfs.pichuang.local
- Disk
  - /dev/vdb1
- IP: 192.168.77.233/24
- Necessary Software
  - RHEL 7.7
  - nfs-utils.x86_64 1:1.3.0-0.65.el7

## 安裝過程

### 安裝 `nfs-utils`
```bash
yum install nfs-utils -y
systemctl enable nfs-server --now
```

### 開啟防火牆
```bash
firewall-cmd --add-service=nfs --permanent
firewall-cmd --add-service=rpc-bind --permanent
firewall-cmd  --add-service=mountd --permanent
firewall-cmd --reload
```

### 指定硬碟空間
```bash
mkdir /exports
echo "/dev/vdb1 /exports ext4 defaults 0 0" >> /etc/fstab
mount -a
```

### nfs export
```bash
echo "/exports *(rw,no_root_squash)" >> /etc/exports
exportfs -a
```

### 設定 SELinux 規則
```bash
#at client
```