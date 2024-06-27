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

ServiceWorker遵循同源策略

- 注册限制：只能为同源下服务的页面注册 Service Worker。
- 作用域限制：不能在 Service Worker 中拦截或缓存跨源资源（除非这些资源允许跨源使用，如使用 CORS 头部的资源）。

举个例子，假设你在网站根目录下注册了一个 Service Worker。

这意味着Service Worker的作用范围是整个网站（因为根目录是整个网站的基础路径）。因此，该 Service Worker 将会控制所有以 `https://YOUR_DOMAIN/`开头的页面。如果Service Worker是在一个子目录下注册的，那么它的作用范围将被限制在该子目录下。

#### Service Worker的用法

Service Worker的预加载可以分为安装阶段和监听阶段，分别对应静态资源和业务请求。

- 静态资源的prefetch
  **在安装阶段**，我们可以提前下载某一个静态资源作为缓存。

  `caches` 是一个全局变量，它是 Cache Storage API 的一部分，提供了一个脚本可用的缓存接口。通过这个接口，你可以在支持的浏览器中创建、读取、修改和删除特定的缓存。`caches` 对象只在 Service Worker、Web Workers 和 Window 上下文中可用（虽然在 Window 上下文中使用它是有限的）。
- 业务请求的prefetch**在监听阶段中**，我们可以监听页面的fetch事件，但是并不对资源进行存储。

#### 静态资源的prefetch的例子

Service Worker比较复杂，它有生命周期。下面是一个进行缓存资源的示例，通过这个例子可以体会到Service Worker生命周期的顺序。

接下来我们来看看流程：

1. **注册**：

   调用 `navigator.serviceWorker.register()` 函数进行注册。

   最好在应用的入口文件（如 React 的 `index.js`）中进行注册。因为 `Service Worker`是在浏览器中全局运行的，所以不管在 `SPA`还是 `MPA`，都会接管多个页面，所以 `Service Worker`越早注册就能越早接管页面。

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
2. **安装** ：注册成功后，Service Worker进入 `install`状态。

   - **打开缓存** (`caches.open`)：打开一个名为 `CACHE_NAME`的缓存。如果这个缓存不存在，浏览器会创建一个新的缓存。
   - **prefetch，添加静态资源到缓存** (`cache.addAll`)：`cache.addAll` 方法接受一个 URL 数组（这些是你想要缓存的资源的路径），然后执行网络请求去下载这些资源，并将响应对象存储在打开的缓存中。
   - `caches.open` 和 `cache.addAll` 都是异步的Promise。
   - **等待异步操作完成** (`event.waitUntil`)：`event.waitUntil` 它内部的任务完成之前，Service Worker 不会进入下一个生命周期阶段。这里用来等待promise执行。

   ```js
   // Service Worker 文件内
   const CACHE_NAME = 'static-cache-v1';
   const STATIC_ASSETS = [
     '/index.html',
     '/styles.css',
     '/script.js',
     '/logo.png'
   ];

   self.addEventListener('install', event => {
     event.waitUntil(
       caches.open(CACHE_NAME)
         .then(cache => {
           return cache.addAll(STATIC_ASSETS);
         })
     );
   });

   ```
3. **激活**：安装步骤后面紧随的是激活阶段。这一个阶段，Service Worker可以清理旧缓存，执行更新后的清理工作等。只有在这之后，Service Worker才有能力控制网页。

   ```js
   // 激活 Service Worker 并且清理旧缓存
   self.addEventListener('activate', event => {
     event.waitUntil(
       caches.keys().then(cacheNames => {
         return Promise.all(
           cacheNames.filter(cacheName => cacheName !== CACHE_NAME)
             .map(cacheName => caches.delete(cacheName))
         );
       })
     );
   });

   ```
4. **控制**：Service Worker 将开始控制激活之后才打开的页面。此时为了让 Service Worker 控制页面，通常需要重新加载页面。或者你也可以调用 `clients.claim()`，这个方法使得活跃的 Service Worker 能够立即控制未被Service Worker 控制的任何客户端。

```js
   // Service Worker 文件内
   self.addEventListener('activate', function(event) {
     event.waitUntil(
       // 异步操作，返回一个promise
       // 表示 Service Worker 取得页面的控制权，不需再重启页面
       clients.claim() 
     );
   });
```

5. **监听** ：激活后，Service Worker 能监听并响应 `fetch` 事件。更具体的见下面业务资源prefetch的例子。

```js
self.addEventListener('fetch', event => {
  if (STATIC_ASSETS.includes(new URL(event.request.url).pathname)) {
    event.respondWith(
      caches.match(event.request).then(cachedResponse => {
        return cachedResponse || fetch(event.request).then(networkResponse => {
          return caches.open(CACHE_NAME).then(cache => {
            cache.put(event.request, networkResponse.clone());
            return networkResponse;
          });
        });
      })
    );
  }
});
```

6. **终止 和 更新**：回调完成之后 Service Worker 会自动终止。不过它**没有销毁**，而是在后台自动检查更新。如果 Service Worker 文件发生变化，浏览器会认为这是新的 Service Worker 并开始安装过程。更新后的 Service Worker 将经历相同的生命周期。

#### 业务资源prefetch的例子

区别在于：

1. `Service Worker`文件内容，需要把需要进行prefetch的页面方法进行调用。

   这里把不同页面的 `prefetch`逻辑剥离到不同的文件中，用 `importScripts`引入。使用 `importScripts()` 引入的脚本文件需要与 Service Worker 在同一域下

   ```js
   importScripts('homePreload.js');
   importScripts('aboutPreload.js');

   self.addEventListener('fetch', event => {
     // 调用 homePreload.js 中的处理函数
     homePreload.handleFetch(event);
     // 调用 aboutPreload.js 中的处理函数
     aboutPreload.handleFetch(event);
   });

   ```

   homePreload.js

   ```js
   // 定义处理函数
   const homePreload = {
     handleFetch: function(event) {
       if (event.request.url.includes('/api/home-data')) {
         event.respondWith((async () => {
           try {
             const apiUrls = ['/api/data1', '/api/data2', '/api/data3'];
             const fetchPromises = apiUrls.map(url => fetch(url));
             const responses = await Promise.all(fetchPromises);
             const data = await Promise.all(responses.map(res => res.json()));

             return new Response(JSON.stringify({data}), {
               headers: { 'Content-Type': 'application/json' }
             });
           } catch (error) {
             return new Response('{"error": "请求失败，请稍候重试"}', {
               headers: { 'Content-Type': 'application/json' }
             });
           }
         })());
       }
     }
   };

   // 将处理函数导出到全局作用域
   self.homePreload = homePreload;

   ```

   aboutPreload.js

   ```javascript
   // 定义处理函数
   const aboutPreload = {
     handleFetch: function(event) {
       if (event.request.url.includes('/api/about-data')) {
         event.respondWith((async () => {
           try {
             const apiUrls = ['/api/about1', '/api/about2'];
             const fetchPromises = apiUrls.map(url => fetch(url));
             const responses = await Promise.all(fetchPromises);
             const data = await Promise.all(responses.map(res => res.json()));

             return new Response(JSON.stringify({data}), {
               headers: { 'Content-Type': 'application/json' }
             });
           } catch (error) {
             return new Response('{"error": "请求失败，请稍候重试"}', {
               headers: { 'Content-Type': 'application/json' }
             });
           }
         })());
       }
     }
   };

   // 将处理函数导出到全局作用域
   self.aboutPreload = aboutPreload;

   ```
2. 页面，使用自定义hook注册监听事件，对Service Worker返回的数据进行监听。

```js
   import { useState, useEffect } from 'react';

   function useServiceWorkerMessage() {
     // 用于存储从 Service Worker 收到的消息数据
     const [messageData, setMessageData] = useState(null);

     useEffect(() => {
       // 定义处理 Service Worker 消息的函数
       const handleMessage = (event) => {
         console.log('收到 Service Worker 的消息:', event.data);
         setMessageData(event.data);
       };

       // 添加消息事件监听到 navigator.serviceWorker
       if (navigator.serviceWorker && navigator.serviceWorker.controller) {
         navigator.serviceWorker.addEventListener('message', handleMessage);
       }

       // 组件卸载或重新执行该 useEffect 时的清理函数
       return () => {
         if (navigator.serviceWorker && navigator.serviceWorker.controller) {
           navigator.serviceWorker.removeEventListener('message', handleMessage);
         }
       };
     }, []); // 该 useEffect 没有依赖，仅在组件挂载时执行

     return messageData;
   }
```

## Service Worker的缓存策略

* **缓存优先** : 优先返回缓存中的内容，如果没有缓存再通过网络请求。
* **网络优先** : 优先通过网络请求内容，如果网络请求失败则返回缓存中的内容。
* **缓存更新** : 返回缓存中的内容，同时在后台进行更新。（意思是每次都返回上一次更新的内容）
* **仅缓存**: 只从缓存中获取内容，不进行网络请求。
* **仅网络**: 只通过网络请求内容，不使用缓存。

### 自定义缓存策略

带有过期时间的缓存策略

* 在存入缓存时添加时间戳 。
* 在读取缓存时检查时间戳 。

```js
self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request).then(cachedResponse => {
      const now = Date.now();

      if (cachedResponse) {
        // 获取自定义的缓存时间戳
        const cachedTime = parseInt(cachedResponse.headers.get('sw-cache-timestamp'), 10);

        // 如果缓存的时间超过 1 小时（3600000 毫秒）
        if (now - cachedTime > 3600000) {
          // 超过1小时，进行网络请求并更新缓存
          return fetchAndCache(event.request);
        } else {
          // 在1小时内，返回缓存
          return cachedResponse;
        }
      } else {
        // 缓存中没有内容，进行网络请求
        return fetchAndCache(event.request);
      }
    })
  );
});

function fetchAndCache(request) {
  return fetch(request).then(networkResponse => {
    return caches.open('dynamic-cache').then(cache => {
      const clonedResponse = networkResponse.clone();
      const headers = new Headers(clonedResponse.headers);
      headers.append('sw-cache-timestamp', Date.now().toString());

      const newResponse = new Response(clonedResponse.body, {
        status: clonedResponse.status,
        statusText: clonedResponse.statusText,
        headers: headers
      });

      cache.put(request, newResponse.clone());
      return newResponse;
    });
  }).catch(error => {
    // 处理网络请求错误
    console.error('Fetching failed:', error);
    throw error;
  });
}

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
