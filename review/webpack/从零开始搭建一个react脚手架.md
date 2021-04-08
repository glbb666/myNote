

### 初始化

[首先我们需要了解`package.json`](./package.json详解.md)

```nginx
yarn init -y
```

>   `yarn init`可以快速初始化 `package.json` 文件。但是执行之后会有一个交互式的命令行让你输入需要的字段值，当然如果你想直接使用默认值，也可以使用` yarn init -y` 来超速初始化。

- `webpack`是执行在`node`环境，不是执行在浏览器的，node环境不支持`import`

### 区分生产环境和开发环境

#### 为什么需要区分生产环境和开发环境

因为开发是一个过程，生产是一个目的。在开发的过程中会产生错误，所以开发环境的配置需要方便开发人员的调试，而生产环境的目的是有效的运行。

我们需要一个公共的配置，然后基于这个公共的配置上，生成生产环境的配置和开发环境的配置

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
    app:'./src/index.js',
    vendor:['react','react-dom']
  },
  output: {
    // 主bundle
    filename:'js/[name][contenthash].js',
    chunkFilename:'js/[name][contenthash].js',
    //此处的__dirname为/Users/bytedance/Desktop/webpack-test/config
    path: path.resolve(__dirname, '../dist'),
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

在生产环境`webpack.prod.config.js`就可以这样使用`merge`

```javascript
const merge = require('webpack-merge');
const common = require('./webpack.common.config.js');

module.exports = merge(common, {
  mode: 'production',
});
```

然后修改`package.json`文件，添加指令

```javascript
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
-   "start": "webpack --config ./config/webpack.common.config.js",
+   "build": "webpack --config ./config/webpack.prod.config.js"
  },
```

#### 生产环境和开发环境的共同需求

- 同样的入口
- 同样的代码处理
- 同样的解析配置

#### 生产环境的需求

- `hotModulePlugin`模块热更新

- `sourceMap `(方便打包调试)
- `proxyTable`接口代理
- `eslint style-lint`代码规范检查

#### 开发环境的需求

- 提取公共代码
- 压缩混淆
- 缓存文件
- 图片进行`base64`编码
- 去除无用的代码

### 公共配置

#### 配置babel

`babel`的原理：`babel-loader`其实只是`webpack`和`babel`的接口，`babel-core`才是负责把`es6`的代码转换成浏览器可执行的`js`代码。

首先`babel`需要把代码首先转换成抽象语法树，接着用`babel-transform`遍历修改抽象语法树，得到新的抽象语法树。接着再用`babel-transform`把

[要了解babel的原理可以看这里](https://juejin.im/post/6844904190343381005)

此时还是编译不了`jsx`模版的，我们需要配置`babel`

首先我们要安装依赖

- **babel-loader：**使用`Babel`和`webpack`来转译`JavaScript`文件。
- **@babel/preset-react：**转译`react`的`JSX`
- **@babel/preset-env：**转译`ES2015+`的语法
- **@babel/core：**`babel`的核心模块

```javascript
yarn add --save-dev babel-loader @babel/preset-react @babel/preset-env @babel/core
```

根目录新建一个配置文件`.babelrc` 配置相关的"presets"

```javascript
{
    "presets": [// presets 就是 plugins 的组合
      [
        "@babel/preset-env",
        {
          "targets": {
            // 大于相关浏览器版本无需用到 preset-env
            "edge": 17,
            "firefox": 60,
            "chrome": 67,
            "safari": 11.1
          },
          // 根据代码逻辑中用到的 ES6+语法进行方法的导入，而不是全部导入
          "useBuiltIns": "usage" //useBuiltIns就是是否开启自动支持 polyfill，它能自动给每个文件添加其需要的poly-fill。
        }
      ],
      "@babel/preset-react"
    ],
    "plugins": [
      ["@babel/plugin-proposal-decorators", { "legacy": true }]// 转义ES7的修饰器@
    ]
  }
```

再修改`webpack.common.config.js` ，添加如下代码：

```javascript
 module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        use: 'babel-loader',
        exclude: /node_modules/,//不需要去转译"node\_modules"这里面的文件。
      }
    ]
  }
```

### 生产环境的需求

#### `hotModulePlugin`模块热更新 

本地开启服务，实时更新。首先要下载`webpack-dev-server`

```javascript
yarn add webpack-dev-server --save-dev
```

接着在`package.json`中配置

```javascript
 "scripts": {
    "start": "webpack-dev-server --config ./config/webpack.dev.config.js",
  }
```

在`webpack.dev.config.js`中进行配置，由于是热更新，所以输出文件的名字中应该包含`hash`而非`contenthash`。

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
  },
  plugins: [
    new webpack.HotModuleReplacementPlugin()
  ]
})
```

**注意：你启动`webpack-dev-server`后，你在目标文件夹中是看不到编译后的文件的,实时编译后的文件都保存到了内存当中。因此很多同学使用`webpack-dev-server`进行开发的时候都看不到编译后的文件**

#### `sourceMap`方便调试

```javascript
module.exports = merge(common,{
  ...
  devtool: 'inline-source-map',
  ...
})
```

#### 代码规范的检查

```javascript

```

### 开发环境的需求

#### 压缩混淆

- `js`压缩

原来一般是用`uglifyjs-webpack-plugin`进行压缩，但是`uglifyjs-webpack-plugin`不支持压缩`es6`。

在`webpack4.0`版本之后，只要设置`mode=production`，就会自动启用`terser-webpack-plugin`进行代码的压缩。同时，还可以进行以下的设置关闭代码的压缩。

```javascript
module.exports = merge(common,{
  ...
  optimization: {minimize:false} 
  ...
})

```

- `css`压缩

```javascript
const OptimizeCss = require('optimize-css-assets-webpack-plugin');
module.exports = merge(common,{
  ...
  plugins: [
    new OptimizeCss(),//压缩css代码
    ...
  ],
  ...
});
```

#### 给文件名加上hash

[Hash&chunkhash&contenthash的使用以及他们之间的区别](./hash&chunkhash&contenthash的差别.md)

使用 webpack 来打包应用程序，webpack 会生成一个可部署的 `/dist` 目录，然后把打包后的内容放置在此目录中。只要 `/dist` 目录中的内容部署到服务器上，客户端（通常是浏览器）就能够访问网站此服务器的网站及其资源。

而最后一步获取资源是比较耗费时间的，浏览器可以通过命中缓存，使网站加载速度更快。然而，如果我们在部署新版本时不更改资源的文件名，浏览器可能会认为它没有被更新，就会使用它的缓存版本。由于缓存的存在，当你需要获取新的代码时，就会显得很棘手。

通过使用 `output.filename` 进行[文件名替换](https://www.webpackjs.com/configuration/output#output-filename)，可以确保浏览器获取到修改后的文件。 

#### 图片进行`base64`编码

- `file-loader`: 生成的默认文件名就是**文件内容的`md5`哈希值**([hash.[ext])，并会保留所引用资源的原始扩展名。
- `url-loader`：和`file-loader`类似，但可以在`options`属性中进行一个`limit`的配置。这是因为当页面图片较多时，需要发起很多`http`请求，会降低页面的性能，所以`url-loader`可以把图片进行编码，打包进`js`文件中，当然，如果图片过大，进行编码会消耗性能。所以在配置中有一个`limit`值，如果文件的大小比`limit`小，就会将文件转为`dataURI`存在`js`文件中，并返回`DATAURI`。如果文件的大小比`limit`大，就会用`file-loader`进行处理。详情可以了解[详解`webpack url-loader`和`file-loader`

#### `splitchunk`代码分割

#### `treeShaking`去除无用的代码

当`webpack4`，且`mode='production'`的时候，会自动进行`treeShaking`。

`es6`模块化的静态分析是`treeShaking`的基础。因为静态分析在解析时就可以明确模块之间的依赖关系，

### Webpack优化

#### 缩短`webpack`的打包时间 

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

##### 利用`thread-loader`进行多进程压缩

把`thread-loader`放在`loader`的前面，这样`thread-loader`之后的`loader`就会在单独的一个`worker`池中运行。

##### 使用`dllPlugin`

这样一些不经常改变的第三方库就可以提前打包好，下次打包的时候的速度就会变快。

#### 提高开发体验

去除掉`console.log`，因为开发的时候虽然有`code review`，但是以前开发的一些代码，还是会打出一些`console.log`的信息。

#### 提高用户体验

- 按需加载
- `splitchunk`，把公共模块提取出来。如果不把代码块进行分割，最后`webpack`就会把所有的代码都打到一个包里面去，这样的包很笨重，也不利于下载。`http2.0`正好支持多路复用，最多支持6个并行的请求，这样的话就可以提高下载的时间，优化用户的体验。

### 参考文章

[Webpack从零配置React的运行环境上篇——搭建基本环境](https://segmentfault.com/a/1190000021464216)

[前端面试之道之webpack篇](https://segmentfault.com/a/1190000011383224)

[从零开始搭建一个webpack脚手架](https://segmentfault.com/a/1190000016661311?utm_source=sf-related)