---
layout: post
title: "ONOS on CentOS7"
description: "Single ONOS instance"
date: 2015-07-27 00:00:00 +0800
updated: 2015-07-27 00:00:00 +0800
category: sdn
tags:
- sdn
- onos
---

這篇是紀錄建立單台 onos controller 的文章

## Enviroment

OS: CentOS7 x86_64  
Network: 192.168.59.202  
Account: root  

## Prepare

```bash
yum install -y wget net-tools
```

## Firewalld setting

```bash
systemctl start firewalld
firewall-cmd --zone=public --add-port=8181/tcp --permanent
firewall-cmd --zone=public --add-port=6633/tcp --permanent
```
* Port 8181: ONOS Web server listen port
* Port 6633: OpenFlow control plane listen port
* 可利用 `firewall-cmd --zone=public --list-all` 確認ports狀況

## Download ONOS Source 
```bash
cd ~/
git clone https://gerrit.onosproject.org/onos
```

## Make directory
```bash
cd ~ && mkdir Downloads Applications
cd Downloads
```
* 必要建立, 後面會用到

## Install JAVA 8
```bash
wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u45-b14/jdk-8u45-linux-x64.rpm"
rpm -ivh jdk-8u45-linux-x64.rpm
```

## Download Karaf & Maven
```bash
wget http://archive.apache.org/dist/karaf/3.0.5/apache-karaf-3.0.5.tar.gz
wget https://archive.apache.org/dist/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
tar -zxvf apache-karaf-3.0.5.tar.gz -C ../Applications/
tar -zxvf apache-maven-3.3.9-bin.tar.gz -C ../Applications/ 
```
* 建議不要自己改版本, 因為有相依性的問題

## bashrc setting
```bash
cat >> ~/.bashrc << EOF
export ONOS_ROOT=~/onos
source $ONOS_ROOT/tools/dev/bash_profile
export JAVA_HOME=/usr/java/jdk1.8.0_45/
export JRE_HOME=/usr/java/jdk1.8.0_45/jre/
export KARAF_ROOT=/root/Applications/apache-karaf-3.0.5
export M2_HOME=/root/Applications/apache-maven-3.3.9
export ONOS_USER=root
export ONOS_GROUP=root
export ONOS_CELL=sdnds-tw
EOF

source ~/.bashrc
```
  * ONOS_CELL 可自行替換成自己的 cell environment name
  * [Execution sequence for .bash_profile, .bashrc, .bash_login, .profile and .bash_logout](http://www.thegeekstuff.com/2008/10/execution-sequence-for-bash_profile-bashrc-bash_login-profile-and-bash_logout) 


## Build ONOS

### 編譯 onos

```bash
cd ~/onos
mvn clean install
```

### Create a cell definition
```bash
cat >> ~/onos/tools/test/cells/sdnds-tw << EOF
# ONOS from Scratch tutorial cell

# Cell name
export ONOS_CELL=sdnds-tw

# the address of the VM to install the package onto
export OC1="192.168.59.202"

# the default address used by ONOS utilities when none are supplied
export OCI="192.168.59.202"
 
# the ONOS apps to load at startup
export ONOS_APPS="drivers,openflow,fwd,proxyarp,mobility"
 
# the Mininet VM (if you have one)
export OCN="192.168.59.202"
 
# pattern to specify which address to use for inter-ONOS node communication (not used with single-instance core)
export ONOS_NIC="192.168.59.*"

# User
export ONOS_USER=root
export ONOS_GROUP=root
EOF

cell sdnds-tw
```

## Run ONOS
```bash
onos-setup-karaf clean 192.168.59.202
karaf clean
```
  * 與 `ok clean`一樣

## Open Web GUI
```
http://192.168.59.202:8181/onos/ui/index.html
```

## Reference
* [ONOS from Scratch](https://wiki.onosproject.org/display/ONOS/ONOS+from+Scratch)
* [Installing and Running ONOS](https://wiki.onosproject.org/display/ONOS/Installing+and+Running+ONOS)
* [Getting ONOS](https://wiki.onosproject.org/display/ONOS/Getting+ONOS)
* [Test Environment Setup](https://wiki.onosproject.org/display/ONOS/Test+Environment+Setup)
