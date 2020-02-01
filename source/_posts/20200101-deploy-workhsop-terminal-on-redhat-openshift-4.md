layout: post
title: 部署 Workshop Terminal 於 Red Hat OpenShift 4
author: Phil Huang
tags:
  - openshift
  - terminal
categories:
  - openshift
date: 2020-01-01 13:08:00
---
當你出門在外，遇到要帶一群人進行 Workshop，但學員的電腦各種無法安裝 Putty、XShell，該怎麼辦? 

本文將使用 [Workshop Terminal][2] 來當作 shell 來做使用

![](/images/terminal-quay.png)

<!--more-->

## 部署環境
- Red Hat OpenShift 4.2.7
- openshifthomeroom/workshop-terminal 3.4.2


## 部署 workshop-terminal

以專案 terminal-admin 為例，很簡單，三行搞定

```bash
# Change project name terminal-admin if you want 
oc project terminal-admin

# Deploy workshop-terminal Template
oc new-app https://raw.githubusercontent.com/openshift-homeroom/workshop-terminal/master/templates/production.json

# Obtain URL link
oc get route terminal
```

- View of Object Overview
![](/images/terminal-overview.png)

- View of Object Resouces
![](/images/terminal-resources.png)


## 移除 workshop-terminal
```bash
oc project terminal-admin
oc delete all,serviceaccount,rolebinding,configmap -l app=terminal
```

## PS1 上色

因為預設的 PS1 太難看了，上個色，好使用

```bash
export PS1="\[\e[1;36m\]\t \[\e[01;31m\]\u\[\e[m\]@\h \n$ "
```

![](/images/terminal-environment.png)

## 設定密碼

上個 `HTTP Basic authentication` 用用

```bash
# Change the USERNAME and PASSWORD if you want
oc set env dc/terminal AUTH_USERNAME=redhat AUTH_PASSWORD=redhat
```

## 想要打包其他的程式進去?

請參考 [Creating a custom image][3] 章節


[1]: https://github.com/openshift-homeroom/workshop-terminal
[2]: https://quay.io/repository/openshifthomeroom/workshop-terminal?tag=master&tab=tags
[3]: https://github.com/openshift-homeroom/workshop-terminal#creating-a-custom-image