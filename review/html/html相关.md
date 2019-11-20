# 对HTML语义化的理解（语义化标签有哪些）
[对html语义化的了解](https://blog.csdn.net/eeeecw/article/details/80591511)

# div布局是否符合html语义化（为什么要用div布局）
[div布局和table布局的比较](https://blog.csdn.net/qq_40128682/article/details/89883319)
不符合。
- 速度快，增强用户体验。DIV+CSS布局较Table布局减少了页面代码，加载速度得到很大的提高。
-  符合w3c标准，在w3c标准中，table是用来存储数据的。
- 布局更加灵活多样，能够通过样式选择来实现界面设计方面的更多要求。。
- 布局改版方便，不需要过多地变动页面内容，通常只要更换相应的css样式就可以将网页变成另外一种风格展现出来。
- 布局可以让一些重要的链接和文字信息等优先让搜索引擎抓取，内容更便于搜索。
- table的表格嵌套问题，搜索引擎一般不抓取三层以上的表格嵌套。所以为了提升网站排名，使用div布局。
# HTML本地存储(localStorage、sessionStorage)
[cookies、sessionStorage和localStorage解释及区别](https://www.cnblogs.com/pengc/p/8714475.html)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190909222836947.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dhbmx1YmFiYTY2Ng==,size_16,color_FFFFFF,t_70)
## webStorage分为哪两类，有什么区别
- localStorage（本地存储）：将数据保存在客户端本地的硬件设备(通常指硬盘，也可以是其他硬件设备)中，**即使浏览器被关闭了，该数据仍然存在**，下次打开浏览器访问网站时仍然可以继续使用。
- sessionStorage（会话存储）：将数据保存在session对象中。所谓session，是指用户在浏览某个网站时，**从进入网站到浏览器关闭所经过的这段时间**，也就是用户浏览这个网站所花费的时间。session对象可以用来保存在这段时间内所要求保存的任何数据。
这**两者的区别**在于，sessionStorage为临时保存，而localStorage为永久保存。
## webStorage的优点
（1）**存储空间更大**：cookie为4KB，而WebStorage是5MB；

（2）**节省网络流量**：WebStorage不会传送到服务器，存储在本地的数据可以直接获取，也不会像cookie一样每次请求都会传送到服务器，所以减少了客户端和服务器端的交互，节省了网络流量；

（3）对于那种只需要在用户浏览一组页面期间保存而关闭浏览器后就可以丢弃的数据，sessionStorage会非常方便；

（4）**快速显示**：有的数据存储在WebStorage上，再加上浏览器本身的缓存。获取数据时可以从本地获取会比从服务器端获取快得多，所以速度更快；

（5）**安全性**：WebStorage不会随着HTTP header发送到服务器端，所以安全性相对于cookie来说比较高一些，不会担心截获，但是仍然存在伪造问题；

（6）WebStorage提供了一些方法，数据**操作比cookie方便**；

```
　　　      setItem (key, value) ——  保存数据，以键值对的方式储存信息。
  
      　　  getItem (key) ——  获取数据，将键值传入，即可获取到对应的value值。

        　　removeItem (key) ——  删除单个数据，根据键值移除对应的信息。

        　　clear () ——  删除所有的数据

        　　key (index) —— 获取某个索引的key
```
## webStorage的特点
- 存储位置：localStorage和sessionStorage都保存在客户端，不与服务器进行交互通信。
- 存储内容类型：localStorage和sessionStorage只能存储字符串类型，对于复杂的对象可以使用ECMAScript提供的JSON对象的stringify和parse来处理
- 获取方式：localStorage：window.localStorage，sessionStorage：window.sessionStorage。
- 应用场景：localStoragese：常用于长期登录（+判断用户是否已登录），适合长期保存在本地的数据。sessionStorage：敏感账号一次性登录
# HTML5 WebSocket
[了解webSocket](https://www.ruanyifeng.com/blog/2017/05/websocket.html)
[webSocket心跳重连机制](https://www.cnblogs.com/tugenhua0707/p/8648044.html)
## 为什么需要WebSocket
因为 HTTP 协议有一个缺陷：通信只能由客户端发起。这种单向请求的特点，注定了如果服务器有连续的状态变化，客户端要获知就非常麻烦。我们只能使用"轮询"来了解服务器有没有新的信息。最典型的场景就是聊天室。
轮询的效率低，非常浪费资源（因为短轮询必须不停连接，长轮询HTTP 连接必须始终打开）。因此，工程师们一直在思考，有没有更好的方法。WebSocket 就是这样发明的。
## WebSocket的优势
服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190916193825136.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dhbmx1YmFiYTY2Ng==,size_16,color_FFFFFF,t_70)
## 其他特点包括
（1）建立在 **TCP** 协议之上，服务器端的实现比较容易。

（2）与 HTTP 协议有着良好的**兼容性**。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。

（3）数据格式比较**轻量**，性能开销小，通信高效。

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
#### webSocket.readyState
-  0 connecting，正在连接
- 1 open，连接成功，可以通信了
- 2 closing，连接正在关闭
- 3 closed 连接已经关闭，或是连接打开失败
#### webSocket.onopen
用于指定连接成功后的回调函数
```js
ws.onopen = function(){
	ws.send('hello server');
}
```
#### webSocket.onclose
用于指定连接关闭后的回调函数
```js
ws.onclose = function(event) {
  var code = event.code;
  var reason = event.reason;
  var wasClean = event.wasClean;
  // handle close event
};
```
#### webSocket.onmessage
用于指定接收到服务器数据后的回调函数
```js
ws.onmessage = function(event) {
  var data = event.data;
  // 处理数据
};
```
#### webSocket.send
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
#### webSocket.bufferedAmount
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
#### webSocekt.onerror
用于指定报错时的回调函数
```js
socket.onerror = function(event) {
  // handle error event
};
```
## 为什么需要心跳重连机制
遇到网络断开的情况时，服务器没有触发onclose的事件，这样服务器就会继续给客户端发送多余的链接，并且这些数据还会丢失。所以就需要一种机制来检测客户端和服务器是否处于正常的连接状态。因此就有了websocket的心跳，有心跳，说明还活着，没有心跳说明已经死了。
## 用自己的话归纳一下心跳重连机制的原理
心跳机制是，每隔一段事件向服务器发送一个数据包，告诉服务器自己还活着，同时服务器会给予回应，客户端也能确认服务器是否还活着。如果一定时间内，服务器没有发来回应，那么就会开启重连。如果服务器发来回应，那么就重置心跳检测（重置倒计时）。
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