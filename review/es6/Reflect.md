### Reflect的设计目的

- 从`Reflect`对象上可以拿到语言内部的方法。
- 修改某些`Object`方法的返回结果，让其变得更合理。

- 让`Object`操作都变成函数行为。某些`Object`操作是命令式，比如`name in obj`和`delete obj[name]`，而`Reflect.has(obj, name)`和`Reflect.deleteProperty(obj, name)`让它们变成了函数行为。

- `Reflect`对象的方法与`Proxy`对象的方法**一一对应**。不管`Proxy`怎么修改默认行为，你总可以在`Reflect`上获取默认行为。

`Reflect`可以和`Proxy`配合使用：用`Proxy`方法拦截`target`对象的行为，用`Reflect`方法确保完成默认的行为，然后再部署额外的功能。下面是一个例子

```javascript
var loggedObj = new Proxy(obj, {
  get(target, name) {
    console.log('get', target, name);
    return Reflect.get(target, name);
  },
  deleteProperty(target, name) {
    console.log('delete' + name);
    return Reflect.deleteProperty(target, name);
  },
  has(target, name) {
    console.log('has' + name);
    return Reflect.has(target, name);
  }
});
```