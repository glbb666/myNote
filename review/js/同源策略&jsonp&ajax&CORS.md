[前端跨域整理](<https://juejin.im/post/5815f4abbf22ec006893b431>)

[由同源策略到前端跨域](<https://juejin.im/post/58f816198d6d81005874fd97>)

###  同源策略和跨域？

> 同源就是要求协议、域名、端口相同。不满足同源策列就会导致跨域。

```
URL                      说明       是否允许通信
http://www.a.com/a.js
http://www.a.com/b.js     同一域名下   允许

http://www.a.com/lab/a.js
http://www.a.com/script/b.js 同一域名下不同文件夹 允许

http://www.a.com:8000/a.js
http://www.a.com/b.js     同一域名，不同端口  不允许

http://www.a.com/a.js
https://www.a.com/b.js 同一域名，不同协议 不允许

http://www.a.com/a.js
http://70.32.92.74/b.js 域名和域名对应ip 不允许

http://www.a.com/a.js
http://script.a.com/b.js 子域不同 不允许（cookie这种情况下也不允许访问）

http://www.cnblogs.com/a.js
http://www.a.com/b.js 不同域名 不允许
```

#### 同源策略的限制

##### `iframe`限制

- 访问同域资源, 可读写
- 访问跨域页面时, 只读，不同的框架之间是可以获取window对象的，但却无法获取相应的属性和方法。

##### `Ajax`限制

Ajax 的限制比 iframe 限制更严.

- 同域资源可读写;
- 跨域请求会直接**被浏览器拦截**(chrome下跨域请求不会发起, 其他浏览器一般是可发送跨域请求, 但响应被浏览器拦截)

#### 理解域名

##### 顶级域名&二级域名&三级域名

**顶级域名**是域名中最高的一级，每个域名都以顶级域名结尾。顶级域名下面是二级域名，它位于顶级域名的左侧，二级域名下面是三级域名。例如，在`zh.wikipedia.org`中，`org`是顶级域名，`wikipedia`是二级域名。，`zh`是三级域名。

##### 子域

**子域**是属于更高一层域的域。比如，`mail.example.com`和`calendar.example.com`是`example.com`的两个子域，而`example.com`则是顶级域`.com`的子域。

**域名带不带www的区别**
`www`代表的是主机名，`www.baidu.com`为三级域名，`baidu.com`属于二级域名。

🌟注意：

1. 如果是**协议**和**端口**造成的跨域问题“前台”是无能为力的；
2. 域不会判断相同的`ip`地址对应着两个域或两个域是否在同一个`ip`上。
### 单向跨域
#### 1.通过`jsonp`跨域
##### 原理
`ajax`受同源策略的影响，不允许进行跨域请求，而`script`标签的`src`中的链接却可以访问跨域的静态资源，利用这个特性，服务端不再返回`JSON`格式的数据，而是返回一段调用某个函数的`js`代码，这样实现了跨域。

##### 过程

- 客户端利用`script`标签可以跨域请求资源的性质，向网页中动态插入`script`标签，来向服务器请求数据
- 服务器会解析请求的`url`，从里面取出`callback`，然后把数据放入callback中返回给客户端
##### `JSONP`的优缺点
  - 优点：它不像Ajax请求那样受到同源策略的限制；它的兼容性更好，在更加古老的浏览器中都可以运行，不需要`XMLHttpRequest`或`ActiveX`的支持；并且在请求完毕后可以通过调用`callback`的方式回传结果。
  - 缺点：它只支持GET请求，因为`script`标签只能使用`get`；没有超时处理；需要和后端协商
##### 实现一个`JSONP`

- 把传入对象转换为`url`
- 处理`url`中的回调函数
- 动态创建`script`标签并插入到页面
- 挂载回调函数

```javascript
<script type="text/javascript">
var url = 'http://localhost:8080/';
function jsonp(obj,time,url){
    //基本类型
    url+=url.indexOf('?')===-1?'?':'&';
    for(let item in obj){
        url+=item+'='+obj[item]+'&';
    }
    var callBackName =('_jsonp'+Math.random()).replace('.','')
    url+='callback='+callBackName;
    var scriptEle = document.createElement('script');
    scriptEle.src = url;
    document.head.appendChild(scriptEle);
    window[callBackName] = function(data){
       	//这里是对数据进行处理
        window.clearTimeout(timer);//清除定时器
        window[callBackName] = null;//把回调函数解除引用
        document.head.removeChild(scriptEle);
    }
    var timer = window.setTimeout(function(){
        document.head.removeChild(scriptEle);//移除script标签
		window[callBackName] = null;//把回调函数解除引用
    },time)
}
jsonp({name:'dd'},5000,url)
</script>
```



#### 2. 通过CORS跨域

> `CORS`（Cross-Origin Resource Sharing）跨域资源共享，定义了必须在访问跨域资源时，浏览器与服务器应该如何沟通。`CORS`背后的**基本思想就是使用自定义的HTTP头部让浏览器与服务器进行沟通**，从而决定请求或响应是应该成功还是失败。目前，所有浏览器都支持该功能，IE浏览器不能低于`IE10`。整个`CORS`通信过程，都是浏览器自动完成，不需要用户参与。对于开发者来说，`CORS`通信与同源的`AJAX`通信没有差别，代码完全一样。浏览器一旦发现AJAX请求跨源，就会自动添加一些附加的头信息，有时还会多出一次附加的请求，但用户不会有感觉。

**因此，实现CORS通信的关键是服务器。只要服务器实现了CORS接口，就可以跨源通信。**

平时的ajax请求可能是这样的:

```javascript
<script type="text/javascript">
    var xhr = new XMLHttpRequest();
    xhr.open("POST", "/damonare",true);
    xhr.send();
</script>
```

以上`damonare`部分是相对路径，如果我们要使用`CORS`，相关`Ajax`代码可能如下所示：

```javascript
<script type="text/javascript">
    var xhr = new XMLHttpRequest();
    xhr.open("￼GET", "http://segmentfault.com/u/trigkit4/",true);
    xhr.send();
</script>
```

代码与之前的区别就在于**相对路径换成了其他域的绝对路径**，也就是你要跨域访问的接口地址。

服务器端对于CORS的支持，主要就是通过设置`Access-Control-Allow-Origin`来进行的。如果浏览器检测到相应的设置，就可以允许Ajax进行跨域的访问。关于`CORS`更多了解可以看下阮一峰老师的这一篇文章：[跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)

- `CORS`和`JSONP`对比
  - `JSONP`只能实现`GET`请求，而`CORS`支持所有类型的`HTTP`请求。
    - 使用`CORS`，开发者可以使用普通的`XMLHttpRequest`发起请求和获得数据，比起`JSONP`有更好的错误处理。
  - `JSONP`主要被老的浏览器支持，它们往往不支持`CORS`，而绝大多数现代浏览器都已经支持了`CORS`）。

`CORS`与`JSONP`相比，无疑更为先进、方便和可靠。

#### 3. 通过window.name跨域

> window对象有个name属性，该属性有个特征：即在一个窗口(window)的生命周期内,窗口载入的所有的页面都是共享一个window.name的，每个页面对window.name都有读写的权限，window.name是持久存在一个窗口载入过的所有页面中的，并不会因新页面的载入而进行重置。

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

```
<iframe id="iframe" src="http://www.google.com/iframe.html"></iframe>复制代码
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

> 比如，有一个页面，它的地址是`www.damona.cn/a.html`， 在这个页面里面有一个`iframe`，它的`src`是`damona.cn/b.html`, 很显然，这里产生了跨域问题。这个时候，我们只要把`www.damona.cn/a.html` 和 `damona.cn/b.html`这两个页面的document.domain都设成相同的域名就可以了。但要注意的是，<font color='red'>我们只能把document.domain设置成自身或更高一级的父域，且主域必须相同。</font>

- 在页面www.damona.cn/a.html 中设置document.domain:

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
    document.domain = 'damonare.cn';//在iframe载入这个页面也设置document.domain，使之与主页面的document.domain相同
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

`postMessage`的使用方法：

- otherWindow.postMessage(message, targetOrigin);
  - otherWindow:指目标窗口，也就是给哪个window发消息，是 window.frames 属性的成员或者由 window.open 方法创建的窗口
  - `message:   `是要发送的消息，类型为 String、Object (IE8、9 不支持)
  - `targetOrigin:`   是限定消息接收范围，不限制请使用 

B页面通过`message`事件监听并接受消息:

```
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





### 后记

> 其它诸如中间件跨域，服务器代理跨域，Flash URLLoader跨域，动态创建script标签（简化版本的jsonp）不作讨论。