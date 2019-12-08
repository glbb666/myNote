实现add函数。add(1)(2)(3)

```js
//定项，函数柯里化
function curry(fn, ...args) {
      return function(){
      	var _args = [...args,...arguments];
      	return _args.length<fn.length?curry.call(this,fn,..._args):fn.apply(this,_args)
      }
}
function add1(a,b,c,d){
    return a+b+c+d
}
let add = curry(add1,0)
//不定项的
function add(...a) {
    let sum = a.reduce((p, n) => p + n);
    function next(...b) {
        let _sum = b.reduce((p, n) => p + n);
        sum = sum + _sum;
        return next;
    }
    next.toString = function () {
        return sum;
    };
    return next;
}
```