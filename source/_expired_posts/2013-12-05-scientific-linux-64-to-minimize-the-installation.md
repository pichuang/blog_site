---
layout: post
title: 'Scientific Linux 6.4 最小化安裝'
date: 2013-12-05 10:11
comments: true
categories: [Linux]
---
# 前言
最近為了測試及建立 OpenvSwitch 原先採用 ArchLinux 建立, 但不巧 kernel 太新 (ovs 要求 3.10.x 以下), 導致建立失敗, 所以聽取牛人意見, 換了一套 OS 實驗用. 我選擇名為 Scientific Linux. 這套 fork CentOS, CentOS 又是 RetHat Enterprise Linux (RHEL) 開放原始碼的版本, 兩者差異在於 RHEL 有許多商業軟體, 在 CentOS 上都被拿掉了, 但基本上不會差太多, 如果要高穩定性高可用性的 Server, CentOS 是個不錯的選擇. 

之所以會選擇 Scientific Linux 是因為這套主要有費米國立加速實驗室 (Fermilab) 及歐洲核子研究組織 (CERN) 維護, 他們的理念是想要替實驗者重複性工作及建立一個共通且基本的環境, 與我想要的一個穩定且基本實驗環境不謀而合.

曾推薦學弟使用 Scientific Linux 來比 Taiwan Students Cluster Competition (TSCC) 還拿到不錯的成績


# 安裝過程

## 下載 
- [SL-64-x86_64-2013-03-18-boot.iso](ftp://ftp.twaren.net/Linux/scientific/6.4/x86_64/iso/SL-64-x86_64-2013-03-18-boot.iso)
- Kernel Version: 2.6.32-358.el6.x86_64

## 選擇使用 Url 安裝
- 請確保網路順暢
- NameServer隨便填一個 ```168.95.1.1```
- Url 空格填 ```http://ftp.twaren.net/Linux/scientific/6.4/x86_64/os/```

## 選擇最小化安裝 Minimal
<img class="center" src="http://user-image.logdown.io/user/5820/blog/5842/post/165596/tSGQShjSQd5A0FDZDGIB_Capture.PNG" alt="sl_minimizw.PNG">
- 如果想要知道 Minimal 裝了哪些套件, 可以點選 "Customize Now" 一個一個看. 
- 如果現在沒有選到的套件, 之後如果想要安裝, 可以透過 yum 來搞定

## 登入後處理
- 裝 ssh 及 vim 方便遠端連線 ```yum install openssh-server vim -y```
  - 記得要 ```service sshd start``` 開啟 ssh server 服務
- 換 miror, 換台灣的 mirror 會快上好幾倍, 就不用從外國慢慢拉檔回來
```
yum install yum-plugin-fastestmirror -y 
cp /etc/yum.repos.d/sl6x.repo /etc/yum.repos.d/sl6x.repo.bak #備份 
sed -i 's/#mirrorlist/mirrorlist/g' /etc/yum.repos.d/* #拿掉mark
yum update -y #更新
reboot #收工
```
- 對時 
```
yum install ntp -y
service ntpd start && chlconfig ntpd on
cp /usr/share/zoneinfo/Asia/Taipei /etc/localtime
```
- 設定網路
  - 位置在```/etc/sysconfig/network-scripts/ifcfg-eth0``` 修改他即可
  - 可參考[鳥哥](http://linux.vbird.org/linux_server/0130internet_connect.php)

## (Options) 第三方套件庫 Rpmforge
```
rpm --import http://apt.sw.be/RPM-GPG-KEY.dag.txt #匯入信任dag的金鑰
wget http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.3-1.el6.rf.x86_64.rpm
rpm -iK rpmforge-release-0.5.3-1.el6.rf.x86_64.rpm
```
利用這種安裝方式, 可以符合偶而想要用到第三方套件的需求 (例如: htop), 但又不想要影響到穩定性, 盡可能還是跟隨官方更新套件, 以保持穩定.

## (Options) 關閉一些不必要 Service 
```
# 可參考 http://portable.easylife.tw/1816 決定一下自身需求
service auditd stop && chkconfig auditd off
service postfix stop && chkconfig postfix off
```

## 最後附上一張 htop
<img class="center" src="http://user-image.logdown.io/user/5820/blog/5842/post/165596/7OLwMZJRECXCP0N0XV1A_Capture.PNG">

#	結語
當然還有其他 Linux Distribution 可以更輕量化, 如 ArchLinux, 但是要安裝的所需基本技能及知識相對的也要更要求, 或許不是每個人都很喜歡在 Linux 建立上打轉, 所以這篇主要目的是想要讓實驗者有一個最基本到不行的實驗Linux環境, 可提供非常穩定及有效的支援任何科學實驗或者是計算, 希望實驗者都能順利的進行實驗.   

# 引用來源
- [Scientific Linux 官網](https://www.scientificlinux.org/)
- [Scientific Linux DistroWatch](http://distrowatch.com/table.php?distribution=scientific)
- [Scientific Linux Public Mirror](http://www.scientificlinux.org/download/mirrors)