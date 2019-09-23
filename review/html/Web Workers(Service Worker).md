## 什么是`Service Worker`

`Service Worker`本质上充当Web应用程序与浏览器之间的代理服务器，也可以在网络可用时作为浏览器和网络间的代理。它们旨在（除其他之外）使得能够创建有效的离线体验，拦截网络请求并基于网络是否可用以及更新的资源是否驻留在服务器上来采取适当的动作。他们还允许访问推送通知和后台同步`API`。

- `Service Worker`的本质是一个`Web Worker`，它独立于`JavaScript`主线程，因此它不能直接访问`DOM`，也不能直接访问`window`对象，但是，`Service Worker`可以访问`navigator`对象，也可以通过消息传递的方式（[postMessage](https://link.juejin.im?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FWindow%2FpostMessage)）与`JavaScript`主线程进行通信。
- `Service Worker`是一个网络代理，它可以控制`Web`页面的所有网络请求。
- `Service Worker`具有自身的生命周期，使用好`Service Worker`的关键是灵活控制其生命周期。

## `Service Worker`的作用

- 用于浏览器缓存
- 实现离线`Web APP`
- 消息推送

## `Service Worker`兼容性

`Service Worker`是现代浏览器的一个高级特性，它依赖于`fetch API`、`Cache Storage`、`Promise`等，其中，`Cache`提供了`Request / Response`对象对的存储机制，`Cache Storage`存储多个`Cache`。



![img](https://user-gold-cdn.xitu.io/2018/9/18/165ece3aa04979ff?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 示例

在了解`Service Worker`的原理之前，先来看一段`Service Worker`的示例：

```
self.importScripts('./serviceworker-cache-polyfill.js');//

var urlsToCache = [
  '/',
  '/index.js',
  '/style.css',
  '/favicon.ico',
];

var CACHE_NAME = 'counterxing';

self.addEventListener('install', function(event) {
  self.skipWaiting();
  event.waitUntil(
    caches.open(CACHE_NAME)
    .then(function(cache) {
      return cache.addAll(urlsToCache);
    })
  );
});

self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.match(event.request)
    .then(function(response) {
      if (response) {
        return response;
      }
      return fetch(event.request);
    })
  );
});


self.addEventListener('activate', function(event) {
  var cacheWhitelist = ['counterxing'];

  event.waitUntil(
    caches.keys().then(function(cacheNames) {
      return Promise.all(
        cacheNames.map(function(cacheName) {
          if (cacheWhitelist.indexOf(cacheName) === -1) {
            return caches.delete(cacheName);
          }
        })
      );
    })
  );
});

复制代码
```

下面开始逐段逐段地分析，揭开`Service Worker`的神秘面纱：

## `polyfill`

首先看第一行：`self.importScripts('./serviceworker-cache-polyfill.js');`，这里引入了[Cache API](https://link.juejin.im?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FAPI%2FCache)的一个[polyfill](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fdominiccooney%2Fcache-polyfill)，这个`polyfill`支持使得在较低版本的浏览器下也可以使用`Cache Storage API`。想要实现`Service Worker`的功能，一般都需要搭配`Cache API`代理网络请求到缓存中。

在`Service Worker`线程中，使用`importScripts`引入`polyfill`脚本，目的是对低版本浏览器的兼容。

## `Cache Resources List` And `Cache Name`

之后，使用一个`urlsToCache`列表来声明需要缓存的静态资源，再使用一个变量`CACHE_NAME`来确定当前缓存的`Cache Storage Name`，这里可以理解成`Cache Storage`是一个`DB`，而`CACHE_NAME`则是`DB`名：

```
var urlsToCache = [
  '/',
  '/index.js',
  '/style.css',
  '/favicon.ico',
];

var CACHE_NAME = 'counterxing';
复制代码
```

## `Lifecycle`

`Service Worker`独立于浏览器`JavaScript`主线程，有它自己独立的生命周期。

如果需要在网站上安装`Service Worker`，则需要在`JavaScript`主线程中使用以下代码引入`Service Worker`。

```
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js').then(function(registration) {
    console.log('成功安装', registration.scope);
  }).catch(function(err) {
    console.log(err);
  });
}
复制代码
```

此处，一定要注意`sw.js`文件的路径，在我的示例中，处于当前域根目录下，这意味着，`Service Worker`和网站是同源的，可以为当前网站的所有请求做代理，如果`Service Worker`被注册到`/imaging/sw.js`下，那只能代理`/imaging`下的网络请求。

可以使用`Chrome`控制台，查看当前页面的`Service Worker`情况：



![img](https://user-gold-cdn.xitu.io/2018/9/18/165ece3aa051666b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



安装完成后，`Service Worker`会经历以下生命周期：

1. 下载（`download`）
2. 安装（`install`）
3. 激活（`activate`）

- 用户首次访问`Service Worker`控制的网站或页面时，`Service Worker`会立刻被下载。之后至少每`24`小时它会被下载一次。它可能被更频繁地下载，不过每`24`小时一定会被下载一次，以避免不良脚本长时间生效。
- 在下载完成后，开始安装`Service Worker`，在安装阶段，通常需要缓存一些我们预先声明的静态资源，在我们的示例中，通过`urlsToCache`预先声明。
- 在安装完成后，会开始进行激活，浏览器会尝试下载`Service Worker`脚本文件，下载成功后，会与前一次已缓存的`Service Worker`脚本文件做对比，如果与前一次的`Service Worker`脚本文件不同，证明`Service Worker`已经更新，会触发`activate`事件。完成激活。

如图所示，为`Service Worker`大致的生命周期：



![img](https://user-gold-cdn.xitu.io/2018/9/18/165ece3aa10a4b5b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



### `install`

在安装完成后，尝试缓存一些静态资源：

```
self.addEventListener('install', function(event) {
  self.skipWaiting();
  event.waitUntil(
    caches.open(CACHE_NAME)
    .then(function(cache) {
      return cache.addAll(urlsToCache);
    })
  );
});
复制代码
```

首先，`self.skipWaiting()`执行，告知浏览器直接跳过等待阶段，淘汰过期的`sw.js`的`Service Worker`脚本，直接开始尝试激活新的`Service Worker`。

然后使用`caches.open`打开一个`Cache`，打开后，通过`cache.addAll`尝试缓存我们预先声明的静态文件。

### 监听`fetch`，代理网络请求

页面的所有网络请求，都会通过`Service Worker`的`fetch`事件触发，`Service Worker`通过`caches.match`尝试从`Cache`中查找缓存，缓存如果命中，则直接返回缓存中的`response`，否则，创建一个真实的网络请求。

```
self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.match(event.request)
    .then(function(response) {
      if (response) {
        return response;
      }
      return fetch(event.request);
    })
  );
});
复制代码
```

如果我们需要在请求过程中，再向`Cache Storage`中添加新的缓存，可以通过`cache.put`方法添加，看以下例子：

```
self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.match(event.request)
    .then(function(response) {
      // 缓存命中
      if (response) {
        return response;
      }

      // 注意，这里必须使用clone方法克隆这个请求
      // 原因是response是一个Stream，为了让浏览器跟缓存都使用这个response
      // 必须克隆这个response，一份到浏览器，一份到缓存中缓存。
      // 只能被消费一次，想要再次消费，必须clone一次
      var fetchRequest = event.request.clone();

      return fetch(fetchRequest).then(
        function(response) {
          // 必须是有效请求，必须是同源响应，第三方的请求，因为不可控，最好不要缓存
          if (!response || response.status !== 200 || response.type !== 'basic') {
            return response;
          }

          // 消费过一次，又需要再克隆一次
          var responseToCache = response.clone();
          caches.open(CACHE_NAME)
            .then(function(cache) {
              cache.put(event.request, responseToCache);
            });
          return response;
        }
      );
    })
  );
});
复制代码
```

> 在项目中，一定要注意控制缓存，接口请求一般是不推荐缓存的。所以在我自己的项目中，并没有在这里做动态的缓存方案。

### `activate`

`Service Worker`总有需要更新的一天，随着版本迭代，某一天，我们需要把新版本的功能发布上线，此时需要淘汰掉旧的缓存，旧的`Service Worker`和`Cache Storage`如何淘汰呢？

```
self.addEventListener('activate', function(event) {
  var cacheWhitelist = ['counterxing'];

  event.waitUntil(
    caches.keys().then(function(cacheNames) {
      return Promise.all(
        cacheNames.map(function(cacheName) {
          if (cacheWhitelist.indexOf(cacheName) === -1) {
            return caches.delete(cacheName);
          }
        })
      );
    })
  );
});
复制代码
```

1. 首先有一个白名单，白名单中的`Cache`是不被淘汰的。
2. 之后通过`caches.keys()`拿到所有的`Cache Storage`，把不在白名单中的`Cache`淘汰。
3. 淘汰使用`caches.delete()`方法。它接收`cacheName`作为参数，删除该`cacheName`所有缓存。

## sw-precache-webpack-plugin

[sw-precache-webpack-plugin](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fgoldhand%2Fsw-precache-webpack-plugin)是一个`webpack plugin`，可以通过配置的方式在`webpack`打包时生成我们想要的`sw.js`的`Service Worker`脚本。

一个最简单的配置如下：

```
var path = require('path');
var SWPrecacheWebpackPlugin = require('sw-precache-webpack-plugin');

const PUBLIC_PATH = 'https://www.my-project-name.com/';  // webpack needs the trailing slash for output.publicPath

module.exports = {

  entry: {
    main: path.resolve(__dirname, 'src/index'),
  },

  output: {
    path: path.resolve(__dirname, 'src/bundles/'),
    filename: '[name]-[hash].js',
    publicPath: PUBLIC_PATH,
  },

  plugins: [
    new SWPrecacheWebpackPlugin(
      {
        cacheId: 'my-project-name',
        dontCacheBustUrlsMatching: /\.\w{8}\./,
        filename: 'service-worker.js',
        minify: true,
        navigateFallback: PUBLIC_PATH + 'index.html',
        staticFileGlobsIgnorePatterns: [/\.map$/, /asset-manifest\.json$/],
      }
    ),
  ],
}
复制代码
```

在执行`webpack`打包后，会生成一个名为`service-worker.js`文件，用于缓存`webpack`打包后的静态文件。

一个最简单的[示例](https://link.juejin.im?target=https%3A%2F%2Fgithub.com%2Fgoldhand%2Fsw-precache-webpack-plugin%2Ftree%2Fmaster%2Fexamples)。

## `Service Worker Cache` VS `Http Cache`

对比起`Http Header`缓存，`Service Worker`配合`Cache Storage`也有自己的优势：

1. 缓存与更新并存：每次更新版本，借助`Service Worker`可以立马使用缓存返回，但与此同时可以发起请求，校验是否有新版本更新。
2. 无侵入式：`hash`值实在是太难看了。
3. 不易被冲掉：`Http`缓存容易被冲掉，也容易过期，而`Cache Storage`则不容易被冲掉。也没有过期时间的说法。
4. 离线：借助`Service Worker`可以实现离线访问应用。

但是缺点是，由于`Service Worker`依赖于`fetch API`、依赖于`Promise`、`Cache Storage`等，兼容性不太好。

## 后话

本文只是简单总结了`Service Worker`的基本使用和使用`Service Worker`做客户端缓存的简单方式，然而，`Service Worker`的作用远不止于此，例如：借助`Service Worker`做离线应用、用于做网络应用的推送（可参考[push-notifications](https://link.juejin.im?target=https%3A%2F%2Fdevelopers.google.com%2Fweb%2Ffundamentals%2Fcodelabs%2Fpush-notifications%2F)）等。

甚至可以借助`Service Worker`，对接口进行缓存，在我所在的项目中，其实并不会做的这么复杂。不过做接口缓存的好处是支持离线访问，对离线状态下也能正常访问我们的`Web`应用。

`Cache Storage`和`Service Worker`总是分不开的。`Service Worker`的最佳用法其实就是配合`Cache Storage`做离线缓存。借助于`Service Worker`，可以轻松实现对网络请求的控制，对于不同的网络请求，采取不同的策略。例如对于`Cache`的策略，其实也是存在多种情况。例如可以优先使用网络请求，在网络请求失败时再使用缓存、亦可以同时使用缓存和网络请求，一方面检查请求，一方面有检查缓存，然后看两个谁快，就用谁。