layout: post
title: Ansible 如何除錯? 誰可以救救我?
author: Phil Huang
tags:
  - ansible
  - troubleshooting
categories:
  - ansible
date: 2019-06-18 10:29:00
---
![](/images/ansible-logo.png)

可喜可賀，恭喜一下 Ansible 來到了 2.8 版號啦! 有興趣可以參考一下 [Release Note][7]

就個人經驗上，還蠻常遇到會寫 Ansible Playbook 的技術工程師，而他們的共通煩惱都是 Ansible 似乎疑難排解 (Trouble Shooting) 有些困難，常常遇到一個問題會看很久不知道怎麼下手開始除錯 (Debug)。本篇將分享一下筆者平時的作法，供廣大看倌們參考省下不少冤枉路。而如果你覺得我寫的看不太懂可以參考一下艦長的兩篇大作  [讓您安心執行-ansible-playbook-的小技巧-1][1] 、[讓您安心執行-ansible-playbook-的小技巧-2][2]

<!--more-->

## 經驗分享
### 改成工程師看得懂的除錯訊息：不打迷糊仗

預設狀況下，Ansible 輸出的格式是以 JSON 格式為主，然後通常輸出的時候都會如下圖顯示變成一坨不知道在寫什麼的訊息，十分難以判讀

![](/images/ansible-callback-debug-1.png)

但 [Ansible Callback Plugins][3] 有支援一個外掛模組叫做 [`debug`][4]，主要是協助把上面那一坨東西的 stdout 及 stderr 格式化 (Format) 成下圖，讓你清晰地了解錯誤訊息在寫什麼

![](/images/ansible-callback-debug-2.png)

開起來的方式很簡單，開啟 `ansible.cfg`，填入以下內容即可

```
[default]
...
stdout_callback = debug
```

### 愛用 debug 模組：其實就是多噴點資訊

若在 Ansible 執行過程中，你想要了解某一個變數裡面的值是什麼，可以使用 [`debug`][5] 模組來獲取運行中的 Ansible 執行變數資訊。要留意的事情是，這邊的 debug 是一個 Ansible 模組 (Modules)，上面章節所提起的則是 Callback 外掛當中一個模組，兩者不一樣。

通常 `debug` 模組都會搭配 [`register`][6] 一併使用，`register` 最主要功用是將單一 Task 結果輸出指派給 `register` 所定義好的變數，如下程式碼

```
---
- hosts: all
  tasks:
    - name: Test connectivity
      ping:
      register: result
      
    - name: Print result
      debug:
        msg: "{{ result }}"
```

可以很清楚的看到，第一個 Task 將會執行 `ping` 模組，而它產生出來的結果將會被指向 register 所定義的 `result` 變數裡面，而第二個 Task 將會執行 `debug` 模組，將裡面 `result` 的結果倒出來顯示在如下畫面

![](/images/ansible-modules-debug.png)


### 實際執行前先跑 Dry Run 看看：永保安康

蠻多工程師因為不熟 Ansible，很擔心會不會一執行錯誤的指令下去，就會導致災難這樣，所以 Ansible 提供一個檢查模式 [`dry-run`][8]，供工程師在正式執行前可以透過模擬指令的方式協助檢查一下當前執行的 Playbook 裡面打算要做什麼事

一般作法應該是在下 `ansible-playbook` 指令的後面多帶 `--check` 指令

```
# 起手式
$ ansible-playbook -i pichuang.yml --check

# 嫌資訊太少時
$ ansible-playbook -i pichuang.yml --check -v

# 想要一個一個 Tasks 慢慢看時
$ ansible-playbook -i pichuang.yml --step --check
```
![](/images/ansible-dry-run.png)

### Playbook Debugger：當你想要成為 Ansible 超高級寫手時...

在先前的社群分享中，筆者有多次提到目前把 Ansible Playbook 寫的第一無敵複雜的範例就是 [GitHub - openshift/openshift-ansible][9] 這個專案了，雖然整個安裝只需要跑兩三行指令，但只要變數寫錯或者是忘記寫，就要反覆跑好幾次，而跑一次大概就要花上好幾個半小時的光陰。

而先前為了抓 `openshift-ansible` 安裝時的其中一個很弔詭的問題，最後動用了 `debugger` 才找到一個神奇的字串問題。正常沒寫到這麼複雜的狀況下，其他上述所列的方式其實就很夠日常維運用了。

開啟 [Playbook Debugger][10] 的方式很簡單，開啟 `ansible.cfg`，填入以下內容即可

```
[default]
...
strategy = debug
```

開啟之後， debugger 會在遇到 `Task Failed` 的時候，自動切換到 debug mode，詳細操作可以參考一下 [Available Commands][11]

![](/images/ansible-debugger.png)

## 後言

Ansible 本質上是遵循三大要素：Simple、Agentless、Powerful，撰寫邏輯上是，能寫得簡單就盡量寫的單純，不要寫得太過複雜。依據不同的情境之下，相信大家都會將越來越多的情境考慮進去，以至於 Ansible Playbook 越來越大，不太容易進行基本除錯，故希望本篇能幫助到各位快速上手除錯技巧。而如果你對於其他 Ansible 撰寫技能有興趣的話，也可以參考一下去年拙作  [幾個小建議改善你的 Ansible 技能 - Phil Huang][12]。

## References
- [讓您安心執行-ansible-playbook-的小技巧-1 - Chengwei Chen][1]
- [讓您安心執行-ansible-playbook-的小技巧-2 - Chengwei Chen][2]
- [Ansible Callback Plugins List][3]
- [Ansible Callback Plugins - Debug][4]
- [Ansible Modules - Debug][5]
- [Ansible 2.8 Release Note][7]
- [GitHub - openshift/openshif-ansible][9]
- [Ansible Strategy - Playbook Debugger][10]
- [幾個小建議改善你的 Ansible 技能 - Phil Huang][12]

[1]: https://medium.com/laraveldojo/%E8%AE%93%E6%82%A8%E5%AE%89%E5%BF%83%E5%9F%B7%E8%A1%8C-ansible-playbook-%E7%9A%84%E5%B0%8F%E6%8A%80%E5%B7%A7-1-c634f2963bc9
[2]: https://medium.com/laraveldojo/%E8%AE%93%E6%82%A8%E5%AE%89%E5%BF%83%E5%9F%B7%E8%A1%8C-ansible-playbook-%E7%9A%84%E5%B0%8F%E6%8A%80%E5%B7%A7-2-856a60b19898
[3]: https://docs.ansible.com/ansible/latest/plugins/callback.html
[4]: https://docs.ansible.com/ansible/latest/plugins/callback/debug.html
[5]: https://docs.ansible.com/ansible/latest/modules/debug_module.html
[6]: https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html?highlight=register#registering-variables
[7]: https://github.com/ansible/ansible/blob/stable-2.8/changelogs/CHANGELOG-v2.8.rst
[8]: https://docs.ansible.com/ansible/2.8/user_guide/playbooks_checkmode.html
[9]: https://github.com/openshift/openshift-ansible
[10]: https://docs.ansible.com/ansible/latest/user_guide/playbooks_debugger.html
[11]: https://docs.ansible.com/ansible/latest/user_guide/playbooks_debugger.html#available-commands
[12]: https://blog.pichuang.com.tw/20180622-suggestions_to_improve_your_ansible_playbook/