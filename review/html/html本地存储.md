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

（6）数据**操作比cookie方便**，WebStorage提供了一些方法；

```
　　　      setItem (key, value) ——  保存数据，以键值对的方式储存信息。
  
      　　  getItem (key) ——  获取数据，将键值传入，即可获取到对应的value值。

        　　removeItem (key) ——  删除单个数据，根据键值移除对应的信息。

        　　clear () ——  删除所有的数据

        　　key (index) —— 获取某个索引的key
```
## webStorage的特点
- 存储位置：**保存在客户端**，不与服务器进行交互通信。
- 存储内容类型：**只存储字符串类型**，对于复杂的对象可以使用ECMAScript提供的JSON对象的stringify和parse来处理
- 获取方式：localStorage：window.localStorage，sessionStorage：window.sessionStorage。
- 应用场景：localStoragese：常用于**长期登录**（+判断用户是否已登录），适合长期保存在本地的数据。sessionStorage：**敏感账号一次性登录**