---
title: git分支规范
date: 2021-11-05 11:05:01
permalink: /pages/5dba3c/
categories: 
  - 速记
  - 杂项
tags: 
  - 
---

### master 分支

- master 为主分支，也是用于部署生产环境的分支，确保 master 分支稳定性；
- master 分支一般由 develop 以及 hotfix 分支合并，任何时间都不能直接修改代码。

### develop 分支

- develop 为开发分支，始终保持最新完成以及bug修复后的代码。

### hotfix 分支

- 分支命名: hotfix/ 开头的为修复分支；
- 线上出现紧急问题时，需要及时修复，以 master 分支为基线，创建 hotfix 分支，修复完成后，需要合并到 master 分支和 develop 分支。
