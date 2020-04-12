layout: post
title: 2020 大改造宅宅雲架構系列文-5 窮人版 PKI
author: Phil Huang
tags:
  - pki
  - ssl
  - linux
categories:
  - linux
date: 2020-02-03 16:11:00
toc: true
---

各位鄉親在家裡自建的服務，或多或少應該都有需要建立 https 的服務，估計大部分都是建好一份自簽憑證及金鑰 (self-signed cert/key) 之後，就丟到 TLS Server 端去，但因為我家宅宅雲的服務有點多，還有依據不同任務性質有做 Domain 分類，今天要來分享一下窮人版宅宅雲 PKI 建立大法，但我不是專門研究這領域的人，所以內容大部分都是實務操作面，如果想要看比較單純的基礎文章可以參考 Haway 哈維哥的 [[SSL 基礎]私有金鑰、CSR 、CRT 與 中繼憑證][7]

![](/images/homecloud-ca.png)

<!--more-->

## 開始之前...

本文將會完全依據 [OpenSSL PKI Tutorial v1.1][1] 的內容進行實踐，如果你沒有試過的人，超級建議自己玩一次看看，這份文件最有價值的地方在於，提供了三個情境啊啊啊啊啊

1. [Simple PKI][4]，包含 root CA + 1 signing CA，且提供 2 種不同簽發的方式 (Email / TLS Certificate)
2. [Advanced PKI][5]，包含 root CA + 3 signing CA (Email / TLS / Software)，且提供 3 種不同簽發的方式 (Email / TLS / Software)
3. [Expert PKI][6]，包含 root CA + 1 intermediate CA + 2 signing CA，且提供多種常見的簽發方式

我自己是以 Expert PKI 為主，簽出 root CA + 1 intermediate CA + 2 signing CA，倘若看官們是 HomeCloud 要用，簡單不複雜，大多都是 Web 服務的話，選擇 Simple PKI 的情境就好了


## 建立人生第一個 Root CA

這個我蠻建議選擇一個你不會常常碰到的資料夾放置好，然後簽出 Intermediate CA 之後，就可以把金鑰 (Key, .key) 藏起來到只有你知道的地方去，憑證 (Certification, .crt) 因為後面還會用到所以要留著放在外面給人串憑證信任用

### 下載 Expert PKI 模板
```bash
# I love example
git clone https://bitbucket.org/stefanholek/pki-example-3 homecloud
cd homecloud
```

### 配置 Root CA 的設定檔

這邊分享針對 `etc/root-ca.conf` 我修改的 patch 檔

```bash
git diff etc/root-ca.conf

diff --git a/etc/root-ca.conf b/etc/root-ca.conf
index 15d6126..a97d179 100644
--- a/etc/root-ca.conf
+++ b/etc/root-ca.conf
@@ -3,7 +3,7 @@
 [ default ]
 ca                      = root-ca               # CA name
 dir                     = .                     # Top dir
-base_url                = http://pki.blue.se    # CA base URL
+base_url                = http://pki.pichuang.local    # CA base URL
 aia_url                 = $base_url/$ca.cer     # CA certificate URL
 crl_url                 = $base_url/$ca.crl     # CRL distribution point
 name_opt                = multiline,-esc_msb,utf8 # Display UTF-8 characters
@@ -22,10 +22,10 @@ distinguished_name      = ca_dn                 # DN section
 req_extensions          = ca_reqext             # Desired extensions

 [ ca_dn ]
-countryName             = "SE"
-organizationName        = "Blue AB"
-organizationalUnitName  = "Blue Root CA"
-commonName              = "Blue Root CA"
+countryName             = "TW"
+organizationName        = "HomeCloud"
+organizationalUnitName  = "HomeCloud Root CA"
+commonName              = "HomeCloud Root CA"

 [ ca_reqext ]
 keyUsage                = critical,keyCertSign,cRLSign
@@ -90,7 +90,7 @@ subjectKeyIdentifier    = hash
 authorityKeyIdentifier  = keyid:always
 authorityInfoAccess     = @issuer_info
 crlDistributionPoints   = @crl_info
-certificatePolicies     = blueMediumAssurance,blueMediumDevice
+certificatePolicies     = HomeCloudMediumAssurance,HomeCloudMediumDevice

 [ crl_ext ]
 authorityKeyIdentifier  = keyid:always
@@ -108,5 +108,5 @@ URI.0                   = $crl_url
 oid_section             = additional_oids

 [ additional_oids ]
-blueMediumAssurance     = Blue Medium Assurance, 1.3.6.1.4.1.0.1.7.8
-blueMediumDevice        = Blue Medium Device Assurance, 1.3.6.1.4.1.0.1.7.9
+HomeCloudMediumAssurance     = HomeCloud Medium Assurance, 1.3.6.1.4.1.0.1.7.8
+HomeCloudMediumDevice        = HomeCloud Medium Device Assurance, 1.3.6.1.4.1.0.1.7.9
```

這邊要特別留意下列四個欄位

1. countryName             = "TW"
2. organizationName        = "HomeCloud"
3. organizationalUnitName  = "HomeCloud Root CA"
4. commonName              = "HomeCloud Root CA"

分別會對應到符合 X.509 標準之識別名稱 (Distinguished Name, DN) 格式: `DN: C=TW, O=HomeCloud, OU=HomeCloud Root CA, CN=HomeCloud Root CA`，詳細之縮寫表示可以參考 [IBM - Distinguished Names][2]

### 建立 Root CA 憑證/金鑰及自簽憑證 - HomeCloud Root CA

準備好一組密碼，建立 Csr 跟金鑰時會用上，另外美國國家標準技術研究所 (NIST) 於 2010 年認為 `2048 bits` 可以[撐到 2030 年][3]，所以對於過期日期直接設定為 `2030/12/31 23:59:59`。(小編: 可能那時候還沒發展出量子電腦...之後這個位元數跟過期時間推測會隨著量子計算的演進會大幅度縮小)

```bash
# Create directories
mkdir -p ca/root-ca/private ca/root-ca/db crl certs
chmod 700 ca/root-ca/private

# Create database 建立序列號
cp /dev/null ca/root-ca/db/root-ca.db
cp /dev/null ca/root-ca/db/root-ca.db.attr
echo 01 > ca/root-ca/db/root-ca.crt.srl
echo 01 > ca/root-ca/db/root-ca.crl.srl

# Create CA Request
# 密碼跟 root-cakey 都應該要妥善藏好
openssl req -new \
    -config etc/root-ca.conf \
    -out ca/root-ca.csr \
    -keyout ca/root-ca/private/root-ca.key

# Create CA Certificate
# 2048-bit RSA keys are deemed safe until 2030/12/31 23:59:59
openssl ca -selfsign \
    -config etc/root-ca.conf \
    -in ca/root-ca.csr \
    -out ca/root-ca.crt \
    -extensions root_ca_ext \
    -enddate 20301231235959Z

# Create initial CRL
# 建立憑證吊銷列表 (Certficiate revocation list)
openssl ca -gencrl \
    -config etc/root-ca.conf \
    -out crl/root-ca.crl
```

![](/images/root-ca.png)

假設你跑完了上面的流程，你應該可以獲得到下列 3 個東西
- 憑證: `ca/root-ca.crt`
- 金鑰: `ca/root-ca/private/root-ca.key` <- 拿去藏好
- 設定檔: `etc/root-ca.conf`


## 建立人生第一個 Intermidate CA

繼承上面的 `HomeCloud Root CA` 的產生結果，繼續簽 `HomeCloud Intermediate CA`，其實到這一步如果你不想要簽 Intermidate CA 的話，可以左轉 [Simple PKI][4] 或 [Advanced PKI][5] 繼續執行下去

### 配置 Intermidate CA 的設定檔

主要是從 `etc/network-ca.conf` 更改
```bash
cp etc/network-ca.conf etc/intermediate-ca.conf
git add etc/intermediate-ca.conf
```

另外分享修改的地方
```bash
git diff etc/intermediate-ca.conf

diff --git a/etc/intermediate-ca.conf b/etc/intermediate-ca.conf
index 6bda509..ca77f92 100644
--- a/etc/intermediate-ca.conf
+++ b/etc/intermediate-ca.conf
@@ -1,9 +1,9 @@
 # Blue Network CA

 [ default ]
-ca                      = network-ca            # CA name
+ca                      = intermediate-ca            # CA name
 dir                     = .                     # Top dir
-base_url                = http://pki.blue.se    # CA base URL
+base_url                = http://pki.pichuang.local    # CA base URL
 aia_url                 = $base_url/$ca.cer     # CA certificate URL
 crl_url                 = $base_url/$ca.crl     # CRL distribution point
 name_opt                = multiline,-esc_msb,utf8 # Display UTF-8 characters
@@ -22,10 +22,10 @@ distinguished_name      = ca_dn                 # DN section
 req_extensions          = ca_reqext             # Desired extensions

 [ ca_dn ]
-countryName             = "SE"
-organizationName        = "Blue AB"
-organizationalUnitName  = "Blue Network CA"
-commonName              = "Blue Network CA"
+countryName             = "TW"
+organizationName        = "HomeCloud"
+organizationalUnitName  = "HomeCloud Intermediate CA"
+commonName              = "HomeCloud Intermediate CA"

 [ ca_reqext ]
 keyUsage                = critical,keyCertSign,cRLSign
@@ -35,9 +35,9 @@ subjectKeyIdentifier    = hash
 # CA operational settings

 [ ca ]
-default_ca              = network_ca            # The default CA section
+default_ca              = intermediate_ca            # The default CA section

-[ network_ca ]
+[ intermediate_ca ]
 certificate             = $dir/ca/$ca.crt       # The CA cert
 private_key             = $dir/ca/$ca/private/$ca.key # CA private key
 new_certs_dir           = $dir/ca/$ca           # Certificate archive
@@ -84,7 +84,7 @@ subjectKeyIdentifier    = hash
 authorityKeyIdentifier  = keyid:always
 authorityInfoAccess     = @issuer_info
 crlDistributionPoints   = @crl_info
-certificatePolicies     = blueMediumAssurance,blueMediumDevice
+certificatePolicies     = HomeCloudMediumAssurance,HomeCloudMediumDevice

 [ signing_ca_ext ]
 keyUsage                = critical,keyCertSign,cRLSign
@@ -93,7 +93,7 @@ subjectKeyIdentifier    = hash
 authorityKeyIdentifier  = keyid:always
 authorityInfoAccess     = @issuer_info
 crlDistributionPoints   = @crl_info
-certificatePolicies     = blueMediumAssurance,blueMediumDevice
+certificatePolicies     = HomeCloudMediumAssurance,HomeCloudMediumDevice

 [ crl_ext ]
 authorityKeyIdentifier  = keyid:always
@@ -111,5 +111,5 @@ URI.0                   = $crl_url
 oid_section             = additional_oids

 [ additional_oids ]
-blueMediumAssurance     = Blue Medium Assurance, 1.3.6.1.4.1.0.1.7.8
-blueMediumDevice        = Blue Medium Device Assurance, 1.3.6.1.4.1.0.1.7.9
+HomeCloudMediumAssurance     = HomeCloud Medium Assurance, 1.3.6.1.4.1.0.1.7.8
+HomeCloudMediumDevice        = HomeCloud Medium Device Assurance, 1.3.6.1.4.1.0.1.7.9
```

### 建立 Intermidate CA 憑證/金鑰 - HomeCloud Intermidate CA
```bash
# Create directories
mkdir -p ca/intermediate-ca/private ca/intermediate-ca/db crl certs
chmod 700 ca/intermediate-ca/private

# Create database
cp /dev/null ca/intermediate-ca/db/intermediate-ca.db
cp /dev/null ca/intermediate-ca/db/intermediate-ca.db.attr
echo 01 > ca/intermediate-ca/db/intermediate-ca.crt.srl
echo 01 > ca/intermediate-ca/db/intermediate-ca.crl.srl

# Create CA request
# 角色: 下游 CA
# 一般流程是寄送 intermediate-ca.csr (Certificate Signing Request) 給上游 CA 做簽發，intermediate-ca.key 要保留著
openssl req -new \
    -config etc/intermediate-ca.conf \
    -out ca/intermediate-ca.csr \
    -keyout ca/intermediate-ca/private/intermediate-ca.key

# Create CA certificate
# 角色: 上游 CA
# 拿到下游 CA 的 intermediate-ca.csr 後，簽署出 intermediate-ca.crt 寄送回去給下游 CA
openssl ca \
    -config etc/root-ca.conf \
    -in ca/intermediate-ca.csr \
    -out ca/intermediate-ca.crt \
    -extensions intermediate_ca_ext \
    -enddate 20301231235959Z

# Create initial CRL
# 建立憑證吊銷列表 (Certficiate revocation list)
openssl ca -gencrl \
    -config etc/intermediate-ca.conf \
    -out crl/intermediate-ca.crl

# Create PEM bundle
# cat 下游 + 中游 + 上游 > ca-chain/ca-bundle
cat ca/intermediate-ca.crt ca/root-ca.crt > \
    ca/intermediate-ca-chain.pem
```

![](/images/intermediate-ca.png)

## 建立 Operation CA

### 配置 Operation CA 的設定檔

從 `etc/component-ca.conf` 修改而來

```bash
cp etc/component-ca.conf etc/operation-ca.conf
git add etc/operation-ca.conf
```

分享修改的地方
```bash
diff --git a/etc/operation-ca.conf b/etc/operation-ca.conf
index 8858261..480e83f 100644
--- a/etc/operation-ca.conf
+++ b/etc/operation-ca.conf
@@ -1,12 +1,12 @@
 # Blue Component CA

 [ default ]
-ca                      = component-ca          # CA name
+ca                      = operation-ca          # CA name
 dir                     = .                     # Top dir
-base_url                = http://pki.blue.se    # CA base URL
+base_url                = http://pki.pichuang.local    # CA base URL
 aia_url                 = $base_url/$ca.cer     # CA certificate URL
 crl_url                 = $base_url/$ca.crl     # CRL distribution point
-ocsp_url                = http://ocsp.blue.se   # OCSP responder URL
+ocsp_url                = http://ocsp.pichuang.local   # OCSP responder URL
 name_opt                = multiline,-esc_msb,utf8 # Display UTF-8 characters
 openssl_conf            = openssl_init          # Library config section

@@ -23,10 +23,10 @@ distinguished_name      = ca_dn                 # DN section
 req_extensions          = ca_reqext             # Desired extensions

 [ ca_dn ]
-countryName             = "SE"
-organizationName        = "Blue AB"
-organizationalUnitName  = "Blue Component CA"
-commonName              = "Blue Component CA"
+countryName             = "TW"
+organizationName        = "HomeCloud"
+organizationalUnitName  = "HomeCloud Operation CA"
+commonName              = "HomeCloud Operation CA"

 [ ca_reqext ]
 keyUsage                = critical,keyCertSign,cRLSign
@@ -36,9 +36,9 @@ subjectKeyIdentifier    = hash
 # CA operational settings

 [ ca ]
-default_ca              = component_ca          # The default CA section
+default_ca              = operation_ca          # The default CA section

-[ component_ca ]
+[ operation_ca ]
 certificate             = $dir/ca/$ca.crt       # The CA cert
 private_key             = $dir/ca/$ca/private/$ca.key # CA private key
 new_certs_dir           = $dir/ca/$ca           # Certificate archive
@@ -86,7 +86,7 @@ subjectKeyIdentifier    = hash
 authorityKeyIdentifier  = keyid:always
 authorityInfoAccess     = @ocsp_info
 crlDistributionPoints   = @crl_info
-certificatePolicies     = blueMediumDevice
+certificatePolicies     = HomeCloudMediumDevice

 [ client_ext ]
 keyUsage                = critical,digitalSignature
@@ -96,7 +96,7 @@ subjectKeyIdentifier    = hash
 authorityKeyIdentifier  = keyid:always
 authorityInfoAccess     = @ocsp_info
 crlDistributionPoints   = @crl_info
-certificatePolicies     = blueMediumDevice
+certificatePolicies     = HomeCloudMediumDevice

 [ timestamp_ext ]
 keyUsage                = critical,digitalSignature
@@ -106,7 +106,7 @@ subjectKeyIdentifier    = hash
 authorityKeyIdentifier  = keyid:always
 authorityInfoAccess     = @issuer_info
 crlDistributionPoints   = @crl_info
-certificatePolicies     = blueMediumDevice
+certificatePolicies     = HomeCloudMediumDevice

 [ ocspsign_ext ]
 keyUsage                = critical,digitalSignature
@@ -116,7 +116,7 @@ subjectKeyIdentifier    = hash
 authorityKeyIdentifier  = keyid:always
 authorityInfoAccess     = @issuer_info
 noCheck                 = null
-certificatePolicies     = blueMediumDevice
+certificatePolicies     = HomeCloudMediumDevice

 [ crl_ext ]
 authorityKeyIdentifier  = keyid:always
@@ -138,4 +138,4 @@ URI.0                   = $crl_url
 oid_section             = additional_oids

 [ additional_oids ]
-blueMediumDevice        = Blue Medium Device Assurance, 1.3.6.1.4.1.0.1.7.9
+HomeCloudMediumDevice        = HomeCloud Medium Device Assurance, 1.3.6.1.4.1.0.1.7.9
```

### 建立 Operation CA 憑證/金鑰 - HomeCloud Operation CA

```bash
# Create directories
mkdir -p ca/operation-ca/private ca/operation-ca/db crl certs
chmod 700 ca/operation-ca/private

# Create database
cp /dev/null ca/operation-ca/db/operation-ca.db
cp /dev/null ca/operation-ca/db/operation-ca.db.attr
echo 01 > ca/operation-ca/db/operation-ca.crt.srl
echo 01 > ca/operation-ca/db/operation-ca.crl.srl

# Create CA request
openssl req -new \
    -config etc/operation-ca.conf \
    -out ca/operation-ca.csr \
    -keyout ca/operation-ca/private/operation-ca.key

# Create CA certificate
openssl ca \
    -config etc/intermediate-ca.conf \
    -in ca/operation-ca.csr \
    -out ca/operation-ca.crt \
    -extensions signing_ca_ext

# Create initial CRL
openssl ca -gencrl \
    -config etc/operation-ca.conf \
    -out crl/operation-ca.crl

# Create PEM bundle
cat ca/operation-ca.crt ca/intermediate-ca.crt ca/root-ca.crt > \
    ca/operation-ca-chain.pem
```

![](/images/operation-ca.png)

## 維運相關

### 簽發單一 TLS Server 憑證
```bash
# Create TLS Server Request
SAN=DNS:www.pichuang.local \
openssl req -new \
    -config etc/server.conf \
    -out certs/www.pichuang.local.csr \
    -keyout certs/www.pichuang.local.key

# Create TLS Server Certificate
openssl ca \
    -config etc/operation-ca.conf \
    -in certs/www.pichuang.local.csr \
    -out certs/www.pichuang.local.crt \
    -extensions server_ext
```

### 簽發單一 Wildcard TLS Server 憑證
```bash
# Create TLS Server Request
SAN=DNS:*.pichuang.local \
openssl req -new \
    -config etc/server.conf \
    -out certs/wildcard.pichuang.local.csr \
    -keyout certs/wildcard.pichuang.local.key

# Create TLS Server Certificate
openssl ca \
    -config etc/operation-ca.conf \
    -in certs/wildcard.pichuang.local.csr \
    -out certs/wildcard.pichuang.local.crt \
    -extensions server_ext
```

### 簽發單一多 Wildcard TLS Server 憑證

![](sign-tls-cert-key.png)

```bash
# Create TLS Server Request
SAN=DNS:*.pichuang.local,DNS:*.misc.local \
openssl req -new \
    -config etc/server.conf \
    -out certs/wildcard.csr \
    -keyout certs/wildcard.key

# Create TLS Server Certificate
openssl ca \
    -config etc/operation-ca.conf \
    -in certs/wildcard.csr \
    -out certs/wildcard.crt \
    -extensions server_ext
```

## 驗證相關

### 驗證憑證 `wildcard.pichuang.local.crt` 受 CA Bundle `ca/operation-ca-chain.pem` 信任
```bash
# Should return OK
openssl verify -verbose -CAfile ca/operation-ca-chain.pem certs/wildcard.pichuang.local.crt
```

### 驗證 RHEL 可正常使用自簽憑證
```bash
# Install PEM into RHEL
cp ca/operation-ca-chain.pem /etc/pki/ca-trust/source/anchors/
update-ca-trust

# Should return OK
openssl verify certs/wildcard.pichuang.local.crt
```

### 檢查 CRT 憑證內容
```bash
openssl x509 -in certs/wildcard.pichuang.local.crt -text -noout
```

### 一般測試 TLS 服務
```bash
openssl s_client -CAfile ca/operation-ca-chain.pem -connect farm.pichuang.local:5001
openssl s_time -connect farm.pichuang.local:5001
```

## 後話

其實我想了想，應該我用 [Simple PKI][4] 的過程簽一簽就好了...


## References
- [OpenSSL PKI Tutorial v1.1][1]
- [[SSL 基礎]私有金鑰、CSR 、CRT 與 中繼憑證][7]


[1]: https://pki-tutorial.readthedocs.io/en/latest/
[2]: https://www.ibm.com/support/knowledgecenter/en/SSFKSJ_7.5.0/com.ibm.mq.sec.doc/q009860_.htm
[3]: https://www.informationsecurity.com.tw/article/article_print.aspx?aid=7511
[4]: https://pki-tutorial.readthedocs.io/en/latest/simple/index.html
[5]: https://pki-tutorial.readthedocs.io/en/latest/advanced/index.html
[6]: https://pki-tutorial.readthedocs.io/en/latest/expert/index.html
[7]: https://haway.30cm.gg/ssl-key-csr-crt-pem/