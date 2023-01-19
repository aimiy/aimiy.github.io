---
layout: post
title:  onbeforeunload事件
date:   2023-01-19 17:50:00 +0800
categories: 
tag: 
---
* content
{:toc}

前段时间遇到一个需求，需要在浏览器关闭前，通过用户确认，继而判断状态来触发事件，于是研究了一下onbeforeunload 事件

## onbeforeunload 事件

事件使网页能够触发一个确认对话框，询问用户是否真的要离开该页面。如果用户确认，浏览器将导航到新页面，否则导航将会取消。

根据规范，要显示确认对话框，事件处理程序需要在事件上调用preventDefault()。

但是请注意，并非所有浏览器都支持此方法，而有些浏览器需要事件处理程序实现两个遗留方法中的一个作为代替：

* 将字符串分配给事件的returnValue属性
* 从事件处理程序返回一个字符串。
* 某些浏览器过去在确认对话框中显示返回的字符串，从而使事件处理程序能够向用户显示自定义消息。但是，此方法已被弃用，并且在大多数浏览器中不再支持。

为避免意外弹出窗口，除非页面已与之交互，否则浏览器可能不会显示在beforeunload事件中创建的提示，甚至根本不会显示它们。

将事件处理程序/监听器加到window或 document的beforeunload事件后，将阻止浏览器使用内存中的页面导航缓存，例如Firefox 的 Back-Forward 缓存或WebKit 的 Page Cache。

HTML 规范指出在此事件中调用window.alert()，window.confirm()以及window.prompt()方法，可能会失效。更多详细信息，请参见HTML 规范。

## 示例

HTML 规范指出作者应该使用 Event.preventDefault() 而非 Event.returnValue，然而，不是所有浏览器都支持这么做。

```js
window.addEventListener('beforeunload', (event) => {
  // Cancel the event as stated by the standard.
  event.preventDefault();
  // Chrome requires returnValue to be set.
  event.returnValue = '';
});
```

## 延时来判断状态

方法是有用的，但是通过开发测试发现，该方法无论用户选择取消，还是确认后刷新，都会触发绑定的事件，且并没有参数方法来传递给开发者用户选择的状态。这无疑与我要达到的需求只差一步。

因此想到了延时来判断用户的选择。用户点击关闭窗口，由于注册了onbeforeunload事件，会弹出弹窗，此时不会触发任何事件。如果用户点击确认，则会执行绑定函数，刷新页面，但由于setTimeout异步运行，又刷新了页面。因此不会执行延时函数中间的事件。如果点击取消，依旧执行绑定函数，由于页面没有关闭也没有刷新，于是延时后，setTimeout中的函数继续执行。

```js
window.onbeforeunload = function unsub(event) {
  if (event && status !== "offwork") {
    that.unSubscribe() // 取消订阅
    setTimeout(() => {
      that.doSubscribe(username) // 订阅
    }, 3000);
  }
  return event;
};
```

## 关于不覆盖第三方的beforeunload绑定事件

有的时候，不仅仅我们要设置关闭浏览器前的事件，第三方插件也会。为了不覆盖别人的事件，需要做一些函数的兼容。

```js
if (window.onbeforeunload) {
  var fn = window.onbeforeunload;
  if (fn.name !== "unsub") {
    window.onbeforeunload = function unsub(event) {
      fn(event);
      if (event && status !== "offwork") {
        that.unSubscribe()
        setTimeout(() => {
          that.doSubscribe(username)
        }, 3000);
      }
      return event;
    };
  }
} else {
  window.onbeforeunload = function unsub(event) {
    let status = CallCenter.getNowStatus()
    that.common.console("坐席状态", status);
    if (event && status !== "offwork") {
      that.unSubscribe()
      setTimeout(() => {
        that.doSubscribe(username)
      }, 3000);
    }
    return event;
  };
}
```

我自己需要绑定的执行函数设为unsub。

1. 先判断window.onbeforeunload是否被定义，没有则onbeforeunload没有被人使用，直接绑定。
2. 如果有，将第三方的绑定函数命为fn，判断fn名称是不是本来就是我们自己的unsub，如果是，则跳过，不是，就意味着有第三方使用过该事件了，需要进行兼容
3. 在自己的unsub函数中先执行第三方的事件`fn(event);`即可，后续执行我们自己的逻辑
