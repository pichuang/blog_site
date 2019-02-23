---
layout: post
title: Install CentOS 7 without VGA
date: 2017-07-17 10:09:00
updated: 2017-07-17 10:09:00
tags:
- headless
- serial
- headless
- centos
---

### Change timeout
- vim isolinux/isolinux.cfg
```
default vesamenu.c32
#timeout 600
timeout 50

display boot.msg
```
- Prompt until booting automatically, in units of 1/10 s. In the case, we change the timeout value from 60 secs to 5 secs

<!--more-->

### Change default booting prompt
```
...
label linux
  menu label ^Install Red Hat Enterprise Linux 7.3
  menu default
  kernel vmlinuz
  append initrd=initrd.img inst.stage2=hd:LABEL=RHEL-7_3\x20SE text console=tty0 console=ttyS0,115200n8
...
```
- Please notice `menu default`

### Remove `quiet`, append `text console=tty0 console=ttyS0,115200n8` in kernel parameter

- vim isolinux/isolinux.cfg
```bash
...

label linux
  menu label ^Install CentOS Linux 7
  menu default
  kernel vmlinuz
  append initrd=initrd.img inst.stage2=hd:LABEL=CENTOS\x207\x20X8 text console=tty0 console=ttyS0,115200n8

...
```
