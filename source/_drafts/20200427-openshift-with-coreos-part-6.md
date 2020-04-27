---
layout: post
title: 愛的走馬看花 Red Hat CoreOS 與 Red Hat OpenShift Part 6
author: Phil Huang
toc: true
tags:
  - null
categories:
  - null
date: 2020-04-27 10:15:40
udpated: 2020-04-27 10:15:40
---


123

<!--more-->

### 新增其他帳號

OpenShift 基本上會有 Users/Groups/Projects 可以進行管理，以下參考 [OpenShift 4之设置用户/组对项目的访问权限][3] 的文章來實作:

- Users
  - Developers
    - developer1/developer1
    - developer2/developer2
    - user-p12/user-p12: 不歸屬在 Groups 裡
    - user-p21/user-p21: 不歸屬在 Groups 裡
  - Testers
    - tester1/tester1
    - tester2/tester2
    - user-p11/user-p11: 不歸屬在 Groups 裡
    - user-p22/user-p22: 不歸屬在 Groups 裡
- Groups
  - developer-group: 對於 `project-cicd` 專案具備設定權限，對 `project-uat` 僅有查看權限
  - tester-group: 對於 `project-uat` 專案具備設定權限，對 `project-cicd` 僅有查看權限
- Projects
  - project-cicd: 開發用環境
  - project-uat: UAT 環境

1. 如果沒存的話，撈出既有環境 `htpasswd` 檔案
```bash
$ oc get secret local-htpasswd-secret  -ojsonpath={.data.htpasswd} -n openshift-config | base64 -d | tee localuser-htpasswd
```

2. 新增 13 個好漢進 `localuser-htpasswd`
```bash
#
# Role to Group for all projects
#

# 平台管理員 user-a1 檢視跨專案權限
$ htpasswd -b localuser-htpasswd user-a1 user-a1

# 平台管理員 user-a2 設定跨專案權限
$ htpasswd -b localuser-htpasswd user-a2 user-a2

# 系統管理員 user-s1 檢視跨資源權限
$ htpasswd -b localuser-htpasswd user-s1 user-s1

# 系統管理員 user-s2 設定跨資源權限
$ htpasswd -b localuser-htpasswd user-s2 user-s2

#
# Role to Users for specific project
#

# 測試人員 user-p11 檢視單一專案 project-cicd 權限
$ htpasswd -b localuser-htpasswd user-p11 user-p11

# 開發人員 user-p12 設定單一專案 project-cicd 權限
$ htpasswd -b localuser-htpasswd user-p12 user-p12

# 開發人員 user-p21 檢視單一專案 project-uat 權限
$ htpasswd -b localuser-htpasswd user-p21 user-p21

# 測試人員 user-p22 設定單一專案 project-uat 權限
$ htpasswd -b localuser-htpasswd user-p22 user-p22


#
# Role to Group for specific project
#

# 開發人員 developer1 設定單一專案 project-cicd 權限 且 檢視單一專案 project-uat 權限
$ htpasswd -b localuser-htpasswd developer1 developer1

# 開發人員 developer2 設定單一專案 project-cicd 權限 且 檢視單一專案 project-uat 權限
$ htpasswd -b localuser-htpasswd developer2 developer2

# 測試人員 tester1 設定單一專案 project-uat 權限 且 檢視單一專案 project-cicd 權限
$ htpasswd -b localuser-htpasswd tester1 tester1

# 測試人員 tester2 設定單一專案 project-uat 權限 且 檢視單一專案 project-cicd 權限
$ htpasswd -b localuser-htpasswd tester2 tester2

# Show all user mapping
$ cat localuser-htpasswd
ocproot:$2y$05$pMTSySH/qa.gTcom9q/Pd.WHoI8VMIpCHWgc9d7cnQuIbRuuPeMia
user-a1:$apr1$T.FGLAA3$BdfIqYHXK9pg3qbQEGtRM0
user-a2:$apr1$m0a95/sq$rov1x31OjqcKYVr1I63rZ0
user-s1:$apr1$rvZGviL3$0BnJowG9OWFZ9BLP29cxK/
user-s2:$apr1$G3jerCSb$fCy8.NFDQYCT3IbAK61iu1
user-p11:$apr1$XzLUlsHt$jTsG/zKMS90CDdCCcz49L0
user-p12:$apr1$N48fhZY5$0r3Toc7P12zpIqfjjT8eW1
user-p21:$apr1$bymF4LqG$obL9vPBo6apsc7wV3w4Gw.
user-p22:$apr1$LiNEAKJ4$oND9r471EPIhoa.yJ95L00
developer1:$apr1$f7fpvHKb$X.MnMay0USf0.SOg/4hm6.
developer2:$apr1$FtL.4QEj$lQgfnYB42klWq5.BYnUxx0
tester1:$apr1$AuGDNNK/$U9Y0ytnT3qrakKat6jYkD1
tester2:$apr1$z1H./q7P$cvi62jxjmDOyMHQ/Gsx6R0
```

3. 更新既有的 `local-htpasswd-secret`
```bash
$ oc create secret generic local-htpasswd-secret --from-file htpasswd=/root/user/localuser-htpasswd --dry-run -o yaml -n openshift-config | oc replace -f -
secret/local-htpasswd-secret replaced
```

### 管理資源及賦予權限

OpenShfit 預設權限控制能做到 3 種層級，分別是：
1. 指定 Roles 給 user 或 groups
  - role-to-user
  - role-to-group
2. 指定 Cluster Roles 給 users 或 groups
  - cluster-role-to-user
  - cluster-role-to-group
3. 指定 SCC (Security Context Constraints) 來管理 Pods 和 Containers 給 users 或 groups
  - scc-to-user
  - scc-to-group

如果想要查詢有什麼 Role 可以使用的話，可以用下列指令撈，相當多記得要 filter 一下
```bash
$ oc describe clusterrole.rbac
```

```bash
# Create Projects
$ oc new-project project-cicd
$ oc new-project project-uat

# Create groups
$ oc adm groups new developer-group
$ oc adm groups new tester-group

# Assgin Users and Groups
$ oc adm groups add-users developer-group developer1 developer2
group.user.openshift.io/developer-group added: ["developer1" "developer2" "user-p12" "user-p21"]

$ oc adm groups add-users tester-group tester1 tester2
group.user.openshift.io/tester-group added: ["tester1" "tester2"]

# Assgin Role to Group within Project
$ oc adm policy add-role-to-group edit developer-group -n project-cicd
$ oc adm policy add-role-to-group view developer-group -n project-uat
$ oc adm policy add-role-to-group edit tester-group -n project-uat
$ oc adm policy add-role-to-group view tester-group -n project-cicd

# Assgin Role to User within Project
$ oc adm policy add-role-to-user view user-p11 -n project-cicd
$ oc adm policy add-role-to-user edit user-p12 -n project-cicd
$ oc adm policy add-role-to-user view user-p21 -n project-uat
$ oc adm policy add-role-to-user edit user-p22 -n project-uat

# Assgin Cluster Role to User for all projects
# https://docs.openshift.com/container-platform/4.3/authentication/using-rbac.html#default-roles_using-rbac
$ oc adm policy add-cluster-role-to-user edit user-a1
$ oc adm policy add-cluster-role-to-user view user-a2
$ oc adm policy add-cluster-role-to-user cluster-status user-s1
$ oc adm policy add-cluster-role-to-user cluster-admin user-s2
```