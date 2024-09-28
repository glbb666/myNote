### 什么是 Rollup

Rollup 是一个 JavaScript 模块打包器，旨在将多个模块打包成一个或多个文件。它主要用于构建 JavaScript 库和应用程序，支持 ES6 模块（ESM）和 CommonJS 模块。Rollup 通过 tree-shaking 功能有效地移除未使用的代码，从而生成小巧且高效的输出文件。

### 为什么重要

1. **小巧的打包结果** ：Rollup 通过静态分析代码，能够有效地移除未使用的模块，从而减少打包文件的大小，提高加载速度。
2. **支持 ES 模块** ：Rollup 支持 ES6 模块，允许开发者使用现代 JavaScript 特性，并保持代码的模块化。
3. **插件生态** ：Rollup 拥有丰富的插件生态系统，支持多种功能扩展，例如代码转译、样式处理等，使其适用于各种项目需求。
4. **适合库开发** ：Rollup 尤其适合用于构建 JavaScript 库，可以生成多种格式的输出（如 UMD、CommonJS、ESM），以便于在不同环境中使用。

### 怎么做

**创建配置文件** ：

1. 在项目根目录下创建 `rollup.config.js`，配置输入、输出和插件等。

```js
export default {
     input: 'src/index.js',
     output: {
       file: 'dist/bundle.js',
       format: 'iife', // 可以是 'cjs', 'esm', 'umd' 等
     },
     plugins: [
       // 可以添加插件
     ]
};

```
