### 块级作用域

`let`，`const`所声明的变量，只在`let`，`const`命令所在的代码块内有效。

`var`没有块级作用域

```javascript
{
  let a = 10;
  var b = 1;
}
a // ReferenceError: a is not defined.
b // 1
```
### for循环

闭包有一个缺陷：**闭包只能取到包含函数中任何变量的最后一个值**

```javascript
var a = [];
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 10

var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 6
```

区别：

var：i是全局变量，包含在全局作用域中，闭包取得的也是i最后被修改的值

let：每一次循环都会在一个**新的作用域**中**重新声明一个i变量**，并且在**上一轮的基础**上对i进行**初始化**

✨特别：设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域。

```javascript
for (let i = 0; i < 3; i++) {
  let i = 'abc';
  console.log(i);
}
// abc
// abc
// abc
```
### 变量提升

`let`和`const`没有变量提升

- var 在声明之前可以被访问，值为`undefined`。
- let、const在声明之前访问会报错（包括typeof）
### 暂时性死区

💥注意：暂时性死区和没有变量提升不是一个东西

“暂时性死区”（**TDZ**）是指：如果块级作用域中用`let`和`const`命令声明的变量会**绑定作用域**，让作用域变成封闭作用域。封闭作用域中，凡是在声明之前就使用这些变量（包括`typeof`），就会报错。

```javascript
var tmp = 123;

if (true) {
  tmp = 'abc'; // ReferenceError
  let tmp;
}
```

### 重复声明

`let`，`const`不允许在相同作用域内，重复声明同一个变量。

`var`可多次声明

`const`一旦声明变量，就必须立即初始化

💥注意：`const`实际上保证的是变量指向的栈内存所保存的数据不得改动。简单类型的数据值就保存在栈内存中，但引用类型的栈内存内保存的只是一个指向堆内存中实际数据的指针，`const`只能保证这个指针是固定的，不能保证堆内存中的数据结构不变。

解决方法：`Object.freeze`递归，冻结对象。

```javascript
const foo = {};

// 为 foo 添加一个属性，可以成功
foo.prop = 123;
foo.prop // 123

// 将 foo 指向另一个对象，就会报错
foo = {}; // TypeError: "foo" is read-only
```

### 全局声明区别

- 全局`var`声明，成为`Global(window)`属性。具有覆盖性
- `let const`声明的变量保存在名为Script的作用域中