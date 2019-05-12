layout: post
title: 'OpenShift Commons Gathering at Red Hat Summit Boston 2019 Recap '
author: Phil Huang
tags:
  - openshift
categories:
  - openshift
date: 2019-05-12 00:03:00
---
![](/images/ocp.png)

就在前陣子 Red Hat 於 Boston 舉辦一年一度的 Red Hat Summit 2019 大會，在大會的前一天還特別舉辦了 OpenShift Commons Gathering 的活動，特別分享各大企業使用 OpenShift 的使用案例及技術經驗談

<!--more-->

## What is OpenShift Commons?

> Where users, partners, customers, and contributors come together to collaborate and work together on OpenShift

[OpenShift Commons](https://commons.openshift.org/) 主要是由 Red Hat 所維護的社群組織，主要目的是歡迎各式各樣的基於 OpenShift 的開發、使用案例、系統整合等等的分享，同時也有相對於 Kubernetes SIG 的組織 - [OpenShift Special Inerest Groups (SIG)](https://commons.openshift.org/index.html#interests)，針對不同的主題有特別做分享

此外 OpenShift Commons 的分享大多都是線上分享，同時也會被錄影起來放到 [Youtube - OpenShift Commons](https://www.youtube.com/user/rhopenshift/videos) 頻道內做分享，所以想要了解任何技術的部分可以直接透過 YouTube 做自我學習，尤其是大多數的用戶因為某些原因不能上到公有雲，而落地端環境的資訊相較缺乏，這邊就是一個不錯的管道可以吸收知識

## 版主小精選

### VMWare
- [State of OpenShift on VMWare][2] 主要是 VMWare [宣布][3]之後 VMWare SDDC + Red Hat OpenShift 可以獲得到雙方的架構認可，High Level 架構如下圖，詳細內容蠻多的之後另開一篇分享

![](/images/ocp-1.png)

### Royal Bank of Canada (RBC)

如果你想了解 Spark on Kubernetes/OpenShift 的話，可以參考世界第15大的加拿大皇家銀行的分享 [Containerized Spark at RBC][4]，然後蠻推薦看一下簡報裡面的逐字搞 XD

![](/images/ocp-2.png)

### Operator Framework

去年講了一整年的 Operator Framework，而這份簡報 [State of the Operators: Framework, SDKs, Hubs and Beyond][5] 是目前公開介紹性質最完整的，這不是曇花一現而已，之後在技術層面上只會看到越來越多的 Operator Framework 應用跟開發分享

![](/images/ocp-3.png)

### UPS

世界最大的快遞貨運業 UPS 出來分享在 OpenShift Infrastructure 維運的經驗，內文蠻值得一看的 [UPS and OpenShift: A Roadmap of Cloud Native Software Upgrade Delivery][6]

![](/images/ocp-4.png)

### OpenShift, Kubernetes and Beyond 未來展望
 
 這篇 [The Road Ahead: OpenShift, Kubernetes and Beyond][7] 雖然沒什麼技術內容，但有幾張簡報蠻值得大家思考整個容器平台的非技術性問題
 
![](/images/ocp-5.png)
 
首先，`容器技術不單單只是平台而已`，還有上面一票技術需要考慮進去的，無論你今天是用 Kubernetes、OpenShift 或其他廠商的平台方案，都一定要思考平台建完之後的議題，譬如 `如何撰寫可以在容器平台上運行的 Contrainer App` 亦或者是 `如何優化容器化程式開發過程` 等一些跟平台無關的問題
 
![](/images/ocp-6.png)

這張圖是表示從 Kubernetes Community Project Release 到各家廠商的 Commercial Product Release Time 間隔時間


### 更多資訊

如果你想看其他的使用案例內容，NASA、Volkswagen、Sabre、Thyssenkrupp Elevators、Best Buy，可以參考英文全文 [OpenShift Commons Gathering at Red Hat Summit Boston 2019 Recap [Videos and Slides]][1]

 
## References
- [OpenShift Commons Gathering at Red Hat Summit Boston 2019 Recap [Videos and Slides]][1]
- [VMware and Red Hat bring Red Hat OpenShift to the VMware SDDC][3]



[1]: https://blog.openshift.com/openshift-commons-gathering-at-red-hat-summit-boston-2019-recap-with-slides/?sc_cid=701f2000000txokAAA&utm_source=bambu&utm_medium=social&utm_campaign=abm
[2]: https://blog.openshift.com/wp-content/uploads/Open-Commons-Deck-v5-FINAL.pdf
[3]: https://octo.vmware.com/vmware-red-hat-bring-red-hat-openshift-vmware-sddc/
[4]: https://blog.openshift.com/wp-content/uploads/RBC-Containerized-Spark-OpenShift-Commons-Deck-May-03-2019.pptx
[5]: https://blog.openshift.com/wp-content/uploads/Talk_-OpenShift-Commons-Summit-2019.pdf
[6]: https://blog.openshift.com/wp-content/uploads/UPS-at-OpenShift-Commons-Gathering-2019-Boston.pdf
[7]: https://blog.openshift.com/wp-content/uploads/BGracely-The-Road-Ahead-OpenShift-Commons-Gathering-Boston-2019-.pdf