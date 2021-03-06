### npm和yarn的异同

`npm`和`yarn`都是管理`js`包的工具

yarn的优势在于

- 下载速度快：

  - 并行安装：`npm`按照队列执行`package`，前面的安装完成才会安装后面的。`yarn`同步执行所有任务，提高了性能。

  - 离线模式：`yarn`有一个全局本地缓存。安装依赖优先从缓存中获取

    ```nginx
    // 查看yarn本地缓存
    yarn cache dir
    ```

    

- 安装版本统一：

  - `yarn`默认会生成`yarn.lock`，记录确切安装的版本号。
    - 新增模块：创建（更新）`yarn.lock`文件
    - 拉取同一个依赖：使用之前的版本。
  - `npm`其实也有这样的文件存在，但是不会默认生成，需要运行`npm shrinkwrap`。

- 更简洁的输出：

  - `yarn`只打必要信息，`npm`会输出所有依赖。

- 语义化：`yarn`明令比`npm`命令更加语义化

  ```
  npm install === yarn 
  npm install taco === yarn add taco
  npm uninstall taco === yarn remove taco
  npm install taco --save-dev === yarn add taco --dev
  npm update === yarn upgrade
  ```


### yarn的使用

**初始化新项目**

```
yarn init
```

**添加依赖包**

```
yarn add [package]
yarn add [package]@[version]
yarn add [package]@[tag]
```

**将依赖项添加到不同依赖项类别**

分别添加到 `devDependencies`、`peerDependencies` 和 `optionalDependencies`：

```
yarn add [package] --dev
yarn add [package] --peer
yarn add [package] --optional
```

**升级依赖包**

```
yarn upgrade [package]
yarn upgrade [package]@[version]
yarn upgrade [package]@[tag]
```

**移除依赖包**

```
yarn remove [package]
```

**安装项目的全部依赖**

```
yarn
```

### 参考文档

[yarn官网](https://classic.yarnpkg.com/zh-Hans/)