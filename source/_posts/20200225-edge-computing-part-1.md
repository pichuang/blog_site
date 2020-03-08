layout: post
title: 邊緣計算大哉問 Part 1
author: Phil Huang
date: 2020-02-25 21:33:18
tags:
---
邊緣計算 (Edge Computing)，相信大家看了一陣子的相關文章、場景和解決方案，但有沒有覺得...好像每個人講的邊緣計算都有點不一樣，本文就是要針對邊緣計算的幾個常見問題來進行介紹，但我的觀點是`以自主自建的角度`作為討論，不是以公有雲角度為觀點的邊緣計算解釋

![](/images/edge-2.png)

> Edge Computing = Cloud resources and services close to data source & service consumers

<!--more-->

### Q1: 什麼原因驅動了邊緣計算的需求?

4 個原因

1. 對於延遲性 (Latency) 有要求： 使用者到服務需要的 RTT 約等於 1ms 或越低越好
2. 頻寬 (Bandwidth) 有要求和成本限制
3. 具備彈性架構：各自的站點需具備自治性，可以依據不同業務需求橫向擴充 (Scale-out)，但最小是 1 個站點，
4. 資料落地和法規限制：需要將敏感資料放在特定區域裡面，不讓其資料流通至其他地方，你應該不會想要把機台的參數不小心傳出去吧 


### Q2: 疑? 這個跟常聽到雲端計算 (Cloud Computing) 有什麼不一樣?

> Edge Computing != (Typically) Cloud Computing

常見的雲端計算的 3 個特點

1. 看起來是無限容量的資源池 (Resource Pool)：多數地雲端供應商的資源預備量是 `供應量 >> 需求量`
2. 多租戶管理 (Multi-Tenant)
3. 按需自助服務 (On-demand self-service)

而邊緣計算的特點`通常`是以下 3 個特點
1. 容量有限：資源預備通常是 `供應量 ≌ 需求量`
2. 大多數單租戶居多：大多數服務都是建立在 Remote Branch 或者是 Remote Office 居多，維運者大多都是在總部或者是資訊中心為主
3. 因業務需求，各自所採用的技術選型，會有不一樣的狀況：這也是為什麼各位看很多人討論的邊緣計算，似乎好像都不太一樣的原因

承如上面的討論，當然如果你用雲端計算去實踐出邊緣計算也是可以 (反正供應量 >> 需求量，不怕)，只要回歸討論 Q1 所提的 4 個點，如果透過雲端供應商的服務也可以達到相關的 SLA 也是可以一種選擇之一。

### Q3: 到底邊緣計算有多少種層級可以被討論的?

依據`地理位置`和`設備的使用目的`來做區隔，我覺得紅帽這個畫得蠻好的，借貼在板上

![](/images/edge-1.png)


有空也可以看看 [Red Hat - edge-computing/approach](https://www.redhat.com/en/topics/edge-computing/approach) 官方的介紹


### Q4: 目前邊緣計算實際上有什麼使用案例?

不考慮任何產品的方案，分為 10 種邊緣使用案例

1. Enterprise Edge： Enterprise 採用 Local Compute / Storage 及透過有線網路，常見於辦公大樓
2. Enterprise Edge (mobile / remote)：Enterprise 採用 Local Compute / Storage 及透過無線網路 (衛星、基地台進行傳輸)，常見於交通設施、煉油平台
3. Enterprise Edge (offline)：Enterprise 採用 Local Compute / Storage 及透過實體儲存設備運送資料，例如 [AWS Snowball Edge][3]
4. IoT Gateway / Device Edge：Enterprise 採用 IoT 設備，用於收集儀器或感測器資料，傳至 Enterprise Edge 之後處理，大部分的具有 [IoT Gateway][4] 解決方案，譬如 [Intel OpenNESS][13] 皆屬此類
5. Provider Network Access Edge：Telco SP 採用純硬體架構式之 Compute / Storage / Network，一般常見於 Central Office 端
6. Provider Network Access Edge (+OpenStack)：Telco SP 除了採用前者所述以外，需要多加一層 OpenStack，以提供 SDN/NFV 的彈性效益，屬於經典的 NFV / MEC 所提的範圍
7. Provider Network Access Edge (+BMaaS)：Telco SP 除了採用前者 `5. Provider Network Access Edge` 所述以外，還外加實體主機託管能力
8. [Provider Far Edge][2]： Telco SP 採用單體較為強的 [Edge Server][5]，部署在較為遠的室外站點或基站裡，最多三個，常見於 vRAN 使用場景
9. [Provider "Ultra Far" Edge][2]：Telco SP 採用 IoT 設備，如 [Nvidia Jetson TX2][6] 部署在基站，進行邊緣計算
10. uCPE (Customer-Premises Equipment)：Telco SP 採用自訂的設備放置在客戶端提供自家電信服務，例如 [AT&T OCP Desgin uCPE][7]

### Q5: 哇...眼花撩亂...有沒有快速了解的方式?

你看上面 10 種的案例，可以理解到客戶角色 (Enterprise、Telco SP) 的不同，對於邊緣計算的理解也會不一樣

針對企業最常見的就是 `1. Enterprise Edge` 跟工廠或製造業討論很久的 `4. IoT Gateway / Device Edge` 的邊緣計算，但比較多討論的還是在跨國或中大型工廠、製造業那邊

針對電信就是國外做一堆的 `6. Provider Network Access Edge (+OpenStack)` 和正在佈局中的 `Far Edge / Ultra Edge` 的情境，簡單來說 Telco 畢竟不能年年都被 CSP 吃豆腐，現在努力擴展 `last mile` 的距離中，獲取更大的控制範圍，提供更深化的服務綁住客戶，譬如說中國移動、Verizon、AT&T 都屬此類

### Q6: 那為什麼有時候會看到 AI/ML 跟邊緣計算一起出現?

這個都是因為 `4. IoT Gateway / Device Edge` 產生的延伸需求，因為工作的關係，目前我看過最經典的案例應該是美國石油巨頭埃克森美孚 (Exxonmobil)

示意圖大概如下

![](https://www.crosser.io/media/1568/crosser-industrial-iot-infographique-ml-at-the-edge.svg)

- [Slide: OpenShift and Machine Learning at ExxonMobil][9]
- [Youtube: OpenShift and Machine Learning at ExxonMobil with Cory Latschkowski][10]

雖然 Exxonmobil 是石化工業，但實際上來說他們所使用到的儀器設備也不亞於製造業的數量，他們透過前面 IoT 設備收集了一票資料後，透過建好的 AI/ML 平台去算出更好的數值去對儀器做參數優化，參數調整頻率可以更快而且還更好，去節省掉不必要浪費的成本

## 後話

邊緣計算的實務面的技術選型，也是有蠻多種樣貌的，下篇來寫目前實務面上，比較常見的架構

## References
- [A CLOUD NATIVE BLUEPRINT FOR ULTRA/FAR EDGE - ONS][2]
- [Red Hat - What is edge computing?][1]
- [AWS Snowball Edge][3]
- [Slide: OpenShift and Machine Learning at ExxonMobil][9]
- [Youtube: OpenShift and Machine Learning at ExxonMobil with Cory Latschkowski][10]
- [Open Network Edge Services Software (OpenNESS)][13]


[1]: https://www.redhat.com/en/topics/edge-computing/what-is-edge-computing
[2]: https://events19.linuxfoundation.org/wp-content/uploads/2018/07/K8S-blueprint-for-Far-Edge-v3.pdf
[3]: https://aws.amazon.com/tw/snowball-edge/?aws-snowball-edge.sort-by=item.additionalFields.postDateTime&aws-snowball-edge.sort-order=desc
[4]: https://h20195.www2.hpe.com/v2/GetPDF.aspx/c04884747.pdf
[5]: https://onestore.nokia.com/asset/205107
[6]: https://developer.nvidia.com/embedded/develop/hardware
[7]: http://files.opencompute.org/oc/public.php?service=files&t=adca4fc92bc6b981d6ef63da18dc997b&download
[8]: https://www.intel.com.tw/content/www/tw/zh/wireless-network/5g-technology/cloud-computing-and-5g.html
[9]: https://blog.openshift.com/wp-content/uploads/OpenShift-Commons-SF-Agile-Data-Science-ExxonMobil.pdf
[10]: https://www.youtube.com/watch?v=VEAUaiMKsuc
[11]: https://searchconvergedinfrastructure.techtarget.com/infographic/Illustrated-guide-to-hyper-converged-edge-computing
[12]: https://speakerdeck.com/pichuang/20190817-container-bare-metal-for-networking?slide=15
[13]: https://builders.intel.com/university/networkbuilders/coursescategory/open-network-edge-services-software-openness
