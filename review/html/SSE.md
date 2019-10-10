# HTML5 SSE
[Server-Sent Events 教程](https://www.ruanyifeng.com/blog/2017/05/server-sent_events.html)
[详解Web端通信方式的演进：从Ajax、JSONP 到 SSE、Websocket](https://juejin.im/entry/59b78393f265da066c22e9d3)

## 什么是SSE
用于服务器主动向客户端发送消息，通常用于 **推送**。

> 使用 HTTP 轮询也可以达到目标，不过需要客户端 **定期的** 给服务器发送请求，而 SSE只要创立了连接，服务器之后就会主动的给客户端发送请求，客户端主需要处理就行。

## 使用方法

1. 创建连接

```js
  let evtSource = new EventSource(serverURL);
```

2. 监听事件

```js
  evtSource.onmessage = function(e) {
    var newElement = document.createElement("li");
    var eventList = document.getElementById('list');

    newElement.innerHTML = "message: " + e.data;
    eventList.appendChild(newElement);
  }
```

> 监听了那些从服务器发送来的所有 **没有指定事件类型** 的消息(没有 event 字段的消息),然后把消息内容显示在页面文档中。

3. 监听指定事件

```js
  evtSource.addEventListener("ping", function(e) {
    var newElement = document.createElement("li");
    
    var obj = JSON.parse(e.data);
    newElement.innerHTML = "ping at " + obj.time;
    eventList.appendChild(newElement);
  }, false);
```

> 只有在服务器发送的消息中包含一个值为 "ping" 的 event 字段的时候才会触发对应的处理函数,也就是将 data 字段的字段值解析为 JSON 数据,然后在页面上显示出所需要的内容。

1. 服务器端发送的响应内容应该使用值为 "text/event-stream" 的媒体类型。

## API

SSE 的客户端 API 部署在EventSource对象上。下面的代码可以检测浏览器是否支持 SSE。
```js
if ('EventSource' in window) {
  // ...
}
```
### 创建
```js
var source = new EventSource(url);
```
### 三种状态

#### EventSource.readyState
- 0：相当于常量EventSource.connecting，**连接未建**，或者断线正在重连。
- 1：相当  于常量EventSource.open，**连接已建**，可以接受数据。
- 2：相当于常量EventSource.closed，**连接已断**，且不会重连。
### 三种事件

`open(连接建立)`,`message(收到服务器数据) `,`error(通信错误/连接中断)`

参数对象`event`有如下属性：

- data：服务器端传回的数据（文本格式）。
- origin： 服务器端URL的域名部分，即协议、域名和端口。
- lastEventId：数据的编号，由服务器端发送。如果没有编号，这个属性为空。

## 和webSocket的异同以及优势

- 相同点
  都是建立浏览器与服务器之间的通信渠道，然后服务器向浏览器推送信息。
- 不同点
  WebSocket 更强大和灵活。因为它是全双工通道，可以双向通信；SSE 是单向通道，只能服务器向浏览器发送，因为流信息本质上就是下载。如果浏览器向服务器发送信息，就变成了另一次 HTTP 请求。
- 优势
  - SSE **使用 HTTP 协议**，现有的服务器软件都支持。WebSocket 是一个独立协议。
  - SSE 属于**轻量级**，使用简单；WebSocket 协议相对复杂。
  - SSE 默认**支持断线重连**，WebSocket 需要自己实现。
  - SSE 一般只用来传送文本，二进制数据需要编码后传送，WebSocket 默认支持传送二进制数据。

## 和ajax的区别

- 数据类型不同: SSE **只能接受 type/event-stream** 类型，AJAX 可以接受任意类型；
- 结束机制不同: 虽然使用AJAX长轮询也可以实现这样的效果，但是服务器端(以nodeJS为例)必须在一定时间内执行res.end()才行。而SSE只需要执行**res.write()** 即可。