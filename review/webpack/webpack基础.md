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
* **`asset`** :这个类型是 `asset/resource` 和 `asset/inline` 的结合，Webpack 会根据文件大小自动选择将文件打包为单独的文件还是内联到代码中。这是因为当页面图片较多时，需要发起很多 `http`请求,会降低页面的性能，所以可以把图片进行编码，打包进 `js`文件中。当然，如果图片过大，进行编码会消耗性能。所以在配置中有一个 `limit`值，如果文件的大小比 `limit`小，就会将文件转为 `dataURI`存在 `js`文件中，并返回 `DATAURI`。如果文件的大小比 `limit`大，则用 `asset/resource`处理。

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

# resolve

## 是什么

`resolve` 配置主要用于控制模块解析的方式和路径。当在代码中使用 `import` 或 `require` 导入模块时，Webpack 使用 `resolve` 配置来决定如何找到这些模块的路径。

## 主要配置项

### **`resolve.alias`** ：

* 你可以使用别名来缩短路径。例如，设置 `@components`为 `src/components`，那么在代码中你可以通过 `@components/Button`来引用 `src/components/Button.js`。

### **`resolve.extensions`** ：

* 指定在导入模块时可以省略的文件扩展名列表。比如 `['.js', '.jsx', '.json']`，可以让你在导入 `Button`组件时省略 `.js`或 `.jsx`。

### **`resolve.modules `**

 定义模块查找路径。默认情况下，Webpack 会从node_modules 中查找模块。你可以添加其他路径，比如 `src`，以便优先从项目的源代码中查找模块。

### **`resolve.mainFiles `**

#### 是什么

当 Webpack 解析一个目录（而不是具体文件路径）时，它需要决定在该目录下应该默认查找哪个文件。`mainFiles` 是用来指定这个默认文件名的列表。

默认情况下，Webpack 会查找 `index` 文件（比如 `index.js` 或 `index.html`），因为 `index` 通常是目录下的主文件。：

#### 怎么做

如果你在代码中导入一个目录而没有指定具体文件，Webpack 会根据 `mainFiles` 配置去寻找目录中的默认文件。

```js
import myModule from './myDir';
```

**默认值**

假设 `myDir` 是一个目录，那么 Webpack 会尝试查找 `myDir/index.js`，或者 `myDir/index.html`， 因为 `index `是 `mainFiles` 默认的文件名。

**自定义**

你可以通过配置 `mainFiles` 来更改这个行为，比如让 Webpack 优先查找 `main.js` 文件：

```js
module.exports = {
  resolve: {
    mainFiles: ['main', 'index']
  }
};

```

  在这个配置下，Webpack 会首先查找 `main.js`，如果没有找到，再去查找 `index.js`。

### `resolve.mainFields`

#### 是什么

当 Webpack 解析一个来自 `node_modules` 的包时，它会查看该包的 `package.json` 文件，决定应该使用哪个文件作为模块的入口。

在 `package.json` 中， `mainFields` 是用来指定应该优先考虑哪些字段来找到这个入口文件。

#### 怎么做

**默认值** ：

默认情况下，Webpack 会按顺序查找 `browser`、`module` 和 `main` 字段。

* `browser`: 如果目标是浏览器环境，Webpack 会优先使用 `package.json` 中的 `browser` 字段指定的入口文件，这通常是为浏览器优化的版本。
* `module`: 如果有这个字段，并且你希望优先使用 ES Module 格式的代码，Webpack 会使用 `module` 字段指定的文件。
* `main`: 这是 CommonJS 模块的入口文件，通常是用于 Node.js 环境的。

如果一个模块的 `package.json` 文件中包含以下字段：

```js
{
  "browser": "dist/browser.js",
  "module": "dist/module.js",
  "main": "dist/index.js"
}

```

Webpack 会根据 `mainFields` 的配置选择合适的入口文件。如果你不改变默认配置，Webpack 会优先使用 `browser` 字段指定的 `dist/browser.js` 作为入口文件。

**自定义** ：

你可以根据项目的需求调整 `mainFields`

如下，这意味着 Webpack 会优先查找 `module` 字段，如果没有找到，再去查找 `main` 字段。

```js
module.exports = {
  resolve: {
    mainFields: ['module', 'main']
  }
};
```

## 怎么做

```js
const path = require('path');

module.exports = {
  // 入口文件  
  ...
  // 输出配置
  ...

  // 模块解析配置
  resolve: {
    // 路径别名
    alias: {
      '@components': path.resolve(__dirname, 'src/components/'),
      '@utils': path.resolve(__dirname, 'src/utils/'),
      '@assets': path.resolve(__dirname, 'src/assets/'),
    },

    // 自动解析扩展名
    extensions: ['.js', '.jsx', '.json', '.css'],

    // 模块查找路径
    modules: [path.resolve(__dirname, 'src'), 'node_modules'],

    // 优先采用的主文件名
    mainFiles: ['index'],

    // 入口字段名
    mainFields: ['browser', 'module', 'main'],
  },

  // 其他配置（如加载器、插件等）
  ...

  // 开发服务器配置
  ...
};

```

# `module`

## 是什么

 `module` 配置主要用于定义如何处理不同类型的模块。在 Webpack 构建过程中，`module` 指定了对各类文件（如 JavaScript、CSS、图片等）的处理方式，即通过什么样的 Loader 来加载和转换这些模块。

## 主要配置项

### **`rules`**

#### 是什么

`module.rules` 用于指定在构建过程中，哪些文件应使用哪些加载器（Loader）进行处理。

通常包括

* **`test`** ：一个正则表达式，用于匹配文件路径。所有符合这个正则表达式的文件都会应用此规则。
* **`use`** ：指定要使用的一组Loader和他预设的版本。Loader可以将文件从一种语言或格式转换为另一种语言或格式。
* **`include`** 和  **`exclude`** ：用于进一步限定应用此规则的文件范围。`include` 只包含指定目录或文件，`exclude` 则排除指定目录或文件。
* **`loader`** ：简写形式，用于指定单一加载器。
* **`options` 或 `query`** ：为 `loader`提供的配置信息。

#### 怎么做

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/, // 匹配 .js 文件
        exclude: /node_modules/, // 排除 node_modules 目录
        use: {
          loader: 'babel-loader', // 使用 babel-loader 处理 JS 文件
          options: {
            presets: ['@babel/preset-env'], // 配置 Babel 预设
          }
        }
      },
    ]
  }
};
```

### **`noParse`**

#### 是什么

`module.noParse` 是 Webpack 中的一个配置项，用于告诉 Webpack 某些模块不需要解析依赖关系。

#### 为什么

如果你使用的第三方库或模块已经是打包好的、独立的文件，可以使用 `module.noParse` 来告诉 Webpack 不要解析它们。这可以减少不必要的解析过程，从而提高构建性能。

#### 怎么做

```js
module.exports = {
  module: {
    noParse: /jquery|lodash/
  }
};

```

### **`parser`**

#### 是什么

和 `noParse`对比， `parser `是一种更细粒度的控制方式，允许你禁用特定模块中的某些语法解析，而不是完全跳过整个模块的解析。例如，你可以禁用 `require `的解析，但仍然保留对 `import` 的解析。

#### 为什么

##### 第三方库

如果你在项目中使用了一个已经经过编译或打包的库，这个库可能已经包含了自己的模块加载逻辑。在这种情况下，禁用 Webpack 对该库中模块语法的解析，可以避免不必要的解析和转换，节省构建时间。

```js
module: {
  rules: [
    {
      test: /some-precompiled-library\.js$/,
      parser: {
        commonjs: false, // 禁用对 require 的解析
        amd: false,      // 禁用对 AMD 模块的解析
        system: false,   // 禁用对 SystemJS 模块的解析
      }
    }
  ]
}

```

##### 构建中不使用的大型文件

对于某些大型文件或复杂的文件结构，解析所有模块语法可能会消耗大量的时间和资源。如果你知道这些文件不会在构建过程中使用（例如，它们是测试文件、文档文件或者包含静态资源的文件），禁用解析可以加快构建速度。

```js
module: {
  rules: [
    {
      test: /large-data-file\.js$/,
      parser: {
        import: false,
        export: false,
      }
    }
  ]
}

```

#### 怎么做

* **作用范围** ：`module` 涉及模块的加载和转换，决定了每个文件会通过什么样的工具进行预处理、编译和转化。

## 怎么做

```js
module.exports = {
  module: {
    rules: [
      {
        // 针对 .js 文件的处理规则
        test: /\.js$/,  // 匹配所有 .js 文件
        exclude: /node_modules/,  // 排除 node_modules 目录
        use: {
          loader: 'babel-loader',  // 使用 Babel 转换 JS 代码
          options: {
            presets: ['@babel/preset-env'],  // 使用 env 预设转换现代 JavaScript 语法
          },
        },
      },
      {
        // 针对 .css 文件的处理规则
        test: /\.css$/,  // 匹配所有 .css 文件
        use: ['style-loader', 'css-loader'],  // 使用 CSS 和样式加载器
      },
      {
        // 针对图片文件的处理规则
        test: /\.(png|jpg|gif)$/,  // 匹配图片文件
        type: 'asset/resource',  // 使用资源模块类型
      },
    ],
    noParse: /jquery|lodash/,  // 不解析这些库的依赖关系，提升构建性能
  },
};

```

# optimization

## 是什么

`optimization` 是 Webpack 用来控制构建产物优化行为的配置项。它能够影响代码拆分、压缩、模块合并等多种操作，以提升构建后的代码运行效率、减少体积，以及加快加载速度。

## 为什么

`optimization` 配置项的存在是为了帮助开发者更好地控制打包优化过程，确保打包后的代码更加高效、紧凑、快速。

## 怎么做

见webpack/从零开始搭建一个react脚手架，会说明在什么环境怎么使用。

# Loader

### Loader是什么

loader 是一个模块转换器，用于将各种类型的文件转换为 Webpack 可以处理的模块。通过使用 loader，你可以在项目中直接引入并处理非 JavaScript 文件，如 CSS、图像、字体、TypeScript 等。

### 为什么要使用Loader

在 Webpack 中，依赖图是一个包含所有模块及其依赖关系的结构。虽然依赖图的主要目标是 JavaScript 模块，但**通过 loader，可以使其他类型的文件（如 CSS、HTML）间接地被包含在依赖图中。**

### Loader是怎么工作的

当 Webpack 遇到某个文件类型时，会根据配置找到相应的 loader，并使用该 loader 对文件进行处理，最后将处理后的结果包含在依赖图中。

### Loader使用

`loader`可以在 `import`或者加载模块时预处理文件。

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

### Loader的链式调用

当链式调用多个 `loader`的时候，它们会以相反的顺序执行。

- 最后的 `loader` 最早调用，将会传入原始资源内容。
- 第一个 `loader` 最后调用，期望值是传出 `JavaScript`和 `source map`（可选）。
- 中间的 `loader` 执行时，会传入前一个 `loader` 传出的结果。

### Loader的具体使用

见webpack/从零开始搭建一个react脚手架，会说明在什么环境怎么使用。

# Plugins

`plugins`和 `loader`都是外部引用

### plugins和loader区别

`loader`负责处理**源文件**，如 `css`、`jsx`，一次处理一个文件。而 `plugins`并不是直接操作单个文件，它直接对**整个构建过程**起作用，在特定的时机使用特定的plugin去做一些事。

### 常用的 `plugins`的用法

见webpack/从零开始搭建一个react脚手架，会说明在什么环境怎么使用。
