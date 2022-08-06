---
title: npm常用命令
date: 2021-11-25 16:46:08
permalink: /pages/12233e/
categories: 
  - 开发者手册
  - Others
tags: 
  - 
---

### 查找帮助

  - `npm -l`

    显示所有可用命令

  - `npm <command> -h`

    查看 `<command>` 命令的简单使用帮助

  - `npm help <command>`

    查看 `<command>` 命令的详细使用指南

### 创建项目

  - `npm init (-y)`

    在命令行所在的文件夹初始化一个项目（创建 `package.json` 文件）

### 安装模块

- `npm install package-name --save(-S)`

  局部安装模块，安装在命令行所在的文件夹；并将模块依赖写入到 `package.json` 文件的 `dependencies` 中（生产环境）

  简写：`npm i package-name`

- `npm install package-name --save-dev(-D)`

  局部安装时将模块依赖写入到 `package.json` 文件的 dev`Dependencies` 中（开发环境）

  简写：`npm install -D package-name`

- `npm install package-name --global(-g)`

  全局安装模块

### 卸载模块

- `npm uninstall package-name`

  卸载局部模块

- `npm uninstall package-name --global(-g)`

  卸载全局模块

### 更新模块

- `npm update package-name`

  更新局部模块

- `npm update -g package-name`

  更新全局模块

- `npm install -g package-name@x.x.x`

  更新全局模块 `package-name` 到 x.x.x 版本

### 清除 npm 缓存

- `npm cache clean --force`

### 查看某个模块的全部版本

- `npm view package-name versions`

### 淘宝 npm 镜像

registry 地址：registry.npm.taobao.org

**使用方法：**

  - 临时使用

  ``` shell
  npm install package-name --registry=https://registry.npm.taobao.org
  ```

  - 长期使用

  ``` shell
  npm config set registry=https://registry.npm.taobao.org
  ```

