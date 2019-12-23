# this指向
- 全局函数中的this指向window对象

- 当函数作为实例方法调用时，函数中的this指向实例

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

这是因为，在非严格模式下，在全局作用域声明的变量会成为window的属性。函数是在全局作用域调用的，所以函数内部的this指向window

- 构造函数内部的this指向创建出的实例

- 使用了call，apply，bind等方法的函数的this指向通过这些方法绑定的this

   💥注意：bind方法返回的函数如果使用new，通过bind绑定的this失效，因为new会将this指向初始化为构造函数创建出的实例

- 箭头函数内部的this

  箭头函数没有`this`，因为它没有原型。它的`this`就是**定义时<font color='red'>所在的对象</font>**，而不是使用时所在的对象。箭头函数不可以当作构造函数，即不可以使用new命令， 不存在arguments对象。如果要用，可以用 rest 参数代替。 不可以使用yield命令，因此箭头函数不能用作 Generator 函数。

  ```javascript
  var name = 'global'
  var obj = {
      name:'obj',
      a:()=>{
          console.log(this.name)
      }
  }
  obj.a();
  var fn = obj.a;
  fn();
  ```


# 改变this指向的方法

## call和apply

`call（）`和`apply（）`的用途是给函数传参，扩充作用域

`apply`接受两个参数：

- 在其中运行函数的作用域(this)

- 参数数组，可以是Array的实例，也可以是arguments对象

`call`接受一到多个参数：

- 在其中运行函数的作用域(this)

- 传递给函数的参数，逐个列举

用`call()`或`apply()`来扩充作用域的最大好处，就是对象不需要与方法有任何的耦合关系。

💛`call`比`apply`的性能要好，`call`传入参数的格式正是函数内部所需要的格式

## bind

`bind`这个方法会创建一个函数的实例，其this值会被绑定到传给bind()函数的值

💥注意：

- 参数可以通过bind和创建出的函数分两次传入
- 如果对创建出的函数调用new，通过bind传入的this会失效，被重新初始化为构造函数创建出的实例对象

### 多次bind,上下文是什么？

```javascript
let a = {}
let fn = function () { console.log(this) }
fn.bind().bind(a)()//window
```

不管我们给函数 bind 几次，函数中的 this 永远由第一次 bind 决定，所以结果永远是 window。

### 多个this规则出现，this指向哪里？

优先级如下

**new > bind/call/apply > 隐式绑定（即对象调方法）> 默认绑定（即严格undefined，非严格window）**