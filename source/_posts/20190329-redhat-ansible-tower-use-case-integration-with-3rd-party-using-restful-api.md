layout: post
title: 'Red Hat Ansible Tower 常見使用案例: 用 RESTful API 與第三方產品結合'
author: Phil Huang
tags:
  - redhat
  - ansible
  - restful
  - openapi
categories:
  - ansible
date: 2019-03-29 02:31:00
---
有些長官看完 [Red Hat Ansible Tower 常見使用案例: 跨多平台設備管理][4] 後說:
`Ansible Tower 看起來好棒棒，但自家已經有自有的 Service Catalog，而我就是不想用 Ansible Tower 當作我的第一層服務，那怎麼辦？`

<!--more-->

[eanylin 哥][5]剛好也分享出一系列影片: `用 Service Catalog 系統戳 Ansible Tower 內建的 RESTful API`

- [透過 Service Catalog 部署 Web Application][1]
- [透過 Service Catalog 新增 Web Server][2]
- [透過 Service Catalog 下版 Web Application][3]

![](/images/restful-api.png)

## Q1: 具體 RESTful API 有哪些功能可以使用?

Red Hat 秉持著專門撰寫`鉅細彌遺地巨擘等級文件`的精神，請參考這邊 [Ansible Tower API Guide v3.4.3][6]

而具體對技術來講，這份 [Tower API Reference Guide][7] 是最重要的，裡面完全遵循標準 OpenAPI 的 Swagger 2.0 格式，讓其他平台要接入的時候，有充分的資訊可以使用

我知道你內心想問 OpenAPI 是蝦米哇歌，傳送門: [進擊的 OpenAPI!!! - Phil Huang][9]

## Q2: 好吧 那我用 RESTful API 具體上到底是控制什麼?

以 Ansible Tower 為核心，分為南北向的資訊流的話，北向就是 RESTful API，而南向就是 Ad-hoc / Playbook / Workflow 等等具體跟其他設備對接的腳本

[Tower API Reference Guide][7] 有列舉所有你可能會用到的 RESTful API，但其實真正可以讓你去執行 Ad-hoc / Playbook / Workflow 能操作其他系統的只需了解三大項:

1. [Ad Hoc Commands][12] => 操作 Ad-hoc
2. [Job Template][10] => 操作 Playbook
3. [Workflow Job Templates][11] =>操作 Workflow

## Q3: 哎呦 那看起來很簡單啊 那可不可以硬刻?

技術上當然可以，但你最少要了解以下幾個東西
- OpenAPI Specification
- Ansible module integration
- Fronend Service Implementation / Design
- Backend Service Implementation / Design
- System Administrator and Management knowledge / integration
- Integration Testing/QA
- 所有前後端網頁常識
- Maintainance / Update process
- Project Roadmap
- <會寫 js 這語言的各位大大...真的太偉大了>
- <我真心不了解前端的世界>
- <抱歉我天生不是前端的料，我是來騙行數的>
- ...

有沒有發現你在做一個全新大專案出來

## Q4: 疑...這樣 Ansible Tower 是不是可以被 Jenkins 戳?

是的，這當然是沒什麼問題，在那個不能說的客戶們跟這個也不能說的客戶們有些是以 `Jenkins` + `Ansible Tower` 當作整個 CICD 核心引擎進行驅動

寫這麼多不如來看個演講 [Red Hat Summit Demo CI/CD deployment][13] (但我知道你不想看，所以可以看下圖 XD)

![](/images/ci_cd_redhatsummit_2016.png)


## Q5: 阿不通通用 Jenkins call Ansible 就好了? 幹嘛還多一個 Ansible Tower 的角色?

因為 Ansible Tower 不是單單只會被開發團隊的 Jenkins 拿來寫在 Jenkinsfile 裡面做使用，同時之間還可以搭配 Workflow 同時進行其他的事情，類似下圖
 
![](/images/restful-api-1.jpg)
 
 如果論工程差異的話，除了角色上有差異，功能性也會差異點，Ansible Tower 相比 Jenkins
- 自帶能做定期排程工作
- RBAC 權限細粒度佳
- 可依據不同環境分配和管理不同的 serects 和 credentials
- 多了個 Workflow 可以用
- 能委派工作給特定的機器使用 (防火牆內)
- 集中化控管
- 版本控制
- 新手上手容易

額外閱讀: [Integrating Ansible with Jenkins in a CI/CD process - Red Hat Blog][14]

## Summary

`所有服務皆 API 化` 這個想法讓系統整合廠商多了無限整合可能，若能好好地善用 RESTful API + Ansible Workflow 可以讓你節省掉很多的系統整合風險和持續保有優秀的架構解隅性，但又不會失去可控性


## References
- [CloudForms/API Driven Workflow with Ansible Tower (Part 1) - Provision Web Application (Catalog)][1]
- [CloudForms/API Driven Workflow with Ansible Tower (Part 2) - Adding Web App Server (Catalog)][2]
- [CloudForms/API Driven Workflow with Ansible Tower (Part 3) - Decommission Web Application (Catalog)][3]
- [Red Hat Ansible Tower 常見使用案例: 跨多平台設備管理][4]
- [Ansible Tower API Guide v3.4.3][6]
- [Tower API Reference Guide][7]
- [進擊的 OpenAPI!!!][9]
- [Tower API Reference Guide - Job Template][10]
- [Tower API Reference Guide - Workflow Job Templates][11]
- [Tower API Reference Guide - Ad Hoc Commands][12]
- [Red Hat Summit Demo CI/CD deployment][13]
- [Integrating Ansible with Jenkins in a CI/CD process][14]

[1]: https://www.youtube.com/watch?v=HOPbhlTBG24
[2]: https://www.youtube.com/watch?v=RqNtJaxHlpU
[3]: https://www.youtube.com/watch?v=2wdyneoiAX0
[4]: https://blog.pichuang.com.tw/20190328-ansible-use-case-multi-network-devices-configuration-management-using-ansible-tower/
[5]: https://www.youtube.com/channel/UCsAGOvR4jdcFQLyTFHnIJgQ/featured
[6]: https://docs.ansible.com/ansible-tower/latest/html/towerapi/index.html
[7]: https://docs.ansible.com/ansible-tower/latest/html/towerapi/api_ref.html
[8]: https://docs.ansible.com/ansible-tower/latest/html/towerapi/tools.html
[9]: https://blog.pichuang.com.tw/20180723-openapi-and-oas/
[10]: https://docs.ansible.com/ansible-tower/latest/html/towerapi/api_ref.html#/Job_Templates
[11]: https://docs.ansible.com/ansible-tower/latest/html/towerapi/api_ref.html#/Workflow_Job%20Templates
[12]: https://docs.ansible.com/ansible-tower/latest/html/towerapi/api_ref.html#/Ad_Hoc%20Commands
[13]: https://www.youtube.com/watch?v=wMTgIIJ-oqQ
[14]: https://www.redhat.com/en/blog/integrating-ansible-jenkins-cicd-process