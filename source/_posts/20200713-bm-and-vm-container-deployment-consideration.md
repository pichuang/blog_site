---
layout: post
title: Kubernetes 部署在虛機好還是裸機好?
author: Phil Huang
toc: true
tags:
  - kubernetes
categories:
  - kubernetes
date: 2020-07-13 00:31:45
udpated: 2020-07-13 00:31:45
---

這幾週常被問到一個問題

> 以技術觀點來看，請問容器 (Container) 或者是容器平台 (Kubernetes) 比較適合部署在虛擬環境 (VM) 中還是裸機環境中 (Bare Metal)?

![](/images/vm-vs-bm.jpg)

其實這個問題在十年前虛擬機技術剛出來的時候，也蠻多人做過架構及性能比較，時光快轉到現在的年代，來到容器的世代，相同的問題也被拿出來討論，本篇就是要特別講述一下關於這些架構上的比較分析、使用影響及建議採用的策略作法建議，這篇會採用比較中立的角度闡述，如果有觀點太偏頗的地方，可以私下跟我說及提供修改建議

不免俗的我還是要來講一下本次新發現，三個感覺不會同框的公司 Microsoft + Red Hat + HPE 一起出了本架構書 [HPE Reference Architecture for delivering insight across all your data with Microsoft SQL Server 2019 Big Data Clusters][15] 我覺得實在太玄了 XD 分享給大家

<!--more-->

## 4 個先備小知識

在討論架構上的優劣之前，要先跟各位清楚地解釋這些架構的實際長相是有哪一些

### Containers are Linux

> A linux container is nothing more than a process that runs on Linux

![](/images/container-performance.png)

一開始要先對容器 (Container) 進行名詞說明，其實這個容器這個技術沒什麼特別的黑魔法，也不是相當前衛的東西，技術本質上就是一個或多個 Linux Process 的呈現，是的！你過往使用的 chroot / LXC / Jail 皆屬這類，所以你過往能對 Linux Process 進行一些深度控制的方式，譬如像是 secomp、cgroups、selinux 等操作，都能在 Container 裡面呈現，更甚至是常見的 Linux Performance Obsevability Tools 效能量測工具，也可以使用，實務上可以參考 [nicolaka/netshoot - a Docker + Kubernetes network trouble-shooting swiss-army container][9]

基於以上的理由，所以有時候你會看到下列幾種說法

- Container Security is Linux Security
- Container Performance is Linux Performance
- Container Reliability is Linux Reliability

![](/images/linux-arch.png)

結論就是，你的容器穩不穩定、安不安全跟你所使用的作業系統其實是有 100% 的關係

### Kubernetes is a Container Orchestration Platform

![](/images/k8s-arch.png)

> Kubernetes (K8s) is an open-source system for automating deployment, scaling, and management of containerized applications.

從 [What is Kubernetes?][10] 可知，應用程式透過容器部署方式所帶來的優勢，具備輕量級資源隔離方式、可共用作業系統及資源可高效率及密集使用等眾多好處，但當我的 Container 數量一多，服務變得更大更多的時候，誰能負責幫這些 Container 找到一個適合的地方運行、持續的監控及故障處理? 所以這時候你會需要一個`負責管理容器運行的控制平台`，也就是現今最熱門的 Kubernetes。

Kubernetes 容器平台內建多種協助管理容器的機制，包含但不限於：

1. 內部服務發現和負載均衡
2. 提出統一且標準的聲明式 (Declarative) API：這點應該是 Kubernetes 誕生後，對於整個 IT 界最大的影響了，有興趣想了解更多的可以參考此文 [云原生时代， Kubernetes 多集群架构初探][11]，這篇真的寫得不錯
3. 內建自我修復容器能力：承上，Kubernetes 中的聲明式 API 其實就是*指定該叢集所期望的運行狀態*，所以出現任何不一致的狀態的時候，Kubernetes 可以自行透過指定的 YAML 文件描述，來進行狀態遷移，作出合適於當下環境的操作
4. 易於資源橫向擴充：以 Kubernetes 的角度進行的容器編排，已經將上層服務跟底層基礎設施脫離掛鉤，IT 人員僅需關心 Kuerbernets 所準備的資源夠不夠上層服務使用，進而形成一個資源池 (Resource Pool) 的觀念，包含 CPU Pool / Memory Pool / Storage Pool
5. 可跨裸機、虛擬環境、公私有雲上使用：其實這個真正的理由是因為作業系統的關係，因為 Kubernetes 的運作是基於在作業系統之上，所以只要確保能在作業系統能在任一環境啟動即可使用

而要讓 Kubernetes 控制平台有這些能力必然會需要另外安裝一些所需要的軟體，譬如說每一個 Worker 必然會需要安裝的作業系統、kubelet、CNI 網路套件等一些常駐軟體，所以一般來講最終能供應用程式所使用的資源不全然是等同於外面的規格，所以針對 Kubernetes 平台建置才會有一種趨勢是`作業系統盡量精簡到僅需可以乘載容器運作即可，其他多餘的都可以拿掉，以達到最大資源利用率`，進而誕生各式各樣的`容器作業系統 (Container OS)` 供大家選擇使用

若想要了解 CPU 和 Memory 的資源配置方式可以參考圖文並茂的 [Allocatable memory and CPU in Kubernetes Nodes - learnk8s][14]

### Container 跟 Pod 跟 Kubernetes 的關係

![](/images/container-pod-k8s.png)

首先是，Pod 跟 Kubernetes

> Pods are the smallest deployable units in Kubernetes

在 Kubernetes 的世界裡，最小調度單位是 `Pod`，而不是 Container，這點相當重要，所以我們所知道的資源調度及應用程式複製，全部都是以 Pod 為角度思考而不是以單ㄧ Container 為主，那有人就會問那 Container 跟 Pods 的關係又是什麼?

> A Pod is a group of one of more containers with shared storage and network

在 Kubernetes 所定義最小單位 `Pod` 當中，可以`存在 1 個或多個`的 Container 運行在其中，並且共通分享同一個 Linux Namespace 所提供的資源，只要在同一個 Pod 裡面內，Container 互相的資源皆可以被存取，然而這些 Container 才是實際上包裝應用程式後所呈現的樣態，運行的環境毫不懷疑地就是跑在作業系統上面

有些人會很好奇為什麼很多時候都會把 Pod 跟 Container 混在一起描述呢? 這是因為絕大部分的實作，一個 Pod 裡面僅會放置一個 Container 在裡面，所以才會有這種錯覺產生

### Kubernetes 資源估算方式

根據 [Managing Resources for Containers - Kubernetes][13]，你會知道其所定義的 Resource Type 主要有下列兩個：

1. CPU
2. Memory

> One cpu, in Kubernetes, is equivalent to 1 vCPU/Core for cloud providers and 1 hyperthread on bare-metal Intel processors.

前者 CPU 的換算方式，就是 `1 CPU = 1 Core = 1,000 millicores = 1,000m`，是採用`絕對數量`來進行計算，Kubernetes 要求 0.1 Core 就是需要完完整整的 0.1 Core，無論底下 CPU 是 Single Core、Dual-Core、32 Core 機器，還是透過 CPU Overcommitment 調整後，且 Kubernetes 計算資源時也無視時脈頻率 (clock rate) 的差異，下列特別列舉出所有跟 Kubernetes 1 CPU 所等價的資源清單，然而一般做 CPU 資源計算的時候都會以 vCPU 角度進行計算

| 1 Kubernetes CPU is equivalent to                                 |
|-------------------------------------------------------------------|
| 1 Hyperthread on a bare-metal Intel processor with Hyperthreading |
| 1 vSphere CPU                                                     |
| 1 IBM vCPU                                                        |
| 1 Azure vCore                                                     |
| 1 GCP Core                                                        |
| 1 AWS vCPU                                                        |
| 1 Non-HT CPU                                                      |

後者 Memory 的計算方式，可以使用二進位表示或者是十進位表示，例如下列 3 個數值是等價關係：

> 128974848 = 123 Mi (power of 2) = 129 M (power of 10)

另外它也是採用`絕對數量`來進行計算，無論你底下是實體記憶體還是虛擬記憶體，或者是有經 VMM 平台所提供的 Memory overcommitment 能力擴張後的數字，一般很少遇到這類問題是因為大多數的人的記憶體用法都是 1:1 進行使用，然而做 Memory 資源計算的時候，建議都以 vRAM 角度進行計算

| 1 Kubernetes memory is equivalent to |
|--------------------------------------|
| 1 physical RAM                       |
| 1 virtual RAM                        |

## 容器部署在虛擬環境好還是實體環境好?

若你有認真把上面的 4 個先備小知識看完之後，理應會有下列結論

1. 容器必然會運行於作業系統之上
2. Kubernetes 的運行是建立於作業系統之上，其本身還是需要消耗一定的資源運作
3. Kubernetes 對於資源的利用是無視虛擬環境、實體環境

首先先把所有常見環境架構擺在一起，包含下列 4 種：

![](/images/4-compare.png)

1. Bare Metal
2. `Container on Bare Metal`
3. VM on Bare Metal
4. `Container on VM on Bare Metal`

當中 2、Container on Bare Metal 跟 4、Container on VM on Bare Metal 是本文需要討論的架構，一般需要考量容器是否要不要放在虛擬環境和實體環境，會有以下 5 大考量點

1. 管理性
2. 資源隔離性
3. 運算效能
4. 是否具備特殊硬體
5. 整體擁有成本

### 管理性

![](/images/pets-cattle.jpeg)

如果你是虛擬機環境使用為主的維運人員，就角度上來說，Kubernetes 就是一群 VM 所組成的，所以既有的 VM 管理方式或功能還是可以直接使用，譬如說 Overcommitment、Snapshot、虛擬網路卡新修刪、Migration，但這些功能對於 Kubernetes 本身運作來講並不是必要的選項，歸屬於技術可以實現但並不是必要的類別，比較常見的討論就是在 VM Migration 能力需要與否，以 Kubernetes 來講，透過內建的 Self Healing 或者是 Node Selector 的做法，可以輕易的辦到跨節點運行相同程式的需求

倘若是以裸機環境為主的維運人員，因為裸機上面直接套用作業系統，可以相當無腦地加入全機資源轉換成 Worker Node 提供給 Kubernetes 加入當中的計算資源池內使用，且所有管理操作皆由 Kubernetes 控制節點統一管理，相對來說減少了所需管理的層數，可以透過較少的硬體來運行相同數量的容器

結論是，如果你想要沿用既有虛擬環境的管理優勢，而不考慮優先使用 Kubernetes 內建管理機制，部署容器至虛擬環境是比較好的選擇；反之，你想要完全使用 Kubernetes 內建所帶來的管理機制，部署容器至裸機環境是最佳選

### 資源隔離性

![](/images/container-vm-isolation.png)

無論是 VM 及 Container 其實皆有提供資源隔離能力給應用程式，只是技術角度不一樣。

若是以裸機環境來說，因為整台機器資源及控制權都交給了 Host OS，所以作業系統的安全性會直接影響到底層硬體的安全性與否，同時再搭配上 Container 本身自帶的資源隔離命名空間 - Linux Namespaces 及其相關的控制能力 seccomp、selinux、cgroups 等提供 `Linux Process 級別`之輕量級資源隔離

以虛擬環境 VMM Type 1 的架構為主，虛擬機具備完整的作業系統，包含記憶體管理和虛擬設備驅動程式，主要於`虛擬機級別`提供隔離能力，並為 Guest OS 提供擬真資源，可以阻止虛擬機執行可能損害主機平台完整性的指令，這種架構想當然而是對主機資源需要有較大的要求，來進行全虛擬化的保護，同時間也採用了 Container 本身所帶來的資源隔離能力，做到雙重隔離

![](/images/vm-coontainer.png)

結論是，倘若你單台設備有需要跟別的單位共用工作負載且作為不同用途的話，且並非是可以透過輕量級隔離能力可以辦到的資源隔離，那使用虛擬環境是比較好的選擇，反之，你是專門想要提供某個服務或某個部門，並未有共用其他工作覆載議題的話，裸機環境基本上是最佳選擇

### 運算效能

![](/images/vm-bm-latency.png)

> Performance: Native > Container > VM

其實這個不用特別討論也可以理解到，裸機整體效能會比虛擬環境好，畢竟裸機全部的資源都拿來使用，針對網路的部分，這篇文章 [Kubernetes on Bare Metal: When Low Network Latency is Key][17]，表明 Network Latency 差距是 3 倍左右，CPU 利用率差距大概是 10% - 15% 左右

有人會問說一般的 Kubernetes 壓力測試平時都是在什麼環境壓的? 答案是主要是在三大公有雲上的環境，因為計算資源公有雲都準備好了，主要測試也會以這些公有雲的環境為主，有趣的事情是三大雲所提供的環境雖然都是相同的量級的 CPU / Memory，但端看測試的時候會有效能上的顯著差異


### 具備特殊硬體

![](/images/pci-passthough.gif)

倘若你是需要運行 GPU / SR-IOV NIC 或者是其他需要高性能運算或對實體網卡有特殊要求的，建議一律就是選擇裸機部署路線了，雖然虛擬環境可以透過 PCI Passthough 進行穿透使用，但現行於 Kubernetes 的使用上大多還是直接裸機環境使用，除了管理方便以外，其次的原因是通常會裝載特殊硬體，所需要的計算資源也是相當大的，大部分常見於醫療、製造、電信等對計算需求相對大的領域

### 整體擁有成本 (TCO)

從 [Five Reasons You Should Run Containers on Bare Metal, not Virtual Machines][2] 借圖

![](/images/tco.png)

> 伺服器所能提供的資源上限就是完全依據硬體規格所提供的，並不會多也不會少

於虛擬環境中，需要先扣除掉 Host OS + 運行 VMM Hypervisor Type 1 所需資源 + Guest OS 這三大計算資源之後，剩下的才是可用計算資源，按照 [Best Practices for Red Hat OpenShift on the VMware SDDC - VMware][16] 所述，約略會需要先`保留 15% `的資源供虛擬環境平台使用；而於裸機環境中，則僅需扣除掉 Host OS 的運作資源即可，可以用最大幅度可用計算資源去運行服務，正常狀況之下，每台伺服器能運行的容器數量，裸機環境會比虛擬環境來的還要高，也意味著更高的利用率、更低的功耗和更低的成本、近一步降低管理費用

基於虛擬環境的容器部署上，最多的成本考慮因素是虛擬化軟體所附帶的成本費用，稱作 vTAX，通常部署上會連帶產生為數可觀的費用，進而增加總體擁有成本

結論是，倘若你想要有最高的 C/P 值的話，底層硬體完全提供給容器運行的話，裸機環境部署是最佳選項，其次才是虛擬環境為主

## 結語

最近蠻多公司都在推廣 Kubernetes on Bare Metal 的架構，來獲得的資源最大使用效率、最低 TCO、最好效能、管理單純等好處，倘若今天有考慮要建立一個基於 Kubernetes 為主的私有雲服務，那的確是一個蠻好的選擇；倘若是要跟既有的環境混用或者是因既有管理需求等因素，則可以考慮以 VM 為主的 Kubernetes 服務；當然還有一種選擇是`混用`，我個人是比較推崇這種架構，依據情境不同提供不同資源，盡可能的有效利用各架構的優勢截長補短

![](/images/vmware-vs-k8s.png)

現實生活中，大部分的人還是會詢問說，VMware 跟 Kubernetes 的架構差異是什麼，我覺得 VMware 這篇比較文章寫得不錯 [Kubernetes Introduction for VMware Users – Part 1: The Theory][12]

## References
- [Bare-metal vs. VM container deployment considerations][1]
- [Five Reasons You Should Run Containers on Bare Metal, not Virtual Machines][2]
- [Container vs VM: When and Why? - Gene][3]
- [Which is Better for Containers, Bare Metal or Virtual Machines?][4]
- [Comparison Between Bare-metal, Container and VM using Tensorflow Image Classification Benchmarks for Deep Learning Cloud Platform - Paper][5]
- [Comparison Between Bare-metal, Container and VM using Tensorflow Image Classification Benchmarks for Deep Learning Cloud Platform - Slide][6]
- [Containers are Linux][7]
- [What is the difference between a process, a container, and a VM?][8]
- [nicolaka/netshoot - GitHub][9]
- [What is Kubernetes?][10]
- [云原生时代， Kubernetes 多集群架构初探][11]
- [Kubernetes Introduction for VMware Users – Part 1: The Theory][12]
- [Managing Resources for Containers - Kubernetes][13]
- [Allocatable memory and CPU in Kubernetes Nodes - learnk8s][14]
- [HPE Reference Architecture for delivering insight across all your data with Microsoft SQL Server 2019 Big Data Clusters][15]
- [Best Practices for Red Hat OpenShift on the VMware SDDC - VMware][16]
- [Kubernetes on Bare Metal: When Low Network Latency is Key][17]
- [Dell EMC Ready Stack for Red Hat OpenShift Container Platform 4.3][18]

[1]: https://searchservervirtualization.techtarget.com/tip/Bare-metal-vs-VM-container-deployment-considerations
[2]: https://climbcsblog.com/2019/03/11/five-reasons-you-should-run-containers-on-bare-metal-not-vms/
[3]: https://igene.tw/container-vs-vm#%E4%BD%BF%E7%94%A8%E6%99%82%E6%A9%9F
[4]: https://containerjournal.com/features/better-containers-bare-metal-virtual-machines/
[5]: https://www.scitepress.org/papers/2018/66806/66806.pdf
[6]: https://lsalab.cs.nthu.edu.tw/home/publication/CLOSER18.pdf
[7]: https://www.openshift.com/blog/containers-are-linux
[8]: https://medium.com/@jessgreb01/what-is-the-difference-between-a-process-a-container-and-a-vm-f36ba0f8a8f7
[9]: https://github.com/nicolaka/netshoot
[10]: https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/
[11]: https://yq.aliyun.com/articles/713012
[12]: https://blogs.vmware.com/cloudnative/2017/10/25/kubernetes-introduction-vmware-users/
[13]: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
[14]: https://learnk8s.io/allocatable-resources
[15]: https://h20195.www2.hpe.com/V2/GetDocument.aspx?docname=a50001963enw
[16]: https://blogs.vmware.com/apps/files/2019/11/Best-Practices-for-Red-Hat-OpenShift-on-the-VMware-SDDC-Final01.pdf
[17]: https://www.ctl.io/developers/blog/post/kubernetes-bare-metal-servers
[18]: https://www.dellemc.com/resources/en-us/asset/technical-guides-support-information/solutions/h18212-openshift-container-dpg.pdf