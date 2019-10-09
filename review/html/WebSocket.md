# HTML5 WebSocket
[了解webSocket](https://www.ruanyifeng.com/blog/2017/05/websocket.html)
[webSocket心跳重连机制](https://www.cnblogs.com/tugenhua0707/p/8648044.html)
## 为什么需要WebSocket
因为 HTTP 协议有一个缺陷：通信只能由客户端发起。这种单向请求的特点，注定了如果服务器有连续的状态变化，客户端要获知就非常麻烦。我们只能使用"轮询"来了解服务器有没有新的信息。最典型的场景就是聊天室。
**轮询的效率低，非常浪费资源**（因为短轮询必须不停连接，长轮询HTTP 连接必须始终打开）。因此，工程师们一直在思考，有没有更好的方法。WebSocket 就是这样发明的。

## WebSocket的优势
服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190916193825136.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dhbmx1YmFiYTY2Ng==,size_16,color_FFFFFF,t_70)

## 其他特点包括
（1）建立在 **TCP** 协议之上，**服务器端容易**实现。

（2）与 HTTP 协议有着良好的**兼容性**。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。

（3）**数据**格式比较**轻量**，性能开销小，**通信高效**。

（4）可以发送**文本**，也可以发送**二进制数据**。

（5）**没有同源限制**，客户端可以与任意服务器通信。

（6）协议标识符是**ws（如果加密，则为wss）**，服务器网址就是 URL。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190916194422868.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dhbmx1YmFiYTY2Ng==,size_16,color_FFFFFF,t_70)
## WebSocket有哪些api
#### 创建webSocket实例
```js
var url = '';
var ws =  new WebSocket(url);
```
webSocket在创建之后就开始和服务端进行握手了。

## 四种状态

#### WebSocket.readyState

-  0 connecting，**正在连接**
- 1 open，**连接成功**，可以通信了
- 2 closing，**正在关闭**
- 3 closed **已经关闭**，或是连接打开失败

## 四个事件

#### WebSocket.onopen
用于指定连接成功后的回调函数
```js
ws.onopen = function(){
	ws.send('hello server');
}
```
#### WebSocket.onclose
用于指定连接关闭后的回调函数
```js
ws.onclose = function(event) {
  var code = event.code;
  var reason = event.reason;
  var wasClean = event.wasClean;
  // handle close event
};
```
#### WebSocket.onmessage
用于指定接收到服务器数据后的回调函数
```js
ws.onmessage = function(event) {
  var data = event.data;
  // 处理数据
};
```
#### WebSocekt.onerror
用于指定报错时的回调函数
```js
socket.onerror = function(event) {
  // handle error event
};
```
## 三种信息

#### WebSocket.send

实例对象的send()方法用于向服务器发送信息
- 发文本
```js
ws.send('your message')
```
- 发blob对象
```js
var file = document
  .querySelector('input[type="file"]')
  .files[0];
ws.send(file);
```
- 发ArrayBuffer(二进制数据缓冲区)
```js

// Sending canvas ImageData as ArrayBuffer
var img = canvas_context.getImageData(0, 0, 400, 320);
var binary = new Uint8Array(img.data.length);
for (var i = 0; i < img.data.length; i++) {
  binary[i] = img.data[i];
}
ws.send(binary.buffer);
```
#### WebSocket.bufferedAmount
实例的bufferedAmount属性，表示还有多少字节的二进制数据还没有发送出去，它可以用来判断发送是否结束
```js
var data = new ArrayBuffer(10000000);
socket.send(data);

if (socket.bufferedAmount === 0) {
  // 发送完毕
} else {
  // 发送还没结束
}
```
## 为什么需要心跳重连机制
遇到网络断开的情况时，服务器没有触发onclose的事件，这样服务器就会继续给客户端发送多余的链接，并且这些数据还会丢失。所以就需要一种机制来检测客户端和服务器是否处于正常的连接状态。因此就有了websocket的心跳，有心跳，说明还活着，没有心跳说明已经死了。
## 用自己的话归纳一下心跳重连机制的原理
心跳机制是，每隔一段事件向服务器发送一个数据包，告诉服务器自己还活着，同时服务器会给予回应，客户端也能确认服务器是否还活着。如果一定时间内，服务器没有发来回应，那么就会开启重连。如果服务器发来回应，那么就重置心跳检测（重置倒计时）。