---
layout: post
title: 愛的走馬看花 Red Hat CoreOS 與 Red Hat OpenShift Part 6
author: Phil Huang
toc: true
tags:
  - null
categories:
  - null
date: 2020-05-01 01:48:52
udpated: 2020-05-01 01:48:52
---


## 走馬看花第 6 天
### Red Hat Container Catalog

![](/images/rhcc.png)

Red Hat 有提供一個訂閱服務叫做 `Red Hat 容器目錄服務 (Red Hat Container Catalog, RHCC)`，這服務是完全對比 Docker Hub 的功能，最大差異就是裡面提供的 Container Images 都是由 Red Hat 官方提供維護和支援

Docker Hub 下載映像檔是沒什麼問題

- [iThome - Docker Hub上映像檔被發現存在挖礦綁架蠕蟲][1]
- [iThome - Docker Hub遭入侵，外洩19萬名用戶憑證][2]
- [iThome - 駭客掃描網路Docker植入挖礦程式，還修改設定、留下後門][3]
- [iThome - Docker移除17個暗藏挖礦程式的惡意容器][4]

以下是 2 條更新規則:
1. 每 6 週更新一次
2. 有重大 CVE 問題會更新



###
```bash
# New project for testing
$ oc new-project test-images


```

## References
- [iThome - Docker Hub上映像檔被發現存在挖礦綁架蠕蟲][1]
- [iThome - Docker Hub遭入侵，外洩19萬名用戶憑證][2]
- [iThome - 駭客掃描網路Docker植入挖礦程式，還修改設定、留下後門][3]
- [iThome - Docker移除17個暗藏挖礦程式的惡意容器][4]


[1]: https://www.ithome.com.tw/news/133655
[2]: https://www.ithome.com.tw/news/130275
[3]: https://www.ithome.com.tw/news/134470
[4]: https://www.ithome.com.tw/news/123887