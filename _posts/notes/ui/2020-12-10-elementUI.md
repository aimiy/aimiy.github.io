---
layout: post
title:  elementUI遇到的经验
date:   2020-12-10 16:44:00 +0800
categories: 笔记
tag: elementUI
---

* content
{:toc}

## 回车会触发刷新

form表单里的回车键不做处理的话，会触发刷新。el-form添加如下

```js
@submit.native.prevent
```
