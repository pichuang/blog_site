---
layout: post
title: 'OpenvSwitch Lab 2$ OpenvSwitch and KVM'
date: 2014-04-25 03:13
comments: true
categories: 
---
#前言
以下的 LAB 是要使用 OpenvSwtich 來取代原先 KVM 所採用的 ```brctl``` 功能, 透過 OpenvSwitch 可能多增加一些網路環境的控制及監控, 但是相對會損失些網路傳輸效能, 本次 Lab 目標是建立兩台 VM (Ubuntu 及 Archlinux), 中間透過 OpenvSwitch 控制網路,

#安裝過程
##環境
- Host OS: Ubuntu 12.04.4 Server LTS X86_64 
- Guest OS: FreeBSD 10.0 x86_64
- Network: OpenvSwitch 2.1.0

## 安裝 Host OS
- 很懶, 忽略不寫

## 測試 Host 是否有支援 Intel VT 或 AMD-V
- ```egrep '(vmx|svm)' --color=auto /proc/cpuinfo```
- 有 vmx 或 svm 之一即可, 這表示 CPU 有支援虛擬化功能
- 若無
	- 如果是實機的話, 要去 BIOS 把 VT-x 或者是 AMD-v 之類的選項開起來
	- 如果是使用 VMware Workstation 的話, 請去 ```Setting > Hardware > Processors > Virtualization engine``` 將 ```Virtualize Intel VT-x/EPT or AMD-V/RVI``` ```Virtualize CPU performance counters``` 開啟
<img class="center" src="http://user-image.logdown.io/user/5820/blog/5842/post/192977/Io35eighTUi9F5JfY8E7_vmware-workstation-setting.PNG" alt="vmware-workstation-setting.PNG">

## 關於 OpenvSwitch 
1. [編譯 OpenvSwitch v2.1.0 on Ubuntu 12.04 LTS](http://roan.logdown.com/posts/165399-compile-openvswitch-on-ubuntu-1204-lts "來來來來來來")
2. 新增 OpenvSwich bridge 
  - ```ovs-vsctl add-br ovs-br```
3. 設定 Controller 
  - ```ovs-vsctl set-controller ovs-br tcp:1.2.3.4:6633```
4. 設定 Fallback 
  - ```ovs-vsctl set-fail-mode ovs-br standalone```
5. 開啟 STP 
  - ```ovs-vsctl set bridge ovs-br stp_enable=true````
  - 這個最好要設定一下, 實作過程中有發生 loop 
6. 更多設定請看 [設定 OpenvSwitch](http://roan.logdown.com/posts/191801-set-openvswitch)


## 關於 KVM
### 安裝套件
- ```aptitude install qemu-kvm libvirt-bin uml-utilities virtinst```

### 新增 KVM 管理帳號
- ```adduser fumou && adduser fumou libvirtd``` 這邊是要透過新增一個 non-root 的使用者 fumou 加入 libvirtd 來控制 KVM 的動作, libvirtd 不用自己創, 安裝時 libvirt-bin 會幫忙設定好, 以下的動作都會透過 fumou 做設定
- ```adduser `id -un` libvirtd``` 如果想要用原本的帳號作控制, 可選擇此項
- 可透過 ```grep libvirtd /etc/group``` 確認是否在 group 裡面
- Re-login 以後使用該設定帳號輸入 ```virsh -c qemu:///system list``` 來確認是否有管理權限

### 關閉 KVM 對 default bridge 的設定
> virsh net-destroy default  
virsh net-autostart --disable default  
aptitude purge ebtables  

### 設定 KVM Network
1. vim /etc/network/if-up.d/ovs-ifup && chmod +x /etc/network/if-up.d/ovs-ifup
> \#!/bin/sh  
switch='ovs-br'  
/sbin/ifconfig $1 0.0.0.0 up  
ovs-vsctl add-port ${switch} $1  

2. vim /etc/network/if-down.d/ovs-ifdown && chmod +x /etc/network/if-down.d/ovs-ifdown
> \#!/bin/sh  
switch='ovs-br'  
/sbin/ifconfig $1 0.0.0.0 down  
ovs-vsctl del-port ${switch} $1  

3. vim /etc/network/interfaces
> \# The loopback network interface  
auto lo  
iface lo inet loopback  
\# The primary network interface  
auto eth0  
iface eth0 inet manual  
up ifconfig $IFACE 0.0.0.0 up  
down ifconfig $IFACE down  
\# OpenvSwitch Interface  
auto ovs-br  
iface ovs-br inet static  
address 192.168.77.100  
netmask 255.255.255.0  
gateway 192.168.77.1  
dns-nameservers 168.95.1.1  

4. vim /etc/init/failsafe.conf
> $PLYMOUTH message --text="Waiting for network configuration..." || :  
sleep 1  
$PLYMOUTH message --text="Waiting up to 60 more seconds for network configuration..." || :  
sleep 1  
$PLYMOUTH message --text="Booting system without full network configuration..." || :  

### 設定 OS FreeBSD 
1. 下載 ISO 檔
> mkdir ~/iso && cd iso  
curl -O http://freebsd.cs.nctu.edu.tw/pub/FreeBSD/releases/amd64/amd64/ISO-IMAGES/10.0/FreeBSD-10.0-RELEASE-amd64-bootonly.iso  

2. Format image
> qemu-img create -f qcow2 /home/roan/image/freebsd.img 20G

3. 利用 iso 安裝
> sudo kvm -smp 1 -drive file=/home/roan/image/freebsd.img,if=virtio \  
-cdrom /home/roan/iso/FreeBSD-10.0-RELEASE-amd64-bootonly.iso \  
-m 1024 \  
--nographic -curses \  
-net nic,model=e1000 -net tap,ifname=freebsd,script=/etc/network/if-up.d/ovs-ifup,downscript=/etc/network/if-down.d/ovs-ifdown \  
-boot d  
  - 若是使用想利用CLI deploy 之類的需要下 ```--nographic -curses```
  - ifname=freebsd, 為 GuestOS interface name, 可從 ```ovs-vsctl show``` 觀察

### Trouble Shooting
1. ```Device ‘tap’ could not be initialized``` => ```ovs-vsctl del-port tap0```
2. ``` 1024 x 768 Graphic mode ``` => 這是因為 Guest 切換成 Graphic mode 導致, [詳解](http://lists.gnu.org/archive/html/qemu-devel/2005-12/msg00084.html)

## KVM 相關指令
- ```virsh nodeinfo``` 檢查硬體規格

# Reference
- [How to install and configure KVM and Open vSwitch on Ubuntu or Debian](http://xmodulo.com/2014/01/install-configure-kvm-open-vswitch-ubuntu-debian.html)
- [Linux-KVM: Ubuntu 12.04 with Openvswitch](http://blog.allanglesit.com/2012/03/linux-kvm-ubuntu-12-04-with-openvswitch/)
- [Installing KVM and Open vSwitch on Ubuntu](http://blog.scottlowe.org/2012/08/17/installing-kvm-and-open-vswitch-on-ubuntu/)
- [How to Use Open vSwitch with KVM](http://git.openvswitch.org/cgi-bin/gitweb.cgi?p=openvswitch;a=blob_plain;f=INSTALL.KVM;hb=HEAD)
- [OpenNebula 4.0 Home Lab – Part 3 – Installing KVM and Open vSwitch on Ubuntu 13.04](http://kloudology.com/2013/06/16/opennebula-4-0-home-lab-part-3-installing-kvm-and-open-vswitch-on-ubuntu-13-04/)
- [Deploy KVM + Open vSwitch + iSCSI + Ubuntu 12.04 x64](http://dev.digibess.it/doku.php?id=hypervisor:deploy)
- [如何在Linux发行版上安装和配置KVM和Open vSwitch？](http://os.51cto.com/art/201401/428401.htm)



