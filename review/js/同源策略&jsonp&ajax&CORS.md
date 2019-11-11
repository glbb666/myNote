[前端跨域整理](<https://juejin.im/post/5815f4abbf22ec006893b431>)

#### 1. 什么是跨域？

> 只要协议、域名、端口有任何一个不同，都被当作是不同的域。

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

#### 2. 通过document.domain跨域

> 浏览器有一个同源策略，其限制之一是不能通过`ajax`的方法去请求不同源中的文档。第二个限制是浏览器中不同域的框架之间是不能进行`js`的交互操作的。不同的框架之间是可以获取window对象的，但却无法获取相应的属性和方法。比如，有一个页面，它的地址是`www.damonare.cn/a.html`， 在这个页面里面有一个`iframe`，它的`src`是`damonare.cn/b.html`, 很显然，这个页面与它里面的`iframe`框架是不同域的，所以我们是无法通过在页面中书写`js`代码来获取`iframe`中的东西的：

```javascript
<script type="text/javascript">
    function test(){
        var iframe = document.getElementById('ifame');
        var win = iframe.contentWindow;//可以获取到iframe里的window对象，但该window对象的属性和方法几乎是不可用的
        var doc = win.document;//这里获取不到iframe里的document对象
        var name = win.name;//这里同样获取不到window对象的name属性
    }
</script>
<iframe id = "iframe" src="http://damonare.cn/b.html" onload = "test()"></iframe>
```

> 这个时候，document.domain就可以派上用场了，我们只要把[www.damonare.cn/a.html](http://www.damonare.cn/a.html) 和 [damonare.cn/b.html](http://damonare.cn/b.html) 这两个页面的document.domain都设成相同的域名就可以了。但要注意的是，document.domain的设置是有限制的，我们只能把document.domain设置成自身或更高一级的父域，且主域必须相同。

- 在页面[www.damonare.cn/a.html](http://www.damonare.cn/a.html) 中设置document.domain:

```javascript
<iframe id = "iframe" src="http://damonare.cn/b.html" onload = "test()"></iframe>
<script type="text/javascript">
    document.domain = 'damonare.cn';//设置成主域
    function test(){
        alert(document.getElementById('iframe').contentWindow);//contentWindow 可取得子窗口的 window 对象
    }
</script>
```

- 在页面[damonare.cn/b.html](http://damonare.cn/b.html) 中也设置document.domain:

```javascript
<script type="text/javascript">
    document.domain = 'damonare.cn';//在iframe载入这个页面也设置document.domain，使之与主页面的document.domain相同
</script>
```

**修改document.domain的方法只适用于不同子域的框架间的交互。**