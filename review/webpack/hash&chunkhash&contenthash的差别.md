### 为什么需要hash？

每次`前端静态`资源需要更新时，客户端必须重新下载资源。因为从网络中获取资源会很慢，这显然非常低效。这也是为什么浏览器会缓存静态资源的原因。但是有一个缺陷：如果在部署新的版本中不修改文件名，浏览器会认为它没有更新，会继续使用缓存中的旧版本。

文件名加上hash可以保证我们应用发版更新的同时客户端也能及时获取最新版本。

### hash

- 基于build
- 所有chunk文件使用相同的hash。
- 项目中任一文件内容发生变化都会影响所有chunk文件hash

```javascript
entry:{
  index:'./src/index.js',// index chunk包含 index.js和index.css
  vendor:['react','react-dom'] //每个entry都会打一个
}
output:{
	filename:'[name].[hash].js', //chunkhash和contenthash不能在这用, 这能用hash
},
plugins:[
  new MiniCssExtractPlugin({
    filename:'[name].[hash].css', //main entry名
  })
]

```

- 修改css文件导致index.css、index.js和vendor.js的hash内容都改变了

```javascript
dist
├── images
│ └── accd7e392b0ddd8dd91f19edd9530282.png
├── index.a0b3b5557d9b4fb15cbe.css
├── index.a0b3b5557d9b4fb15cbe.js
├── index.c4275599e9a903cd4997.css
├── index.c4275599e9a903cd4997.js
├── index.html
├── vendor.a0b3b5557d9b4fb15cbe.js
└── vendor.c4275599e9a903cd4997.js
```

### chunkhash

- 基于 webpack 的 `entry point`
- 任意文件改变只会影响其所属的chunk，不会影响其它chunk。

```javascript
entry:{
  index:'./src/index.js',// index chunk包含 index.js和index.css
  vendor:['react','react-dom'] //每个entry都会打一个
}
output:{
  filename:'[name].[chunkhash].js', //chunkhash和contenthash不能在这用, 这能用hash
},
plugins:[
  new MiniCssExtractPlugin({
    filename:'[name].[chunkhash].css', //main entry名
  })
]

```

- 修改（css和index.js各一次）都导致index.css、index.js的hash改变，但没有影响vendor的hash

```javascript
dist
├── images
│ └── accd7e392b0ddd8dd91f19edd9530282.png
├── index.1e3c981f3953ec527872.css
├── index.1e3c981f3953ec527872.js
├── index.85beca714bffd11c5c3d.css
├── index.85beca714bffd11c5c3d.js
├── index.bde5e43b7c6b7cd96777.css
├── index.bde5e43b7c6b7cd96777.js
├── index.html
└── vendor.c6fa41a5d8c4f0002b09.js
```

### contenthash

- 基于文件内容产生的hash
- 影响范围只限于`本文件`

```javascript
entry:{
  index:'./src/index.js',// index chunk包含 index.js和index.css
  vendor:['react','react-dom'] //每个entry都会打一个
}
output:{
  filename:'[name].[contenthash].js', //chunkhash和contenthash不能在这用, 这能用hash
},
plugins:[
    new MiniCssExtractPlugin({
      filename:'[name].[contenthash].css', //main entry名
    })
  ]
]
```

- 修改css内容，只有css的hash改变了

```javascript
dist
├── images
│ └── accd7e392b0ddd8dd91f19edd9530282.png
├── index.1fd2b4940eda072b7e6b.js
├── index.30bcc0d7e84e3620dc24.css
├── index.4a410042517d488eecd4.css
├── index.html
└── vendor.d67f4f207409e75aec17.js
```

### 场景与使用

- production 只需要`contenthash`就可以了，修改哪个文件才改变哪个文件的`hash`。其它的`hash`不变可以继续从缓存里读取，以加快访问速度
- development环境 不需要hash直接展示名称，毕竟生成hash也需要消耗一定资源，cache还会影响开发体验。

