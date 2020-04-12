layout: post
title: 2019 大改造宅宅雲架構系列文-1 總綱及裝備清單
author: Phil Huang
tags:
  - linux
  - infra
  - homecloud
categories:
  - homecloud
date: 2019-11-14 23:39:00
toc: true
---
最近這兩三年發現技術越變越快，而且還越變越多，等著別人給我資源學習，不如還是靠自己準備資源來做一些 Lab 性質的測試，但台北花費真的是太高了，薪水不太夠付電費，所以為了節省電費救救錢包君，所以把整個家裡的宅宅雲改造了一下。

本系列文還蠻適合想要嘗試在家裡土砲基礎架構做一些奇怪測試的人交流，雖然我覺得應該蠻多人都知道相關的技術了，只是整合在一起的樣子可能不是很多人看過，反正就當心路歷程看看吧。

<!--more-->

## 宅宅雲總體戰略目標

1. 從世界各地的某個角落，遠端啟動或關閉**整座**宅宅雲
2. 宅宅雲要支援多網段，方便將不同目的或專案在不同網段上切割實作
3. 所有機器需具備 FQDN，宅宅雲內部的所有機器都可正反解
4. 懶得每次都在那邊點例外處理的按鈕，所以自簽 SSL/TLS 而有用到的服務全都要上
5. 所有服務，能虛擬化的全部採用虛擬化
6. 因為沒錢，所以不考慮任何高可用性的問題
7. 無聊的設定工作，做超過三次，考慮或使用 Ansible 自動化流程
8. 建立 Chat Center，用 bot 跟自己聊天

## HLD 架構圖

![](/images/homecloud-1.png)

## 核心硬體裝備

打英雄聯盟的時候，每個角色都會有所謂的核心裝備，而相同的在做基礎架構，我個人推薦也真的有使用的核心裝備如下：

1. SSL VPN: Synology RT2600ac
2. Bastion: Lenovo T480s (Red Hat Employee Special Edition)
3. Storage: Synology DS916+
4. 宅宅雲: 自組塔式伺服器 (內建 IPMI)

在平時沒用宅宅雲的狀態下，1/2/3 項次會保持全年全天開機常駐的狀態，有需要做 Lab 的時候才會把 4 開起來，因為最吃電

### SSL VPN: Synology RT2600ac

有一天的晚上，發現 Synology RT2600ac 除了有一般 802.11ac / DHCP 等家用功能以外，還有 VPN Plus 的功能可以使用，所以就開啟了大改造宅宅雲之路。

Q: 為什麼會說改造呢?

因為一開始的時候 WAN 是走 4G USB Dongle 上網的，SSL VPN 不能用，後來退掉換成光世代，申請 1+7 固定 IP 方案，才能正常運作 SSL VPN。此外，Synology VPN Plus 預設只有授權一個人使用，要多人連線的話要另外採購授權，詳細請洽他們業務。

![](/images/vpn-plus.png)

Q: 那有沒有可以替代的方案呢?

答案是有的，只要有服務能支援 OpenVPN / IPSec (例: pfSense / OPNsense / ...) 能上 Public Static IP 都可以當作 VPN Server 使用，我個人先前是比較傾向用 OpenVPN on VM 為主，架構可參考以前寫的 [Build Testing Infrastructure - Phil Huang][1]。家用因為還有其他設備需要無線上網 (手機 / 筆電 / Chromecast / PS4 / NS)，所以把主要網路核心放在 RT2600ac 身上。


### Bastion: Lenovo T480s (Red Hat Employee Special Edition)

這台是入職 Red Hat 的時候配發的筆電，老闆說：**身為一個 Red Hat Infra Solution Architect 就應該要灌 RHEL / Fedora...**，恩...所以我把他當堡壘機用，沒事不關機。

既然身為一台堡壘機，就是期望在什麼狀況下都要屹立不搖，包含停電和瞬斷，剛好這台是個筆電，內建有電池，平時又插著電源，故入職後到進行大改造宅宅雲之前，累積了 386 天的 Uptime，上次養這麼久的 Server 應該是研究所的時代了... (菸

![](/images/rhel7-poweron.jpg)

Q: 他既然是個堡壘機，那幹嘛動它?

理由是資源使用率太低了，先前只是寫寫一些怪東西，跑幾個 Ansible Playbook，絕大部分時間空有 8C / 24G 在那邊閒置，不如讓他做點有意義的事情，所以改造如下：

#### RHEL 7.7 重灌成 RHEL 8.1
![](/images/rhel8-1.jpg)

剛好順便要開始熟悉 RHEL 8 系列了，所以趁機換裝練習看看，關於詳細的新功能條列可以參考 [Red Hat Enterprise Linux 8.0 正式發布 - 開源工場][5]


#### RHEL 8.1 裝備 Cockpit 管理 VM

![](/images/rhel8-2.jpg)

基礎就是使用 KVM libvirt，所以只要建好 Linux Bridage 對接 Physical NIC 即可好好使用........................................

Q: 為什麼要點點點...

因為 RHEL 8 開始要開始宣揚 NetworkManager 的美好 (?) 了，所以如果你習慣開場把 NM 關掉的小夥伴 (小弟我)，會發現到一些神祕的現象，所以眾多考量下，還是學一下 `nmcli`。關於 Linux Bridge with nmcli 詳細步驟請參閱 [Create a Linux Network Bridge on RHEL 8 / CentOS 8 with nmcli][4]，步驟寫的很乾淨的一篇。

Q: VM 主要跑什麼?

最主要是` Ansible 自動化`，過往我都是寫完 Ansible Playbook 就直接在堡壘機上執行，但我後來實在是寫到花掉，忘記怎麼下，所以就動用了紅帽員工的神力建了 Red Hat Ansible Tower 起來，後來我的所有操作都在 Ansible Tower 上面，包含用 IPMI 去開關機宅宅雲...

![](/images/homecloud-powerstate.png)

[工商時間] 就在 2019/11/14 這天 Red Hat Ansible Tower 發佈了 3.6.1 版本，自帶 Approval 按紐啊啊啊啊啊啊!!!! 

![](/images/ansible-tower-approval.png)

關於我之前未改造前的 Ansible Tower 的定位，後面會找個一篇講下，這服務對我個人使用來說跟 SSL VPN 同等重要。

#### RHEL 8.1 裝備 Cockpit 管理 Container 

![](/images/rhel8-3.jpg)

這邊要特別講一下關於 Container 的事情，Red Hat 目前主要推行的容器專案是以 (下圖由左至右) Podman / Buildah / Skopeo 為主，不主推 Docker 為主的專案，但兩者都是支援 OCI (Open Container Initiative) Runtime 的標準規範，所以在使用上起來幾乎是大同小異，有興趣想要使用同一份 Dockerfile 自己用 podman 及 docker 編起來試試看的，可以參考 [pichuang/debug-container][3] 內有 Makefile 方便理解；若想要理解關於 OCI 的兩三事，請參考 [[Day3] 淺談 Container 實現原理, 探討 OCI 實作 - hwchiu][2]

![](/images/buildah-podman-skopeo.png)

### Storage: Synology DS916+

NAS 理所當然地被我拿來當 NFS Server 裡面存一些特殊性資料

除了 NFS 的服務以外，全站的 DNS Server 我也是放在上面，不得不說...這個 S 社 GUI 真的做得還可以啊...DNS Entry 點點敲敲就有了。

回到架構面上，NAS 很少人在關機吧？個人判斷 NAS 被關機機會基本上還是會小於堡壘機 (斷電除外)，剛好宅宅雲的建置條件都有配置內部的 FQDN，我沒有習慣寫 `/etc/hosts`，基於 `/etc/nsswitch.conf` 內指示

```
cat /etc/nsswitch.conf
...omit...
hosts:      files dns myhostname
...omit...
```

所以 DNS 對宅宅雲來說跟 NFS 服務一樣角色吃重，所以選擇放在跟 NAS 一起的位置。

### 宅宅雲: 自組塔式伺服器

這台資訊太多之後下一篇講 XD
傳送門: [2019 大改造宅宅雲架構系列文-2][6]


## 總結

這篇主要是留個紀錄，基本架構都是基於先前在 Edgecore Network 工作時自建遠端環境的經驗 [Build Testing Infrastructure - Phil Huang][1] 轉換情境到家中這種小規模時的架構紀錄，沒有絕對對錯 (有一些物理上的考量)，僅供參考


[1]: https://speakerdeck.com/pichuang/build-testing-infrastructure?slide=17
[2]: https://www.hwchiu.com/container-design-ii.html
[3]: https://github.com/pichuang/debug-container
[4]: https://computingforgeeks.com/how-to-create-a-linux-network-bridge-on-rhel-centos-8/
[5]: https://openingsource.org/6629/zh-tw/
[6]: https://blog.pichuang.com.tw/20191115-homelcloud-high-level-design-2
