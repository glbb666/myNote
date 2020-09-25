#### `AMD`

前端模块采用同步方式来引入必会造成`UI`初始化过长，不利于用户体验

`AMD`是“异步模块定义” ，用于浏览器，需要在声明的时候指定所有的依赖，通过形参传递依赖到模块内容中。

#### `CMD`

**用于服务器，把模块作为对象，整体引入**，输入时必须查找对象属性，因为只有运行时才能得到这个对象，导致完全没办法在编译时做“静态优化”。

`CommonJS` 模块的输出都定义在`module.exports`这个属性上面。`Node` 的`import`命令加载 CommonJS 模块，Node 会自动将`module.exports`属性，当作模块的默认输出，即等同于`export default xxx`。

```javascript
module.exports = {
  foo: 'hello',
  bar: 'world'
};
// 等同于
export default {
  foo: 'hello',
  bar: 'world'
};
```

- 输入：用`require`把模块作为对象整体引入(即加载模块内的所有方法)。

```javascript
let { stat, exists, readFile } = require('fs');
```
- 输出：

#### `esModule`

`esModule`是浏览器和服务器通用的模块方案。它的设计思想是尽量的静态化，**使得编译时就能确定模块的依赖关系，以及输入和输出的变量，完成模块的加载。**

`export`命令用于规定模块的对外接口，在接口名与模块内部变量之间，必须建立一一对应的关系。

```javascript
// 报错
var m = 1;
export m;
// 写法一
export var m = 1;
// 写法二
var m = 1;
export {m};
//写法三,给输出变量重命名
var n = 1;
export {n as m};
```

`import`命令用于输入其他模块提供的功能。`import`命令输入的变量都是只读的，不允许在加载模块的脚本里面，改写接口。

```javascript
import { stat, exists, readFile } from 'fs';
import { lastName as surname } from './profile.js';//用as将输入的变量重命名
import {a} from './xxx.js'
a = {}; //不允许改写接口
a.foo = 'hello'; // 但是如果接口是一个对象,可以改写它的属性
import * as circle from './circle';//整体加载模块的值
//模块整体加载所在的那个对象（circle），应该是可以静态分析的，所以不允许运行时改变。
//下面是不允许的
circle.foo = 'hello';
```

`export`和`import`命令都不能放在块级作用域内，否则就没有办法做静态优化了

`import`具有提升效果，会提升到整个模块的头部，首先执行

如果多次重复执行同一句`import`语句，那么只会执行一次，而不会执行多次，因为`import`是单例模式

`export default`命令其实只是输出一个叫做`default`的默认变量，所以它后面不能跟变量声明语句。一个模块只能有一个默认输出，因此`export default`命令只能使用一次。`import`命令可以为该匿名函数指定任意名字。

```javascript
export default function () {
  console.log('foo');
}
import customName from './export-default';//import后不加{}，因为只唯一对应export default
// 错误
export default var a = 1;
// 正确
var a = 1;
export default a;//将变量a的值赋给变量default
//export default也可以用来输出类[
export default class { ... }
```
#### 浏览器加载和`node`加载

浏览器加载` ES6 `模块，也使用`<script>`标签，但是要加入`type="module"`属性。


```html
<script type="module" src="./foo.js"></script>//等同于打开了<script>标签的defer属性
<script type="module" src="./foo.js" async></script>//按async的规则 
```

Node 有自己的 `CommonJS `模块格式，与 `ES6 `模块格式是不兼容的。

Node 要求 `ES6` 模块采用`.mjs`后缀文件名。`import`或者`export`命令，那么就必须采用`.mjs`后缀名。`require`命令不能用来加载或者使用在`.mjs`文件中，`import`可以。

## `commonJS `与` ESmodule`差异

- `commonJs`是被加载的时候运行，`esModule`是编译的时候运行，效率要比` CommonJS` 高，但是没法引用` ES6` 模块本身，因为它不是对象。
- `esModule`输出接口与模块内的值动态绑定，所以可以取到模块内部实时的值。`commentJS `在第一次被加载时，会完整运行整个文件并输出一个对象的浅拷贝，缓存在内存中。下次加载文件时，直接从内存中取值。