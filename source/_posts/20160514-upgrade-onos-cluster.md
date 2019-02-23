---
layout: post
title: "Upgrade ONOS Cluster"
description: "upgrade~"
date: 2016-05-14 00:00:00 +0800
updated: 2017-04-19 00:00:00 +0800
category: sdn
tags:
- sdn
- onos
---

## 前言
接續上篇 [Build ONOS Cluster](http://blog.pichuang.com.tw/build-onos-cluster)，部署運行後，總會需要打個 patch 升級一下，故本篇記錄該流程

<!--more-->

## 目標
接續上篇文章的環境，升級已經建好的 ONOS Cluster

## Environment
- Build Machine
  - MAC OSX 10.11.3
  - Java 1.8.0
  - Apache Karaf 3.0.5
  - Apache Maven 3.3.9
  - Misc
    - IP: 10.211.55.2
    - Cell name: pichuang
- ONOS1 Machine
  - Ubuntu 14.04
  - Java 1.8.0
  - User/Group: sdn/sdn
  - IP: 10.211.55.10
- ONOS2 Machine
  - Ubuntu 14.04
  - Java 1.8.0
  - User/Group: sdn/sdn
  - IP: 10.211.55.11

### Compile ONOS Source Code on Build Machine
```
cd ~/onos
git pull # Pull latest source code
git checkout onos-1.9 # Following 1.9 (Falcon) branch
onos-buck build onos --show-output
```
- For now, you can see the tarball in /tmp/onos-*.tar.gz

### Deploy ONOS Package
```
stc setup
```

### Check ONOS Latest Version
```
onos $OCI
onos> onos:summary
```
