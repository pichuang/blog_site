layout: post
title: 何時該採用多叢集 Kubernetes
author: Phil Huang
date: 2021-11-25 12:55:32
tags:
  - kubernetes
categories:
  - kubernetes
toc: true
---

<!--more-->

## 何謂 1 座 Kubernetes?

想必大家都知道若要操作管理 Kubernetes 都需要使用 `kubectl`


## Kubernetes 隔離性考慮

1. 不同網路環境，如 Prod / SIT / UAT / DR 等
2. 單一叢集 Kubernetes 合規性，如 PCI-DSS、HIPPA 等
3. 底層 CPU 架構集不同，如 x86_64、ppc 等
4. 非技術議題，如不同部門權限劃分、內帳切割、權責問題等
5. 不同專案屬性，如核心系統、以 API Gateway 為主的 MASA 設計、組織內共用服務平台（CI/CD）、GPU 分享技術池等

[1]:
