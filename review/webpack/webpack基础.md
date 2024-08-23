[前端面试之webpack篇](https://segmentfault.com/a/1190000011383224)

# 概念介绍

## Webpack是什么

`webpack`是一个**模块打包机**。它从一个给定的的主文件开始找到项目的所有依赖文件，使用 `loader`进行处理，最后打包成浏览器可识别的 `js`文件。

在使用 `webpack`命令的时候，他将接受 `webpack`的配置文件，除非我们使用其他的操作。

## 模块是什么

**模块** 是 Webpack 中的基本构建单元。每个文件都被视为一个模块。可以通过 `import` 或 `require` 语句引用其他模块。

## chunk是什么

**Chunk** 是 Webpack 构建过程中生成的代码块。简单来说，Chunk 是模块的一个集合，它是打包输出的基本单位。在 Webpack 中，多个模块会被合并成一个或多个 Chunk，以便最终输出为一个或多个文件（通常是 JavaScript 文件）。

具体可以见webpack/webpack拆分代码。

# webpack运行流程

Webpack的核心运行在 Node.js 环境中。并且可以通过 Node.js 提供的 API 来处理文件系统、路径等。

以下是 Webpack 在 Node.js 环境中运行的基本过程：

## 1. **启动 Webpack**

当在命令行中运行 `webpack` 命令时，Node.js 会加载并执行 Webpack 的 CLI 脚本。这个脚本用来解析配置文件（如 `webpack.config.js`）。

## 2. **读取 Webpack 配置**

Webpack 的配置文件通常是一个 CommonJS 模块（通过 `module.exports` 导出配置对象），它定义了入口文件、输出选项、模块解析规则、插件等。Node.js 会使用 `require()` 加载这个配置文件，并将配置对象传递给 Webpack。

## 3. **创建 Compiler 实例**

Webpack 使用配置对象创建一个 `Compiler` 实例，这是整个构建过程的核心。`Compiler` 负责从入口文件开始，递归解析模块之间的依赖关系，应用各种加载器（Loader）和插件（Plugin），最终生成打包后的文件。

## 4. **入口解析 & 代码拆分**

Webpack 从指定的 `entry` 文件开始，使用 Node.js 的 `fs` 模块读取文件内容，递归解析所有的依赖模块。

每个入口点会作为一个构建的起点，Weback 会从这些入口点出发，构建依赖树。

同时Webpack 的代码分割功能（如 `SplitChunksPlugin`）可以将模块分离到不同的 chunks 中，以便更高效地加载和缓存。这也意味着公共库可以被提取到独立的 chunks 中

## 5. 使用 Loader**模块转换**

对于每个模块，Webpack 会根据配置中的 `rules` 查找合适的 loader，并应用这些 loader 进行转换。例如，Babel 可以用于转换 JavaScript 代码，CSS loader 可以处理 CSS 文件等。

## 6. **应用插件**

* 在整个流程中，Webpack 会在恰当的时机执行插件的逻辑。Webpack 插件可以在特定的事件（如编译开始、模块解析、文件输出等）触发时执行自定义的逻辑。

## 7. **输出和插件执行**

 Webpack 会将所有生成的 chunks 转换为最终的文件，并根据 `output` 配置中的 `filename` 和 `path` 选项将这些文件写入到指定的输出目录。

## 8. **生成 Source Map（可选）**

如果配置了 Source Map，Webpack 会生成映射文件，用于将打包后的代码映射回源代码。这个过程通常也依赖于 Node.js 提供的文件系统操作。

# entry

## 是什么

`entry` 是 Webpack 配置中的一个关键属性，用来指定构建的入口文件。Webpack 会从这个入口文件开始，创建一个依赖图，从而把应用程序所需的模块打包成一个或多个最终输出的文件。

## **为什么需要配置 `entry`**

配置 `entry` 是为了告诉 Webpack 应用程序的入口点在哪里，这样 Webpack 就可以从这个起点出发，递归解析出所有的依赖关系。对于单页面应用，通常只有一个入口文件，而对于多页面应用或复杂项目，可能需要配置多个入口文件。

## 怎么做 (如何配置 `entry`)

#### **1. 单入口配置**

如果你的项目是一个简单的单页面应用，可以直接指定一个入口文件：

```
var baseConfig = {
    entry: './src/index.js'  // 单一入口
}

```

在这种情况下，Webpack 会从 `./src/index.js` 开始打包所有的依赖。

#### **2. 多入口配置**

对于需要多个入口文件的情况，可以将 `entry` 配置成一个对象：

```js
var baseConfig = {
    entry: {
        main: './src/index.js',  // 主入口文件
        app: './src/index1.js'   // 另一个入口文件
    }
}

```

当有多个入口时，webpack会创建多张依赖图，这些依赖图是彼此完全分离、互相独立的

在这个配置中，Webpack 会为每个入口文件生成独立的依赖图，并输出对应的文件。每个 `entry` 的 `key` 会成为输出文件的 `[name]`。

#### **3. 第三方库的单独打包**

为了优化缓存，可以将第三方库（如 React、Lodash 等）单独打包，这样在代码变更时，公共库可以继续从缓存中读取，不需要重新加载：

```js
const config = {
    entry: {
        app: './src/app.js',    // 应用的入口文件
        vendors: ["react", "react-redux", "redux"] // 第三方库
    }
};

```

通过这种方式，`vendors` 的内容在不变的情况下会保持相同的哈希值，从而优化加载速度。

# output

## **是什么** :

* `output` 是 Webpack 配置中的一个关键属性，用于指定打包后的文件应该输出到哪里以及如何命名。
* 它决定了 Webpack 最终生成的文件的路径和文件名模式。

## **为什么需要 `output`** :

* `output` 定义了打包文件的存放位置，这对于构建后的部署和发布至关重要。
* 通过 `output` 配置，你可以控制文件名的格式，路径结构，以及如何处理诸如图片、字体等静态资源。
* 配置正确的 `output` 可以帮助你优化缓存，避免文件名冲突，
* 并且可以通过配置根路径，与服务器部署环境更好地兼容。

## **怎么做 (`output` 配置方式)** :

#### 1. **基本配置**

```js
module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',  // 指定输出文件的名称
    path: path.resolve(__dirname, 'dist')  // 指定输出文件的路径（绝对路径）
  }
};

```

* `filename`: 定义输出文件的名称。可以使用模板字符串，比如 `[name].js`（基于 `entry` 的 `key` 命名），以及使用 `hash` 来确保文件的唯一性。详情见hash&chunkhash&contenthash的差别
* `path`: 定义文件输出的目录。注意，必须是绝对路径，所以通常需要使用 Node.js 的 `path.resolve` 来设置。

**output定义入口文件的输出格式，**即使入口文件有多个，但是**只有一个输出配置**

- `filename` 用于输出文件的文件名。
- `path`目标输出目录的绝对路径。
- `[name]`：模块的名称。

#### 2. **多入口配置**

```js
module.exports = {
  entry: {
    app: './src/app.js',
    admin: './src/admin.js'
  },
  output: {
    filename: '[name].[chunkhash].js',  // 使用chunkhash进行文件名控制
    path: path.resolve(__dirname, 'dist')
  }
};

```

多入口配置允许为每个入口文件生成独立的输出文件。`[name]` 会自动替换为 `entry` 中的 `key`，如 `app` 和 `admin`。

这种配置可以确保每个入口文件打包后的输出文件具有独立的名称，如 `app.chunkhash.js` 和 `admin.chunkhash.js`。

#### 3. **CDN 配置**

```js
module.exports = {
  output: {
    filename: '[name].[contenthash].js',
    path: path.resolve(__dirname, 'dist'),
    publicPath: 'https://cdn.example.com/assets/'  // 指定静态资源的公共路径，如CDN路径
  }
};

```

是什么：

* `publicPath`: 定义了所有输出文件的公共 URL 路径。用于在运行时动态加载资源。

为什么：

* 当你的静态资源托管在 CDN 上时，配置 `publicPath` 可以确保所有资源从正确的 CDN 路径加载，提高加载速度和可用性。

怎么做：

* 根据部署环境配置 `publicPath`，在生产环境中指定 CDN 路径，确保资源加载的高效性。

#### 4. **分离静态资源：优化图片、字体等资源的输出**

```js
module.exports = {
  output: {
    filename: '[name].[contenthash].js',
    path: path.resolve(__dirname, 'dist'),
    assetModuleFilename: 'images/[hash][ext][query]'  // 自定义静态资源（如图片）的输出路径和名称
  }
};

```

##### **是什么？**

* **`assetModuleFilename`** : 用于自定义资源模块（如图片、字体等）的文件名和路径，Webpack 5 引入的 Asset Modules 特性可以取代 `file-loader` 和 `url-loader`。

##### **为什么？**

通过 `assetModuleFilename` 可以统一管理和优化静态资源的输出路径，减少重复配置，并利用 `hash` 机制优化缓存。

##### **怎么做？**

* 使用 `assetModuleFilename` 取代传统的加载器，配置静态资源的输出路径和名称，结合 `hash` 优化缓存。

`assetModuleFilename`: 针对资源模块（如图片、字体等），你可以自定义它们的文件名和路径。

Webpack 5 中的 `assetModuleFilename` 和 Asset Modules 的引入，实际上可以取代之前常用的 `file-loader` 和 `url-loader`

* **`asset/resource`** :类似于 `file-loader`，会将文件单独打包并输出到指定目录。
* **`asset/inline`** :类似于 `url-loader`，将文件以 base64 字符串的形式内联到打包的代码中，适合小文件。
* **`asset`** :这个类型是 `asset/resource` 和 `asset/inline` 的结合，Webpack 会根据文件大小自动选择将文件打包为单独的文件还是内联到代码中。

#### **5. 多页面应用的路径处理**

```js
output: {
  path: path.resolve(__dirname, 'dist'),
  filename: '[name]/js/[name].[contenthash].js',  // 根据页面生成不同目录
  publicPath: '/static/',
},

```

##### **是什么？**

* 在多页面应用中，为每个页面生成独立的输出路径，确保各页面的资源加载路径正确。

##### **为什么？**

- 多页面应用通常需要为每个页面生成独立的资源路径，以避免资源冲突并确保正确的资源加载。

##### **怎么做？**

* 使用 `[name]` 模板结合目录结构配置 `filename`，确保每个页面的资源路径独立且有序。

这种配置会生成类似于：

* `/static/home/js/home.abc123.js`
* `/static/about/js/about.abc123.js`

### **6. 动态路径设置：基于环境变量**

当Webpack解析到一个文件引用另一个文件时，它会将这个引用转换为相对于 `output.publicPath`的路径。

举个例子，a文件中相对引用了b文件，在读取时是相对于a文件的路径。接着Webpack 会解析这个相对路径引用，并将其转换为基于 `output.publicPath` 的绝对路径。

有时，开发和生产环境路径的不同是由部署环境决定的。可以使用环境变量来动态调整路径是一个常见的做法。

```js
const isProduction = process.env.NODE_ENV === 'production';

module.exports = {
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'js/[name].[contenthash].js',
    publicPath: isProduction ? '/prod_static/' : '/',  // 生产环境与开发环境路径不同
  },
};

```

#### **是什么？**

- `publicPath`配置的是资源在运行时的根路径。

* 动态设置 `publicPath`，根据当前环境（开发或生产）调整资源的加载路径。

#### **为什么？**

开发环境与生产环境的路径需求不同。生产环境通常需要固定的路径结构，而开发环境则需要灵活性。

#### **怎么做？**

* 使用环境变量来动态调整 `publicPath`，确保在开发和生产环境中路径配置合理，且不影响开发体验。

# Loader

### Loader是什么

loader 是一个模块转换器，用于将各种类型的文件转换为 Webpack 可以处理的模块。通过使用 loader，你可以在项目中直接引入并处理非 JavaScript 文件，如 CSS、图像、字体、TypeScript 等。

### 为什么要使用Loader

在 Webpack 中，依赖图是一个包含所有模块及其依赖关系的结构。虽然依赖图的主要目标是 JavaScript 模块，但**通过 loader，可以使其他类型的文件（如 CSS、HTML）间接地被包含在依赖图中。**

### Loader是怎么工作的

当 Webpack 遇到某个文件类型时，会根据配置找到相应的 loader，并使用该 loader 对文件进行处理，最后将处理后的结果包含在依赖图中。

### Loader使用

`loader`可以在 `import`或者加载模块时预处理文件。

在你的应用程序中，有三种使用 loader 的方式：

- [配置
  ](https://www.webpackjs.com/concepts/loaders/#configuration)

在 Webpack 配置文件中，使用 `module.rules` 字段来定义 loader。每个 loader 的配置通常包括 `test`、`use` 和可选的 `exclude` 或 `include` 字段：

* **test** : 用于匹配文件的正则表达式。
* **use** : 指定使用的 loader，可以是字符串、对象或数组。
* **exclude/include** : 排除或包含特定的文件或目录。

以下是一个示例 Webpack 配置文件，展示了如何配置多个 loader：

```js
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: 'babel-loader'
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      },
      {
        test: /\.tsx?$/,
        use: 'ts-loader',
        exclude: /node_modules/
      }
    ]
  }
};

```

- [内联](https://www.webpackjs.com/concepts/loaders/#inline)：在每个 `import` 语句中显式指定 loader。使用 `!` 将资源中的 loader 分开。分开的每个部分都相对于当前目录解析。

```javascript
import Styles from 'style-loader!css-loader?modules!./styles.css';
```

- [CLI](https://www.webpackjs.com/concepts/loaders/#cli)：在 shell 命令中指定它们。

```javascript
webpack --module-bind jade-loader --module-bind 'css=style-loader!css-loader'
```

### Loader的链式调用

当链式调用多个 `loader`的时候，它们会以相反的顺序执行。

- 最后的 `loader` 最早调用，将会传入原始资源内容。
- 第一个 `loader` 最后调用，期望值是传出 `JavaScript`和 `source map`（可选）。
- 中间的 `loader` 执行时，会传入前一个 `loader` 传出的结果。

### 常用的loader

#### **CSS Loader 和 Style Loader**：

`css-loader` 和 `style-loader` 的组合让 Webpack 可以处理 CSS 文件。

他们像这样配置

```js
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      }
    ]
  }
};
```

他们的作用

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

- **注入样式** ：
  `style-loader` 会接收到 `css-loader` 导出的 CSS 字符串，并将其动态地插入到 HTML 的 `<style>` 标签中。具体来说，它会创建一个 `<style>` 元素，将 CSS 字符串设置为其内容，并将其添加到文档的 `<head>` 部分。
  `style-loader` 的核心代码可能类似于以下内容（简化版）：

  ```js
  var css = require('./styles.css');

  var style = document.createElement('style');
  style.type = 'text/css';
  style.appendChild(document.createTextNode(css));

  document.head.appendChild(style);
  ```

  如果要把 `css`输出成单独的文件，而不是往 `DOM`里插入 `<style>`标签，那么就不能用 `style-loader`，而需要用 `MiniCssExtractPlugin`。
- `babel-loader`： 让下一代的 `js`文件转换成现代浏览器能够支持的 `js`文件。`babel`有些复杂，所以大多数都会新建一个 `.babelrc`进行配置。
- `less-loader`和 `sass-loader`可以加载和转译 `less`和 `sass`文件

```javascript
{test:/\.less$/,use:['style-loader','css-loader','less-loader']},
//配置处理.less文件的第三方loader规则
{test:/\.scss$/,use:['style-loader','css-loader','sass-loader']},
//配置处理.scss文件的第三方loader规则
```

# Plugins

`plugins`和 `loader`都是外部引用

### plugins和loader区别

`loader`负责处理**源文件**，如 `css`、`jsx`，一次处理一个文件。而 `plugins`并不是直接操作单个文件，它直接对**整个构建过程**起作用，在特定的生命周期用特定的钩子处理回调。

### 常用的 `plugins`的用法

#### `MiniCssExtractPlugin`

作用:将 `css`提取到单独的文件中。它为每个包含 `CSS`的 `JS`文件创建一个 `CSS`文件，它支持 `CSS`和 `SourceMap`的按需加载。

一般在生产环境下使用。

在开发环境中，一般使用 `style-loader`，把 `css`变成 `style`标签插入到 `html`中，更新时，只要从内存中读取新的 `html`文件。

在生产环境中，由于以下情况，我们必须要使用 `MiniCssExtractPlugin`

- 把文件全部放在 `html`中，`html`文件可能过大，影响用户体验。而加载多个文件，由于 `html2.0`有多路传输的特性，所以会比加载一个文件更快。（在开发环境中，则要用io读取文件，所以更慢）
- 把 `CSS`都变成 `style`标签插入到 `html`中有点乱，不美观。

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin'); //通过 npm 安装
const webpack = require('webpack'); //访问内置的插件
const config = {
	...
  plugins: [
    new webpack.optimize.UglifyJsPlugin(),
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};

```

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

#### `HotModuleReplacementPlugin`

它允许你在修改组件代码时，自动刷新实时预览修改后的结果。注意永远不要在生产环境中使用 `HMR`。这儿说一下一般情况分为开发环境，测试环境，生产环境。
用法如 `new webpack.HotModuleReplacementPlugin()`
webapck.config.js的全部内容

```javascript
const webpack = require("webpack")
        const HtmlWebpackPlugin = require("html-webpack-plugin")
        var ExtractTextPlugin = require('extract-text-webpack-plugin')
        var lessRules = {
            use: [
                {loader: 'css-loader'},
                {loader: 'less-loader'}
            ]
        }
        module.exports = {
            entry: {
                    main: './src/index.js'
                },
            output: {
                filename: '[name].js',
                path: path.resolve('./build')
            },
            devServer: {
                contentBase: '/src',
                historyApiFallback: true,
                inline: true,
                hot: true
            },
            module: {
                rules: [
                    {test: /\.less$/, use: ExtractTextPlugin.extract(lessRules)}
                ]
            },
            plugins: [
                new ExtractTextPlugin('main.css')
            ]
        }
```

# 产品阶段的构建

目前为止，在开发阶段的东西我们已经基本完成了。但是在产品阶段，还需要对资源进行别的
处理，例如压缩，优化，缓存，分离 `css`和 `js`。首先我们来定义产品环境

```javascript
var ENV = process.env.NODE_ENV
    var baseConfig = {
        plugins: [
            new webpack.DefinePlugin({
                'process.env.NODE_ENV': JSON.stringify(ENV)
            })
        ]
    }
```

然后还需要修改我们的script命令

```javascript
"scripts": {
    "start": "NODE_ENV=development webpack-dev-server",
    "build": "NODE_ENV=production webpack"
}
```

`process.env.NODE_ENV` 将被一个字符串替代，它运行压缩器排除那些不可到达的开发代码分支。
当你引入那些不会进行生产发布的代码，下面这个代码将非常有用。

```javascript
if (process.env.NODE_ENV === 'development') {
   console.warn('这个警告会在生产阶段消失')
}
```

### 缓存

使用 webpack 来打包应用程序，webpack 会生成一个可部署的 `/dist` 目录，然后把打包后的内容放置在此目录中。只要 `/dist` 目录中的内容部署到服务器上，客户端（通常是浏览器）就能够访问网站此服务器的网站及其资源。

而最后一步获取资源是比较耗费时间的，浏览器可以通过命中缓存，使网站加载速度更快。然而，如果我们在部署新版本时不更改资源的文件名，浏览器可能会认为它没有被更新，就会使用它的缓存版本。由于缓存的存在，当你需要获取新的代码时，就会显得很棘手。

通过使用 `output.filename` 进行[文件名替换](https://www.webpackjs.com/configuration/output#output-filename)，可以确保浏览器获取到修改后的文件。
