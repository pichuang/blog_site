---
layout: post
title: 'Compile OpenvSwitch'
date: 2017-03-21 16:07:00 +0800
updated: 2017-03-21 16:07:00 +0800
categories: sdn
tags:
- sdn
- ovs
---

## Environment
  - OS: Ubuntu 14.04.5 Server LTS X86_64 
  - Kernel version: 4.4.0-59-generic

## Pre-install
```bash
apt install dh-autoreconf libssl-dev openssl
```

<!--more-->

## Compile OpenvSwitch
```bash
git clone https://github.com/openvswitch/ovs
git checkout branch-2.7
./boot.sh
./configure --with-linux=/lib/modules/`uname -r`/build
make
sudo make install
sudo make modules_install
sudo modprobe openvswitch
sudo modprobe libcrc32c
```
  * 使用 `lsmod |grep openvswitch` 確認ovs正確地掛載起來
  * ./configure 部份其實還可以下一個 `--prefix=` 參數, 可以讓openvswitch 完全裝在該目錄底下而不會亂掉

## Setting ovsdb
```bash
mkdir -p /usr/local/etc/openvswitch
ovsdb-tool create /usr/local/etc/openvswitch/conf.db /usr/local/share/openvswitch/vswitch.ovsschema
```
  * 建立一次即可, 如果設定有問題的話可以把conf.db砍掉重建

## (二選一) Start ovsdb-server (Non-SSL)
```bash
mkdir -p /usr/local/var/run/openvswitch
ovsdb-server --remote=punix:/usr/local/var/run/openvswitch/db.sock \
--remote=db:Open_vSwitch,Open_vSwitch,manager_options \
--pidfile --detach --log-file
```
  * `cat /usr/local/var/log/openvswitch/ovsdb-server.log` 可查到 ovsdb log 

## (二選一) Start ovsdb-server (SSL)
```bash
mkdir -p /usr/local/var/run/openvswitch
ovsdb-server --remote=punix:/usr/local/var/run/openvswitch/db.sock \
--remote=db:Open_vSwitch,Open_vSwitch,manager_options \
--private-key=db:Open_vSwitch,SSL,private_key \
--certificate=db:Open_vSwitch,SSL,certificate \
--bootstrap-ca-cert=db:Open_vSwitch,SSL,ca_cert \ 
--pidfile --detach --log-file
```
  * 若要使用 ssl 連線, 務必在編譯前要先準備好 `libssl-dev` 及 `openssl`, 否則會噴出 `Private key specified but Open vSwitch was built without SSL support` 的錯誤
  * [OpenvSwitch Lab 6$ TLS SSL](http://roan.logdown.com/posts/208707-openvswitch-lab-6-ssl)

## Launch `ovs-vsctl`
```bash
ovs-vsctl --no-wait init
```

## Start `ovs-switchd`
```bash
ovs-vswitchd --pidfile --detach --log-file
```
  * `/usr/local/var/log/openvswitch/ovs−vswitchd.log` 可查到 ovs-vswitchd log

### 觀察
* `ps aux |grep ovs`

<img class="center" src="http://user-image.logdown.io/user/5820/blog/5842/post/165399/C7SX9zsQQNOyIRBntqXb_ovs.png" alt="ovs_install_complete.png">


## 開機自動化
### 設定開機自動 load module
```bash 
echo "openvswitch " >> /etc/modules
echo "libcrc32c" >> /etc/modules
```

### 設定開機自動啟動
```bash
cat >> /etc/init.d/openvswitch << EOF
#!/bin/sh
start-stop-daemon -q -S -x /usr/local/sbin/ovsdb-server -- --remote=punix:/usr/local/var/run/openvswitch/db.sock --remote=db:Open_vSwitch,Open_vSwitch,manager_options --pidfile --detach --log-file
sleep 3 # waiting ovsdb-server
start-stop-daemon -q -S -x /usr/local/bin/ovs-vsctl -- --no-wait init
start-stop-daemon -q -S -x /usr/local/sbin/ovs-vswitchd -- --pidfile --detach --log-file
EOF
chmod +x /etc/init.d/openvswitch
update-rc.d -f openvswitch defaults
```

### 修改 Failsafe

vim /etc/init/failsafe.conf
```
...
$PLYMOUTH message --text="Waiting for network configuration..." || :
sleep 1
$PLYMOUTH message --text="Waiting up to 60 more seconds for network configuration..." || :
sleep 1
$PLYMOUTH message --text="Booting system without full network configuration..." || :
...
```

## References
- [OpenvSwitch Open vSwitch on Linux, FreeBSD and NetBSD][1]
- [OpenvSwitch OverView - hwchiu][2]
- [設定OpenvSwitch][3]

## Update
* Update: 2017/04/21 Rename `Compile OpenvSwitch`
* Update: 2014/08/15 [編譯 OpenvSwitch v2.3.0 on Ubutnu 14.04.1 LTS](http://roan.logdown.com/posts/220671-compile-openvswitch-v230-on-ubutnu-14041-lts)
* Update: 2014/08/15 Upgrade to OpenvSwitch 2.1.3
* Update: 2014/05/03 [Upgrade to OpenvSwitch 2.1.2](http://openvswitch.org/releases/NEWS-2.1.2)
* Update: 2014/04/30 [Upgrade to OpenvSwitch 2.1.1](http://openvswitch.org/releases/NEWS-2.1.1)
* Update: 2014/04/26 Add boot network interface setting
* Update: 2014/03/31 [Upgrade to OpenvSwitch 2.1.0](http://openvswitch.org/releases/NEWS-2.1.0)


[1]: https://github.com/openvswitch/ovs/blob/master/Documentation/intro/install/general.rst
[2]: http://hwchiu.logdown.com/posts/167510-openvswitch-overview
[3]: http://roan.logdown.com/posts/191801-set-openvswitch
