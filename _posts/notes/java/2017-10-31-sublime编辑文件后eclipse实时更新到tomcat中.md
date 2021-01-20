---
layout: post
title:  sublime编辑文件后eclipse实时更新到tomcat中
date:   2017-10-30 23:21:00 +0800
categories: 笔记
tag: java
---

* content
{:toc}

平时做web开发用sublime,但需要用eclipse把项目部署到tomcat中.以前用的eclipse版本是可以在sublime更新文件后自动把新文件部署到tomcat中的,最近换个了新版本的eclipse版本这个功能就消失了,必须手动F5在eclipse中刷新一次,多了一步操作,不爽。解决方式：
Window -> Preferences -> General -> Workspace -> Refresh using native hooks or polling 把这个勾选上就可以了。
Build automaticall 和 Refresh on access 应该是默认就勾选了的，如果没有的话也一起勾选上。
