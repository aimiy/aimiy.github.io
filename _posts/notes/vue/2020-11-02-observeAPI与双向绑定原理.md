---
layout: post
title:  observeAPI与双向绑定原理.md
date:  2020-11-02 11:00:00 +0800
categories: 
tag: 
---
* content
{:toc}

## Vue.observable()进行状态管理

公司项目写了一个vue插件，之前使用的eventBus实现，可读性很差，所以考虑使用vue的状态管理，但由于vuex被建议在大项目使用，因此，使用了vue的API，来处理一些些简单的跨组件数据状态共享。

参考文档，我的实现方案如下：

### 建立store.js文件

```js
let store = {
  state: Vue.observable({
    jsurl: "",
    wsurl: "",
    token: null,
    sceneId: null,
    callStatus: 1, // 前端存储的通话状态，0-通话中，1-挂机
    chooseMap: null, // 选中的流程导航的id
    clickNodeName: "", // 点击的节点名称
    processDTOs: [],
    fills: [],
    entityInfo: [], // 实体信息
    userPortraits: [],
    nowPortrait: [],
    wordsRemind: {},
    chat: [],
    doneDTOs: [],
    wordsLibraries: [],
    qualityInspectionNum0: null, // 质检告警次数
    qualityInspectionNum1: null // 质检告警次数
  })
}
export default store
```

这样store.state里面声明的每一个值都可以响应式使用。

但是，有的时候不是所有的值都是默认设置进去的，如果直接state添加值，不会响应式，因此

### 添加一个setState方法

看一下文档
>[Vue.set](https://cn.vuejs.org/v2/api/#Vue-set)( target, propertyName/index, value )
>
>* 参数：
>
>   * {Object | Array} target
>   * {string | number} propertyName/index
>   * {any} value
>* 返回值：设置的值。
>
>* 用法：
>    向响应式对象中添加一个 property，并确保这个新 property 同样是响应式的，且触发视图更新。它必须用于向响应式对象上添加新 property，因为 Vue 无法探测普通的新增 property (比如 this.myObject.newProperty = 'hi')
>
>注意对象不能是 Vue 实例，或者 Vue 实例的根数据对象。

实现如下

```js
let store = {
  state: Vue.observable({
    jsurl: "",
    wsurl: "",
    token: null,
    sceneId: null,
    callStatus: 1, // 前端存储的通话状态，0-通话中，1-挂机
    chooseMap: null, // 选中的流程导航的id
    clickNodeName: "", // 点击的节点名称
    processDTOs: [],
    fills: [],
    entityInfo: [], // 实体信息
    userPortraits: [],
    nowPortrait: [],
    wordsRemind: {},
    chat: [],
    doneDTOs: [],
    wordsLibraries: [],
    qualityInspectionNum0: null, // 质检告警次数
    qualityInspectionNum1: null // 质检告警次数
  }),
  setState(key, val) {
    if (key) {
      Vue.set(this.state, key, val)
      localStorage.setItem("sdk" + key, JSON.stringify(val))
    }
  },
}
export default store
```

我这里同时做了一些存储状态与localstorage同步的处理。`localStorage.setItem("sdk" + key, JSON.stringify(val))`

### 添加mapState方法

对state的引用每一个都使用this.store.state的方式写在计算属性种，是比较麻烦的，因此

```js
function mapState(keys) {
  const map = {}
  keys.forEach((key) => {
    map[key] = () => {
      return store.state[key]
    }
  })
  return map
}

export { mapState };
```

### mutations

其实同理我们也可以设置一些mutations来当作全局方法，但这个其实是最简单的函数，跟vuex同名而已

```js
export let mutations={
    setCount(count){
        store.count=count;
    },
    changeName(name){
        store.name=name;
    }
}
```

**一些搭配方法记录**
由于做vuex与localstorage同步，写了一个从localstorage种填充vuex值得方法

```js
refreshState() {
    for (let key in store.state) {
      let ls = localStorage.getItem("sdk" + key);
      if (ls) {
        try {
          store.state[key] = JSON.parse(ls)
          Vue.set(store.state, key, JSON.parse(localStorage.getItem("sdk" + key)))
        } catch (error) {
          store.state[key] = ls;
          Vue.set(store.state, key, localStorage.getItem("sdk" + key))
        }
      }
    }
  },
```

还有一个初始化vuex状态的方法

```js
initAllState() {
    for (let key in store.state) {
      let type = Object.prototype.toString.call(store.state[key])
      switch (type) {
        case "[object Array]":
          store.setState(key, [])
          break;
        case "[object Object]":
          store.setState(key, {})
          break;
        case "[object Set]":
          store.setState(key, new Set())
          break;
        case "[object Map]":
          store.setState(key, new Map())
          break;
        default:
          store.setState(key, null)
      }
    }
    // localStorage.clear()
  }
```
