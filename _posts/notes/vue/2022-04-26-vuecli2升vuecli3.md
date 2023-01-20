---
layout: post
title:  vuecli2升级vuecli3遇到的问题
date:   2022-03-21 11:00:00 +0800
categories: 笔记
tag: vuecli
---
* content
{:toc}

## 挂载节点app的方法不同

main.js挂载文件有区别
    以前是`el:#app`

```js
//vuecli2
new Vue({
    el: '#app',
    router,
    store,
    components: { App },
    template: '<App/>',
});
```

现在是

```js
//vuecli3
new Vue({
    router,
    store,
    render: h => h(App)
}).$mount('#app')

```

## vue-router的版本，then报错

新的router跳转增加的回调，旧代码不写就会报错
也可以全局处理then的问题,讲touter实例的方法覆盖掉

```js
const originalPush = Router.prototype.push
Router.prototype.push = function push(location) {
  return originalPush.call(this, location).catch(err => err)
}
```

## router-link写法报错

新的router-link版本删除了 `<router-link>` 中的 event 和 tag 属性。

`<router-link>` 中的 event 和 tag 属性都已被删除。你可以使用 v-slotAPI 来完全定制 `<router-link>`：
将

```vue
 <router-link to="/about" tag="span" event="dblclick">About Us</router-link>
```

 替换成

```vue
<router-link to="/about" custom v-slot="{ navigate }">   
<span @click="navigate" @keypress.enter="navigate" role="link">About Us</span> 
</router-link> 
```

原因：这些属性经常一起使用，以使用与 `<a>` 标签不同的东西，但这些属性是在 v-slot API 之前引入的，并且没有足够的使用，因此没有足够的理由为每个人增加 bundle 包的大小。

## 一些文件上的不同

vuecli2 | vuecli3
--|--
jsconfig.json | \
.project | \
.postcssrc.js | postcss.config.js
.editorconfig | \
.babelrc | babel.config.js
static | public
config | \
build | \
\ | .eslintrc
\ | .browserslistrc

src内部
删除views文件夹
router.js -> 文件夹 用法无区别

store.js ->文件夹 用法无区别

## 一些插件

npm install --save axios crypto-js dayjs element-ui nprogress vue-particles less-loader

改index.html，icon，config.js

## 配代理

从webpack的config，改到vue.config.js

## sass-loader8不支持/deep/变为::v-deep

也可以采取版本降级
