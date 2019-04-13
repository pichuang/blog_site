layout: post
title: 新增 SSL/TLS 憑證和金鑰
author: Phil Huang
tags:
  - openssl
  - rhel
categories:
  - rhel
date: 2018-11-20 14:13:00
---
## 動機
大多數的時候服務都會架設在不用對開放到 Internet 上的地方去給外部使用者接觸，也就是所謂的 Intranet。那如果想要在 Intranet 建立自建簡單的 PKI (Public Key Infrastructure) 的話，該如何做呢?

<!--more-->

## 實際做法
### SSL/TLS Self-Signed Certificate and Key

若想要一行產生出 Self-Signed Certificate 和 Key 可使用下列指令:

```
openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout server.key -out server.crt
```

最重要的是 Common Name (CN) 的內容要寫對
![](/images/ssl-1.png)

### SSL/TLS Signed by Intermediate Certificate and Key

通常建立一個僅限 Intranet 使用的 Root & Intermediate CA 不外乎有以下的事情要處理：
1. 建立 Root CA 及 Intermediate CA
2. 透過 Intermediate CA 簽出一張 Server Cert/Key 或 Wildcard domain Cert/Key
3. 將 Root CA Cert 匯入到使用系統 (e.g. Windows/MAC/Linux)上，並設定 All Trusted

這以上的步驟可以用 `openssl` 的指令配合一大票參數可以將每一步拆解出來，詳細可以參考 [Certificate Authority - Roll Your Own Network][1] 及 [OpenSSL PKI Tutorial][2] 兩篇文章自行研究，但簽出來可能因為參數沒下好或是下錯導致憑證無效，故這邊提供一個比較簡單的方式，採用 Google 非官方支援的 [easypki][3] 專案


```bash
# Get the CLI
# 別忘了要將 ~/go/bin 放到 PATH 裡
go get github.com/google/easypki/cmd/easypki

# 設置參數
export PKI_ROOT=/tmp/pki
export PKI_ORGANIZATION="HomeLab Inc."
export PKI_ORGANIZATIONAL_UNIT=Home
export PKI_COUNTRY=TW
export PKI_LOCALITY="Taipei City"
export PKI_PROVINCE="Taipei"

# 新建 Root CA
easypki create --filename root --ca "HomeLab Inc. Root Certificate Authority" --expire 3650

# 新建 intermediate CA
easypki create --ca-name root --filename intermediate --intermediate "HomeLab Inc. Intermediate CA" --expire 730

# 新建一張 Wildcard certificate，且透過 intermediate ca 簽核
easypki create --ca-name intermediate --dns "*.pichuang.local" "*.pichuang.local" --expire 365

# (Optional) 新建一張 Server certidicate 包含 quay.pichuang.local，且透過 intermediate ca 簽核
easypki create --ca-name intermediate --dns quay.pichuang.local quay.pichuang.local --expire 365

# (Optional) 新建一張 Server certidicate 包含 quay.pichuang.local 和 www.pichuang.local，且透過 intermediate ca 簽核
easypki create --ca-name intermediate --dns quay.pichuang.local --dns www.pichuang.local quay.pichuang.local --expire 365
```

當你透過以上步驟建立完之後，你應該會需要三個檔案做使用
1. `/tmp/pki/root/certs/root.crt` 給 Client 端系統認證用
2. `/tmp/pki/intermediate/certs/wildcard.pichuang.local.crt` 給 Server side 使用或給 Client 端系統認證用
3. `/tmp/pki/intermediate/keys/wildcard.pichuang.local.key` 給 Server side 使用

## 僅供參考用的 SSL Certificate 等級
因應各家的系統規模都不盡相同，所切分的 Class 可能也會有不同
大體上可分為
1. Class 1 for individuals, intended for email.
2. Class 2 for organizations, for which proof of identity is required.
3. Class 3 for servers and software signing, for which independent 4. verification and checking of identity and authority is done by the issuing certificate authority.
4. Class 4 for online business transactions between companies.
5. Class 5 for private organizations or governmental security.

## 於 RHEL7 系統裡匯入 TLS/SSL 憑證
```bash
cp *.crt /etc/pki/ca-trust/source/anchors/
update-ca-trust enable
update-ca-trust extract
```


## Reference
- [Certificate Authority - Roll Your Own Network][1]
- [OpenSSL PKI Tutorial][2]
- [GitHub - google/easypki][3]
- [Public key certificate][4]

[1]: https://roll.urown.net/ca/index.html
[2]: https://pki-tutorial.readthedocs.io/en/latest/simple/
[3]: https://github.com/google/easypki
[4]: https://en.wikipedia.org/wiki/Public_key_certificate