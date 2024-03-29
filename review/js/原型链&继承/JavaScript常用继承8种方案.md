## 1.原型链继承

做法：父类实例作为子类原型，方法写在父类原型上

缺点：

- 子类实例共享父类实例的引用类型值，存在篡改的可能
- 创建子类无法向父类传参

```javascript
function Parent () {
    this.name = 'kevin';
}
Parent.prototype.getName = function () {
    console.log(this.name);
}
function Child () {
}
Child.prototype = new Parent();
var child1 = new Child();
console.log(child1.getName()) // kevin
```

## 2.借用构造函数(经典继承)

做法：子类用`call`调用父类

优点：原型链模式的缺点（1、引用类型不被实例共享2、子类可向父类传参）

缺点：

- 方法都在构造函数中定义，每次创建实例都会创建一遍方法，没有函数复用。
- 只能继承父类的实例属性和方法，不能继承父类的原型属性和方法。

```javascript
function Parent () {
    this.names = ['kevin', 'daisy'];
}
function Child () {
    Parent.call(this);
}
var child1 = new Child();
child1.names.push('yayu');
console.log(child1.names); // ["kevin", "daisy", "yayu"]
var child2 = new Child();
console.log(child2.names); // ["kevin", "daisy"]
```

## 3.组合继承

原型链继承和经典继承双剑合璧。

做法：1.父类实例作为子类原型，方法写在父类原型上2.子类用`call`调用父类

缺点：父类调用两遍，子类原型有冗余属性

```javascript
function Parent (name) {
    this.name = name;
    this.colors = ['red', 'blue', 'green'];
}
Parent.prototype.getName = function () {
    console.log(this.name)
}
function Child (name, age) {
    Parent.call(this, name);
    this.age = age;
}
Child.prototype = new Parent();
Child.prototype.constructor = Child;
var child1 = new Child('kevin', '18');
child1.colors.push('black');
console.log(child1.name); // kevin
console.log(child1.age); // 18
console.log(child1.colors); // ["red", "blue", "green", "black"]
var child2 = new Child('daisy', '20');
console.log(child2.name); // daisy
console.log(child2.age); // 20
console.log(child2.colors); // ["red", "blue", "green"]
```

## 4.原型式继承

做法：传入一个对象作为函数实例的原型。

```javascript
function createObj(o) {
    function F(){}
    F.prototype = o;
    return new F();
}
```

缺点：

跟原型链继承一样

- 多个实例引用类型的指向相同，存在篡改的可能
- 子类无法向父类传参

## 5. 寄生式继承

作用：在原型式继承的基础上增强对象

做法：1.基于原有的对象创造新对象2.给新对象添加方法

```javascript
function createObj (o) {
    var clone = Object.create(o);
    clone.sayName = function () {
        console.log('hi');
    }
    return clone;
}
```

缺点：

跟借用构造函数模式一样

- 每次创建对象都会创建一遍方法

跟原型式继承一样

- 多个实例引用类型的指向相同，存在篡改的可能
- 子类无法向父类传参

## 6. 寄生组合式继承

寄生组合继承=原型式继承+组合式继承

目的：为了解决组合继承中父类被调用两次(1.在子类中调用父类2.把父类实例作为子类原型)

做法：用寄生式继承对组合继承进行优化(创建父类和子类的中间层)

组合继承最大的缺点是会调用两次父构造函数，但是我们想要的只是父类的原型上的方法

```javascript
function Parent (name) {
    this.name = name;
    this.colors = ['red', 'blue', 'green'];
}
Parent.prototype.getName = function () {
    console.log(this.name)
}
function Child (name, age) {
    Parent.call(this, name);
    this.age = age;
}
Child.prototype = new Parent()//第一次是设置子类型实例的原型的时候
var child1 = new Child('kevin', '18');//第二次在创建子类型实例的时候
console.log(child1)
```

所以，`Child.prototype` 和 `child1` 都有一个属性为`colors`，属性值为`['red', 'blue', 'green']`。

为了避免这一次重复调用，我们不使用 `Child.prototype = new Parent()` ，而是间接的让 Child.prototype 访问到 `Parent.prototype`

看看如何实现：

```javascript
function Parent (name) {
    this.name = name;
    this.colors = ['red', 'blue', 'green'];
}
Parent.prototype.getName = function () {
    console.log(this.name)
}
function Child (name, age) {
    Parent.call(this, name);
    this.age = age;
}
//关键的三步----------------------------------------------------------------------------
var F = function () {};
F.prototype = Parent.prototype;
Child.prototype = new F();
--------------------------------------------------------------------------------------
var child1 = new Child('kevin', '18');
console.log(child1);
```

最后我们封装一下这个继承方法：

```javascript
function object(o) {
    function F() {}
    F.prototype = o;
    return new F();
}
function prototype(child, parent) {
    var prototype = object(parent.prototype);
    prototype.constructor = child;
    child.prototype = prototype;
}
// 当我们使用的时候：
prototype(Child, Parent);
```

寄生组合式继承只调用了一次 Parent 构造函数，并且因此**避免了在 Parent.prototype 上面创建多余的属性**。与此同时，原型链还能保持不变；因此，还能够正常使用 `instanceof` 和 `isPrototypeOf`。开发人员普遍认为寄生组合式继承是引用类型最理想的继承范式。

**这是最成熟的方法，也是现在库实现的方法**

## 7.混入方式继承多个对象
类似于组合继承，但是可以同时继承多个对象
1.假定A为想继承的父类1，B为想继承的父类2
2.在子类中用call方法执行A，B，子类实例可以获得父类1，父类2的属性
3.利用原型式继承，把继承父类1的属性的对象设为子类的原型，用`Object.assign`把父类2原型的属性拷贝到子类原型上，让子类的所用实例都可以用父类2的方法

```javascript
function MyClass() {
     SuperClass.call(this);
     OtherSuperClass.call(this);
}
// 继承一个类
MyClass.prototype = Object.create(SuperClass.prototype);
// 混合其它
Object.assign(MyClass.prototype, OtherSuperClass.prototype);
// 重新指定constructor
MyClass.prototype.constructor = MyClass;
MyClass.prototype.myMethod = function() {
     // do something
};
```

`Object.assign`会把 `OtherSuperClass`原型上的函数拷贝到 `MyClass`原型上，使 `MyClass` 的所有实例都可用 `OtherSuperClass` 的方法。

## 8. Class的继承
### 简介
做法：Class 可以通过`extends`关键字实现继承

**子类必须在`constructor`方法中调用`super`方法(父类的构造函数)**，这是因为子类自己的`this`对象，必须先通过父类的构造函数完成塑造，得到与父类同样的实例属性和方法，然后再对其进行加工，加上子类自己的实例属性和方法。如果不调用`super`方法，子类就得不到`this`对象。

```
子类this = 父类塑造+子类塑造
```

如果子类没有定义`constructor`方法，这个方法会被**默认添加**

**在子类的构造函数中，只有调用`super`之后，才可以使用`this`关键字**，否则会报错。这是因为子类实例的构建，基于父类实例，只有`super`方法才能调用父类实例。

**子类会继承父类的静态方法**

```javascript
class A {
  static hello() {
    console.log('hello world');
  }
}
class B extends A {
}

B.hello()  // hello world
```

### super 关键字

`super`这个关键字，既可以当作函数使用，也可以当作对象使用，使用`super`的时候，必须显式指定是作为函数、还是作为对象使用，否则会报错。

#### 当作函数使用

1. `super`作为函数调用时，代表**父类的构造函数**，ES6 要求，子类的构造函数必须执行一次`super`函数，如果不调用`super`方法，子类就得不到`this`对象。

```javascript
class A {}
class B extends A {
  constructor() {
    super();
  }
}
```

2. 注意，<font color='red'>`super`虽然代表了父类`A`的构造函数，但是返回的是子类`B`的实例，即`super`内部的`this`指的是`B`的实例</font>，因此`super()`在这里相当于`A.prototype.constructor.call(this)`。

```javascript
class A {
  constructor() {
    console.log(new.target.name);
  }
}
class B extends A {
  constructor() {
    super();
  }
}
new A() // A
new B() // B
```

上面代码中，`new.target`指向当前正在执行的函数。可以看到，在`super()`执行时，它指向的是子类`B`的构造函数，而不是父类`A`的构造函数。也就是说，`super()`内部的`this`指向的是`B``

3. `super()`**只能用在子类的构造函数**之中

#### 当作对象使用

1. 在**普通方法（包括constructor）**中，指向**父类的原型对象**；在**静态方法**中，指向**父类**。

   💥注意，在普通方法中，由于`super`指向父类的原型对象，所以<font color='red'>定义在父类实例上的方法或属性，是无法通过`super`调用的</font>

   ```javascript
   class A {
     constructor() {
       this.p = 2;
     }
   }
   A.prototype.x = 2;
   class B extends A {
     getValue () {
       console.log(super.p,super.x)
     }
   }
   
   let b = new B();
   b.getValue() // undefined 2
   ```

   上面代码中，`p`是父类`A`实例的属性，x是父类`A`原型的属性，可以看出super只能引用原型上的属性
   
2. 在普通方法中通过`super`调用父类的方法时，**<font color='red'>父类方法内部</font>的`this`指向当前的子类实例**


```javascript
class A {
  constructor() {
    this.x = 1;
  }
}

class B extends A {
  constructor() {
    super();
    this.x = 2;
    super.x = 3;
    console.log(super.x); // undefined
    console.log(this.x); // 3
  }
}

let b = new B();
```

上面代码中，`super.x`赋值为`3`，这时等同于对`this.x`赋值为`3`。而当读取`super.x`的时候，读的是`A.prototype.x`，所以返回`undefined`。

🎈所以可以总结得出，当把super作为对象使用时：

直接使用`super`

- `super`的指向：在普通方法中，指向父类的原型，在静态方法中，指向父类。

通过`super`调用父类方法，父类内部`this`的指向

- 在普通方法中指向子类实例，在静态方法中指向子类，

  💥注意：<font color='red'>如果通过`super`对某个属性赋值</font>，这时`super`指向等同于通过`super`调用父类方法，父类内部`this`的指向

```javascript
class A {}

class B extends A {
  constructor() {
    super();
    console.log(super); // 报错
  }
}
```

上面代码中，`console.log(super)`当中的`super`，无法看出是作为函数使用，还是作为对象使用，所以 JavaScript 引擎解析代码的时候就会报错。这时，如果能清晰地表明`super`的数据类型，就不会报错。

```javascript
class A {}

class B extends A {
  constructor() {
    super();
    console.log(super.valueOf() instanceof B); // true
  }
}

let b = new B();
```

上面代码中，`super.valueOf()`表明`super`是一个对象，因此就不会报错。同时，由于`super`使得`this`指向`B`的实例，所以`super.valueOf()`返回的是一个`B`的实例。

最后，由于对象总是继承其他对象的，所以可以在任意一个对象中，使用`super`关键字。

```javascript
var obj = {
  toString() {
    return "MyObject: " + super.toString();
  }
};

obj.toString(); // MyObject: [object Object]
```

### 类的 prototype 属性和__proto__属性 

Class同时有`prototype`属性和`__proto__`属性，因此同时存在两条继承链。

（1）子类的`__proto__`属性，表示构造函数的继承，总是指向父类。

（2）子类`prototype`属性的`__proto__`属性，表示方法的继承，总是指向父类的`prototype`属性。

### 原生构造函数的继承

以前原生构造函数是无法继承的，因为原生构造函数的`this`无法绑定，导致子类无法获得原生构造函数的内部属性

**ES5 是先新建子类的实例对象`this`，再将父类的属性添加到实例上**，由于父类的内部属性无法获取，导致无法继承。ES6 允许继承原生构造函数定义子类，**因为 ES6 是先新建父类的实例对象`this`，然后再用子类的构造函数修饰`this`，使得父类的所有行为都可以继承**。

## 总结

**（1）严格模式**

类和模块的内部，**默认就是严格模式**，所以不需要使用`use strict`指定运行模式。

**（2）不存在提升**

**类不存在变量提升**（hoist），这是因为**必须保证子类在父类之后定义**。

**（3）继承的差别**

`ES5`的继承实质上是**先创建子类的实例对象**，然后再将父类的属性和方法添加到this上

`ES6`的继承有所不同，先新建父类的实例对象`this`，然后再用子类的构造函数修饰`this`，使得父类的所有行为都可以继承