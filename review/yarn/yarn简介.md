# 什么是 Yarn

Yarn 是一个 JavaScript 的包管理工具。

# 为什么重要

- 下载速度快：

  - 并行安装：`npm`按照队列执行 `package`，前面的安装完成才会安装后面的。`yarn`同步执行所有任务，提高了性能。
  - 离线模式：`yarn`有一个全局本地缓存。安装依赖优先从缓存中获取

```nginx
// 查看yarn本地缓存
yarn cache dir
```

- 安装版本统一：

  - `yarn`默认会生成 `yarn.lock`，记录确切安装的版本号。
    - 新增模块：创建（更新）`yarn.lock`文件
    - 拉取同一个依赖：使用之前的版本。
  - `npm`其实也有这样的文件存在，但是不会默认生成，需要运行 `npm shrinkwrap`。
- 更简洁的输出：

  - `yarn`只打必要信息，`npm`会输出所有依赖。
- 语义化：`yarn`明令比 `npm`命令更加语义化

  ```
  npm install === yarn 
  npm install taco === yarn add taco
  npm uninstall taco === yarn remove taco
  npm install taco --save-dev === yarn add taco --dev
  npm update === yarn upgrade
  ```


# 怎么做

1. **安装 Yarn** ：可以通过 npm 安装 Yarn：

   ```bash
   npm install --global yarn
   ```
2. **初始化项目** ：在项目目录中运行以下命令来创建 `package.json` 文件：

   ```bash
   yarn init
   ```
3. **安装依赖** ：

   安装一个包（如 lodash）：

   ```bash
   yarn add lodash
   yarn add lodash@[version]
   yarn add lodash@[tag]
   ```

   分别添加到 `devDependencies`、`peerDependencies` 和 `optionalDependencies`：

   ```bash
   yarn add lodash --dev
   yarn add lodash --peer
   yarn add lodash --optional
   ```
4. **卸载依赖** ：

   ```bash
   yarn remove lodash
   ```
5. **运行脚本** ：在 `package.json` 中定义脚本后，可以通过以下命令运行：

   ```bash
   yarn run script-name
   ```
6. **更新依赖** ：

   ```bash
   yarn upgrade
   yarn upgrade [package]
   yarn upgrade [package]@[version]
   yarn upgrade [package]@[tag]
   ```
7. **安装项目的全部依赖**

   ```bash
   yarn
   ```

# 参考文档

[yarn官网](https://classic.yarnpkg.com/zh-Hans/)
