layout: post
title: '導讀 Intel Select Solutions: Red Hat OpenShift Container Platform'
author: Phil Huang
tags:
  - intel
  - openshift
  - redhat
categories:
  - openshift
date: 2019-04-21 01:54:00
toc: true
---
最近 Intel 不知道吹什麼風，發了一個 [Intel® Select Solutions for Red Hat OpenShift Container Platform - April, 2019][1] 出來，這篇文章蠻適合當作 Hardware Configuration 的指南進行基礎探討

![](/images/intel-openshift.png)

<!--more-->

## What are Intel Select Solutions?

[Intel® Select 解決方案 (Intel® Select Solutions)][5] 是經過驗證的硬件和軟件全套解決方案，針對計算 Compute，存儲 Storage 和網絡 Network 中的特定工作負載進行了優化。 

除了與世界領先的數據中心和服務提供商的廣泛合作之外，這些解決方案還源自英特爾與行業解決方案提供商的深厚經驗。要獲得英特爾精選解決方案的資格，解決方案提供商必須：

1.遵循 Red Hat 和 Intel 共同合作的軟體和硬體堆棧要求
2.複製或超過英特爾的參考基準性能基準
3.發布詳細的實施指南以促進客戶部署落地

## Red Hat OpenShift Container Platform 3.11

Red Hat OpenShift Container Platform 3.11 是長期支援版本 (Long-Term Support, LTS)，也是現行最穩定的容器平台服務，同時能支持使用 Ansible Playbook 以 in-place upgrade 小版本進行軟體修補

對於該平台有興趣的朋友可以參考以下文章
- [那些年的 OpenShift 3.11 容器平台技術選型_20190122][6]
- [Red Hat OpenShift v3.11 東西南北向網路探討][7]
- [OpenShift Container Platform 3.11 Release Notes][8]

## Total Solution

下列章節摘錄 [適用於 Red Hat OpenShift Container Platform* 的 Intel® Select 解決方案之基本與加強組態][12] 一文

### Hardware Configuration

文件提供兩個不同的 Configuration:
- Base
- Plus

其中 Plus 可以提供 2.25 倍以上的並發工作負載 (concurrent workloads) 和 1.9 倍以上的同時上線用戶 (concurrent users)，響應速度提高 2 倍 (response times)

Intel + Red Hat 聯名文件主要推薦配置為以下 (以 Base 最小等級為例):
- Control-plane nodes (20c / 192GB RAM / 480G for boot / X710 Dual Port SFP+)
  - Bation node * 1 
  - Master nodes * 3 
  - Infrastructure nodes * 2
- Applications nodes (20c / 256GB RAM / 480G for boot / 4TB for data / X710 Dual Port SFP+)
  - Applications nodes * 4
- (Optional) Storage nodes (20c / 256G RAM / 480G for boot / 16 * 3.8TB for data / X710 Dula Port SFP+)
  - Storage nodes * 3
  
至於詳細的 BOM (Bill Of Material) 表請參閱 [Intel® Select Solutions for Red Hat OpenShift Container Platform - April, 2019][1] 

### Minimum Performance Capabilities

主要量測的工具有 3 個
- [Magento 2.2.2][2]
- [Apache JMeter 4.0][3]
- [Sysbench][4]

Target|Base|Plus
---|---|---
部署至所有應用程式節點的 Magento* 實例工作負載之最小數量|32|72
Sysbench* Pods 的數量（部署至兩個基礎架構節點）	|32|72
並行使用者的最小數量（Apache Jmeter* 效能標竿 [使用者網路流量] 部署至 Bastion 節點）	|725|1400
應用程式輸送量（每秒平均異動數量）|310|620


### 同場加映: OpenShift Partner Reference Architectures

[OpenShift Partner Reference Architectures][10] 集合了各大廠對於 Red Hat OpenShift 安裝時的硬體配置建議，不仿可以參考看看

[OpenShift Container Platform Reference Architecture Implementation Guides][11] 則收集了 Red Hat - OpenShift 安裝於各大環境上時的實作架構圖，包含:
- Amazon Web Services (AWS)
- Microsoft Azure
- Google Cloud Platform (GCP)
- Red Hat OpenStack Platform
- VMWare vSphere
- Red Hat Virtualization (RHV).

## References
- [Intel® Select Solutions for Red Hat OpenShift Container Platform][1]
- [magento/magento2 - GitHub][2]
- [How to benchmark Magento Open Source with Apache JMeter][3]
- [akopytov/sysbenc - GitHub][4]
- [Intel® Select 解決方案 (Intel® Select Solutions)][5]
- [那些年的 OpenShift 3.11 容器平台技術選型_20190122][6]
- [Red Hat OpenShift v3.11 東西南北向網路探討][7]
- [OpenShift Container Platform 3.11 Release Notes][8]
- [Reference Architecture: Red Hat OpenShift Container Platform on Lenovo ThinkSystem Servers][9]
- [OpenShift Partner Reference Architectures][10]
- [OpenShift Container Platform Reference Architecture Implementation Guides][11]
- [適用於 Red Hat OpenShift Container Platform* 的 Intel® Select 解決方案之基本與加強組態][12]

[1]: https://www.intel.com.tw/content/www/tw/zh/products/docs/select-solutions/select-solutions-for-red-hat-openshift-container-platform-brief.html
[2]: https://github.com/magento/magento2/tree/2.3-develop/setup/performance-toolkit
[3]: https://upcloud.com/community/tutorials/benchmark-magento-with-jmeter/
[4]: https://github.com/akopytov/sysbench
[5]: https://www.intel.com.tw/content/www/tw/zh/architecture-and-technology/intel-select-solutions-overview.html
[6]: https://speakerdeck.com/pichuang/na-xie-nian-de-openshift-3-dot-11-rong-qi-ping-tai-ji-shu-xuan-xing-20190122
[7]: https://blog.pichuang.com.tw/20190404-openshift-network-traffic-overview/
[8]: https://docs.openshift.com/container-platform/3.11/release_notes/ocp_3_11_release_notes.html
[9]: https://lenovopress.com/lp0968.pdf
[10]: https://blog.openshift.com/openshift-partner-reference-architectures/
[11]: https://blog.openshift.com/openshift-container-platform-reference-architecture-implementation-guides/
[12]: https://www.intel.com.tw/content/www/tw/zh/products/solutions/select-solutions/cloud/red-hat-openshift-container-platform.html