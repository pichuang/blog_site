layout: post
title: 2019 大改造宅宅雲架構系列文-2 宅宅雲硬體介紹
author: Phil Huang
tags:
  - linux
  - infra
  - homecloud
categories:
  - homecloud
date: 2019-11-15 13:21:00
---
承上篇 [2019 大改造宅宅雲架構系列文-1 - Phil Workspace][1] 所提到一些關於核心裝備的列舉和使用理由，本篇會比較著重在宅宅雲塔式 Server 的介紹

<!--more-->

## 宅宅雲: 自組塔式伺服器

這台是我在 2018 年尾的時候採購的一台塔式伺服器，當初會買這台的理由非常簡單，`為了深刻理解 Red Hat OpenShift 3.11 on VMware vSphere 跑起來的樣子會長什麼樣子`，我自稱這個叫做 `客戶驅動部署 Customer Driven Deployment` (???)

但因為這台會放在我住宿的地方，所以我一開始的設計要求如下：

1. 安靜第一第二第三
2. 效能第四
3. 省電第五

所以整體上的選擇就是`靜音`為優先的設計，實際上開機後也真的是蠻安靜的，比 Synolgy DS916+ 裡面硬碟運轉的聲音還小...

### 硬體菜單

![](/images/tower-details.png)

以前在 Mobile01 不知哪裡看到一篇開箱文，跟我需求蠻像的，所以拿來修改成如下：

- CPU: Intel Xeon E5-2620 v4 (8C/16T) / 2011 / 2.1G / 無內顯 * 1
- MB: ASUS Z10PE-D16 WS (E-ATX / C612 / 雙 2011-V3.V4 / 16*DDR4 / M.2 / 2 * NIC) * 1
- Memory: Samsung 16GB DDR 2133MHz ECC Reg * 16
- OS Disk: HIKVISION E100NI 256G / M.2 SATA / R:500M / W 250M / TLC * 1
- 機箱: Corsair Obsidian 750D Airflow 顯卡長45 / CPU高17 / SSD x 8 (6共用) / 前面板網孔 /E-ATX  * 1
- CPU 散熱器: Corsair H60 12cm冷排/ 扁平化幫浦/ (方)銅底冷頭/ 厚:5.2cm * 1
- 裝機顯卡: ASUS GT 710-1-SL / 954MHz / 1G DDR3 / 靜音版 / 13.5cm * 1
- Power: be quiet! Straight Power 11(E11) 650W/金牌/全模/5年保/LLC+DC-DC/CPU主線:16AWG * 1
- (Optional) 外掛網卡: Intel i350-T4 * 1

Q: 該買?

光華歡迎你，最近記憶體便宜到爆炸錢包君覺得非常哀傷


### 詳細說明

關於計算資源，[Intel E5-2620 v4][6] 一顆很貴啊...錢包君真的會吃土，所以那時候先只買一顆，然後記憶體先買滿，但實際上只會用到一半而已。等人捐贈另一顆 E5-2620 v4 或等拋售 QQ。另外要注意的地方是，Memory Types 僅支援到 DDR4 2133 ECC Reg，所以一開始沒注意買 DDR4 2600 ECC Reg 當冤大頭。

關於機箱，直接看別人開箱文章 [Corsair Obsidian 750D ATX機殼之裝機使用分享][2] 比較快，配上 Corsair H60 散熱整體上蠻安靜的，如果有點聲音的話拿吸塵器吸一吸風扇就好了。然後這個機箱給 E-ATX 主機板，所以真的是蠻大台的，相對擴充性也蠻好的，等待好心人捐 Nvidia V100 / T4 玩玩。

![](/images/tower-view.jpg)

關於電源，無愛好，我就是被 `be quiet!` 這名字打到，**產品命名很重要**

關於網路，這個就蠻重要的，首先主機板 [ASUS Z10PE-D16 WS ][3] 有加裝一個 BMC，支援相容於 `IPMI 2.0` 的 ASMB8-iKVM 晶片，所以就可以用 `ipmitool` 或 [` pyghmi`][4] 來做 OOB 遠端伺服器管理，個人使用主要都是用 [ansible/ipmi_power][5] 這個模組為主。

Q: 如果主機板沒有 BMC，那有沒有替代 IPMI 的方案?

有，但我個人覺得不太好，PDU 遠端 IPMI 切開關電源的方式，或者是一些 IoT 有類似這種遠端開關電源的方案。這個要謹慎設計使用，因為跟直接拔電源沒啥兩樣 ，屬於冷關機，如果真要用的話，記得要先做熱關機，再切電源比較好。

> Ansible Workflow <3 你


另外主機板有提供兩個內建 1Gb 可以用，這邊選了一個當 WAN 使用，這應該是沒什麼懸念的

![](/images/tower-nics.jpg)

Q: 為什麼會有一個 Intel i350-T4 在那邊晾著?

完全是一個腦衝的結果，原先預期目標是要做以這台塔式 Server 為核心的架構設計，做 Two-Arm，但看了看覺得有點顧慮就不搞了。如果對網路處理吃重的，可以把 `SR-IOV` 開起來，可以直接有 8 VFs 可以使用，網路效率會蠻好的。

後面會有一篇解釋 {pfSense, OPNsense, ooxx UTM} on {vSphere, Proxmox, KVM Libvirt} 虛擬網路設計怎麼做 Two-Arm，自建 Lab 來說還蠻好用的架構

## 總結

整體上來說，IPMI 這個功能對於我從 Bastion 堡壘機上，透過 Ansible Tower 控制服務器來講非常重要，其他資源都是為了要承載 Lab 所需要的運算資源而生的，可以自行做刪減。


[1]: https://blog.pichuang.com.tw/20191114-homelcloud-high-level-design-1/
[2]: https://www.mobile01.com/topicdetail.php?f=299&t=5074578
[3]: https://www.asus.com/tw/Motherboards/Z10PED16_WS/
[4]: https://pypi.org/project/pyghmi/
[5]: https://docs.ansible.com/ansible/latest/modules/ipmi_power_module.html#ipmi-power-module
[6]: https://ark.intel.com/content/www/us/en/ark/products/92986/intel-xeon-processor-e5-2620-v4-20m-cache-2-10-ghz.html