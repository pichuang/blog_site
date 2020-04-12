layout: post
title: '導讀 Reference Architecture: Red Hat Ceph Storage'
author: Phil Huang
date: 2019-04-27 01:37:31
tags:
  - ceph
  - storage
  - sds
categories:
  - infra
toc: true
---
本文主要是以 Red Hat Ceph Storage (RHCS) 及依據兩份 [Brief: How to configure Red Hat Ceph Storage][1] 和 [Reference Architecture: Red Hat Ceph Storage hardware selection guide][2] 文件為基礎撰寫，現行Red Hat 官方發布的版本號來到 [v3.2][8]

![](/images/ceph.png)

<!--more-->

## Red Hat Ceph Storage
一座 Ceph 集群會具備以下五種角色:
- Management/Dashboard node (ceph-mgr)
- Monitor nodes (ceph-mon / ceph-osd-container)
- Object Gateway nodes (ceph-radosgw / ceph-radosgw-container)
- MDS nodes (ceph-mds / ceph-mds-container)
- OSD nodes (ceph-osd / ceph-osd-container)

一般討論最多的就是 OSD nodes 的原始空間大小及可用空間大小 (因要推算硬體規格及評估效能)，其餘角色都是使用或維運時會使用

對於角色作用可以參考 [Ceph 分散式儲存系統介紹 - KaiRen's Blog][16] 的內容

## Total Solution

### 規模定義

基本上只要是屬於 OSD Node 角色的，每一台主機都會配置一個 OSD (Object Storage Daemon)，用戶主要是將資料存取於此使用。而空間定義上主要分為原始空間 (Raw Capacity) 跟依據不同集群優化策略 (3 * Replication / Erasure Coding) 推算後的可用空間 (Usuable Capacity)

規模 | 可用空間大小
---|---
Small|250 TB+
Medium|1 PB - 2 PB
Large| 2 PB+

### 集群優化定義

集群優化定義|特性|舉例
---|---|---
IOPS 優化|- Lowest cost per IOPS <br> - Highest IOPS per GB <br>- 99% latency consistency|- Block storage <br>- 3 x replication<br>- MySQL on OpenStack Clouds
Throughput 優化|- Lowest cost per MBps (throughput) <br>- Highest MBps per TB<br>- Highest MBps per BTU<br>- Highest MBps per Watt<br>- 97% latency consistency|- Block or Object storage<br>- 3 x replication<br>- Streaming media/data/images
Cost/Capacity 優化|- Lowest cost per TB<br>- Lowest BTU per TB<br>- Lowest Watts required per TB|- Object storage<br>- Erasure coding<br>- Object archive

### Referencea Archtictures (RAs)
建議於最小規模 (Small) 最少需三台實體機，而每台 Ceph Server 最小規格，可參考 Red Hat Partner 推薦型號

- [QCT QxStor Red Hat Ceph Storage Edition - QxStor RCT-200][3]
- [Dell PowerEdge R730xd Performance and Sizing Guide for Red Hat Ceph Storage][12]
- [Cisco UCS S3260 Storage Server and Red Hat Ceph Storage Performance][13]
- [Lenovo Reference Architecture: Red Hat Ceph Storage][18]
- [Micron RA: 9200 MAX NVMeTM With 5210 QLC SATA SSDs for Red Hat® Ceph Storage 3.2 and BlueStore on AMD EPYCTM][20]

倘若想自行精算配置的話，則可參考此篇 [Technical Detail: Red Hat Ceph Storage on Servers with Intel Processors and SSDs][10] 推算硬體規格



## About BlueStore

另因 Red Hat Ceph Storage 3.2 宣布可使用 `BlueStore` 方式部署，底下是 BlueStore v.s. Filestore 的技術堆疊比較

![](/images/ceph-2.png)


整體 IOPS 會跟硬碟規格選型會呈現完全正相關，故硬體越好越快

From [BlueStore Unleashed][15]
![BlueStore Unleashed](/images/ceph-1.png) 

除了硬體以外，Ceph 參數調教得當的話可以，讓系統 Latency 及 IOPS 的表現都會非常好

![](/images/ceph-3.png)

### Red Hat Ceph Storage 3.2 PoC 評測文
- [Part - 1: BlueStore (Default vs. Tuned) Performance Comparison][17]
- [Part - 2: Ceph Block Storage Performance on All-Flash Cluster with BlueStore backend][21]
- [Part - 3: RHCS Bluestore performance Scalability (3 vs 5 nodes)][22]


## References
- [Brief: How to configure Red Hat Ceph Storage][1]
- [Reference Architecture: Red Hat Ceph Storage hardware selection guide][2]
- [QCT: QxStor Red Hat Ceph Storage Edition][3]
- [Complete OpenStack Storage from Red Hat][4]
- [Technical Detail: Red Hat Data Analytics Infrastructure Solution][5]
- [RED HAT BLOG - Why Spark on Ceph? (Part 3 of 3)][6]
- [Red Hat Ceph Storage Hardware Selection Guide - CHAPTER 6. RECOMMENDED MINIMUM HARDWARE][9]
- [Technical Detail: Red Hat Ceph Storage on Servers with Intel Processors and SSDs][10]
- [Ceph USE CASES / REFERENCE ARCHITECTURES][14]
- [BlueStore Unleashed][15]
- [Ceph 分散式儲存系統介紹 - KaiRen's Blog][16]
- [Lenovo Reference Architecture: Red Hat Ceph Storage][18]
- [Micron Accelerated Solutions for Red Hat Ceph Storage][19]
- [Reference Architecture: Micron® 9200 MAX NVMeTM With 5210 QLC SATA SSDs for Red Hat® Ceph Storage 3.2 and BlueStore on AMD EPYCTM][20]
- [Part - 1: BlueStore (Default vs. Tuned) Performance Comparison][17]
- [Part - 2: Ceph Block Storage Performance on All-Flash Cluster with BlueStore backend][21]
- [Part - 3: RHCS Bluestore performance Scalability (3 vs 5 nodes)][22]
- [Part – 4 : RHCS 3.2 Bluestore Advanced Performance Investigation][23]


[1]: https://www.redhat.com/en/resources/red-hat-ceph-storage-hardware-selection-guide
[2]: https://www.redhat.com/en/resources/how-configure-red-hat-ceph-storage
[3]: http://go.qct.io/solutions/software-defined-storage/qxstor-red-hat-ceph-storage-edition/
[4]: https://www.redhat.com/cms/managed-files/st-complete-openstack-storage-f13309wg-201807-en_0.pdf
[5]: https://www.redhat.com/cms/managed-files/st-data-data-analytics-infrastructure-technology-detail-f14280-201811-en.pdf
[6]: https://www.redhat.com/en/blog/why-spark-ceph-part-3-3
[7]: https://rhcs-test-drive.readthedocs.io/en/latest/#getting-to-know-red-hat-ceph-storage
[8]: https://access.redhat.com/documentation/en-us/red_hat_ceph_storage/3.2/html-single/release_notes/index
[9]: https://access.redhat.com/documentation/en-us/red_hat_ceph_storage/3/html/red_hat_ceph_storage_hardware_selection_guide/ceph-hardware-min-recommend
[10]: https://www.redhat.com/cms/managed-files/st-ceph-storage-intel-configuration-guide-technology-detail-f11532-201804-en.pdf
[12]: https://downloads.dell.com/solutions/cloud-solution-resources/Dell_R730xd_RedHat_Ceph_Performance_SizingGuide_WhitePaper.pdf
[13]: https://www.cisco.com/c/en/us/products/collateral/servers-unified-computing/ucs-s-series-storage-servers/Whitepaper_c11-738915.html
[14]: https://ceph.com/use-cases/#red-hat-ceph-storage-on-intel-processors-and-ssds
[15]: https://f2.svbtle.com/ceph-bluestore-unleashed
[16]: https://k2r2bai.com/2015/11/19/ceph/ceph-intro/
[17]: https://ceph.com/community/bluestore-default-vs-tuned-performance-comparison/
[18]: https://lenovopress.com/lp1147.pdf
[19]: https://www.micron.com/solutions/micron-accelerated-solutions/micron-accelerated-solutions-for-ceph-storage
[20]: https://www.micron.com/-/media/client/global/documents/products/other-documents/5210_9200_amd_ceph_reference_architecture.pdf?la=en
[21]: https://ceph.com/community/ceph-block-storage-performance-on-all-flash-cluster-with-bluestore-backend/
[22]: https://ceph.com/community/part-3-rhcs-bluestore-performance-scalability-3-vs-5-nodes/
[23]: https://ceph.com/community/part-4-rhcs-3-2-bluestore-advanced-performance-investigation/