[前端面试之webpack篇](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fskychenbo)

# Webpack是什么

`webpack`是一个**模块打包机**，它把项目当作一个整体，从一个给定的的主文件开始找到项目的所有依赖文件，使用`loader`进行处理，最后打包成浏览器可识别的`js`文件。

![img](https://user-gold-cdn.xitu.io/2017/9/27/2395e1d01728fbb0d740d53c4530ed5b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

# install

首先添加我们即将使用的包：

```bash
npm install webpack webpack-dev-server --save-dev
```

`webpack`是我们需要的模块打包机，`webpack-dev-server`用来创建本地服务器，监听你的代码修改，并自动刷新修改后的结果。这些是有关`devServer`的配置

```javascript
contentBase,  // 为文件提供本地服务器
port, // 监听端口，默认8080
inline, // 设置为true,源文件发生改变自动刷新页面
historyApiFallback  // 依赖HTML5 history API,如果设置为true,所有的页面跳转指向index.html
devServer:{
    contentBase: './src' // 本地服务器所加载的页面所在的目录
    historyApiFallback: true, // 不跳转
    inline: true // 实时刷新
}
然后我们在根目录下创建一个'webpack.config.js'，在'package.json'添加两个命令用于本地开发和生产发布
"scripts": {
            "start": "webpack-dev-server",
            "build": "webpack"
}
```

在使用`webpack`命令的时候，他将接受`webpack`的配置文件，除非我们使用其他的操作

# entry(多个入口文件)

**entry:** 用来写入口文件，它将是整个依赖关系的根

```javascript
var baseConfig = {
        entry: './src/index.js'
}
```

当我们需要多个入口文件的时候，可以把entry写成一个对象

```javascript
var baseConfig = {
        entry: {
            main: './src/index.js',
            app:'./src/index1.js'
        }
}
```

我建议使用后面一种方法，因为他的规模会随你的项目增大而变得繁琐

# output（一个输出配置）

**output:** 即使入口文件有多个，但是**只有一个输出配置**

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
实现对不同格式的文件的处理，比如说将`scss`转换为`css`，或者`typescript`转化为`js`，从而使其能够被添加到依赖图中

`loader`是`webpack`最重要的部分之一，通过使用不同的`Loader`，我们能够调用外部的脚本或者工具，实现对不同格式文件的处理，`loader`需要在`webpack.config.js`边单独用`module`进行配置，配置如下：

```javascript
test: 匹配所处理文件的扩展名的正则表达式（必须）
    loader: loader的名称（必须）
    include/exclude: 手动添加处理的文件，屏蔽不需要处理的文件（可选）
    query: 为loaders提供额外的设置选项
    ex: 
        var baseConfig = {
            // ...
            module: {
                rules: [
                    {
                        test: /*匹配文件后缀名的正则*/,
                        use: [
                            loader: /*loader名字*/,
                            query: /*额外配置*/
                        ]
                    }
                ]
            }
        }
```

要是loader工作，我们需要一个正则表达式来标识我们要修改的文件，然后有一个数组表示
我们表示我们即将使用的Loader,当然我们需要的loader需要通过npm 进行安装。例如我们需要解析less的文件，那么webpack.config.js的配置如下：

```javascript
var baseConfig = {
                entry: {
                    main: './src/index.js'
                },
                output: {
                    filename: '[name].js',
                    path: path.resolve('./build')
                },
                devServer: {
                    contentBase: './src',
                    historyApiFallBack: true,
                    inline: true
                },
                module: {
                    rules: [
                        {
                            test: /\.less$/,
                            use: [
                                {loader: 'style-loader'},
                                {loader: 'css-loader'},
                                {loader: 'less-loader'}
                            ],
                            exclude: /node_modules/
                        }
                    ]
                }
            }
```

### 常用的loader

从下往上从右往左执行

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

# Plugins

`plugins`和`loader`都是外部引用

### plugins和loader区别

`loader`负责处理**源文件**，如`css`、`jsx`，一次处理一个文件。而`plugins`并不是直接操作单个文件，它直接对**整个构建过程**起作用，在特定的生命周期用特定的钩子处理回调。

### 常用的`plugins`的用法
#### `ExtractTextWebpackPlugin`（`webpack 4.x`版本不可用)

作用:将`js`文件中引用的样式单独抽离成`css`文件

```javascript
var ExtractTextPlugin = require('extract-text-webpack-plugin')
        var lessRules = {
            use: [
                {loader: 'css-loader'},
                {loader: 'less-loader'}
            ]
        }

        var baseConfig = {
            // ... 
            module: {
                rules: [
                    // ...
                    {test: /\.less$/, use: ExtractTextPlugin.extract(lessRules)}
                ]
            },
            plugins: [
                new ExtractTextPlugin('main.css')
            ]
        }
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

`HotModuleReplacementPlugin`: 它允许你在修改组件代码时，自动刷新实时预览修改后的结果注意永远不要在生产环境中使用HMR。这儿说一下一般情况分为开发环境，测试环境，生产环境。
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

# 优化插件

下面介绍几个插件用来优化代码
`OccurenceOrderPlugin`： 为组件分配ID,通过这个插件`webpack`可以分析和优先考虑使用最多 的模块，然后为他们分配最小的`ID`
`UglifyJsPlugin`： 压缩代码
下面是他们的使用方法

```javascript
var baseConfig = {
	new webpack.optimize.OccurenceOrderPlugin()
	new webpack.optimize.UglifyJsPlugin()
}
```


然后在我们使用`npm run build`会发现代码是压缩的

# 总结

`webpack`的配置文件的复杂度，依赖于你项目的需要。小心的运用他们。因为随着项目的增长，它们会变得很难驯服。