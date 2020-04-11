---
layout: post
title: How to contribute on GitHub
date: 2017-04-13 19:32:57
updated: 2017-04-13 19:32:57
tags:
  - misc
  - github
categories:
  - misc
---

Fork from [Github 發 Pull Request & 貢獻流程速查/timdream](https://gist.github.com/timdream/5968469)

# Github 發 Pull Request & 貢獻流程速查

## 前言

此文目標讀者需先自行學會
 
* 開 Github 帳號
* 會 fork GitHub repository
* 會使用 `git`
* 會 clone 自己的 repository
* 會開 feature branch ，commit 修改，且 push 到自己的 Github remote repository
* 會發 Pull Request

## 那到底還要教什麼？

除了極端的情況（例如 reviewer 對 pull request 來者不拒，照單全收）之外，貢獻者通常需要再度修改自己的 pull request 以順利的讓自己的作品被接受。這些步驟需要一些非初階且有時危險的 git 指令，且無法三言兩語對沒作過的人解釋。
此文件的目標就是要解釋這些步驟。

<!--more-->

### 情境：向 upstream 更新 Source Code

**解答：**新增一個新的 remote branch `upstream`
```
git remote add upstream https://github.com/xxx/xxx.git
gir pull upstream master
```

### 情境：還需要小修改

**解答：**進行 reviewer 要求的修改，commit 在原本的 feature branch 上然後 push 到 Github。新的 commit 會自動出現在 Pull Request。

### 情境：需要修改 commit message

**解答：**有時候 commit message 會有錯字，或是 reviewer 需要符合規則的文字（例如加上 issue 編號），這時你需要修改原本的 commit。輸入：

```
git commit --amend
git push -f
```

注意 `git push -f` 會將您原本的 commit 複寫，請注意自己是否操作正確。操作正確的話回到 pull request 上面會看到一個 commit，對應完整與正確可以被接受的修改，以及 commit message。

### 情境：需要小修改且需要「squash commits」

**解答 A：**Reviewer 不希望在 Pull request 來來回回的小修改永遠留在紀錄裡，希望您在修改時保持還是只有一個 commit 在 pull request 裡的狀態。
在本地修改後請在您的 feature branch 上使用下面的指令修改原本的 commit，**而不產生新的 commit**。

```
git commit --amend
git push -f
```

注意 `git push -f` 會將您原本的 commit 複寫，請注意自己是否操作正確。操作正確的話回到 pull request 上面會看到一個 commit，對應完整與正確可以被接受的修改。

**解答 B：**所謂的「squash」指的是 interactive rebase，也就是把**已經不幸留下的多個 commit 壓扁回一個 commit**。您的 reviewer 可能會請您先把修改的 commit push 上 pull request 之後，最後被接受前再 squash。這時您需要：

```
git rebase -i HEAD~<n>
...<在文字編輯器內將所有的修改 commit 標為「s」>
git push -f
```
- 其中，`<n>` 是您的總共的 commit 數量。我們需要告訴 git 您一共需要回朔幾步，複寫已經發生的紀錄。操作正確的話回到 pull request 上面會看到一個 commit，對應完整與正確可以被接受的修改。

### 情境：Pull Request 過期，無法合併

**解答：**Pull request 在 Github 可以用一個大的綠色按鈕直接合併回 master branch。如果 master 的修改已經和 pull request 上的修改有衝突，按鈕會變成灰色，且顯示訊息 `We can’t automatically merge this pull request.`

這時您的 reviewer 會希望您能夠更新您的 pull request 來確保 pull request 的修改紀錄無誤。
操作方法如下：

1. 首先，先新增 remote repo：`git remote add upstream <源頭 repository 的 git read-only URL>`
2. 擷取源頭 repository 的最新修改：`git fetch upstream`
3. 將您的 feature branch 的修改套用到最新的 master 之上：`git rebase upstream/master`
4. 此時 git 會抱怨檔案有衝突。打開文字編輯器將有衝突的檔案修好（檔案會亂掉被 git 加上很多箭頭），然後 `git add <修好的檔案>`。全部修完之後輸入：`git rebase --continue`。
5. 最後將整理好的 branch push 出去：`git push -f`

## 結語

git 只是工具，有興趣的話市面上有很多資源可以深究上面那些指令的意義。
Github 也是工具，只是剛好在當代「Github workflow」成為 open source 貢獻的顯學，所以您需要學這些（笑）。或許 Pull Request 的介面未來會新增功能來讓人們能用網頁操作上面的那些動作。

強烈建議開發者在 repository 裡留下 [CONTRIBUTE.md](http://contribute.md/) 來解釋 Pull Request 的接受規則，以免讓新的貢獻者覺得無所適從。或是如果您摸透了此 repository 作者的品味的話，幫他寫好發 pull request 給他也不錯。