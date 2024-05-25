# 是什么

* **Web Worker** 是渲染进程中的一个线程，允许执行耗时任务而不冻结用户界面。
* **Service Worker** 是一个进程，主要用于拦截和处理网络请求，缓存资源以实现离线访问，背景同步和推送通知等。
* **Shared Worker** 是一个进程，可以把他理解为一种特殊类型的Web Worker，可以被多个执行上下文（如多个标签、iframe或window）共享。

# **为什么**

* 使用 **Web Worker** 可以将耗时的计算放置在后台执行，避免阻塞主线程，提高了应用的响应性能。
* 使用 **Service Worker** 可以让Web应用进行离线工作，提高了应用的加载速度和可靠性，也可以接收服务器推送的消息，即使Web应用没有打开也能够更新数据。
* 使用 **Shared Worker** 可以在多个浏览器上下文中共享同一个Worker实例，使得不同的浏览器标签之间可以共享数据和消息，有利于省去了不同标签页或iframe之间设置复杂通讯机制的麻烦。

# 怎么用

## Web Worker

1. **创建：**
   在主线程中创建一个Web Worker实例，通过指定一个JavaScript文件来初始化这个Worker：

   ```javascript
   var myWorker = new Worker('worker.js');
   ```

   在 `worker.js` 文件中，编写你的后台代码，就像这样：

   ```javascript
   // worker.js
   //直接在Worker脚本中调用onmessage和postMessage方法是可以工作的，因为它们都是全局作用域内的方法
   onmessage = function(e) {
     var result = doSomeWork(e.data);
     postMessage(result);
   };

   function doSomeWork(data) {
     // 此处执行耗时任务
     return '处理完成';
   }
   ```
2. **使用：**
   主线程与Web Worker之间通过事件和消息进行通信。发送消息使用 `postMessage()` 方法，接收消息使用 `onmessage` 事件监听：

   ```javascript
   // 主线程
   myWorker.postMessage('some data'); // 发送数据到Worker

   myWorker.onmessage = function(e) { // 从Worker收到数据
     console.log('Message received from worker: ', e.data);
   };

   myWorker.onerror = function(error) { // 从Worker收到错误
     console.error('Error received from worker: ', error);
   };
   ```
3. **销毁：**
   当你不再需要Web Worker时，应该关闭Worker来节省系统资源。可以在主线程或者Worker内部调用 `close()` 方法：

   ```javascript
   // 主线程
   myWorker.terminate();

   // 或在 Worker 内部，直接在Worker脚本中调用close是可以工作的，因为它是全局作用域内的方法
   close();
   ```

## **Service Worker**

1. **创建：**
   Service Worker的注册通常在网站的入口页面JavaScript中进行：

   ```javascript
   // 检查Service Worker是否可用
   // navigator 是一个内建的对象，它表示用户的浏览器信息
   if ('serviceWorker' in navigator) {
     // 注册Service Worker
     navigator.serviceWorker.register('/service-worker.js').then(function(registration) {
       console.log('Service Worker 注册成功:', registration.scope);
     }).catch(function(error) {
       console.log('Service Worker 注册失败:', error);
     });
   }
   ```

   在 `service-worker.js` 文件里，处理安装、激活、抓取请求等逻辑：

   ```javascript
   // service-worker.js
   addEventListener('install', function(event) {
     // 在安装阶段预缓存资源
   });

   addEventListener('activate', function(event) {
     // 激活时清理旧缓存
   });

   addEventListener('fetch', function(event) {
     // 拦截网络请求并作出处理
   });
   ```
2. **使用：**
   一旦注册成功，Service Worker就会控制它注册时所在目录范围内的所有页面，并在相关事件触发时执行指定的事件处理程序。
3. **销毁：**
   Service Worker**不需要显式销毁**。当Service Worker不再控制任何页面并且事件监听回调都完成时，它会被终止，直到下一个事件触发时重新激活。

### 控制页面的范围

它注册时所在目录范围内的所有页面

举个例子，假设你在网站根目录下注册了一个 Service Worker。

```js
// 位于 / 根目录的一个脚本文件内
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/service-worker.js')
    .then(function(registration) {
      // 注册成功
    })
    .catch(function(error) {
      // 注册失败
    });
}

```

这意味着Service Worker的作用范围是整个网站（因为根目录是整个网站的基础路径）。因此，该 Service Worker 将会控制所有以 `https://YOUR_DOMAIN/`开头的页面。如果Service Worker是在一个子目录下注册的，那么它的作用范围将被限制在该子目录下。

### 控制页面的时机

仅在Service Worker完成**注册、安装、激活并且页面重新加载（如用户手动刷新）**后，它才能开始控制页面。

下面是Service Worker生命周期的基本序列（顺序进行）：

1. **注册**：这个动作通常发生在页面的JavaScript代码中，用来告知浏览器Service Worker的脚本位置。注册成功后，浏览器会在后台下载Service Worker文件并尝试安装它。

   ```js
   // 检查Service Worker API是否可用，并注册Service Worker
   if ('serviceWorker' in navigator) {
     navigator.serviceWorker.register('/service-worker.js').then(function(registration) {
       // 注册成功
     }).catch(function(error) {
       // 注册失败
     });
   }

   ```
2. **安装** ：注册成功后，Service Worker进入 `install`状态。这一个阶段主要用于打开缓存，缓存应用的重要资源。

   ```js
   // Service Worker 文件内
   self.addEventListener('install', function(event) {
     event.waitUntil(
       caches.open('v1').then(function(cache) {
         return cache.addAll([
           // 列出要缓存的文件
         ]);
       })
     );
   });

   ```
3. **激活**：安装步骤后面紧随的是激活阶段。这一个阶段，Service Worker可以清理旧缓存，执行更新后的清理工作等。只有在这之后，Service Worker才有能力控制网页。

   ```js
   // Service Worker 文件内
   self.addEventListener('activate', function(event) {
     // 生命周期的激活部分
   });

   ```
4. **控制**：在 `activate`事件完成后，Service Worker就能控制其作用范围内的所有页面了，但它控制的仅限于激活之后新加载或重新加载的页面。

   如果你想让Service Worker在注册后立即控制页面，你可以利用 `clients.claim()`方法在激活阶段调用，这样Service Worker就可以直接控制当前页面而不需重新加载

   ```js
   // Service Worker 文件内
   self.addEventListener('activate', function(event) {
     event.waitUntil(
       clients.claim() // Service Worker 取得页面的控制权，不需再重启页面
     );
   });

   ```

## **Shared Worker**

1. **创建：**
   在主线程中创建一个Shared Worker连接，通过指定一个JavaScript文件来初始化这个Worker：

   ```javascript
   var mySharedWorker = new SharedWorker('shared-worker.js');
   ```

   在 `shared-worker.js` 文件中，编写你的共享代码：

   ```javascript
   // shared-worker.js
   var connections = []; // 存储所有连接的端口

   onconnect = function(e) {
     var port = e.ports[0];
     connections.push(port);

     port.onmessage = function(event) {
       // 处理消息
     };

     port.start();
   };
   ```
2. **使用：**
   主线程与Shared Worker之间的通信使用端口对象(port)，主要是通过 `postMessage()` 和 `onmessage`：

   ```javascript
   // 主线程
   mySharedWorker.port.postMessage('some data'); // 发送数据到Worker

   mySharedWorker.port.onmessage = function(e) { // 从Worker收到数据
     console.log('Message received from shared worker: ', e.data);
   };
   ```
3. **销毁：**
   Shared Worker的资源会在没有连接时自动释放，不需要手动销毁。

注意，由于Shared Worker的支持不如Web Worker那么广泛，加之其API比较复杂，实际开发中使用得比较少。如果你的网站需要在多个标签或iframe之间共享状态或数据，考虑使用其他机制例如LocalStorage、IndexedDB、BroadcastChannel API等。

# 共同点

* **同源限制** 分配给 Worker 运行的脚本文件，必须与主线程的脚本文件同源。在现代浏览器中，可以通过CORS（跨源资源共享）机制加载跨源脚本，或使用Blob URL/可执行的JavaScript字符串绕过这一限制。
* **DOM 限制** Worker无法直接访问或修改DOM。然而，Worker可以访问 `navigator`对象和 `location`对象的部分属性（可读不可写）
* **通信限制** Worker通过 `postMessage`接口和主线程沟通，并侦听 `message`事件来响应主线程的消息。这是双向的：主线程和Worker线程都可以发送消息和接收消息。
* **脚本限制** Worker中禁止使用那些对用户界面进行即时操作的方法如 `alert()`和 `confirm()`，但它可以使用 `XMLHttpRequest`或更现代的 `fetch()`方法来发送网络请求。
* **文件限制** Worker确实无法读取本地文件系统。然而，它们可以通过网络请求（如使用 `fetch`）来加载脚本。此外，通过主线程传输，Workers可以处理传导给它们的File对象或Blob对象。

## API

**共通的方法：**

* **postMessage()** ：用于发送数据给worker，或者在worker之间互相发送数据。
* **close() / terminate()** ：用于关闭worker。Web Worker和Shared Worker中通常用 `terminate()`来终止worker的执行，而Service Worker的关闭是通过浏览器控制，不需要显式调用这些方法。但是，你可以调用 `self.close()`来关闭Service Worker。

**Web Worker特有的方法和事件：**

* **importScripts()** ：允许worker导入一或多个脚本。
* **onmessage** ：在接收到消息时触发。
* **onerror** ：在发生错误时触发。
* **onmessageerror** ：在接收到的消息不能反序列化时触发。

**Service Worker特有的方法和事件：**

* **self.skipWaiting()** ：允许Service Worker在安装阶段跳过等待，立即进入激活阶段。
* **self.clients.claim()** ：允许Service Worker激活后立即控制打开的页面。
* **oninstall** ：在Service Worker安装时触发。
* **onactivate** ：在Service Worker激活时触发。
* **onfetch** ：在拦截到网络请求时触发。
* **onpush** ：在接收到服务器推送消息时触发。
* **onbackgroundfetchsuccess / onbackgroundfetchfail** ：在后台数据拉取成功/失败时触发。
* **onsync** ：在后台数据同步时触发。
* **onnotificationclick / onnotificationclose** ：在通知点击/关闭时触发。

**Shared Worker特有的方法和事件：**

* **onconnect** ：新的client连接到Shared Worker时触发。

### 扩展

[☄️ Service Worker](https://juejin.im/post/5b06a7b3f265da0dd8567513)

[💥 Shared Worker](https://www.zhuwenlong.com/blog/article/590ea64fe55f0f385f9a12e5)
