layout: post
title: 雲端大廠們的 VM 資源配置參考
tags:
  - cloud
  - azure
  - aws
  - gcp
categories:
- misc
date: 2018-08-15 23:55:00
udpated: 2018-08-15 23:55:09
---


## 前言
對於多數系統硬體資源配置多數就是一個公版 Instance Template 為基準，透過監控來進行效能調整或組件置換。本篇提供大家一個簡單的選擇方向，各大 CSP (Cloud Service Provider) 分別都有提供對應的 VM 實例資源配置參考，可先用`使用情境`進行初始評估，再依據經驗選擇不同大小的資源 (e.g. medium / large / xlarge / ...)，後續如果真心不夠用可以再 resize 資源即可。

<!--more-->

目前有公開出來的 VM 資源配置參考如下：
- [Google GCP][1]
- [Amazon AWS][2]
- [Microsoft Azure][3]

## 各大廠配置清單
### Amazon AWS

AWS 提供了針對幾個使用情境來供各位選擇 [Amazon AWS Machine Type][2]

1. 一般用途 (General Purpose) T2 / M5 / M4
  - T2: 網站和 Web 應用程式、開發環境、建置伺服器、程式碼儲存庫、微型服務、測試和模擬環境，以及企業營運應用程式
  - M5 / M4: 用於小型和中型資料庫、需要附加記憶體的資料處理任務以及快取叢集，也用於執行 SAP、Microsoft SharePoint、叢集運算和其他企業應用程式的後端伺服器
2. 運算優化 (Compute Optimized) C5 / C4
  - C5 / C4: 高效能 Web 伺服器、科學建模、批次處理、分散式分析、高效能運算 (HPC)、機器/深度學習推論、廣告服務、可高度擴展的多玩家遊戲，以及影片編碼
3. 記憶體優化 (Memory Optimized) X1e / X1 / R5 / R4 /z1d
  - X1e / X1: 高效能資料庫、記憶體內資料庫 (如 SAP HANA)，以及記憶體密集型應用程式。
  - R5 / R4: 高效能資料庫、分散式 Web 規模記憶體內快取、中型記憶體內資料庫、即時大數據分析，以及其他企業應用程式
4. 加速運算 (Accelerated Computing) P3 / P2 / G3 / F1
  - P3 / P2: 機器/深度學習、高效能運算、計算流體動力學、計算金融學、地震分析、語音辨識、自動駕駛汽車、藥物研發
  - G3: 3D 視覺化、圖形密集型遠端工作站、3D 轉譯、應用程式串流、影片編碼和其他伺服器端圖形工作負載
  - F1: 基因體研究、財務分析、即時影片處理、大數據搜尋和分析，以及安全性
5. 儲存優化 (Storage Optimized) H1 / I3 / D2
  - H1: MapReduce 工作負載、分散式檔案系統 (如 HDFS 和 MapR-FS)、網路檔案系統、日誌或資料處理應用程式 (如 Apache Kafka) 以及大數據工作負載叢集
  - I3: NoSQL 資料庫 (如 Cassandra、MongoDB、Redis)、記憶體內資料庫 (如 Aerospike)、可擴展交易處理資料庫、資料倉儲、Elasticsearch、分析工作負載
  - D2: 大規模並行處理 (MPP) 資料倉儲、MapReduce 和 Hadoop 分散式運算、分散式檔案系統、網路檔案系統、日誌或資料處理應用程式

[EC2 Instance Type](https://www.ec2instances.info/) 這網站提供即時 EC2 VM Instance 配置清單，可以直接搜尋需要的資料

### Google GCP

Google GCP 也針對幾個使用情境來供各位選擇，如果有在看 Kubernetes 相關技術的人，想要落地在自家環境，或許可以以 GCP 配置為主，AWS / Azure 作為參照，這邊特別一點的是 GPU 的資源估算，可以做為參考依據

[Google GCP Machine Type][1]

1. Standard machine types
2. High-memory machine types
3. High-CPU machine types
4. Memory-optimized machine types
5. [GPU][4]

### Microsoft Azure
Microsoft 之所以為 Microsoft 就是因為有 Windows OS，若自家環境有需要考量以 Windows 為主的環境，下列連結就是主要環境配置參照對象
[Azure 中 Windows 虛擬機器的大小][3]

當然 Azure 也有提供 Linux 為主配置選項，支持服務在混合雲上的應用
[Azure 中的 Linux 虛擬機器大小][5]

Microsoft 所提供的資源預估還蠻全的，連 RDMA 的應用都有放進去

## Reference
- [Google GCP - Machine Types][1]
- [Amazon AWS - Machine Type][2]
- [Microsoft Azure - Machine Type][3]

[1]: https://cloud.google.com/compute/docs/machine-types
[2]: https://aws.amazon.com/tw/ec2/instance-types/
[3]: https://docs.microsoft.com/en-us/azure/virtual-machines/windows/sizes
[4]: https://cloud.google.com/compute/docs/gpus/
[5]: https://docs.microsoft.com/zh-tw/azure/virtual-machines/linux/sizes