---
layout: post
title: "ArchLinux rescue mode"
description: "Into rescue mode to change root passwd on ArchLinux"
date: 2015-07-20 00:00:00 +0800
updated: 2015-07-20 00:00:00 +0800
category:
- Infra
- Linux
tags: 
- linux
- infra
- linux
---


## Step
1. 進入 grub 後, 按 `e` 進入編輯模式
2. 於 linux 該行最後輸入
> init=/bin/bash systemd.unit=rescue.target
<img src="https://lh3.googleusercontent.com/FxOE-jrqFWDI2zdsyW7Pag0zy40kZ7TykCR5BPv7jRg=w651-h413-no" width="400" height="200"><

3. 輸入 ctrl + x 或 F10 啟動
4. remount root filesystem
> mount -n -o remount,rw /

5. Change root password
> passwd

6. Reboot

## Reference
- [Reset root password](https://wiki.archlinux.org/index.php/Reset_root_password)
