---
layout: post
title: "常用git指令"
description: "General GIT command"
date: 2015-05-12 00:00:00 +0800
updated: 2015-05-12 00:00:00 +0800
category: Misc
tags: 
- misc
- git
---

* 丟棄 non-commit code
    * `git reset --hard HEAD`
* 抓取 upstream source
    1. `git remote add upstream https://github.com/xxx/xxx`
    2. `git fetch upstream`
* 移除 local branch [trash]  
    * `git branch -d trash`
* 移除 remote branch [origin/trash]  
    * `git push origin :trash`
    * `git branch -rd origin/trash`
* 強制移動 branch 指向某個 commit [HEAD -> HEAD~3] #level rampup3
    * `git branch -f master HEAD~3`
* local branch 退回至上一個 commit [HEAD -> HEAD~1] #level rampup4
    * `git reset HEAD~1`
* remote branch 退回至上一個 commit 並將狀態分享出來 #level rampup4
    * `git revert HEAD`
* 複製多個 commit 至 HEAD 底下 [C2 C4] #level move1
    * `git cherry-pick C2 C4`
* 重新排序或忽略某些 commit [HEAD <-> HEAD~4] #level move2
    * `git rebase -i HEAD~4 --aboveall`
* 對某個 hash 上 tag [v1 C3] #level miexd4
    * `git tag v1 C3`
* branch rebase [parent children] #level advanced1
    1. `git rebase master feature1`
    2. `git rebase feature1 feature2`
    3. `git rebase feature2 master`
* 選擇第二個 parent commit #level advanced2 
    * `git checkout HEAD^2`
* 解決 diverged history, 使用 rebase 整合 origin/master 和 master #level remote7, remoteAdvanced2
    * `git pull --rebased`
* 參考 remote branch 來 checkout 一個新的 branch #level remoteAdvanced3
    * `git checkout -b new-branch origin/master`
    * `git pull`
* 設定 remote tracking 來 push 至 remote branch 
    * `git branch -u origin/master new-branch`
    * `git push`
* push 至不存在 remote branch
    * `git push origin master:new-branch`
* 抓下 remote branch #level remoteAdvanced6
    * `git fetch origin branch-name`
* About git pull 
    * `git pull origin foo` 等同於 `git fetch origin foo; git merge origin/foo`
    * `git pull origin bar~1:bugFix` 等同於 `git fetch origin bar~1:bugFix; git merge bugFix`
* 修改已經 commit 的 comment
    * `git commit --amend`
    * `git rebase --continue`
* rebase 發生錯誤, 修復後要繼續進行修改
    * `git rebase --continue`

Reference
---------
- [learnGitBranching](http://pcottle.github.io/learnGitBranching/)
- [git-tips](https://github.com/git-tips/tips)