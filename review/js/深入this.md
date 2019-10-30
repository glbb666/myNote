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

  箭头函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象。

# 改变this指向的方法

- call
- apply
- bind