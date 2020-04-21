---
layout: post
title: 於 OpenShift 4 上運行混沌工程測試 KubeInvaders
author: Phil Huang
toc: true
tags:
  - openshift4
  - chaos engineering
  - kubeinvaders
categories:
  - openshift
date: 2020-04-14 17:14:50
udpated: 2020-04-14 17:14:50
---

混沌工程 (Chaos Engineering)，最早期我知道這個單字是因為做 SDN 網路的時候，BigSwitch Network (現已被 Arista 收購) 那時候有實作出對網路進行不定期、無法預知之斷線行為，來確保網路 HA 是否暢通，[Chaos Monkey and Big Cloud Fabric - Big Switch Networks][3]。當時才認真看了下最源頭來自於 NetFlix 的 [Netflix/chaosmonkey][4] 指導原則

1. 建立一個圍繞穩定狀態行為的假說 (Build a Hypothesis around Steady State Behavior)
2. 多樣化真實世界的事件 (Vary Real-world Events)
3. 在生產環境中運行實驗 (Run Experiments in Production)
4. 持續自動化運行實驗 (Automate Experiments to Run Continuously)
5. 最小化爆炸半徑 (Minimize Blast Radius)

基於上面的指導原則，這邊選了一套適用於 Kubernetes 的混沌工程原則的工具，名叫 `KubeInvaders`

{% youtube 3OOXOCTAYF0 %}

<!--more-->

## KubeInvaders

> KubeInvaders: 針對 Kubernetes 遊戲化混沌工程，把 Pod 當作外星人攻擊

這專案提供了一個簡單的網頁介面，裡面用 `Defold` 寫的一個`射擊外星人 (Space Invaders)` 的小遊戲，但這個小遊戲裡面的外星人，通通都是你指定的 Kubernetes Namespace 裡面的實際運行的 Pod

按照 Kubernetes 的設計，只要對 Kubernetes 聲明 (Declarative) 說，要在這個集群內要保持幾個 Pods，Kubernets 就會在資源充足的前提之下，把錯誤或被移出的 Pod 重新跑新的起來

所以回到剛剛講的 5 個混沌工程指導原則，套用在這個 KubeInvaders 及 Red Hat OpenShift 身上的解釋就會變成

1. 建立一個圍繞穩定狀態行為的假說 (Build a Hypothesis around Steady State Behavior) => OpenShift 容器平台及其上面容器服務本身是`穩定且正常工作`
2. 多樣化真實世界的事件 (Vary Real-world Events) => 譬如說砍錯 Pod、移除錯 Worker 節點等`具備破壞穩定狀態的事情`
3. 在生產環境中運行實驗 (Run Experiments in Production) => 我相信是 NetFlix 限定，不然大部分應該都是會選擇在 UAT / SIT
4. 持續自動化運行實驗 (Automate Experiments to Run Continuously) => KubeInvaders 自動化射擊外星人 (Pods)
5. 最小化爆炸半徑 (Minimize Blast Radius) => 只砍 Pod，其他不動

講這麼多，來動手玩一下就知道了

## 環境資訊

- Red Hat OpenShift 4.3.10 (Kubernetes v1.16.2)
- KubeInvaders 使用 `master branch`
- Red Hat Enterprise Linux 7.7 as bastion server
- Target Project: `bookinfo`

## 如何安裝?

我都照 [lucky-sideburn/KubeInvaders][2] README 所述的步驟執行，算寫的蠻清楚的，這邊選擇的方式是用 `Install KubeInvaders on Openshift`

### 下載 `KubeIvaders`
```bash
$ git clone https://github.com/lucky-sideburn/KubeInvaders
$ cd KubeInvaders
```

### 安裝 `KubeInvaders`
```bash ~/KubeInvaders
$ oc create clusterrole kubeinvaders-role --verb=watch,get,delete,list --resource=pods

$ TARGET_NAMESPACE=bookinfo
$ ROUTE_HOST=kubeinvaders.apps.ocp4.internal

$ oc new-project kubeinvaders --display-name='KubeInvaders'
$ oc create sa kubeinvaders -n kubeinvaders
$ oc adm policy add-cluster-role-to-user kubeinvaders-role -z kubeinvaders -n kubeinvaders

$ KUBEINVADERS_SECRET=$(oc get secret -n kubeinvaders --field-selector=type==kubernetes.io/service-account-token | grep 'kubeinvaders-token' --color=never | awk '{ print $1}' | head -n 1)
$ echo $KUBEINVADERS_SECRET

$ oc process -f openshift/KubeInvaders.yaml -p ROUTE_HOST=$ROUTE_HOST -p TARGET_NAMESPACE=$TARGET_NAMESPACE -p KUBEINVADERS_SECRET=$KUBEINVADERS_SECRET | oc create -f -
```

### 獲得網址
```bash
$ KubeInvaders_URL=$(oc -n kubeinvaders get route kubeinvaders -o jsonpath='{.spec.host}') && echo "KubeInvaders URL: https://$KubeInvaders_URL"

KubeInvaders URL: https://kube.apps.ocp4.internal
```

### 打開瀏覽器

![](/images/kubeinvaders.png)

- 按 `a` 會自動攻擊
- 按 `m` 切回手動攻擊
- 按 `i` 可以顯示攻擊目標的 Pod 名字
- 按 `n` 可以切換已經設定好在 `TARGET_NAMESPACE` 的 namespaces

## 實際 Demo

小弟原汁原味配音

{% youtube kXm2uU5vlp4 %}

## 清除 KubeInvaders
```bash
oc delete project kubeinvaders
oc delete clusterrole kubeinvaders-role
```

## Refererences
- [KubeInvaders - Gamified Chaos Engineering Tool for Kubernetes][1]
- [lucky-sideburn/KubeInvaders][2]
- [Chaos Monkey and Big Cloud Fabric - Big Switch Networks][3]
- [Netflix/chaosmonkey][4]
- [混沌工程原则 （PRINCIPLES OF CHAOS ENGINEERING）][5]

[1]: https://kubernetes.io/blog/2020/01/22/kubeinvaders-gamified-chaos-engineering-tool-for-kubernetes/
[2]: https://github.com/lucky-sideburn/KubeInvaders
[3]: http://go.bigswitch.com/rs/974-WXR-561/images/Chaos_Monkey_and_Big_Cloud_Fabric.pdf
[4]: https://github.com/Netflix/chaosmonkey
[5]: https://github.com/wizardbyron/principlesofchaos_zh-cn