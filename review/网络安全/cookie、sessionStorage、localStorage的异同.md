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

**cookie：**4KB左右

**localStorage和sessionStorage：**可以保存5MB或更大的信息。

#### 存储格式

**localStorage**和**sessionStorage**只能存储字符串类型

#### http请求

**cookie**：`cookie`数据始终在同源的`http`请求中携带（即使不需要），即`cookie`在浏览器和服务器间来回传递，所以无形中增加了流量。

**localStorage**和**sessionStorage**：仅在客户端（即浏览器）中保存，不参与和服务器的通信

#### 范围

**cookie**：`cookie`也是在所有**同源窗口中都是共享**的，但`cookie`数据有路径（path）的概念，可以限制`cookie`只属于某个路径下

**localStorage**：在所有**同源窗口中都是共享的**。

**sessionStorage**：在**一同存在的同源窗口中共享**。