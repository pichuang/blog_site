layout: post
title: 邊緣計算大哉問 Part 2
author: Phil Huang
tags: []
categories: []
date: 2020-03-02 00:16:00
---
前幾天寫了篇 [邊緣計算大哉問 Part 1 ][1]，最後面有提到要提到實務面的事情，但在提到實務面的規劃前，想先跟大家介紹一下現在場上與 Edge Computing 相關的開源軟體 (Open Source Software, OSS) 大型社群，內文是我本人自己的觀點，一定有偏頗，所以各位看官請自己拿捏

![](/images/edge-6.png)

<!--more-->

## Q7: 那現在有多少常見的 Edge Computing 國際社群?

> 多到不可思議

目前我自己知道的就以下這麼多專案

- 隸屬 OpenStack Foundation 底下的 [Edge Computing Group][2]
	- [StarlingX][11]
	- [Airship][10]
- 隸屬 Linux Foundation 底下的分支 [LF Edge Foundation][3] 的專案們，當中[分為 3 階段][28] (階段越大專案越成熟)：
	- Stage 1: At Large
		- [Baetyl][8]
    	- [Fledge][9]
    - Stage 2: Growth
    	- [Edge Virtualization Engine (EVE)][5]
    	- [Home Edge][6]
        - [Open Glossary][29]
    - Stage 3: Impact
		- [EdgeX Foundry][7]
    	- [Akraino Edge Stack][4]
- 隸屬歐洲電信標準協會 (ETSI) 的 [Multi-access Edge Computing, MEC][12]
- 隸屬 Kubernetes 底下的 [IoT Edge Working Group][13]
- 隸屬電信基礎專案組織 (Telecom Infra Project, TIP) 的 [Edge Computing][14]
- 隸屬 [Linaro][16] 底下的 [Edge & Fog Computing][15]
- [Open Edge Computing Initiative][30] //這個組織我個人不是很清楚在做什麼的，歡迎補充
- 專門針對自駕車的汽車邊緣計算聯盟 [Automotive Edge Computing Consortium, AECC][31]
- [OPNFV][32] 的 Edge Cloud
- [Open Network Automation Platform, ONAP][33]


你的內心一定在罵說怎麼那麼多專案...跟我的心情是一樣的呢~

## Q8: 哪個專案比較有搞頭?

老實說，看上面的隸屬的母組織，其實都有各自的立場和方案，所以很難一言以蔽之

但因為小弟現在在 Red Hat 打工，所以會比較關注 LF Edge 底下的 `Akraino Edge Stack` 及 `Kubernetes IoT Edge Working Group`，尤其是前者，其他的專案就都只會順便看看

## Q9: 關於 Kubernetes IoT Edge WG?

> 專門以使用 Kubernetes，對 IoT 及 Edge 相關應用討論為主的工作組

![](/images/edge-3.png)

去年 KubeCon EU 2019，Kubernetes IoT Edge WG 發布了一份 [Intro and Deep Dive Combined Session][17]，這份還蠻值得一看的

{% youtube 5UgOjvK1IN8 %}

簡單來說，底下涵蓋 3 個專案，包含台灣有陣子有在討論蠻熱烈的 [`k3s`][20] 以外，[`KubeEdge`][21] 和 [`Virtual Kubelet`][22] 也在討論的範圍之內，只是兩者對應的是不同的場景，詳細可以看一下投影片 (p9)

除此之外，底下提供一下幾個傳送門

- [WG White Paper][18]
- [GitHub Communtiy][19]

另外，剛好前陣子 iThome 第 11 屆鐵人賽，台中科技大學的 IMAC 團隊有介紹了裡面所提的兩個專案，大家可以看看

- [[Day 2] KubeEdge 介紹][24]
- [[Day 14] K3s 介紹][25]

## Q10: 關於 LF Edge?

> LF Edge = The New Open "Edge" = IoT + Telecom + Cloud + Enterprise

> 筆者：不負責任推測，下次大概就要多一個 AI/ML =_=

![](/images/edge-5.png)

就是這麼剛好，2020/02 的時候 LF Edge 發布了最新的 [Overview Slide][26] 再搭配 [Wiki][27] 看，可以快速抓個框架

但下面這張圖我一定要特別講一下，以最高層組織 Linux Foundation 的角度看待這個子組織，可以了解到...

> 來自 Linux Foundation の 野望!!

![](/images/edge-4.png)


回歸正題，誠如在 Q7 中所列的專案，LF Edge 把這些專案分為[三個階段][28]

- Stage 1: At Large (formerly 'Sandbox')
- Stage 2: Growth (formerly 'Incubating')
- Stage 3: Impact (formerly 'Top-Level')

因工作關係，主要都是看 `Stage 3` 當中的 `Akraino Edge Stack`，而裡面的內容相當多...是真的非常多...後面再來介紹


## 結語

Part 3 預計要來寫 Akraino Edge Stack 相關的內容，還有 Kubernetes Native Infrastructure (KNI) 的架構

另外對邊緣人計算有興趣的朋友們，看一看研究的差不多，快來 [Cloud Native Taiwan User Group][23] 報名講者分享啊~

## References
- [邊緣計算大哉問 Part 1 - Phil Huang][1]
- [OpenStack - Edge Computing Group][2]
- [LF Edge Homepage][3]
- [LF Edge - Akranio Edge Stack][4]


[1]: https://blog.pichuang.com.tw/20200225-edge-computing-part-1/
[2]: https://wiki.openstack.org/wiki/Edge_Computing_Group
[3]: https://www.lfedge.org/
[4]: https://www.lfedge.org/projects/akraino/
[5]: https://www.lfedge.org/projects/eve/
[6]: https://www.lfedge.org/projects/homeedge/
[7]: https://www.lfedge.org/projects/edgexfoundry/
[8]: https://www.lfedge.org/projects/baetyl/
[9]: https://www.lfedge.org/projects/fledge/
[10]: https://www.airshipit.org/
[11]: https://www.starlingx.io/
[12]: https://www.etsi.org/technologies/multi-access-edge-computing
[13]: https://github.com/kubernetes/community/tree/master/wg-iot-edge
[14]: https://telecominfraproject.com/edge-computing/
[15]: https://www.linaro.org/engineering/edge-and-fog-computing/
[16]: https://www.linaro.org/
[17]: https://static.sched.com/hosted_files/kccnceu19/e2/edge-wg.pdf
[18]: https://docs.google.com/document/d/1We-pRDV9LDFo-vd9DURCPC5-Bum2FvjHUGZ1tacGmk8/edit#
[19]: https://github.com/kubernetes/community/tree/master/wg-iot-edge
[20]: https://k3s.io/
[21]: https://kubeedge.io/en/
[22]: https://virtual-kubelet.io/
[23]: https://www.facebook.com/groups/cloudnative.tw/
[24]: https://ithelp.ithome.com.tw/articles/10215792
[25]: https://ithelp.ithome.com.tw/articles/10222869
[26]: https://www.lfedge.org/wp-content/uploads/2020/02/LF-Edge-web-feb2020.pdf
[27]: https://wiki.lfedge.org/
[28]: https://wiki.lfedge.org/display/LE/Project+Stages%3A+Definitions+and+Expectations
[29]: https://www.lfedge.org/projects/openglossary/
[30]: https://www.openedgecomputing.org/
[31]: https://aecc.org/
[32]: https://wiki.opnfv.org/
[33]: https://www.onap.org/