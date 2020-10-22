### 什么是`Proxy`(可以把`Proxy`和`Reflect`配合回答)

`Proxy`用于修改某些操作的默认行为，也可以理解为在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种**代理**机制，可以对外界的访问进行过滤和改写。

```javascript
var proxy = new Proxy(target, handler);
```

`Proxy`接受两个参数，第一个参数`target`代表拦截的目标，第二个参数`handler`代表拦截的行为，如果**`handler`没有设置任何拦截，那就直通原对象。**

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
- **ownKeys(target)**：对象自身的读取操作（对象遍历方法，除了`Reflect.ownKeys`)，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而`Object.keys()`的返回结果仅包括目标对象**<font color='red'>自身</font>**的**<font color='red'>可遍历</font>**的**<font color='red'>非Symbol</font>**属性，`for...in`的返回结果为目标对象**<font color='red'>自身或继承的</font>**的**<font color='red'>可遍历</font>**的**<font color='red'>非Symbol</font>**属性。
- **getOwnPropertyDescriptor(target, propKey)**：拦截`Object.getOwnPropertyDescriptor(proxy, propKey)`，返回属性的描述对象。
- **defineProperty(target, propKey, propDesc)**：拦截`Object.defineProperty(proxy, propKey, propDesc）`、`Object.defineProperties(proxy, propDescs)`，返回一个布尔值。
- **preventExtensions(target)**：拦截`Object.preventExtensions(proxy)`，返回一个布尔值。
- **getPrototypeOf(target)**：拦截`Object.getPrototypeOf(proxy)`，返回一个对象。
- **isExtensible(target)**：拦截`Object.isExtensible(proxy)`，返回一个布尔值。
- **setPrototypeOf(target, proto)**：拦截`Object.setPrototypeOf(proxy, proto)`，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。

当 `Proxy` 实例作为函数时，还有以下两种拦截：

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

`Proxy.revocable`方法**返回一个对象**，该对象的`proxy`属性是`Proxy`实例，`revoke`属性是一个函数，可以取消`Proxy`实例。

`Proxy.revocable`的一个使用场景是，**目标对象必须通过代理访问，访问结束就收回代理权，不允许再次访问。**

### `this` 问题 

```javascript
const target = {
  m: function () {
    console.log(this === proxy);
  }
};
const handler = {};

const proxy = new Proxy(target, handler);

target.m() // false
proxy.m()  // true
```

目标对象内部的`this`关键字会指向 `Proxy` 代理，`this`绑定原始对象，就可以解决这个问题。

### `proxy`和`Object.definedProperty`区别

- 使用 `defineProperty` 只能拦截属性的读取（get）和设置（set）行为， `Proxy`，可以拦截13种行为，比如 in、delete、函数调用等更多行为
- `Object.defineProperty()`只能对某个`key`进行监测，如果想对每个属性都监测的话就需要遍历，而`Proxy`是直接监测整个对象，不需要遍历
- `Object.defineProperty`无法监控到数组下标的变化，导致直接通过数组的下标给数组设置值，不能实时响应。
- 当使用 `defineProperty`，我们修改原来的 obj 对象就可以触发拦截，而使用 proxy，就必须修改代理对象，即 `Proxy` 的实例才可以触发拦截。
- `proxy`的兼容性目前还没有提上来，并且对应的`polyfill`也不完善

### 用proxy实现观察者机制

```javascript
observe(data) {
        const that = this;
        let handler = {
         get(target, property) {
            return target[property];
          },
          set(target, key, value) {
            let res = Reflect.set(target, key, value);
            that.subscribe[key].map(item => {
              item.update();
            });
            return res;
          }
        }
        this.$data = new Proxy(data, handler);
      }
```
