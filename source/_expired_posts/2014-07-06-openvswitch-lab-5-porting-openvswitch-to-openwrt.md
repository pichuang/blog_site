---
layout: post
title: 'OpenvSwitch Lab 5$ Porting OpenvSwitch to OpenWrt'
date: 2014-07-06 21:57
comments: true
categories: 
---
* 因 OpenvSwitch 已經回到 [OpenWrt Packages Repository](https://github.com/openwrt/packages/tree/master/net/openvswitch) 故此篇不再更新

市面上常見的 OpenFlow switch (Pica8, HP...etc) 都非常的昂貴, 對於一般的窮苦研究生來講是一筆不可能支出的開銷, 那我們要如何建立一個簡單便宜又有AP功能的實體設備呢? 答案是 ```可以刷 OpenWrt 的 AP``` [可刷OpenWrt詳細列表](http://wiki.openwrt.org/toh/start) 但因為這種ap本身不是專門再處理這種計算的, 所以效率上會比較差一點, 但功能上還是能用的.

如果有興趣想刷的人, 可以先參考 [編譯OpenWrt](http://roan.logdown.com/posts/165911-compiled-openwrt) 準備好環境

<img class="center" src="https://lh6.googleusercontent.com/-Ix65c7GZIWc/U-2oTcwL4VI/AAAAAAAAFKs/HVAIJYkWdFY/w622-h425-no/Capture.PNG">

# 設備
- D-Link DIR-835
- Ubuntu 14.04.1 x86_64
- OpenvSwitch 2.3.0


# 安裝 OpenvWrt
1. 下載 OpenWrt trunk source
> git clone git://git.openwrt.org/openwrt.git && cd openwrt

2. Add feeds
> mv feeds.conf.default feeds.conf  
echo 'src-git openvswitch git://github.com/pichuang/openvwrt.git' >> feeds.conf  

3. Update feeds
> ./scripts/feeds update -a  
./scripts/feeds install -a  

4. 打上 libatomic patch
> wget https://gist.githubusercontent.com/pichuang/7372af6d5d3bd1db5a88/raw/4e2290e3e184288de7623c02f63fb57c536e035a/openwrt-add-libatomic.patch -q -O - | patch -p1

5. 設定 profile
> make menuconfig
	* 選擇 ```Network -> openvswitch-switch, openvswitch-switch, openvswitch-ipsec (Optional)```
  * 選擇 ```Advanced configuration options (for developers) -> Toolchain Options -> Binutils Version -> Linaro binutils 2.24```
  * 取消選擇 ```Advanced configuration options (for developers) -> Target Options -> Build packages with MIPS16 instructions```

6. 取消 bridge
> echo '#CONFIG_KERNEL_BRIDGE is not set' >> .config
  * 這是因為 ovs 替代了 linux bridge, 所以要特別取消掉
	* 要注意, 只要每下一次 ```make menuconfig``` 這步都要重做一次, 因為 ```make menuconfig``` 會蓋掉 .config 裡面的設定, 而 CONFIG_KERNEL_BRIDGE 並不再這設定檔裡面, 它是在 ```make kernelconfig``` 被定義的.

7. 編譯 image
> make V=s

基本上走完上面七步後, AP 就能支援 OpenFlow 了. 剩下的大部分都是跟 OpenvSwitch 設定或 OpenWrt 設定相關, 這部分再未來的某天寫出來給大家參考, 祝研究順利.

# Reference
- [OpenvSwitch-Openwrt - pichuang GitHub](https://github.com/pichuang/openvwrt)
- [編譯 OpenWrt](http://roan.logdown.com/posts/165911-compiled-openwrt)
- [設定 OpenvSwitch](http://roan.logdown.com/posts/191801-set-openvswitch)

