---
layout: post
title: 'SDN Lab 5$ REST and Ryu '
date: 2014-09-18 09:22
comments: true
categories: 
---
本次目標是要透過 REST 跟 Ryu 做一些 POST GET 來控制底下的 OpenvSwitch, 包含Query, 下 Flow entry...等等

首先要請大家安裝 [Chrome Extension - POSTMAN](https://chrome.google.com/webstore/detail/postman-rest-client/fdmmgilgnpjigdojojpjoooidkmcomcm) 來快速測試 REST API, 別再傻傻的用 curl 慢慢 key 指令了

<img class="right" src="https://lh6.googleusercontent.com/KBLgnDnUfeVkQ4VrJ0dWb2NArO14S4tE9xmFXfGocukGuJOJCelV23QbzTLHnWwDrGKDEAaCeQ=s128-h128-e365">

* POSTMAN 主要特色有
	* 可以透過 chrome 運行各種 HTTP request, 包含常見的 GET POST 及 REFTful 裡的 PUT DELETE
  * 純 Web GUI
  * 有 Eviroments, Collection 及可儲存送出的 request 內容
  * 回傳時有 Pretty JSON 可以看 (送出時沒有檢查機制, 要自己找其他網站做 JSON Validator)

然而 Ryu 的 REST 是透過 loading application 實現的, 詳細可以參考 Ryu 內建的 [App 清單](https://github.com/osrg/ryu/tree/master/ryu/app), 以下會舉例透過 POSTMAN 要到所有 switch stats 及如何下一條簡單 Flow entry

# 環境
- Ryu
  - OS: Ubuntu 14.04.1 Server LTS X86_64 
  - IP: 192.168.1.45
- Mininet
  - OS: Ubuntu 14.04.1 Server LTS X86_64
  - IP: 192.168.1.44
- My Computer
  - Browser: Google Chrome
	- IP: 192.168.1.11
 
# Ryu

* 安裝 Ryu
  * Ubuntu 使用者請參考 [John's Lin Blog - [筆記] Install Ryu 3.6 - SDN Framework](http://linton.tw/2014/02/15/note-install-ryu-36-sdn-framework/) 做安裝的動作, 本篇不再撰述
		* [SDNDS-TW 本月鉅獻 超懶人安裝 ryu ](https://github.com/sdnds-tw/ryuInstallHelper)
  * ArchLinux 使用者可使用 ```yaourt ryu-git```

* 啟動 Ryu
> ryu-manager --verbose rest.py simple_switch_13.py ofctl_rest.py rest_topology.py
  * 以上這四個除了 simple_switch_13.py 以外, 其他都有處理到 REST 的部分, 下面會挑一兩個來解釋如何看及用

# Mininet
* 安裝 Mininet
  * 參考 [SDN Lab 4$ Mininet Upgrade](http://roan.logdown.com/posts/230671-sdn-lab-4-mininet-upgrade)
  
* 啟動 Mininet
> \#!/bin/sh -ev  
\#Reference: http://www.routereflector.com/wp-content/uploads/2013/11/linear.png  
\#           http://www.routereflector.com/2013/11/mininet-as-an-sdn-test-platform/  
CONTROLLER_IP=192.168.1.45  
mn --topo linear,2 --mac --switch ovsk,protocols=OpenFlow13 --controller remote,$CONTROLLER_IP  

  * CONTROLLER_IP 請自行替換
  * topo template 可以參考 [MiniNet as an SDN test platform](http://www.routereflector.com/2013/11/mininet-as-an-sdn-test-platform/) 自行更改

# Use RESTful
## 如何查詢 all switches?

1. 找到 [rest_topology.py source code](https://github.com/osrg/ryu/blob/master/ryu/app/rest_topology.py)
  
2. 在 rest_topology.py 註解裡有詳細說明 REST API 該如何使用
  
3. 得知使用 get all the switches 即可獲得
> GET /v1.0/topology/switches
  
4. 填入 POSTMAN
<img class="center" src="https://lh4.googleusercontent.com/-fwmtC_TfVJA/VBoqYmB5ovI/AAAAAAAAFZ4/iMxrk-VBFnY/w1173-h483-no/rest-and-ryu-1.PNG" width="50%" height="50%">
  
5. Result
<img class="center" src="https://lh5.googleusercontent.com/-tPWeaLHvjZ8/VBoqaAvYbnI/AAAAAAAAFaA/38GS6Rk8dkE/w1172-h655-no/rest-and-ryu-2.PNG" alt="POSTMAN view" width="50%" height="50%">
<img class="center" src="https://lh3.googleusercontent.com/-hJGO4R-6DeE/VBoqaYEhS1I/AAAAAAAAFZ0/m1j2I2ftQQ4/w1025-h171-no/rest-and-ryu-3.PNG" alt="ryu controller view" width="50%" height="50%">
  
6. (Optional) 儲存 request
<img class="center" src="https://lh5.googleusercontent.com/-mvwJlPiwZNY/VBoqYjdr_8I/AAAAAAAAFZw/JiWrD8ntZHs/w1182-h594-no/postman1.PNG" width="50%" height="50%">
<img class="center" src="https://lh6.googleusercontent.com/-L7bCsactXdg/VBoq3Sg4ezI/AAAAAAAAFag/s8tvtpejwqk/w1174-h262-no/postman2.PNG" width="50%" height="50%">

## 如何下一條 Flow entry 至 OpenvSwitch?

1. 找到 [ofctl_rest.py source code](https://github.com/osrg/ryu/blob/master/ryu/app/ofctl_rest.py)
  
2. 得知使用 add a flow entry
> POST /stats/flowentry/add
  
3. 填入 POSTMAN
<img class="center" src="https://lh4.googleusercontent.com/7xK8PhpUt2cnWY8HI99yK-VhGSLFYtSwMgD2wg3xXJI=w1305-h497-no" width="50%" height="50%">
  
4. 填入 JSON 及送出, 200 OK
<img class="center" src="https://lh6.googleusercontent.com/-i6LmvLEBCRs/VBouMSx33sI/AAAAAAAAFa0/u0SYQhLVFtI/w1309-h629-no/rest-and-ryu-5.PNG" alt="correct" width="50%" height="50%">
	
  * 這邊要注意的是填入的 key-value, 都不得有錯, 有錯的話可能會有三種情況
  
	> 1. controller 回報 200 OK, 但 switch 是錯的
	> 2. controller 回報 400 Bad Request, 因為 raw data 有問題 
	> 3. controller 回報 500 Internal Server Error, 大部分都是OpenFlow格式錯誤
  
  * 送出的 raw data 並沒有進行檢查 JSON Format 的功能, 所以有可能會有少了一個 ```}``` 但你卻沒發現, 建議送出去之前先透過 [JSONLint](http://jsonlint.com/) 之類的 JSON Validator 做檢查
    
5. 因為少了一個 ```{```, 產生錯誤的 400 Bad Request 回應
<img class="center" src="https://lh5.googleusercontent.com/-8CheXhkd1ck/VBouMcU7TOI/AAAAAAAAFbA/vSZYmHK0roY/w1306-h616-no/rest-and-ryu-6.PNG" width="50%" height="50%">
  
6. 回報 200 OK 但 OpenvSwitch 卻出錯的範例
<img class="center" src="https://lh4.googleusercontent.com/-OCFb1xyx6Mg/VBoxiLvMQUI/AAAAAAAAFbU/ZFop2NMqQLI/w1309-h469-no/rest-and-ryu-7.PNG" width="50%" height="50%">
<img class="center" src="https://lh4.googleusercontent.com/PdDQU9IObXQNqJexJIuP_9AQtXfI0TshMwJDT332FgU=w1027-h218-no" width="50%" height="50%">



# Reference
- [Postman - 測試 API 的好工具](http://blog.roachking.net/blog/2012/11/07/postman-restful-client/)
- [qmaw的部落格](http://qmaw.pixnet.net/blog/category/4690154)
- [SDNDS-TW HACKPAD](https://sdnds-tw.hackpad.com/)
