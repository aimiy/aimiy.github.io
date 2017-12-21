---
layout: post
title:  关于requestAnimationFrame的游戏循环与处理用户输入
date:   2017-12-19 17:15:00 +0800
categories: notes
tag: canvas
---

* content
{:toc}


# requestAnimationFrame
使用 requestAnimationFrame 在画布上播放视频只是可以执行的多项互动操作之一。

这一块requestAnimationFrame内容还不是很明白，等日后再找一找资料。

要创建更复杂的应用程序，不仅需要更多地思考我们在屏幕上向用户显示的内容，还要思考用户可能生成的需要我们处理的任何输入（键盘、鼠标、音频）。

游戏循环是使用应用或游戏时连续运行的一系列事件。requestAnimationFrame 会处理大多数繁重的工作，以便确保在活跃查看应用时，应用的运行速度尽可能达到 60 帧/秒。假设我们已经创建了计划调用的函数，具体化的游戏循环可能类似于此：

```
function draw() {
    // request to execute this function at the next earliest convenience
    requestAnimationFrame(draw);
    processInput();
    moveObjectsAndEnemies();
    drawAllTheThings();
}
```
# 处理键盘输入

尽管手动处理键盘流程并不太难，但我更愿意站在巨人的肩膀上，使用具有能帮助我执行所需操作的完善库的开源项目。

在遵循 Kibo 公共命名 ('a', '3', 'up') 的基础上允许你引用按键值而不是它们的键码值，这将会极大地简化你的代码。你也可以在按下或者松开的时候附加事件同样适用于修饰键和通配符。
```
var k = new Kibo();
k.down(['up', 'w'], function() {
    // Do something cool on the canvas
});

k.up(['enter', 'q'], function() {
    // Do other stuff.
});
```

# 处理鼠标输入
就像许多其他的 DOM 元素，画布可以接收 点击 和 鼠标按下事件。我们确实需要做点小工作来确定用户在画布上的哪个位置点击了。鼠标点击事件返回 clientX 和 clientY 在浏览器窗口全局中的位置。每个元素知道它相对于浏览器 (0,0) 即 (offsetLeft和offsetTop)的位置。要获取一次相对于画布的点击，你需要从 clientX 和 clientY 中减去 offsetLeft 和 offsetTop 值。 查看下面的示例代码:
```
var c = document.querySelector("canvas");

function handleMouseClick(evt) {
        x = evt.clientX - c.offsetLeft;
        y = evt.clientY - c.offsetTop;
        console.log("x,y:"+x+","+y);
}
c.addEventListener("click", handleMouseClick, false);
```
[Kibo.js](https://github.com/marquete/kibo) - 处理键盘输入的一个 JavaScript 库。
