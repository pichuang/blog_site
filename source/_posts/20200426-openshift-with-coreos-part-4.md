---
layout: post
title: 愛的走馬看花 Red Hat CoreOS 與 Red Hat OpenShift Part 4
author: Phil Huang
toc: true
tags:
  - openshift4
  - openshift
categories:
  - openshift
date: 2020-04-26 02:54:49
udpated: 2020-04-26 02:54:49
---

## 走馬看花之旅: 第四天

承襲上篇 [愛的走馬看花 Red Hat CoreOS 與 Red Hat OpenShift Part 3][4] 精神，繼續寫一點東西，但因本週太忙了，只講一個大家都很關注的 Kubernetes 套件管理程式 - `Helm 3` 內容

![](/images/helm3-ocp4.png)

<!--more-->

### 使用 Helm 3

> The package manager for Kubernetes

Red Hat OpenShift 4 終於支援了 [Helm 3][5]，所以目前共有 2 種套件管理方式可以做選擇使用:

1. Operator Framework
2. Helm 3

兩者的能力差異如下

|  | Helm 3 | Operator Framework |
|----------------------|------|--------------------|
| `套件化` | Y | Y |
| `App 安裝能力` | Y | Y |
| `App 更新能力` | Y | Y |
| App 自動備份和還原 | X | Y |
| 工作負載分析和紀錄 | X | Y |
| App 層級自動化擴縮容 | X | Y |
| 自動最佳化 | X | Y |

至於要用哪一個...嘿嘿 要看你的角色是什麼，這邊就不提太多了

#### 安裝 Helm 3

因為 helm 3 之後移除了原先 v2 的 Tiller 角色，單純僅使用 helm 來進行操作，所以安裝過程其實相當簡單，不需要碰到 OpenShift 就可以搞定，詳細可以參考 [CNCF發布Kubernetes應用程式管理工具Helm 3 - IThome][1] 一文

```bash bastion.ocp4.internal
$ wget https://mirror.openshift.com/pub/openshift-v4/clients/helm/3.1.1/helm-linux-amd64
$ chmod +x helm-linux-amd64
$ mv helm-linux-amd64 /usr/local/bin/helm

$ helm version
version.BuildInfo{Version:"v3.1+unreleased", GitCommit:"7ebdbb86fca32c77f2fce166f7f9e58ebf7e9946", GitTreeState:"clean", GoVersion:"go1.13.4"}
```

#### 使用外部匯入 helm charts

當你有網路可以連的時候，這個是蠻常見的用法，可以參考一下使用過程

```bash
# 使用 Helm Stable Charts
$ helm repo add helm-stable-charts https://kubernetes-charts.storage.googleapis.com/
"helm-stable-charts" has been added to your repositories

# 使用 Helm Incubartor Charts
$ helm repo add helm-incubator-charts https://kubernetes-charts-incubator.storage.googleapis.com
"helm-incubator-charts" has been added to your repositories

# Update charts from these repos
$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "helm-incubator-charts" chart repository
...Successfully got an update from the "helm-stable-charts" chart repository
Update Complete. ⎈ Happy Helming!⎈

# Search charts
# helm search repo <name>
$ helm search repo mysql
NAME                                            CHART VERSION   APP VERSION   DESCRIPTION
helm-incubator-charts/mysqlha                   2.0.0           5.7.13        MySQL cluster with a single master and zero or ...
helm-stable-charts/mysql                        1.6.3           5.7.28        Fast, reliable, scalable, and easy to use open-...
helm-stable-charts/mysqldump                    2.6.0           2.4.1         A Helm chart to help backup MySQL databases usi...
helm-stable-charts/prometheus-mysql-exporter    0.5.2           v0.11.0       A Helm chart for prometheus mysql exporter with...
helm-stable-charts/percona                      1.2.1           5.7.26        free, fully compatible, enhanced, open source d...
helm-stable-charts/percona-xtradb-cluster       1.0.3           5.7.19        free, fully compatible, enhanced, open source d...
helm-stable-charts/phpmyadmin                   4.3.5           5.0.1         DEPRECATED phpMyAdmin is an mysql administratio...
helm-stable-charts/gcloud-sqlproxy              0.6.1           1.11          DEPRECATED Google Cloud SQL Proxy
helm-stable-charts/mariadb                      7.3.14          10.3.22       DEPRECATED Fast, reliable, scalable, and easy t...

# Inspect mysql
# helm inspect all|chart|values|readme <name>
$ helm inspect chart helm-stable-charts/mysql
apiVersion: v1
appVersion: 5.7.28
description: Fast, reliable, scalable, and easy to use open-source relational database
  system.
home: https://www.mysql.com/
icon: https://www.mysql.com/common/logos/logo-mysql-170x115.png
keywords:
- mysql
- database
- sql
maintainers:
- email: o.with@sportradar.com
  name: olemarkus
- email: viglesias@google.com
  name: viglesiasce
name: mysql
sources:
- https://github.com/kubernetes/charts
- https://github.com/docker-library/mysql
version: 1.6.3

# Create a new project - test-mariadb
$ oc new-project test-mysql
Now using project "test-mysql" on server "https://api.ocp4.internal:6443".

# Install charts - mysql
$ helm install mysql-dev helm-stable-charts/mysql
NAME: mysql-dev
LAST DEPLOYED: Sun Apr 26 18:27:10 2020
NAMESPACE: test-mysql
STATUS: deployed
REVISION: 1
NOTES:
MySQL can be accessed via port 3306 on the following DNS name from within your cluster:
mysql-dev.test-mysql.svc.cluster.local

To get your root password run:

    MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace test-mysql mysql-dev -o jsonpath="{.data.mysql-root-password}" | base64 --decode; echo)

To connect to your database:

1. Run an Ubuntu pod that you can use as a client:

    kubectl run -i --tty ubuntu --image=ubuntu:16.04 --restart=Never -- bash -il

2. Install the mysql client:

    $ apt-get update && apt-get install mysql-client -y

3. Connect using the mysql cli, then provide your password:
    $ mysql -h mysql-dev -p

To connect to your database directly from outside the K8s cluster:
    MYSQL_HOST=127.0.0.1
    MYSQL_PORT=3306

    # Execute the following command to route the connection:
    kubectl port-forward svc/mysql-dev 3306

    mysql -h ${MYSQL_HOST} -P${MYSQL_PORT} -u root -p${MYSQL_ROOT_PASSWORD}

# Check Charts list
$ helm list
NAME            NAMESPACE       REVISION        UPDATED                                       STATUS          CHART           APP VERSION
mysql-dev       test-mysql      1               2020-04-26 18:27:10.979294793 +0800 CST       deployed        mysql-1.6.3     5.7.28

# Check Mysql server is ready
$ oc get all
NAME                            READY   STATUS    RESTARTS   AGE
pod/mysql-dev-d8b597f5f-b2nxn   1/1     Running   0          31s

NAME                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/mysql-dev   ClusterIP   172.30.4.152   <none>        3306/TCP   31s

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mysql-dev   1/1     1            1           31s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/mysql-dev-d8b597f5f   1         1         1       31s

# Uninstall charts and project
$ helm uninstall mysql-dev
release "mysql-dev" uninstalled

$ oc delete project test-mysql
project.project.openshift.io "test-mysql" deleted
```

#### 使用自行下載 helm charts

現實是，一票環境都不給連網路，所以自然也沒有什麼 `helm repo` 可以用，所以要大部分應該都會改用這個作法，整體核心還是在如何離線操作 `GitOps` 及使用離線 `Container Registry` 身上

```bash
# Example: IBM/helm101
$ git clone https://github.com/IBM/helm101

# Create a new project - test-mariadb
$ oc new-project test-my-first-helm-chart
Now using project "test-my-first-helm-chart" on server "https://api.ocp4.internal:6443".

# Install guestbook using helm 3
$ cd helm101/charts
$ helm install guestbook-demo ./guestbook/ --namespace test-my-first-helm-chart

# Check Charts list
$ helm list
NAME            NAMESPACE                       REVISION        UPDATED                                       STATUS          CHART                 APP VERSION
guestbook-demo  test-my-first-helm-chart        1               2020-04-26 18:45:34.084900528 +0800 CST       deployed        guestbook-0.2.0

# Check Guestbook server is ready
$ oc get all
NAME                                  READY   STATUS    RESTARTS   AGE
pod/guestbook-demo-67f5b45d45-kscnt   1/1     Running   0          2m39s
pod/guestbook-demo-67f5b45d45-tv4dr   1/1     Running   0          2m39s
pod/redis-master-68857cd57c-7m585     1/1     Running   0          2m39s
pod/redis-slave-bbd8d8545-6nwh5       1/1     Running   0          2m39s
pod/redis-slave-bbd8d8545-xgkmb       1/1     Running   0          2m39s

NAME                     TYPE           CLUSTER-IP       EXTERNAL-IP
 PORT(S)          AGE
service/guestbook-demo   LoadBalancer   172.30.216.24    <pending>
 3000:32765/TCP   2m39s
service/redis-master     ClusterIP      172.30.64.237    <none>
 6379/TCP         2m39s
service/redis-slave      ClusterIP      172.30.143.221   <none>
 6379/TCP         2m39s

NAME                             READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/guestbook-demo   2/2     2            2           2m39s
deployment.apps/redis-master     1/1     1            1           2m39s
deployment.apps/redis-slave      2/2     2            2           2m39s

NAME                                        DESIRED   CURRENT   READY
  AGE
replicaset.apps/guestbook-demo-67f5b45d45   2         2         2
  2m39s
replicaset.apps/redis-master-68857cd57c     1         1         1
  2m39s
replicaset.apps/redis-slave-bbd8d8545       2         2         2
  2m39s

# Uninstall charts and project
$ helm uninstall guestbook-demo
release "guestbook-demo" uninstalled

$ oc delete project test-my-first-helm-chart
project.project.openshift.io "test-my-first-helm-chart" deleted
```

#### 環境資訊

- Red Hat OpenShift 4.3.13 (Kubernetes v1.16.2)
- Red Hat Enterprise Linux 7.7 as bastion server
- helm v3.1+unreleased

## 結語

我覺得 Red Hat 官方開始支援是對的，剛好 Helm 3 後對底層進行了大改造，原先的資安問題除了移除 Tiller 以外，也同時透過 OpenShift 預設相較嚴苛 RBAC 來做到比較好的保護，算是一個各得其所的使用方式

## References
- [CNCF發布Kubernetes應用程式管理工具Helm 3 - IThome][1]
- [helm/charts - GitHub][2]
- [IBM/helm101 - GitHub][3]
- [愛的走馬看花 Red Hat CoreOS 與 Red Hat OpenShift Part 3][4]
- [helm][5]

[1]: https://www.ithome.com.tw/news/134233
[2]: https://github.com/helm/charts
[3]: https://github.com/IBM/helm101
[4]: https://blog.pichuang.com.tw/20200412-openshift-with-coreos-part-3/
[5]: https://helm.sh/