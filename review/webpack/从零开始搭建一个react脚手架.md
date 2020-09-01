

### 初始化

```nginx
yarn init -y
```

>   `yarn init`可以快速初始化 `package.json` 文件。但是执行之后会有一个交互式的命令行让你输入需要的字段值，当然如果你想直接使用默认值，也可以使用` yarn init -y` 来超速初始化。

- webpack是执行在node环境，不是执行在浏览器的，node环境不支持import
- "start": "webpack --config ./config/webpack.config.js"，修改默认打包的地址，否则是直接从当前目录找到webpack.config.js





### 参考文章

[Webpack从零配置React的运行环境上篇——搭建基本环境](https://segmentfault.com/a/1190000021464216)

