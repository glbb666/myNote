[前端面试之webpack篇](https://segmentfault.com/a/1190000011383224)

# Webpack是什么

`webpack`是一个**模块打包机**，它从一个给定的的主文件开始找到项目的所有依赖文件，使用`loader`进行处理，最后打包成浏览器可识别的`js`文件。

在使用`webpack`命令的时候，他将接受`webpack`的配置文件，除非我们使用其他的操作。

# entry

**entry:** 用来写入口文件，它将是整个依赖关系的根

```javascript
var baseConfig = {
        entry: './src/index.js'
}
```

- 当我们需要多个入口文件的时候，可以把entry写成一个对象

```javascript
var baseConfig = {
        entry: {
            main: './src/index.js',
            app:'./src/index1.js'
        }
}
```

- 我们还可以向`entry`中传入一个数组，这样将创建多个主入口。
- 我们还可以分离应用`app`和第三方库入口

 webpack 从 `app.js` 和 `vendors.js` 开始创建依赖图。这些依赖图是彼此完全分离、互相独立的

第三方库入口的代码打包一次之后就不会变了，除非你手动更新`package.json`的依赖

```javascript
const config = {
  entry: {
    app: './src/app.js',
    vendors: ["react", "react-redux", "react-router", "redux", "rc-form", "lodash"]
  }
};
```

`vendor`允许你使用`CommonsChunkPlugin`从「应用程序 bundle」中提取 vendor 引用到 vendor bundle，并把引用 vendor 的部分替换为 `__webpack_require__()` 调用。如果应用程序 bundle 中没有 vendor 代码，那么你可以在 webpack 中实现被称为[长效缓存](https://www.webpackjs.com/guides/caching)的通用模式。

缓存我们在后面讲。

# output

**output:** 即使入口文件有多个，但是**只有一个输出配置**

- `filename` 用于输出文件的文件名。
- `path`目标输出目录的绝对路径。

单个入口文件

```
output: {
        filename: 'main.js',
        path: path.resolve('./build')
}
```

如果你定义的入口文件有多个，那么我们需要使用占位符来确保输出文件的唯一性

```javascript
output: {
        filename: '[name].js',
        path: path.resolve('./build')
}
```

```javascript
var path = require('path')
    var baseConfig = {
        entry: {
            main:path.join(__dirname,'./src/main.js'),
        		app:path.join(__dirname,'./src/main.js')
        },
        output: {
            filename: 'main.js',
            path: path.resolve('./build')
        }
    }
    module.exports = baseConfig
```

![webpack.png](https://github.com/glbb666/myNote/blob/master/review/webpack/images/webpack.png?raw=true)

如今这么少的配置，就能够让你运行一个服务器并在本地使用命令`npm start`或者`npm run build`来打包我们的代码进行发布

# Loader

### loader的作用

`loader`可以将文件从不同的语言转化成`js`，从而使其能够被添加到依赖图中。

`loader`可以在`import`或者加载模块时预处理文件。

在你的应用程序中，有三种使用 loader 的方式：

- [配置](https://www.webpackjs.com/concepts/loaders/#configuration)（推荐）：在 **webpack.config.js** 文件中指定 loader。

```javascript
module.exports = {
  module: {
    rules: [
      { test: /\.ts$/, use: 'ts-loader' },
      { 
        test: /\.css$/,
        use: [
          { loader: 'style-loader' },
          {
            loader: 'css-loader',
            options: {
              modules: true
            }
          }
        ]
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

### loader的链式调用

当链式调用多个` loader `的时候，请记住它们会以相反的顺序执行。取决于数组写法格式，从右向左或者从下向上执行。

- 最后的 `loader` 最早调用，将会传入原始资源内容。
- 第一个 `loader` 最后调用，期望值是传出` JavaScript `和 `source map`（可选）。
- 中间的 `loader` 执行时，会传入前一个 `loader` 传出的结果。

### 常用的loader

- `style-loader`：在 `DOM` 里插入一个 `<style>` 标签，并且将 `CSS` 写入这个标签内。

- `css-loader`解析`CSS`文件后，使用 `import `加载，并且返回 CSS 代码

- `babel-loader`： 让下一代的`js`文件转换成现代浏览器能够支持的`js`文件。

`babel`有些复杂，所以大多数都会新建一个`.babelrc`进行配置

```javascript
{test:/\.css$/,use:['style-loader','css-loader']}
```

`less-loader`和`sass-loader`可以加载和转译`less`和`sass`文件

```javascript
{test:/\.less$/,use:['style-loader','css-loader','less-loader']},
//配置处理.less文件的第三方loader规则
{test:/\.scss$/,use:['style-loader','css-loader','sass-loader']},
//配置处理.scss文件的第三方loader规则
```

`file-loader`: 生成的文件名就是**文件内容的`md5`哈希值**并会保留所引用资源的原始扩展名
`url-loader`: 功能类似 `file-loader`,但是文件大小低于指定的限制时，可以返回一个`DataURL`

### loader的解析

loader 遵循标准的[模块解析](https://www.webpackjs.com/concepts/module-resolution/)。多数情况下，loader 将从[模块路径](https://www.webpackjs.com/concepts/module-resolution/#module-paths) `node_modules`解析。

# Plugins

`plugins`和`loader`都是外部引用

### plugins和loader区别

`loader`负责处理**源文件**，如`css`、`jsx`，一次处理一个文件。而`plugins`并不是直接操作单个文件，它直接对**整个构建过程**起作用，在特定的生命周期用特定的钩子处理回调。

### 常用的`plugins`的用法
#### `ExtractTextWebpackPlugin`（`webpack 4.x`版本不可用)

作用:将`js`文件中引用的样式单独抽离成`css`文件

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

#### `HtmlWebpackPlugin`:
作用：根据模版生成`html`文件，并自动引用打包后的`js`，`css`文件

```javascript
var HTMLWebpackPlugin = require('html-webpack-plugin')
var baseConfig = {
    plugins: [
        new htmlWebpackPlugin({//创建一个在内存中生成html页面的插件
            template:path.join(__dirname,'./src/index.html'),//指定模板页面，将来会根据指定的页面路径，去生成内存中的页面
            filename:'index.html'//指定生成的页面的名称
        })
    ]
}
```

#### `HotModuleReplacementPlugin`

它允许你在修改组件代码时，自动刷新实时预览修改后的结果。注意永远不要在生产环境中使用`HMR`。这儿说一下一般情况分为开发环境，测试环境，生产环境。
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
处理，例如压缩，优化，缓存，分离`css`和`js`。首先我们来定义产品环境

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