layout: post
title: 建立私有 Swagger UI 服務
author: Phil Huang
tags:
  - swagger
  - openapi
categories:
  - misc
date: 2020-01-31 10:45:00
---

想必常使用 RESTful API 開發的人對於好看好讀的 API Docs 有相當程度的需求，而眾多開源軟體中，最知名的框架就是 `Swagger` 這個框架，什麼？你說你不知道這是什麼，我相信你一定多多少少應該都有看過這個蠻經典的 [petstore.swagger.io][2] API 文件，本文就是要來梳理一下這 Swagger UI 要怎麼放到自家，用容器運行非常簡單

![](/images/swagger.png)

<!--more-->

## Q: 為什麼我需要在家裡建一個 Swagger UI Server?

petstore.swagger.io 是一個放在外網 Internet 上的服務供大家做`線上`使用，所以你只需要把你的 API Endpoints 指向到 Swagger 上，Swagger UI 會依據你提供的網址去拉取 OpenAPI Spec 回來做文件化顯示。

但因為大多數企業端的服務都在內網 Intranet，所以放置在外網的 Swagger UI，是沒有辦法拉到相關的程式進行呈現，所以在內網內架設一個私有服務是有其必要性的

## 環境介紹

- OS: Red Hat Enterprise Linux 7.7
- Container Runtime: podman 1.4.4
- Swagger Generator: 2.4.12

## 安裝步驟

### Firewall 開通
```bash
firewall-cmd --list-all
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --reload
firewall-cmd --list-all
```

這邊是開啟 host 上的 80 port，如果要選擇其他的 port 也可以，請自行調整

### 確認版本及下載 Swagger-Generator

```bash
# 採用 Skopeo 來檢查現行有什麼標籤可以選，選擇當前最新的標籤 2.4.12
yum install -y skopeo
skopeo inspect docker://docker.io/swaggerapi/swagger-generator | jq ".RepoTags"

# 下載 Container Image - swaggerapi/swagger-generator:2.4.12
podman pull docker.io/swaggerapi/swagger-generator:2.4.12

# 執行
podman run --rm -d \
    -e GENERATOR_HOST=http://quay.misc.internal/api/v1/discovery \
    -p 80:8080 \
    docker.io/swaggerapi/swagger-generator:2.4.12

# 打開你的瀏覽器 http://IP:80
```

如果各位比較習慣用 `docker` 的話，也可以自行將 `podman` 置換成 `docker` 指令，並沒有什麼問題；除此之外，因為現在執行的方式是關機後就消失，所以如果想要讓他開機啟動的話，可以參考這篇 [pichuang/etc_sysconfig_gitlab][3] 自行撰寫 Systemd Unit 來管理服務


[1]: https://swagger.io/
[2]: http://petstore.swagger.io/
[3]: https://gist.github.com/pichuang/7ce8be00c3de3f51e5c8db0689f1e08a
