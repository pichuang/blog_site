---
layout: post
title: 愛的走馬看花 Red Hat CoreOS 與 Red Hat OpenShift Part 5
author: Phil Huang
toc: true
tags:
  - null
categories:
  - null
date: 2020-04-28 01:14:15
udpated: 2020-04-28 01:14:15
---

本日要來探討一下 OpenShift 4 內建的 RBAC (Role Based Access Control) 機制，

<!--more-->

## 走馬看花之旅: 第五天

預設剛裝完 OpenShift 4，應該都會拿到 3 個資訊

- Web Console: https://console-openshift-console.apps.ocp4.internal/dashboards
- Username: `kubeadmin`
- Password: `XXXX-XXXX-XXXX-XXXX`

但因為預設帳號導致的資安議題，其實這個預設的 `kubeadmin` 不建議持續使用，建議是換成別的帳號替換它既有的角色 `cluster-admin`，詳請可參考 [Removing the kubeadmin user - OpenShift 4.3][2]

### 準備 HTPasswd Identity Provider

預計將 `kubeadmin` 置換成 `ocproot`，權限也要一併移轉，將透過 `HTPasswd Identity Porvider` 的方式將帳號設定好

#### 設定 OAuth 資源

這邊先開啟一個檔案 `oauth.yaml`，新增 `identityProvider` 名字為 `local_htpasswd_provider`，形式為 `HTPasswd`，帳號密碼對照表放在 `local-htpasswd-secret` 裡面。

當中的 `local-htpasswd-secret` 將之後再新增，所以不用這時候先準備

```yaml oauth.yaml
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: local_htpasswd_provider
    challenge: true
    login: true
    mappingMethod: claim
    type: HTPasswd
    htpasswd:
      fileData:
        name: local-htpasswd-secret
```

```bash
# Update OAuth CRD
$ oc apply -f oauth.yaml
oauth.config.openshift.io/cluster configured

# Check
$ oc describe oauth cluster

# Update in the future if needed
$ oc get oauth cluster -o yaml > oauth.yaml
```

### 設定新最高權限帳號 - ocproot

1. 確保你有 `htpasswd` 可以使用

```bash
# Make sure you have htpasswd
$ sudo yum install httpd-tools -y
```

2. 新增 `local-htpasswd-secret` 檔案，新增 `ocproot/ocproot` 進去

```bash
# Create
$ htpasswd -c -B -b localuser-htpasswd ocproot ocproot
Adding password for user ocproot

$ cat localuser-htpasswd
ocproot:$2y$05$pMTSySH/qa.gTcom9q/Pd.WHoI8VMIpCHWgc9d7cnQuIbRuuPeMia

# Update if needed
# $ htpasswd -b localuser-htpasswd ocproot ocproot

# Remove if needed
# $ htpasswd -D localuser-htpasswd ocproot ocproot
```

3. 新增 `HTpasswd` 進去到 OpenShift 裡面

```bash
$ oc create secret generic local-htpasswd-secret --from-file htpasswd=/root/user/localuser-htpasswd -n openshift-config
secret/local-htpasswd-secret created
```

4. 新增叢集最高權限 `cluster-admin` 給 `ocproot`

```bash
$ oc adm policy add-cluster-role-to-user cluster-admin ocproot
clusterrole.rbac.authorization.k8s.io/cluster-admin added: "ocproot"
```

5. 測試 `ocproot`
```bash
$ oc login -u ocproot
oc login -u ocproot
Authentication required for https://api.ocp4.internal:6443 (openshift)
Username: ocproot
Password:
Login successful.

$ oc whoami
ocproot

$ oc get users
NAME      UID                                    FULL NAME   IDENTITIES
ocproot   3635b015-3664-4836-a5a0-2aff1100d99c               local_htpasswd_provider:ocproot
```

![](/images/ocproot.png)

#### 移除 `kubeadmin` 帳號 (Optional)

根據 [Removing the kubeadmin user - OpenShift 4.3][2] 來移除預設帳號 `kubeadmin`，務必要記得，你的集群裡面一定要有

- 一個以上的 `Identity Provider`，一般都是用 `HTpasswd`
- 一個以上的帳號具備 `cluster-admin` 的權限

不然你會登不進去集群 = =，這是不可逆的指令

```bash
# Remove it
$ oc delete secrets kubeadmin -n kube-system
```



## Appendix

### Q: 登入時，遇到 `error: x509: certificate signed by unknown authority`
```bash
$ oc login -u ocproot --insecure-skip-tls-verify=true
error: x509: certificate signed by unknown authority
```

這問題是因為自簽憑證，系統沒有安裝 ca 所以要特別拉出來安裝在系統或者是在你的電腦上

```bash
$ oc project openshift-authentication

# Same content
$ oc get pods
NAME                               READY   STATUS    RESTARTS   AGE
oauth-openshift-74fbdf999f-6kpnn   1/1     Running   0          10m
oauth-openshift-74fbdf999f-787cm   1/1     Running   0          10m

# Pick one and copy the cert to outside
$ oc rsh oauth-openshift-74fbdf999f-6kpnn cat /run/secrets/kubernetes.io/serviceaccount/ca.crt > ocp4-ingress-ca.crt

# It work on RHEL 7.7
$ cp ocp4-ingress-ca.crt /etc/pki/ca-trust/source/anchors/
$ update-ca-trust extract

# Verify Cert is working
$ openssl verify ocp4-ingress-ca.crt
ocp4-ingress-ca.crt: OK
```

### 切換回 `system:admin`

```bash
$ oc whoami
ocproot
$ oc config use-context admin
Switched to context "admin".
$ oc whoami
system:admin
```

## References

- [KB1519813: How to install a CA certificate on Red Hat Enterprise Linux 6 and later][1]
- [Removing the kubeadmin user - OpenShift 4.3][2]
- [OpenShift 4之设置用户/组对项目的访问权限][3]
- [OpenShift 4 之增加管理员用户][4]

[1]: https://access.redhat.com/solutions/1519813
[2]: https://docs.openshift.com/container-platform/4.3/authentication/remove-kubeadmin.html
[3]: https://blog.csdn.net/weixin_43902588/article/details/103412127
[4]: https://blog.csdn.net/weixin_43902588/article/details/103391438
