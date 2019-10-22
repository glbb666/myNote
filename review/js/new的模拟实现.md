# new的模拟实现

1.可以访问到构造函数的属性

2.可以访问到构造函数原型的属性

3.构造函数的返回值：如果是基本类型，没影响，如果是对象，new出来的实例只能访问该对象中的属性

```js
function new2(Constructor,...args){
   var obj = new Object();
   obj.__proto__ = Constructor.prototype;
   var ret = Constructor.apply(obj,args);
   return  typeof ret==='object'?ret:obj;
}
```



