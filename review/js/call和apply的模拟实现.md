# call的模拟实现

`call()方法在指定一个this和若干个参数的情况下调用某个函数或方法`



1.首先，让要执行的函数变成属于对象的方法（改变this指向）

2.然后把后续的参数传入对象的方法中

3.让方法执行

4.如果context是null或者undefined返回window对象

5.接受方法执行的返回值

6.删除对象的方法

7.返回方法执行的返回值

```js
Function.prototype.call2 = function(context){
	context = context||window
	context.fn = this;
	var args = [...args].slice(1).toString();
	var result = eval(context.fn(args));
	delete context.fn
	return result
}
```

# apply的模拟实现

`apply和call唯一的区别就是apply接受的第二个参数为数组`

```js
Function.prototype.applu2 = function(context){
	context = context||window
	context.fn = this;
	var [,...args] = arguments;
	var result = eval(context.fn(args));
	delete context.fn
	return result
}
```

