# new的模拟实现

1. 首先新建一个对象
2. 然后将对象的原型指向 Person.prototype
3. 然后 Person.apply(obj)
4. 接受构造函数的返回值，如果是基本类型，没影响，如果是对象，new出来的实例只能访问该对象中的属性
5. 返回这个对象

```js
function new2(Constructor,...args){
   var obj = new Object();
   obj.__proto__ = Constructor.prototype;
   var ret = Constructor.apply(obj,args);
   return  typeof ret==='object'?ret:obj;
}
```



