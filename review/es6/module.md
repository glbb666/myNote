### `esModule`

`esModule`是浏览器和服务器通用的模块方案。它的设计思想是静态化，**使得编译时就能确定模块的依赖关系，以及输入和输出的变量，完成模块的加载。**

#### export导出

##### 用export导出本模块变量

```javascript
// 写法一，导出声明
export var m = 1;
// 写法二，导出模块的部分内容
var m = 1;
export {m};
// 写法三，重命名导出变量
var n = 1;
export {n as m};
```

##### 用export导出非当前模块变量

我们经常会去扩展其它模块，并且只导出那个模块的**部分**内容。 **重新导出功能并不会在当前模块导入那个模块或定义一个新的局部变量。**

ParseIntBasedZipCodeValidator.ts

```typescript
// 写法四：导出非本模块变量
export { ZipCodeValidator } from "./ZipCodeValidator";
// 写法五：导出非本模块变量，并重命名
export { ZipCodeValidator as RegExpBasedZipCodeValidator } from "./ZipCodeValidator";
```

##### 用export导出多个模块

或者一个模块可以包裹多个模块，并把他们导出的内容联合在一起通过语法：`export * from "module"`。

AllValidators.ts

```typescript
export * from "./StringValidator"; // exports interface StringValidator
export * from "./LettersOnlyValidator"; // exports class LettersOnlyValidator
export * from "./ZipCodeValidator";  // exports class ZipCodeValidator
```

我们怎么导入`AllValidators.ts`的导出呢？

首先我们要明白，`AllValidators.ts`把所有导出的内容都放到一个对象中，所以我们有两种导入方式

```typescript
//方式1，导出对象的部分内容
import {StringValidator} from './AllValidators'
//方式2，导出整个对象
import * as All from './AllValidators'
let a:All.StringValidator = {
    isAcceptable: (s)=>!!s
}
```

##### 默认导出

一个模块只能有一个默认导出，可以是基本类型的值，也可以是类和函数

```javascript
// 基本类型的值
export default 123;
export default 'string';
export default false;
....
// 类和函数声明可以直接被标记为默认导出。标记为默认导出的类和函数的名字是可以省略的。
// 函数
export default function (number) {
    return number++;
}
export default class {
    isAcceptable(s: string) {
        return lettersRegexp.test(s);
    }
}
```

##### 总结

在这里我们可以知道，`export`可以导出

- 本模块的变量
- 非本模块的变量
- 多个模块的变量
- 默认导出

注意

- 当没有进行默认导出的时候，`export`导出的变量都会放在对象中，所以使用`import`导入的时候，导入的始终是一个对象。

- 当进行默认导出的时候，导出的就是默认的值。因此默认导出可以不要命名，因为使用`import`导入的时候，会给默认导出重新命名。

#### import导入

`import`命令用于输入其他模块提供的功能。

##### 导入整个模块

```typescript
//整体加载模块的值
import * as person from './person';
//使用整体模块中的某个变量
let Jane = person.createPerson('Jane');
//模块整体加载所在的那个对象（circle），应该是可以静态分析的，所以不允许运行时改变。
//修改整体模块中的某个属性值（不允许）
circle.Benny = 'Benny';
```

##### 导入模块中局部变量

```javascript
//导入局部变量
import {Jane} from 'person'
//用as给局部变量重命名
import { Jane as wife } from 'person'
//修改局部变量的值（不允许）
Jane = {}; 
// 但是如果接口是一个对象,可以改写它的属性
person.Ann  = new Person()
```

#### 导入默认导出

```javascript
import Jane from 'person'
```

##### 总结

在这里我们可以知道，`import`可以导入**非本模块**的

- 整个模块
- 模块中局部变量
- 默认导出

当导入的模块非默认导出时，始终是一个对象。

当导入的模块是默认导出时，是默认导出的值。在导入的时候，会对默认导出的值进行命名。

#### 一些关于import和export的知识点

- `export`和`import`命令都不能放在块级作用域内，否则就没有办法做静态优化了
- `import`具有提升效果，会提升到整个模块的头部，首先执行
- 因为`import`是单例模式，所以如果多次重复执行同一句`import`语句，那么只会执行一次。
- `export default`命令其实只是输出一个叫做`default`的默认变量，所以它后面不能跟变量声明语句。一个模块只能有一个默认输出，因此`export default`命令只能使用一次。`import`命令可以为该匿名函数指定任意名字。

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
### `commonJS`

**用于服务器，把模块作为对象，整体引入**，输入时必须查找对象属性，因为只有运行时才能得到这个对象，导致完全没办法在编译时做“静态优化”。

- 输出：

`CommonJS` 模块的输出都定义在`module.exports`这个属性上面。`Node` 的`import`命令加载` CommonJS `模块，`Node `会自动将`module.exports`属性，当作模块的默认输出，即等同于`export default xxx`。

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

### 浏览器加载和`node`加载

浏览器加载` ES6 `模块，也使用`<script>`标签，但是要加入`type="module"`属性。


```html
<script type="module" src="./foo.js"></script>//等同于打开了script标签的defer属性
<script type="module" src="./foo.js" async></script>//按async的规则 
```

Node 有自己的 `CommonJS `模块格式，与 `ES6 `模块格式是不兼容的。

Node 要求 `ES6` 模块采用`.mjs`后缀文件名。`import`或者`export`命令，那么就必须采用`.mjs`后缀名。`require`命令不能用来加载或者使用在`.mjs`文件中，`import`可以。

### `commonJS `与` ESmodule`差异

- `commonJs`是被加载的时候运行，`esModule`是编译的时候运行，效率要比` CommonJS` 高，但是没法引用` ES6` 模块本身，因为它不是对象。
- `esModule`输出接口与模块内的值动态绑定，所以可以取到模块内部实时的值。`commentJS `在第一次被加载时，会完整运行整个文件并输出一个对象的浅拷贝，缓存在内存中。下次加载文件时，直接从内存中取值。

