---
layout: post
title:  vue-router复用组件不刷新的问题
date:   2021-01-21 11:00:00 +0800
categories: 笔记
tag: vue-router
---
* content
{:toc}

使用vue-router时，有的时候不同的路由会映射到相同的页面。这个时候vue-router是有组件的缓存优化的，不会刷新页面。

比如：

```js
const User = {
  template: '<div>User</div>',
}

// 这些都会传递给 `createRouter`
const routes = [
  // 动态段以冒号开始
  { path: '/users/:id', component: User },
]
```

或者我自己做的路由配置化，没有id直接配置的相同路由路径

```js
            {
                id: 4,
                menuName: "意图标签",
                menuType: 0,
                menuUrl: "/tagManage/intent",
                filePath: "/tagManage/index",
            },
            {
                id: 4,
                menuName: "情感标签",
                menuType: 0,
                menuUrl: "/tagManage/emotion",
                filePath: "/tagManage/index",
            },
            {
                id: 4,
                menuName: "句式标签",
                menuType: 0,
                menuUrl: "/tagManage/sentence",
                filePath: "/tagManage/index",
            },
```

这个时候我们的路由映射的是页面，不是子组件，还是希望页面刷新。

## 页面对应的routerview添加:key，来阻止不同页面相同路由的复用

```vue
<router-view :key="$route.fullPath"></router-view>
```

这样可以保证每个路由都针对页面path来缓存，而不是组件本身

## 用watch监听

有的时候并不想全局修改页面组件的复用，还是希望同组件不刷新。仅针对特定页面做刷新处理，就可以使用watch

```vue
mounted() {
 this.getList();
},
watch: { //通过watch来监听路由变化
 "$route": function(){
    this.getList();
    }
}
```

## 使用路由钩子

类似与watch但又是全局处理的方法

```vue
beforeRouteEnter (to, from, next) {
 // 在渲染该组件的对应路由被 confirm 前调用
 // 不！能！获取组件实例 `this`
 // 因为当钩子执行前，组件实例还没被创建
},
beforeRouteUpdate (to, from, next) {
 // 在当前路由改变，但是该组件被复用时调用
 // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
 // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
 // 可以访问组件实例 `this`
},
beforeRouteLeave (to, from, next) {
 // 导航离开该组件的对应路由时调用
 // 可以访问组件实例 `this`
}
```

参考文档：
[vue-router不同的路由共用一个组件component的问题](https://blog.csdn.net/qq_32615575/article/details/106194104?utm_term=vue%E5%90%8C%E4%B8%80%E7%BB%84%E4%BB%B6%E4%B8%8D%E5%90%8C%E8%B7%AF%E7%94%B1&utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~all~sobaiduweb~default-0-106194104&spm=3001.4430)
