---
layout: post
title: "維護 ArchLinux"
description: ""
date: 2015-07-21 00:00:00 +0800
updated: 2015-07-21 00:00:00 +0800
category:
- Infra
- Linux
tags: 
- linux
- infra
- archlinux
---

### 更新系統
> pacman -Syu  
pacdiff  
systemctl daemon-reload  

### 確認服務正常與否
> systemctl --faied

### 檢查 journal 是否有異常
> journalctl -f  
\# 檢查 unit 是否有異常  
journalctl -u sshd  
\# 檢查特定時間的 jorunal  
journalctl --since "2015-07-21 00:00:00"  



### Reference
- [pacman tips](https://wiki.archlinux.org/index.php/Pacman_tips)
- [System maintenance](https://wiki.archlinux.org/index.php/System_maintenance)
- [journalctl](https://wiki.archlinux.org/index.php/Systemd#Journal)
- [How To Use Journalctl to View and Manipulate Systemd Logs](https://www.digitalocean.com/community/tutorials/how-to-use-journalctl-to-view-and-manipulate-systemd-logs)
