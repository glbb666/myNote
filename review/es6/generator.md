### 基本概念

Generator 函数是一个状态机，封装了多个内部状态。执行 Generator 函数会返回一个遍历器对象（一个指向内部状态的指针对象）。 返回的遍历器对象，可以依次遍历 Generator 函数内部的每一个状态。

特征：

- `function`关键字与函数名之间有一个星号
- 函数体内部使用`yield`表达式，定义不同的状态（`yield`在英语里的意思就是“产出”）。

实例

```javascript
function* helloWorldGenerator() {
  yield 'hello';
  yield 'world';
  return 'ending';
}

var hw = helloWorldGenerator();

hw.next()
// { value: 'hello', done: false }

hw.next()
// { value: 'world', done: false }

hw.next()
// { value: 'ending', done: true }

hw.next()
// { value: undefined, done: true }
```

### yield 表达式

由于 Generator 函数返回的遍历器对象，**只有调用`next`方法才会遍历下一个内部状态**。`yield`表达式就是**暂停标志**。

遍历器对象的`next`方法的运行逻辑如下。

（1）遇到`yield`表达式，就暂停执行后面的操作，并将紧跟在`yield`后面的那个表达式的值，作为返回的对象的`value`属性值。

（2）下一次调用`next`方法时，再继续往下执行，直到遇到下一个`yield`表达式。

（3）如果没有再遇到新的`yield`表达式，就一直运行到函数结束，直到`return`语句为止，并将`return`语句后面的表达式的值，作为返回的对象的`value`属性值。

（4）如果该函数没有`return`语句，则返回的对象的`value`属性值为`undefined`。

⚠️注意

1. 一个对象的`Symbol.iterator`方法，等于该对象的遍历器生成函数，调用该函数会返回该对象的一个遍历器对象。
2. `yield`表达式只能用在 Generator 函数里面，用在其他地方都会报错。
3. `yield`表达式如果用在**另一个表达式之中**，必须放在圆括号里面。

```javascript
 	console.log('Hello' + yield); // SyntaxError
  console.log('Hello' + (yield)); // OK
```

4. `yield`表达式用作函数参数或放在**赋值表达式的右边**，可以不加括号。

```javascript
function* demo() {
  foo(yield 'a', yield 'b'); // OK
  let input = yield; // OK
}
```

### next 方法的参数 

`yield`表达式本身没有返回值，或者说总是返回`undefined`。`next`方法可以带一个参数，该参数**就会被当作上一个`yield`表达式的返回值。**

```javascript
function* dataConsumer() {
  console.log('Started');
  console.log(`1. ${yield}`);
  console.log(`2. ${yield}`);
  return 'result';
}
let genObj = dataConsumer();
genObj.next();
// Started
genObj.next('a')
// 1. a
genObj.next('b')
// 2. b
```

### for...of 循环

`for...of`循环可以自动遍历 Generator 函数运行时生成的`Iterator`对象，且此时不再需要调用`next`方法。

```javascript
function* foo() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  return 5;
}

for (let v of foo()) {
  console.log(v);
}
// 1 2 3 4
```

一旦next方法的返回对象的done属性为true，for...of循环就会中止，且不包含该返回对象，所以上面代码的return语句返回的5，不包括在for...of循环之中。

#### yield *

用于在`generator`函数的内部执行另一个`generator`函数。解决了`genertor`函数嵌套需要手动遍历的问题。

```javascript
function* foo() {
  yield 'a';
  yield 'b';
}

function* bar() {
  yield 'x';
  yield* foo();
  yield 'y';
}
// 等同于
function* bar() {
  yield 'x';
  yield 'a';
  yield 'b';
  yield 'y';
}
// 等同于
function* bar() {
  yield 'x';
  for (let v of foo()) {
    yield v;
  }
  yield 'y';
}

for (let v of bar()){
  console.log(v);
}
// "x"
// "a"
// "b"
// "y"
```



