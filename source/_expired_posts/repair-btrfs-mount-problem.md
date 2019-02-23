---
layout: post
title: "Repair btrfs mount problem"
description: ""
date: 2015-07-20 00:00:00 +0800
updated: 2015-07-20 00:00:00 +0800
category:
- Infra
- Linux 
tags:
- linux
- infra
- btrfs
- linux
---

假設有問題的硬碟為 `/dev/sda3`

1. 清除 cache
`mount -t btrfs -o recovery,nospace\_cache,clear\_cache,autodefrag /dev/sda3 /mnt`

2. 進行簡單的 repair 程序
`btrfs check --repair /dev/sda3`

3. 重建 checksum tree
`btrfs check --init-csum-tree /dev/sda3`

4. 重建 extent tree (要等很久)
`btrfs check --init-extent-tree /dev/sda3`

{% img https://lh3.googleusercontent.com/wpCcVUJMcsQVOeDdr7RM_bM7WWFdJ1z6u6MdXrBQAwM=w810-h244-no 600 400 %}

我遇到的狀況是連 mount 都失敗的狀況, 故透過 livecd 進去系統後, 依序上述```2,3,4```的方式做重建的動作

## Reference
- [initrd-helpers/btrfs-mount-repair](https://github.com/sailfishos/initrd-helpers/blob/master/btrfs-mount-repair)
- [Btrfsck](https://btrfs.wiki.kernel.org/index.php/Btrfsck)
