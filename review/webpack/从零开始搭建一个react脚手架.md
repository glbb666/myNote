

### 初始化

```nginx
yarn init -y
```

>   `yarn init`可以快速初始化 `package.json` 文件。但是执行之后会有一个交互式的命令行让你输入需要的字段值，当然如果你想直接使用默认值，也可以使用` yarn init -y` 来超速初始化。

- webpack是执行在node环境，不是执行在浏览器的，node环境不支持import
- "start": "webpack --config ./config/webpack.config.js"，修改默认打包的地址，否则是直接从当前目录找到webpack.config.js

### 区分生产环境和开发环境

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
    app: './src/app.js',
  },
  output: {
    filename: 'js/bundle.js',
    path: path.resolve(__dirname, '../dist')
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

### 配置babel

此时还是编译不了`jsx`模版的，我们需要配置`babel`

首先我们要安装依赖

- **babel-loader：**使用Babel和webpack来转译JavaScript文件。
- **@babel/preset-react：**转译react的JSX
- **@babel/preset-env：**转译ES2015+的语法
- **@babel/core：**babel的核心模块

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

### 给文件名加上hash

[Hash&chunkhash&contenthash的使用以及他们之间的区别](./hash&chunkhash&contenthash的差别.md)

```javascript
entry:{
  index:'./src/index.js',// index chunk包含 index.js和index.css
  vendor:['react','react-dom'] //每个entry都会打一个
}
output:{
  filename:'[name].[contenthash].js', 
},
plugins:[
    new MiniCssExtractPlugin({
      filename:'[name].[contenthash].css', //main entry名
    })
  ]
]
```

### 代码分割

splitchunk

### 代码规范

Eslint,stylelint,prettier

### 参考文章

[Webpack从零配置React的运行环境上篇——搭建基本环境](https://segmentfault.com/a/1190000021464216)

[前端面试之道之webpack篇](https://segmentfault.com/a/1190000011383224)