---
layout: post
title: NetDevOps 風格之網路設備連接方式
date: 2018-08-05 23:57:15
udpated: 2018-08-05 23:57:15
tags:
  - netdevops
  - ansible
categories:
  - ansible
toc: true
---

## NetDevOps = Networking + DevOps
近年來網路管理受到軟體定義網路 (SDN) 及 DevOps 的發展驅動了不小的變革，延伸至 - [JUNOS DEVOPS尤便捷 更精彩 - SDNLAB][13] 文中所帶出：快速服務是目的、自動化是手段、DevOps 則是思路或方法論，而 NetDevOps 這詞則是將既有 DevOps 的方法論套用在 Networking 領域中所產生的維運方法論。

<!--more-->

> 如何透過自動化的方式來管理更多的機器或網路設備到日常維運中?
如何借助 NetDevOps 方法論來降低人工出錯率，簡化流程，提升服務品質?
是不是一定會寫 Code 才能做到自動化?

以上的問題筆者經常被問到，其實要看不同的需求來回答，建議都是先把 DevOps 的思路先了解一下，再選用`對自己來說`正確的工具，都可滿足大多數業務需求，以下介紹幾種目前常見的網路設備連接方式

## 方式
### 手動命令列登入操作

身為一位網路工程師大多數登入設備內部的手段不外乎以下方式：
1. Serial Console
2. Telnet
3. SSH

這邊要特別提一下，於系統規劃中，Serial Console 的連線方式比較常見於 Day1 Configure 的階段，透過 Serial Console 連線進去後設定 IP 後改用 Telnet 或 SSH 的方式進行操作，當然因資安考量之下，多數都會優先選擇 SSH 為主的加密連線方式，而不採用明文傳輸的 Telnet。但在某些特殊情境架構下，Serial Console 是可以被視為 Day2 Operation 裡對設備長期控管的 Out-of-Band (OOB) 方案之一，可搭配 Serial Console Server 來進行管理。筆者以前有遇過除了主要管理網路設備透過 SSH 以外，有針對幾台 Core 設備另外佈 Console 專用網路，走另一條備援線路出來，避免網路炸掉時 SSH 登不進去。

使用時優點當然是對於許多網路工程師來說非常地熟悉，這是一切的基礎，下指令後馬上就能獲得回饋，但就長期管理層面來說並不是好事，常見於除錯時，下了太多的指令，可能跑到後面都忘記下了些什麼指令或更動了哪些檔案，造成設定飄移 (Configuration Drift)，而要反覆地敲打重複的指令檢查狀態及備份當前狀態，若有權限管理的人數更多時候，經常設定會漂移的更嚴重。

故 NetDevOps 概念有提倡一個概念：`管理設備時能盡可能做到 Zero Touch Provisioning`，也就是工程師`不直接`透過登入到設備做操作，而是透過某種已經寫好或訂好流程來進行管理，讓原先人工操作中間所有繁瑣的過程都盡可能地自動化，以確保操作一致性、可重複性和可預測性。

### expect 自動化命令列操作

這方式需要工程師會使用 `expect` 來完成，剛剛上述手動命令列登入操作，大多數都能透過這兩者的搭配做到最基本的自動化，尤其是對命令列操作吃重的網路設備，expect 可以`模擬`命令列輸入字元，但壞處就是中間如果想多新增寫條件判斷就會有以下的兩個範例的差異了，語法太過複雜不易維護，若要整合其他應用或被其他程式使用，將會大幅提高複雜度。故以個人使用及易維護為前提，最基本的建議還是以 Bash 搭 Telnet 及 SSH 撰寫比較恰當。

常見範例:
```bash
#!/usr/bin/expect
set timeout 5
spawn minicom
expect "password for xxx:"
send "password\r"
interact
```

多了條件判斷的[範例][1]
```bash
#!/usr/bin/expect
proc login {user pass} {
    expect "login:"
    send "$user\r"
    expect "password:"
    send "$pass\r"
}

set username spongebob 
set passwords {squarepants rhombuspants}
set index 0

spawn telnet 192.168.40.100
login $username [lindex $passwords $index]
expect {
    "login incorrect" {
        send_user "failed with $username:[lindex $passwords $index]\n"
        incr index
        if {$index == [llength $passwords]} {
            error "ran out of possible passwords"
        }
        login $username [lindex $passwords $index]
        exp_continue
    }
    "prompt>" 
}
send_user "success!\n"
```

### 具有 Python 魂的你

如果你是具備寫 Python 程式底子的工程師，目標想要走向高度客製化，這邊有幾個模組可以推薦使用，分別對應
1. Serial Console - [pySerial][2]
2. Telnet - [telnetlib][3]
3. SSH - [paramiko][4] 和基於 parmiko 開發的 [netmiko][6]
4. expect - [pexpect][7]
5. NETCONF - [ncclient][8]
6. RESTful - [requests][9]

日常管理中 SSH 會最常使用，所以建議要寫請先以 SSH 連線為主的 paramiko 做為模組基礎，往上開發自己專屬的管理程式，目前 paramiko 也被列為 [Ansible Connection Plugin][5] 之一。netmiko 則是基於 paramiko `針對網路設備操作`而生的專案，這專案有處理一些不同網路廠商所帶來的設備差異性，也可以列入參考選擇。

[`pySerial`][2] 模組可以讓你基於 Python 程式來操控 Serical Console 的行為，如果你是靠 Serial Console 維生的工程師或開發者，非常推薦使用。

範例:
```python
import serial
# 115200/8/N/1
ser=serial.Serial(port='/dev/tty.pichuang', baudrate=115200, bytesize=8, parity='N', stopbits=1, timeout=1)
```

是的，各位看倌 expect 也有出 Python 模組 - [pexpect][7] 來幫助你用 Python 去模擬命令列交互操作，可自行 Google 參閱使用方式。

雖然這些模組可以提供你高度客製化的能力，但理想很豐滿，現實很骨感，維運工程師價值是要`維護系統的穩定`，而不在開發新的工具來增加無謂的開發人力成本，這時候就是自動化工具的空間啦！

### 自動化配置管理工具 Ansible

對於大多數沒有寫程式經驗的網路工程師來說，透過寫程式的方式來管理網路基本是無疑找碴，但介於在手動跟高度客製化程式之中，有否快速地開始實行 NetDevOps 的方式呢? 筆者是推薦使用 Ansible 作為主要的使用工具

Ansible 目前是 Red Hat 底下的一個自動化維運工具平台，可以協助工程師對系統做配置管理 (Configuration Management)，配合版本控制 (推薦目前已成顯學的 Git)，以達到基礎設施即代碼 (Infrastructure as Code) 的目標。Ansible 語言基礎是使用 Python，但具體上不會寫相對複雜的 Python 也沒關係，因為主要是透過撰寫 YAML 文件來`描述` Ansible Playbook，搭配適當使用 Ansible Module 來快速做到自動化的工作。

Ansible 目前除了支持的模組除了既有透過 SSH 和 WinRM 所延伸出的一大堆[模組][10]以外，再來就是憑著最大優勢 `Agnetless` 的特性，在網路設備管理領域大放異彩。通常網路設備都不會讓使用者任意的塞 Agent，但透過各廠商的支持下，基於 Ansible 誕生出不少的[網路模組][11]供使用者做使用。然而大多數的網路設備都還是以 SSH 為主要通信手段，Ansible 主要的連線手段也是透過 SSH，基本上是非常容易兼容而不用更改現在對網路設備的管理方式，也可以配合基於 NETCONF 的協議來進行管理。

[Cisco IOS 範例][12]
```yaml
---
- hosts: ios_devices
  gather_facts: no
  connection: local
 
  vars_prompt:
  - name: "mgmt_username"
    prompt: "Username"
    private: no
  - name: "mgmt_password"
    prompt: "Password"

  tasks:

  - name: SYS | Define provider
    set_fact:
      provider:
        host: "{{ inventory_hostname }}" 
        username: "{{ mgmt_username }}"
        password: "{{ mgmt_password }}"

  - name: IOS | Show clock
    ios_command:
      provider: "{{ provider }}"
      commands:
        - show clock
    register: clock

  - debug: msg="{{ clock.stdout }}"
```

## Reference
- [Automating the Console using pySerial][2]
- [telnetlib - Telnet client][3]
- [paramiko - A Python implementation of SSHv2.][4]
- [Ansible Connection Plugin][5]
- [netmiko - Multi-vendor library to simplify Paramiko SSH connections to network devices][6]
- [pexpect][7]
- [ncclient][8]
- [Requests: HTTP for Humans][9]
- [Ansible Modules][10]
- [Ansible Network modules][11]
- [JUNOS DEVOPS尤便捷 更精彩 - SDNLAB][13]

[1]: https://stackoverflow.com/questions/1538444/using-conditional-statements-inside-expect?answertab=votes#tab-top
[2]: https://pynet.twb-tech.com/blog/automation/pyserial.html
[3]: https://docs.python.org/2/library/telnetlib.html#module-telnetlib
[4]: http://www.paramiko.org/
[5]: https://docs.ansible.com/ansible/2.6/plugins/connection.html
[6]: https://github.com/ktbyers/netmiko
[7]: https://pexpect.readthedocs.io/en/stable/
[8]: https://github.com/ncclient/ncclient
[9]: http://docs.python-requests.org/en/master/
[10]: https://docs.ansible.com/ansible/latest/modules/modules_by_category.html
[11]: https://docs.ansible.com/ansible/2.6/modules/list_of_network_modules.html#network-modules
[12]: https://github.com/brona/ansible-cisco-ios-example
[13]: https://www.sdnlab.com/18581.html