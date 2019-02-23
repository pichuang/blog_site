---
layout: post
title: 'ArchLinux Installation'
date: 2014-04-11 02:25
comments: true
categories: 
---
#前言
本人使用 Ubuntu 好幾年了(大二到碩一), 會使用 Ubuntu 不外乎是因為拿個 LiveUSB 放進去到ㄧ台電腦, 約略 15 分就裝完了, 很快很肥很棒棒. 後來到了碩一進了系計中開始接觸了 ArchLinux, 發現裝起來其實還蠻輕鬆的

ArchLinux 最大特色:

1. 簡單 ([ArchWiki](https://wiki.archlinux.org/) 沒看過哪家 Offical Distro Wiki 這麼豐富的)  
2. 快速 (對於 x86 有特別優化)  
3. 高客製化 (對 Linux 有中度以上理解)  
4. 無縫升級 (每天都可以 ```pacman -Syu``` 升級系統)  

# 安裝過程
## 下載
- [archlinux-2014.04.01-dual.iso](http://archlinux.cs.nctu.edu.tw/iso/2014.04.01/archlinux-2014.04.01-dual.iso)

## 使用 LiveUSB 開機
- 忽略不寫

## 設定網路
- 關閉 dhcp  ```dhcpd -k``` 
- 查詢 NIC name ```ip link```
- 設定 ip
> ip link set \<DEVICENAME\> up  
ip addr add x.x.x.x/24 dev \<DEVICENAME\>  
ip route add default via \<ROUTE_IP\>  

- dns 設定 ```vim /etc/resolv.conf```
> nameserver 168.95.1.1  
nameserver 8.8.8.8  

- 設定 ssh
> passwd
systemctl start sshd
  - 設定到這裡就可以出機房用 ssh 進去設定
  - 如果是在 NAT 底下, 可以用 SSH Reverse tunnel 做控制
  - ```ssh -NfR 55688:localhost:22 roan@public.ip``` 
  - ```ssh localhost -p55688```

## 切 Partitions

<img class="center" src="http://user-image.logdown.io/user/5820/blog/5842/post/193251/GmHV0XLHTzqKwZvTCZTP_partitions.png" alt="partitions.png">

## Copy Partitions
- 複製 GPT Table 至 /dev/sdb ```sgdisk /dev/sda  -R=/dev/sdb```
- 產生新的GUID ```sgdisk -G /dev/sdb```

## RAID
- 利用 mdadm 做 RAID
- / 和 /boot 做 RAID1 
> mdadm --create --verbose /dev/md1 --level=mirror --raid-devices=2 /dev/sd{a,b}1

- /tmp 做 RAID0  
> mdadm --create --verbose /dev/md3 --level=mirror --raid-devices=2 /dev/sd{a,b}2

- 觀察 RAID 建立進度 
> cat /proc/mdstat

- 儲存 RAID 設定 
> mdadm --detail --scan | tee -a /etc/mdadm.conf

## Make Filesystem
- Swap
> mkswap /dev/sda3  
mkswap /dev/sdb3  
swapon /dev/sd{a,b}3  

- {/boot,/tmp,/}
> \#/boot  
mkfs.ext2 /dev/md4  
\#/  
mkfs.ext4 /dev/md1  
\#/tmp  
mkfs.ext4 /dev/md2  

## Pre-install Filesystem
- Mount Filesystem
> mount /dev/md1 /mnt && \  
mkdir /mnt/boot && \  
mount /dev/md4 /mnt/boot && \  
mkdir /mnt/tmp && \  
mount /dev/md2 /mnt/tmp  

- 設定 mirror ```vim /etc/pacman.d/mirrorlist```
> Server = http://linux.cs.nctu.edu.tw/archlinux/$repo/os/$arch

- 安裝 base 和 base-devel ``pacstrap /mnt base base-devel```
- 產生新的 fstab ```genfstab -p /mnt | tee /mnt/etc/fstab```
- 複製 mdadm.conf ```cp /etc/mdadm.conf /mnt/etc/```

## chroot
- Enviroment setting
> arch-chroot /mnt  
. /etc/profile  
PS1="(chroot) $PS1"  

- 設定 hostname 
> echo yaya > /etc/hostname  
hostnamectl set-hostname yaya  

- 設定時間
> timedatectl set-timezone Asia/Taipei  
hwclock --systohc --utc --adjfile /etc/adjtime  

- 設定 keymap 與 console 字體
> echo "KEYMAP=us" > /etc/vconsole.conf

- 設定 locale ```vim /etc/locale.gen```
> en_US.UTF-8 UTF-8  
en_US ISO-8859-1  
zh_TW.UTF-8 UTF-8  
zh_TW BIG5  

- 設定 default locale 
> echo "LANG=en_US.UTF-8" > /etc/locale.conf

- 網路設定
> systemctl disable network  
cp /etc/netctl/examples/ethernet-static /etc/netctl/ethernet  
\#設定網路  
vim /etc/netctl/ethernet  
netctl enable ethernet  

- 設定 initramfs ```vim /etc/mkinitcpio.conf```
> HOOKS="base udev autodetect pata scsi sata filesystems usbinput fsck mdadm_udev"

- 製作 initramfs ```mkinitcpio -p linux```

- 安裝 OpenSSH
> pacman -S openssh  
systemctl enable sshd.service  

- 設定 bootloader
> pacman -S grub  
grub-install --target=i386-pc --recheck /dev/sda  
grub-install --target=i386-pc --recheck /dev/sdb  
cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo  
grub-mkconfig -o /boot/grub/grub.cfg  

- 收尾
> passwd  
exit  
reboot  

