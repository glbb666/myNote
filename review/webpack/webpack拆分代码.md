# 什么是代码拆分

代码拆分的目的是将代码分成多个chunk，这些chunk可以按需加载，从而减少初始加载时间和提升页面性能。

# 为什么要代码拆分

使用 Webpack 进行代码拆分可以显著优化应用的加载性能，确保首页只加载必要的代码，其余代码按需加载。

# 代码拆分的情景

## 1. 入口点拆分

Webpack 的入口点配置允许你为每个入口文件生成单独的包。

```js
module.exports = {
  entry: {
    main: './src/index.js',
    vendor: './src/vendor.js',
  },
  output: {
    filename: '[name].bundle.js',
    path: __dirname + '/dist',
  },
  // 其他配置
};

```

这将生成 `main.bundle.js` 和 `vendor.bundle.js` 两个文件。

**`vendor` 文件**

* 通常包含第三方库或插件，例如 `lodash`、`react`、`react-dom` 等。
* 这些库通常不会频繁更新，可以更好地利用缓存。

#### 显示数组配置

提供一个字符串数组时，Webpack 会遍历 `node_modules` 中对应的包名，并将这些包打包到 `vendor` 文件中。

```js
module.exports = {
  entry: {
    main: './src/index.js',
    vendor: ['lodash', 'react', 'react-dom'], // 显式配置第三方库作为入口点
  },
  output: {
    filename: '[name].[contenthash].js',
    path: path.resolve(__dirname, 'dist'),
  }
};

```

## 2. 动态导入

当应用程序使用动态导入（`import()`）时，Webpack 会为这些动态导入的模块生成独立的 Chunk，并在运行时按需加载。

```js
// src/index.js
import('./module').then(module => {
  module.default();
});

// src/module.js
export default function() {
  console.log('Module loaded!');
}

```

例如：Webpack 会自动将 `module.js` 拆分成一个独立的包，并在需要时加载。

## 3. `splitChunks`进行公共依赖拆分

### 为什么要拆分公共依赖？

当没有拆分公共依赖时，所有依赖会被重复打包到入口文件中。

```js
module.exports = {
  entry: {
    main: './src/index.js',
    admin: './src/admin.js',
  },
  output: {
    filename: '[name].[contenthash].js',
    path: path.resolve(__dirname, 'dist'),
  },
};

```

在这个配置中，如果 `main.js` 和 `admin.js` 都依赖于 `lodash`，`lodash` 会被分别打包到 `main.[contenthash].js` 和 `admin.[contenthash].js` 中。**会导致重复加载**。

### 怎么拆分公共依赖？

通过 `optimization.splitChunks` 插件可以将公共依赖拆分成单独的包。

```js
module.exports = {
  // 其他配置
  optimization: {
    splitChunks: {
      chunks: 'all',// 适用于所有 chunk
    },
  },
};

```

公共依赖一般有两部分，本地公共依赖和第三方公共依赖，第三方公共依赖常被放入vendor文件中，之前入口点拆分的时候就是这样做的。

**本地公共文件**

* 通常包含项目内部的共享模块或组件，例如在多个入口点中复用的业务逻辑或 UI 组件。
* 这些模块可能会更频繁地更新，因此需要更细粒度的控制和优化。

这次我们不通过入口点拆分vendor文件，而是通过splitChunks的cacheGroups分组进行拆分。

#### `chunks`

* **含义** ：指定哪些类型的 chunk 会被拆分（同步、异步、或所有）。
* **最佳实践** ：
* **`'all'`** ：适用于需要全面拆分所有代码（同步和异步）的情况。
* **`'async'`** ：适用于只拆分异步加载的代码，减少初始加载时的代码量。
* **`'initial'`** ：适用于只拆分同步加载的代码，适合希望优化初始加载性能的场景。

#### `minSize`

* **含义** ：指定 chunk 的最小体积，低于此体积的模块不会被拆分。
* **合理的值** ：通常设置为 30KB 到 50KB。
* **为什么**：过小的 `minSize` 可能导致生成过多的小 chunk，影响加载性能。

#### `maxSize`

* **含义** ：指定 chunk 的最大体积，超过此体积的模块将被拆分成更小的 chunk。
* **合理的值**：通常设置为 100KB 到 200KB。
* **为什么** ：防止单个文件过大，影响加载速度和用户体验。

#### `minChunks`

* **含义** ：指定一个模块被包含到 chunk 中的最小次数。只有当模块被多个 chunk 引用时，它才会被拆分到公共 chunk 中。
* **合理的值**：通常设置为 2
* **为什么** ：提高重复利用率

#### `maxAsyncRequests`

* **含义** ：指定在单个请求中允许的最大异步请求数。超过此数目的异步请求将被限制。
* **合理的值**：通常设置为 5 到 10。
* **避免过多请求** ：防止生成过多的异步请求影响性能。

#### `maxInitialRequests`

* **含义** ：指定在初始加载中允许的最大请求数。超过此数目的请求将被限制。
* **合理的值**：通常设置为 3 到 5
* **为什么**：减少初始加载请求数

#### `cacheGroups` 配置

可以在splitChunks中使用cacheGroups配置分组。一般配置这两个分组

* **`vendors` cache group** ：`test: /[\\/]node_modules[\\/]/` 表示匹配所有在 `node_modules` 目录中的模块，这些模块会被提取到 `vendors.[contenthash].js` 文件中。这个配置确保了第三方库被统一打包到一个 `vendor` chunk 中。
* **`default` cache group** ：这个配置用于提取本地共享模块。`minChunks: 2` 表示至少被两个模块引用的本地模块会被提取到一个单独的 chunk 中，避免重复打包。

```js
module.exports = {
  entry: {
    main: './src/index.js',
    admin: './src/admin.js',
  },
  output: {
    filename: '[name].[contenthash].js',
    path: path.resolve(__dirname, 'dist'),
  },
  optimization: {
    splitChunks: {
      chunks: 'all', // 适用于所有 chunk
      cacheGroups: {
        vendors: {
          test: /[\\/]node_modules[\\/]/,
          priority: -10, // 优先级
          chunks: 'all',//确保所有类型的模块都被提取到公共 chunk 中，不管是同步还是异步
        },
        default: {
          minChunks: 2,//至少被两个模块引用才有必要拆分，可以减少不必要的公共chunk
          priority: -20,
          reuseExistingChunk: true,//复用已经存在的 chunk
	  //这里没有手动配置chunks是因为走的是Webpack的默认配置
        },
      },
    },
  },
};


```

##### 配置优先级确定

 `splitChunks` 用 `priority`来控制不同 cache group 的优先级。优先级高的先拆分。

一般是按以下策略来给优先级：

**`vendors` cache group** （高优先级）

* Webpack 会首先将 `lodash` 和其他第三方库提取到 `vendors` chunk 中。这个 chunk 是相对稳定的，不会因为本地模块的变动而改变。
* 提取到 `vendors` chunk 中有助于缓存利用，减少重复加载。

**`default` cache group** （低优先级）

* 接着，Webpack 会处理本地共享模块（如应用中的业务逻辑模块）。
* 如果这些本地模块已经被拆分到 `vendors` chunk 中（因为它们复用了第三方库），`default` cache group 会尽量复用已经存在的 chunk，而不是重复打包。

##### 拆出多个包的情况

在 `cacheGroups` 中配置时，如果同一个 `cacheGroup` 中的资源超过了设定的 `maxSize`，Webpack 会自动拆分出多个包。

Webpack 会使用类似 `[缓存组name]~[入口name]~[chunkname].js` 的命名方式来区分这些拆分出来的文件。

##### 限制并行请求数

**maxAsyncRequests** : 限制的是异步加载时（例如使用 `import()` 动态导入）的最大并行请求数。如果拆分出的包过多，Webpack 会限制最多有多少个包可以同时通过异步请求加载，超出的部分会被合并。

**maxInitialRequests** : 限制的是初始页面加载时（同步加载）的最大并行请求数。也就是说，当你首次加载页面时，Webpack 会限制可以同时请求的文件数量。

#### 最后实现

```js
module.exports = {
  optimization: {
    splitChunks: {
      chunks: 'all', // 将异步和同步模块都进行代码分离
      minSize: 30000, // 模块最小体积，只有超过这个体积的模块才会被分割
      minRemainingSize: 0, // 确保拆分后剩余的块不小于这个体积
      maxSize: 0, // 尝试将拆分后的块限制在最大体积，实际体积可能会大于该值
      minChunks: 1, // 模块被引用多少次后才会被分割
      maxAsyncRequests: 6, // 按需加载时最大的并行请求数
      maxInitialRequests: 4, // 入口点的最大并行请求数
      automaticNameDelimiter: '~', // 文件生成后的名称连接符
      cacheGroups: { // 配置缓存组，用于控制哪些模块要打包到一个 chunk 中
        defaultVendors: {
          test: /[\\/]node_modules[\\/]/, // 匹配从 node_modules 引入的模块
          priority: -10, // 优先级，值越大优先级越高
          reuseExistingChunk: true, // 如果当前 chunk 包含的模块已经被打包过，那么直接复用
        },
        default: {
          minChunks: 2, // 至少被两个模块引用时才会被提取为一个新的 chunk
          priority: -20,
          reuseExistingChunk: true,//复用已经存在的 chunk
        },
      },
    },
  },
};


```

# 代码拆分的管理

runtime chunk 是 Webpack 的一个核心部分，用于处理模块加载和执行。

只要有代码拆分的情况（无论是多入口点、动态导入还是显式配置 `runtimeChunk`）。Webpack 通常会生成一个 runtime chunk 来管理模块的加载和执行。runtime chunk 的存在确保了模块能够按照正确的顺序和方式加载，并且共享的依赖只会被加载一次。

**示例：显式配置runtimeChunk**

```js
module.exports = {
  entry: './src/index.js',
  output: {
    filename: '[name].[contenthash].js',
    path: path.resolve(__dirname, 'dist'),
  },
  optimization: {
    runtimeChunk: 'single', // 或 { name: 'runtime' }
  },
};

```

# 代码拆分的原理

* **独立解析依赖** ：
  每个入口点会独立解析其依赖项。Webpack 会构建一个依赖图，每个入口点都有自己独立的依赖图。
* **生成独立的包** ：
  根据依赖图，Webpack 会将每个入口点及其依赖打包成独立的包。这样可以确保每个入口点的代码和其依赖分离，避免不必要的代码加载。
* **避免重复依赖** ：
  为了避免多个入口点之间的重复依赖，Webpack 的优化插件会将公共依赖拆分成一个单独的包，这样多个入口点可以共享这些公共依赖。

# 总结

通过使用 Webpack 进行代码拆分，可以显著优化应用的加载性能，确保首页只加载必要的代码，其余代码按需加载。这不仅减少了初始加载时间，还提升了用户体验。在实际应用中，可以结合入口点拆分、动态导入和公共依赖拆分等多种技术，实现更高效的代码拆分和加载优化。
