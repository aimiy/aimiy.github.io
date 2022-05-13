---
layout: post
title:  vuerouter报错相关
date:   2021-01-21 11:00:00 +0800
categories: 笔记
tag: vuerouter
---
* content
{:toc}

## 重复点击相同路由，出现 Uncaught (in promise) NavigationDuplicated: Avoided redundant navigation

### 更改路由push方法

方案一：只需在 router 文件夹下，添加如下代码：

```js
// src/router/index.js
Vue.use(Router)
const router = new Router({
  routes
})

const VueRouterPush = Router.prototype.push
Router.prototype.push = function push (to) {
  return VueRouterPush.call(this, to).catch(err => err)
}
```

### 路由导航守卫添加判断

方案二：在跳转时，判断是否跳转路由和当前路由是否一致，避免重复跳转产生问题。

```js
toMenu (item) {
  if (this.$route.path !== item.url) {
    this.$router.push({ path: item.url })
  }
}
```

### push回调添加catch方法

方案三：使用 catch 方法捕获 router.push 异常。

this.$router.push(route).catch(err => {
  console.log('输出报错',err)
})

参考：<https://blog.csdn.net/qq_36437172/article/details/108269846>
