---
layout: post
title: "PKI Lab$3 Create Single domain SSL Certificate"
description: ""
date: 2015-07-27 00:00:00 +0800
updated: 2015-07-27 00:00:00 +0800
tags:
  - infra
  - easyrsa
  - pki
categories:
  - linux
---

採用 ```easyrsa``` 來建立 PKI CA, 必需要有 Root CA, 可參考 [PKI Lab$1 Create Root CA](http://blog.pichuang.com.tw/pki-lab-1-create-root-ca/) 做建立的動作

### Step by Step
- Setting vars for Root CA
> export KEY_SIZE=2048  
export CA_EXPIRE=365  
export KEY_EXPIRE=365  
export KEY_COUNTRY="TW"  
export KEY_PROVINCE="Taiwan"  
export KEY_CITY="HsinChu"  
export KEY_ORG="Night9 Studios"  
export KEY_EMAIL="roan@night9.cc"  
export KEY_OU="www.night9.cc"  
export KEY_NAME=""  
export KEY_CN="roan.night9.cc"  

  - [Source](https://github.com/pichuang/easy-rsa/blob/single/vars)
  - 重點在於 ```KEY_CN```, 此設定可以 match ```roan.night9.cc```, 但不能 match ```night9.cc``` ```xxx.night9.cc```

- Create Key
> source ./vars  
./build-key-server roan.night9.cc  
\<enter\>...\<enter\>  

- Check key and crt
> ls -la ./keys/roan.night9.cc.{key,crt}

- Check roan.night9.cc crt Info
> openssl x509 -in roan.night9.crt -text -noout

- Show
<img src="https://lh3.googleusercontent.com/jEjmgkHRcXctCtCis_yvaNSvPZpAa7Sp4eG6oVaRUSQ=w1034-h1278-no" width="600" height="700">

### Reference
- [OpenSSL PKI Tutorial](http://pki-tutorial.readthedocs.org/en/latest/simple/)
- [How To Setup a CA](http://pages.cs.wisc.edu/~zmiller/ca-howto/)
- [OpenSSL command line Root and Intermediate CA including OCSP, CRL and revocation](https://raymii.org/s/tutorials/OpenSSL_command_line_Root_and_Intermediate_CA_including_OCSP_CRL%20and_revocation.html)
