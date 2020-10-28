# call的模拟实现

`call()方法在指定一个this和若干个参数的情况下调用某个函数或方法，传递给call的参数必须依次列举出来`

```js
Function.prototype.call2 = function(context,...args){
	context =context||window;
	context.fn = this;
	let result =context.fn(...args）;
	delete context.fn;
	return result;
}
```
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

