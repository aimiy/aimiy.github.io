---
layout: post
title:  call,apply,bind的区别及原理实现
date:   2021-01-21 11:00:00 +0800
categories: 笔记
tag: js
---
* content
{:toc}

## call,apply,bind

由于call()、apply()与bind()都是属于Function.prototype对象下的方法，所以每个function实例都拥有有call、apply与bind属性。
相同点：都是为改变this指向而存在的。
异同点：使用call()方法时，传递给函数的参数必须逐个列举出来，使用apply()方法时，传递给函数的是参数数组。bind()和call()很相似，第一个参数是this的指向，从第二个参数开始是接收的参数列表。bind() 方法不会立即执行，而是返回一个改变了上下文 this后的函数，用于稍后调用。 call()、apply()则是立即调用。

## call实现原理

```js
Function.prototype.mycall = function (context) {
    // 当context为null时，其值则为window
    context = context || window;
    // this为调用mycall的函数。将this赋值给context的fn属性
    context.fn = this;
    // 将arguments转为数组，并从下标1位置开如截取
    let arg = [...arguments].slice(1);
    // 将arg数组的元素作为fn方法的参数执行，结果赋值给result
    let result = context.fn(...arg);
    // 删除fn属性
    delete context.fn;
    // 返回结果
    return result;
}
```

测试：

```js
function add(c, d){
    return this.a + this.b + c + d;
}
var obj = {a:1, b:2};
console.log(add.mycall(obj, 3, 4)); // 10
```

## apply实现原理

```js
Function.prototype.myapply = function (context) {
    // 当context为null时，其值则为window
    context = context || window
    // this为调用myapply的函数。将this赋值给context的fn属性
    context.fn = this;
    // 如果未传值，则为一空数组
    let arg = arguments[1] || [];
    // 将arg数组的元素作为fn方法的参数执行，结果赋值给result
    let result = context.fn(...arg);
    // 删除fn属性
    delete context.fn
    // 返回结果
    return result
}
```

测试：

```js
function add(c, d){
    return this.a + this.b + c + d;
}
var obj = {a:1, b:2};
console.log(add.myapply(obj, [5, 6])); // 14
```

## bind实现原理

```js
Function.prototype.mybind = function (context) {
    // this为调用mybind的函数。将this赋值给变量_this
    let _this = this;
    // 将arguments转为数组，并从下标1位置开如截取
    let arg = [...arguments].slice(1);
    // 返回函数fn
    return function fn(){
        // 通过apply方法调用函数并返回结果。
        return _this.apply(context, arg.concat(...arguments));
    }
}
```

测试：

```js
var obj = {
    siteName: "zhangpeiyue.com"
}
function printSiteName() {
    console.log(this.siteName);
}
var site = printSiteName.mybind(obj);
// 返回的是一个函数
console.log(site) // function () { … }
// 通过mybind使其this发生了变化
site();// zhangpeiyue.com
```
