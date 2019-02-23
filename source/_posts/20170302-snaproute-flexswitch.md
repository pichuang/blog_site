---
layout: post
title: SnapRoute FlexSwitch
date: 2017-03-02 01:40:34
updated: 2017-03-02 01:40:34
tags:
- sdn
- snaproute
- flexswitch
---

![SnapRoute Logo](https://1v51ha1y9z9f2hpq55t67p7a3u-wpengine.netdna-ssl.com/wp-content/uploads/2016/12/snaproute-logo-2016.jpg)

## Introduction to SnapRoute

這是 2016 年 6 月左右一群 Apple Data Center 工程師們跑出來開的網路新創公司名叫 SnapRoute，而他們開發的產品則叫 FlexSwitch。

<!--more-->

這專案已經開放 L2/L3 Protocol 相關的原始碼跟其架構貢獻給 [Open Comupter Project, OCP][2]，也把 Spec 公開在網路上 [FlexSwitch OCP Specification v0.1][3] 供大家參閱。當然若要理解 FlexSwitch 目前的架構，最準的還是以 [GitHub][4] 上的為主 

最近他們跟 Dell/EMC 及另一個 Linux Foundation 底下的專案 [OpenSwitch][17] 在進行一些[整合][16]，這兩個專案之所以可以整合的原因是，SnapRoute 其實本身只是個 Debian Package，目前是建立在 [Open Network Linux, ONL][18] 之上，而 OpenSwitch 則跟 ONL 地位相當，故 SnapRoute 理論上可以疊加在 OpenSwitch 之上運行，但 OpenSwitch 跟 SnapRoute L2/L3 功能互有重疊，不知道實際上要怎麼整，等後續他們的消息吧

[SnapRoute 成立的小趣事][9]

## Support Platform

依據 [Source code - reltools/pkgInfo.json][5]，目前 OpenSource 支援以下平台:
- [Accton/Edgecore AS5712-54X][6]
- [Accton/Edgecore Wedge40][7]
- [Accton/Edgecore Wedge100][8]
- Celestica Redstone
- Docker

商業版本請參考 [FlexSwitch - Hardware Compatibility List][21]

<!--more-->

## Installation Process

既然這專案是 OpenSource 想當然爾是可以自己動手編譯的，參考文件順序如下:

1. [Building FlexSwitch from source][10]

2. [GitHub - OpenSnaproute/vagrantFlexSwitchDev][11]
  - 如果不想用 Vagrant 的話，請自行解析 [Vagrantfile][12]

3. [GitHub - OpenSnaproute/reltools][13]
  - Main Project，執行 `fab setupDevEnv`
    - 如果遇到 rake last_comment 問題，起因是 [Apache Thrift][14] 專案的這行 [code][15]，遇到 rake version 大於 11.0.1 才會遇到，解法是請把 rake version 鎖定在 11.0
    - 注意 PATH 的內容，建議是 ~/.bashrc 跟 /root/.bashrc 裡的變數要一樣
    - 如果遇到一些 git clone 問題，username 請填 `OpenSnapRoute`

4. 編譯
```bash
python makePkg.py
```

5. Show Debian Packages
``` bash
pichuang@workstation-vm  ~/git/reltools (master*)
$ ls -a |grep deb
flexswitch_accton_wedge100-pichuang_1.0.0.171.44_amd64.deb
flexswitch_accton_wedge40-pichuang_1.0.0.171.44_amd64.deb
flexswitch_bcm_accton_as5712-pichuang_1.0.0.171.44_amd64.deb
flexswitch_cel_redstone-pichuang_1.0.0.171.44_amd64.deb
flexswitch_docker-pichuang_1.0.0.171.44_amd64.deb
```

6. Install Debian Package on AS5712-54
```bash
dpkg -i flexswitch_bcm_accton_as5712-pichuang_1.0.0.171.44_amd64.deb && apt install -f
```
  - Open Network Linux 版本建議 `2016.12.22.18.28.604af0c9b3dc9504870c30273ab22f2fb62746c3`

7. WEB UI
```
http://<ip>:8080/api-docs/#
```
![FlexSwitch Screen Shot](https://lh3.googleusercontent.com/5KzExo8JTIMKfZEnLtsCHcRphWCMzG4Tk5Bxhfo2swc3lElhuZJGnoVslycPlpV2IFvdYXmutnplvL00cKbG6xaL5MODPq2dmWztFghMhrPd4pXLrM0TotzGcyWjqhPLXsz7RZhf-0sIeUzIs010LdW0PI_8Q1S4848guBDjlHuxCYSn4DtNngVSLWFm43dZQJS4qhn3XKrVB6yGwjnOsAG_6Uc9u093k6511QYPRTT6sPW2Hnio-uXLrlDwnZ7utRDepXzMmyUwLmuZQ-S_dILTaHrOE63Yy_f9RAGswG6cYdFpb00GzrsSvOKaWqvqrrCL7VNENutA6PzXzl8w69fexVLJagDKaowrWIuaUFRkhG7UauHwO2FFXKj5uAWnCdXwjnno-H-mJXFc_fwX3IPxOzvXCBGJzCHIaStL7uTJS9Y7WRKebw283aHdJ_9aXoPOZGYIdj2qY6Y-wfBj9xqC8hFBRXqiwgqmKjDDgZpm5yWEdjeVsAP2WYYSeDbV1yK2pZs1xkVNLL4ETEqht_8ltmNiTHWPiXf5u8nsRsyN0vxen81_GCucfnegnLuX-AuXI2_-fte46NS1578ZNLt-t_ySaZ-dY1syhYisO75HGFhft0NGXg=w1948-h626-no)

More: [FlexSwitch API][19]

## Reference
- [SnapRoute][1]
- [Open Compute Project][2]
- [FlexSwitch OCP Specification v0.1][3]
- [GitHub - OpenSnapRoute][4]
- [List of Network Operating Systems][20]

## 社群服務

本人的文章大多都會發布在 [SDNDS-TW](https://www.facebook.com/groups/sdnds.tw/)。這社團文章都跟 SDN/NFV 國內外產業新聞或資訊有關，有任何技術問題也歡迎在上面詢問。
![SDNDS-TW Facebook Group](https://lh3.googleusercontent.com/KizynpieyTlykuyk-po1tgasnVI4Oxl9_vPXXpHYyj_JiUc0V5bRb8lHxGpZDQiTqMNCn_A6NqF-4Gx8KMex8XsjgdCGByIqtIpKBc4TvQoUtNtXf0RK_eBf2pwm8wNJRiWXfkz3vf6Tf465o2vzmH2G4iR28kf_Wc_ADeqDJDfQQlV0XTtQcSBBRG2N-zs1ue6dhg38lMoO47n0-SI6yl3x-HhyRIK8penHyHXe0i9q08IDqYEdtMGyKExcihESNthqc4r74kceAJUYhfqsFRpGLteEMXDkShC74r79Frr5aVwsYml0x1WVHkmXbxLZB381B0gkJWUXaoCeCC8rdeYtb6vn60vLsNYgYLRI0wT0VjEbJTCgcHUbNk8sBkXEiATLTKQiES5VcIEJ9nQ7FYTxsF8BiB7exUxr0b3kfTLVJF2RBqc0hpHojoiMh-nL-OY30a9rgf1vr4n-44sDGfa8-f7xWX3JEe-7RgNjwZ5glxGM1lkrf2_MHwbgVRNd2tzUrxmRNmMrVpFog-NcByDo824K8GJLxyLq0Dg12jkx90uOBKmhAE-_JUL5iIYzzUUKGJu1-ORReKmYR63UvTeUvwtxSEKUnpax4co6s6dQoS2f5bB37Q=w1652-h590-no)

[1]: https://snaproute.com/
[2]: http://opencompute.org/
[3]: https://www.circleb.eu/wp-content/uploads/2016/06/FlexSwitch_OCP_Specification_v0.1.pdf
[4]: https://github.com/OpenSnaproute
[5]: https://github.com/OpenSnaproute/reltools/blob/master/pkgInfo.json#L7
[6]: http://www.edge-core.com/productsInfo.php?cls=1&cls2=8&cls3=44&id=15
[7]: http://www.edge-core.com/productsInfo.php?cls=1&cls2=7&cls3=57&id=110
[8]: http://www.edge-core.com/productsInfo.php?cls=1&cls2=5&cls3=67&id=128
[9]: http://36kr.com/coop/yidian/post/5063241.html
[10]: https://opensnaproute.github.io/docs/build.html#building-flexswitch-from-source
[11]: https://github.com/OpenSnaproute/vagrantFlexSwitchDev
[12]: https://github.com/OpenSnaproute/vagrantFlexSwitchDev/blob/master/Vagrantfile#L67-L81
[13]: https://github.com/OpenSnaproute/reltools
[14]: https://github.com/OpenSnaproute/reltools/blob/master/fabfile.py#L212
[15]: https://github.com/apache/thrift/blob/master/lib/rb/thrift.gemspec#L35
[16]: https://www.linuxfoundation.org/announcements/snaproute-and-dell-emc-to-help-advance-linux-foundation%E2%80%99s-openswitch-project
[17]: http://www.openswitch.net/
[18]: https://opennetlinux.org/
[19]: https://drive.google.com/open?id=0B19ucAmtemIQYkZhX2FUYi13QTA
[20]: http://packetpushers.net/virtual-toolbox/list-network-operating-systems/
[21]: http://docs.snaproute.com/hcl/index.html