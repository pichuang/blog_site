---
layout: post
title: "Build CORD Fabric"
date: 2015-07-28 00:00:00 +0800
updated: 2015-07-28 00:00:00 +0800
description: ""
category: sdn
tags:
- onos
- cord
- sdn
---

## 前言
實際運行一下 CORD Fabric 的運行情況，以便後續研究

## About ONOS

### Install ONOS
- 點擊右邊連結 [Build ONOS Cluster](http://blog.pichuang.com.tw/build-onos-cluster)
- 這邊採用 ONOS Master Branch

### CORD-Fabric Cell file 
```bash
cat >> ~/onos/tools/test/cells/cord-fabric << EOF
# Cell name
export ONOS_CELL=cord-fabric

# IP addresses of the VMs hosting ONOS instances. More OC instances may be set, if necessary.
export OC1="10.211.55.10"
export OC2="10.211.55.11"

# The default target node IP. This is an alias for OC1.
export OCI="10.211.55.10"

# The ONOS apps to load at startup
export ONOS_APPS="drivers,openflow,segmentrouting"

# Pattern to specify which address to use for inter-ONOS node communication (not used with single-instance core)
export ONOS_NIC="10.211.55.*"

# User
export ONOS_USER=sdn # Change your username
export ONOS_GROUP=sdn # Change your group name

# WEB User
export ONOS_WEB_USER=onos
export ONOS_WEB_PASS=rocks
EOF

cell cord-fabric
```
* ONOS_APPS 不能開 ```proxyarp```, 因為跟 ```segmentrouting``` 會衝 XD

### CORD-Fabric Network Config
```bash
wget https://raw.githubusercontent.com/opennetworkinglab/onos/master/tools/package/config/samples/network-cfg-fabric-2x2-min.json
onos-netcfg $OCI network-cfg-fabric-2x2-min.json
```

## About CPqD Software Switch

### Install Mininet
```bash
git clone http://github.com/mininet/mininet
./mininet/util/install.sh -n3f
```

### Fabric Python Script
```bash
wget https://raw.githubusercontent.com/rascov/docker-fabric/master/fabric.py
python fabric.py # Should change controller IP in Line 55 by yourself
```

## Reference
- [Software Switch Installation Guide](https://wiki.onosproject.org/display/ONOS/Software+Switch+Installation+Guide)
- [rascov/docker-fabric](https://github.com/rascov/docker-fabric)
