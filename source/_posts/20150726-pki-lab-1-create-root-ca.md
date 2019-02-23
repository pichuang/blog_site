---
layout: post
title: "PKI Lab$1 Create Root CA"
description: ""
date: 2015-07-26 00:00:00 +0800
updated: 2015-07-26 00:00:00 +0800
tags:
- infra
- easyrsa
- pki
---

採用 ```easyrsa``` 來建立 PKI CA

### Step by Step
- Setting vars for Root CA
> export KEY_SIZE=2048  
export CA_EXPIRE=3650  
export KEY_EXPIRE=3650  
export KEY_COUNTRY="TW"  
export KEY_PROVINCE="Taiwan"  
export KEY_CITY="HsinChu"  
export KEY_ORG="Night9 Studios"  
export KEY_EMAIL="root@night9.cc"  
export KEY_OU="www.night9.cc"  
export KEY_NAME=""  
export KEY_CN="Night9 Local Authority Root CA"  

  - [Source](https://github.com/pichuang/easy-rsa/blob/root_ca/vars)
  - 上面的寫法參考許多現有的 Root CA 的寫法, 特別是 ```KEY_CN KEY_NAME```
  - CA\_EXPIRE KEY\_EXPIRE 建議設長一點, 如果設太短, 只要 Root CA expire 底下的 key 就得全部重簽

- Create Root CA
> source ./vars  
./build-ca  
\<enter\>...\<enter\>  

- Check Root CA key and crt
> ls -la ./keys/ca.{key,crt}

- Check Root CA crt Info
> openssl x509 -in ca.crt -text -noout

- Show
<img src="https://lh3.googleusercontent.com/BqIPGP-VpmBI0_6TJNqwt2klqYSdOwvwn5DlSESKo_g=w1044-h1278-no" width="600" height="700">
  - 因為 Root CA 已經是最上層的, 沒人可以幫他驗證, 所以只能自己簽自己



### Reference
- [OpenSSL PKI Tutorial](http://pki-tutorial.readthedocs.org/en/latest/simple/)
- [Create a Public Key Infrastructure Using the easy-rsa Scripts](https://wiki.archlinux.org/index.php/Create_a_Public_Key_Infrastructure_Using_the_easy-rsa_Scripts)
- [作者：依瑪貓 http://www.imacat.idv.tw/tech/sslcerts.html.zh-tw](http://www.imacat.idv.tw/tech/sslcerts.html.zh-tw)
- [How To Setup a CA](http://pages.cs.wisc.edu/~zmiller/ca-howto/)
- [OpenSSL command line Root and Intermediate CA including OCSP, CRL and revocation](https://raymii.org/s/tutorials/OpenSSL_command_line_Root_and_Intermediate_CA_including_OCSP_CRL%20and_revocation.html)
