layout: post
title: 設計出 Production-Ready Container 的 5 條戒律
author: Phil Huang
tags:
  - container
categories:
  - container
date: 2019-06-20 17:37:00
---
![](/images/rhel8-logo.jpg)

Red Hat 官方前陣子推出 Red Hat Enterprise Linux 8 (RHEL8)，裡面包含一個很大的變化就是針對 Linux 容器技術有非常大的改進和變化。應對 Docker 商業策略上的變化，由 Google、Red Hat、SUSE、IBM、Intel 攜手一起開發了一個`專門對 Kubernetes 進行優化`的容器運行環境 (Container Runtime) 的專案 [CRI-O][3] (Container Runtime Interface Orchestrator)。故 RHEL 8 內建的容器技術將以 CRI-O 為核心，連帶對應的容器工具 Podman、Buildah、Skopeo 一起往未來發展。而今天要分享的是如何設計出生產環境等級的容器映像檔 (Container Images)，基於 Red Hat Summit 2019 分享出的簡報 [Building Production-Ready Containers - Scott McCarty & Ben Breard][4] 撰寫，一位是專職的 Linux Container Product Manager，另一位是 CoreOS Product Manager

<!--more-->

## 容器在生產環境中真正的挑戰是什麼?

> Container is NOT magic, it is technical skill

![](/images/container-challenge.png)

True 的欄位就不用看了，替大家翻譯右邊常見誤解

- 每個人都可以做他們想做的事情，而開發人員將可以自己做所有的事情，不需要有專家的存在...?
- 完全可移植性：建構一次，隨處運行...?
- 容器是非常簡單的，開發人員只要使用它們，不用擔心太多...?
- 你必須要完全解構掉你既有的應用程式...?
- 忘掉你所學到的一切，這都是魔法...?

若想深度了解 Linux Container 內部技術的話，蠻推薦閱讀這份投影片 [Linux Container Intermals - how they really work][1]

## 你我的容器映像檔?

> 金玉其外，或許...敗絮其內?

很多所被建構出來的容器映像檔 (Container Images)，實際上使用可以用，但內部卻是亂糟糟的，譬如：

- 混亂不明的依賴性
- 不知道為什麼會在那邊的 Process (ex: sshd、telenetd)
- 發現 CVE 的時候不知道要怎麼修怎麼更新

以上三個狀況是平時維運或使用容器技術時最常見的狀況：升級障礙、誤設計、資安修補

## 情境思考

其實，設計容器映像檔分層 (Layers) 的技術選型，就像你買一台伺服器回家，你還是得要思考上面要裝什麼 OS、要切多少磁區、要裝什麼軟體等等，裝完之後還是要持續維護、更新等操作維護面的事情，一點都沒有例外，故需要的是`如何選擇一個穩定且安全的基礎映像檔當作你的 Base Image` (其實就像是選作業系統一樣) 和`優良的容器化設計` (這跟軟體系統設計也很類似)。

有些人會說：把程式包成容器映像檔後，不就變成不可變動 (Immutable) 的狀態，有必要做維護嗎? 有這些疑問的可以參考一下這篇新聞 [報告：前十大熱門Docker映像檔都有至少30個以上的漏洞 - iThome][2]，當你發現你的映像檔有潛在資安風險，能不更新繼續上線對外使用嗎？答案應該是蠻明顯的，故需要的是`一個如何順暢進行更新容器映像檔的作法`。

## 一個容器映像檔的設計難度? 理想與現實的距離

![](/images/container-level.png)

一般容器設計考慮大概要盡量維持在 Easy 和 Moderate 這當中的距離，如果要做到 Difficult 等級的設計的話，倒不如直接使用虛擬機 (VM) 還比較適當，可參考 [Red Hat OpenShift 把 VM 當 Container 管!?][5] 了解其細節。

## 應用程式交付就跟組樂高積木一樣

![](/images/application-delivery.png)

各位可以把容器映像檔想成樂高積木來看待

- 容器平台 (Container Platform) = 樂高積木平台
- 容器映像檔 (Container Images) = 樂高積木
- 應用程式定義 (Application Definition)= 產出物設計圖
- 應用程式 (Application) = 產出物

以容器為主體的應用程式交付，應該要盡量地做到類似於組樂高積木，容器能方便重複利用，可依據不同的程式設計，在容器平台上組出想要的應用程式。當然，要做到應用程式交付不單單只是要把容器映像檔做出來而已，還有一些額外的事情需要考慮：

1. 映像檔建構指引 (Image Build Instructions)
2. 映像檔來源控制 (Container Registries)
3. 編排定義 (Orchestration Definitions)
4. 維運 (Operators)

這當中有非常多層面的需要去考慮，那有沒有一個簡單明白的容器構建原則呢?

## 容器構建的原則

以下的 5 條戒律的討論基礎是基於所有內容、工作都視為可以從原始碼 (Source Code) 構建，具體內容為以下：

1. 標準化 (Standardize)：盡可能確保每一個人都使用相同的基礎映像檔 (Base Images)
2. 最小化 (Minimize)：映像檔僅裝載實際工作負載所需的內容
3. 委派 (Delegate)：保持映像檔各分層 (Layers) 由各具備專門技術的人負責
4. 過程和自動化 (Process and Automation)：透過 Helm Charts、Ansible Playbooks 或 Operators 等部署方式，盡可能做到自動化佈建
5. 迭代 (Iterate)：盡可能在迭代過程修正任何發現的原始碼錯誤

### 第一戒律：Standardize

> 好的基礎是成功的一半

![](/images/Commandent-1-standardize.png)


基礎映像檔的選擇一般會有三個路徑：

1. 從 Dockerhub `library/*` 選一個帶有 OS 基底往上疊
2. 從 Scratch 自己慢慢刻
3. 拿別人的 Middleware 映像檔當作自己的基礎映像檔往上疊

而 Red Hat 近期隨 RHEL8 釋出後，推出了一個比較通盤性的工具選擇： [紅帽通用基礎映像檔 (Red Hat Universal Base Image, UBI)][6]，但這以後再開一篇介紹，這邊不贅述。

### 第二戒律：Minimize

> 太小不一定好

![](/images/Commandent-2-minimize.png)

最小化不完全代表是檔案大小的最小化的意思，應是盡力減少不必要的依賴庫或者是程式在容器裡面，減少外部攻擊面為主，這邊推薦以下兩個連結學習如何構建出最小化容器的技術
- [Building Small Containers (Kubernetes Best Practices - YouTube)][7]
- [Best practices for building containers - Google Cloud][8]

### 第三戒律：Delegate

> 人的一天只有 1440 分鐘

![](/images/Commandent-3-delegate.png)

雖然每一個人拿到的 Base Image 是相同，但所要實作的程式卻不盡相同，有些人會想要用 Python、有些人需要是 Java，然而這些不同領域的經驗差距是非常大的，譬如程式庫相依性管理，所以建議是要將這類的中間層設計委任給有經驗的人維護或構造

### 第四戒律：Process and Automation

> 專注在真的需要你關心的工作上

![](/images/Commandent-4-focus-on-automation.png)

盡可能地將反覆操作的流程自動化，無論你是自建 CICD Pipeline 、寫 Ansible Playbook 亦或者是用 Operator Framework 作為自動化的實踐都沒有關係，主要方向是要將構建的時間縮短，不要有太多無謂的人為操作

### 第五戒律：Iterate

> 及早發現 及早治療

![](/images/Commandent-5-iterate.png)

透過流程自動化後，大幅縮短迭代時間，讓改變或修補的`過程`變為更為清晰，而做變更管理 (Change Management) 的時候也相對會有比較強的信心去做應對

## 附錄: 與自動化的距離

![](/images/xkcd.png)


## References
- [報告：前十大熱門Docker映像檔都有至少30個以上的漏洞 - iThome][2]
- [CRI-O official website][3]
- [Building Production-Ready Containers - Scott McCarty & Ben Breard][4]
- [Linux Container Intermals - how they really work][1]
- [Red Hat OpenShift 把 VM 當 Container 管!?][5]
- [Building Small Containers (Kubernetes Best Practices - Google)][7]
- [Best practices for building containers][8]

[1]: http://crunchtools.com/files/2019/05/Linux-Container-Internals-2.0.pdf
[2]: https://www.ithome.com.tw/news/129018
[3]: https://cri-o.io/
[4]: http://crunchtools.com/files/2019/05/Summit-2019_-Building-Production-Ready-Containers.pdf
[5]: https://blog.pichuang.com.tw/20190420-redhat-openshift-kubevirt
[6]: https://www.redhat.com/en/blog/introducing-red-hat-universal-base-image
[7]: https://www.youtube.com/watch?v=wGz_cbtCiEA
[8]: https://cloud.google.com/solutions/best-practices-for-building-containers