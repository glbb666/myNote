# this指向

`this`是运行时绑定的，所以取决于**函数的执行上下文**。

> 当一个函数被调用时，会创建一个执行上下文，this也是这里的一个属性。

## 独立函数调用

全局作用域中，独立函数调用，函数中的 `this`指向 `window`对象。严格模式下为undefined。

```javascript
function foo(){
    console.log(this.a)
}
var a = 2
foo()  // 2
```

## 对象上下文（隐式绑定）

当函数作为对象的方法**调用的那一刻**，函数的执行上下文会隐式绑定 `this`

```javascript
 function foo() { 
    console.log( this.a );
}
var obj = { 
    a: 2,
    foo: foo
};
obj.foo(); // 2
```

 `foo`虽然被定义在全局作用域，但是调用的时候是通过 `obj`上下文引用的，可以理解为在 `foo`调用的那一刻它被 `obj`对象拥有。所以 `this`指向 `obj`。

这里存在两个特殊情况：

### 链式调用

链式调用的情况下只有最后一层才会影响调用位置，例如：

```javascript
obj1.obj2.obj3.fn() //这里的fn中的this指向obj3
```

这里是因为执行上下文在函数调用的时候创建，fn方法是在obj3上调用的，当函数作为obj3的方法**调用的那一刻**，函数的执行上下文会隐式绑定 obj3

### 引式丢失

```javascript
function foo() { 
    console.log( this.a );
}
var obj = { 
    a: 2,
    foo: foo 
};
var bar = obj.foo; // 函数别名!
var a = "xxxxx"
bar(); // xxxxx
```

这里是因为foo函数的执行上下文栈的this取决于函数执行时的绑定方式。

这里的 `bar`其实是引用了 `obj.foo`的地址，这个地址指向的是一个函数。

而bar在调用时符合“独立函数调用”规则。所以它的 `this`不是 `obj`。

稍微改一下上面的代码：

```javascript
function foo() { 
    console.log( this.a );
}
var obj = { 
    a: 2,
    foo: foo 
};
var a = "xxxxx"
setTimeout(obj.foo ,100); // xxxxx
```

我们看到，回调函数虽然是通过 `obj`引用的，但是 `this`也不是 `obj`了。其实内置的 `setTimeout()`函数实现和下面的伪代码类似：

```javascript
function setTimeout(fn, delay){
    //等待delay毫秒
    fn()
}
```

其实这段代码隐藏这一个操作就是 `fn=obj.foo`，这和上面例子中的 `bar=obj.foo`异曲同工

## 显式绑定

指的是通过 `call`、`apply`、`bind`显式地绑定 `this`指向。

见执行上下文&作用域&this/call和apply和bind的差别。

## new绑定

`Js`中 `new`与传统的面向类的语言机制不同，`Js`中的“构造函数”其实和普通函数没有任何区别。

其实当我们使用 `new`来调用函数的时候，发生了下列事情：

- 创建一个全新的对象
- 这个新对象会被执行“原型”链接
- 这个新对象会被绑定到调用的this
- 如果函数没有对象类型的返回值，这个对象会被返回

  其中，第三步绑定了 `this`,所以“构造函数”和原型中的 `this`永远指向 `new`出来的实例。

## 箭头函数内部的 `this`

见函数&闭包/箭头函数

# 优先级

对于普通函数来说，可以按照以下规则（优先级从先到后）进行判断：
- 函数是 `new`出来的，`this`指向实例
- 函数通过 `call`、`apply`、`bind`绑定过，`this`指向绑定的第一个参数
- 函数在某个上下文对象中调用（隐式绑定），`this`指向上下文对象
- 以上都不是，`this`指向全局对象

这些规则对于匿名函数不适用。具体看见函数&闭包/箭头函数

# 严格模式下的差异

以上所说的都是在非严格模式下成立，严格模式下的 `this`指向是有差异的。

- 独立函数调用： `this` 指向 `undefined`
- 其他没有区别

# this指向题目

```javascript
var name = 'global'
var obj = {
    name:'obj',
    a(){
        console.log(this.name)
    }
}
obj.a()//'obj'
//💥注意
var fn = obj.a;
fn();//'global'
```

这是因为，在非严格模式下，在全局作用域声明的变量会成为 `window`的属性。函数是在全局作用域调用的，所以函数内部的 `this`指向 `window`。
