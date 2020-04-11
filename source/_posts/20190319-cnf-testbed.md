layout: post
title: Cloud Native Network Functions (CNF) Testbed 心得感想
tags:
  - cnf
  - container
categories:
  - openshift
date: 2019-03-07 22:04:00
udpated: 2019-03-07 22:04:12
---

2019/2/24 CNCF [宣布發展 CNF Testbed][1]，計畫主要針對在 ONAP (Open Network Automation Platform) 上的 VNF (Virtual Network Function) 及發展中的 CNF (Cloud native Network Function) 進行雙方的功能測試，相互比較優劣勢及性能彈性

<!--more-->

Dataplane 則統一都是用 FD.io VPP (Vector Packet Processing) 技術來做到，至於為什麼可以參考一下 [Why This Was a Challenging Project: OpenStack][6] 及 [Why This Was a Challenging Project: Kubernetes][7] 兩張簡報的底下說明


## 架構演進
下圖表示了整個從 ONAP Amsterdam -> ONAP Casablanca -> Future 的架構演進
![](/images/cnf-1.png)

## CNF Testbed High Overview
目前是混搭 OpenStack + Kubernetes 的平台，若想要更深入了解平台的技術細節可以參考 [Comparing chained network function CNF deployment models][5]

![](/images/cnf-2.png)

## Test Cases
基本的 Snake Test 是一定會有的，只是平台的部分有拆分成 OpenStack 跟 Kubernetes，但之後應該會隨著生態系的完善，項目會越來越多，後續要追蹤的話蠻建議看 [GitHub - cncf/cnf-testbed][4]

![](/images/cnf-3.png)

## vCPE Use Case

詳細架構細節 [Use Case: Residential Broadband vCPE (Approved)][8]

![](/images/cnf-4.png)

## References

- [CNCF Launches Cloud Native Network Functions (CNF) Testbed][1]
- [CNF Testbed - Google Slide][2]
- [Intro: Cloud Native Network Functions (CNF) BoF - North America 2018][3]
- [GitHub - cncf/cnf-testbed][4]

[1]: https://www.cncf.io/announcement/2019/02/25/cncf-launches-cloud-native-network-functions-cnf-testbed/
[2]: https://docs.google.com/presentation/d/1nsPINvxQwZZR_7E4mAzr-50eFCBhbCHsmik6DI_yFA0/edit#slide=id.g5036f143e9_3_113
[3]: https://schd.ws/hosted_files/kccna18/c1/KubeCon%20NA%202018%20Intro_%20Cloud%20Native%20Network%20Functions%20BoF%2012-12-2018%20FINAL.pdf
[4]: https://github.com/cncf/cnf-testbed
[5]: https://github.com/cncf/cnf-testbed/tree/master/comparison/kubecon18-chained_nf_test
[6]: https://docs.google.com/presentation/d/1nsPINvxQwZZR_7E4mAzr-50eFCBhbCHsmik6DI_yFA0/edit#slide=id.g4fe85c61a7_48_102
[7]: https://docs.google.com/presentation/d/1nsPINvxQwZZR_7E4mAzr-50eFCBhbCHsmik6DI_yFA0/edit#slide=id.g4fe85c61a7_48_108
[8]: https://wiki.onap.org/pages/viewpage.action?pageId=3246168
