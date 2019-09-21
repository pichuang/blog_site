layout: post
title: 容器平台強固化保護思路
author: Phil Huang
tags:
  - openshift
  - container
categories:
  - openshift
date: 2019-07-23 23:59:00
---
不知道各位看了這麼久的容器平台 (Container Platform) 技術，一聽到 Kubernetes 的反應，是如何在腦袋中反射出技術堆棧圖 (Technology Stack) 的呢？今天要來分享一下筆者針對於 Kubernetes 容器平台，如何看待加固 (Hardening) 及系統基礎保護的技術觀點，本篇因沒有涉及到資安弱掃，故以強固化 (Hardening) 作為出發點

![](/images/container-platform-technology-stack.png)

<!--more-->

## 一座 Kubenetes 最小技術棧?

剔除掉多出的上層整合功能部分，最小情況下任何說自己是 Kubernetes 的平台方案，`最少`皆須具備以下的技術棧：

1. Operating System (如： Ubuntu、CentOS、RHEL、Fedora CoreOS)
2. Container Runtime Interface (如：Docker、CRI-O) 
3. Container Network Interface (如：Calico、Flannel) 
4. Container Orchestration  (如：Kubernetes、Docker Swarm)

如同上述所述，Kubenetes 是 CNCF 其中一個專案，且它需要很多不同的專案一同協作才能把它運行起來，包含像是操作系統 (Operating System, OS)。相關的專案都已經羅列在下面 [CNCF Cloud Native Interactive Landscape][4] 的大圖裡

![](https://landscape.cncf.io/images/landscape.png)

若想要體驗全手工安裝過程來理解細節的話，可以參考 [Kubernetes v1.11.x HA 全手動苦工安裝教學(TL;DR) - KaiRen's Blog][7]，而從中可以透過手把手安裝可以清楚地了解到，其實 Kubernetes 不是真的只有 Kubernetes 一個專案 (Project) 而已，還有 CNI、Etcd、Container Runtime 等組件要做選擇


## OS 在這邊重要嗎?

蠻多看倌在使用 Kubernetes 的時候，都會不小心遺忘其實容器平台的相關技術是仰賴於操作系統的技術 cgroup、selinux、seccomp 等達成，所以 OS 本身的選型和組態強固化是非常重要的一環，並不是使用了容器平台就可以忽視其安全性。對於想要了解實際上容器技術的本質，可以參考 [Linux Container Intermals - How They Really Work][8]

以 RH 體系的套件管理系統為例，實務上，對於作業系統的加固，除了看到 CVE 要上修補 (Patch)、定期更新套件 `yum --secure update`，有時候也會依據各自公司政策上的不同會搭配合規性驗證 (Compliance Validation)，以確保於操作系統層級上，與其他的環境是保持一致的，也就是常講的系統強化或系統標準化 (Standard of Environment, SoE)。

而常見的稽核準則有下列 5 個標準可以參考：
1. 美國聯邦資訊處理標準 (Federal Information Processing Standard, FIPS 140-2)
2. 信用卡產業資料安全標準 (Payment Card Industray Data Security Standard, PCI DSS)
3. 美國國家工業安全計畫操作手冊 (National Industrial Security Program Operating Manual, NISPOM, or DoD. 5220.22-M)
4. 美國防禦資訊系統代理機構之安全性技術實作指南 (DISA Security Technical Implementation Guide, STIG)
5. 美國政府組態基準 (United States Government Configuration Baseline, USGCB)

或者 [OpenSCAP Security Guides](https://static.open-scap.org/) 提出的參考基準：
1. [Red Hat Corporate Profile for Certified Cloud Providers (RH CCP)][32]
2. [Standard System Security Profile for RHEL7][31]

當然如果有各自公司有內部的稽核規則，也可以透過 Ansible + OpenSCAP 一併規劃使用，以達到 [Security Automation][9] 的效果

## 那 Kubernetes 也同樣有類似的規範可依循嗎?

目前筆者知道的就以下 5 個規範：

1. 應用容器資安手冊 (NIST SP 800-190. Application Container Security Guide)
2. 互聯網安全中心 (The Center for Internet Security, CIS) Kubernetes Benchmark
3. 資訊安全國際標準 ISO/IEC 27001:2013
4. 美國聯邦資訊安全管理法 (Federal Information Security Management Act, FISMA)
5. 信用卡產業資料安全標準 (Payment Card Industray Data Security Standard 3.2, PCI DSS 3.2)

因為 Kubernetes 所帶動的技術發展太快了，如果是一直衝非常前沿的技術基本上這個章節沒有參考價值，但如果是為了企業內長久穩定使用則可以參考一下這些不同的規範來做強固化依據

(工~商~時~間~) 目前 Red Hat OpenShift 3.11 截至今日為止共提供以下 3 個驗證規範手冊:

1. [Red Hat OpenShift Container Platform Product Applicability Guide for PCI DSS 3.2][14]
2. [Red Hat OpenShift Container Platform Product Applicability Guide for FISMA Moderate][15]
3. [Red Hat OpenShift Container Platform Product Applicability Guide for ISO/IEC 27001:2013][16]

## 那 Kubernetes 裝完就可以不需升級了嗎?

[Kubernetes爆重大漏洞！不法人士可取得管理員權限，竊取機敏資料、癱瘓企業應用 - iThome][17]，Kubernetes 2018/12 發出了第一個重大漏洞 [`CVE-2018-1002105`][19]。該漏洞是可以在 Kubernetes 上直接提權殺到底層去，還不會留 Log。Common Vulnerability Scoring System (CVSS) 直接給了 [9.8][20] 分數 (滿分10分)，而解決的唯一辦法就是：

> 請升級

而今年初，[容器執行元件runC含有安全漏洞，波及Docker、containerd及CRI-O等眾多容器平台 - iThome][21]，[CVE-2019-5736][23] - RunC Escape Vulnerability，CVSS 給了 [7.7][22] 分

依據 Kubenetes 這種發展神速的步調下去 (現在已經來到 1.15 了)，可以預期往後的 CVE 也會隨著大家越來越熟悉 Kubernetes 而慢慢多起來，故不可能完全避免資安風險。結論上，不是裝完 Kubernetes 就沒事了，得要考慮後續長期維護和升級的作法。當然，如果只是自用或可以做到短時間重建 (Re-build)，就不用考慮這麼遠就是了。

## CNI Plugin 可以亂選嗎?

容器網路介面 (Container Network Interface, CNI) 在 Kubernetes 來說是一個抽象的標準介面和規範，功能需額外實踐，而該介面是掌控 Kubernetes 網路的核心組件，在技術選型上就格外重要，詳細可參閱 [常見 CNI (Container Network Interface) Plugin 介紹 - hwchiu][10] 及 [CNI 常見問題整理 - hwchiu][12] (編: Hwchiu 這人跑去美國爽，羨慕嫉妒)

而發展至今，以包含虛擬網路 (Overlay Networking)、實體網路 (Underlay Networking) 之整體網路觀點來看，可以區分以下 5 種類型 (編: 這邊的歸納名是自取，並非通用說法，雖然目前也沒有通用說法，歡迎修正)：
1. Pure CNI Plugin: 可獨立運作，不需要深度依靠實體網路能力，絕大部分 CNI Plugins 皆屬此類，如 Calico、Flannel、OVN 等
2. Hypervisor CNI Plugin: 基於 Hypervisor 提供的虛擬化技術，不需要深度依靠實體網路能力，提供整合能力，如 VMware NSX-T Container Plugin (NCP)
3. Hybrid CNI Plugin: 可獨立運作或可與實體網路進行虛實整合，如 [Juniper Contrail][30]、[ONOS SONA-CNI][33]
4. Mixed CNI Plugin: 需與實體網路進行整合，如 Cisco ACI、BigSwitch Big Cloud Fabric (BCF)
5. Proxy CNI Plugin: 可透過該網路介面再接其他的 CNI Plugin，以達到單 Pod 多網卡的需求，如 [Intel Multus](https://github.com/intel/multus-cni)、[Huawei Genie](https://github.com/Huawei-PaaS/CNI-Genie)、[Nokia DAMN](https://github.com/nokia/danm)

說 CNI 是整個 Kubernetes 的核心靈魂也不為過，為了 Kubernetes 平台系統穩定，通常建議是依下列項目依序考慮：

1. 功能
2. 整合穩定度
3. 有否支援

實務上，常見選擇帶有 `NetworkPolicy` 能力的 CNI Plugin，因可基於策略 (Policy) 來隔離 Pod/Namespace 等級的內部流量或者是外部來減少攻擊面

## 我的容器映像檔乾淨嗎?

![](/images/docker-dig.png)

[Docker移除17個暗藏挖礦程式的惡意容器 - iThome][25] 這新聞應該是最實際的例子，塞了挖礦程式的容器映像檔存在公開網路一年，下載次數超過 500 萬，還真的有挖到幣 XD。另外還有 [報告：前十大熱門Docker映像檔都有至少30個以上的漏洞 - iThome][29] 等漏洞存在在映像檔內等情事發生。

為了避免類似情況發生，會有以下 3 種建議作法：
1. 只用官方提供的映像檔: DockerHub library/\*、Red Hat Container Catalog (RHCC) 等
2. 執行前要弱掃: Clair、KubeXray、Snyk 等
3. 容器標準化 (Standardize)

除了筆者前陣子有分享過 [設計出 Production-Ready Container 的 5 條戒律 - Phil Workspace][26]，Red Hat 官方也針對 Container Security 提出了一套概念 [Ten layers of container security - Red Hat][27]。而當中針對容器標準化的基礎映像檔也提出了`通用基礎映像檔` [Universal Base Image, UBI][28] 的設計思考。

> 好的開始是成功的一半，好的映像檔能讓你睡眠品質提升

真心不知道怎麼開始的話，建議就先從 `第一戒律：Standardize` 開始慢慢做起吧

## 結語

- 我最一開始只是想強調的是 OS 真的是很重要，請不要忽視他 XD
- 呈上，整體系統整合穩定度也是一樣很重要，還記得木桶理論嗎?
- Multus 穩定釋出後，對於所有規劃容器平台網路的人都會是一個 __ 的開始，Container Network Function (CNF) 笑而不語中
- 感謝 iThome 讓我充實了很多版面


## References
- [【關鍵資安議題：容器安全】剖析容器的資安風險與防護 - iThome][1]
- [10 Docker Image Security Best Practices - synk][2]
- [Technical Details - TEN LAYERS OF CONTAINER SECURITY Red Hat][3]
- [CNCF Cloud Native Interactive Landscape][4]
- [33 Kubernetes security tools - Sysdig][5]
- [聯邦資訊處理標準 - Wiki][6]
- [Kubernetes v1.11.x HA 全手動苦工安裝教學(TL;DR) - KaiRen's Blog][7]
- [Linux Container Intermals - How They Really Work][8]
- [Security Automation with Ansible][9]
- [常見 CNI (Container Network Interface) Plugin 介紹 - hwchiu][10]
- [How to deal second interface service discovery and load balancer in kubernetes][11]
- [CNI 常見問題整理 - hwchiu][12]
- [FISMA Compliance for the OpenShift Container Platform][13]
- [Red Hat OpenShift Container Platform Product Applicability Guide for PCI DSS 3.2][14]
- [Red Hat OpenShift Container Platform Product Applicability Guide for FISMA Moderate][15]
- [Red Hat OpenShift Container Platform Product Applicability Guide for ISO/IEC 27001:2013][16]
- [Kubernetes爆重大漏洞！不法人士可取得管理員權限，竊取機敏資料、癱瘓企業應用 - iThome][17]
- [The Kubernetes privilege escalation flaw: Innovation still needs IT security expertise][18]
- [CVE-2018-1002105: proxy request handling in kube-apiserver can leave vulnerable TCP connections #71411 - kubernetes/kubernetes][19]
- [CVSS:3.0/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H][20]
- [容器執行元件runC含有安全漏洞，波及Docker、containerd及CRI-O等眾多容器平台 - iThome][21]
- [Runc and CVE-2019-5736 - Kubernetes][23]
- [CVE Details - Kubernetes Security Vulnerabilities][24]
- [Docker移除17個暗藏挖礦程式的惡意容器 - iThome][25]
- [設計出 Production-Ready Container 的 5 條戒律 - Phil Huang][26]
- [Ten layers of container security - Red Hat][27]
- [Introducing the Red Hat Universal Base Image - Red Hat][28]
- [報告：前十大熱門Docker映像檔都有至少30個以上的漏洞 - iThome][29]
- [導讀 OpenShift Commons Telco SIG: Juniper Contrail: Kubernetes Multi-Interface Pods For Telco Use Cases][30]


[1]: https://www.ithome.com.tw/article/129426
[2]: https://snyk.io/blog/10-docker-image-security-best-practices/
[3]: https://www.redhat.com/cms/managed-files/cl-container-security-openshift-cloud-devops-tech-detail-f7530kc-201705-en.pdf
[4]: https://landscape.cncf.io/images/landscape.png
[5]: https://sysdig.com/blog/33-kubernetes-security-tools
[6]: https://zh.wikipedia.org/wiki/%E8%81%AF%E9%82%A6%E8%B3%87%E8%A8%8A%E8%99%95%E7%90%86%E6%A8%99%E6%BA%96
[7]: https://k2r2bai.com/2018/07/17/kubernetes/deploy/manual-install/
[8]: http://crunchtools.com/files/2019/05/Linux-Container-Internals-2.0.pdf
[9]: https://www.ansible.com/hubfs/2018_Content/AnsibleAutomates-AnsibleForSecurityAutomation.pdf?hsLang=en-us
[10]: https://www.hwchiu.com/cni-compare.html
[11]: https://www.slideshare.net/MengZeLi4/how-to-deal-second-interface-service-discovery-and-load-balancer-in-kubernetes
[12]: https://www.hwchiu.com/cni-questions.html
[13]: https://blog.openshift.com/wp-content/uploads/openshift_commons_.GOV-SIG_briefing-20170104-2.pptx.pdf
[14]: https://www.redhat.com/cms/managed-files/cl-red-hat-openshift-product-applicability-guide-datasheet-f8620-201708-en.pdf
[15]: https://www.redhat.com/cms/managed-files/cl-openshift-applicability-guide-fisma-moderate-analyst-paper-f15232bf-201811-en.pdf
[16]: https://www.redhat.com/cms/managed-files/cl-openshift-applicability-guide-iso-iec-27001-analyst-paper-f15231bf-201811-en.pdf
[17]: https://www.ithome.com.tw/news/127431
[18]: https://www.redhat.com/en/blog/kubernetes-privilege-escalation-flaw-innovation-still-needs-it-security-expertise
[19]: https://github.com/kubernetes/kubernetes/issues/71411
[20]: https://www.first.org/cvss/calculator/3.0#CVSS:3.0/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H
[21]: https://www.ithome.com.tw/news/128718
[22]: https://www.first.org/cvss/calculator/3.0#CVSS:3.0/AV:L/AC:H/PR:N/UI:R/S:C/C:H/I:H/A:H
[23]: https://kubernetes.io/blog/2019/02/11/runc-and-cve-2019-5736/
[24]: https://www.cvedetails.com/vulnerability-list/vendor_id-15867/product_id-34016/Kubernetes-Kubernetes.html
[25]: https://www.ithome.com.tw/news/123887
[26]: https://blog.pichuang.com.tw/20190621-building-production-ready-containers/
[27]: https://www.redhat.com/en/resources/container-security-openshift-cloud-devops-whitepaper
[28]: https://www.redhat.com/en/blog/introducing-red-hat-universal-base-image
[29]: https://www.ithome.com.tw/news/129018
[30]: https://blog.pichuang.com.tw/20190405-guide-openshift-commons-telco-sig-juniper-contrail-kubernetes-multi-interface-pods-for-telco-use-cases/
[31]: https://static.open-scap.org/ssg-guides/ssg-centos7-guide-standard.html
[32]: https://static.open-scap.org/ssg-guides/ssg-rhel7-guide-rht-ccp.html
[33]: https://github.com/sonaproject/sona-cni