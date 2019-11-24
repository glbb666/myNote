# this指向
- 全局函数中的this指向window对象

```javascript
console.log(this);//Window
```

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
```

  💥注意：有一种情况

  ```javascript
  var name = 'global'
  var obj = {
      name:'obj',
      a(){
          console.log(this.name)
      }
  }
  var fn = obj.a;
  fn();//'global'
  ```

这是因为，在非严格模式下，在全局作用域声明的变量会成为window的属性。函数是在全局作用域调用的，所以函数内部的this指向window

- 构造函数内部的this指向创建出的实例

  ```javascript
  function CreateObj(){
      this.name = 'a'
      console.log(this);//CreateObj {name: "a"}
  }
  var a = new CreateObj();
  ```
  
- 使用了call，apply，bind等方法的函数的this指向通过这些方法绑定的this

   💥注意：bind方法返回的函数如果使用new，通过bind绑定的this失效，因为new会将this指向初始化为构造函数创建出的实例

- 箭头函数内部的this

  箭头函数体内的this对象，就是**定义时<font color='red'>所在的对象</font>**，而不是使用时所在的对象。箭头函数不可以当作构造函数，也就是说，不可以使用new命令，否则会抛出一个错误。 不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替。 不可以使用yield命令，因此箭头函数不能用作 Generator 函数。

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

`call（）`和`apply（）`的用途都是在特定的作用域中调用函数，实际上等于设置函数体内this对象的值

`apply`接受两个参数：

- 在其中运行函数的作用域(this)

- 参数数组，可以是Array的实例，也可以是arguments对象

`call`接受一到多个参数：

- 在其中运行函数的作用域(this)

- 传递给函数的参数，逐个列举

用`call()`或`apply()`来扩充作用域的最大好处，就是对象不需要与方法有任何的耦合关系。

## bind

`bind`这个方法会创建一个函数的实例，其this值会被绑定到传给bind()函数的值

💥注意：

- 参数可以通过bind和创建出的函数分两次传入
- 如果对创建出的函数调用new，通过bind传入的this会失效，被重新初始化为构造函数创建出的实例对象