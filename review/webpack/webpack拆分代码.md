# 什么是代码拆分

代码拆分的目的是将代码分成多个小块，这些小块可以按需加载，从而减少初始加载时间和提升页面性能。

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

## 2. 动态导入

使用动态导入语法 `import()` 可以实现按需加载。

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

Webpack 会自动将 `module.js` 拆分成一个独立的包，并在需要时加载。

## 3. 公共依赖拆分

### 为什么要拆分公共依赖？

当没有显式配置 `splitChunks` 时，所有依赖会被打包到入口文件中。

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

公共依赖一般有两部分，本地公共依赖和第三方公共依赖，第三方公共依赖常被放入vendor文件中。

**`vendor` 文件**

* 通常包含第三方库或插件，例如 `lodash`、`react`、`react-dom` 等。
* 这些库通常不会频繁更新，可以更好地利用缓存。

**本地公共文件**

* 通常包含项目内部的共享模块或组件，例如在多个入口点中复用的业务逻辑或 UI 组件。
* 这些模块可能会更频繁地更新，因此需要更细粒度的控制和优化。

一般来说有以下配置方式

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
  },
  optimization: {
    splitChunks: {
      chunks: 'all',
    },
  },
};

```

#### `splitChunks.cacheGroups` 配置

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
          name: 'vendors',
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

#### 配置优先级确定

 `splitChunks` 用 `priority`来控制不同 cache group 的优先级。优先级高的先拆分。

一般是按以下策略来给优先级：

**`vendors` cache group** （高优先级）

* Webpack 会首先将 `lodash` 和其他第三方库提取到 `vendors` chunk 中。这个 chunk 是相对稳定的，不会因为本地模块的变动而改变。
* 提取到 `vendors` chunk 中有助于缓存利用，减少重复加载。

**`default` cache group** （低优先级）

* 接着，Webpack 会处理本地共享模块（如应用中的业务逻辑模块）。
* 如果这些本地模块已经被拆分到 `vendors` chunk 中（因为它们复用了第三方库），`default` cache group 会尽量复用已经存在的 chunk，而不是重复打包。

#### 两种配置方式的优缺点

**显式数组配置：**

* **优点** ：简单明了，适合项目依赖较少的场景。
* **缺点** ：需要手动列出所有第三方库，不适合依赖较多或频繁变动的项目。

**`splitChunks.cacheGroups` 配置**

* **优点** ：自动化程度高，不需要手动列出第三方库，适合依赖较多的项目。
* **缺点** ：配置稍微复杂，需要理解 `splitChunks` 的工作原理和各个选项的作用。

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
