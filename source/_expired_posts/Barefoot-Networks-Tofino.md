---
layout: post
title: Barefoot Networks Tofino
date: 2017-03-05 17:28:01
updated: 2017-03-05 17:28:01
category: sdn
tags:
- sdn
- barefoot
- tofino
- p4 
---

![Barefoot Networks Logo](https://upload.wikimedia.org/wikipedia/en/b/b9/Barefoot_Networks_Logo.png)

## Introduction to Barefoot Networks

[Barefoot Networks][1] 本身是一家新創公司，創辦人之一有在 SDN 領域非常知名的 [Nick McKeown][10]，主要成立目的是想要讓 Network 可程式化跟 CPU 可程式化一樣簡單，所以他們針對整個生態系推出了:

* 語言 [`P4`][3]
* 晶片 `Tofino`
  - 6.5Tb/s (65 x 100GbE or 260 x 25GbE) and 3.2Tb/s fully user-programmable switch
  - 原生支援 `P4`
* IDE `Capilano`
  - 整合 P4 編譯器及除錯器的 IDE
  - P4 模擬器
  - 整合七家不同的 Network OS 包含 OpenSwitch (SnapRoute/Dell EMC), SONiC (Microsoft Azure), and FBOSS (Facebook).

極度推薦閱讀 [The World's Fastest & Most Programmable Networks - Barefoot Networks whitepaper][11]

## Support Platform
根據新聞指出 [Barefoot Lands Edgecore and WNC for its Ethernet Switching Chip][7]，兩家都是台灣的 ODM 廠商，支援的平台如下:
- Accton/Edgecore Wedge100BF-32X
  - 32x100GbE
  - Facebook OCP Wedge series
- Accton/Edgecore Wedge100BF-65X
  - 65x100GbE
  - Facebook OCP Wedge series
- WNC OSW1800
  - 48x25GbE + 6x100GbE
- WNC OSW6500
  - 65x100GbE

<!--more-->

## Network Field Day 14 - Barefoot Networks - Barefoot Networks ScreenShot

在 [Network Field Day](http://techfieldday.com/) 推出的影片是近期針對 Barefoot Networks 最詳細的介紹，強烈推薦每一個影片都從頭到尾看完，尤其是 `Barefoot Networks Tofino P4 Demo with Roberto Mari`

### How will Network engineers & administrator use Tofino

{% img https://lh3.googleusercontent.com/f9JainI_A2qym0-ckQTU0cQe6QUG2XB9tNt-b6FSyWY0hiQaq4e7Gat4LwxpHfdKv8E_CrKTFmS4hCEloU4vmWnzjJi-DcGrnzPgyTnHlDx9DtceOfgmBTxJ-oy8DylmfhoiPdxEj1muqh8cxkY31Y0DumryH--I_vIqSmqKecJfPSz-pUMNI0_1BIdfhgkYV34NRPAbk3nl-dJxjCwLCazSOeuI1Bl6xShkuP2ODgsk6gohiURfUwAu17_StFXOn86DZOwRlwo85jg4yU9bHy6E1u7mjUpMzf0cy8j0T1g5jeHGdi-AtA43eF9WnO1R9za12oPkZAJK4JsfyXPOXI-FvVjTU7qt985KjbEOqs0N6ViAgYq0oRx-d3dPWY9YOiiKUBjp21NbyI_7Ud-mIvv1857yoZOnsh8mhnEJl10rpfuxV9M5rqTKm00Q31KYnxVkdahjBeVWXeWlacAwjuoLXoLxihJCjDMCWCfOLmrXqYpMw1rnlRr9fmNOVqZEB5Jo3VTNnfX8ivITg2J6o6IxvCRh5kyKqe5d8QH1V_Y0QgjumcXjLp2xL6kKeB_wndl_lYjy8u7SpKUcXzTdvIaqQey8hjRAblozJvU0umDkmNerIHRwbQ=w1286-h724-no 436 659 How will Network engineers & administrator use Tofino%}

### Domain Specific Processors

{% img https://lh3.googleusercontent.com/np0lEYCZtsPFtH2gUtqew3WIxzz0iiKttGdshMW3O1bQdyWXVmKGfSN_aEGcPpchU5mL7xitOu2sUFQfV5MQwvm6I_VVLyTJO3y02A4Zhm2nXPm4_as3IDc_OpXRz9O2nQ7ecpA4rjJbDewQTQTXemqNHSdbqXH5Ilz-inGS2DNiUiQ0n9T_ZV9dU8vakuvkxcSod_AIyqlWvxxTZ4Es58er_Z0ygadm6gJru5qT9D7ZaQkjAHBa8RE_oIP7FTjcXWvj9SFjKPtcE7Q5M0FcRIqfjQsrHPJOuEhGy8bZtw0nK58IPET5O4FKE8yqqIvdHOD2wjlsyaulyzr-0Q_zElrz1L64ioihGnS9ukyJbuZCbD0ydIbrsPHsAh4YHhiEzJfAo1mHrM8l3RS41CFOZ1ludd2b_TJpIq-kwdY_71_AIip5o0vbosJbFh_KZtf2wIaQ4mwGkWnBCdd_VxuhiVG1n88ZsLvNnvsszs-HY7F9BTDzBj0cWWmrtEpCXD_fH8lIROKuSMI9wLgcP5KKsR1Tivn9dWyJeky9eMzqyHboWjdqgQwz1amV0RXbQNvurqgO3G142b20BwUWvlB8vXCP9811eEM4cBr7hgKMyy1rIigRkKEykEHWMygLT1MIZK02DmwowPo=w445-h252-no 436 659 Why Does the Internet Need a Programmable Forwarding Plane with Nick McKeown https://vimeo.com/200192012
 %}

看圖說故事，P4 經過 Capilano 編譯後才能放到符合 PISA 架構的 Tofino 晶片上執行

### Protocol Independent Switch Architecture, PISA

{% img https://lh3.googleusercontent.com/4bfcVFhC8tAyURGK_IaT7M_6Hejp_QdLnnmDUDYSv3fnh_T2UyBdZCirDlP__qO-P5h1gbfL_0Zd_SblTPYWbYcvGFEhZFKvAUKJX_gQbFRdy7Ok8LIBofXLkTE85kGF90OkfioPrsorsgtLiWMxmrvcMldkQmwadEFybEjgRNXwqMED4BioIYorDGSsGDRfHqGL7G9TRlGKZgO5otY2KhZzidO7LKFYSBVQEdSA4s5tLcTCBF8SAMuayT4Ija2Ts4lIcqmLgnFGtpMBkXaP8vCKnZqs5tRBtPmRDi44YoKzsbLt0sA_ITpysEMJkqXhpScMOZvQkpkFQvc3zmJBVVWlBgPbjKdvfDk9w0jAkqN9gqJKTqfGQ6w2-qG8LiS_v9C3sefz1T6OtddhK44CSG1mg40rJvQQaCW473kbaenlG4g2BwWQmzHD_yu7fA54c7kHhqBGuiMiDU96Ny2H6WYWzeSD28y19dXJZRMKS5BIG3cxVRLkzkEIZoYB-qcNihex_Z0lUIBF2BFjTCJ0Um8Wp1sw6aLALL9W_TGmEROy3CSIq1eDbJ66qk-xh8om2fcgK_AcowHt37gpD88EU5SZmvoGSBuzl2bW9cm4ZlG5gfhjD6duu3AYXTXOoJafxn0xF8hoCYE=w514-h296-no 436 659
PISA: Protocol Independent Switch Architecture %}

[Programming the Forwarding Plane - Nick McKeown][12] 太高深我沒看懂 XD，No comment

### Use Case
{% img https://lh3.googleusercontent.com/qyjUp9d7d87kBhfKu_fOBGKSKyqYyrtC1Ajhk1R8mYEbvN7yA3Rp_D-0MU-Ob8p8b1Vw4sRhu0HMTOZoWkCi9FNB3gOZy40i3Z1KscEjwoT5mMXizrH5Xf3flfDcQaeuvfDTmIUvxBjnzWDr6TRFkCM6N3Wf_j5D7GjTIM32RM4DnXS1GzTtGZemn8i9lGhhSZFnu3Fq8hz99bHVsCa22Y9kc8k1MqWpCaqzCLPGceIXKw43rSR-UOhezTvRZ4rDTldsxZuANdIolqCQzf5y8Vv7yQ7uXwU65eAvZAdlJTwBS0tjm7iaQRtBAfHYa87Ybks0kSrOE2zqF5tZw78-cEYAJogmyj3vaNpOh4U9jgh5YZ_7ZHNyrHoFyqHQs7o1oCN64y5rHLEgYZ6v2FL32Y_-zXrHmIpCbgM4ntjW78Joq-B-QuhtPMdLWbtyZK3aqX_RHH3A2F4FXTgFyXMNMr2hAWw-xxAVBIiBLUbPZ2ppHYPtrpxO4cB7GLw0Xy6G0f6biwlaFWQcXS6bWjbvvjWWSZn9aA9Zg13dUghZWbKoIGBr3BiKc-8Ixzu_1SBtrodY43UYfGiWvjDZwcyPk6l13LYWCT9U5VaxdD6_YLEgSdD10D3njQ=w1282-h734-no 436 659 Use Case%}

針對企業，個人認為 Firewall 跟 Load Balacing 的應用比較符合性價比。畢竟一台機器就可以完全撐到 6.5Tb/s，還具備可程式化的功能和 on-the-fly 立即更新，程式寫得好的話，還可以先擋掉一堆不需要使用的協定。

## 實作相關

- [P语言基础与实战][13]
- [P4 bmv2 debugger 使用方式 - P4TW.org][14]
- [P4:开创数据平面可编程时代][15]

## Reference
- [Barefoot Networks Official Website][1]
- [Network Field Day 14–Barefoot Networks - Barefoot Networks][2]
- [P4.Org Official Website][3]
- [P4 Taiwan Website][4]
- [阿里巴巴、腾讯投资Barefoot，助力C轮融资8000万美元 - SDNLAB][5]
- [Barefoot Networks进军白盒市场 -SDNLAB][6]
- [Barefoot Lands Edgecore and WNC for its Ethernet Switching Chip - Converge Network Digest][7]
- [第一個原生支援 P4 的晶片 – Tofino][8]
- [用P4对数据平面进行编程][9]
- [The World's Fastest & Most Programmable Networks - Barefoot Networks whitepaper][11]

## 社群服務

本人的文章大多都會發布在 [SDNDS-TW](https://www.facebook.com/groups/sdnds.tw/)。這社團文章都跟 SDN/NFV 國內外產業新聞或資訊有關，有任何技術問題也歡迎在上面詢問。
![SDNDS-TW Facebook Group](https://lh3.googleusercontent.com/KizynpieyTlykuyk-po1tgasnVI4Oxl9_vPXXpHYyj_JiUc0V5bRb8lHxGpZDQiTqMNCn_A6NqF-4Gx8KMex8XsjgdCGByIqtIpKBc4TvQoUtNtXf0RK_eBf2pwm8wNJRiWXfkz3vf6Tf465o2vzmH2G4iR28kf_Wc_ADeqDJDfQQlV0XTtQcSBBRG2N-zs1ue6dhg38lMoO47n0-SI6yl3x-HhyRIK8penHyHXe0i9q08IDqYEdtMGyKExcihESNthqc4r74kceAJUYhfqsFRpGLteEMXDkShC74r79Frr5aVwsYml0x1WVHkmXbxLZB381B0gkJWUXaoCeCC8rdeYtb6vn60vLsNYgYLRI0wT0VjEbJTCgcHUbNk8sBkXEiATLTKQiES5VcIEJ9nQ7FYTxsF8BiB7exUxr0b3kfTLVJF2RBqc0hpHojoiMh-nL-OY30a9rgf1vr4n-44sDGfa8-f7xWX3JEe-7RgNjwZ5glxGM1lkrf2_MHwbgVRNd2tzUrxmRNmMrVpFog-NcByDo824K8GJLxyLq0Dg12jkx90uOBKmhAE-_JUL5iIYzzUUKGJu1-ORReKmYR63UvTeUvwtxSEKUnpax4co6s6dQoS2f5bB37Q=w1652-h590-no)

[1]: https://www.barefootnetworks.com/
[2]: http://techfieldday.com/appearance/barefoot-networks-presents-at-networking-field-day-14/
[3]: http://p4.org/
[4]: https://p4tw.org/
[5]: http://www.sdnlab.com/18111.html
[6]: http://www.sdnlab.com/18413.html
[7]: http://www.convergedigest.com/2017/01/barefoot-networks-lands-edgecore-and.html
[8]: https://p4tw.org/%E7%AC%AC%E4%B8%80%E5%80%8B%E5%8E%9F%E7%94%9F%E6%94%AF%E6%8F%B4-p4-%E7%9A%84%E6%99%B6%E7%89%87-tofino/
[9]: http://yuba.stanford.edu/~nickm/papers/cccf.pdf
[10]: http://yuba.stanford.edu/~nickm/
[11]: https://barefootnetworks.com/white-paper/the-worlds-fastest-most-programmable-networks/
[12]: https://forum.stanford.edu/events/2016/slides/plenary/Nick.pdf
[13]: https://fast-switch.github.io/wiki/%E6%9D%A8%E5%B8%85-P4%E8%AF%AD%E8%A8%80%E5%9F%BA%E7%A1%80%E4%B8%8E%E5%AE%9E%E6%88%98.pdf
[14]: https://p4tw.org/p4-bmv2-debugger-%E4%BD%BF%E7%94%A8%E6%96%B9%E5%BC%8F/
[15]: http://www.sdnlab.com/17795.html
