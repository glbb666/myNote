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