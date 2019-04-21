layout: post
title: '導讀 Intel Select Solutions: Red Hat OpenStack Platform 13 for NFVi'
author: Phil Huang
tags:
  - openstack
  - redhat
  - intel
  - nfv
categories:
  - openstack
date: 2019-04-21 07:54:00
---
承上篇 [導讀 Intel Select Solutions: Red Hat OpenShift Container Platform][1]，本篇同時收錄 Red Hat OpenStack 13 代號 Queens 的推薦硬體 BOM 表 [Intel® Select Solutions for NFVI v2 with Red Hat OpenStack® Platform (OSP) Release 13][2]，然而想要了解其他 Intel Select Solutions for NFVi 的方案可以參考 [Intel® Select Solutions for NFVI][4]

<!--more-->

## What are Intel Select Solutions?

[Intel® Select 解決方案 (Intel® Select Solutions)][3] 是經過驗證的硬件和軟件全套解決方案，針對計算 Compute，存儲 Storage 和網絡 Network 中的特定工作負載進行了優化。 

除了與世界領先的數據中心和服務提供商的廣泛合作之外，這些解決方案還源自英特爾與行業解決方案提供商的深厚經驗。要獲得英特爾精選解決方案的資格，解決方案提供商必須：

1.遵循 Red Hat 和 Intel 共同合作的軟體和硬體堆棧要求  
2.複製或超過英特爾的參考基準性能基準  
3.發布詳細的實施指南以促進客戶部署落地  

## Red Hat OpenStack Platform 13 Queens

Red Hat OpenStack 13 Queens 是長期支援版本 (Long-Term Support, LTS)，生命週期最多可長達五年，同時也因基礎技術採用 Container 方式服務為主，故於這版支援 Fast Forward Upgrade (FFU)，可以從 13 直接跳往下一個 LTS 版本 16 進行 in-place 升級

![](/images/ffu.png)

## Total Solution
### Hardware Configuration

針對 OpenStack for NFVi 角色主要拆分兩個:
- Controller nodes
- Cloud (or Compute) nodes

而其中 Cloud/Compute nodes 的角色，依據不同的硬體 BOM 表會有兩種: Base、Plus 的區別

Intel + Red Hat 聯名文件主要推薦配置為以下 (以 Base 最小等級為例):
- Controller node
  - 32c
  - 192GB RAM
  - 2 * XL710 Dual Port
  - 2 * LOM (Lan on Motherboard) Port
  - 2 * 480 GB for boot
  - 2 * 2TB for data
  - QAT
- Cloud (or Compute) node
  - 40c
  - 384GB RAM
  - 2 * XL710 Dual Port
  - 2 * LOM (Lan on Motherboard) Port
  - 2 * 480 GB for boot
  - 2 * 2TB for data
  - QAT
  
至於詳細的 BOM (Bill Of Material) 表跟硬體效能指標內容請參閱 [Intel® Select Solutions for NFVI v2 with Red Hat OpenStack® Platform (OSP) Release 13][2]

### Software Configuration

毫無懸念地，你可以參考 [Network Functions Virtualization Product Guide - Red Hat OSP 13 Guide][5] 這裡面的文件來了解 NFVi 的架構是長得如何，下圖是基於標準 NFV ETSI 架構，以 Red Hat 現行產品進行對應的圖表

![](/images/osp-nfv.png)

一般來說 IaaS 的部分是以 Red Hat OpenStack 為主這點是沒錯的，Storage 的技術選型現行是以 Ceph 為優先考量，同時也跟 OpenStack 不同類型的 storage backend project (swift/cinder/glance/...)整合性最好。當然也是可以替換成第三方廠商的方案，並沒有說一定要選擇某個技術才能運行。而網路的部分，因為牽扯到實體網路的整合及最大化利用網路頻寬議題，通常都會在搭配一家 SDN 廠商在內進行 Overlay + Underlay 網路整合，例如以下幾家都是不錯的組合

- Cisco ACI
- Juniper Contrail
- BigSwitch BCF




## References

- [導讀 Intel Select Solutions: Red Hat OpenShift Container Platform][1]
- [Intel® Select Solutions for NFVI v2 with Red Hat OpenStack® Platform (OSP) Release 13][2]
- [Intel® Select Solutions for NFVI][4]
- [Network Functions Virtualization Product Guide - Red Hat OSP 13 Guide][5]
- [Red Hat OpenStack* Platform with Red Hat Ceph* Storage - Intel][6]

[1]: https://blog.pichuang.com.tw/20190421-intel-select-solutions-redhat-openshift-container-platform/
[2]: https://builders.intel.com/docs/networkbuilders/intel-select-solutions-for-nfvi-with-red-hat-enterprise-linux-and-red-hat-openstack.pdf
[3]: https://www.intel.com.tw/content/www/tw/zh/architecture-and-technology/intel-select-solutions-overview.html
[4]: https://builders.intel.com/intelselectsolutions/intelselectsolutionsfornfvi
[5]: https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/13/html-single/network_functions_virtualization_product_guide/index
[6]: https://www.intel.com/content/www/us/en/data-center-blocks/cloud/redhat-cloud-blocks-reference-architecture.html