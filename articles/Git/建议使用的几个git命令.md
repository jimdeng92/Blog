---
title: 建议使用的几个git命令
date: 2022-05-10 14:35:34
permalink: /pages/007030/
categories:
  - 开发者手册
  - Git
tags:
  -
---

现在的 vscode 已经足够优秀，它内置了git，方便我们通过图形化的界面去操作git。但图形化的界面也有自己的不足之处---有时候用户自己都不知道自己在干什么，经常搞得一团雾水。

所以我一般能使用命令行解决的一般都会使用命令行，自己知道自己在干什么，操作起来感觉思路更清晰。今天就说说我经常使用的几个git技巧。

## stash

`git stash`方便我们同时去开发多个任务，一般来说，一个分支用来负责一个任务，但是在实际的工作中经常会有多个任务并行的情况，可能是因为接口还没有联调，只能搁置，可能是需求变动，需要修改。这时候我们开发到一半的任务不能直接提交，就可以使用`git stash`进行暂存。

常用的stash指令：

``` shell
# 保存当前未commit的代码
git stash

# 保存当前未commit的代码并添加备注
git stash save "备注的内容"

# 列出stash的所有记录
git stash list

# 删除stash的所有记录
git stash clear

# 应用最近一次的stash
git stash apply

# 应用最近一次的stash，随后删除该记录
git stash pop

# 删除最近的一次stash
git stash drop
```

当有多条stash，可以指定操作stash，首先使用 stash list 列出所有记录：

``` shell
$ git stash list
stash@{0}: WIP on ...
stash@{1}: WIP on ...
stash@{2}: On ...
```

应用第二条记录：

``` shell
git stash apply --index 1
```

pop和drop同理。

## reset

> 完全不接触索引文件或工作树（但会像所有模式一样，将头部重置为）。这使您的所有更改的文件更改为“要提交的更改”。

回退你已提交的 commit，并将 commit 的修改内容放回到暂存区。

一般我们在使用 reset 命令时，git reset --hard 会被提及的比较多，它能让 commit 记录强制回溯到某一个节点。而 git reset --soft 的作用正如其名，--soft (柔软的) 除了回溯节点外，还会保留节点的修改内容。

### 应用场景

回溯节点，为什么要保留修改内容？

应用场景1：有时候手滑不小心把不该提交的内容 commit 了，这时想改回来，只能再 commit 一次，又多一条“黑历史”。

应用场景2：规范些的团队，一般对于 commit 的内容要求职责明确，颗粒度要细，便于后续出现问题排查。本来属于两块不同功能的修改，一起 commit 上去，这种就属于不规范。这次恰好又手滑了，一次性 commit 上去。

### 命令使用

学会 `reset --soft` 之后，你只需要：

``` shell
# 恢复最近一次 commit
git reset --soft HEAD^
```

`reset --soft` 相当于后悔药，给你重新改过的机会。对于上面的场景，就可以再次修改重新提交，保持干净的 commit 记录。

以上说的是还未 push 的commit。对于已经 push 的 commit，也可以使用该命令，不过再次 push 时，由于远程分支和本地分支有差异，需要强制推送 `git push -f` 来覆盖被 reset 的 commit。

还有一点需要注意，在 `reset --soft` 指定 commit 号时，会将该 commit 到最近一次 commit 的所有修改内容全部恢复，而不是只针对该 commit。

举个栗子：

commit 记录有 c、b、a。

reset 到 a。

``` shell
git reset --soft 1a900ac29eba73ce817bf959f82ffcb0bfa38f75
```

此时的 HEAD 到了 a，而 b、c 的修改内容都回到了暂存区。

## rebase

rebase 也就是变基。它能让我们的git记录保持线性的结构，比merge看起来更清爽，当然使用merge也不是不可以，它也能体现我们开发提交的整个过程。

只是由于我们公司项目代码提交规范必须使用feat/fix/refator...，而merge的话会产生一条多余的记录，因此在我们这里不适用。

![basic-rebase-1](https://cdn.jsdelivr.net/gh/jimdeng92/static_1@main/basic-rebase-1.1qj3m134f5c0.webp)

我们在 experiment 上进行开发，这时候同事提交了一条commit并且合并到了master上。

![basic-rebase-2](https://cdn.jsdelivr.net/gh/jimdeng92/static_1@main/basic-rebase-2.5wqxupxk8s80.webp)

如果我们要将我们的开发完成的分支合并到 master 上，就会产生一条多余的历史 C5。

此时，我们就可以使用 rebase，将 experiment 分支变基到 master 分支上。

``` shell
git switch experiment
git rebase master
```

![basic-rebase-3](https://cdn.jsdelivr.net/gh/jimdeng92/static_1@main/basic-rebase-3.1th8swpx2h8g.webp)

回到 master 分支上，进行一次快进合并。

``` shell
git switch master
git merge experiment
```

![basic-rebase-4](https://cdn.jsdelivr.net/gh/jimdeng92/static_1@main/basic-rebase-4.30t7jtyfztk0.webp)

