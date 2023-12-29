---
layout: post
title:  el-dialog同级的列表被push数据后for循环渲染失效，视图不更新
date:   2023-01-21 11:00:00 +0800
categories: 
tag: 
---
* content
{:toc}

## 问题

遇到一个bug，v-for与dialog同级的时候，当dialog的`:append-to-body="true"`，一旦框被打开过，无论怎么更新list都不会触发视图的更新，如果list单独显示，会发现，list本身是变化了的，就是v-for更新不了。

```vue
<template>
    <div>
        <el-button type="primary" @click="openDialog">打开dialog</el-button>
        <el-button type="primary" @click="updateList">更新list</el-button>
        {{list}}
        <div v-for="(item, index) of list" :key="index">
            {{ item }}
        </div>
        <el-dialog title="测试" :visible.sync="visible" :append-to-body="true">
            测试dialog
        </el-dialog>
    </div>
</template>

<script>

export default {
    data() {
        return {
            list: [0, 1, 2, 3],
            visible: false
        };
    },
    computed: {},
    methods: {
        openDialog() {
            this.visible = true;
        },
        updateList() {
            this.list.push(this.list.length)
        }
    },
    created() { },
    mounted() { },
    components: {}
}
</script>
<style lang='less' scoped></style>
```

后来临时查询到别人也遇到此类问题，解决方案是，将list用一个div包裹。（<https://blog.csdn.net/weixin_45714898/article/details/126757983）>

## 探究

就很疑惑，到底vue的v-for更新与dialog有什么矛盾。

### `:append-to-body="true"`

首先比较表象的就是`:append-to-body="true"`的区别。
当为false时，与v-for div同级的元素没有增多减少，只是显隐状态发生变化

```html
<div data-v-0fbfe4e5=""> 0 </div>
<div data-v-0fbfe4e5=""> 1 </div>
<div data-v-0fbfe4e5=""> 2 </div>
<div data-v-0fbfe4e5=""> 3 </div>
<div data-v-0fbfe4e5=""> 4 </div>
<div data-v-0fbfe4e5=""> 5 </div>
<div data-v-0fbfe4e5="" class="el-dialog__wrapper" style="z-index: 2005; display: none;">
</div>
```

当为true时，elmentui会将同级的元素删除，并且在body中新增弹窗

弹窗打开前的代码层级（简化）

```html
<body>
  <div id="app">
      <div data-v-0fbfe4e5="">
          <div data-v-0fbfe4e5=""> 0 </div>
          <div data-v-0fbfe4e5=""> 1 </div>
          <div data-v-0fbfe4e5=""> 2 </div>
          <div data-v-0fbfe4e5=""> 3 </div>
          <div data-v-0fbfe4e5="" class="el-dialog__wrapper" style="display: none;">
          </div>
      </div>
  </div>
</body>
```

弹窗打开关闭后

```html
<body class="">
    <div id="app">
        <div data-v-0fbfe4e5="">
            <div data-v-0fbfe4e5=""> 0 </div>
            <div data-v-0fbfe4e5=""> 1 </div>
            <div data-v-0fbfe4e5=""> 2 </div>
            <div data-v-0fbfe4e5=""> 3 </div>
        </div>
    </div>
    <div data-v-0fbfe4e5="" class="el-dialog__wrapper" style="z-index: 2001; display: none;">
    </div>
</body>
```

这些都是以前知道的，所以同级元素的变更，会影响v-for的渲染？我做一个元素的删除按钮证明一下就好了

### 测试v-if删除

```vue
<template>
    <div>
        <el-button type="primary" @click="openDialog">打开dialog</el-button>
        <el-button type="primary" @click="updateList">更新list</el-button>
        {{ list }}
        <div v-for="(item, index) of list" :key="index">
            {{ item }}
        </div>
        <div v-if="visible"></div>
    </div>
</template>

<script>

export default {
    data() {
        return {
            list: [0, 1, 2, 3],
            visible: true
        };
    },
    computed: {},
    methods: {
        openDialog() {
            this.visible = false
        },
        updateList() {
            this.list.push(this.list.length)
        }
    },
    created() { },
    mounted() { },
    components: {}
}
</script>
<style lang='less' scoped></style>
```

我们知道v-if在html的表现就是元素的清除，理论上应该与dialog的显隐，在html的表现一致。但是，并不一致！所有的视图更新都是正常的。

### 测试el的删除

element就是用的这种方法

```js
destroyed() {
  // if appendToBody is true, remove DOM node after destroy
  if (this.appendToBody && this.$el && this.$el.parentNode) {
    this.$el.parentNode.removeChild(this.$el);
  }
}
```

```vue
<template>
    <div>
        <el-button type="primary" @click="openDialog">打开dialog</el-button>
        <el-button type="primary" @click="updateList">更新list</el-button>
        {{ list }}
        <div v-for="(item, index) of list" :key="index">
            {{ item }}
        </div>
        <div ref="remove">测试dialog</div>
    </div>
</template>

<script>

export default {
    data() {
        return {
            list: [0, 1, 2, 3],
        };
    },
    computed: {},
    methods: {
        openDialog() {
            const element = this.$refs.remove;
            element.parentNode.removeChild(element);
        },
        updateList() {
            this.list.push(this.list.length)
        }
    },
    created() { },
    mounted() { },
    components: {}
}
</script>
<style lang='less' scoped></style>
```

复现了。

这个时候就有两个问题:

1. v-for如何渲染的，为何同级元素被remove会被影响。
2. v-if到底是如何删除元素的，为何没有影响到v-for，与removeChild有什么区别。

### v-if源码
