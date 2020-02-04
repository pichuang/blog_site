layout: post
title: 關於容器倉庫的那些二三四
author: Phil Huang
tags:
  - container
categories:
  - container
date: 2020-02-01 03:45:00
---

隨著使用容器 (Container) 技術也有段時間了，多數人人生第一次使用 Container 應該都會嘗試下，`docker pull docker.io/library/centos` 的指令試試，那到底 `docker.io/library/centos` 有什麼玄機，今天想分享一下關於容器倉庫 (Container Repository/Registry) 的觀念探討，文章內的操作會以 `podman`、`skopeo` 為主

![](/images/container-repos.png)

<!--more-->

## Q&A
### Q1: 一般來說，正常的下載路徑長怎樣?

通用標準格式如下:
```bash
<HOSTNAME>/<NAMESPACE>/<IMAGE>:<TAG>
```

- Hostname: 就是一般常見的網址，譬如 `https://docker.io`、`https://registry.redhat.io` 或者是私有的 `http://quay.misc.internal` 等符合 [RFC 3986 - Uniform Resource Identifier (URI): Generic Syntax][2] 標準且以 `http` 或 `https` 為主的通訊協議皆可
- Namespace: 有些會稱作 project、project-id、Organization，意思是相同的，主要是分類的功用，不可省略，最知名的命名就是來自於 docker.io 的 `library`
- Image: 實際上要使用的 Container Image 名稱
- Tag: 針對每一個 Image，區分特定的版本，一般來說如果沒有特別寫的話，`多數預設狀況下會尋找標記 latest` 的標記，後面會討論這件事

至於有一些特殊的路徑，譬如 `<HOSTNAME>/<IMAGE>:<TAG>`，也是可以放，但我個人不太建議就是了，盡量保持命名規則的一致性及通用性


### Q2: 遠端跟地端的 Namespace 及 Image name 是否一定保持相同命名才能使用?

不需要，這邊有幾個情況可以討論

1. 在同一個倉庫裡，不同的 Namespace 可以有相同的 Image 命名，譬如下列狀況

```bash
# Test Case
# Local 1: quay.misc.internal/library/centos:latest
# Local 2: quay.misc.internal/golden/centos:latest

# 複製
skopeo copy docker://quay.misc.internal/library/centos:latest docker://quay.misc.internal/golden/centos:latest

# 比對
skopeo inspect docker://quay.misc.internal/library/centos:latest
skopeo inspect docker://quay.misc.internal/golden/centos:latest
```

這樣兩邊的 Container Images 內容會是一模一樣的，差異只有在 Namespace 命名不一樣而已，可以透過`skopeo` 進行對照

2. 在不同倉庫裡，相同 Namespace 命名可以有相同 Image 命名，譬如下列狀況

```bash
# Test Case
# Remote: docker.io/library/centos:latest
# Local: quay.misc.internal/library/centos:latest

# 複製
skopeo copy docker://docker.io/library/centos:latest docker://quay.misc.internal/library/centos:latest

# 比對
skopeo inspect docker://docker.io/library/centos:latest
skopeo inspect docker://quay.misc.internal/library/centos:latest
```

兩邊的 Container Images 內容還是一模一樣的，這常見用於定期同步 (Mirror / Replica) 遠端跟地端的映像檔

### Q3: 有沒有什麼映像檔標籤的命名慣例呢?

依據 [Tagging images - OpenShift Container Platform 4.2][3] 推薦使用方式，其實說穿了基本也是跟隨常用的 [語意化版本 Semantic Versioning 2.0.0][4] 規矩走，除此之外，也有公司因為一天發布的頻率非常高，會搭配用 [ISO 8601 - Data and time format][5] 版本 `2004` 所規範的表示方式 `YYYYMMDDhhmm` 當作流水號紀錄

描述 | 範例
--|--
語意化版本 |myimage:v2.0.1
語意化版本+架構|myimage:v2.0-x86_64
語意化版本+基礎鏡像|myimage:v1.2-centos7
ISO 8601 (YYYYMMDDhhmm) |myimage:202002011200
ISO 8601 (YYYYMMDD) |myimage:20200201
最新穩定|myimage:stable
最新 (可能不穩定)|myimage:latest

至於這個標籤 `latest`，可能蠻多人對它有點誤解

### Q4: 關於 `latest` 有什麼誤解呢?

這邊直接引用 [What's Wrong With The Docker :latest Tag?][1] 的文章，寫的蠻直白的，摘要如下：

1. `:latest` 只是一個標籤 (Latest is Just a Tag)
2. `:latest` 不會自動更新 (Latest is Not Dynamic)
3. 預設狀況下，`:latest` 很容易被覆蓋掉 (Latest is Easily Overwritten By Default)
4. `:latest` 本身不具備任何描述性，不易生產使用 (Latest is Not Descriptive and Hard to Work With)

舉一個實際狀況，在 2019/06 那段時間 `library/centos:latest` 是指向 `library/centos:7` 這個版本，而在某一天 `library/centos:8` 出了，而這個 `library/centos:latest` 就被上游管理者手動指過去了，而並不是指向 `library/centos:7.x` 的最新版本，導致使用 `:latest` 為標籤的使用者有不小的災難

根本上的問題就是 latest 這個字的語意不太好，字面上的意思並不能完全表達他的真正含義，而在現行一個程式擁有多版本維護需求狀況下 (e.g: centos:7.x, centos:8.x)，不是特別適當，故寫 Dockerfile/Containerfile 務必要把指定版號寫進去才不會照成混亂，也相對比較好維護

### Q5: 有建議的命名方式嗎?

公用的標籤可以取名為 `stable`，或者是直接依照 [語意化版本 Semantic Versioning 2.0.0][4] 進行命名

個人經驗上，多數沒有什麼需要版本化命名的需求，預設都是採用 `ISO 8601 (YYYYMMDD)` 當作標籤名，既可以確保不會重複標籤，也可以直觀理解版本意義

譬如像 [pichuang/debug-container](https://quay.io/repository/pichuang/debug-container?tab=tags) 所呈現的樣子

### Q6: 所以 latest 是不是應該要完全被禁止?

不需要，目前現行約定俗成的作法就是，當你沒有特別指定標籤名的話，預設就是會先去抓 `:latest` 的標籤，可以撈到 manifest 的資訊回來，裡面通常都會包含完整的 RepoTags 可供查詢

```bash
# 各位試試這條指令，然後看著 latest 標籤，試問他指向哪一個版本?
skopeo inspect docker://docker.io/library/centos | jq ".RepoTags"
```

## 結語

Latest 標籤引發的誤解，應該或多或少都有導致一些狀況發生，請大家一定要依據正確的觀念來使用這個 latest 標籤，這邊也有一篇 [Docker中latest标签引发的困惑][6] 探討相關的議題也可以補充參考

## References
- [What's Wrong With The Docker :latest Tag?][1]
- [Tagging images - Red Hat OpenShift Container Platform 4.3][3]
- [語意化版本 Semantic Versioning 2.0.0][4]
- [ISO 8601:2004][5]
- [Docker中latest标签引发的困惑][6]

[1]: https://vsupalov.com/docker-latest-tag/
[2]: https://tools.ietf.org/html/rfc3986
[3]: https://access.redhat.com/documentation/en-us/openshift_container_platform/4.3/html/images/managing-images#tagging-images
[4]: https://semver.org/lang/zh-TW/
[5]: https://zh.wikipedia.org/wiki/ISO_8601
[6]: http://dockone.io/article/165
