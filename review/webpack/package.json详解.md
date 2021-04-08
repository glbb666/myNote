# package.json详解

**目标：你从本文可以学到什么？**

- 1、关于npm
- 2、package.json是什么？
- 3、如何创建package.json
- 4、全局安装和本地安装到底是怎么区分的？
- 5、安装、更新、卸载模块
- 6、package.json中模块的版本`^2.0.0``~2.0.0`等 （安装版本使用到`@`表示什么意思？`npm install jquery@3.4.1`）
- **7、package.json中每个字段的详解**
- **8、版本号version的意义，如，大更新、增加一些功能上线、**修补一些小布丁的上线**，我们改怎么打包版本号？与git如何配合使用**
- 9、版本号的周期、阶段，（-alpha、-beta、-rc都是什么周期的哪个阶段项目，哪些是开发人员打包的项目，哪些是测试人员的项目，上线的项目）
- 10、一些其他的npm命令
- 11、了解package.json与项目直接的关系
- 12、require引入模块的时候是怎么查找模块的

> 在讲解package.json时，必须要先简单介绍一下npm，那npm是什么呢？要怎么安装呢？

## npm

> npm 是前端开发广泛使用的包管理工具， 为您和您的团队打开了整个JavaScript的世界。 来自各大洲的开源开发人员使用npm来共享和借用包，许多组织也使用npm来管理私有开发。

**1、npm是什么？**

官方介绍：

> npm makes it easy for JavaScript developers to share and reuse code, and it makes it easy to update the code that you’re sharing.

ps： 官方网站 [www.npmjs.com/](https://www.npmjs.com/) 浏览、搜索、下载安装想要的别人上传的模块，也可以直接在命令行中 search 一下。

我们程序开发中常常需要依赖别人提供的框架，我们可以看到一些大型的vue、react框架中也是有依赖别人的模块或者说是包，这样可以提高效率，避免重复造轮子，且往往这些轮子用户体验也必比较好。一个包或者说模块都会有一个package.json文件用来描述这个包。

使用`npm`，我们可以更方便查看依赖这些模块的更新。

**npm都干什么了呢？**

`npm`由三个不同的组件组成：

- 网站： [www.npmjs.com/](https://www.npmjs.com/)
- 命令行界面（CLI）：开发者使用它来管理、安装、更新、发布模块；
- 注册表：保存着人们分享的js模块的大数据库

**2、安装npm**

从node.js站点安装npm

第一步：安装node.js和npm

在这里不具体讲解安装过程了，网上有很多。 npm 是依附于 node.js 的，我们可以去它的官网 [nodejs.org/en/download…](https://nodejs.org/en/download/) 下载安装 node.js。

第二步：测试您的安装

安装后，运行`node -v`。没有报错，出现版本号为成功。安装node.js是，会自动安装npm。同样运行`npm -v`，可查看版本号。



![img](images/1714490246f24fa1)



第三步：如果你需要更新npm

npm 更新地可比 node 勤快多了，因此你下载的 node 附带的 npm 版本可能不是最新的，你可以使用如下命令下载最新 npm:

```
npm install npm@latest -g
复制代码
```

在这讲解一下这句话的意思

- install：就表示是要安装；
- `npm@latest`： 使用 `@` 的格式，表示作用域，指定版本，latest表示最新模板；我们在下载其他模块时也是这个格式。
- -g：表示全局安装

安装 -g 是全局安装，可以在任意命令下使用,如：

```
npm i http-server -g
复制代码
```

\> 我们在命令行中`http-server`可以在命令中使用,是因为我们配置了环境变量.

/usr/local/bin/http-server -> /usr/local/lb/node_modules/http-server/bin/http-server

## package.json

> 管理本地安装的npm包的最佳方法是创建一个 `package.json`文件。

### 了解package.json

每个项目（npm上下载的包，或者其他的nodejs项目）的根目录下面，一般都有一个package.json文件， 定义了这个项目所需要的各种模块，以及项目的配置信息（比如名称、版本、许可证、如何启动项目、运行脚步等元数据）。`npm install`命令根据这个配置文件，自动下载所需的模块。

`package.json`文件就是一个JSON对象，该对象的每一个成员就是当前项目的一项设置。比如`name`就是项目名称，`version`是版本（遵守“大版本.次要版本.小版本”的格式）。还会在项目的生命周期中扮演多个角色、开发、测试、上线版本。

### package.json的作用

- 作为一个描述文件，描述了你的项目所依赖的包
- 允许我们使用 “语义化版本规则”（后面介绍）指明你项目依赖包的版本
- 让你的构建更好地与其他开发者分享，便于重复使用

### package.json如何创建

**1、使用命令行工具客户端CLI**

```
npm init
```

这将启动命令行调查问卷，该调查问卷将`package.json`在您启动命令的目录中创建.



![img](images/17144905f4316bd5)



2、创建默认值

要获取默认值`package.json`，请`npm init`使用`--yes` or `-y`标志运行：

```
 npm init -y
复制代码
```

此方法将`package.json`使用从当前目录中提取的信息生成默认值, 跳过回答问题步骤 。



![img](images/1714490860602216)



3、手动创建

直接在项目根目录新建一个 package.json 文件，然后输入相关的内容。 具体请查看package.json的注意事项。

### package.json文件常见字段详解

1、 name

> 必须字段 ，当前模块\包名称， 长度必须小于等于214个字符，不能以"."(点)或者"_"(下划线)开头，不能包含大写字母。

这个名字可能会作为参数被传入require()，所以它应该比较短，但也要意义清晰。

**2、 version**

> 必须字段 ，当前包的版本号，初次建立默认为`1.0.0`。

version必须可以被npm依赖的一个node-semver模块解析 。定义了当前项目的版本迭代进度。 （遵守“大版本.次要版本.小版本”的格式）

可能现在很多小伙伴们没有注意或者不在乎版本号，更多使用产品的版本号，或者git hashcode方式。

3、 description

> 可选字段，必须是字符串。当前包的描述信息，是一个字符串。它可以帮助人们在使用npm search时找到这个包。

如果 package.json 中没有 `description` 信息，npm 使用项目中的 README.md 的第一行作为描述信息。这个描述信息有助于别人搜索你的项目，因此建议好好写 `description` 信息。

4、 main

> 可选字段， 指定了项目加载的入口文件。

这个字段的默认值是模块根目录下面的`index.js`。

5、 scripts

> 可选字段，`scripts`是一个由脚本命令组成的hash对象，他们在包不同的生命周期中被执行。key是生命周期事件，value是要运行的命令。 指定了运行脚本命令的npm命令行缩写，比如start指定了运行npm run start时，所要执行的命令。我们可以自定义我们想要的运行脚步命令。

参考： [www.ruanyifeng.com/blog/2016/1…](http://www.ruanyifeng.com/blog/2016/10/npm_scripts.html)

**scripts配置执行的脚步**

- 1）执行命令 echo xxx
- 1. 执行node_modules/.bin 下的文件

**为什么可以执行呢？**

当我执行 npm run的时候，就会自动新建一个 Shell，在这个 Shell 里面执行指定的脚本命令。因此，只要是 Shell（一般是 Bash）可以运行的命令，就可以写在 npm 脚本里面。会把当前目录下的node_modules/.bin也拷贝到当前的系统的path（只是临时拷贝，执行结束后，在将PATH变量恢复原样）， 所以当前目录的node_modules/.bin子目录里面的所有脚本，都可以直接用脚本名调用，而不必加上路径。

例如：

我们用node执行一个node.js服务是，`node + 文件` 可以用`node server.js`; 我们也可以用webpack的打包前端文件，`webpack-dev-server`,当然webpack 和webpack-dev-server是要安装依赖的模块；

```
"scripts": {   
    "build": "webpack --mode=development",
    "dev": "webpack-dev-server --mode=development --contentBase=./dist",
    "server":"node app.js"
  }
复制代码
```

我们在命令行工具中输入`npm run server` ，就会调用node app.js帮我们运行。

**简写形式：**

```
npm start是npm run start
npm stop是npm run stop的简写
npm test是npm run test的简写
npm restart是npm run stop && npm run restart && npm run start的简写
复制代码
```

**常用脚本** -----网上收集转

```
// 删除目录
"clean": "rimraf dist/*",

// 本地搭建一个 HTTP 服务
"serve": "http-server -p 9090 dist/",

// 打开浏览器
"open:dev": "opener http://localhost:9090",

// 实时刷新
 "livereload": "live-reload --port 9091 dist/",

// 构建 HTML 文件
"build:html": "jade index.jade > dist/index.html",

// 只要 CSS 文件有变动，就重新执行构建
"watch:css": "watch 'npm run build:css' assets/styles/",

// 只要 HTML 文件有变动，就重新执行构建
"watch:html": "watch 'npm run build:html' assets/html",

// 部署到 Amazon S3
"deploy:prod": "s3-cli sync ./dist/ s3://example-com/prod-site/",

// 构建 favicon
"build:favicon": "node scripts/favicon.js",

"start": "cross-env NODE_ENV=production node server/index.js",

复制代码
```

6、 dependencies、devDependencies

> 可选字段， `dependencies`字段指定了项目运行所依赖的模块 ， `devDependencies`指定项目开发所需要的模块 。

值指向一个对象。该对象的各个成员，分别由模块名和对应的版本要求组成，表示依赖的模块及其版本范围。

默认创建的package.json没有，当我们安装`npm install`一个模块时就会生成。

```
npm install express
npm install express --save
npm install express --save-dev
复制代码
```

上面代码表示单独安装express模块，

- 后边没有参数时，表示安装到`dependencies`属性，
- `--save`参数表示将该模块写入`dependencies`属性，
- `--save-dev`表示将该模块写入`devDependencies`属性。

7、bundledDependencies

> 可选字段，发布包时同时打包的其他依赖。

8、 peerDependencies

> 可选字段，兼容性依赖，如果你的项目或者模块，同时依赖另一个模块，但是所依赖的版本不一样。

比如，你的项目依赖A模块和B模块的1.0版，而A模块本身又依赖B模块的2.0版。

```
{
  "name": "chai-as-promised",
  "peerDependencies": {
    "chai": "1.x"
  }
}
复制代码
```

上面代码指定，安装`chai-as-promised`模块时，主程序`chai`必须一起安装，而且`chai`的版本必须是`1.x`。如果你的项目指定的依赖是`chai`的2.0版本，就会报错。

9、 bin

> 可选字段，bin字段用来指定各个内部命令对应的可执行文件的位置。

例如： 之前我在写创建一个自己的脚手架时，也用到了这个。 [juejin.im/post/684490…](https://juejin.im/post/6844904045845577742#heading-9)

具体使用可查看，这篇文章的第三步，工程创建。

在项目根目录创建/bin/www文件

```
#! /usr/bin/env node
复制代码
```

package.json中配置

```
"bin":{
  "lee-cli":"./bin/www"
}
复制代码
```

`npm link` 将package中的属性bin的值路径添加全局链接 创建快捷方式连接

在命令行中执行`lee-cli`就会执行bin/www文件。过程是：

在上面的例子中，www会建立符号链接`node_modules/.bin/www`。由于`node_modules/.bin/`目录会在运行时加入系统的PATH变量，因此在运行npm时，就可以不带路径，直接通过命令来调用这些脚本。

10、 config

> config字段用于向环境变量输出值

```
{
  "name" : "package",
  "config" : { "port" : "8080" },
  "scripts" : { "start" : "node server.js" }
}
复制代码
```

如果想改变我们可以使用

```
npm config set package:port 80
复制代码
```

11、engines

> 可选字段，指明了该模块运行的平台版本，比如 Node 的某个版本或者浏览器， 也可以指定适用的`npm`版本 。

```
"engines" : { 
"node" : 
">=0.10.3 <0.12"
} 
复制代码
```

12、license

> 可选字段， 表示定义适用于package.json所描述代码的许可证。不同的协议有不同的限制。让用户知道他们有何权限来使用你的模块，以及使用该模块有哪些限制。

可参考： [choosealicense.com/](https://choosealicense.com/) 选择许可证。

如：MIT：最大许可，别人下载你的代码可以改你的代码,默认安装值。

13、author

> 可选字段，项目开发者。

14、private

> 可选字段，布尔值，是否私有，设置为 true 时，npm 拒绝发布。

这是防止私有包被以外发布的一种方法。如果你希望包装某个包只能被发布到特定的一个registry中(比如，一个内部的registry)，则可以使用下面的publishConfig字典来描述以在publish-time重写registry配置参数。

15、keywords

> 可选字段，项目关键字，是一个字符串数组。它可以帮助人们在使用npm search时找到这个包。

16、os

> 可选字段，指定模块可以在什么操作系统上运行

17、style

> style指定供浏览器使用时，样式文件所在的位置。

18、repository

> 包代码存放的地方的类型，可以是 git 或 svn，git 可在 Github 上

19、homepage

> 可选字段，没有[http://](http://blog.csdn.net/woxueliuyun/article/details/39294375)等带协议前缀的URL。

**版本问题：**

> version:"1.0.0"
>
> 1.0.0:
>
> 第一位改变表示：不兼容老代码,大规模的更新，新版本发布；
>
> 第二位表示：增加了一些功能，向下兼容；
>
> 第三位表示：小的补丁，bug修改；

**我们发表项目的时候，一盘使用npm + git**

- 使用`npm version patch` (patch打补丁) 这种会改变版本的第三位；使用`git tag` 执行即会自动在git上版本号
- 使用`npm version minor`这种改变的版本号的第二位；同步git版本；
- 使用`npm version major` 这种改变版本号的第一位；同步git版本；

```
npm version [<newversion> | major | minor | patch | premajor | preminor | prepatch | prerelease | from-git]

major：主版本号

minor：次版本号

patch：补丁号

premajor：预备主版本

prepatch：预备次版本

prerelease：预发布版本
```

ps：注意，如果报错：Git working directory not clean 是说明你现在需要`git status`是clean的。

```
git add .
git commit -m"package.json详解  "
npm versin monir -m"增加版本号"
git push -u origin master
```

**如何制定规则？**

作为使用者，我们可以在 package.json 文件中写明我们可以接受这个包的更新程度（假设当前依赖的是 1.2.4 版本）：

- 如果只打算接受补丁版本的更新（也就是最后一位的改变），就可以这么写：

  ```
   1.2
   1.2.x
   ~1.2.4
  ```
  
- 如果接受小版本的更新（第二位的改变），就可以这么写：

  ```
  1
  1.x
  ^1.2.4
  ```
  
- 如果可以接受大版本的更新（自然接受小版本和补丁版本的改变），就可以这么写：

  ```
  *
  x
  ```
  
小结一下：总共三种版本变化类型，接受依赖包哪种类型的更新，就把版本号准确写到前一位。

**版本周期、阶段：**

- α -Alpha 第一阶段 一般只供内测使用
- β -Beta 第二阶段 已经消除了软件中大部分的不完善之处，但是仍有可能还存在缺陷和漏洞，一般提供给特定的用户群里来测试使用；
- γ - RC 是第三阶段，此时产品已经相当成熟，只需要在个别地方在做进一步的优化处理即可上市发布

例如：

`2.1.0-beta.1` 一般这样用户不会安装这种的，这种可以用内侧、测试人员使用。

### 依赖包的版本问题

| 实例   | 说明                                               |
| ------ | -------------------------------------------------- |
| ~1.2.3 | 主版本+次要版本+补丁版本;1.2.3 <= version < 1.3.0; |
| ~1.2   | 主版本+次要版本;1.2.0 <= version < 1.3.0           |
| ~1     | 主版本;1.0.0 <= version < 2.0.0                    |

 

| 符号                      | 实例           | 版本范围                       | 说明                                                         |
| ------------------------- | -------------- | ------------------------------ | ------------------------------------------------------------ |
|                           | 1.0.0          | 1.0.0                          | 锁定1.0.0版本，必须这个版本。                                |
| ^会匹配最新的大版本依赖包 | ^1.2.3、^0.2.3 | >=1.2.3 <2.0.0、>=0.2.3 <0.3.0 | 表示安装1.x.x的最新版本（不低于1.2.3，包括1.3.0），但是不安装2.x.x，也就是说安装时不改变大版本号。需要注意的是，如果大版本号为0，则插入号的行为与波浪号相同，这是因为此时处于开发阶段，即使是次要版本号变动，也可能带来程序的不兼容。(主版本) |
| ~会匹配最近的小版本依赖包 | ~1.2.3         | >=1.2.3 <1.3.0                 | 表示安装1.2.x的最新版本（不低于1.2.3），但是不安装1.3.x，也就是说安装时不改变大版本号和次要版本号。 |
| >=                        | >=2.1.0        | >=2.1.0                        | 大于等于2.1.0                                                |
| <=                        | <=2.0.0        | <=2.0.0                        | 小于等于2.0.0                                                |
| laster                    |                |                                | 安装最新的版本                                               |
| *                         |                | >=0.0.0                        | 任何版本                                                     |
| -                         | 1.2.3 - 2.3.4  | >=1.2.3 <=2.3.4                |                                                              |

### 区分安装`devDependencies`和`dependencies`？

`devDependencies`是开发所需要的模块，所以我们可以在开发过程中需要的安装上去，来提高我们的开发效率，比如一些知名的第三方库， `webpack`、`rollUp`、`less`、`babel`这些。 就没必要在生产环境安装。

以下类库都建议安装到`devDependencies`:

- 单元测试支撑（mocha、chai）；
- 语法兼容（babel）；
- 语法转换（jsx to js、coffeescript to js、typescript to js）
- 程序构建与优化（webpack、gulp、grunt、uglifyJS）；
- css 处理器（postCSS、SCSS、Stylus）；
- 代码规范（eslint）；

### 依赖包（指定、更新、本地、使用、卸载）

1、安装本地依赖包

```
npm install jquery
```

这个命令会在当前目录创建一个 `node_modules` 目录，然后下载我们指定的包到这个目录中。

2、指定安装版本，可以在package name后`@版本号`。

如果包的名称以包开头`@`，则它是一个**范围包**。

```
npm install jquery@3.4.1
npm install jquery@">=1.1.0 <2.2.0"
npm install jquery@latest
```

更新之后，dependencies内的版本号也会改变。

3、更新依赖包

```
npm update jquery
```

4、使用包

```
let jquery = require('jquery');
<script src="/node_modules/jquery/dist/jquery.js">//这个需要注意路径</script> 
```

6、卸载依赖包

```
npm uninstall jquery
```

### Semantic versioning（语义化版本规则）

[docs.npmjs.com/about-seman…](https://docs.npmjs.com/about-semantic-versioning)

[github.com/npm/node-se…](https://github.com/npm/node-semver)

### package.json注意事项

根据上边我们在使用`npm init`会询问我们填几项内容，有的可以不填，有的必须填，这些必填都是一个`package.json`内容必须要具备的字段：`name`和`version`,如果没有，无法执行`install`

- name:全是小写，没有空格,循序使用连字符和下划线
- version: 版本号,遵循上边说的语义化版本规则`x.x.x`

**其他注意事项：**

- 添加中文注释会编译出错

# 延伸

## 其他

1、我们经常会在安装一半退出在继续安装会报错，这是因为有缓存的原因。

```
npm cache clean --force
```

2、npx

npx可以直接执行node_modules/.bin文件 不需要在去配置scripts

如果模块不存在可以安装，安装完有后会自己销毁，避免安装全局模块

**参考资料：**

[cloud.tencent.com/developer/s…](https://cloud.tencent.com/developer/section/1490235)

[docs.npmjs.com/about-seman…](https://docs.npmjs.com/about-semantic-versioning)

[semver.org/lang/zh-CN/](https://semver.org/lang/zh-CN/)

[docs.npmjs.com/misc/semver…](https://docs.npmjs.com/misc/semver#advanced-range-syntax)

[www.ruanyifeng.com/blog/2016/1…](http://www.ruanyifeng.com/blog/2016/10/npm_scripts.html)

