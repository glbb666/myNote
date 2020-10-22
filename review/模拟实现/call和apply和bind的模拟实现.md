# call的模拟实现

`call()方法在指定一个this和若干个参数的情况下调用某个函数或方法，传递给call的参数必须依次列举出来`

1.首先，让要执行的函数变成属于对象的方法（改变this指向）

2.然后把后续的参数传入对象的方法中

3.让方法执行

4.如果context是null或者undefined返回window对象

5.接受方法执行的返回值

6.删除对象的方法

7.返回方法执行的返回值

我的es6实现

```js
Function.prototype.call2 = function(context,...args){
	context =context||window;
	context.fn = this;
	var result =context.fn(...args）;
	delete context.fn;
	return result;
}
```
讶羽的es5实现
```js
Function.prototype.call2 = function(context){
    context = context||window;
    context.fn = this;
    var args = [];
    for(let i = 1;len=arguments.length,i<len;i++){
        args.push('arguments['+i+']');
    }
    //这步执行完之后args = ['arguments[1]','arguments[2]'...]这样的一个元素为字符串的数组
    var result = eval('context.fn('+args+')');
    //因为eval内部传入实参应该传'"kevin"'，不然会被当作变量
    //这里的args会自动调用toString()方法
    delete context.fn;
    return result;
}
```

> 这个代码有一点没覆盖，就是当值为原始值（数字，字符串，布尔值）的 this 会指向该原始值的自动包装对象。

# apply的模拟实现

`apply和call唯一的区别就是apply接受的第二个参数为数组或者是类数组对象`

```js
Function.prototype.apply2 = function(context,...args){
	context = context||window;
	context.fn = this;
    //下面这一步是为了避免context的属性被引用导致context无法释放。
	let result = context.fn(...args);
	delete context.fn;
	return result;
}
```

#  bind的模拟实现

`bind方法会创建一个函数，当这个新函数被调用，bind()的第一个参数将会作为它运行时的this，之后的一序列参数将会在传递的实参前传入作为它的参数。`

1.返回一个函数

2.可以分两次传参（bind时以及执行bind返回的函数时）

3.如果把返回的函数当作构造函数，那么相当于把原函数当作一个构造器，提供的this值被忽略，同时调用时的参数被提供给模拟函数

4.对于返回的函数，可以通过判断this指向来判断有没有被当成构造函数。具体的实现是让返回的函数的原型指向原函数的原型（因为原函数是一个构造器），但是为了防止原型上的属性被修改，应该加一个中间层

5.在返回的函数中的实现就类似于call了

```js
Function.prototype.bind2 = function(context,...args){
    	if(typeof this!=='function'){
           throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
        }
        var self = this;
        var fNOP = function(){}
        var fBound = function(){
            var bindArgs = [...args,...arguments];
            var _this = this instanceof self?this:context;
            _this.fn = self;
            var result = _this.fn(...bindArgs);
            delete _this.fn;
            return result;
        }
        fNOP.prototype = this.prototype;
        fBound.prototype = new fNOP();
        return fBound;
}
```

