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

# 配置前置：调研webpack4和webpack5和之前的版本比分别做了什么优化

当我们进行配置前，我们可以看看代码中有没有做一些预先配置，这样会节约很多工作。

## **Webpack 4 优化**

3.**生产模式优化（TODO: 让gpt给出例子）**

* Tree Shaking（删除未使用的代码）
* Scope Hoisting（作用域提升）
* 默认启用 UglifyJS 插件进行代码压缩。不止可以压缩 `JS`代码，还可以压缩 ` HTML`、`CSS` 代码，并且在压缩 ` JS` 代码的过程中，我们还可以通过配置实现比如删除 ` console.log` 这类代码的功能。
* 自动处理 `process.env.NODE_ENV` 变量

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

9.**扩展性增强**

* 引入了多种新的插件钩子和 API，使得 Webpack 的扩展性和自定义能力更强。

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

#### webpack.DefinePlugin

##### 是什么？

`webpack.DefinePlugin` 用于创建在编译时可以配置的全局常量。通过这个插件，你可以在代码中使用类似 `process.env.NODE_ENV` 这样的变量来区分不同的环境（如开发环境和生产环境）。

##### 为什么？

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

##### 怎么做？

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

## optimization

### `splitchunk`代码分割

#### 是什么

`splitChunks` 是 Webpack 提供的一个配置项，用于将代码拆分为多个不同的代码块，以便于实现代码的按需加载和缓存优化。

#### 为什么

通过分离第三方库、共享模块、和业务代码，`splitChunks` 可以减少重复代码的加载，提高页面的加载速度。

#### 怎么做

见webpack拆分代码/`splitChunks`进行公共依赖拆分。

* **`runtimeChunk`** ：用于将运行时代码提取到一个单独的 chunk 中，避免每次构建时的重复代码。
* 适合在公共配置中使用，以提升缓存利用率。
* **`moduleIds` 和 `chunkIds`** ：
* `deterministic`: 避免模块 ID 和 chunk ID 的频繁变化，提升缓存命中率，通常配置在公共环境中。

# 开发环境的需求

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

## optimization

1. **`removeAvailableModules` 和 `removeEmptyChunks`** ：

* 用于清理空模块和空 chunk，通常在开发环境中启用，以减少不必要的构建时间。

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

**`minimize`** ：默认值false，在生产环境中通常设置为 `true`，用于压缩和优化输出代码。

```

```

**`minimizer`** ：

配置代码压缩工具，比如 `TerserPlugin`（用于 JS 代码压缩）和 `CssMinimizerPlugin`（用于 CSS 代码压缩）。

* **`concatenateModules`** ：
* 启用模块的合并优化（Scope Hoisting），减少函数包装和代码体积，通常在生产环境中启用。
* **`sideEffects`** ：
* 开启 tree-shaking，以删除没有副作用的代码。生产环境常用。
* **`usedExports`** ：
* 标记使用到的导出内容以进行 tree-shaking，通常在生产环境中启用。

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
