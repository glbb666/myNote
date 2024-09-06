# 是什么

Cache Storage 是浏览器中的一个存储机制，它允许 Service Worker 将网络请求的响应对象缓存起来。

# CacheStorage存储什么

Cache Storage存储请求和对应的响应。响应是不能直接修改的，但是可以通过clone的方式进行修改。

# Cache存储在哪里

Cache Storage 存储在浏览器的磁盘中，属于浏览器的持久化存储

在实际使用中，浏览器可能会将频繁访问的缓存数据暂时加载到内存中，以加快访问速度。这种机制由浏览器管理，用户和开发者无需直接控制。

# Cache的过期时间

## **默认情况下缓存不会过期**

Cache Storage 中的资源不会自动失效，这意味着一旦缓存被添加到 Cache Storage 中，它将一直保留，直到开发者明确删除或替换它。

## 过期管理

### 手动设置过期时间

对比当前时间和服务端的响应头的时间，确定是否过期

```js
const CACHE_NAME = 'my-cache-v1';
const MAX_CACHE_AGE = 24 * 60 * 60 * 1000; // 缓存时间 24 小时

self.addEventListener('fetch', event => {
  event.respondWith(
    caches.open(CACHE_NAME).then(async cache => {
      const cachedResponse = await cache.match(event.request);
      if (cachedResponse) {
        const age = Date.now() - new Date(cachedResponse.headers.get('date')).getTime();
        if (age < MAX_CACHE_AGE) {
          return cachedResponse; // 如果缓存不过期，使用缓存
        }
      }
      const networkResponse = await fetch(event.request);
      cache.put(event.request, networkResponse.clone()); // 更新缓存
      return networkResponse;
    })
  );
});

```

这个策略比较简单一般不怎么用。

### Webpack版本管理

使用 Webpack 的 hash 机制进行静态资源缓存 ，通常不需要再手动管理 Cache Storage 的过期时间。

Webpack 的 hash 机制通过在文件名中包含内容 hash 值，确保每次文件内容变化时，生成的文件名也是不同的。这意味着：

1. **文件内容不变时，文件名不变** ：缓存不会过期，浏览器会继续使用缓存的文件，减少网络请求。
2. **文件内容变化时，文件名变动** ：这会导致浏览器请求新的文件，而不是使用缓存文件。因此，旧的缓存文件虽然还在 Cache Storage 中，但不会再被使用。

由于这些 hash 文件名是唯一的、基于内容的标识符，旧版本文件通常可以留在缓存中，直到有策略明确去清理它们。这样，静态资源的有效期基本由文件内容的变化来决定。

但是，Webpack的文件名是hash生成，那么端在代码中怎么能知道Webpack的完整文件名呢？

#### 使用 `webpack-assets-manifest` 插件来生成清单文件

```js
// webpack.config.js
const WebpackAssetsManifest = require('webpack-assets-manifest');

module.exports = {
  // 其他配置
  plugins: [
    new WebpackAssetsManifest({
      output: 'asset-manifest.json',
      publicPath: true, // 确保路径包含公共路径
    }),
    // 其他插件
  ],
};

```

生成的清单文件如下，是文件名和hash名的映射

```js
{
  "main.js": "main.abc123.js",
  "style.css": "style.def456.css",
  "logo.png": "logo.ghi789.png"
}

```

在 Service Worker 中，可以通过 `fetch` 请求该清单文件，动态更新缓存。

#### asset-manifest不走缓存

当 Webpack 重新打包生成新的资源文件时，`asset-manifest.json` 会更新。Service Worker 在下一次安装时会获取新的清单文件，并缓存新的资源。同时旧的资源将自动被移除。

虽然webpack能解决文件缓存更新的问题。

但是有的时候还是会存在旧缓存问题，就比如某个文件在新版本已经被删除了。仅使用Webpack进行版本管理是无法进行更新的。

结合Cache版本管理可以解决这个问题。

### **Cache版本管理**

通过更新缓存版本号来强制资源过期。每次发布新版本时更新缓存名称，例如将 `CACHE_NAME` 从 `'my-cache-v1'` 改为 `'my-cache-v2'`，从而使旧版本缓存失效。

```js
self.addEventListener('activate', (event) => {
  const cacheWhitelist = [CACHE_NAME]; // 保留当前版本的缓存
  
  event.waitUntil(
    caches.keys().then((cacheNames) => {
      return Promise.all(
        cacheNames.map((cacheName) => {
          if (!cacheWhitelist.includes(cacheName)) {
            console.log('Deleting old cache:', cacheName);
            return caches.delete(cacheName); // 删除旧版本缓存
          }
        })
      );
    })
  );
});

```

Webpack版本管理一般和Cache版本管理一起使用。在长期缓存的同时，到下一个版本就可以直接卸载。

# Cache的缓存溢出怎么处理

### 1. **存储空间限制**

* **限制因浏览器和设备而异** ：不同的浏览器有不同的存储限制。一般来说，浏览器会为每个域名分配总存储空间，这个空间是给包括 Cache Storage、IndexedDB、LocalStorage 等所有存储机制共同使用的。存储空间的大小也取决于设备的可用磁盘空间，一般为几个到几十 MB。
* **配额管理** ：部分浏览器使用配额管理系统来限制每个域名可使用的存储空间。Chrome 和 Firefox 会将可用存储空间的一定比例分配给每个域名，而 Safari 的限制通常更严格。

### 2. **溢出检查**

浏览器不会在写入数据时抛出错误，而是在超出配额时直接删除旧的或最少使用的数据来腾出空间。要主动进行溢出检查，可以使用以下策略：

* **通过 `estimate()` 检查存储空间** ：可以使用 `navigator.storage.estimate()` 方法来估计当前的存储使用情况和总配额。一般来说缓存空间会限制在200MB。也要考虑剩余可用空间，两者取小。

```js
navigator.storage.estimate().then(estimate => {
  console.log(`使用空间：${estimate.usage} bytes`);
  console.log(`可用总空间：${estimate.quota} bytes`);
  const availableSpace = total - used;  // 剩余可用空间
});
```

* **缓存条目数量限制** ：为了防止缓存过多数据，开发者可以设置缓存条目的数量限制，超过这个数量时删除旧的条目。一般是设置在**50 - 100 个条目。**
* **可以使用lru先进先出的策略：响应是不能直接修改的，但是可以通过clone的方式进行修改。在clone的时候给响应加上时间戳。每次使用或者获取到缓存，都更新时间戳。**

```js
async function limitCacheSize(cacheName, maxItems) {
  const cache = await caches.open(cacheName);
  const keys = await cache.keys();
  if (keys.length > maxItems) {
    cache.delete(keys[0]);
  }
}

```

一般需要把存储空间和缓存条目结合起来使用，这样可以提升用户体验 ，同时保证缓存不会过度占用用户的存储空间。

### 3. **存储空间溢出的情况**

如果存储空间超出浏览器的配额限制，浏览器可能会自动清理旧数据，或拒绝新数据的存储。

# 为什么

* **提升性能** ：静态资源（如 HTML、CSS、JS、图片等）一般不会频繁改变，将其缓存后加快页面加载速度。
* **支持离线访问** ：通过缓存静态资源，用户可以在没有网络连接的情况下继续使用应用。
* **减少服务器压力**

# 怎么做

**在 `install` 事件中预缓存资源** ：
当 Service Worker 首次安装时，可以在 `install` 事件中将静态资源缓存到 Cache Storage。

```js
self.addEventListener('install', event => {
  event.waitUntil(
    caches.open('static-cache-v1').then(cache => {
      return cache.addAll([
        '/',
        '/index.html',
        '/styles.css',
        '/script.js',
        '/images/logo.png'
      ]);
    })
  );
});

```

**在 `fetch` 事件中拦截请求并返回缓存的资源** ：
在 `fetch` 事件中，你可以检查请求是否已经缓存，并返回缓存的资源。如果资源未缓存，可以选择通过网络请求获取并缓存它。

```js
self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request).then(response => {
      return response || fetch(event.request).then(fetchResponse => {
        return caches.open('static-cache-v1').then(cache => {
          cache.put(event.request, fetchResponse.clone());
          return fetchResponse;
        });
      });
    })
  );
});

```

通过这种方式，静态资源将被缓存到浏览器的 Cache Storage 中，可以在离线时访问并加快页面的加载速度。
