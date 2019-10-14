## 基本类型

js有**七种**（5+2）基本数据类型

- `Null`:只包含一个值`null`，表示一个空对象指针，用`typeof`检测的时候返回一个`"object"`
- `Undefined`:只包含一个值`undefined`
- `Boolean`:包含两个值`true`和`false`
- `Number`:整数或浮点数，还包括特殊值（`-Infinity`、`+Infinity`、`NaN`）
- `String`:一串表示文本值的字符序列
- `Symbol`：一种实例是唯一且不可改变的数据类型
- `BigInt`:es10新增
## 对象类型
`Object`:除了常用的Object，**Array、Function等都属于特殊的对象**

## 区别
### 基本类型
- 存储在**栈**中
- 大小固定，不可变性
- 做函数参数传递时，原始类型将值本身赋给局部变量。
```js
let str='123';
str+='6'
console.log(str)//1236
//str确实变了，但实际上他会开辟新的内存进行存储
```
- 比较时比较值，若值相等==》true
#### 栈
- 存储的值大小固定
- 空间较小
- 可以直接操作其保存的变量，运行效率高
- 由系统自动分配存储空间
### 引用类型

- `值`存储在**堆**中，`栈`中`存储固定长度地址`，指向堆中
- 引用的比较，**比较地址**，若地址同==》true，否则false
>ECMAScript中所有函数都是按值传递的。
- 做函数参数传递时，引用类型复制的是**指向堆的地址**。 
#### 堆

- 存储的值大小不定，可动态调整
- 空间较大，运行效率低
- 无法直接操作其内部存储，使用引用地址读取
- 通过代码进行分配空间


## null 和 undefined
- null**转数字为 0** ，对象赋值为null表示空
- undefined，表示“缺少值”，即此处应有一个值，但还没有定义。**转数字为NaN**。

## Symbol
### 独一无二

直接使用Symbol()创建新的symbol变量，可选用一个字符串用于描述。当参数为对象时，将调用对象的toString()方法。
```js
var sym1 = Symbol();  // Symbol() 
var sym2 = Symbol('ConardLi');  // Symbol(ConardLi)
var sym3 = Symbol('ConardLi');  // Symbol(ConardLi)
var sym4 = Symbol({name:'ConardLi'}); // Symbol([object Object])
console.log(sym2 === sym3);  // false
```
相同字符串创建symbol并不相等。
如果我们想创造两个相等的Symbol变量，可以使用Symbol.for(key)。

```js
var sym5 = Symbor.for('ConardLi');
//这里是描述符为'CornardLi'的Symbol第一次在全局注册表中进行注册
console.log(sym2 === sym5)	//false
console.log(sym3 === sym5)	//false
var sym6 = Symbol.for('CornardLi');
console.log(sym5 === sym6);	//true
```

> 使用给定的key搜索现有的symbol，如果找到则返回该symbol。否则将使用给定的key在全局symbol注册表中创建一个新的symbol。
### 原始类型

注意是使用Symbol()函数创建symbol变量，并非使用构造函数，使用new操作符会直接报错。

我们可以使用typeof运算符判断一个Symbol类型：
```
typeof Symbol() === 'symbol'
typeof Symbol('ConardLi') === 'symbol'
```
### 不可枚举
不可使用`for in`枚举属性，可用`Object.getOwnPropertySymbols()`获取Symbol属性
````js
var obj = {
  name:'ConardLi',
  [Symbol('name2')]:'code秘密花园'
}
Object.getOwnPropertyNames(obj); // ["name"]
Object.keys(obj); // ["name"]
for (var i in obj) {
   console.log(i); // name
}
Object.getOwnPropertySymbols(obj) // [Symbol(name)]
````
### 应用

#### 私有属性
```js
const privateField = Symbol();
class myClass {
  constructor(){
    this[privateField] = 'ConardLi';
  }
  getField(){
    return this[privateField];
  }
  setField(val){
    this[privateField] = val;
  }
}
```
### 防止属性污染
模拟call
````js
Function.prototype.myCall = function (context) {
  if (typeof this !== 'function') {
    return undefined; // 用于防止 Function.prototype.myCall() 直接调用
  }
  context = context || window;
  const fn = Symbol();
  context[fn] = this;
  const args = [...arguments].slice(1);
  const result = context[fn](...args);
  delete context[fn];
  return result;
}
````
## Number类型
Number 类型存在问题：小数计算不精确。例如：0.1+0.2!==0.3

计算机中所有的数据都是以二进制存储的，所以在计算时计算机要把数据先转换成二进制进行计算，然后在把计算结果转换成十进制。
#### js对二进制小数的存储方式
小数的二进制大多数都是无限循环的。而JS遵循IEEE 754标准。使用64位固定长度来表示。

#### IEEE 754
IEEE 754标准包含一组实数的二进制表示法。它有三部分组成：

- 符号位: 标识正负的，1表示负，0表示正
- 指数位: 存储科学计数法的指数
- 尾数位: 存储科学计数法后的有效数字


JavaScript 使用的是64位双精度浮点数编码，所以它的符号位占1位，指数位占11位，尾数位占52位。

0.1为例

它的二进制为：`0.0001100110011001100...`

为了节省存储空间，在计算机中它是以科学计数法表示的，也就是
`1.100110011001100... X 2-4`

所以我们通常看到的二进制，其实是计算机实际存储的尾数位.

由于限制,有效数字第 53 位及以后的数字是不能存储的，它遵循，如果是1就向前一位进1，如果是0就舍弃的原则。所以0.1会发生精度丢失，0.2也是。导致了`0.1+0.2!=0.3`
## BigInt类型
**BigInt** 是一种内置对象，可以表示大于 2^53 的整数。而在Javascript中，[`Number`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number) 基本类型可以精确表示的最大整数是 253。**BigInt** 可以表示任意大的整数。

可以用在一个整数字面量后面加 `n` 的方式定义一个 `BigInt` ，如：`10n`，或者调用函数`BigInt()`。

>注意， `BigInt()` 不是构造函数，因此不能使用 [`new`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new) 操作符。

它在某些方面类似于 [`Number`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number) ，但是也有几个关键的不同点：不能用与 [`Math`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Math) 对象中的方法；不能和任何 [`Number`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number) 实例混合运算，两者必须转换成同一种类型。在两种类型来回转换时要小心，因为 `BigInt` 变量在转换成 [`Number`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number) 变量时可能会丢失精度。

#### 类型信息

使用 `typeof` 测试时， `BigInt` 对象返回 "bigint" ：

```js
typeof 1n === 'bigint'; // true
typeof BigInt('1') === 'bigint'; // true
```

使用 `Object` 包装后， `BigInt` 被认为时一个普通 "object" ：

```js
typeof Object(1n) === 'object'; // true
```

#### 运算

以下操作符可以和 `BigInt` 一起使用： `+`、``*``、``-``、``**``、``%`` 。除 `>>>` （无符号右移）之外的 [位操作](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators) 也可以支持。因为 BigInt 都是有符号的 `>>>` （无符号右移）不能用于 BigInt。[为了兼容 asm.js ](https://github.com/tc39/proposal-bigint/blob/master/ADVANCED.md#dont-break-asmjs)，.BigInt 不支持单目 (`+`) 运算符。

```js
const previousMaxSafe = BigInt(Number.MAX_SAFE_INTEGER);
// ↪ 9007199254740991n

const maxPlusOne = previousMaxSafe + 1n;
// ↪ 9007199254740992n
 
const theFuture = previousMaxSafe + 2n;
// ↪ 9007199254740993n, this works now!

const multi = previousMaxSafe * 2n;
// ↪ 18014398509481982n

const subtr = multi – 10n;
// ↪ 18014398509481972n

const mod = multi % 10n;
// ↪ 2n

const bigN = 2n ** 54n;
// ↪ 18014398509481984n

bigN * -1n
// ↪ –18014398509481984n
```

`/` 操作符对于整数的运算也没问题。可是因为这些变量是 `BigInt` 而不是 `BigDecimal` ，该操作符结果会向零取整，也就是说不会返回小数部分。

当使用 `BigInt` 时，带小数的运算会被取整。

```js
const expected = 4n / 2n;
// ↪ 2n

const rounded = 5n / 2n;
// ↪ 2n, not 2.5n
```

#### 比较

`BigInt` 和 [`Number`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number) 不是严格相等的，但是宽松相等的。

```js
0n === 0
// ↪ false

0n == 0
// ↪ true
```

[`Number`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number) 和 `BigInt` 可以进行比较。

```js
1n < 2
// ↪ true

2n > 1
// ↪ true

2 > 2
// ↪ false

2n > 2
// ↪ false

2n >= 2
// ↪ true
```

两者也可以混在一个数组内并排序。

```js
const mixed = [4n, 6, -12n, 10, 4, 0, 0n];
// ↪  [4n, 6, -12n, 10, 4, 0, 0n]

mixed.sort();
// ↪ [-12n, 0, 0n, 10, 4n, 4, 6]
```

注意被  `Object` 包装的 `BigInt`s 使用 object 的比较规则进行比较，只用同一个对象在比较时才会相等。

```js
0n === Object(0n); // false
Object(0n) === Object(0n); // false

const o = Object(0n);
o === o // true
```

### 条件[节](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/BigInt#条件)

`BigInt` 在需要转换成 [`Boolean`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean) 的时表现跟 [`Number`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number) 类似：如通过 [`Boolean`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Boolean) 函数转换；用于 [`Logical Operators`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Logical_Operators)  `||`, ``&&``, 和 `!` 的操作数；或者用于在像 [`if statement`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/if...else) 这样的条件语句中。

```js
if (0n) {
  console.log('Hello from the if!');
} else {
  console.log('Hello from the else!');
}

// ↪ "Hello from the else!"

0n || 12n
// ↪ 12n

0n && 12n
// ↪ 0n

Boolean(0n)
// ↪ false

Boolean(12n)
// ↪ true

!12n
// ↪ false

!0n
// ↪ true
```

## 另外的引用类型

平时使用的很多引用类型的变量，并不是由Object构造的，但是它们原型链的终点都是Object，这些类型都属于引用类型
- Array 数组
- Date 日期
- RegExp 正则
- Function 函数
### 包装类型
为了便于操作基本类型值，ECMAScript还提供了几个特殊的引用类型
- String

- Number

- Boolean

  包装类型与原始类型的区别
```js
true === new Boolean(true) // false
123 === new Number(123) // false
'Mike' === new String('Mike') // false
console.log(typeof new String('Mike')) // object
console.log(typeof 'Mike') // string
```
> 主要区别就是对象的生存期，使用new操作符创建的引用类型的实例，在**执行流离开当前作用域之前都一直保存在内存中**，而基本类型则只**存在于一行代码的执行瞬间**，然后立即被销毁，这意味着我们不能在运行时为基本类型添加属性和方法。
### 装箱和拆箱
- 装箱转换：基本类型转换为对应的包装类型
- 拆箱操作：引用类型转换为基本类型

每当我们操作一个基础类型时，后台就会自动创建一个包装类型的对象
```js
const name = 'Mike'
const name2 = name.substring(2)
console.log(name2)//ke
```
发生如下过程
- 创建一个String的包装类型实例
- 在实例上调用substring方法
- 销毁实例

拆箱的过程中，一般会调用引用类型的`valueOf`和`toString`方法，你也可以直接重写toPeimitive方法。

一般转换成不同类型的值遵循的原则不同，例如：

- 引用类型转换为`Number`类型，先调用`valueOf`，再调用`toString`
- 引用类型转换为`String`类型，先调用`toString`，再调用`valueOf`

若valueOf和toString都不存在，或者没有返回基本类型，则抛出TypeError异常。

还可以手动进行拆箱和装箱操作。我们可以直接调用包装类型的valueOf或toString，实现拆箱操作：
```js
var num =new Number("123");  
console.log( typeof num.valueOf() ); //number
console.log( typeof num.toString() ); //string
```