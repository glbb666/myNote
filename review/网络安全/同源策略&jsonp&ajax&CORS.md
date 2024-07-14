[前端跨域整理](https://juejin.im/post/5815f4abbf22ec006893b431)

[由同源策略到前端跨域](https://juejin.im/post/58f816198d6d81005874fd97)

[九种跨域方式实现原理（完整版）](https://juejin.im/post/5c23993de51d457b8c1f4ee1#heading-2)

[Jquery ajax, Axios, Fetch区别](https://juejin.im/post/5acde23c5188255cb32e7e76)

[CORS 简单请求+预检请求（彻底理解跨域）](https://github.com/amandakelake/blog/issues/62)

### 同源策略和跨域？

> 如果两个页面的协议，端口和主机都相同，则两个页面具有相同的**源**。不满足同源策列，就会产生跨域问题。谷歌浏览器跨域请求不会发起，其他浏览器可发送跨域请求，但是相应会被浏览器拦截。

🌟但是有三种标签允许跨域加载资源

- `<img src=XXX>`
- `<link href=XXX>`
- `<script src=XXX>`

### AJAX

#### `AJAX`是什么

`AJAX`的核心是 `XMLHttpRequest`对象，它不必刷新页面就可以从服务器获取到新数据。

- 我们先创建一个 `XMLHttpRequest`对象，并且监听它的 `onreadyStatuschange`事件，当 `readyStatus`的值为4，说明已经收到完整响应了。
- 我们再判断 `status`（状态码），当它的值为 `200~300`或 `304`的时候，就从 `responseText`中读取服务器相应内容的文本。
- 接着我们用 `open`启动一个请求，为请求指定 `url`地址，方法，同步性。
  - `get`请求的 `Content-type`默认为 `text/plain`。
  - 非 `get`请求，要 `setRequestHeader`，设置 `Content-type`的值。
- 我们还可以用 `timeout`设置超时，并且在 `ontimeout`中调用 `abort`取消请求。用 `onerror`处理请求错误的情况。
- 接着我们调用 `send`方法，在 `send`方法执行之前，`AJAX`都是同步的。`send`方法执行过后，浏览器就会为 `http`请求创建一个 `http`请求线程，这个请求独立于 `js`引擎线程，是异步的，`js`引擎并不会等待这个异步请求的结果，而是会继续执行下去。当浏览器收到服务器的响应，浏览器事件触发线程就会捕获到 `ajax`的回调事件 `onreadyStatechange`或者是 `onerror`事件，浏览器事件触发线程会把回调事件添加到任务队列末尾，直到 `js`引擎线程空闲，任务队列的任务才会被添加到执行栈中依次执行。

#### 原生ajax发起请求

```javascript
function getParams(data) {
  let arr = []
  for (let i in data) {
    arr.push(encodeURIComponent(i) + '=' + encodeURIComponent(data[i]))
  }
  return arr.join('&')
}
function Ajax(options = {}) {
  let {type='GET',async=true,timeout=8000,data,url} = options;
  const xhr = new XMLHttpRequest();
  xhr.onreadystatechange = function(e) {
    if (xhr.readyState === 4) {
      if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304) {
        console.log(xhr.responseText)
      }
    }
  }
  xhr.timeout = timeout
  xhr.ontimeout = function(e) {
    xhr.abort()//终止请求
    console.log(xhr.readyStatus)//0
  }
  if (type === 'GET') {
    xhr.open(
      'GET',
      url + '?' + getParams(data),
      async
    )
    xhr.send(null)
  } else if (type === 'POST') {
    xhr.open('POST', url, async)
    xhr.setRequestHeader(
      'Content-Type',
      'application/x-www-form-urlencoded'
    )
    xhr.send(data)
  }else{
      return;
  }
}
```

#### 用 `promise`封装 `ajax`

```javascript
function getParams(data) {
  let arr = []
  for (let i in data) {
    arr.push(encodeURIComponent(i) + '=' + encodeURIComponent(data[i]))
  }
  return arr.join('&')
}
function Ajax(options = {}) {
  let {type='GET',async=true,timeout=8000,data,url} = options;
  return new Promise((resolve,reject)=>{
      const xhr = new XMLHttpRequest();
	  xhr.onreadystatechange = function(e) {
        if (xhr.readyState === 4) {
          if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304) {
            resolve(xhr.responseText)
          }
        }
      }
      // 超时处理
      xhr.timeout = timeout
      xhr.ontimeout = function(e) {
        xhr.abort()//终止请求
        reject('timeout')
      }
      if (type === 'GET') {
        xhr.open(
          'GET',
          url + '?' + getParams(data),
          async
        )
        xhr.send(null)
      } else if (type === 'POST') {
        xhr.open('POST', url, async)
        xhr.setRequestHeader(
          'Content-Type',
          'application/x-www-form-urlencoded'
        )
        xhr.send(data)
      }else{
          reject('invalid method')
      }
  })
}
```

#### AJAX返回的状态

| 状态码 | 含义                                         |
| ------ | -------------------------------------------- |
| 0      | 未初始化。未调用open()                       |
| 1      | 启动。已调用open()，未调用send()             |
| 2      | 发送。调用send(),但未收到响应                |
| 3      | 接收。受到部分响应数据                       |
| 4      | 完成。接收全部响应数据，且可以在客户端使用。 |

🌟注意：`send`之前，都是同步的

#### 如何发出一个有序的 `AJAX`

回调函数，`Promise.then`，`async`

#### `ajax`和 `jquery`, `Fecth`,`Axios`比有什么区别

##### `jquery`

是对原生 `XMLHttpRequest`对象的封装,还增添了对 `jsonp`的支持,但是 `jquery`整个项目太大了,单纯使用 `ajax`就要引入整个 `jquery`非常的不合理

##### `fecth`

优点:基于 `promise`，支持 `node`,比较轻量

缺点：

- `promise`的状态，除了**网络故障或者请求被阻止**，其他情况下 `fecth`返回的 `promise`状态都为 `resolve`(但是会将 resolve 的返回值的 `ok` 属性设置为 `false` )。
- `cookie`，默认情况下，`fecth`不会从服务器接受或发送 `cookie`，需要设置 `credenitials`选项。

  所以需要我们再次封装

> fetch发送post请求的时候，总是发送两次，第一次状态码是204，第二次才成功？
>
> 因为第一次发送Options请求，询问服务器是否支持修改的请求头，如果服务器支持，则在第二次中发送真正的请求。

##### `axios`

也是对原生 `XMLHttpRequest`对象的封装,但是是 `promise`的实现版本,它有以下功能

- 从 `node.js`创建 `http` 请求
- 支持 ` Promise API`
- 客户端支持防止 `CSRF`
- **提供了一些并发请求的接口**（重要，方便了很多的操作）

```javascript
axios.all([getUserAccount(), getUserPermissions()])
  .then(axios.spread(function (acct, perms) {
    // Both requests are now complete
  }));
//并发案例
```

### 单向跨域

#### 1.`JSONP`

##### 原理

`ajax`受同源策略的影响，不允许进行跨域请求。`JSONP`利用 `script`标签中的 `src`属性可以访问跨域的静态资源的特性，动态插入 `script`标签，客户端通过 `get`请求，向服务器发送回调函数的名称，同时在 `window`对象中设置回调函数。服务端返回一段调用回调函数的 `js`代码，把参数放到回调函数中。通过 `script`标签加载执行，从而实现跨域。

##### 过程

- 客户端利用 `script`标签可以跨域请求资源的性质，向网页中动态插入 `script`标签，来向服务器请求数据
- 服务器会解析请求的 `url`，从里面取出 `callback`，然后把数据放入 `callback`中返回给客户端

##### `JSONP`的优缺点

优点：

- 不受到同源策略的限制
- 兼容性更好，不需要 `XMLHttpRequest`或

缺点：

- 只支持 `GET`请求，因为 `script`标签只能使用 `get`
- 没有超时处理
- 需要和后端协商

##### 实现一个 `JSONP`

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>JSONP Example</title>
</head>
<body>
    <div id="result"></div>

    <script>
        // 定义回调函数
        function handleResponse(data) {
            document.getElementById('result').innerText = JSON.stringify(data);
        }

        // 创建 script 标签
        var script = document.createElement('script');
        script.src = 'http://example.com/data?callback=handleResponse';
        document.head.appendChild(script);
    </script>
</body>
</html>
```

#### 2. `CORS(Cross-Origin Resource Sharing)跨域资源共享`

`CORS`允许服务器声明哪些源站通过浏览器有权限访问哪些资源。

基本使用

```js
server.all('*',function(req,res,next){
  
    //其中`*` 表示通配, 所有的域都能访问此资源
    res.header("Access-Control-Allow-Origin",'*');
    //只允许B站访问
    res.header("Access-Control-Allow-Origin",<B-DOMAIN>)
   
    //CORS需要指定METHOD访问, 对于GET和POST请求, 至少要指定以下三种methods, 如下:
    res.header("Access-Control-Allow-Methods","PUT,POST,GET,DELETE,OPTIONS");
  
    //如果是POST请求, 且提交的数据类型是json, 那么, CORS需要指定headers.
    res.header("Content-Type", "application/json;charset=utf-8");
  
    //CORS默认是不带cookie的, 设置以下字段将允许浏览器发送cookie
    res.header('Access-Control-Allow-Credentials', true);
  
    next()
})
```

##### 前端携带 `cookie`

后端需要如此设置

- `Access-Control-Allow-Origin` 字段将不允许设置为 `*`，需要明确指定与请求网页一致的域名
- `res.header('Access-Control-Allow-Credentials', true);`

同时，前端需要做如下显式设置才能真正发送 `cookie`

```
xhr.withCredentials = true;
```

##### `简单请求`和 `预检请求（preflighted requests）`

对于非简单请求，**浏览器必须首先使用 `OPTIONS`方法发起一个预检请求，从而获知服务端是否允许该跨域请求(同源不会发送 `OPTIONS`)。**

**服务器确认允许之后，浏览器会记住，然后再发起实际的 HTTP 请求，且之后每次就都直接请求而不用再询问服务器是否可以跨域**。在预检请求的返回中，服务器端也可以通知客户端，是否需要携带身份凭证（包括 Cookies 和 HTTP 认证相关数据）。

![2ec957a3-b220-49c5-8fa1-59a82a030e89](images/50205846-accac300-03a4-11e9-8654-2d646d237820.png)

**简单请求就是不会触发CORS预检的请求**，满足以下**所有条件**的才会被视为简单请求，基本上我们日常开发只会关注前面两点

1. 使用 `GET、POST、HEAD`其中一种方法
2. 只使用了如下的安全首部字段，不得人为设置其他首部字段
   - `Accept`
   - `Accept-Language`
   - `Content-Language`
   - `Content-Type`仅限以下三种

     - `text/plain`
     - `multipart/form-data`
     - `application/x-www-form-urlencoded`
   - HTML头部header field字段：`DPR、Download、Save-Data、Viewport-Width、WIdth`
3. 请求中的任意 `XMLHttpRequestUpload` 对象均没有注册任何事件监听器；XMLHttpRequestUpload 对象可以使用 XMLHttpRequest.upload 属性访问
4. 请求中没有使用 ReadableStream 对象

预检请求要求必须**首先使用 `OPTIONS` 方法发起一个预检请求到服务器，以获知服务器是否允许该实际请求**。"预检请求“的使用，可以避免跨域请求对服务器的用户数据产生未预期的影响

非简单请求就是会触发的预检请求

1. 使用了 `PUT、DELETE、CONNECT、OPTIONS、TRACE、PATCH`方法
2. 人为设置了非规定内的其他首部字段，参考上面简单请求的安全字段集合，还要特别注意 `Content-Type`的类型
3. `XMLHttpRequestUpload` 对象注册了任何事件监听器
4. 请求中使用了 `ReadableStream`对象

##### 完整请求流程

![img](images/50205881-c409b080-03a4-11e9-8a57-a2a6d0e1d879.png)

##### `CORS`和 `JSONP`对比

- 请求类型：`JSONP`只能实现 `GET`请求，而 `CORS`支持所有类型的 `HTTP`请求。
- 错误处理：使用 `CORS`，开发者可以使用普通的 `XMLHttpRequest`发起请求和获得数据，比起 `JSONP`有更好的错误处理。
- `JSONP`主要被老的浏览器支持，它们往往不支持 `CORS`，而绝大多数现代浏览器都已经支持了 `CORS`。

`CORS`与 `JSONP`相比，无疑更为先进、方便和可靠。

### 3. **代理服务器** ：

* 使用服务器端代理来解决跨域问题。前端请求发送到同源的代理服务器，由代理服务器再请求目标资源，并将结果返回给前端。
* 优点：可以解决复杂的跨域问题，并能处理各种请求类型。
* 缺点：需要额外的服务器配置和资源。

#### 4. 通过 `HTML5`的 `postMessage`方法跨域

`postMessage` 是一种浏览器提供的 API，用于在不同源的窗口之间进行安全通信。它可以用于以下场景：

* 父窗口与嵌入的 iframe 之间的通信。
* 不同标签页或窗口之间的通信。
* 同一页面内不同 iframe 之间的通信。

`postMessage` 允许指定消息的目标源（origin），这样可以确保消息只被特定的域接收，防止潜在的安全风险。

```js
otherWindow.postMessage(message, targetOrigin, [transfer]);
```

参数：

* `message`：要发送的消息，可以是字符串、对象等。
* `targetOrigin`：目标窗口的源（origin），指定哪些源可以接收消息。使用 `*` 表示不限制源。
* `[transfer]`（可选）：传递的对象列表，主要用于传递 `MessageChannel` 或 `ArrayBuffer`。

```js
// 在父窗口中发送消息给 iframe
const iframe = document.getElementById('myIframe');
iframe.contentWindow.postMessage('Hello from parent', 'http://example.com');

// 在子窗口（iframe）中发送消息给父窗口
window.parent.postMessage('Hello from iframe', 'http://parent.com');

```

#### 接收消息

可以在接收窗口中监听 `message` 事件来接收消息。

示例：

```js
window.addEventListener('message', (event) => {
  // 进行安全性检查
  if (event.origin !== 'http://expected-origin.com') {
    return;
  }

  // 处理接收到的消息
  console.log('Received message:', event.data);
});

```

#### 理解域名

##### 顶级域名&二级域名&三级域名

**顶级域名**是域名中最高的一级，每个域名都以顶级域名结尾。顶级域名下面是二级域名，它位于顶级域名的左侧，二级域名下面是三级域名。例如，在 `zh.wikipedia.org`中，`org`是顶级域名，`wikipedia`是二级域名。，`zh`是三级域名。

##### 子域

**子域**是属于更高一层域的域。比如，`mail.example.com`和 `calendar.example.com`是 `example.com`的两个子域，而 `example.com`则是顶级域 `.com`的子域。

**域名带不带 `www`的区别**
`www`代表的是主机名，`www.baidu.com`为三级域名，`baidu.com`属于二级域名。

🌟注意：

1. 如果是**协议**和**端口**造成的跨域问题“前台”是无能为力的；
2. 域不会判断相同的 `ip`地址对应着两个域或两个域是否在同一个 `ip`上。

### 后记

> 其它诸如中间件跨域，服务器代理跨域，Flash URLLoader跨域，动态创建script标签（简化版本的jsonp）不作讨论。
