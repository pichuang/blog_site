---
layout: post
title: 愛的走馬看花 Red Hat CoreOS 與 Red Hat OpenShift Part 6
author: Phil Huang
toc: true
tags:
  - null
categories:
  - null
date: 2020-05-02 10:15:40
udpated: 2020-05-02 10:15:40
---

繼[上一篇][5]準備好了基礎的 OpenShift 4 超級管理員 `ocproot` 設定後，接下來就是比較針對 Project/Namespace 來進行比較細微的 RBAC 設定，盡可能讓 User 或 Group 適用於`最小權限原則`

![](/images/rbac-4.png)

<!--more-->

## 走馬看花之旅: 第六天
### 常見 OpenShift Project 權限指派

OpenShift 基本上會有 Users/Groups/Projects 可以進行管理，以下參考 [OpenShift 4之设置用户/组对项目的访问权限][2] 的文章來實作

針對專案 `project-cicd`、`project-uat` 之 RBAC 權限分配如圖示

![](/images/rbac-3.png)

- Users
  - Administrators
    - admin1/admin1
    - admin2/admin2
  - Developers
    - developer1/developer1
    - developer2/developer2
  - Testers
    - tester1/tester1
    - tester2/tester2
- Groups
  - admin-group: 對於 `project-cicd` 及 `project-uat`，具備最高權限
  - developer-group: 對於 `project-cicd` 專案具備設定權限，對 `project-uat` 僅有查看權限
  - tester-group: 對於 `project-uat` 專案具備設定權限，對 `project-cicd` 僅有查看權限
- Projects
  - project-cicd: 開發用環境
  - project-uat: UAT 環境

1. 如果沒存的話，撈出既有環境 `htpasswd` 檔案
```bash
$ oc get secret local-htpasswd-secret  -ojsonpath={.data.htpasswd} -n openshift-config | base64 -d | tee localuser-htpasswd
```

2. 新增 6 個好漢進 `localuser-htpasswd`
```bash
# 管理人員 admin1、admin2 皆具備 project-cicd、project-uat 最高權限
$ htpasswd -b localuser-htpasswd admin1 admin1
$ htpasswd -b localuser-htpasswd admin2 admin2

# 開發人員 developer1 設定單一專案 project-cicd 權限 且 檢視單一專案 project-uat 權限
$ htpasswd -b localuser-htpasswd developer1 developer1

# 開發人員 developer2 設定單一專案 project-cicd 權限 且 檢視單一專案 project-uat 權限
$ htpasswd -b localuser-htpasswd developer2 developer2

# 測試人員 tester1 設定單一專案 project-uat 權限 且 檢視單一專案 project-cicd 權限
$ htpasswd -b localuser-htpasswd tester1 tester1

# 測試人員 tester2 設定單一專案 project-uat 權限 且 檢視單一專案 project-cicd 權限
$ htpasswd -b localuser-htpasswd tester2 tester2
```

3. 檢查 localuser-htpasswd
```bash
# Show all user mapping
$ cat localuser-htpasswd
ocproot:$2y$05$pMTSySH/qa.gTcom9q/Pd.WHoI8VMIpCHWgc9d7cnQuIbRuuPeMia
developer1:$apr1$f7fpvHKb$X.MnMay0USf0.SOg/4hm6.
developer2:$apr1$FtL.4QEj$lQgfnYB42klWq5.BYnUxx0
tester1:$apr1$AuGDNNK/$U9Y0ytnT3qrakKat6jYkD1
tester2:$apr1$z1H./q7P$cvi62jxjmDOyMHQ/Gsx6R0
admin1:$apr1$tUyAimmD$uXc2my4xMGitt/hekx6SH/
admin2:$apr1$rbFALSqT$6emskSJsH8LD0ptM5eXlK1
```

4. 更新既有的 `local-htpasswd-secret`
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

如果想要查詢有什麼 Role 可以使用的話，可以用 `oc get clusterrole.rbac` 撈，相當多記得要 filter 一下

```bash
# Create Projects
$ oc new-project project-cicd
$ oc new-project project-uat

# Create groups
$ oc adm groups new admin-group
$ oc adm groups new developer-group
$ oc adm groups new tester-group

# Assgin Users and Groups
$ oc adm groups add-users admin-group admin1 admin2
group.user.openshift.io/admin-group added: ["admin1" "admin2"]

$ oc adm groups add-users developer-group developer1 developer2
group.user.openshift.io/developer-group added: ["developer1" "developer2"]

$ oc adm groups add-users tester-group tester1 tester2
group.user.openshift.io/tester-group added: ["tester1" "tester2"]

# Assgin Role to Group within Project
# For admin-group
$ oc adm policy add-role-to-group admin admin-group -n project-cicd
$ oc adm policy add-role-to-group admin admin-group -n project-uat

# For developer-group
$ oc adm policy add-role-to-group edit developer-group -n project-cicd
$ oc adm policy add-role-to-group view developer-group -n project-uat

# For tester-group
$ oc adm policy add-role-to-group edit tester-group -n project-cicd
$ oc adm policy add-role-to-group view tester-group -n project-uat
```

## Appendix

### 了解 RBAC 角色權限

![](/images/rbac-6.png)

依照 [OpenShift 預設 RBAC 角色定義][1]，我們新增以下帳號進去觀測，並且做個基本歸納
  - cluster-admin-user/cluster-admin
  - cluster-status-user/cluster-status
  - admin-user/admin
  - basic-user/basic-user
  - self-provisioner-user/self-provisioner
  - edit-user/edit
  - view-user/view



1. 新增帳號
```bash
# cluster-admin，類同於 Linux root 在 OpenShift 中的超級管理員角色
$ htpasswd -b localuser-htpasswd cluster-admin-user cluster-admin

# cluster-status，可以獲取基本的 Cluster 資訊
$ htpasswd -b localuser-htpasswd cluster-status-user cluster-status

# admin，專案管理者，可以查看修改專案中任何資訊
$ htpasswd -b localuser-htpasswd admin-user admin

# basic-user，可以檢視專案中任何資訊
$ htpasswd -b localuser-htpasswd basic-user basic-user

# self-provisioner，可以自己開專案
$ htpasswd -b localuser-htpasswd self-provisioner-user self-provisioner

# edit，可以修改資源，但不能看且修改 Role 綁定
$ htpasswd -b localuser-htpasswd edit-user edit

# view，只能檢視資源，但不能進行任何修改
$ htpasswd -b localuser-htpasswd view-user view
```

2. 更新既有的 `local-htpasswd-secret`
```bash
$ oc create secret generic local-htpasswd-secret --from-file htpasswd=/root/user/localuser-htpasswd --dry-run -o yaml -n openshift-config | oc replace -f -
secret/local-htpasswd-secret replaced
```

3. 各個顯示 User 狀態
```bash
# https://docs.openshift.com/container-platform/4.3/authentication/using-rbac.html#default-roles_using-rbac

# Cluster Level
# 常用於集群管理員，類同於 Linux root 的存在
$ oc adm policy add-cluster-role-to-user cluster-admin cluster-admin-user
$ oc describe clusterrole.rbac.authorization.k8s.io/cluster-admin

$ oc adm policy add-cluster-role-to-user cluster-status cluster-status-user
$ oc describe clusterrole.rbac cluster-status

# Project Level
# 常用於指定 Project 具備 administrator 權限
$ oc adm policy add-role-to-user admin admin-user
$ oc describe clusterrole.rbac admin

# 常用於指定 Project 具備 edit 權限，例如開發人員
$ oc adm policy add-role-to-user edit edit-user
$ oc describe clusterrole.rbac edit

# 常用於指定 Project 具備 view 權限，例如非開發，但具專案相關性之人員
$ oc adm policy add-role-to-user view view-user
$ oc describe clusterrole.rbac view

# Misc
# 遊客帳號
$ oc adm policy add-role-to-user basic-user basic-user
$ oc describe clusterrole.rbac basic-user

$ oc adm policy add-role-to-user self-provisioner self-provisioner-user
$ oc describe clusterrole.rbac self-provisioner
```

## 後話

依據 [Introduction To Ethical Hacking][6] 中的安全 (Security), 功能 (Functionality) 和方便 (Usability) 大三角

![](/images/rbac-5.png)

這個大三角是代表，若你是要系統，比較安全及具備完整功能，那就不可能會很方便使用，反之，如果要安全又要方便使用，那功能就不會多。而 OpenShift 看起來是比較偏向安全和功能這邊，所以不熟悉基本操作的話，會覺得有點玄，你有沒有覺得這個跟當年學 Linux 作業系統非常像呢？

## References
- [Using RBAC - OpenShift 4.3][1]
- [OpenShift 4之设置用户/组对项目的访问权限][2]
- [OpenShift 4之访问权限分级授权][3]
- [OpenShift 4 Hands-on Lab (9) 用户身份认证和资源访问限制][4]
- [愛的走馬看花 Red Hat CoreOS 與 Red Hat OpenShift Part 5][5]
- [Introduction To Ethical Hacking][6]


[1]: https://docs.openshift.com/container-platform/4.3/authentication/using-rbac.html#default-roles_using-rbac
[2]: https://blog.csdn.net/weixin_43902588/article/details/103412127
[3]: https://blog.csdn.net/weixin_43902588/article/details/103414273
[4]: https://blog.csdn.net/weixin_43902588/article/details/104470294
[5]: https://blog.pichuang.com.tw/20200427-openshift-with-coreos-part-5/
[6]: https://www.slideshare.net/chakrekevin/introduction-to-ethical-hacking-41597834