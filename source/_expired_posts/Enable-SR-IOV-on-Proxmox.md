---
layout: post
title: Enable SR-IOV on Proxmox
date: 2017-04-22 00:47:02
updated: 2017-04-22 00:47:02
category:
- Infra
tags:
- infra
- linux
- sriov
- intel
- proxmox
---

pichuang (Author): The post should support other debian distribution (Ubuntu, Debain...etc)

## What is SR-IOV?
In network virtualization, a single root input/output virtualization or SR-IOV is a network interface that allows the isolation of the PCI Express resources for manageability and performance reasons

## Environment
- Proxmox
  - pve-manager/4.4-13/7ea56165 (running kernel: 4.4.49-1-pve)
  - Based on Debian 8 (jessie)
- NIC
  - Intel Corporation Ethernet Controller X710
  - Logical NIC
    - eth{2..5}

<!--more-->

### Enable IOMMU

- Enabled with kernel parameter `intel_iommu`

- Please Append `intel_iommu=on` to the `GRUB_CMDLINE_LINUX` entry in `/etc/default/grub` if not exist
```bash
# Make sure enable SR-IOV function and Intel VT-d in BIOS

# Append configuration
root@pve:~# cat /etc/default/grub |grep GRUB_CMDLINE_LINUX_DEFAULT
GRUB_CMDLINE_LINUX_DEFAULT="quiet processor.max_cstates=1 idle=poll pcie_aspm=off intel_iommu=on"

# Update GRUB configuration 
root@pve:~# grub2-mkconfig –o /boot/grub2/grub.cfg

# Rebbot for the iommu change to take effect
root@pve:~# reboot

# Check kernel command line in Linux and look for `intel_iommu=on`
root@pve:~# cat /proc/cmdline
BOOT_IMAGE=/boot/vmlinuz-4.4.49-1-pve root=/dev/mapper/pve-root ro quiet intel_iommu=on processor.max_cstates=1 idle=poll pcie_aspm=off

# Check dmesg log
root@pve:~# dmesg | grep IOMMU | grep enabled
[    0.000000] DMAR: IOMMU enabled
```

### Create Virtual Functions (VFs)

- Linux does not create VFs by default. The X710 support upto 32VFs per port. The XL710 support upto 64VFs per port.
```bash
# For example(eth2), X710 support 32VFs per physical port
root@pve:~# cat /sys/class/net/eth2/device/sriov_totalvfs
32
```

- Create 4 VFs on eth2 (You can choose any interface name which is in X710 adapter)
```bash
root@pve:~# echo "4" > /sys/class/net/eth2/device/sriov_numvfs
root@pve:~# cat /sys/class/net/eth2/device/sriov_numvfs
4

root@pve:~# lspci | grep -i 'Virtual Function'
01:02.0 Ethernet controller: Intel Corporation XL710/X710 Virtual Function (rev 02)
01:02.1 Ethernet controller: Intel Corporation XL710/X710 Virtual Function (rev 02)
01:02.2 Ethernet controller: Intel Corporation XL710/X710 Virtual Function (rev 02)
01:02.3 Ethernet controller: Intel Corporation XL710/X710 Virtual Function (rev 02)
```

- Bring up link on VFs
```bash
root@pve:~# ip link set dev eth2 up
root@pve:~# ip link set dev eth2 vf 0 mac aa:bb:cc:dd:ee:00
root@pve:~# ip link set dev eth2 vf 1 mac aa:bb:cc:dd:ee:01
root@pve:~# ip link set dev eth2 vf 2 mac aa:bb:cc:dd:ee:02
root@pve:~# ip link set dev eth2 vf 3 mac aa:bb:cc:dd:ee:03
```
***Tip***: If a VFs MAC address is not assigned in the host, then the VF driver will use a random MAC address. This random MAC address may change each time the VF driver is reloaded. You can assign a static MAC address in the host machine. `This static MAC address will survive a VF driver reload`.

- Reload vf module
```bash
root@pve:~# rmmod i40evf
root@pve:~# modprobe i40evf

# 10G Intel NIC module name: ixgbevf
```


- Check eth2 Status
```bash
root@pve:~# ip link show eth2
3: eth2: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc mq portid 8cea1b30da53 state DOWN mode DEFAULT group default qlen 1000
    link/ether 8c:ea:1b:30:da:53 brd ff:ff:ff:ff:ff:ff
    vf 0 MAC aa:bb:cc:dd:ee:00, spoof checking on, link-state auto
    vf 1 MAC aa:bb:cc:dd:ee:01, spoof checking on, link-state auto
    vf 2 MAC aa:bb:cc:dd:ee:02, spoof checking on, link-state auto
    vf 3 MAC aa:bb:cc:dd:ee:03, spoof checking on, link-state auto
```

- Check interface of VFs
```bash
# iproute2
root@pve:~# ip link show eth6
root@pve:~# ip link show eth7
root@pve:~# ip link show eth8
root@pve:~# ip link show eth9

# ethtool
root@pve:~# ethtool -i eth6
driver: i40evf
version: 1.4.15-k
firmware-version: N/A
bus-info: 0000:01:02.0
supports-statistics: yes
supports-test: no
supports-eeprom-access: no
supports-register-dump: no
supports-priv-flags: yes
```

## Screenshot from Proxmox
{%img https://lh3.googleusercontent.com/9N5BvoNC175V1VBkfJn0H1MpsupjiKEMycGM7eFZVj_HQF1Bj78ZDYtMk2azExkNOGGuXFuwWjBf-OunkvYHlUB0FoowhY3daaCCazJLpXQ0UV4NMWvhioKxUsvF4ChyDghZTWdrbe4wAmJ8eed1qfyaskjT_43u2N8lBl0H7I8mVa6bxZnRyg14ve4pMKQsZtayBghSfZyRMpnaNyK89V7Mudgu3X4vKcJ5TjRboIxXyrMNsXr7puPXbzRty0pBAMvlSiepPzbnd76C5noJvkYS8DBTSrm_4L0iyKv88r1SMKTX1fi2X0JEYOHeYQx76OzYYmEGveWRvgQKf-MJ_wWnJqbtgtOE1oqBm9CJXdJ5AdJ9se-6jUb4QQ5CXN8DCkvg-YWWWICNW_8PezeET3rnHa_H_bo-2hHFJ5TfVGzsUPKxl8P_f2BOkHD1zK4ucBqBwts-2QirYIFFjcefmKl9vY-H6bL3X8GgrXEACMN4eBk49zNg26Z9SVa3ut3Lo5aNp-x3WjnF90fvGKp7av7Et2FVv84glqdzrZ01x8EZMh0MIAOZ4a-FelNokcUS9jlLhY7lopXsPQF78oHwEH8VeTemaULjooTUOwb7TiwhfqU_Tr8qIw=w1182-h580-no %}

## Reference
- [Single-Root Input/Output Virtualization (SR-IOV) with Linux* Containers][1]
- [XL710 SR-IOV Config Guide][2]
- [SR-IOV HandsOn_v1.0.pptx - Meetup][3]
- [Using SR-IOV on OpenStack][4]
- [Two aways to set mac address of SR-IOV VF][5]
- [Linux* Driver for Intel(R) XL710/X710 Virtual Function][6]
- [KVM 介绍（4）：I/O 设备直接分配和 SR-IOV][7]
[1]: https://software.intel.com/en-us/articles/single-root-inputoutput-virtualization-sr-iov-with-linux-containers
[2]: http://www.intel.com/content/dam/www/public/us/en/documents/technology-briefs/xl710-sr-iov-config-guide-gbe-linux-brief.pdf
[3]: http://files.meetup.com/19623913/SR-IOV%20HandsOn_v1.0.pptx
[4]: http://www.netdevconf.org/1.1/proceedings/slides/duyck-sr-iov-openstack.pdf
[5]: http://hustcat.github.io/two-way-to-set-vf-mac/
[6]: https://downloadmirror.intel.com/24693/eng/readme.txt
[7]: http://www.cnblogs.com/sammyliu/p/4548194.html