#  什么是WebWorker

web worker 是运行在后台的 JavaScript，不会影响页面的性能。

### Web Worker简介

当在 HTML 页面中执行脚本时，页面的状态是不可响应的，直到脚本已完成。

web worker 是运行在后台的 JavaScript，独立于其他脚本，不会影响页面的性能。您可以继续做任何愿意做的事情：点击、选取内容等等，而此时 web worker 在后台运行。

## 

### 使用方法

1. 创建**外部脚本**。

在这里，我们创建了计数脚本。该脚本存储于"demo_workers.js" 文件中：

```js
  // onmessage || addEventListener('message', fun, false)
  self.addEventListener('message', function (e) {
    // 监听主线程发送的消息
    // 发送消息给主线程
    self.postMessage('You said: ' + e.data);
  }, false);

  // 关闭连接
  self.close();

  // onerror
```

以上代码中重要的部分是 **postMessage()** 方法，它用于向 HTML 页面传回一段消息。

2. 创建 Worker 对象 用 onmessage 方法监听 worker 发送的消息。

```js
  if(typeof(w)=="undefined") {
    w = new Worker("demo_workers.js");
    w.onmessage = function(event) {
      // 处理...
    };
  }
```

3. 主线程调用 postMessage 向 worker 对象发消息

```js
  w.postMessage('ok');
  w.postMessage({ name: 'a', age: '12' });
```

4. 终止 Worker

```js
  w.terminate();
```



### 注意事项

1. **同源限制** 分配给 Worker 线程运行的脚本文件，必须与主线程的脚本文件同源。

2. **DOM 限制** Worker 线程所在的全局对象，与主线程不一样，无法读取主线程所在网页的 DOM 对象，也无法使用 document、window、parent 这些对象。但是，Worker 线程可以访问 navigator 对象和 location 对象。

3. **通信联系** Worker 线程和主线程不在同一个上下文环境，它们不能直接通信，必须通过消息完成。

4. **脚本限制** Worker 线程不能执行 alert() 方法和 confirm() 方法，但可以使用 XMLHttpRequest 对象发出 AJAX 请求。

5. **文件限制** Worker 线程无法读取本地文件，即不能打开本机的文件系统（file://），它所加载的脚本，必须来自网络。

   注意：如果worker是引用的别的js文件，应该用服务器进行读取

   ```js
   const express = require('express');
   const static = require('express-static');
   var app = new express();
   app.use(static('./'));
   console.log('创建');
   app.listen(8080);
   ```

   以上是服务器的代码，服务器nodejs应该放在文件的最外层，因为它只会往内部访问了。

### 使用实例

1. 浏览器轮询
2. Worker 创建新 Worker

### 更多

#### 主线程

- Worker.onerror：指定 error 事件的监听函数。
- Worker.onmessage：指定 message 事件的监听函数，发送过来的数据在Event.data属性中。
- Worker.onmessageerror：指定 messageerror 事件的监听函数。发送的数据无法序列化成字符串时，会触发这个事件。
- Worker.postMessage()：向 Worker 线程发送消息。
- Worker.terminate()：立即终止 Worker 线程。

#### Worker 线程

Web Worker 有自己的全局对象，不是主线程的window，而是一个专门为 Worker 定制的全局对象。因此定义在window上面的对象和方法不是全部都可以使用。

Worker 线程有一些自己的全局属性和方法。

- self.name： Worker 的名字。该属性只读，由构造函数指定。
- self.onmessage：指定message事件的监听函数。
- self.onmessageerror：指定 messageerror 事件的监听函数。发送的数据无法序列化成字符串时，会触发这个事件。
- self.close()：关闭 Worker 线程。
- self.postMessage()：向产生这个 Worker 线程发送消息。
- self.importScripts()：加载 JS 脚本。

### 扩展

[☄️ Service Worker](https://juejin.im/post/5b06a7b3f265da0dd8567513)

[💥 Shared Worker](https://www.zhuwenlong.com/blog/article/590ea64fe55f0f385f9a12e5)