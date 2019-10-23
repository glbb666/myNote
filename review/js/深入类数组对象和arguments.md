# 深入类数组对象和arguments

## 类数组对象

所谓的类数组对象:

> 拥有一个 length 属性和若干索引属性的对象

## 调用数组方法

我们可以用 Function.call 间接调用：

```
var arrayLike = {0: 'name', 1: 'age', 2: 'sex', length: 3 }

Array.prototype.join.call(arrayLike, '&'); // name&age&sex

Array.prototype.slice.call(arrayLike, 0); // ["name", "age", "sex"] 
// slice可以做到类数组转数组

Array.prototype.map.call(arrayLike, function(item){
    return item.toUpperCase();
}); 
// ["NAME", "AGE", "SEX"]
```