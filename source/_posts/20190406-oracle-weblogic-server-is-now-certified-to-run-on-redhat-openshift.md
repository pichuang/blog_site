layout: post
title: Oracle WebLogic 正式支援跑在 Red Hat OpenShift 上
author: Phil Huang
tags:
  - openshift
  - redhat
  - weblogic
  - oracle
  - operator
categories:
  - openshift
date: 2019-04-06 13:57:00
---
Oracle Blog 於 2019/3/11 釋放出一則文章 [WebLogic Server is now certified to run on OpenShift! - ORACLE][1]，是的，Oracle WebLogic 可以正式運行在 Kubernetes / OpenShift 上的平台了!

![](/images/weblogic.png)

<!--more-->

## 版本支援
Product | Version
---|---
WebLogic Server|12.2.1.3+
WebLogic Kubernetes Operator|2.0.1+
Red Hat OpenShift|3.11.43+
Kubernetes|1.11.0+
Docker|1.13.1ce+

有關於容器平台的部分，是以 Kubernetes 1.11 對齊，所以 Red Hat OpenShift v3.11 是完全可以使用的

## 技術概觀

基礎上就是將 Oracle WebLogic 透過 [Operator 化][5]後，產生出一個專案 [oracle/weblogic-kubernetes-operator - GitHub][3]，爾後所有的安裝維護都是透過 Operator 協助，詳細的安裝過程可以參考 [Running WebLogic on OpenShift - ORACLE][2]，更詳細的技術說明文件可參考 [Oracle WebLogic Server Kubernetes Operator][4]


## References
- [WebLogic Server is now certified to run on OpenShift! - ORACLE][1]
- [Running WebLogic on OpenShift - ORACLE][2]
- [oracle/weblogic-kubernetes-operator - GitHub][3]
- [Oracle WebLogic Server Kubernetes Operator][4]
- [Operator Framework][5]

[1]: https://blogs.oracle.com/weblogicserver/weblogic-server-is-now-certified-to-run-on-openshift-v2
[2]: https://blogs.oracle.com/weblogicserver/running-weblogic-on-openshift
[3]: https://github.com/oracle/weblogic-kubernetes-operator
[4]: https://oracle.github.io/weblogic-kubernetes-operator/quickstart/
[5]: https://coreos.com/operators/