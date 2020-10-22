[前端跨域整理](<https://juejin.im/post/5815f4abbf22ec006893b431>)

[由同源策略到前端跨域](<https://juejin.im/post/58f816198d6d81005874fd97>)

[九种跨域方式实现原理（完整版）](<https://juejin.im/post/5c23993de51d457b8c1f4ee1#heading-2>)

[Jquery ajax, Axios, Fetch区别](<https://juejin.im/post/5acde23c5188255cb32e7e76>)

[CORS 简单请求+预检请求（彻底理解跨域）](<https://github.com/amandakelake/blog/issues/62>)

###  同源策略和跨域？

> 如果两个页面的协议，端口和主机都相同，则两个页面具有相同的**源**。不满足同源策列，就会产生跨域问题。谷歌浏览器跨域请求不会发起，其他浏览器可发送跨域请求，但是相应会被浏览器拦截。
>

🌟但是有三种标签允许跨域加载资源

- `<img src=XXX>`
- `<link href=XXX>`
- `<script src=XXX>`

### AJAX

#### `AJAX`是什么

`AJAX`的核心是`XMLHttpRequest`对象，它不必刷新页面就可以从服务器获取到新数据。

- 我们先创建一个`XMLHttpRequest`对象，并且监听它的`onreadyStatuschange`事件，当`readyStatus`的值为4，说明已经收到完整响应了。
- 我们再判断`status`（状态码），当它的值为`200~300`或`304`的时候，就从`responseText`中读取服务器相应内容的文本。
- 接着我们用`open`启动一个请求，为请求指定`url`地址，方法，同步性。
  - `get`请求的`Content-type`默认为`text/plain`。
  - 非`get`请求，要`setRequestHeader`，设置`Content-type`的值。
- 我们还可以用`timeout`设置超时，并且在`ontimeout`中调用`abort`取消请求。用`onerror`处理请求错误的情况。
- 接着我们调用`send`方法，在`send`方法执行之前，`AJAX`都是同步的。`send`方法执行过后，浏览器就会为`http`请求创建一个`http`请求线程，这个请求独立于`js`引擎线程，是异步的，`js`引擎并不会等待这个异步请求的结果，而是会继续执行下去。当浏览器收到服务器的响应，浏览器事件触发线程就会捕获到`ajax`的回调事件`onreadyStatechange`或者是`onerror`事件，浏览器事件触发线程会把回调事件添加到任务队列末尾，直到`js`引擎线程空闲，任务队列的任务才会被添加到执行栈中依次执行。

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

#### 用`promise`封装`ajax`

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

#### AJAX请求头Content-Type

[Content-Type的理解](<https://www.cnblogs.com/tugenhua0707/p/8975121.html>)

**Content-Type**是指`http/https`发送信息至服务器时的**内容编码类型**，默认值为`text/plain;charset=UTF-8`

- `application/x-www-form-urlencoded`：在发送前编码所有字符。原生的`form`表单，如果不设置`enctype`属性，那么就以这种方式提交数据，如下

  ```
  title=test&sub%5B%5D=1&sub%5B%5D=2&sub%5B%5D=3
  //所有数据变成键值对的形式，并且特殊字符需要转义成utf-8编号
  ```
  
- `multipart/form-data`：不对字符编码。在使用表单上传文件时，必须使用该值。请求体被` --boundary`分割成多部分

- `application/json`：用来告诉服务器消息主题是序列化后的JSON字符串，一般用来传递复杂的数据对象。用这个值时发送的是非简单请求，非简单请求就是会触发预检的请求。


#### AJAX返回的状态

| 状态码 | 含义                                         |
| ------ | -------------------------------------------- |
| 0      | 未初始化。未调用open()                       |
| 1      | 启动。已调用open()，未调用send()             |
| 2      | 发送。调用send(),但未收到响应                |
| 3      | 接收。受到部分响应数据                       |
| 4      | 完成。接收全部响应数据，且可以在客户端使用。 |

🌟注意：`send`之前，都是同步的

#### 如何发出一个有序的`AJAX`

回调函数，`Promise.then`,`async`

#### `ajax`和`jquery`, `Fecth`,`Axios`比有什么区别

##### `jquery`

是对原生`XMLHttpRequest`对象的封装,还增添了对`jsonp`的支持,但是`jquery`整个项目太大了,单纯使用`ajax`就要引入整个`jquery`非常的不合理

##### `fecth`

优点:基于`promise`，支持`node`,比较轻量

缺点：

- `promise`的状态，除了**网络故障或者请求被阻止**，其他情况下`fecth`返回的`promise`状态都为`resolve`(但是会将 resolve 的返回值的 `ok` 属性设置为 `false` )。

- `cookie`，默认情况下，`fecth`不会从服务器接受或发送`cookie`，需要设置`credenitials`选项。

  所以需要我们再次封装

> fetch发送post请求的时候，总是发送两次，第一次状态码是204，第二次才成功？
>
> 因为第一次发送Options请求，询问服务器是否支持修改的请求头，如果服务器支持，则在第二次中发送真正的请求。

##### `axios`

也是对原生`XMLHttpRequest`对象的封装,但是是`promise`的实现版本,它有以下功能

- 从` node.js `创建 `http` 请求
- 支持` Promise API`
- 客户端支持防止`CSRF`
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
`ajax`受同源策略的影响，不允许进行跨域请求。`JSONP`利用`script`标签中的`src`属性可以访问跨域的静态资源的特性，动态插入`script`标签，客户端通过`get`请求，向服务器发送回调函数的名称，同时在`window`对象中设置回调函数。服务端返回一段调用回调函数的`js`代码，把参数放到回调函数中。通过`script`标签加载执行，从而实现跨域。

##### 过程

- 客户端利用`script`标签可以跨域请求资源的性质，向网页中动态插入`script`标签，来向服务器请求数据
- 服务器会解析请求的`url`，从里面取出`callback`，然后把数据放入`callback`中返回给客户端
##### `JSONP`的优缺点
优点：

- 不受到同源策略的限制
- 兼容性更好，在更加古老的浏览器中都可以运行，不需要`XMLHttpRequest`或`ActiveX`的支持
- 在请求完毕后可以通过调用`callback`的方式回传结果。

缺点：

  - 只支持`GET`请求，因为`script`标签只能使用`get`
  - 没有超时处理
  - 需要和后端协商
##### 实现一个`JSONP`(`promise`)

- 把传入对象转换为`url`
- 给回调函数名设置随机标识，并且拼接到`url`中
- 动态创建`script`标签并把`url`赋值给`src`
- 挂载回调函数
- 出错处理
- 超时处理
- 把`script`标签插入到页面

```javascript
<script type="text/javascript">
var url = 'http://localhost:8080/';
function getParams(data) {
  let arr = []
  for (let i in data) {
    arr.push(encodeURIComponent(i) + '=' + encodeURIComponent(data[i]))
  }
  return arr.join('&')
}
function jsonp(options){
    let {data,time,url,timeout} = options
    let timer = null;
    //基本类型
    url=url+'?'+getParams(data)+'&'
    var callBackName =('_jsonp'+Math.random()).replace('.','')
    url+='callback='+callBackName;
    var scriptEle = document.createElement('script');
    scriptEle.src = url;
    return new Promise((resolve,reject)=>{
            //这里是对数据进行处理
            window[callBackName] = function(data){
                resolve(data)
                window.clearTimeout(timer);//清除定时器
                window[callBackName] = null;//把回调函数解除引用
                document.head.removeChild(scriptEle);
            }
            //超时处理
        	timer = setTimeout(()=>{
                reject('timeout');
                window[callBackName] = null;//把回调函数解除引用
                document.head.removeChild(scriptEle);
            },timeout);
            //出错处理
            scriptEle.onerror = function(){
               reject('scriptEle error');
               window.clearTimeout(timer);//清除定时器
               window[callBackName] = null;//把回调函数解除引用
               document.head.removeChild(scriptEle);
            }
            document.head.appendChild(scriptEle);
    })
}
jsonp({name:'dd'},5000,url)
</script>
```

#### 2. `CORS(Cross-Origin Resource Sharing)跨域资源共享`

`CORS`允许服务器声明哪些源站通过浏览器有权限访问哪些资源。

字段记忆（`ACA`->`Access-Control-Allow`)

##### 基本使用

```js
server.all('*',function(req,res,next){
    
    //其中`*` 表示通配, 所有的域都能访问此资源,为了跨站发送cookie等验证信息， `Access-Control-Allow-Origin` 字段将不允许设置为`*`, 它需要明确指定与请求网页一致的域名
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

同时，前端需要做如下显式设置才能真正发送`cookie`

```javascript
xhr.withCredentials = true;
```

##### `简单请求`和`预检请求（preflighted requests）`

对于非简单请求，**浏览器必须首先使用` OPTIONS `方法发起一个预检请求，从而获知服务端是否允许该跨域请求(同源不会发送`OPTIONS`)。**

**服务器确认允许之后，浏览器会记住，然后再发起实际的 HTTP 请求，且之后每次就都直接请求而不用再询问服务器是否可以跨域**。在预检请求的返回中，服务器端也可以通知客户端，是否需要携带身份凭证（包括 Cookies 和 HTTP 认证相关数据）。

![2ec957a3-b220-49c5-8fa1-59a82a030e89](images/50205846-accac300-03a4-11e9-8654-2d646d237820.png)

**简单请求就是不会触发CORS预检的请求**，满足以下**所有条件**的才会被视为简单请求，基本上我们日常开发只会关注前面两点

1. 使用`GET、POST、HEAD`其中一种方法
2. 只使用了如下的安全首部字段，不得人为设置其他首部字段
   - `Accept`

   - `Accept-Language`

   - `Content-Language`

   - `Content-Type`仅限以下三种
     - `text/plain`
      - `multipart/form-data`
       - `application/x-www-form-urlencoded`

   - HTML头部header field字段：`DPR、Download、Save-Data、Viewport-Width、WIdth`
4. 请求中的任意`XMLHttpRequestUpload` 对象均没有注册任何事件监听器；XMLHttpRequestUpload 对象可以使用 XMLHttpRequest.upload 属性访问
5. 请求中没有使用 ReadableStream 对象

预检请求要求必须**首先使用 `OPTIONS` 方法发起一个预检请求到服务器，以获知服务器是否允许该实际请求**。"预检请求“的使用，可以避免跨域请求对服务器的用户数据产生未预期的影响 

非简单请求就是会触发的预检请求

1. 使用了`PUT、DELETE、CONNECT、OPTIONS、TRACE、PATCH`方法
2. 人为设置了非规定内的其他首部字段，参考上面简单请求的安全字段集合，还要特别注意`Content-Type`的类型
3. `XMLHttpRequestUpload` 对象注册了任何事件监听器
4. 请求中使用了`ReadableStream`对象

##### 完整请求流程

![img](images/50205881-c409b080-03a4-11e9-8a57-a2a6d0e1d879.png)

##### `CORS`和`JSONP`对比

  - 请求类型：`JSONP`只能实现`GET`请求，而`CORS`支持所有类型的`HTTP`请求。
    
  - 错误处理：使用`CORS`，开发者可以使用普通的`XMLHttpRequest`发起请求和获得数据，比起`JSONP`有更好的错误处理。
    
  - `JSONP`主要被老的浏览器支持，它们往往不支持`CORS`，而绝大多数现代浏览器都已经支持了`CORS`。

`CORS`与`JSONP`相比，无疑更为先进、方便和可靠。

#### 3. 通过window.name跨域

> 在一个窗口（window） 的生命周期内,窗口载入的所有的页面都是共享一个window.name的，每个页面对window.name都有读写的权限，window.name是持久存在一个窗口载入过的所有页面中的，并不会因新页面的载入而进行重置。

比如：我们在任意一个页面输入

```javascript
window.name = "My window's name";
setTimeout(function(){
    window.location.href = "http://damonare.cn/";
},1000)
```

进入damonare.cn页面后我们再检测再检测 window.name :

```javascript
window.name; // My window's name
```

可以看到，如果在一个标签里面跳转网页的话，我们的 window.name 是不会改变的。
基于这个思想，我们可以在某个页面设置好 window.name 的值，然后跳转到另外一个页面。在这个页面中就可以获取到我们刚刚设置的 window.name 了。

> 由于安全原因，浏览器始终会保持 window.name 是string 类型。

同样这个方法也可以应用到和iframe的交互来：
比如：我的页面([damonare.cn/index.html)…](http://damonare.cn/index.html)中内嵌了一个iframe：)

```javascript
<iframe id="iframe" src="http://www.google.com/iframe.html"></iframe>
```

在 iframe.html 中设置好了 window.name 为我们要传递的字符串。
我们在 index.html 中写了下面的代码：

```javascript
var iframe = document.getElementById('iframe');
var data = '';

iframe.onload = function() {
    data = iframe.contentWindow.name;
};
```

Boom!报错！肯定的，因为两个页面不同源嘛，想要解决这个问题可以这样干：

```javascript
var iframe = document.getElementById('iframe');
var data = '';

iframe.onload = function() {
    iframe.onload = function(){
        data = iframe.contentWindow.name;
    }
    iframe.src = 'about:blank';
};
```

**或者将里面的 about:blank 替换成某个同源页面（about:blank，javascript: 和 data: 中的内容，继承了载入他们的页面的源。）**

这种方法与 document.domain 方法相比，放宽了域名后缀要相同的限制，可以从任意页面获取 string 类型的数据。

### 双向跨域

#### 4. 通过document.domain跨域

> 例子：`www.damona.cn/a.html`页面中`iframe`的`src`是`damona.cn/b.html`。这个时候，我们只要把`www.damona.cn/a.html` 和 `damona.cn/b.html`这两个页面的`document.domain`都设成相同的域名就可以了。
>
> 把两个页面的`document.domain`设置成相同的域名就可以进行跨域。但是由于<font color='red'>我们只能把document.domain设置成自身或更高一级的父域，且主域必须相同。</font>所以修改`document.domain`的方法只适用于不同子域的框架间的交互。

- 在页面www.damona.cn/a.html 中设置`document.domain`:

```javascript
<iframe id = "iframe" src="http://damona.cn/b.html" onload = "test()"></iframe>
<script type="text/javascript">
    document.domain = 'damona.cn';//设置成主域
    function test(){
        alert(document.getElementById('iframe').contentWindow);//contentWindow 可取得子窗口的 window 对象
    }
</script>
```

- 在页面[damona.cn/b.html](http://damonare.cn/b.html) 中也设置document.domain:

```javascript
<script type="text/javascript">
    document.domain = 'damona.cn';//在iframe载入这个页面也设置document.domain，使之与主页面的document.domain相同
</script>
```

**修改document.domain的方法只适用于不同子域的框架间的交互。**

#### 5. 通过location.hash跨域

> 因为父窗口和iframe可以对对方进行URL读写，URL有一部分被称为hash，就是#号及其后面的字符，它一般用于浏览器锚点定位，应该说HTTP请求过程中不会携带hash，所以这部分的修改不会产生HTTP请求，但是会产生浏览器历史记录。此方法的**原理就是改变URL的hash部分来进行双向通信**。每个window通过改变其他 window的location来发送消息（由于两个页面不在同一个域下IE、Chrome不允许修改parent.location.hash的值，所以要借助于父窗口域名下的一个代理iframe），并通过监听自己的URL的变化来接收消息。这个方式的通信会造成一些不必要的浏览器历史记录，而且有些浏览器不支持`onhashchange`事件，需要轮询来获知URL的改变，最后，这样做也存在缺点，诸如数据直接暴露在了`url`中，数据容量和类型都有限等。下面举例说明：

假如父页面是baidu.com/a.html,iframe嵌入的页面为google.com/b.html（此处省略了域名等url属性），要实现此两个页面间的通信可以通过以下方法。

- a.html传送数据到b.html
  - a.html下修改iframe的src为google.com/b.html#paco
  - b.html监听到url发生变化，触发相应操作
- b.html传送数据到a.html，由于两个页面不在同一个域下IE、Chrome不允许修改parent.location.hash的值，所以要借助于父窗口域名下的一个代理iframe
  - b.html下创建一个隐藏的iframe，此iframe的src是baidu.com域下的，并挂上要传送的hash数据，如src="[www.baidu.com/proxy.html#…](http://www.baidu.com/proxy.html#data)"
  - proxy.html监听到url发生变化，修改a.html的url（因为a.html和proxy.html同域，所以proxy.html可修改a.html的url hash）
  - a.html监听到url发生变化，触发相应操作

b.html页面的关键代码如下:

```javascript
try {  
    parent.location.hash = 'data';  
} catch (e) {  
    // ie、chrome的安全机制无法修改parent.location.hash，  
    var ifrproxy = document.createElement('iframe');  
    ifrproxy.style.display = 'none';  
    ifrproxy.src = "http://www.baidu.com/proxy.html#data";  
    document.body.appendChild(ifrproxy);  
}
```

proxy.html页面的关键代码如下 :

```javascript
//因为parent.parent（即baidu.com/a.html）和baidu.com/proxy.html属于同一个域，所以可以改变其location.hash的值  
parent.parent.location.hash = self.location.hash.substring(1);
```

#### 6. 通过`HTML5`的`postMessage`方法跨域

> 高级浏览器Internet Explorer 8+, chrome，Firefox , Opera  和 Safari 都将支持这个功能。这个功能主要包括接受信息的"message"事件和发送消息的"postMessage"方法。比如damonare.cn域的A页面通过iframe嵌入了一个google.com域的B页面，可以通过以下方法实现A和B的通信

A页面通过postMessage方法发送消息：

```javascript
window.onload = function() {  
    var ifr = document.getElementById('ifr');  
    var targetOrigin = "http://www.google.com";  
    ifr.contentWindow.postMessage('hello world!', targetOrigin);  
};
```

`postMessage`的使用方法：调用目标窗口的`postMessage`方法，第一个参数是消息，第二个参数是域名

- otherWindow.postMessage(message, targetOrigin);
  - otherWindow:指目标窗口，也就是给哪个window发消息，是 window.frames 属性的成员或者由 window.open 方法创建的窗口
  - `message:   `是要发送的消息，类型为 String、Object (IE8、9 不支持)
  - `targetOrigin:`   是限定消息接收范围，不限制请使用 

B页面通过`message`事件监听并接受消息:

```javascript
var onmessage = function (event) {  
  var data = event.data;//消息  
  var origin = event.origin;//消息来源地址  
  var source = event.source;//源Window对象  
  if(origin=="http://www.baidu.com"){  
console.log(data);//hello world!  
  }  
};  
if (typeof window.addEventListener != 'undefined') {  
  window.addEventListener('message', onmessage, false);  
} else if (typeof window.attachEvent != 'undefined') {  
  //for ie  
  window.attachEvent('onmessage', onmessage);  
}
```

同理，也可以B页面发送消息，然后A页面监听并接受消息。

#### 理解域名

##### 顶级域名&二级域名&三级域名

**顶级域名**是域名中最高的一级，每个域名都以顶级域名结尾。顶级域名下面是二级域名，它位于顶级域名的左侧，二级域名下面是三级域名。例如，在`zh.wikipedia.org`中，`org`是顶级域名，`wikipedia`是二级域名。，`zh`是三级域名。

##### 子域

**子域**是属于更高一层域的域。比如，`mail.example.com`和`calendar.example.com`是`example.com`的两个子域，而`example.com`则是顶级域`.com`的子域。

**域名带不带`www`的区别**
`www`代表的是主机名，`www.baidu.com`为三级域名，`baidu.com`属于二级域名。

🌟注意：

1. 如果是**协议**和**端口**造成的跨域问题“前台”是无能为力的；
2. 域不会判断相同的`ip`地址对应着两个域或两个域是否在同一个`ip`上。

### 后记

> 其它诸如中间件跨域，服务器代理跨域，Flash URLLoader跨域，动态创建script标签（简化版本的jsonp）不作讨论。