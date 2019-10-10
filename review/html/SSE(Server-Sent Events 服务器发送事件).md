## SSE(Server-Sent Events 服务器发送事件)

> SSE是 WebSocket 的一种轻量代替方案，使用 HTTP 协议。 严格地说，HTTP 协议是没有办法做服务器推送的，但是当服务器向客户端声明接下来要发送流信息时，客户端就会保持连接打开，SSE 使用的就是这种原理。 简单的说，就是浏览器向服务器发送一个HTTP请求，然后服务器不断单向地向浏览器推送“信息”（message）。这种信息在格式上很简单，就是“信息”加上前缀“data: ”，然后以“\n\n”结尾。

#### 优点：

- 实现一个完整的服务仅需要少量的代码；
- 可以在现有的服务中使用，不需要启动一个新的服务；
- 可以用任何一种服务端语言中使用；
- 基于 HTTP ／ HTTPS 协议，可以直接运行于现有的代理服务器和认证技术

#### 缺点：

- SSE 是单向通道，只能服务器向客户端发送消息，如果客户端需要向服务器发送消息，则需要一个新的 HTTP 请求。 这对比 WebSocket 的双工通道来说，会有更大的开销。

#### 应用场景：

例如邮箱服务的新邮件提醒，微博的新消息推送、管理后台的一些操作 实时同步等。

#### 服务器

首先向客户端声明接下来发送的是事件流（ text/event-stream ）类型的数据，然后就可以向客户端多次发送消息。 事件流是一个简单的文本流，仅支持 UTF-8 格式的编码。每条消息以一个空行作为分隔符。

```js
const http = require('http');

http.createServer((req, res) => {
	res.writeHead(200, {
		'Content-Type' : 'text/event-stream',
		'Cache-Control' : 'no-cache',
		'Connection' : 'keep-alive',
		'Access-Control-Allow-Origin' : '*'
	});

	var interval = setInterval(() => {
		//设置消息ID
		res.write(`id: ${+new Date()}\n`);
		//设置触发事件 如果没有则默认为message
        res.write("event: foo\n");
		//设置消息数据
		res.write('data: ' + (new Date()) + '\n\n');
		//设置重连时间
		res.write("retry: 10000\n");
        res.write('\n\n'); // 消息结束
	},1000);

	req.connection.addListener('close', () => {
		clearInterval(interval);
	}, false);
}).listen(2000)
```

规范中为消息定义了 4 个字段：

> **event** 消息的事件类型。客户端收到消息时，会在当前的 EventSource 对象上触发一个事件，这个事件的名称就是这个字段的值，如果消息没有这个字段，客户端的 EventSource 对象就会触发默认的 message 事件。 例如：
>
> ```js
> res.write('event: foo\n');
> res.write('data: a foo event\n\n');
> 
> res.write('data: an unnamed event\n\n');
> 
> res.write('event: bar\n');
> res.write('data: a bar event\n\n');
> ```
>
> 上面的代码创造了三条信息。第一条是foo，触发浏览器端的foo事件；第二条未取名，表示默认类型，触发浏览器端的message事件；第三条是bar，触发浏览器端的bar事件。

> **id** 这条消息的 ID。客户端接收到消息后，会把这个 ID 作为内部属性 Last-Event-ID，在断开重连 成功后，会把 Last-Event-ID 发送给服务器。

> **data** 消息的数据字段。 指定具体的消息体，可以是对象或者字符串。 如果数据只有一行，则像下面这样，以“\n\n”结尾：
> data: message\n\n
>
> 如果数据有多行，则最后一行用“\n\n”结尾，前面行都用“\n”结尾：
> data: begin message\n
> data: continue message\n\n
>
> 总之，最后一行的data，结尾要用两个换行符号，表示数据结束。浏览器需要等待数据接收完后才触发事件
>
> 以发送JSON格式的数据为例:
>
> ```
> data: {\n
> data: "foo": "bar",\n
> data: "baz", 555\n
> data: }\n\n
> ```

> **retry** 指定客户端重连的时间。只接受整数，单位是毫秒。如果这个值不是整数则会被自动忽略。浏览器默认的是，如果服务器端三秒内没有发送任何信息，则开始重连。服务器端可以用retry头信息，指定通信的最大间隔时间。

#### 客户端

创建了一个 EventSource 对象，传入参数：url。并且根据服务器的状态和发送的信息作出响应。

```js
window.onload = function() {
	const source = SSE();       //创建EventSource对象 
	if(source) {
		source.addEventListener('open', () => {
			console.log('connected.');
		}, false);

		source.addEventListener('message', e => {
			console.log(`data : ${ e.data}`);
		},false);

		//设置自定义事件
		source.addEventListener('foo', e => {
		    console.log(`foo data : ${ e.data}`);
		},false);
	
		source.addEventListener('error', e => {
		    //判断source.readyState 属性的取值 判断连接的状态
			if(e.target.readyState === EventSource.CLOSED) {
				console.log('disconnected.');
			}else if(e.target.readyState === EventSource.CONNECTING) {
				console.log('connecting');
			}
		}, false)
	}
}

function SSE() {
//检测兼容性
	if(window.EventSource) {
		return (new EventSource('http://localhost:2000'));
	}else {
		console.log('SSE is not supported.');
	}
}
```

#### readyState 取值

- 0，相当于常量EventSource.CONNECTING，表示连接还未建立，或者连接断线。
- 1，相当于常量EventSource.OPEN，表示连接已经建立，可以接受数据。
- 2，相当于常量EventSource.CLOSED，表示连接已断，且不会重连。

#### 事件:

- open ：连接一旦建立，就会触发open事件，可以定义相应的回调函数
- message ： 收到数据就会触发message事件
  - 参数对象event有如下属性：
    - data：服务器端传回的数据（文本格式）。
    - origin： 服务器端URL的域名部分，即协议、域名和端口。
    - lastEventId：数据的编号，由服务器端发送。如果没有编号，这个属性为空。）
- error：如果发生通信错误（比如连接中断），就会触发error事件。
- 自定义事件 ： 服务器可以与浏览器约定自定义事件。

