---
layout: post
title:  nvm管理nodejs
date:   2017-04-15 11:00:00 +0800
categories: 笔记
tag: nodejs
---
* content
{:toc}

## [nvm](https://github.com/nvm-sh/nvm)

在实际的前端开发过程中，可能会经常遇见 node.js 的版本问题，不同的项目需要使用不同的 node.js 版本。

直接安装的话，只能安装和使用 node.js 的一个版本。可以使用 nvm 来安装和管理不同版本的 node.js。

如果有安装过node，最好进行卸载，因为自己安装的，nvm控制不了，会出现一些意想不到的报错，先安装nvm，再使用nvm命令安装需要的版本即可！

### 常用命令

nvm ls ：列出所有已安装的 node 版本
nvm list ：列出所有已安装的 node 版本
nvm list available ：显示所有可下载的版本
nvm install latest：安装最新版 node
nvm install [node版本号] ：安装指定版本 node
nvm uninstall [node版本号] ：删除已安装的指定版本
nvm use [node版本号] ：切换到指定版本 node
nvm current ：当前 node 版本
nvm unalias [别名] ：删除已定义的别名
