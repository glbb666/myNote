### 什么是`Proxy`

`Proxy`用于修改某些操作的默认行为，也可以理解为在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种**代理**机制，可以对外界的访问进行过滤和改写。

`ES6` 原生提供 `Proxy` 构造函数，用来生成 `Proxy` 实例。

```javascript
var proxy = new Proxy(target, handler);
```

`target`参数表示所要拦截的目标对象，`handler`参数也是一个对象，用来定制拦截行为，如果`handler`没有设置任何拦截，那就等同于直接通向原对象。

```javascript
var proxy = new Proxy({}, {
  get: function(target, property) {
    return 35;
  }
});

proxy.time // 35
proxy.name // 35
proxy.title // 35
```

🌟注意，要使得`Proxy`起作用，必须针对`Proxy`实例（上例是`proxy`对象）进行操作，而不是针对目标对象（上例是空对象）进行操作。

### proxy能拦截什么

下面是 Proxy 支持的拦截操作一览，一共 **13 种**。

- **get(target, propKey, receiver)**：拦截对象属性的读取，比如`proxy.foo`和`proxy['foo']`。
- **set(target, propKey, value, receiver)**：拦截对象属性的设置，比如`proxy.foo = v`或`proxy['foo'] = v`，返回一个布尔值。
- **has(target, propKey)**：拦截`propKey in proxy`的操作，返回一个布尔值。`has`拦截只对`in`运算符生效，对`for...in`循环不生效。可以用来隐藏属性。
- **deleteProperty(target, propKey)**：拦截`delete proxy[propKey]`的操作，返回一个布尔值。
- **ownKeys(target)**：拦截`Object.getOwnPropertyNames(proxy)`、`Object.getOwnPropertySymbols(proxy)`、`Object.keys(proxy)`、`for...in`循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而`Object.keys()`的返回结果仅包括目标对象**<font color='red'>自身</font>**的**<font color='red'>可遍历</font>**的**<font color='red'>非Symbol</font>**属性，`for...in`的返回结果为目标对象**<font color='red'>自身或继承的</font>**的**<font color='red'>可遍历</font>**的**<font color='red'>非Symbol</font>**属性。
- **getOwnPropertyDescriptor(target, propKey)**：拦截`Object.getOwnPropertyDescriptor(proxy, propKey)`，返回属性的描述对象。
- **defineProperty(target, propKey, propDesc)**：拦截`Object.defineProperty(proxy, propKey, propDesc）`、`Object.defineProperties(proxy, propDescs)`，返回一个布尔值。
- **preventExtensions(target)**：拦截`Object.preventExtensions(proxy)`，返回一个布尔值。
- **getPrototypeOf(target)**：拦截`Object.getPrototypeOf(proxy)`，返回一个对象。
- **isExtensible(target)**：拦截`Object.isExtensible(proxy)`，返回一个布尔值。
- **setPrototypeOf(target, proto)**：拦截`Object.setPrototypeOf(proxy, proto)`，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。
- **apply(target, object, args)**：拦截 Proxy 实例作为函数调用的操作，比如`proxy(...args)`、`proxy.call(object, ...args)`、`proxy.apply(...)`。
- **construct(target, args)**：拦截 Proxy 实例作为构造函数调用的操作，比如`new proxy(...args)`，返回值必须是一个对象。

### Proxy.revocable()

`Proxy.revocable`方法返回一个可取消的 Proxy 实例。

```javascript
let target = {};
let handler = {};

let {proxy, revoke} = Proxy.revocable(target, handler);

proxy.foo = 123;
proxy.foo // 123

revoke();
proxy.foo // TypeError: Revoked
```

`Proxy.revocable`方法返回一个对象，该对象的`proxy`属性是`Proxy`实例，`revoke`属性是一个函数，可以取消`Proxy`实例。

`Proxy.revocable`的一个使用场景是，目标对象必须通过代理访问，访问结束就收回代理权，不允许再次访问。

### this 问题 

目标对象内部的`this`关键字会指向 Proxy 代理，`this`绑定原始对象，就可以解决这个问题。