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

### CPU Selection & Capability

主要的 CPU 選擇是以 Intel Xeon Scalable Processors 為主，CPU 自帶以下能力 (翻中文怪怪的直接貼):

1. Intel® Platform Trust Technology (Intel PTT) or a discrete Trusted Platform Module (TPM) 2.0
  - Protects the system start-up process by ensuring the boot hardware is tamper-free and provides secured storage for sensitive data.

2. Intel®Hyper-ThreadingTechnology (Intel HT Technology)
  - Ensures that systems use processor resources more efficiently and increases processor throughput to improve overall performance on threaded software.

3. Intel Turbo Boost Technology
  - Accelerates processor and graphics performance for peak loads.

4. Intel Speed Shift technology
  - Allows the processor to select its best operating frequency and voltage to deliver optimal performance and power efficiency.

5. Adaptive Double DRAM Device Correction (ADDDC)
  - Offers an innovative approach in managing errors to extend DIMM longevity.

6. Advanced Error Detection and Correction (AEDC)
  - Improves fault coverage by identifying and correcting errors.

7. Local Machine Check Exception (LMCE): Helps improve performance.

### Minimum Performance Capabilities

主要量測的工具有 3 個
- [Magento 2.2.2][2]
- [Apache JMeter 4.0][3]
- [Sysbench][4]

Target|Base|Plus
---|---|---
Minimum number of magento instances workloads deployed to all applications nodes|32|72
Number of sysbench pods (deployed to two infra nodes)|32|72
Minimum number of concurrent users (Apache JMeter benchmark user web traffic from bastion node)|725|1400
Application throughput (Average number of transactions per second)|310|620

## References
- [Intel® Select Solutions for Red Hat OpenShift Container Platform][1]
- [magento/magento2 - GitHub][2]
- [How to benchmark Magento Open Source with Apache JMeter][3]
- [akopytov/sysbenc - GitHub][4]
- [Intel® Select 解決方案 (Intel® Select Solutions)][5]
- [那些年的 OpenShift 3.11 容器平台技術選型_20190122][6]
- [Red Hat OpenShift v3.11 東西南北向網路探討][7]
- [OpenShift Container Platform 3.11 Release Notes][8]

[1]: https://www.intel.com.tw/content/www/tw/zh/products/docs/select-solutions/select-solutions-for-red-hat-openshift-container-platform-brief.html
[2]: https://github.com/magento/magento2/tree/2.3-develop/setup/performance-toolkit
[3]: https://upcloud.com/community/tutorials/benchmark-magento-with-jmeter/
[4]: https://github.com/akopytov/sysbench
[5]: https://www.intel.com.tw/content/www/tw/zh/architecture-and-technology/intel-select-solutions-overview.html
[6]: https://speakerdeck.com/pichuang/na-xie-nian-de-openshift-3-dot-11-rong-qi-ping-tai-ji-shu-xuan-xing-20190122
[7]: https://blog.pichuang.com.tw/20190404-openshift-network-traffic-overview/
[8]: https://docs.openshift.com/container-platform/3.11/release_notes/ocp_3_11_release_notes.html