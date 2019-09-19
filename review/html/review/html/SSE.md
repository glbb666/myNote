# HTML5 SSE
[Server-Sent Events 教程](https://www.ruanyifeng.com/blog/2017/05/server-sent_events.html)
[详解Web端通信方式的演进：从Ajax、JSONP 到 SSE、Websocket](https://juejin.im/entry/59b78393f265da066c22e9d3)
## 什么是SSE
SSE使用的是http协议，该协议无法做到服务器主动推送信息。但是有一种变通方法，服务器向客户端声明，接下来发送的是流信息。
## 和webSocket的异同以及优势
- 相同点
都是建立浏览器与服务器之间的通信渠道，然后服务器向浏览器推送信息。
- 不同点
WebSocket 更强大和灵活。因为它是全双工通道，可以双向通信；SSE 是单向通道，只能服务器向浏览器发送，因为流信息本质上就是下载。如果浏览器向服务器发送信息，就变成了另一次 HTTP 请求。
- 优势
	+ SSE **使用 HTTP 协议**，现有的服务器软件都支持。WebSocket 是一个独立协议。
	+ SSE 属于**轻量级**，使用简单；WebSocket 协议相对复杂。
	+ SSE 默认**支持断线重连**，WebSocket 需要自己实现。
	+ SSE 一般只用来传送文本，二进制数据需要编码后传送，WebSocket 默认支持传送二进制数据。
## 和ajax的区别
-  数据类型不同: SSE **只能接受 type/event-stream** 类型，AJAX 可以接受任意类型；
- 结束机制不同: 虽然使用AJAX长轮询也可以实现这样的效果，但是服务器端(以nodeJS为例)必须在一定时间内执行res.end()才行。而SSE只需要执行**res.write()** 即可。
## SSE的API
SSE 的客户端 API 部署在EventSource对象上。下面的代码可以检测浏览器是否支持 SSE。
```js
if ('EventSource' in window) {
  // ...
}
```
#### 创建
```js
var source = new EventSource(url);
```
#### EventSource.readyState
- 0：相当于常量EventSource.CONNECTING，表示连接还未建立，或者断线正在重连。
- 1：相当于常量EventSource.OPEN，表示连接已经建立，可以接受数据。
- 2：相当于常量EventSource.CLOSED，表示连接已断，且不会重连。
#### EventSource.onopen
连接一旦建立，就会触发open事件，可以在onopen属性定义回调函数。
```js
source.onopen = function (event) {
  // ...
};
```
#### EventSource.onmessage
客户端收到服务器发来的数据，就会触发message事件，可以在onmessage属性的回调函数。
```js
source.onmessage = function (event) {
  var data = event.data;
  // handle message
};
```
上面代码中，事件对象的data属性就是服务器端传回的数据（文本格式）。
#### EventSource.onerror
如果发生通信错误（比如连接中断），就会触发error事件，可以在onerror属性定义回调函数。
```js
source.onerror = function (event) {
  // handle error event
};
```

