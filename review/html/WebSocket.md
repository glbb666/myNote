# `HTML5 WebSocket`

## 什么是`WebSocket`

`WebSocket`是`HTML5`中的协议，支持**持久连接**，`http`协议**不支持持久性连接**。`Http1.0`和`Http1.1`都不支持持久性的连接，`http1.1`中的`keep-alive`是将多个`http`请求合并成一个。无法获知服务器有连续的状态变化。**轮询的效率低，非常浪费资源**（因为短轮询必须不停连接，长轮询HTTP 连接必须始终打开）。`WebSocket` 就是这样发明的。

## `WebSocket`的优势
服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息
![在这里插入图片描述](https://github.com/glbb666/myNote/blob/master/review/html/images/WebSocket_1.png)

## 其他特点包括
（1）建立在 **TCP** 协议上，**服务器端容易**实现。

（2）与 HTTP 协议有着良好的**兼容性**。默认端口也是80和443，并且**握手阶段采用 HTTP** 协议，因此**握手时不容易屏蔽**，能**通过各种 HTTP 代理服务器**。

（3）**数据**格式比较**轻量**（WebSocket的**首部**比http首部**体积小**得多），性能开销小，**通信高效**。

（4）可以发送**文本**，也可以发送**二进制数据**。

（5）**没有同源限制**，客户端可以与任意服务器通信。

（6）**协议标识符**是ws（如果加密，则为wss），服务器网址就是 URL。

（7）**实时性**提高，连接建立可一直保持连接状态，发消息不需要额外的连接建立过程。
![在这里插入图片描述](https://github.com/glbb666/myNote/blob/master/review/html/images/WebSocket_2.png)

## `WebSocket`不会取代`http`

然而，`WebSocket` 在实现高效实时通信的过程却也不再享有在一些本由浏览器提供的**服务和优化，如状态管理、压缩、缓存**等。以后浏览器厂商是不是会针对 `WebSocket `作出一些服务和优化不得而知，但至少现在 `WebSocket` 还完全不足以撼动 `HTTP` 的地位。总的来说，`WebSocket` 弥补了 `HTTP` 在某些通信领域的短板，但绝不可能完全取代 `HTTP`。

## `WebSocket`有哪些`api`

#### 创建webSocket实例

### WebSocket(url: string, protocols?: string[] | string)

- url
  表示要连接的 URL。注意协议名是 `ws` 或者 `wss`。
  
  ```js
  var ws = new WebSocket('ws://localhost:8080');
  ```
  
- protocols
    可以是一个单个的协议名字**字符串**或者包含多个协议名字字符串的**数组**。这些字符串用来表示**子协议**，这样做可以让一个服务器实现多种 WebSocket 子协议（例如你可能希望通过制定不同的协议来处理不同类型的交互）。如果没有制定这个参数，它会默认设为一个空字符串。

`WebSocket`在创建之后就开始和服务端进行握手了。

## 四种状态

#### WebSocket.readyState

-  0 connecting，**正在连接**
- 1 open，**连接成功**，可以通信了
- 2 closing，**正在关闭**
- 3 closed **已经关闭**，或是连接打开失败

## 四个事件

事件遵从原生 Javascript 的两种写法：

- `onevent = handler`
- `addEventListener(event, handler)`

支持的事件有 `open(连接成功)`，`message（收到消息）`，`close（连接关闭）` 和 `error（报错）`。注意 `message` 事件在接收到所有数据时出发。

`hander`就是触发事件之后的回调，一般是长这样

```js
ws.onevent = function(event){
	console.log(event.data)//event.data就是从服务器获取的数据
}
```
用于指定连接关闭后的回调函数
## 两个方法

#### send(data: string | ArrayBuffer | Blob): **void**

实例对象的send()方法用于向服务器发送信息（三种信息）
- 发字符串
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
>其中，字符串是文本信息，ArrayBuffer和blob是二进制信息
#### close(code?: number, reason?: string): void

关闭WebSocket连接或停止正在进行的连接请求。如果连接的状态已经是closed，这个方法不会有任何效果。

- code 表示关闭连接的**状态号**，表示**关闭原因**。如果这个参数没有被指定，默认的取值是 1000 ，即正常连接关闭。更多取值查看 [CloseEvent](<https://developer.mozilla.org/en-US/docs/Web/API/CloseEvent>) 页面。
- reason 一个**描述性**的字符串，表示连接被关闭的原因。

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
### 服务器端

在执行客户端程序前，需要创建一个支持websocket的服务，也就是说需要服务端语言环境支持。我使用nodejs来作为服务端环境，通过安装`nodejs-websocket`模块来支持websocket。
#### 创建连接

```js
var ws = require('nodejs-websocket')//引入模块
var server = ws.createServer(function(connect){
	...（connect用来监听客户端给服务器发送的消息）
})//创建一个新连接
```

#### 接收消息

```js
connect.on("text", function (str) {
    	//其中str为服务器端从客户端接收到的消息
    })
```

#### 关闭连接

```js
connect.on("close",fucntion(code,reason){
	//第一个参数状态码，第二个参数字符串描述
})
```

#### 完整代码

```js
var ws = require("nodejs-websocket"); //引入websocket模块  
console.log("开始建立连接...");//后台打印状态信息  
var server = ws.createServer(function(connect) { //创建一个新连接  
    connect.on("text",function(msg){ //接收触发事件  
        console.log("收到的消息是：" + msg); //接收到新消息之后在后台打印出来  
        if (msg) { //如果消息不是空，将接收到的消息发送给客户端  
            connect.send('服务端已收到消息：' + msg + '服务端发来消息： Hello,' + msg); 
        }  
    });  
    connect.on("close",fucntion(code,reason){
	//第一个参数状态码，第二个参数字符串描述
	})
}).listen(8088);
```

保存为index.js文件，然后运行：node index.js，这个时候服务端就跑起来了，nodejs在监听8088端口。用浏览器打开客户端，输入信息，点击发送试试吧。

当我们输入“李旺”并发送后，服务端收到了内容为“李旺”的消息后，发送了相应的消息给客户端。效果如图：

![图片描述](https://segmentfault.com/img/bV6RkK?w=370&h=172)

## `WebSocket`适用场景

WebSocket 适用于需**要高效实时通信**的场景，比如网页聊天，对战游戏等

### 1. 为什么叫心跳包呢？

它就像心跳一样每隔固定的时间发一次，来告诉服务器，我还活着。

### 2. 心跳机制是？

心跳机制是每隔一段时间会向服务器发送一个数据包，告诉服务器自己还活着，同时客户端会确认服务器端是否还活着，如果还活着的话，就会回传一个数据包给客户端来确定服务器端也还活着，否则的话，有可能是网络断开连接了。需要重连

## 用自己的话归纳一下心跳重连机制的原理
心跳机制是，每隔一段时间向服务器发送一个数据包，告诉服务器自己还活着，同时服务器会给予回应，客户端也能确认服务器是否还活着。如果一定时间内，服务器没有发来回应，那么就会开启重连。如果服务器发来回应，那么就重置心跳检测（重置倒计时）。
## 本文参考
[了解webSocket](https://www.ruanyifeng.com/blog/2017/05/websocket.html)

[使用WebSocket构建实时性应用](<https://juejin.im/post/5a3cb04951882525822793f5>)

[webSocket心跳重连机制](https://www.cnblogs.com/tugenhua0707/p/8648044.html)