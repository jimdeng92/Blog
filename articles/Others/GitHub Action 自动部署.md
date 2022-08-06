---
title: GitHub Action 自动部署
date: 2021-12-09 11:14:31
permalink: /pages/d971a3/
categories: 
  - 开发者手册
  - Others
tags: 
  - 
---

## 前言

GitHub Action 是在 2019 推出的，用来实现 github 仓库的持续集成。阮大神还特地写了一篇 [《GitHub Actions 入门教程》](http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html?20191227113947#comment-last)，GitHub Action 能非常方便的自动部署你的项目，比如我的静态博客网站使用 vuepress 构建。以前在我使用 deploy.sh 构建时，它确实也很方便，只需要执行 `npm run deploy` 就能自动打包并 push 到我的仓库。但是因为我肯定不止在一台电脑上提交我的代码，因此，我不得不在每次构建时想想我还应该把源代码也 push 到另一个分支上。这并不麻烦，但需要每次都提醒自己。而 GitHub Action 则只需要我在编辑后提交，而后的所有事情都能自动完成。让我们来看一看它是怎么做到的。

## 开始

GitHub Action 需要我们在项目目录下放一个 .github/workflows/ci.yml 文件，其中 ci.yml 文件名随意，而文件后缀必须为 .yml。

我的 ci.yml 文件：

``` yml
name: CI

#on: [push]

# 在master分支发生push事件时触发。
on:
  push:
    branches:
      - master

env: # 设置环境变量
  TZ: Asia/Shanghai # 时区（设置时区可使页面中的`最近更新时间`使用时区时间）

jobs:
  build: # 自定义名称
    runs-on: ubuntu-latest # 运行在虚拟机环境ubuntu-latest

    strategy:
      matrix:
        node-version: [14.x]

    steps:
      - name: Checkout # 步骤1
        uses: actions/checkout@v1 # 使用的动作。格式：userName/repoName。作用：检出仓库，获取源码。 官方actions库：https://github.com/actions
      - name: Use Node.js ${{ matrix.node-version }} # 步骤2
        uses: actions/setup-node@v1 # 作用：安装nodejs
        with:
          node-version: ${{ matrix.node-version }} # 版本
      - name: run deploy.sh # 步骤3
        env: # 设置环境变量
          GITHUB_TOKEN: ${{ secrets.ACCESS_TOKEN }} # toKen私密变量
        run: npm install && npm run deploy
```

这个文件所描述的大致意思就是：我们在 master 分支上 push 代码的时候，获取仓库源码，安装 nodejs，把 token 设置到环境变量，安装项目依赖并运行 deploy.sh 文件。

里面涉及到一个 `secrets.ACCESS_TOKEN` 的变量，这个需要我们到 github 上配置。

### 配置 `secrets.ACCESS_TOKEN`

进入 github，点击右上角头像，依次打开 Setting - Developer settings - Personal access tokens。点击 Generate new token：

::: center
![微信截图_20211209162310](https://cdn.jsdelivr.net/gh/jimdeng92/static_1@main/微信截图_20211209162310.kpl2wr08dz4.png)
:::

输入 access token 名称，过期时间并勾选 repo。（过期时间可以尽量选长ken一些，如果过期了自动部署会报 401，需要更新 token）点击下方 Generate token 后得到 token，复制它。

::: center
![微信截图_20211209162442](https://cdn.jsdelivr.net/gh/jimdeng92/static_1@main/微信截图_20211209162442.20r26022ew8w.png)
:::

打开仓库的 Settings/Secrets ，点击 New repository secret，**Name 填入名称为 ci.yml 中定义的变量名**，我的就是 `ACCESS_TOKEN`，Value 就是 token，点击保存。

::: center
![微信截图_20211209163548](https://cdn.jsdelivr.net/gh/jimdeng92/static_1@main/微信截图_20211209163548.19slk94i7aw0.png)
:::

### deploy.sh

deploy.sh 的作用就是自动打包并发布到 github 的指定分支上。

``` sh
#!/usr/bin/env sh

# 确保脚本抛出遇到的错误
set -e

# 生成静态文件
npm run build

# 进入生成的文件夹
cd docs/.vuepress/dist

# deploy to github pages
echo 'imlinhe.com' > CNAME

if [ -z "$GITHUB_TOKEN" ]; then
  msg='deploy'
  githubUrl=git@github.com:jimdeng92/jimdeng92.github.io.git
else
  msg='来自github actions的自动部署'
  githubUrl=https://jimdeng92:${GITHUB_TOKEN}@github.com/jimdeng92/jimdeng92.github.io.git
  git config --global user.name "jimdeng92"
  git config --global user.email "jimdeng92@gmail.com"
fi
git init
git add -A
git commit -m "${msg}"
git push -f $githubUrl master:gh-pages # 推送到github gh-pages分支

cd -
rm -rf docs/.vuepress/dist
```

到这里配置就完成了，尝试 push 一下代码，就会在 Actions 里看到正在部署的进程。
