# Cookie、sessionStorage、localStorage的异同

[cookies、sessionStorage和localStorage解释及区别](https://www.cnblogs.com/pengc/p/8714475.html)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190909222836947.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dhbmx1YmFiYTY2Ng==,size_16,color_FFFFFF,t_70)

### 共同点

都是保存在浏览器端，并且是同源的

### 不同点

#### 生命周期

**cookie：**看服务端有没有设置过期时间，如果有设置的话，在这个周期内cookie有效，否则仅关闭浏览器关闭前有效（会话cookie）

**sessionStorage：**仅在当前**浏览器同源的窗口关闭前**有效，只要同源窗口中的任意一个存在，sessionStorage就存在，如果当前同源窗口都被关闭后，即使再进入同一个窗口同一个页面，sessionStorage也是不一样的。

**localStorage：**始终有效，因此用作持久数据

#### 存储空间大小

**cookie：**4KB左右，因为浏览器每次发起请求都会携带`cookie`，`cookie`太大会消耗性能

**localStorage和sessionStorage：**可以保存5MB或更大的信息。

#### 存储格式

**localStorage**和**sessionStorage**只能存储字符串类型

#### 范围

**cookie**：`cookie`也是在所有**同源窗口中都是共享**的，但`cookie`数据有路径（path）的概念，可以限制`cookie`只属于某个路径下

**localStorage**：在所有**同源窗口中都是共享的**。

**sessionStorage**：同源不共享,在**内嵌页面会拷贝一份过去,两者没有关系**。

#### 数据操作

数据**操作比cookie方便**，`WebStorage`提供了一些方法；
```
　　       setItem (key, value) ——  保存数据，以键值对的方式储存信息。

      　　  getItem (key) ——  获取数据，将键值传入，即可获取到对应的value值。
    
        　　removeItem (key) ——  删除单个数据，根据键值移除对应的信息。
    
        　　clear () ——  删除所有的数据
    
        　　key (index) —— 获取某个索引的key
```

#### 显示速度

有的数据存储在`WebStorage`上，再加上浏览器本身的缓存。获取数据时可以从本地获取会比从服务器端获取快得多，所以速度更快

#### 检测`localStorage`的最大容量

检测`localStorage`的剩余容量只要把`localStorage`的最大容量减去`localStorage`的已用容量即可。

`js`中一个英文占一个字节

1kb = 1024b

```javascript
(function() {
   if(!window.localStorage) {
       console.log('当前浏览器不支持localStorage!')
   }
   var test = '0123456789';
   var add = function(num) {
     num += num;
     if(num.length == 10240) {//10kb
       test = num;
       return;
     }
     add(num);
   }
   add(test);
   var sum = test;
   var show = setInterval(function(){
      sum += test;
      try {
       window.localStorage.removeItem('test');
       window.localStorage.setItem('test', sum);
       console.log(sum.length / 1024 + 'KB');
      } catch(e) {
       console.log(sum.length / 1024 + 'KB超出最大限制');
       clearInterval(show);
      }
   }, 0.1)
 })()
```

