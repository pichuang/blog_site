layout: post
title: Nvidia DGX 和 Red Hat OpenShift 會產生什麼火花?
author: Phil Huang
tags:
  - redhat
  - openshift
  - nvidia
  - dgx
  - tesla
  - gpu
  - ngc
categories:
  - openshift
date: 2019-03-30 00:30:00
---
一日江湖行走，路上撿到一個問題：`Nvidia DGX 要怎麼跟透過容器化調度應用呀?` 然後我就開始了一連串的研究探索...

![](/images/ngc-openshift.png)

<!--more-->

## 先來個名詞解釋
- [NVIDIA Tesla V100 Tensor][2]
  - 白話文: 一張超高階 GPU 卡
  > NVIDIA® Tesla® V100 是最先進的資料中心 GPU，專為加快人工智慧、HPC 和繪圖運算速度而設計。採用 NVIDIA Volta 架構，提供 16 GB 和 32 GB 設定，單一 GPU 即可展現媲美 100 個 CPU 的效能。資料科學家、研究人員和工程師可以省下配置記憶體使用最佳化的時間，投入更多精力設計下一個人工智慧的重大突破。

- [NVIDIA DGX-1][3]
  - 白話文: 一台 x86 Server，內可包含至多 8 張超高階 GPU 卡， GPU Memory 可達 256G 
  
- [NVIDIA GPU Cloud (NGC)][4]
  - 白話文: 一堆各式各樣 Nvidia 出品的容器，請洽 https://ngc.nvidia.com/catalog/landing
  > NVIDIA GPU Cloud (NGC) 容器 registry 提供一系列完備的 GPU 加速人工智慧容器，這些容器經過最佳化、測試，並可立即透過本機和雲端上支援的 NVIDIA GPU 執行。TensorFlow、PyTorch、MXNet、NVIDIA TensorRT™ 等 NGC 人工智慧容器，透過 NVIDIA 人工智慧的強大功能，賦予你挑戰極困難專案所需的效能與彈性。這能幫助資料科學家和研究人員迅速建立、訓練和部署人工智慧模型，以滿足與時俱進的需求
- [Red Hat OpenShift Container Platform](https://www.openshift.com/)
  - 白話文: 一座容器平台
  - 紅帽企業級容器平台，可依據不同的環境需求，調整基礎架構，同時於平台之上也能搭載各式各樣的容器服務，如 NGC，目前長期支援版本為 v3.11

## 所以雙方到底會是什麼火花？

這影片 [Red Hat OpenShift, NVIDIA DGX and NGC Container Integration - Youtube][1] 充分表達了`如何使用 OpenShift 部署 NGC 的容器服務，進而控制 DGX-1 的運算資源`

{% youtube 9iVYjA_WJgU %}

1. OpenShift Service Catalog 選取已經定義好的 Container (TensorFlow, PyTorch, MXNet, TensorRT, ...) 要多少 GPU 來做運算
2. 從 OpenShift 已經開起來的 Pod 中，來調度 DGX-1 裡面的資源利用

## 怎麼做到的?

這份投影片 [Deep Learning Inference on Openshift with GPUs][5] 其實介紹蠻詳細的，尤其是下圖

![](/images/ngc-openshift-1.png)

- Device Manager Support, Priority and Preemption
  - OpenShift 3.10 後就開始正式支援 GPUs with DevicePlugin，詳細可參考 [Blog: How to use GPUs with DevicePlugin in OpenShift 3.10][6]
- OpenShift/Red Hat Enterprise Linux on Nvidia DGX-1/Tesla - NVIDIA NGC on OpenShift
  - DGX-1 目前 N 家官方是支援 Red Hat Enterprise Linux 7.5+，詳細可以參考 [NVIDIA, Red Hat certify NVIDIA DGX-1 for Red Hat Enterprise Linux][7]
- Preview of TensorRT Inferencing on OpenShift
  - 請自行觀看 [GitHub - NVIDIA/tensorrt-inference-server](https://github.com/NVIDIA/tensorrt-inference-server)
- GPU Sharing, Heterogeneous Clusters, GPU Topology, Install experience with Operators
  - 各位可以期待一下 [`GPU-Operator`](https://github.com/NVIDIA/gpu-operator)，你問什麼是 Operator？傳送門在這邊 [What is an Operator? - Operatorhub.io](https://operatorhub.io/what-is-an-operator)

## Summary

那個影片後來發現其實在去年的 [SuperComputing 2018 (SC18)](https://sc18.supercomputing.org) 的會議上，就有被拿出來 Demo 了，從那之後 Red Hat 跟 Nvidia 在 AI/HPC 上的合作就越來越多了，前者出基礎架構平台，後者出 GPU 跟運算框架，在不久的將來應該還可以看到越來越多 GPU + Container 的使用方式出來，關於更多 NVDIA 的資訊，可以參考 [擴充應用程式映像，Nvidia簡化叢集環境與兩大產業容器部署 - iThome][8] 一文

## References
- [Red Hat OpenShift, NVIDIA DGX and NGC Container Integration - Youtube][1]
- [NVIDIA Tesla V100 Tensor][2]
- [NVIDIA DGX-1][3]
- [人工智慧 (AI) 容器][4]
- [Deep Learning Inference on Openshift with GPUs][5]
- [Blog: How to use GPUs with DevicePlugin in OpenShift 3.10][6]
- [NVIDIA, Red Hat certify NVIDIA DGX-1 for Red Hat Enterprise Linux][7]
- [擴充應用程式映像，Nvidia簡化叢集環境與兩大產業容器部署 - iThome][8]

[1]: https://www.youtube.com/watch?v=9iVYjA_WJgU
[2]: https://www.nvidia.com/zh-tw/data-center/tesla-v100/
[3]: https://www.nvidia.com/zh-tw/data-center/dgx-1/
[4]: https://www.nvidia.com/zh-tw/gpu-cloud/deep-learning-containers/
[5]: https://blog.openshift.com/wp-content/uploads/Nvidia-RedHat-Commons-Kubecon-2018.pdf
[6]: https://blog.openshift.com/how-to-use-gpus-with-deviceplugin-in-openshift-3-10/
[7]: https://blogs.nvidia.com/blog/2018/10/23/red-hat-enterprise-linux-dgx-1/
[8]: https://www.ithome.com.tw/review/128970