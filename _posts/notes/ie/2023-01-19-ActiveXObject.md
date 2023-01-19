---
layout: post
title:  ActiveXObject
date:   2023-01-21 11:00:00 +0800
categories: 笔记
tag: IE
---
* content
{:toc}

## ActiveXObject

>警告: 该对象是微软的私有拓展名, 只在微软的IE浏览器上支持, 在win8的应用商店下载的其他浏览器应用也不被支持.
ActiveXObject 启用会返回一个自动化对象的引用

这个对象只能用于实例化自动化对象，它没有任何成员对象.

### 语法

```js
let newObj = new ActiveXObject(servername, typename[, location])  
```

### 参数

* servername
提供对象的应用程序的名称。
* typename
要创建的对象的类型或类。
* location 可选
要创建对象的网络服务器的名称。

### 备注

自动化服务器提供至少一种类型的对象。例如，文字处理应用程序可以提供应用程序对象、文档对象和工具栏对象。

您可以在HKEY_CLASSES_ROOT注册注册表项中识别主机PC上的servername.typename的值。下面是您可以找到的一些示例，它们要取决于你的电脑安装了哪些程序:

```js
Excel.Application

Excel.Chart

Scripting.FileSystemObject

WScript.Shell

Word.Document
```

注意: ActiveX 对象可能会出现安全问题。要使用ActiveXObject, 你可能需要调整IE浏览器的相关安全区域的安全设置。比如说，对于本地局域网，你通常需要将自定义设置更改为"对未标记为可安全执行脚本ActiveX控件执行初始化并执行脚本"。

## 通过WScript.Shell打开chrome浏览器

由于我们开发的插件是基于vue的，兼容不了银行内部低于ie9版本的系统，因此尝试了该方法

```js
function openGoole () {
    /** 如果用户使用IE浏览器，则跳转到Chrome浏览器以获取最佳体验 */
    var userAgent = navigator.userAgent; // 取得浏览器的userAgent字符串
    // 判断是否是IE11以下版本的浏览器
    var isIE = userAgent.indexOf("compatible") > -1 && userAgent.indexOf("MSIE") > -1
    // 判断是否是IE11浏览器
    var isIE11 = userAgent.indexOf("Trident") > -1 && userAgent.indexOf("rv:11.0") > -1
    if(isIE || isIE11) {
        var objShell = new ActiveXObject("WScript.Shell")
        // cmd调用Chrome打开当前网页
        objShell.Run("cmd.exe /c start chrome " + window.location.href, 0, true) 
        /** 关闭当前IE浏览器标签*/
        if(isIE) {
            window.open("", "_self");
            window.close();
        }else{
            window.open("", "_top");
            window.close();
        }
    }
}
```
