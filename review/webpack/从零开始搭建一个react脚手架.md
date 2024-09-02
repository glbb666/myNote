# 初始化

[首先我们需要了解 `package.json`](./package.json详解.md)

`yarn init`可以快速初始化 `package.json` 文件。但是执行之后会有一个交互式的命令行让你输入需要的字段值，当然如果你想直接使用默认值，也可以使用 ` yarn init -y` 来超速初始化。

# 区分生产环境和开发环境

## 为什么需要区分生产环境和开发环境

因为开发是一个过程，生产是一个目的。在开发的过程中会产生错误，所以开发环境的配置需要方便开发人员的调试，而生产环境的目的是有效的运行。

我们需要一个公共的配置，然后基于这个公共的配置上。

生成生产环境的配置和开发环境的配置

- 创建一个公共打包的配置文件名为 `webpack.common.config.js`

```
WebpackReact
├── config
│   ├── webpack.common.config.js
```

```javascript
const path = require('path');

module.exports = {
  entry: {
    ...
  },
  output: {
    ...
  }
}
```

- 生成生产环境的配置和开发环境的配置

```javascript
yarn --save-dev webpack-merge
```

```javascript
WebpackReact-----------------------------------根目录
├── config
│   ├── webpack.common.config.js---------------公共的配置
│   ├── webpack.dev.config.js------------------生产环境的配置
│   └── webpack.prod.config.js-----------------开发环境的配置
```

在生产环境 `webpack.prod.config.js`就可以这样使用 `merge`

```javascript
const merge = require('webpack-merge');
const common = require('./webpack.common.config.js');

module.exports = merge(common, {
  mode: 'production',
});
```

然后修改 `package.json`文件，添加指令

```javascript
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
-   "start": "webpack --config ./config/webpack.common.config.js",
+   "build": "webpack --config ./config/webpack.prod.config.js"
  },
```

当我们进行配置前，我们可以看看代码中有没有做一些预先配置，这样会节约很多工作。

# 生产模式下默认开启

## plugin

### webpack.DefinePlugin

#### 是什么？

`webpack.DefinePlugin` 用于创建在编译时可以配置的全局常量。通过这个插件，你可以在代码中使用类似 `process.env.NODE_ENV` 这样的变量来区分不同的环境（如开发环境和生产环境）。

#### 为什么？

**减少运行时开销**

`process.env.NODE_ENV` 这样的环境变量通常是在运行时进行的，会影响性能。

`webpack.DefinePlugin` 会在编译时将源代码中的常量替换为你指定的值。比如：

```js
if (process.env.NODE_ENV === 'production') {
  // 生产环境特定代码
}

```

在使用 `webpack.DefinePlugin` 后， `process.env.NODE_ENV` 会被替换为 `'production'`，最终生成的代码类似于：

```js
if ('production' === 'production') {
  // 生产环境特定代码
}

```

这种替换方式可以帮助 Webpack 更好地进行代码优化，比如去除死代码（未被执行的代码）。

**避免环境差异**

开发初，我们会在 `package.json` 中，通过 `scripts` 可以设置 `NODE_ENV`：

```javascript
"scripts": {
    "start": "NODE_ENV=development webpack-dev-server",
    "build": "NODE_ENV=production webpack"
}
```

但是 `NODE_ENV`只在当前的 Node.js 进程中有效，并不会自动传递到浏览器环境。因此在浏览器中执行的代码不会识别 `process.env.NODE_ENV`。直接使用 `process.env.NODE_ENV` 可能会导致报错。

通过 `webpack.DefinePlugin`，你可以在编译时模拟这个变量的存在，避免环境差异带来的问题。

**代码中的安全性**

通过 `webpack.DefinePlugin`，你可以将一些敏感信息（如 API 密钥）通过环境变量的方式注入到代码中，而不需要直接在源码中硬编码这些信息，降低安全风险。

#### 怎么做？

```js
const webpack = require('webpack');

module.exports = {
  plugins: [
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV || 'development')
    })
  ]
};

```

## `optimization`

### **`removeAvailableModules`**

生产模式默认启用，移除父级 chunk 中已经存在的模块，减少冗余代码和提升加载性能。

### **`removeEmptyChunks`**

生产模式默认启用，移除打包过程中生成的空 chunk，避免生成无用的文件，进一步优化输出。

以下配置全部开启，就是TreeShaking的步骤了。

### concatenateModules

#### **是什么** ：

`concatenateModules` 用于在生产模式下默认启用 Scope Hoisting。

#### **为什么** ：

在 Webpack 打包过程中，每个模块通常会被包裹在一个单独的函数中，以实现模块隔离。这种做法虽然确保了模块的独立性，但会引入额外的闭包开销。

通过合并模块减少闭包的数量，降低代码体积，提升性能。

#### **怎么做** ：

在 Webpack 4 及以上版本中，`concatenateModules` 在生产模式下默认启用。

### Tree Shaking（总流程）

### **是什么**

Tree Shaking 用于在打包过程中移除那些代码中未被使用的部分。

它通常针对 ES6 模块（`import` 和 `export`）进行工作，通过静态分析确定哪些模块或模块中的部分代码没有被引用，从而在最终打包的文件中剔除它们。

### **怎么做**

在 Webpack 4 及以上版本中，生产模式 (`mode: 'production'`) 会默认启用 Tree Shaking。

它包含以下优化。

### **`usedExports`** ：

在生产模式下，Webpack 会自动启用 `usedExports`，通过静态分析来决定哪些代码是未被使用的。对于没有被使用的导出，Tree Shaking 会将其标记为“未使用”，并在打包时移除。

### **`sideEffects`**：

##### 是什么

`sideEffects` 是一种用于告诉 Webpack 某个模块或者某些文件是否具有副作用的配置。你可以在 `package.json` 中的 `sideEffects` 字段中进行设置。

##### 为什么：

Tree Shaking 默认假设模块是无副作用的，可以安全地移除未使用的导入和导出。而如果某个模块的文件包含副作用，直接移除可能会导致程序行为异常。因此，正确配置 `sideEffects` 可以确保 Webpack 在移除未使用代码时不会误删有副作用的代码。

##### **怎么做** ：

* **检查 `sideEffects` 配置** ：Webpack 会先检查你的 `package.json` 中是否配置了 `sideEffects`，如果配置为 `false`，Webpack 会认为整个模块没有副作用，所有未被引用的导入和导出可以安全地移除。
* **未配置 `sideEffects`** ：
  * 如果模块在执行时可能产生副作用（例如修改全局状态、产生副作用的函数调用），Webpack 会保留这些代码，因为无法确定移除这些代码是否会影响应用的执行。
  * 如果模块不会产生副作用，会被移除

### **`minimize`** ：

生产环境默认开启，用于用Terser压缩和优化输出代码。

Terser更详细的配置见下面的生产环境配置 `optimization.minimizer`。

# 开发环境的默认开启


**持久缓存**

* Webpack 5 引入了持久缓存机制，将缓存存储在文件系统中，使得重新构建时可以复用缓存，大幅减少构建时间，尤其是在开发环境下。



## **Webpack 5 优化**

1.**持久缓存（Persistent Caching）**

* Webpack 5 引入了持久缓存机制，将缓存存储在文件系统中，使得重新构建时可以复用缓存，大幅减少构建时间，尤其是在开发环境下。

3.**模块联邦（Module Federation）**

* Webpack 5 正式发布了 `Module Federation`，允许多个独立的应用共享代码，这种微前端架构的实现方式极大地优化了代码重用和开发效率。

6.**更好的 WebAssembly 支持**

* Webpack 5 改进了对 WebAssembly 模块的支持，允许更高效地与 JavaScript 代码进行交互，并且提供了更好的优化选项。

8.**改进的模块解析**

* Webpack 5 对模块解析机制进行了优化，使得解析速度更快，并且在大型项目中更加高效。
* 支持基于 URL 的模块导入，以及更好的 ESM（ES Modules）支持。

10.**改进的开发体验**

* Webpack 5 增强了开发环境下的错误提示和诊断能力，使得调试过程更为顺畅。

# 公共配置

公共配置指的是在生产环境和开发环境都会使用的配置，它会被提取到公共配置中。

## reslove

reslove在webpack基础/resolve中

## Loader

### babel-loader：js，ts，react框架处理

在 Webpack 5 的公共配置中，通常只需要使用 Babel 来转换 JavaScript 和 TypeScript 文件，React框架

根目录新建一个配置文件 `.babelrc` 进行更详细的配置

```javascript
{
  "presets": [
    [
      "@babel/preset-env", // 这个 preset 会根据目标环境自动选择需要的 Babel 插件来转译现代 JavaScript 代码
      {
        "targets": {
          "edge": 17,
          "firefox": 60,
          "chrome": 67,
          "safari": 11.1
          // 目标浏览器版本，指示 Babel 仅转译不被这些浏览器支持的 ES6+ 语法
        },
        "useBuiltIns": "usage", 
        // 自动引入代码中用到的 polyfill，根据代码中实际使用的 ES6+ 语法选择所需的 polyfill
        "corejs": 3 
        // 指定 core-js 版本，避免不同 core-js 版本之间的不兼容问题
      }
    ],
    "@babel/preset-react" 
    // 用于转译 React JSX 语法
  ],
  "plugins": [
    [
      "@babel/plugin-proposal-decorators", 
      { "legacy": true }
      // 支持 ES7 的装饰器语法，使用 "legacy" 模式进行转译
    ],
    [
      "@babel/plugin-transform-runtime", 
      {
        "corejs": 3, 
        // 使用 core-js 3 版本提供的 polyfill，以避免全局污染
        "helpers": true, 
        // 启用后，Babel 会在需要时插入辅助代码，以减少重复代码
        "regenerator": true, 
        // 启用对异步函数的支持，允许使用 generator 和 async/await 语法
      }
    ]
  ]
}

```

再修改 `webpack.common.config.js` ，添加如下代码：

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(js|jsx|ts|tsx)$/,  // 匹配 .js, .jsx, .ts 和 .tsx 文件
        exclude: /node_modules/,    // 排除 node_modules 目录
        use: {
          loader: 'babel-loader',   // 使用 babel-loader
          options: {
            presets: ['@babel/preset-env', '@babel/preset-react', '@babel/preset-typescript'], // 使用的预设
          },
        },
      },
    ],
  }
};

```

### css-loader：css代码的解析

#### 是什么

* **解析和转换 CSS** ：
  `css-loader` 读取 `css`文件的内容并解析为 CSS AST（抽象语法树），然后将其转换为一个 JavaScript 模块。这个模块包含了一个包含所有解析后的 CSS 字符串的对象。
* **导出 CSS 模块** ：
  生成的 JavaScript 模块会导出一个包含 CSS 内容的字符串。例如，转换后的模块可能如下所示（简化版）：
  ```js
  // 转换后的 JavaScript 模块（简化版）
  module.exports = `
    body {
      background-color: lightblue;
    }
    h1 {
      color: darkblue;
    }
  `;

  ```

#### 怎么做

```js
// common.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          'css-loader' // 只负责解析 CSS 文件
        ]
      }
    ]
  }
};
```

### less-loader：less

#### 是什么

`less-loader` 是 Webpack 的一个加载器（Loader），用于将 `.less` 文件编译成 `.css` 文件。一般把它放在 `css-loader`之前使用

#### 怎么做

```js
// 安装相关依赖
// npm install less less-loader css-loader style-loader --save-dev

module.exports = {
  module: {
    rules: [
      {
        test: /\.less$/, // 匹配所有 .less 文件
        //执行顺序从后到前
        use: [
          'css-loader', // 解析 CSS 文件中的 @import 和 url() 等外部资源
          'less-loader' // 将 LESS 编译为 CSS
        ]
      }
    ]
  }
};

```

## Plugin

#### `HtmlWebpackPlugin`

作用：根据模版生成 `html`文件，并自动引用打包后的 `js`，`css`文件。这对于在文件名中包含每次会随着编译而发生变化哈希的 `webpack bundle `尤其有用。

```javascript
var HTMLWebpackPlugin = require('html-webpack-plugin')
var baseConfig = {
    plugins: [
        new HTMLWebpackPlugin({//创建一个在内存中生成html页面的插件
            template:path.join(__dirname,'../src/index.html'),//指定模板页面，将来会根据指定的页面路径，去生成内存中的页面
            filename:'index.html'//指定生成的页面的名称
        })
    ]
}
```

#### optimization

### `splitchunk`代码分割

#### 是什么

`splitChunks` 是 Webpack 提供的一个配置项，用于将代码拆分为多个不同的代码块，以便于实现代码的按需加载和缓存优化。

#### 为什么

通过分离第三方库、共享模块、和业务代码，`splitChunks` 可以减少重复代码的加载，提高页面的加载速度。

#### 怎么做

见webpack拆分代码/`splitChunks`进行公共依赖拆分。

### `runtimeChunk`

#### **是什么**

`runtimeChunk` 用于处理 运行时代码。

运行时代码是 Webpack 用来管理模块化系统的代码，主要负责模块加载，缓存，依赖关系的解析，确保在浏览器中按需加载和执行代码。

`runtimeChunk` 拆分出一个独立的文件，包含所有 Webpack 生成的运行时代码，而不与业务代码混在一起。

#### **为什么**

**减少不必要的重复打包** ：将运行时代码分离到单独的文件中，避免了每次业务代码变动时，运行时代码也需要重新下载。这样可以更好地利用浏览器缓存，减少不必要的重复打包，从而提高性能。

**便于调试与维护** ：将运行时代码独立出来后，生成的代码结构更清晰，方便开发者进行调试和维护。

#### **怎么做**

见webpack拆分代码/代码拆分的管理。

### moduleIds & chunkIds

#### 是什么

`moduleIds` 是 Webpack 中用于标识模块的唯一标识符。每个模块在打包时都会分配一个 `moduleId`，用于在生成的打包文件中引用和加载该模块。默认情况下，Webpack 会为每个模块分配一个递增的数字 ID（整数 ID），但你可以通过配置来改变这一行为。

`chunkIds` 是 Webpack 中用于标识代码块（chunk）的唯一标识符。一个 chunk 通常包含一个或多个模块，并且可以在应用程序加载时或按需加载时使用。

#### **为什么**

* **优化打包文件的缓存利用率** ：默认情况下，Webpack 使用数字作为 `moduleIds` 和 `chunkIds`，但每次构建时这些 ID 可能会变化，导致浏览器缓存失效，即使代码本身没有发生变化。通过定制 `moduleIds` 和 `chunkIds`，可以更稳定地生成文件，避免不必要的缓存失效，从而提升性能。
* **代码分割与动态导入** ：当应用程序使用代码分割或动态导入时，`chunkIds` 用于标识这些按需加载的代码块。正确的 ID 分配有助于优化代码加载顺序和资源利用。
* **调试与可读性** ：在开发环境中，合理配置 `moduleIds` 和 `chunkIds` 可以提高打包文件的可读性和可维护性。例如，使用基于路径的 `moduleIds` 和 `chunkIds` 代替默认的数字 ID，方便开发者调试。

#### **怎么做**

Webpack 提供了多种方式来配置 `moduleIds` 和 `chunkIds`，以便优化和定制构建输出。

#### **配置 `moduleIds`**

在 Webpack 的 `optimization` 配置中，你可以通过 `optimization.moduleIds` 选项来控制 `moduleIds` 的生成方式：

```js
module.exports = {
  // 其他配置
  optimization: {
    moduleIds: 'deterministic', // 'natural', 'named', 'hashed'
  },
};

```

* **`natural`** ：使用模块的默认顺序（即数字递增）作为 `moduleIds`，不稳定，适用于小型项目或开发环境。
* **`named`** ：使用模块的相对路径作为 `moduleIds`，可读性高，适用于开发环境，但会增加打包体积。
* **`deterministic`** ：基于模块的相对路径以及模块的顺序来生成较短的 `moduleIds`。它会确保在代码未发生变化的情况下，生成的 ID 尽可能一致，但会在保证一致性和生成较短 ID 之间做一个平衡。适用于生产环境。
* **`hashed`** ：生成基于内容的哈希值 ID。尽管 ID 较长，但它几乎保证了在模块内容不变的情况下，ID 始终不变。适合开发环境，那些对缓存稳定性要求极高的大型项目。

#### **配置 `chunkIds`**

类似于 `moduleIds`，你可以通过 `optimization.chunkIds` 选项来控制 `chunkIds` 的生成方式：

```js
module.exports = {
  // 其他配置
  optimization: {
    chunkIds: 'deterministic', // 'natural', 'named', 'size', 'total-size'
  },
};

```

* **`natural`** ：使用默认的递增顺序分配 `chunkIds`，简单但不稳定。
* **`named`** ：使用 chunk 名称作为 `chunkIds`，适合开发环境，增强可读性。
* **`deterministic`** ：Webpack 在构建每个 chunk 时，会分析这个 chunk 内的所有模块，并基于这些模块的相对路径和导入顺序生成一个确定性的 ID。这意味着 chunk 的 ID 是由其内部所有模块的组合内容决定的，而不是单一模块。如果两个 chunk 包含相同的模块集合且这些模块的导入顺序一致，那么它们的 `chunkIds` 会相同。
* **`size`** ：Webpack 会根据每个 chunk 的文件大小进行排序。较小的 chunk 会获得较小的 ID。适用于需要按大小优化 chunk 加载顺序的场景。例如，你希望小的 chunk（通常是关键资源）优先加载，而较大的 chunk 可以稍后加载。这有助于优化初次加载性能和用户体验。
* **`total-size`** ：Webpack 会计算整个 chunk 的最终输出大小，并根据大小来生成 ID。**实际输出大小** 包括了代码压缩、模块合并、依赖管理、运行时代码等所有经过 Webpack 处理后的最终文件大小。这个选项更为全面地考虑了 chunk 的实际加载成本。

#### QA

##### 为什么 `chunkIds` 没有 `hashed` 选项

在实际项目中，`chunkIds` 通常不会像 `moduleIds` 那样频繁变动。`chunkIds` 更关注的是块的整体特性而不是单个模块的内容。

## Loader

### style-loader

##### **是什么**

`style-loader` 用于将经过处理的 CSS 文件以 `<style>` 标签的形式动态注入到 HTML 文档的 `<head>` 部分中。它主要用于开发环境，便于实时加载和更新样式。

##### **为什么**

在开发环境中，用io读取文件。把样式和放在html文件中可以少读一个文件，更快进行处理。在使用 Webpack 的热更新功能时，可以使样式的更改在不刷新页面的情况下即时生效。这对于提高开发效率非常有用。

##### **怎么做**

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          'style-loader', // 开发环境配置，将 CSS 通过 <style> 标签注入页面
        ]
      }
    ]
  }
};

```

## webpack-dev-server

### 是什么

`webpack-dev-server` 是一个用于开发环境的 HTTP 服务器，它封装开发环境所需的功能，允许开发者在本地快速开发，而不需要额外配置复杂的服务器环境。主要包含以下功能：

**实时刷新** ：

`webpack-dev-server` 会监视项目文件的变化，一旦检测到文件变动，会自动重新编译代码并刷新浏览器。这种实时刷新功能可以加快开发迭代速度。

**热模块替换** ：

`webpack-dev-server` 支持 HMR，这是 Webpack 的一项重要功能。HMR 允许在不刷新整个页面的情况下，替换、添加或删除模块。这使得开发者可以保留应用的状态（如表单输入、游戏进度等）而仅更新修改的模块。

**跨域和代理** ：

`webpack-dev-server` 支持将特定路径的请求代理到其他服务器。这对于处理 API 请求特别有用，开发时可以将前端代码与实际的后端 API 服务器分离开来。

**统一托管资源** ：

`webpack-dev-server` 可以统一托管所有资源，包括 JavaScript、CSS、图片等，这有助于模拟真实的生产环境，测试资源加载、路径解析等问题。直接从本地文件系统加载静态资源可能无法正确模拟这些场景，特别是在你使用了 `publicPath` 等配置项时。

开发者可以通过配置 `contentBase` 来指定静态文件的目录。

**减少 I/O 开销：**
`webpack-dev-server` 使用的是内存中的虚拟文件系统，这意味着编译后的文件不会写入物理磁盘，而是直接存储在内存中。这种方式极大地提高了读取和写入的速度，减少了 I/O 操作的开销，从而加快了构建速度。

### 与hotModulePlugin的区别

* **前置工具** ：`webpack-dev-server` 可以看作是 Webpack 的开发辅助工具，它提供了 HMR 的便捷实现方式。

### 怎么做

本地开启服务，实时更新。首先要下载 `webpack-dev-server`

```javascript
yarn add webpack-dev-server --save-dev
```

接着在 `package.json`中配置

```javascript
 "scripts": {
    "start": "webpack-dev-server --config ./config/webpack.dev.config.js",
  }
```

在 `webpack.dev.config.js`中进行配置，由于是热更新，所以输出文件的名字中应该包含 `hash`而非 `contenthash`。

```javascript
const webpack = require('webpack');
module.exports = merge(common,{
  devServer:{
    contentBase: path.resolve(__dirname, '../dist'),
    open: true,//浏览器自动打开页面
    inline: true,//每次修改源文件后保存会自动刷新页面
    port: 9000,
    compress: true,//配置是否启用 gzip 压缩
    hot: true//启用热更新
  }
})
```

**注意：你启动 `webpack-dev-server`后，你在目标文件夹中是看不到编译后的文件的,实时编译后的文件都保存到了内存当中。因此很多同学使用 `webpack-dev-server`进行开发的时候都看不到编译后的文件**

## `sourceMap`方便调试

* **`eval-source-map`** : 适合开发环境，构建速度快，带有完整的源代码映射，但可能会增加编译时间。
* **`cheap-module-source-map`** : 在开发中提供较快的构建速度，同时提供基本的源代码映射，忽略列信息。
* **`inline-source-map`** : 将 `sourceMap` 作为 Data URL 内联到生成的 JavaScript 文件中。

```javascript
module.exports = merge(common,{
  ...
  devtool: 'inline-source-map',
  ...
})
```

#### 代码规范的检查

- `proxyTable`接口代理
- `eslint style-lint`代码规范检查

# 生产环境的需求

## Plugin

### `MiniCssExtractPlugin`：css提取

#### 是什么

`MiniCssExtractPlugin` 是 Webpack 的一个插件，用于将 CSS 从 JavaScript 中分离出来，并生成单独的 CSS 文件。

#### 为什么

由于 `html2.0`有多路传输的特性，单独的 CSS 文件方便浏览器并行加载资源。并且更好地缓存 CSS 文件。

#### 怎么做

```javascript
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = {
    plugins: [
        new MiniCssExtractPlugin({
            filename: '[name].[contenthash].css',
        }),
    ],
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [MiniCssExtractPlugin.loader],
            },
        ],
    },
};


```

## optimization

### `minimizer`

#### **是什么**

`minimizer` 是 Webpack 用于压缩和优化最终输出文件的配置项。它允许你指定用于优化 JavaScript、CSS 等资源的插件，通常用于在生产环境中减小文件大小和提高加载速度。

#### **为什么**

在生产环境中，减小文件大小可以显著提高页面加载速度，减少带宽消耗，提升用户体验。

默认情况下，Webpack 会使用 TerserPlugin 来压缩 JavaScript 文件，但有时你可能需要使用其他压缩工具或者对 Terser 进行自定义配置，这时 `minimizer` 选项就非常重要。

#### **怎么做**

要使用 `minimizer` 配置，你可以在 Webpack 的配置文件中设置 `optimization.minimizer`。以下是如何配置的步骤：

默认情况下 ，Webpack 在生产模式下会自动使用 TerserPlugin 来压缩 JavaScript。你不需要手动配置。

自定义压缩配置如下，这种配置可以让你同时压缩 JavaScript 和 CSS 文件，并且你可以根据项目需求微调每个插件的选项。

```js
const TerserPlugin = require('terser-webpack-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');

module.exports = {
  mode: 'production',
  optimization: {
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          compress: {
            drop_console: true, // 移除 console.log 语句
          },
        },
      }),
      new CssMinimizerPlugin(), // 用于压缩 CSS 文件
    ],
  },
};

```

#### Webpack优化

#### 缩短 `webpack`的打包时间

##### 利用缓存缩短构建时间

```javascript
yarn add cache-loader
```

在一些性能开销较大的 `loader` 之前添加此 `loader`，以将结果缓存到磁盘里。

```javascript
{
        test:/\.(js|jsx)$/,
        use:['cache-loader','babel-loader'],
        // exclude: /node_modules/,//不需要去转译"node\_modules"这里面的文件。
},
```

##### 利用 `thread-loader`进行多进程压缩

把 `thread-loader`放在 `loader`的前面，这样 `thread-loader`之后的 `loader`就会在单独的一个 `worker`池中运行。

##### 使用 `dllPlugin`

这样一些不经常改变的第三方库就可以提前打包好，下次打包的时候的速度就会变快。

DllPlugin 可以将特定的类库提前打包然后引入。这种方式可以极大的减少打包类库的次数，只有当类库更新版本才有需要重新打包，并且也实现了将公共代码抽离成单独文件的优化方案。

```
// 单独配置在一个文件中
// webpack.dll.conf.js
const path = require('path')
const webpack = require('webpack')
module.exports = {
  entry: {
    // 想统一打包的类库
    vendor: ['react']
  },
  output: {
    path: path.join(__dirname, 'dist'),
    filename: '[name].dll.js',
    library: '[name]-[hash]'
  },
  plugins: [
    new webpack.DllPlugin({
      // name 必须和 output.library 一致
      name: '[name]-[hash]',
      // 该属性需要与 DllReferencePlugin 中一致
      context: __dirname,
      path: path.join(__dirname, 'dist', '[name]-manifest.json')
    })
  ]
}
```

然后我们需要执行这个配置文件生成依赖文件，接下来我们需要使用 DllReferencePlugin 将依赖文件引入项目中

```
// webpack.conf.js
module.exports = {
  // ...省略其他配置
  plugins: [
    new webpack.DllReferencePlugin({
      context: __dirname,
      // manifest 就是之前打包出来的 json 文件
      manifest: require('./dist/vendor-manifest.json'),
    })
  ]
}
```

#### 提高用户体验

- 按需加载：如果我们将页面全部打包进一个 `JS` 文件的话，虽然将多个请求合并了，但是同样也加载了很多并不需要的代码，耗费了更长的时间。那么为了首页能更快地呈现给用户，我们肯定是希望首页能加载的文件体积越小越好，这时候我们就可以使用按需加载，将每个路由页面单独打包为一个文件。
- `splitchunk`，把公共模块提取出来。如果不把代码块进行分割，最后 `webpack`就会把所有的代码都打到一个包里面去，这样的包很笨重，也不利于下载。`http2.0`正好支持多路复用，最多支持6个并行的请求，这样的话就可以提高下载的时间，优化用户的体验。

### 参考文章

[Webpack从零配置React的运行环境上篇——搭建基本环境](https://segmentfault.com/a/1190000021464216)

[前端面试之道之webpack篇](https://segmentfault.com/a/1190000011383224)

[从零开始搭建一个webpack脚手架](https://segmentfault.com/a/1190000016661311?utm_source=sf-related)
