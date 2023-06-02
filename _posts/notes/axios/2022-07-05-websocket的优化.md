---
layout: post
title:  websocket的优化
date:   2021-01-21 11:00:00 +0800
categories: 笔记
tag: websocket
---
* content
{:toc}

## 验签鉴权及对应的容错策略（登录态要求、峰值访问、服务端宕机异常）

背景与目的：

websocket 握手后，接口请求即可以放弃 HTTP 改走 weboskcet，但大部分业务接口都要求登录态，因此握手成功后必须先走一次签名鉴权，获取登录态
当出现大流量访问的场景（如大促、热点活动等）或服务端出 bug 而导致服务端宕机，前端会做 对应容错，将位于内存的等待队列中的待发送请求立即降级成 HTTP 发送出去

```js
SocketTask.onOpen(function () {
  SocketTask.sendSocketMessage({
     msg_type: '验签'，
     token: 'xxx'
  }, (response) => {
      console.log(response.user_id, response.access_token)

      // 通道可用，打个标记
      global.isSocketAvaliable = true;
  })
})
```

## 心跳保活（减少 TCP 占用）

背景与目的：为了减少 TCP 连接的无效占用，客户端定时发送一个空包到服务端，告知服务端不要销毁这条 socket，如果服务端超过一定时间都没收到心跳包，则将关闭并销毁该 socket

伪码示意：

```js
SocketTask.onOpen(function () {
  SocketTask.sendSocketMessage({
     msg_type: '验签'，
     token: 'xxx'
  }, (response) => {
      console.log(response.user_id, response.access_token)

      // 通道可用，打个标记
      global.isSocketAvaliable = true;
      
      // 验签成功，开始定时发送心跳包
      setInterval(() => {
          SocketTask.sendSocketMessage({
            msg_type: '心跳'
          });
      });
   });
})
```

### 心跳监测

心跳保活这里可以做一个心跳监测，客户端发送ping，后端回应pong，在发送ping后给一个定时器，定时器做的操作是，一段时间后，关闭这个连接。收到pong后清除这个定时器，不进行关闭操作。

因此在一定时间内收不到pong的时候，视为网络断开了，通过关闭来触发重连操作。

## 模拟 RTT（用于弱网体验优化）

背景与目的：在发送心跳包时，可得知一个心跳包的 RTT，以此模拟当前用户网络环境的 TCP RTT，并据此计算出平滑 RTO，用于弱网体验优化

```js
SocketTask.onOpen(function () {
  SocketTask.sendSocketMessage({
     msg_type: '验签'，
     token: 'xxx'
  }, (response) => {
      console.log(response.user_id, response.access_token)

      // 通道可用，打个标记
      global.isSocketAvaliable = true;
      
      // 验签成功，开始定时发送心跳包
      setInterval(() => {
          // 计算 RTT
          const begin = Date.now();

          SocketTask.sendSocketMessage({
            msg_type: '心跳'
          }, () => {
            const end = Date.now();
            
            const RTT = begin - end;
            
            const smoothedRTO = cal(RTT);
            
            global.smoothedRTO = smoothedRTO;
          });
      });
   });
});
```

### Snappy 压缩（横向对比了 gzip / zip / 7z）

背景与目的：在小程序中引入第三方压缩包（牺牲小程序包体积），减少 websocket 传输的字节数

```js
  import Snappy from 'snappy';

  SocketTask.sendSocketMessage = function (msg) {
     const encryptedMsg = Snappy.encode(msg);

     wx.send(encryptedMsg);
  }
```

### 重连（阶梯式错位重连，避免拥挤）

背景与目的：用户的网络环境不稳定，可能会存在主动 / 被动断开 socket 的情况，需要进行自动重连

```js
SocketTask.onClose(function () {
  // 限定最大重连次数
  if (retryCount > maxCount) {
    return;
  }
  
  retryCount++;

  setTimeout(() => {
    SocketTask.connectSocket();
  }, retryCount *1000 + Math.random()* 1000);
});
```

### 埋点中间层缓存（重复的用户信息可以不用每次都上报，支持刷新缓存）

背景与目的：为减少网络传输的包体积，通过 websocket 上报埋点日志时，可以把部分重复字段值在第一次上报时缓存在服务端，从第二次上报开始只上报值不重复的字段，然后由服务端做日志合并

伪码示意：

```js
SocketTask.sendSocketMessage({
     msg_type: '埋点日志'，
     logs: {
       country: 'China', // 可缓存字段
       city: '北京', // 可缓存字段
       platform: '安卓', // 可缓存字段
       click_some_btn: true // 动态变化的埋点字段
     },
     cacheFields: ['country', 'city', 'platform'] // 只在第一次上报时携带
 });
 ```

### 启用 TCP_NODELAY

TCP_NODELAY 是用来禁用 Nagle 算法的。Nagle 算法设计的目的是提高网络带宽利用率，其核心思路是「合并小的 TCP 包为一个大的 TCP 包」，避免过多的小包的 TCP 头部浪费网络带宽
