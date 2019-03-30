layout: post
title: 'Red Hat Ansible Tower 常見使用案例: 跨多平台設備管理'
author: Phil Huang
tags:
  - redhat
  - ansible
  - tower
  - iac
  - ztp
categories:
  - ansible
date: 2019-03-28 09:47:00
---
最近[強者我朋友 eanylin 哥][5]分享個整合的 Ansible Tower 實際整合案例，推薦用兩倍速看

- [Multi-Network Devices Configuration Management Using Ansible Tower][1]

{% youtube UHXNen1ZXq4 %}

這影片有個核心技術關鍵，就是`工作流 (Ansible Workflow)` 概念，而這裡面包含幾個角色...

<!--more-->

- 控制誰 Who => Inventory
- 做什麼 Do What => Ansible Workflow
- 怎麼做 How to do => Ansible Playbook


## Q1: 什麼是 Ansible Workflow?

假設你對於 Ansible 有一定的[了解][6]，你應該會知道一個正常能運作的腳本稱之為 Ansible Playbook 劇本

然而一本 Playbook 劇本通常裡面包含了:
- 主角 (Inventory)
- 劇情 (Tasks)
- 道具 (Modules)

但不會是一本劇本就可以演完全整齣戲 (不然我們幹嘛常要追劇/追番 XD)，一齣好的大戲，是要用結合多本 Playbook 劇本，分別做到承上啟下的作用

而將這些不同的劇本串再一起變成一齣大戲，在 Ansible 術語稱作 [Workflow 工作流][7]

## Q2: 怎麼那麼麻煩，阿不就依序執行 Ansible Playbook 下來就好了?

NoNoNo, 每一次 Playbook 劇本演完之後，會拋出三種狀態之一的情況出來:
1. On Succesful (綠線)
2. On Failure (紅線)
3. Always (藍線)

依據不同的狀態，可以讓你的工作流方向也能走的不太一樣，可以做到如下圖這樣的多個工作流

![](/images/workflow-1.jpg)


## Q3: 疑...等等 為什麼 Ansible 有 WEB 介面?

這是紅帽全球賣的嚇嚇叫的產品 `Red Hat Ansible Tower`

他的角色就類似中央指揮塔，負責存放大家的 Workflow / Playbook 和兼顧企業運作時該有的功能，還有支援自助表單 (Survey) 的功能，團隊要操作 Ansible Playbook/Workflow 都是集中登入到 Tower 上面做操作，按照這個做法下來...

- IaC, Infrastructure as Code
- ZTP, Zero Touch Provisioning
- DevOps: Automation / Measuring

通通都能在上面做到最佳實踐，搭一個版控系統 (GitHub/Gitlab/BitBucket)，簡直神鵰俠侶組合

意者請洽詢台灣紅帽 XD 

## Q4: 那我硬刻 Workflow 總可以吧?

技術上能做到，範例就是紅帽自家容器平台的安裝程式 [openshift/openshift-ansible][8]
這個等級是`用 Ansible 寫 Ansible` 了，用腦袋當 debugger 算變數傳遞位置 XD

舉其中一定會用到的 Playbook 為例 `prerequisites.yml`，截其中一小段 code 來看

```yaml
- name: Fail openshift_kubelet_name_override for new hosts
  hosts: "{{ l_scale_up_hosts | default('nodes') }}"
  tasks:
  - name: Fail when openshift_kubelet_name_override is defined
    fail:
      msg: "openshift_kubelet_name_override Cannot be defined for new hosts"
    when: openshift_kubelet_name_override is defined

- import_playbook: init/main.yml
  vars:
    l_install_base_packages: True
    l_repo_hosts: "{{ l_scale_up_hosts | default('oo_all_hosts') }}"

- import_playbook: init/validate_hostnames.yml
  when: not (skip_validate_hostnames | default(False))
```

試問：我要如何知道之前 playbook 跑完的狀態跟當下要怎麼喂參數及最後如何寫判斷 (condition)? 很遺憾地，寫到後面你會發現你在寫超複雜的程式，而不是專注你的真正的工作價值上

無論任何時候，撰寫或設計階段應該都要保持 KISS (Keep It Simple and Stupid) 原則
Ansible Playbook 能解耦 (拆成tasks) 就別聚合，而 Ansible Workflow 再做聚合的事情 (串串樂)

## Summary

Ansible 是一個極易學難精的 DSL，任何技術職位只要花點時間，基本上都能很快應用上，而老實說多數情況之下，完全也不必要學到像 [openshift/openshift-ansible][8] 這專案的奇淫技巧，那是有眾多歷史因素而成的

而我自己寫到現在，只需要能了解 `ansible-galaxy init` 所建出來的資料結構就夠了，也會用 `ansible -i hosts -m command -a oxxx` 就差不多可以開始應用了

![](/images/workflow-2.png)

什麼?? 你問我說 [`ansible-galaxy`][10] 是什麼東西? 欲知詳請，請待下回我心血來潮的時候再寫


## References
- [Multi-Network Devices Configuration Management Using Ansible Tower][1]
- [Ansible 技術概觀介紹_20190130][6]
- [GETTING STARTED: WORKFLOW JOB TEMPLATES][7]
- [GitHub - openshift/openshift-ansible][8]
- [Ansible Galaxy][10]

[1]: https://www.youtube.com/watch?v=UHXNen1ZXq4
[5]: https://www.youtube.com/channel/UCsAGOvR4jdcFQLyTFHnIJgQ/featured
[6]: https://speakerdeck.com/pichuang/ansible-ji-shu-gai-guan-jie-shao-20190130
[7]: https://www.ansible.com/blog/getting-started-workflow-job-templates
[8]: https://github.com/openshift/openshift-ansible/tree/release-3.11
[9]: https://github.com/openshift/openshift-ansible/blob/release-3.11/playbooks/prerequisites.yml
[10]: https://docs.ansible.com/ansible/latest/cli/ansible-galaxy.html