layout: post
title: Red Hat Ansible 和 HashiCorp Terraform 結合作法
author: Phil Huang
tags:
  - terraform
  - ansible
categories: []
date: 2019-03-20 02:03:00
---
雖然 Terraform 及 Ansible 都是非常知名的 IaC (Infrastructure as Code) 的工具，但經常會被拉出來討論說這兩者的差異，所以經歷了一陣奮鬥(?)，Red Hat 及 HashiCorp 發出聯合聲明 [HashiCorp Terraform and Red Hat Ansible Automation][1]

<!--more-->

依據不同的角度，應該會有以下兩種不同的結合方式 (前者整後者)

1. Red Hat Ansible + HashiCorp Terraform
2. HashiCorp Terraform + Red Hat Ansible 

當然你也可以繼續堆疊下去，但至多三層就算多了，但本文不討論這個狀況

## Terraform Invoking Ansible

核心部分是以 `Terraform` 為主，搭配內建的兩個 Provisioner: [remote-exec][4] 和 [local-exec][5] 來呼叫 ansible-playbook 指令進行設定檔配置，範例請參考以下作法

```bash

resource "aws_instance" "web" {
  provisioner "local-exec" {
  command = "ansible-playbook -u rhel -i hosts main.yml"
 }
  provisioner "remote-exec" {
    inline = [
      "ansible-playbook -u rhel -i hosts main.yml",
      "...",
    ]
  }
}

# Commands
terraform plan
terraform apply
```

至於各自執行時詳細的步驟，可參閱 [HashiCorp Terraform and Red Hat Ansible Automation][1] p4/p5

## Ansible Invoking Terraform

核心部分是以 `Ansible` 為主，採用 Ansible 所提供的 [terraform][6] 模組，依據需求設計出 Plan / Apply / Destroy /... 等等的 Playbooks 供使用者使用

```yaml
---
- name: Ansible Automation Invoking Terraform
  hosts: all
  gather_facts: false
  connection: local
  tasks:
  - name: plan
    terraform:
    <...Ignore...>
  - name: apply
    terraform:
    <...Ignore...>
  - name: destroy
    terraform:
    <...Ignore...>
```

## 到底哪個方案比較好?

其實這個沒有一定的好壞，重點是你一定要能掌握端到端業務需求 (End-to-End) 的系統全域觀念

就我個人認知上，Terraform 是比較`垂直深度`針對特定環境控制管理資源，尤其是 Public Cloud Resource，而 Ansible 比較是容易針對`水平橫跨`不同混合環境創造出工作流 (Workflow) 或者是做面向 OS 內部細微的 Configuration Management (CM)

## References
- [HashiCorp Terraform and Red Hat Ansible Automation][1]
- [Automation for everyone][2]
- [Terraform by HashiCorp][3]
- [Terraform remote-exec Provisioner][4]
- [Terraform local-exec Provisioner][5]
- [Ansible Module - terraform][6]
- [阿貝好威的實驗室 - infrastructure as code 的組合技 Ansible + Terraform][7]

[1]: https://www.redhat.com/en/resources/hashicorp-terraform-ansible-infrastructure-as-code-overview
[2]: https://www.ansible.com/
[3]: https://www.terraform.io/
[4]: https://www.terraform.io/docs/provisioners/remote-exec.html
[5]: https://www.terraform.io/docs/provisioners/local-exec.html
[6]: https://docs.ansible.com/ansible/latest/modules/terraform_module.html
[7]: http://lab.howie.tw/2019/03/infrastructure-as-code-ansible-terraform.html