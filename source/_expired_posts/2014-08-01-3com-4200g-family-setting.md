---
layout: post
title: '3COM 4200g Family 設定'
date: 2014-08-01 12:11
comments: true
categories: 
---
## 3COM Reset
```
登入迅速按 ctrl + b
選 7. Skip current configuration file
選 Y 確定
選 0. Reboot
```


## 設定 management ip
```
system-view
interface vlan 1
ip address <switch_ip> <switch_netmask>
quit
ip route-static 0.0.0.0 0.0.0.0 <gateway>
save
```

## 改密碼
```
local-user admin
password cipher 1234567
save
```

## 改 hostname 
```
sysname mptcp
```

## Link Aggregation 
  1. setting 
```
link-aggregation group 1 mode manual
interface GigabitEthernet 1/0/1
port link-aggregation group 1
interface GigabitEthernet 1/0/2
port link-aggregation group 1
interface GigabitEthernet 1/0/3
```

  2. view
```
display interface Giga
```
    - ```It belongs to a link-aggregation. It is the master port```

## 更新韌體 (以下在 User-view)
  1. 下載 [官網韌體載點](https://h10145.www1.hp.com/Downloads/SoftwareReleases.aspx?ProductNumber=JE015A&lang=en&cc=us&prodSeriesId=4236411)
    - select s3t03_02_07s168
    - 解壓縮後取 s3t03_02_07s168.app s3v02_10.web s3u02_04.btm

  2. tftp upload
```
tftp <tftp_server_ip> get s3t03_02_07s168.app
tftp <tftp_server_ip> get s3v02_10.web
tftp <tftp_server_ip> get s3u02_04.btm
```

  3. 設定 boot
```
boot web-package s3v02_10.web main
boot boot-loader s3t03_02_07s168.app
boot bootrom s3u02_04.btm
```

  4. save & reboot 
  
  5. check
```
display boot-loeader
```

## 設定 ssh 
```
rsa local-key-pair create
2048
user-interface vty 0 4
authentication-mode scheme
protocol inbound ssh
local-user admin
password cipher MySSHpassword
service-type ssh
quit
ssh user admin authentication-type password
save
```


## 其他
  1. show running-config
```
display current-configuration
```

# Reference 
  - [Enable SSH and remove Telnet on a 3COM 4200G Switch Script](http://kendrickcoleman.com/index.php/Tech-Blog/enable-ssh-and-remove-telnet-on-a-3com-4200g-switch-script.html)