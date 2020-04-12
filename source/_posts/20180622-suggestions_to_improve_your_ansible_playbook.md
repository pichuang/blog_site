---
layout: post
title: "幾個小建議改善你的 Ansible 技能"
description: ""
date: 2018-06-22 00:00:00 +0800
updated: 2018-06-22 00:00:00 +0800
tags:
  - automation
  - ansible
  - iac
categories:
  - ansible
toc: true
---

# 前言

先說本文謹代表個人撰寫 Ansible Playbook 及 Ansible Tower 之經驗累積，不定期會更新內容，僅供參考 XD

Ansible 本身具備著幾個很顯著的特性: `好寫`, `易讀`, `Agentless`，但說到好寫，也因為寫的花樣特別多，所以 特別撰寫此文留下平時我寫 Ansible Playbook 時會注意的一些地方。當然有些內容會從 Ansible 官方所列的 [Best Practices][1] 截錄過來，但多數還是由我個人撰寫經驗的角度為出發點。

<!--more-->

## Suggestion 1: 建議遵從最佳實踐的 Ansible Direcory Layout 最佳建議

若有常在撰寫 Anisble 的人都知道，若改 `ansible.cfg` 可以讓很多 playbook 呼叫變數及路徑變得非常的彈性。可以改，但十分不建議這樣做。主要是你同事或跟你交接的工程師不一定會知道你的個人習慣是什麼，建議走最佳實踐及建議之資料夾命名

```yaml
production_inventory         # production servers 主機目錄
staging_inventory            # staging environment 主機目錄

group_vars/
   group1                 # 以 Group 為單位，賦予對應之變數
   group2                 
host_vars/
   hostname1              # 以 host 為單位，賦予對應之變數
   hostname2              

library/                  # 任何自己寫或自己維護 (也就是非 Built-in) 的 module 都建議放這邊
module_utils/             # 個人很少在用
                          # 可參考 `/usr/lib/python2.7/site-packages/ansible/module_utils` 的內容
filter_plugins/           # 任何自己寫或自己維護 (也就是非 Built-in) 的filter 都建議放這邊

Do_something_for_someone_1.yml       # 為了某個服務做某事，後面會有詳細建議
Do_something_for_someone_2.yml
Do_something_for_someone_3.yml

roles/
    common/               # 其中一個 role 的名字 `common`
        tasks/            # common 的 tasks 資料夾
            main.yml      # common 相關的 tasks action 請先寫在這
        handlers/         # common 裡的事件處理
            main.yml      # common 相關的 event handler 請先寫在這
        templates/        # common 相關的 jinja2 模板
            ntp.conf.j2   # common 相關的 file template 請先寫在這
        files/            # common 相關的檔案庫
            bar.txt       # common 相關的檔案
            foo.sh        # common 相關的 script 放這邊
        vars/             #
            main.yml      # 跟這個 common 有相關的區域變數宣告請放這邊
        defaults/         # common 相關的預設變數
            main.yml      # 可以寫變數預設值在這邊，但我個人比較推薦寫到 tasks 裡面的 playbook 裡，後面會有詳細建議和寫法
        meta/             # common 相關的其他角色依賴
            main.yml      # 宣告角色依賴
        library/          # 針對 common role 的函式庫
        module_utils/     # 個人很少在用
        lookup_plugins/   # 個人很少在用

    webtier/              # 呈上的結構
        ...
        ...
        ...
```

## Suggestion 2: 建議 Playbook 命名法

`Do something for someone` 為命名遵照，我知道很多人都是直接用 someone 當作 Playbook 的命名，但因為後面會帶入的其中一個建議 Ansible Tower 所帶來 Workflow 的觀念 - `多個 Playbook 序列化執行` 及期望維護者能一眼看出這份 Playbook 主要是在針對`什麼目標做什麼事情`，所以建議此規範。

```yaml
#不建議
webserver.yml
database.yml

#建議
deploy-webserver.yml
deploy-database.yml
```

## Suggestion 3: 一個專案一個業務目標

Ansible 最頂端的資料夾叫做 `專案 (Project)`，其次才是 Playbook 及 roles 等等的文件結構。

假設今天要部署一個 http server，需要有 lb、www、db，我會將 Playbook 分為以下幾個：

- Deploy_http_server
    - install_lb.yml
    - install_www.yml
    - install_db.yml
    - check_www_service.yml

以此案例，專案名字叫做 Deploy_http_server，裡面包含上述所列的 playbook，以`一個專案一個業務目標`為核心，進行 playbook 的撰寫。

## Suggestion 4: 變數無給值，預設帶入 default 值

常寫 Shell Script 的人應該常寫以下寫法：

```bash
FOO=${VARIABLE:-default} # 如果 VARIABLE 無給值，及使用 default 值
```

其實 ansible 也有支援這寫法

```bash
---
- hosts: "{{ variable_host | default('all') }}"
  tasks:
    - name: ...
    ...
```

這個使用情境我個人是應用在部署新 VM 或更新系統時，針對`特定主機`時會用到，過去要針對新部署的機器上一些修正時，都要去改 hosts 裡面的值，現在用這個寫法可以直接...

```bash
ansible-playbook -i inventory deploy_new_vm_env --extra-vars='variable_host=192.168.100.1'
```

針對 192.168.100.1 這台機器作部署或上 patch，倘若我都沒有下任何 extra-vars 的話就會直接針對 inventory 內所有機器 all 做處理。

這個寫法也可以結合 Ansible Tower 裡 Survey 功能進行 Self-service 應用。

## Suggestion 5: Ansible Workflow 實作提示

其實這概念是從 Ansible Tower 延伸出來的，將 `將多個 Playbook 序列化執行`，這個是在你的業務目標是由多個業務目標所建立起來才會需要的一個作法，你可以把它想成 Agile 專案術語 Epic / Story / Task 對應到 Ansible Workflow / Playbook / Roles。


Workflow 具體寫法是...
```yaml
cat deploy_openshift_cluster.yml
---
- import_playbook: prerequisite_env.yml
- import_playbook: deploy_core_componets.yml
- import_playbook: check_componets.yml
```

倘若你想要的話，也可以 workflow in workflow 持續疊下去，除非你對抽象化程度非常有把握，不然頂多做一層 workflow 應可應付多數需求。

此外這功能主要是為了 Ansible Tower 設計的，除了具備可視化 Workflow 以外，還有多加錯誤處理，當 playbook 跑道有問題的時候，可以跳到錯誤處理進行應對，譬如發信通知等等。

## Suggestion 6: 上版本控制系統 Version Control System

這邊的版控可以是任何一家的方案 git, svn, mercurial ...etc，沒有最佳只有最適合自己的方案，我自己是一律推崇使用 `git`，無論你是用中央式或分散式的管理方式，都可以用 git 辦到，極度好用

遵照 Infrastrcture as Code 的思想，盡量對所有 source code 的變動都要有所記錄。上版控有幾個好處，可以...
- 允許使用者將檔案恢復到某個時間的狀態
- 進行不同時間點的修改
- 確認該時間點誰進行了變更導致問題產生
- 容易進行備份管理

Ansible Tower 的專案內容更新也是需要`基於版本控制`進行設定檔的變動，方便進行 playbook 的內容管理

## Suggestion 7: 一 host 搭配多 group 無妨
host 是唯一值，而 group 可以將多個 host 定義再一起，還可以多重定義 host，換個角度說，你可以把 group 當作 tag 來用，對不同的角色、位置、任務來定義，這在下` --limit` 時會非常好用。

具體 inventory 寫法如下
```yaml
[infra:children]
db
www
lb

[db]
tw-db.pichuang.local

[www]
tw-www.pichuang.local

[lb]
tw-lb.pichuang.local

[tw]
tw-db.pichuang.local
tw-lb.pichuang.local
tw-www.pichuang.local
```

## Suggestion 8: 少用 `ansible_connection=local`，改用 `ansible_connection=ssh`

這邊不是要各位不能用 local，而是設計的時候請先以 ssh 連線為優先考量，有其他特殊需求再用 local，這主要會牽扯到 Ansible 連線行為，可以使用 -vvv 觀察；先前有遇過地雷是用 local 執行時，發現會有 permission 和環境變數的問題，經研究 issue 後，確定改成以 ssh 連線後就沒有此問題。此外 Ansible 預設 ansible_connection=ssh 在 inventory 不用特別明寫沒關係。

具體寫法如下
```yaml
# 不建議
localhost ansible_connection=local

# 建議
localhost ansible_connection=ssh ansible_host=localhost
```

## Suggestion 9: 多利用 `ping` 模組
習慣會在跑任何 playbook 前，先針對我要執行的機器跑 ping 確保我的機器狀況連接是正常，有時候可以透過這個動作提早發生網路跟執行問題。多數時候我都是把這個工作寫成一份通用 playbook，搭配 workflow 來做檢查。此外 Ansible 的 ping 是一個 Ansible 模組，不是單純的 linux tools - ping。

具體 ping playbook 寫法如下
```yaml
cat pingall.yml
---
- hosts: "{{ variable_host | default('all') }}"
  tasks:
    - name: Test connectivity
      ping:
      register: result
    - name: Print result
      debug:
        msg: "{{ result }}"
```

## Suggestion 10: 關閉 gather_facts 加速執行速度
因為 Ansible 本身是 agentless，執行程式預設一開始會先去收集 host 上的環境資訊 (e.g. IP, hostnmae, OS version…)，但管理的機器太多的時候，發現到收集的時間過長，可以選擇把 gather_facts: false，節省收集資訊的時間。
```yaml
---
- hosts: all
  gather_facts: false
  tasks:
    - name: ...
      ...
      ...
```

## Suggestion 11: 設定 fact_cache 用快取環境變數
倘若你真的需要用到 gather_facts 環境資訊，但又不想要短時間內一直等候收集資訊的時間，可以在 ansible.cfg 使用 fact_caching 來當作快取
```yaml
cat /etc/ansible/ansible.cfg

[defaults]
...
# 除了支援JSON file 以外，還另外支援 redis、memcached，在管大量機器的時候才需要搭配這方面的方案
fact_caching = jsonfile
fact_caching_connection = /tmp/facts
# hardtimeout
fact_caching_timeout = 600
...
```

## Suggestion 12: 增加 ssh 參數來提升連線的效率
其實這個比較偏向是 ssh 的傳輸調教，最主要是 ControlPersist，可以讓 ssh 建立的連線一直保持著連線，建議開起來以減少重複連線的時間，而 Pipelining 則可以減少 ansible control node 到 host 之間的連線數量，也有助於執行效率
```yaml
cat /etc/ansible/ansible.cfg

[ssh_connection]
...
ssh_args = -o ControlMaster=auto -o ControlPersist=300s
control_path = %(directory)s/%%h-%%r
pipelining = True
...
```

## Suggestion 13: 收集 tasks 處理時間

如果每次執行都不知道到底哪個 task 跑得最久和卡在哪裡，可以多加一個 ansible plugin profile_tasks 來做統計處理時間。各位可以看看那精美的 Gathering Facts 所消耗的時間有多久…

```yaml
/etc/ansible/ansible.cfg

[defaults]
...
callback_whitelist = profile_tasks
...
```

## Suggestion 14: 盡可能對 Inventory 提供資訊
盡可能在 Inventory 提供明喻資訊，而不採用隱喻資訊，尤其是 `ansible_host`，確保每個人看到這份 Inventory 的認知是一樣的

```bash
# 不建議
db1

# 建議
db1 ansible_host=10.1.2.75
```

## Suggestion 15: 對敏感工作不留 Log

若不想在 log 留下任何敏感性資料，可以在該 Task 多加一個 `no_log: True`

```bash
- name: Secret Task
  shell: /usr/bin/gg --value={{ secret_value }}
  no_log: True
```

# References
- [Ansible - Best Practies][1]
- [Best Pratices - Directory Layout][2]

[1]: https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#best-practices
[2]:
https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html#directory-layout