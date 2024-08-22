# 为什么文件名中需要带hash？

每次 `前端静态`资源需要更新时，客户端必须重新下载资源。因为从网络中获取资源会很慢，这显然非常低效。这也是为什么浏览器会缓存静态资源的原因。但是有一个缺陷：如果在部署新的版本中不修改文件名，浏览器会认为它没有更新，会继续使用缓存中的旧版本。

文件名加上hash可以保证我们应用发版更新的同时客户端也能及时获取最新版本。

# hash的类型

## hash

**生成依据** ：当你使用 `[hash]` 时，Webpack 会基于整个项目的构建状态生成一个全局哈希值。如果项目中的任何一个文件（无论是 JavaScript、CSS、图片，还是其他资源文件）发生变化，或者是 Webpack 配置改变了，整个构建的哈希值 `[hash]` 都会发生变化。

**影响范围** ：由于 `[hash]` 是全局的，当[hash]发生变化，所有使用了 `[hash]` 的文件名都会更新。

**适用场景** ：`[hash]` 通常适用于生成一次构建对应的文件名。比如开发环境，因为它保证每次构建后文件名都是唯一的，避免缓存问题。

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

如下：修改css文件导致index.css、index.js和vendor.js的hash内容都改变了

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

## chunkhash

**生成依据** ：`[chunkhash]` 基于 chunk 的内容生成。因此当某个 chunk 中的文件内容发生变化时，只有该 chunk 的 `[chunkhash]` 会变化，而其他 chunk 的 `[chunkhash]` 不受影响。

**影响范围** ：用于生成 `[chunkhash]`的 所有文件。

**适用场景** ：适用于分离打包后的文件（chunk）之间的缓存控制。例如，在动态加载的代码块中，只有当某个代码块发生变化时，相关的 `[chunkhash]` 才会更新，其他没有变化的代码块可以继续使用旧的缓存文件。

```javascript
entry:{
  index:'./src/index.js',// index chunk包含 index.js和index.css
  vendor:['react','react-dom'] //每个entry都会打一个
}
output:{
  filename:'[name].[chunkhash].js', 
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

## contenthash

* **生成依据** : `[contenthash]` 是基于单个文件内容生成的哈希值。
* **影响范围**: 用于生成 `[contenthash]`的文件。
* **适用场景** : `[contenthash]` 常用于处理静态资源文件（如 CSS、图片等），因为它可以更好地控制文件级别的缓存。当文件内容不变时，浏览器缓存可以继续使用旧的文件，减少不必要的网络请求。

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

## 场景与使用

### 开发环境

开发环境不需要hash直接展示名称，毕竟生成hash也需要消耗一定资源，cache还会影响开发体验。

### 生产环境

- 公共库使用chunkhash：公共库（如 React、Lodash 等第三方库）通常更新频率较低。使用 `chunkhash` 能确保当这些库的代码没有变化时，它们的哈希值保持不变，这样浏览器可以有效地缓存这些文件，减少重新下载的次数。
- 非库使用要 `contenthash`：修改哪个文件才改变哪个文件的 `hash`。其它的 `hash`不变可以继续从缓存里读取，以加快访问速度
