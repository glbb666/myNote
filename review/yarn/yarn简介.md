# 什么是 Yarn

Yarn 是一个 JavaScript 的包管理工具。

# 为什么重要

1. **快速安装** ：Yarn 使用缓存机制，能够加快依赖包的安装速度，特别是在重复安装时。
2. **一致性** ：通过 `yarn.lock` 文件，确保所有环境中的依赖版本一致，避免因版本不一致导致的问题。
3. **离线模式** ：Yarn 支持离线安装，能够从本地缓存中快速安装已下载的包。
4. **并行安装** ：Yarn 同时处理多个包的安装，进一步提高效率。

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
