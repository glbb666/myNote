# 自定义prefetch

## 步骤

### 1. 同名文件执行prefetch

在跳转页面的时候，检测页面同名的后缀带.prefetch的文件。

prefetch文件存储在特定文件中，以页面名进行命名。

如果文件存在，执行文件。

### 2. prefetch文件内容

文件的内容是需要进行预请求的接口。

当请求成功时，把prefetch的结果存储在内存中，格式是页面名+接口名。

### 3. 读取值时机

把fetch方法做一个封装，某一个参数是需要prefetch，默认为false。

当prefetch的时候，去内存读取相应的值。

读取完值之后释放内存响应的值，避免内存占用。

## 实现

### 1. **创建 `.prefetch.js` 文件**

首先，为每个页面创建一个 `.prefetch.js` 文件，用于定义需要预请求的 API。例如：

```js
// Home.prefetch.js
export const fetchData = async () => {
  const response = await fetch('https://api.example.com/home');
  return response.json();
};

```

### 2. **预请求管理模块**

创建一个预请求管理模块，用于检测 `.prefetch.js` 文件并执行预请求操作，将结果存储在内存中。

```js
// prefetchManager.js
const cache = new Map();

export const executePrefetch = async (pageName) => {
  try {
    const prefetchModule = require(`./${pageName}.prefetch.js`);
    if (prefetchModule) {
      const data = await prefetchModule.fetchData();
      cache.set(`${pageName}_data`, data);
    }
  } catch (error) {
    console.error(`Prefetch failed for ${pageName}:`, error);
  }
};

export const getCachedData = (pageName, apiName) => {
  const cacheKey = `${pageName}_${apiName}`;
  const data = cache.get(cacheKey);
  if (data) {
    cache.delete(cacheKey); // 清除缓存
  }
  return data;
};

export const clearCache = (pageName) => {
  const keysToDelete = [];
  cache.forEach((_, key) => {
    if (key.startsWith(`${pageName}_`)) {
      keysToDelete.push(key);
    }
  });
  keysToDelete.forEach(key => cache.delete(key));
};

```

### 3. **在页面导航时触发预请求**

在导航逻辑中，检查是否存在同名 `.prefetch.js` 文件，并执行预请求。

```js
// navigationHandler.js
import { executePrefetch } from './prefetchManager';

export const navigateToPage = (pageName, navigation) => {
  executePrefetch(pageName).then(() => {
    navigation.navigate(pageName);
  });
};

```

### 4. **封装 `fetch` 方法**

封装一个自定义的 `fetch` 方法，支持从内存中读取预请求的结果。

```js
// customFetch.js
import { getCachedData } from './prefetchManager';

export const customFetch = async (url, options = {}, prefetch = false) => {
  const apiName = new URL(url).pathname; // 提取 API 名称
  const pageName = options.pageName; // 从参数中获取页面名称

  if (prefetch) {
    const cachedData = getCachedData(pageName, apiName);
    if (cachedData) {
      return cachedData; // 返回缓存的数据
    }
  }

  // 如果没有缓存，执行正常的请求
  const response = await fetch(url, options);
  return response.json();
};

```

### 5. **在页面组件中使用**

在页面组件的 `useEffect` 中使用封装的 `fetch` 方法。

```js
// HomePage.js
import React, { useEffect, useState } from 'react';
import { customFetch } from './customFetch';

const HomePage = () => {
  const [data, setData] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      const result = await customFetch('https://api.example.com/home', { pageName: 'Home' }, true);
      setData(result);
    };

    fetchData();

    // 清理缓存
    return () => {
      clearCache('Home');
    };
  }, []);

  return (
    <View>
      {/* 使用 data 渲染页面 */}
    </View>
  );
};

export default HomePage;

```

## 效果

首屏请求时间从3.9s->1.8s

任务页面从3s->1.2s

## QA

### 为什么这样做prefetch会快

因为是并行。
之前：在useEffect中请求。
现状：在页面跳转时检测到同名 `.prefetch.js` 文件并立即执行预请求操作。网络请求是在网络进程进行。页面渲染是在主进程进行，不会发生阻塞。

### 为什么选取fcp

因为一般请求是从页面的useEffect方法中发出。

但是埋点平台没有记录每个页面useEffect方法触发的数据。

fcp这个节点，有统一标准，且是性能指标中与useEffect方法时间最接近的节点，比useEffect略前一点。通过这个时间点可以推测出请求可以提前发出多久。
