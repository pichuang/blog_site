layout: post
title: '過了三年後的 NetDevOps '
author: Phil Huang
tags: []
categories: []
date: 2020-02-29 22:28:00
---
三年前(2017)，那時工作在 [Edgecore Network 上班][1]，經常接觸到各式各樣的網路設備，那時候看到一家新創公司 [Cumulus Networks][3] 開始推所謂的 NetDevOps 的概念，也就是利用組態管理系統 (Configuration Management) 去管理底下資料中心的 Switch / Router 確保不會有設定飄移 (Configuration Drift)，時間轉換到2020，我以個人角度來跟大家分享一下經歷
![](/images/netdevops-overview.png)

<!--more-->

## 2017
### Switch as a Server ?

> OpenNetworking = Network OS + Open (Spec) Hardware

要講 NetDevOps 首先一定要講一下 Cumulus Networks 這家公司

![](https://static.cumulusnetworks.com/static/images/shared/cumulus-og@2x.1e8aac69c04e.png)

Cumulus Networks 是一家做 Network OS 的新創公司，他們的產品名稱就是叫做 `Cumulus Linux`，而他們真的只有負責做軟體的部分，底下的硬體就是使用開放規格的 `Open Networking` 為核心的硬體交換機，白話來說就是，底下硬體規格只要是一樣的，理想上無論哪家 ODM/OEM 硬體都是可以安裝 Cumulus Linux 在上面跑，就像是你裝個 Windows 作業系統，理論上你也不用太擔心到底是裝在誰家的硬體，反正都是 x86_64 的架構，都可以裝。

這有趣的地方就是在這個 `Cumulus Linux` 身上，它本質上就是用 Debian 作業系統下去改的，連網路的操作都是用你我常見的 Linux command，包含像是升級 (apt update/upgrade)、修改檔案 (vim)、網路操作 (ip)，這操作體驗壓根兒就是個... Linux OS，只是它跑在網路交換機上。你可以把它想成 `一台灌著 Debian Linux 的伺服器，插著總共 54 port 的網卡`，所以那時候我蠻常說，只要你會用 Linux 相關的指令，那這這個 Cumulus Linux 一定會非常好上手。

正因為 Cumulus Linux 是這種特性，所以管理這些 Switch 的方式，其實就跟管 Linux Server 沒什麼特別的差異，所以 `Switch as a Server` 用管理 Server 的方式來管理 Switch 是這樣的緣故。

想瞭解更多 SDN / OpenNetworking 的基礎概念的話，可以參考一下 [建豪哥先前的分享 SDN 101][4]，老實說比我當年講的還要好 XD


### Why NetDevOps?

> NetDevOps = Networking + DevOps

畢竟這是網路領域，日常維運 (Day 2 Operation) 還是跟 Server 是有所不一樣，常見一個 DC Leaf-Spine Network Fabric 最小就是 2 Spine + 4 Leaf 起跳，就是 6 台的量。大家搞 IT 的都知道，動任何一台設備的網路設定，有可能會`改一設定毀全網`

所以當時 Cumulus Networks 提出了一個概念叫做 NetDevOps，講白了就是把現行 DevOps 的概念，套用在 Networking 領域之上實踐

1. 自動化維運（IT Automation)：透過軟體操作來完成大量維運的工作
2. 基礎架構即代碼 (Infastracture as Code, IaC)：原始碼可重複 (Reusable)、可共享 (Shareable)、可驗證 (Validable)
3. 易於持續交付/整合 (Continous Deployment / Continous Integration)

具體實踐的幾種方式可以參考一下以前寫的文章 [NetDevOps 風格之網路設備連接方式 - Phil Huang][8]

而 Cumulus Networks 挑的基礎技術，其中之一就是 `Ansible`

關於 NetDevOps 的部分，有兩本書還蠻推薦看的
- [NetDevOps 入門與實踐][6]
- [使用 Python、Powershell、Ansible 實踐網路自動化 (中文版)][7]


### Why Ansible ?

![](/images/ansible-logo.png)

瞭解到了 NetDevOps 的演進，對於 Network Engineer 來說，基本上很多是不寫程式的，如果有現有的模組 (Module) 可以套一套就可以用，可以做到上面篇幅所提的好處，會有相當的吸引力

Ansible 是 2013 冒出來的，Red Hat 於 2015 年收購，至於理由我覺得這篇寫得不錯 [解读：Red Hat 为什么收购 Ansible][5]

Ansible 於當時為人所知，做得較多的還是在 Linux Server 端的管理，後面慢慢的才開始有支援 Windows、Networks、Security 等領域

Cumulus Linux 剛好本身就是泛 Linux 操作，而當時 Ansible 網路廠商不是非常多家支援的狀況下，挟著`我就是一個 Linux 只是生在網路交換機上`的角度，搭配 Ansible 剛好切入了 NetDevOps 的議題，所以才於 2017 年在 DevOpsDay Taipei 上分享了 [NetDevOps: Next-Generation Network Engineer - Phil Huang][2]


## 2020

### 更多網路廠商願意支持 Ansible

通常來說，Ansible 內建三大優勢：

1. Simple
2. Powerful
3. **Agentless**

對於各大網路廠商來說，最大的特點是 `Agentless`，使用 Ansible 不需要特別先塞程式在設備上面，各廠商可以各自撰寫自己的 SDK 進行維護，Ansible 及各網路廠商的架構搭配，可以參考 [NetDevOps 101 - Phil Huang][9]

台灣較為知名的 Cisco、Juniper，也都相繼針對自動化推出了各自的證照，可以參考一下這新聞 [思科認證近年來最大變革！新增軟體開發系列證照，將軟體技能與實務經驗帶至網路管理領域 - iThome][11]

- Cisco DevNet
![](/images/netdevops-ansible.png)

- [Juniper JNCIA-DevOps - Automation and DevOps Certification Track][13]
![](https://www.junipercertified.com/wp-content/uploads/2019/01/JNCIA-DevOps.png)


當中 Cisco 蠻有趣的，Red Hat 跟 Cisco 合作的文章 [Automate Cisco Environments with Red Hat Ansible Automation][21] 範圍，其實不單單是 Switch / Router，連 Server / Wireless / Firewall / SDN 都有，包山包海

![](/images/netdevops-cisco.png)

### NetDevOps 實踐上更普及

[Network to code][17] 是一家利用網絡自動化以及DevOps技術和方法，幫助公司轉變其網絡的部署，管理和使用方式，且不受限於任何供應商技術或工具的一家公司

這公司於 2016 就有發佈 NetDevOps 的調查 [2016 NetDevOps Survey][18]

經過了三年後，發布了 `2019 NetDevOps Survey: State of network operations through automation` - [Slide][14], [YouTube][15], [Survey][16]


這邊 Highlight 一下幾個我覺得有趣的內容，

![](https://dgarros.github.io/netdevops-survey/graphs/png/netdevops_survey_operation-automated_compare.png)

三年後，使用場景不僅僅只是做最簡單的設定檔定期備份或收收資料，其他的場景也變高了

![](https://dgarros.github.io/netdevops-survey/graphs/png/netdevops_survey_config-gen-deploy_compare.png)

Ansible 持續海放中

![](https://dgarros.github.io/netdevops-survey/graphs/png/netdevops_survey_2019_trend-tools_stack.png)

使用 Ansible 務必要搭個版本控制使用，畢竟基礎建設即代碼 ( Infrastrcuture as Code, IaC) 的靈魂是版本控制

第一次接觸版控的人，可以直接從 Git 開始入手了

![](https://dgarros.github.io/netdevops-survey/graphs/png/netdevops_survey_2019_env-nbr-devices_bar.png)

無論大小環境都很適用實踐自動化

### 網路測試覆蓋率更廣

Ansible 使用上不需要預先塞代理軟體的，很容易可以跨平台使用，可以執行一個劇本，搭配 CI/CD 跨多個平台管理使用

{% youtube 22vaD3QH1rg %}

所以對於網路測試的理解，不單單只是單一設備的測試，而是端到端的業務邏輯測試

{% youtube LinGy8DGIJ8%}

就連於 GitHub 的年度開源報告 [`The State of the octoverse`][23] 當中的 `Top and trending projects`

> Open source projects are growing on GitHub, from one-line programs to projects with nearly 20,000 contributors. The open source repositories created this year make up 30% of all open source projects on GitHub.﻿

`Ansible` 跟 `Microsoft VSCode` 是為二並列從 2016 年一路都在前十排行榜的開源專案，就可以反映出 Ansible 受歡迎的程度

![](/images/github-trending.png)

## 結語

![](/images/netdevops-baby.jpeg)

若你是一名網路工程師，用最小學習成本，學習一個符合 DevOps 理念的工具，那 Ansible 是你一個投資報酬率會相當高的選擇

透過 Ansible，可以更實際且迅速地跟其他部門合作管理好自家公司 (或客戶家) 的環境

什麼? 你問我說如何開始? 先從加入 [Ansible 台灣使用者社群][22] 開始吧~

## References
- [解读：Red Hat 为什么收购 Ansible][5]
- [NetDevOps: Next-Generation Network Engineer - Phil Huang][2]
- [NetDevOps 風格之網路設備連接方式 - Phil Huang][8]
- [NetDevOps 101 - Phil Huang][9]
- [Ansible for Network Automation][10]
- [思科認證近年來最大變革！新增軟體開發系列證照，將軟體技能與實務經驗帶至網路管理領域 - iThome][11]
- [DevNet Associate Exam v1.0 (200-901)][12]
- [What is DevOps? - Juniper][13]
- [Building a NetDevOps CI/CD Pipeline - Hank Preston (DevNet Create 2018)][14]
- [Ansible 台灣使用者社群][22]

[1]: https://blog.pichuang.com.tw/about/

[2]: https://speakerdeck.com/pichuang/netdevops-next-generation-network-engineer
[3]: https://cumulusnetworks.com/
[4]: https://drive.google.com/file/d/1aYUoVzlJi-LgNnFFfWbpMzFtmQpyf3B1/view
[5]: https://www.infoq.cn/article/2015/10/Red-Hat-DevOps
[6]: https://www.tenlong.com.tw/products/9787111599098?list_name=srh
[7]: https://www.tenlong.com.tw/products/9789865020484?list_name=srh
[8]: https://blog.pichuang.com.tw/20180825-netdevops/
[9]: https://speakerdeck.com/pichuang/netdevops-101?slide=7
[10]: https://docs.ansible.com/ansible/latest/network/index.html
[11]: https://ithome.com.tw/news/131227
[12]: https://developer.cisco.com/certification/exam-topic-associate/
[13]: https://www.juniper.net/us/en/training/certification/certification-tracks/devops?tab=jnciadevops
[14]: https://storage.googleapis.com/site-media-prod/meetings/NANOG78/2108/20200212_Garros_Netdevops_Survey__v1.pdf
[15]: https://www.youtube.com/watch?v=1DhDGEvLQ8Q&feature=youtu.be
[16]: https://dgarros.github.io/netdevops-survey/reports/2019
[17]: https://www.networktocode.com/
[18]: https://dgarros.github.io/netdevops-survey/reports/2016
[19]: https://www.youtube.com/watch?v=22vaD3QH1rg
[20]: https://www.youtube.com/watch?v=LinGy8DGIJ8
[21]: https://www.ansible.com/hubfs/pdf/ansible-automate-cisco-environments.pdf?hsLang=en-us
[22]: https://ansible.tw/#!index.md
[23]: https://octoverse.github.com/