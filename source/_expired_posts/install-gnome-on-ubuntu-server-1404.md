---
layout: post
title: "Install GNOME on Ubuntu server 14.04"
description: ""
date: 2015-07-25 00:00:00 +0800
updated: 2015-07-25 00:00:00 +0800
category:
- Infra
- Linux
tags: 
- infra
- ubuntu
---

### Install GNOME3
> aptitude update  
aptitude --install-recommends install gnome-desktop  
reboot

### (Optional) Configure lightdm to allow manual logins
> echo "greeter-show-manual-login=true" >> /usr/share/lightdm/lightdm.conf.d/50-ubuntu.conf  
reboot  

### Reference
- [How to configure lightdm to allow manual logins in Ubuntu 14.04](http://askubuntu.com/questions/451950/how-to-configure-lightdm-to-allow-manual-logins-in-ubuntu-14-04)
