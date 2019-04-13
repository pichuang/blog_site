layout: post
title: Container Bare Metal for Networking 參考架構
author: Phil Huang
tags:
  - intel
  - openshift
  - networking
  - cni
  - multus-cni
  - nfv
categories:
  - openshift
date: 2019-04-11 23:06:00
---
> Intel 網卡黑科技 + 容器網路的現在進行式

在 April 2019 的時候, Intel 釋放出這份 Reference Architecture [Intel Network Builders: Containers Experience Kits - Container Bare Metal for 2nd Generation Intel® Xeon® Scalable Processor Reference Architecture][1]

各位看看以 Multus Plugin 為核心的上下層整合，上面另外接了 Flannel / SR-IOV / Userspace CNI Plugin，下面接的是容器化的 CNF (Cloud-native Network Function) 應用，還順便針對 vBNG / vCMTS 做了 ovs-dpdk/VPP 的網卡加速

![](/images/intel-1.png)

<!--more-->

若想要詳細了解裡面技術內容的話可以參考此篇 [Intel Network Builders: Containers Experience Kits - Advanced Networking Features in Kubernetes* and Container Bare Metal][3]

倘若想看影片 Demo 的話，不能錯過這個 [Multiple Network Interfaces in Kubernetes Demo][4]

![](/images/intel-2.png)

[Intel Network Builders: Containers Experience Kits][2] 在這網站提供了最新的 Container Networking + Intel Technology 相關的技術文件，而目前 Red Hat 跟 Intel 正在如火如荼地合作把重中之重的 [intel/multus-cni - GitHub][5] 加緊進行驗證測試中要準備要在 OpenShift 上 GA 了 


## References
- [Intel Network Builders: Containers Experience Kits - Container Bare Metal for 2nd Generation Intel® Xeon® Scalable Processor Reference Architecture][1]
- [Intel Network Builders: Containers Experience Kits][2]
- [Intel Network Builders: Containers Experience Kits - Advanced Networking Features in Kubernetes* and Container Bare Metal][3]
- [Multiple Network Interfaces in Kubernetes Demo][4]
- [intel/multus-cni - GitHub][5]


[1]: https://builders.intel.com/docs/networkbuilders/container-bare-metal-for-2nd-generation-intel-xeon-scalable-processor.pdf
[2]: https://networkbuilders.intel.com/network-technologies/container-experience-kits
[3]: https://builders.intel.com/docs/networkbuilders/adv-network-features-in-kubernetes-app-note.pdf
[4]: https://networkbuilders.intel.com/social-hub/video/doxsehe0
[5]: https://github.com/intel/multus-cni