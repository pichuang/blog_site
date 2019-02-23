---
layout: post
title: BRCM Shell Cheat Sheet
draft: true
date: 2017-03-07 18:06:27
updated: 2017-03-07 18:06:27
category: Infra
tags:
- infra
- brcm
- shell
---

## `show` command

- `show unit`
  - Show chipset information
- `show features`
  - Show chipset feature
- `show params`
  - Show chipset parameter

## `port` command

- `port ce0`
  - Show information at port ce0
- `port ce0 an=off`
  - Diable auto negotiation at port ce0
- `port ce0-ce5 sp=100`
  - Change speed to 100Mbps from port ce0 to port ce5
- `port ce0 MIDX=Xover`
- `help port`

## `combo` command
- `combo ce0-ce1`
- `combo ce0 fiber preferred=1`
- `combo ce0 copper enable=0`
  - Disable copper medium at port ce0
- `combo ce0 watch=on`
- `combo ce0 f autoneg_enable=0`

## `link` command
- `link`
- `link ?`
- `link force=ce0`
  - Forces link scan to update ports status instanly


## Reference
- [shell命令的使用][1]
- [bcm shell常用调试命令][2]

[1]: http://www.51wendang.com/doc/245fa102e4805cfef0de8a66/
[2]: http://mixin125.m.blog.chinaunix.net/uid-22238267-id-5748466.html
[3]: http://www.cnblogs.com/helloworldtoyou/p/5177932.html
