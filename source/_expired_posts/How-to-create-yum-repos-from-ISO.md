---
layout: post
title: How to create yum repos from ISO
date: 2017-07-29 21:15:18
updated: 2017-07-29 21:15:18
category: Infra
tags:
- rhel
- centos
- yum
- mirror
---

## Environment
- rhel-mirror.on.ec

## Version information
- RHEL 7.3

<!--more-->

## Copy pakages to local
### In rhel-mirror
```
# copy rhel-server-7.3-x86_64-dvd.iso into rhel-mirro
mkdir -p /media/cdrom/
mv rhel-server-7.3-x86_64-dvd.iso /media/cdrom

# Mount rhel-server-7.3-x86_64-dvd.iso
mkdir -p /media/rh73
mount -o loop /media/cdrom/rhel-server-7.3-x86_64-dvd.iso /media/rhel73

# Copy Packages to local
mkdir -p /repos/centos/7/3
cp -arv /media/rhel73/Packages/* /repos/centos/7/3

# Create Repo metadata in local
yum install httpd createrepo
cd /repos/centos/7/3 && createrepo .

# Setup httpd
ln -s /repos/centos /var/www/html/centos
firewall-cmd --zone=public --add-service=http

# For SELinux Enable
chcon -R -t httpd_sys_content_t /reops/centos
chcon -R -t httpd_sys_content_t /var/www/html/centos

# Clean
umount /media/rhel73
rm -rf /media/rhel73
```

## Configure Repository on Client
```
cat << 'EOF' > /etc/yum.repos.d/RHEL73.repo
[RHEL73]
name=Red Hat Enterprise Linux 7.3
baseurl=http://bsshost1.on.ec/CentOS/7/3
gpgcheck=0
EOF
yum repolist
yum update
```
- [How to create a local repository for updates](https://access.redhat.com/solutions/9892)
- [Upgrading the System Off-line with ISO and Yum](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/5/html/Deployment_Guide/s1-yum-upgrade-system.html)