---
layout: post
comments: true
title: "Build ONOS Cluster"
description: "Detail of Building ONOS Cluster"
date: 2016-05-14 00:00:00 +0800
updated: 2017-11-05 00:00:00 +0800
category: sdn
tags:
- sdn
- onos
---

#### History
2017/11/05: Update to 1.11 branch
2017/03/21: Update to 1.9.0 version
2017/01/09: Update to 1.8.0 version

## 前言
雖然之前有寫過 [ONOS on CentOS7](https://blog.pichuang.com.tw/onos-on-centos7)，但因為開發上還是在 MAC OSX 上較為易用，故另開一篇紀錄個人的使用過程

## 目標
從 MAC OSX 上針對 ONOS 1.11.0 Loon 版本做編譯後，部署至 ONOS1, ONOS2 Machine 上組成一個 ONOS Cluster

## Enviroment
- Build-ONOS
  - MAC OSX 10.12.1
  - Java 1.8.0
  - Apache Karaf 3.0.5
  - Apache Maven 3.3.9
  - Misc
    - IP: 192.168.100.42
    - Cell name: pichuang
- ONOS-1 Machine
  - Ubuntu 14.04
  - Java 1.8.0
  - User/Group: sdn/sdn
  - IP: 192.168.100.45
- ONOS-2 Machine
  - Ubuntu 14.04
  - Java 1.8.0
  - User/Group: sdn/sdn
  - IP: 192.168.100.46
- ONOS-3
  - Ubuntu 14.04
  - Java 1.8.0
  - User/Group: sdn/sdn
  - IP: 192.168.100.47

### Install Oracle Java 8 on Build Machine, ONOS-1, ONOS-2 and ONOS-3
```bash
sudo apt-get install software-properties-common -y
sudo add-apt-repository ppa:webupd8team/java -y
sudo apt-get update
echo "oracle-java8-installer shared/accepted-oracle-license-v1-1 select true" | sudo debconf-set-selections
sudo apt-get install oracle-java8-installer oracle-java8-set-default unzip zip curl -y
```
* For Build Machine, ONOS-1, ONOS-2 and ONOS-3
* If you are using Ubuntu 16.04, please make sure all of machine have python2. The install command is "sudo apt intall -y python"

### Add user and switch to `sdn` user
```
sudo adduser sdn --system --group 
```
- Make sure you append NOPASSWD in sudo files, e.g. `%sudo	ALL=(ALL) NOPASSWD:ALL` in

### Prepare Installation Enviroment of Build-ONOS
```bash
cd ~/ && mkdir Downloads Applications
cd Downloads
wget http://archive.apache.org/dist/karaf/3.0.5/apache-karaf-3.0.5.tar.gz
wget https://archive.apache.org/dist/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
tar -zxvf apache-karaf-3.0.5.tar.gz -C ../Applications/
tar -zxvf apache-maven-3.3.9-bin.tar.gz -C ../Applications/
```

### Setting .bashrc
```bash
cat >> ~/.bashrc << EOF
export ONOS_ROOT=~/onos
source ~/onos/tools/dev/bash_profile
export ONOS_CELL=pichuang # Change your cell name
cell $ONOS_CELL
EOF
```
- In build-onos machine

### Compile ONOS Source Code
```bash
cd ~/ && git clone https://github.com/opennetworkinglab/onos
cd ~/onos && git checkout onos-1.11
source ~/.bashrc
tools/build/onos-buck build onos --show-output
```
- DO NOT install it when you are **root**
- You will get the ONOS package in `buck-out/gen/tools/package/onos-package/onos.tar.gz`
- Run the ONOS service with debug mode. `tools/build/onos-buck run onos-local -- clean debug`
- Attach to ONOS CLI `tools/test/bin/onos localhost`

### Create a cell definition
```bash
cat > ~/onos/tools/test/cells/pichuang << EOF
# Cell name
export ONOS_CELL=pichuang

# IP addresses of the VMs hosting ONOS instances. More OC instances may be set, if necessary.
export OC1="192.168.100.45"
export OC2="192.168.100.46"
export OC3="192.168.100.47"

# The default target node IP. This is an alias for OC1.
export OCI="192.168.100.45"

# The ONOS apps to load at startup
# For trellis
export ONOS_APPS="drivers,openflow,segmentrouting,fpm,dhcprelay,netcfghostprovider,routeradvertisement"

# Pattern to specify which address to use for inter-ONOS node communication (not used with single-instance core)
export ONOS_NIC="192.168.100.*"

# User
export ONOS_USER=sdn # Change your username
export ONOS_GROUP=sdn # Change your group name

# WEB User
export ONOS_WEB_USER=onos
export ONOS_WEB_PASS=rocks
EOF

cell pichuang
```
* Change your configuraiton depend on the requirement by yourself
* `ONOS_USER/ONOS_GROUP` 是指 ONOS-{1,2,3} 運行 ONOS 的 Linux User/Group, 並非是 Build Machine 內的
* `ONOS_USER/ONOS_GROUP` is means user and group in Linux who will run ONOS applications, not for Build Machine.
* `ONOS_WEB_USER/ONOS_WEB_PASS` 是指 ONOS-{1,2,3} 運行 ONOS 內於 `~/Applications/apache-karaf-3.0.5/etc/users.properties` 所設定好的帳號密碼, 只要有關 RESTful API 的存取, 皆是使用這組帳密
* 每次要 Deploy 的時候, 建議都在跑一次 `cell $ONOS_CELL` 確定變數有載入到環境

### SSH Login Without Password
```bash
# Generate SSH Key on build machine
ssh-keygen

# Login via SSH from build machine to ONOS-1 machine
ssh-copy-id -i ~/.ssh/id_rsa.pub sdn@192.168.100.45

# Login via SSH from build machine to ONOS-2 machine
ssh-copy-id -i ~/.ssh/id_rsa.pub sdn@192.168.100.46

# Login via SSH from build machine to ONOS-3 machine
ssh-copy-id -i ~/.ssh/id_rsa.pub sdn@192.168.100.47
```

### Check package and set-up is ready on node
```bash
stc prerequisites
```

### Deploy ONOS Package
```bash
stc setup
```
* 如果對 `stc setup` 感到興趣的話, 可以參考 `$ONOS_ROOT/tools/test/scenarios/setup.xml` 的內容
  * 若對 setup.xml 運行的指令有興趣, 可以參考 `$ONOS_ROOT/tools/test/bin/` 的 Shell Script
* `stc setup` `stc startup` `stc shutdown` 這三個指令建議可以都用看看

### Uninstall ONOS Package
```bash
stc teardown
```

{% img https://lh3.googleusercontent.com/UwYDAR59NOASmqhGEEeAKzhlWpl8VkSIHie3WuqUTvNUzbU4EDt9M2bVYh5qePOZfZDxdmVFL9729mNuekyv1kKmFL1h9YESEOCzdn1B9EjnKOLRcgmH404Ce-9MKI3rJNowEmtji6v5dRdrbRVY9Kl5-rDWmuw8Fis6PFhyFwMcfkCjrK6sLWQX3FC2Hc1B1TstqVh2An411GrjmdbVY37YAv5wuAiOYlN5wSeSDPnrnehognv8ta-ohh5pTtVNS7ls5DLyfwo9i4RSMXY3ozFTMHVwEdf475RrAkWf4s0fAYXLfzbP6u1P7frzsToY6ngn8sowxfJdsh6OdXzREve2eDXaqfdetkPBTKZEmal60ADozGCozrSQNsPMXE6mM2Z7mvzPUGlKQGNkdX_53shFOdIKIf4U_5XLS4lnV7FCEQllFrZaShEVbQ9ie6VEk7QZIFi15HGsnArnJUhk-eUKGky_cxT8wtF78lUCUd49OizygRFg7xFHaYT_f7J4mRE02IqdS4ifTm_akf7rSChGDidIaKJtvu17VQp3CaKXlMeFL_CYXfyRNYIib8PjgIMqRzCk_pP8vz-p0JHzOxD1GOFCAsg=w1342-h1025-no %}

### Login ONOS Cluster CLI
```bash
onoss # or use command `onos $OCI`
```

### Open WEB GUI
```bash
http://192.168.100.45:8181/onos/ui/index.html
```
* ONOS WEB User: onos
* ONOS WEB Password: rocks

### ONOS API
```bash
http://192.168.100.45:8181/onos/v1/docs/
```

## Reference
- [Test Environment Setup](https://wiki.onosproject.org/display/ONOS/Environment+setup+with+cells)
- [3 Steps to Perform SSH Login Without Password Using ssh-keygen & ssh-copy-id](http://www.thegeekstuff.com/2008/11/3-steps-to-perform-ssh-login-without-password-using-ssh-keygen-ssh-copy-id/)
- [Appendix A : List of ONOS Utility Scripts (onos-* scripts)](https://wiki.onosproject.org/pages/viewpage.action?pageId=1048691)
- [Building ONOS](https://wiki.onosproject.org/display/ONOS/Building+ONOS)
